---
layout: post
title: Tomcat中如何保证多个应用程序的类库互相独立
categories: Tomcat 
description: Tomcat中如何保证多个应用程序的类库互相独立
keywords: Tomcat 依赖冲突
---

部署在同一个服务器上的多个Web应用程序可能会依赖同一个第三方类库的不同版本，服务器应当保证多个应用程序的类库可以互相独立使用，那么服务器是如何实现的呢？先看了一下《深入理解JVM》，但是还是没有很好地理解，于是网上搜了一下，很多都是摘抄自《深入理解JVM》，但是对于WebAppClassLoader具体是如何相互隔离的说的并不是很明白，于是就看了一下WebAppClassLoader的源码。

首先先看一下《深入理解JVM》中的内容：

## 1 Web服务器要解决的问题

一个功能健全的Web服务器，要解决如下几个问题：

1. 部署在同一个服务器上的两个Web应用程序所使用的Java类库可以实现相互隔离。
2. 部署在同一个服务器上的两个Web应用程序所使用的Java类库可以相互共享。
3. 服务器需要尽可能地保证自身的安全不受部署的Web应用程序影响。
4. 支持JSP应用的Web服务器，大多数都需要支持HotSwap功能。“主流”的Web服务器都会支持JSP生成类的热替换，当然也有“非主流”的，运行在生产模式（Production Mode）下的WebLogic服务器默认就不会处理JSP文件的变化。

## 2 Tomcat5以及之前版本服务器的类加载架构

  在Tomcat目录结构中，有3组目录（“/common/\*”，“/server/\*”，“/shared/\*”）可以存放Java类库,另外还可以加上Web应用程序自身的目录“/WEB-INF/\*”:

需要注意的是：这些目录是Tomcat5以及之前版本的目录，Tomcat6以及之后的目录并不是这样子的。

1. 放置在/common目录中：类库可被Tomcat和所有的Web应用程序共同使用。
2. 放置在/server目录中：类库可被Tomcat使用，对所有的Web应用程序都不可见。
3. 放置在/shared目录中：类库可被所有的Web应用程序共同使用，但对Tomcat自己不可见。
4. 放置在/WebApp/WEB-INF目录中：类库仅仅可以被此Web应用程序使用，对Tomcat和其它Web应用程序都不可见。

