---
layout: post
title: 深入理解Java虚拟机（六）——类加载器(ClassLoader)
categories: 深入理解JVM
description: 深入理解Java虚拟机（六）——类加载器(ClassLoader)
keywords: Java虚拟机，JVM
---

## 1. 类加载器

类加载器(ClassLoader)用来加载 class字节码到 Java 虚拟机中。

一般来说，Java 虚拟机使用 Java 类的方式如下：Java 源文件在经过 Javac之后就被转换成 Java 字节码文件(.class 文件)。类加载器负责读取 Java 字节代码，并转换成 java.lang.Class 类的一个实例。每一个这样的实例用来表示一个 Java 类。实际的情况可能更加复杂，比如 Java 字节代码可能是通过工具动态生成的，也可能是通过网络下载。

## 2. 类与类加载器

类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远远不限于类加载阶段。对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟中的唯一性。

说通俗一些，比较两个类是否“相等”，只有在两个类是由同一个类加载器的前提之下才有意义，否则，即使这两个类来源于同一个class文件，只要加载它的类加载器不同，那这两个类必定不相等。这里所指的“相等”包括代表类的Class对象的equal方法、isAssignableFrom()、isInstance()方法及instance关键字返回的结果。

## 3. 类加载器分类

![class-loader.png](http://i.imgur.com/4ElERtQ.jpg)
主要分为Bootstrap ClassLoader、Extension ClassLoader、Application ClassLoader和User Defined ClassLoader。

### 3.1 启动类加载器(Bootstrap ClassLoader)

这个类加载器使用C++语言实现，并非ClassLoader的子类。主要负责加载存放在JAVA_HOME /  jre /  lib / rt.jar里面所有的class文件，或者被-Xbootclasspath参数所指定路径中以rt.jar命名的文件。

### 3.2 扩展类加载器(Extension ClassLoader)

这个加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载AVA_HOME /  lib / ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库。

### 3.3 应用程序类加载器(Application ClassLoader)

这个加载器由sun.misc.Launcher$AppClassLoader实现，它负责加载classpath对应的jar及目录。一般情况下这个就是程序中默认的类加载器。

### 3.4 自定义类加载器(User Defined ClassLoader)

开发人员继承ClassLoader抽象类自行实现的类加载器，基于自行开发的ClassLoader可用于并非加载classpath中(例如从网络上下载的jar或二进制字节码)、还可以在加载class文件之前做些小动作 如：加密等。

## 4. 双亲委托模型：

上图中所展示的类加载器之间的这种层次关系，就称为类加载器的双亲委托模型。双亲委托模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。这里类加载器之间的父子关系一般不会以继承的关系来实现，而是使用组合关系来复用父加载器的代码。

```java
public abstract class ClassLoader {  
  
    private static native void registerNatives();  
    static {  
        registerNatives();  
    }  
  
    // The parent class loader for delegation  
    private ClassLoader parent;  
  
    // Hashtable that maps packages to certs  
    private Hashtable package2certs = new Hashtable(11);  
}  
```

双亲委托的工作过程：如果一个类加载器收到了一个类加载请求，它首先不会自己去加载这个类，而是把这个请求委托给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父类加载器反馈自己无法完成加载请求(它管理的范围之中没有这个类)时，子加载器才会尝试着自己去加载。

使用双亲委托模型来组织类加载器之间的关系，有一个显而易见的好处就是Java类随着它的类加载器一起具备了一种带有优先级的层次关系，例如java.lang.Object存放在rt.jar之中，无论那个类加载器要加载这个类，最终都是委托给启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类，相反，如果没有双亲委托模型，由各个类加载器去完成的话，如果用户自己写一个名为java.lang.Object的类，并放在classpath中，应用程序中可能会出现多个不同的Object类，java类型体系中最基本安全行为也就无法保证。

## 5. 类加载器SPI：

java.lang.ClassLoader 类提供的几个关键方法：

### 5.1 loadClass

此方法负责加载指定名字的类，首先会从已加载的类中去寻找，如果没有找到；从parent ClassLoader[ExtClassLoader]中加载；如果没有加载到，则从Bootstrap ClassLoader中尝试加载(findBootstrapClassOrNull方法), 如果还是加载失败，则抛出异常ClassNotFoundException, 在调用自己的findClass方法进行加载。如果要改变类的加载顺序可以覆盖此方法；如果加载顺序相同，则可以通过覆盖findClass方法来做特殊处理，例如：解密，固定路径寻找等。当通过整个寻找类的过程仍然未获取Class对象，则抛出ClassNotFoundException异常。

如果类需要resolve，在调用resolveClass进行链接。

```java
   protected synchronized Class<?> loadClass(String name, boolean resolve)  
throws ClassNotFoundException  
   {  
// First, check if the class has already been loaded  
Class c = findLoadedClass(name);  
if (c == null) {  
    try {  
    if (parent != null) {  
        c = parent.loadClass(name, false);  
    } else {  
        c = findBootstrapClassOrNull(name);  
    }  
    } catch (ClassNotFoundException e) {  
               // ClassNotFoundException thrown if class not found  
               // from the non-null parent class loader  
           }  
           if (c == null) {  
        // If still not found, then invoke findClass in order  
        // to find the class.  
        c = findClass(name);  
    }  
}  
if (resolve) {  
    resolveClass(c);  
}  
return c;  
   }  
```

### 5.2 findLoadedClass 
此方法负责从当前ClassLoader实例对象的缓存中寻找已加载的类，调用的为native方法。

```java
protected final Class<?> findLoadedClass(String name) {  
(!checkName(name))  
 return null;  
urn findLoadedClass0(name);  
}  
private native final Class findLoadedClass0(String name);  
```

### 5.3 findClass 
此方法直接抛出ClassNotFoundException异常，因此要通过覆盖loadClass或此方法来以自定义的方式加载相应的类。

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {  
ow new ClassNotFoundException(name);  
}  
```

### 5.4 findSystemClass  
此方法是从sun.misc.Launcher$AppClassLoader中寻找类，如果未找到，则继续从BootstrapClassLoader中寻找，如果仍然未找到，返回null

```java
   protected final Class<?> findSystemClass(String name)  
throws ClassNotFoundException  
   {  
ClassLoader system = getSystemClassLoader();  
if (system == null) {  
    if (!checkName(name))  
    throw new ClassNotFoundException(name);  
           Class cls = findBootstrapClass(name);  
           if (cls == null) {  
               throw new ClassNotFoundException(name);  
           }   
    return cls;  
}  
return system.loadClass(name);  
   }  
```

### 5.5 defineClass 
此方法负责将二进制字节流转换为Class对象，这个方法对于自定义类加载器而言非常重要。如果二进制的字节码的格式不符合jvm class文件格式规范，则抛出ClassFormatError异常；如果生成的类名和二进制字节码不同，则抛出NoClassDefFoundError；如果加载的class是受保护的、采用不同签名的，或者类名是以java.开头的，则抛出SecurityException异常。

```java
protected final Class<?> defineClass(String name, byte[] b, int off, int len,  
                     ProtectionDomain protectionDomain)  
    throws ClassFormatError  
    {  
         return defineClassCond(name, b, off, len, protectionDomain, true);  
    }  
  
    // Private method w/ an extra argument for skipping class verification  
    private final Class<?> defineClassCond(String name,  
                                           byte[] b, int off, int len,  
                                           ProtectionDomain protectionDomain,  
                                           boolean verify)  
        throws ClassFormatError  
    {  
    protectionDomain = preDefineClass(name, protectionDomain);  
  
    Class c = null;  
        String source = defineClassSourceLocation(protectionDomain);  
  
    try {  
        c = defineClass1(name, b, off, len, protectionDomain, source,  
                             verify);  
    } catch (ClassFormatError cfe) {  
        c = defineTransformedClass(name, b, off, len, protectionDomain, cfe,  
                                       source, verify);  
    }  
  
    postDefineClass(c, protectionDomain);  
    return c;  
    }  
```

### 5.6 resolveClass 
此方法负责完成Class对象的链接，如果链接过，则直接返回。

## 6. 常见异常

### 6.1 ClassNotFoundException  
这是最常见的异常，产生这个异常的原因为在当前的ClassLoader 中加载类时，未找到类文件。

### 6.2 NoClassDefFoundError  
这个异常是因为加载到的类中引用到的另外类不存在，例如要加载A，而A中引用到了B，B不存在或当前的ClassLoader无法加载B，就会抛出这个异常。

### 6.3 LinkageError 
该异常在自定义ClassLoader的情况下更容易出现，主要原因是此类已经在ClassLoader加载过了，重复的加载会造成该异常。