Tomcat5以及之前版本服务器的类加载架构：
![](http://i.imgur.com/yQSWBlh.jpg)

其中WebApp类加载器和Jsp类加载器通常会存在多个实例，每一个Web应用程序对应一个WebApp类加载器，每一个JSP文件对应一个Jsp类加载器。

从上图可以看出：CommonClassLoader能加载的类都可以被CatalinaClassLoader和SharedClassLoader使用，而CatalinaClassLoader和SharedClassLoader自己能加载的类则与对方相互隔离。WebAppClassLoader可以使用SharedClassLoader加载到的类，但各个WebAppClassLoader实例直接相互隔离。而JasperLoader的加载范围仅仅是这个JSP文件所编译出来的那一个Class，它出现的目的就是为了被丢弃：当服务器检测到JSP文件被修改时，会替换掉目前的JasperLoader的实例，并通过再建立一个新的Jsp类加载器来实现JSP文件的HotSwap功能。

## 3 Tomcat6以及之后版本服务器的类加载架构

对于Tomcat的6.x版本，只有指定了Tomcat/conf/catalina.properties配置文件的server.loader和share.loader项后才会真正建立CatalinaClassLoader和SharedClassLoader的实例，否则会用到这两个类加载器的地方都会用CommonClassLoader的实例代替，而默认的配置文件中没有设置这两个loader项，所以Tomcat6.x顺理成章地把/common、/server和/shared三个目录默认合并到一起变成一个/lib目录，这个目录里的类库相当于以前/common目录中类库的作用。

Tomcat6以及之后版本服务器的类加载架构：
![](http://i.imgur.com/crujN4R.jpg)

## 4 WebAppClassLoader中的loadClass方法

```java
public Class loadClass(String name, boolean resolve)
		throws ClassNotFoundException {

		if (log.isDebugEnabled())
			log.debug("loadClass(" + name + ", " + resolve + ")");
		Class clazz = null;

		if (!started) {
			try {
				throw new IllegalStateException();
			} catch (IllegalStateException e) {
				log.info(sm.getString("webappClassLoader.stopped", name), e);
			}
		}

		// (0) 检查WebappClassLoader之前是否已经load过这个资源
		clazz = findLoadedClass0(name);
		if (clazz != null) {
			if (log.isDebugEnabled())
				log.debug("  Returning class from cache");
			if (resolve)
				resolveClass(clazz);
			return (clazz);
		}

		// (0.1) 检查ClassLoader之前是否已经load过
		clazz = findLoadedClass(name);
		if (clazz != null) {
			if (log.isDebugEnabled())
				log.debug("  Returning class from cache");
			if (resolve)
				resolveClass(clazz);
			return (clazz);
		}

		// (0.2) 先检查系统ClassLoader，因此WEB-INF/lib和WEB-INF/classes或{tomcat}/libs下的类定义不能覆盖JVM 底层能够查找到的定义
		try {
			clazz = system.loadClass(name);
			if (clazz != null) {
				if (resolve)
					resolveClass(clazz);
				return (clazz);
			}
		} catch (ClassNotFoundException e) {
			// Ignore
		}

		// (0.5) 使用SecurityManager时访问此类的权限
		if (securityManager != null) {
			int i = name.lastIndexOf('.');
			if (i >= 0) {
				try {
					securityManager.checkPackageAccess(name.substring(0,i));
				} catch (SecurityException se) {
					String error = "Security Violation, attempt to use " +
						"Restricted Class: " + name;
					log.info(error, se);
					throw new ClassNotFoundException(error, se);
				}
			}
		}

		//JVM的类加载机制建议先由parent去load，load不到自己再去load，而Servelet规范的建议则恰好相反，Tomcat的实现则做个折中，由用户去决定(context的 delegate定义)，默认使用Servlet规范的建议，即delegate=false

		boolean delegateLoad = delegate || filter(name);

		// (1) 先由parent去尝试加载，如上说明，除非设置了delegate，否则这里不执行
		if (delegateLoad) {
			if (log.isDebugEnabled())
				log.debug("  Delegating to parent classloader1 " + parent);
			ClassLoader loader = parent;
			
			if (loader == null)
				loader = system;
			try {
				clazz = loader.loadClass(name);
				if (clazz != null) {
					if (log.isDebugEnabled())
						log.debug("  Loading class from parent");
					if (resolve)
						resolveClass(clazz);
					return (clazz);
				}
			} catch (ClassNotFoundException e) {
				;
			}
		}

		// (2) 到WEB-INF/lib和WEB-INF/classes目录去搜索，细节部分可以再看一下findClass，会发现默认是先搜索WEB-INF/classes后搜索WEB-INF/lib
		if (log.isDebugEnabled())
			log.debug("  Searching local repositories");
		try {
			clazz = findClass(name);
			if (clazz != null) {
				if (log.isDebugEnabled())
					log.debug("  Loading class from local repository");
				if (resolve)
					resolveClass(clazz);
				return (clazz);
			}
		} catch (ClassNotFoundException e) {
			;
		}

		// (3) 由parent再去尝试加载一下
		if (!delegateLoad) {
			if (log.isDebugEnabled())
				log.debug("  Delegating to parent classloader at end: " + parent);
			ClassLoader loader = parent;
			if (loader == null)
				loader = system;
			try {
				clazz = loader.loadClass(name);
				if (clazz != null) {
					if (log.isDebugEnabled())
						log.debug("  Loading class from parent");
					if (resolve)
						resolveClass(clazz);
					return (clazz);
				}
			} catch (ClassNotFoundException e) {
				;
			}
		}

		throw new ClassNotFoundException(name);
	}
```

通过上面的源代码，可以看出来Tomcat在加载webapp级别的类的时候，默认情况下是不遵守parent-first的，这样做便更好的实现了应用的隔离。



