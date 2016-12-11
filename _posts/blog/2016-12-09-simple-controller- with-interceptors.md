---
layout: post
title: JavaWeb实现简单的登录验证(实现拦截器来记录日志)
categories: JavaWeb
description: JavaWeb实现简单的登录验证(实现拦截器来记录日志)
keywords: Java，Servlet，JSP，controller，interceptor
---

　　在[JavaWeb实现简单的登录验证(controller采用配置文件)](http://qixingjun.tech/2016/12/03/simple-controller-based-on-configuration-file/)的基础上增加拦截器的功能：定义一个POJO LogWriter作为拦截器，在该类中定义方法log（），log方法实现的功能为记录每次客户端请求的action名称、类型、访问开始时间、访问结束时间、请求返回结果result值，并将信息追加至log.xml，保存在PC磁盘上，这里主要就是用到了xml的写入。。

***

　　附，源码地址：[simplecontroller](https://github.com/QiXingjun/simplecontroller)

## 1 开发环境

- IDE：Intellij IDEA
- JDK：1.7
- Tomcat：7

## 2 项目结构
![](http://i.imgur.com/TPuzdsk.png)

## 3 详细开发

　　在[JavaWeb实现简单的登录验证(controller采用配置文件)](http://qixingjun.tech/2016/12/03/simple-controller-based-on-configuration-file/)的基础上增加拦截器的功能：定义一个POJO LogWriter作为拦截器，在该类中定义方法log（），log方法实现的功能为记录每次客户端请求的action名称、类型、访问开始时间、访问结束时间、请求返回结果result值，并将信息追加至log.xml，保存在PC磁盘上，这里主要就是用到了xml的写入。

　　具体来讲，dao层和service层的变化不大。当客户端请求某个类型Action时，控制器检查该action是否配置了LogWriter拦截器。如果有配置，在Action执行之前，使用LogWriter记录Action的名称、类型、访问开始时间，并在Action执行之后记录Action访问结束时间，及返回的结果result，将以上内容追加到log.xml。如果无配置，则直接访问Action。采用动态代理实现的拦截器，所以建立了一个LoginActionFactory的类，实现在调用LoginAction之前首先生成LoginAction的代理。由于是这次作业任然需要解析配置文件的方式，所以根据配置文件的结构，还需要一个封装Interceptor节点的InterceptorXmlBean。


### 3.1 简单java对象(Bean）

```java
package com.qixingjun.pojo;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.3.0
 * @Date 2016/12/6
 * @Description 封装interceptor节点
 */
public class InterceptorXmlBean {
    // 请求路径名称
    private String name;
    // 处理interceptor类的全名
    private String className;
    // 处理方法
    private String method;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getMethod() {
        return method;
    }

    public void setMethod(String method) {
        this.method = method;
    }

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }
}
```

### 3.2 拦截器(LogWriter)

```java
package com.qixingjun.interceptor;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;

import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStreamWriter;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.3.0
 * @Date 2016/12/6
 * @Description 拦截器,在该类中定义方法log()，log 方法实现的功能为记录每次客户端请求的action 名称、类型、访问开始时间、访问结束时间、请求返回结果result 值，并将信息追加至 log.xml，保存在PC 磁盘上。
 */
public class LogWriter {
    public void log(String name,String type,String s_time,String e_time,String result){
        //读取配置文件
        try {
            // 1.得到解析器
            SAXReader reader = new SAXReader();

            // 2.加载文件
            InputStream inputStream = this.getClass().getResourceAsStream("/log.xml");
            Document doc = reader.read(inputStream);

            //3.获取根
            Element root = doc.getRootElement();

            //4.添加action节点
            Element newActionElement = root.addElement("action");

            //5.在新的action节点下，添加name,type,s-time,e-time,result节点
            Element newNameElement = newActionElement.addElement("name");
            newNameElement.setText(name);
            Element newTypeElement = newActionElement.addElement("type");
            newTypeElement.setText(type);
            Element newSTimeElement = newActionElement.addElement("s_time");
            newSTimeElement.setText(s_time);
            Element newETimeElement = newActionElement.addElement("e_time");
            newETimeElement.setText(e_time);
            Element newResultElement = newActionElement.addElement("result");
            newResultElement.setText(result);
            writer(doc);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    /**
     * 把document对象写入新的文件
     * @param document
     * @throws Exception
     */
    public void writer(Document document) throws Exception {
        // 紧凑的格式
        // OutputFormat format = OutputFormat.createCompactFormat();
        // 排版缩进的格式
        OutputFormat format = OutputFormat.createPrettyPrint();
        // 设置编码
        format.setEncoding("UTF-8");
        // 创建XMLWriter对象,指定了写出文件及编码格式
        XMLWriter writer = new XMLWriter(new OutputStreamWriter(
                new FileOutputStream(new File("G:/USTC/IdeaWorkSpace/simplecontroller/src/log.xml")), "UTF-8"), format);
        // 写入
        writer.write(document);
        // 立即写入
        writer.flush();
        // 关闭操作
        writer.close();
    }
}
```

### 3.3 请求处理层(LoginActionFactory)

 　　由于需要使用动态代理来实现拦截器（使用的是JDK动态代理），故将LoginAction拆分成了ILoginAction和LoginActionImpl，即：LoginAction的接口和实现类，并且写一个代理工厂类LoginActionFactory，返回LoginAction的代理代理。

```java
package com.qixingjun.action;

import com.qixingjun.action.impl.LoginActionImpl;
import com.qixingjun.pojo.ActionXmlBean;
import com.qixingjun.pojo.InterceptorXmlBean;
import com.qixingjun.util.ParseXml;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/12/8
 * @Description 代理对象的工厂
 */
public class LoginActionFactory {
    private static ILoginAction loginAction = new LoginActionImpl();
    public static ILoginAction getInstance(){
        ILoginAction loginActionProxy = (ILoginAction) Proxy.newProxyInstance(loginAction.getClass().getClassLoader(), loginAction.getClass().getInterfaces(),
                new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                HttpServletRequest request = (HttpServletRequest) args[0];
                //获取请求的uri,得到请求路径名称
                String uri = request.getRequestURI();
                //得到login
                String actionName = uri.substring(uri.lastIndexOf("/")+1, uri.indexOf(".scaction"));

                ParseXml parseXml = new ParseXml();
                //得到interceptorXmlBean
                InterceptorXmlBean interceptorXmlBean = parseXml.getInterceptor("logWriter");
                ActionXmlBean actionXmlBean = parseXml.getAction(actionName);
                String name = null;
                String type = null;
                String s_time = null;
                String e_time = null;
                if (interceptorXmlBean!=null){
                    String interceptorClassName = interceptorXmlBean.getClassName();
                    String interceptorMethod = interceptorXmlBean.getMethod();
                    //反射：创建对象，调用方法，获取方法返回的标记
                    Class<?> clazz = Class.forName(interceptorClassName);
                    Object object = clazz.newInstance();
                    Method m = clazz.getDeclaredMethod(interceptorMethod,String.class,String.class,String.class,String.class,String.class);
                    //调用方法
                    name = actionName;
                    Date date = new Date();
                    DateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                    s_time = format.format(date);
                    //String e_time,String result
                    m.invoke(object,name,"",s_time,"","");
                }
                Object object = method.invoke(loginAction,args);
                if (interceptorXmlBean!=null){
                    String interceptorClassName = interceptorXmlBean.getClassName();
                    String interceptorMethod = interceptorXmlBean.getMethod();
                    //反射：创建对象，调用方法，获取方法返回的标记
                    Class<?> clazz = Class.forName(interceptorClassName);
                    Object object2 = clazz.newInstance();
                    Method m = clazz.getDeclaredMethod(interceptorMethod,String.class,String.class,String.class,String.class,String.class);
                    //调用方法
                    name = actionName;
                    type = actionXmlBean.getResults().get(object).getType();
                    Date date = new Date();
                    DateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                    e_time = format.format(date);
                    m.invoke(object2,name,type,s_time,e_time,object);
                }
                return object;
            }
        });
        return loginActionProxy;
    }
}
```

### 3.4 配置文件(controller.xml)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<action-controller>
    <interceptor>
        <name>logWriter</name>
        <class>
            <name>com.qixingjun.interceptor.LogWriter</name>
            <method>log</method>
        </class>
    </interceptor>

    <action>
        <name>login</name>
        <class>
            <name>com.qixingjun.action.impl.LoginActionImpl</name>
            <method>login</method>
        </class>
        <interceptor-ref>
            <name>logWriter</name>
        </interceptor-ref>
        <result>
            <name>fail</name>
            <type>forward</type>
            <value>jsp/login_fail.jsp</value>
        </result>
        <result>
            <name>success</name>
            <type>forward</type>
            <value>jsp/login_success.jsp</value>
        </result>
    </action>
</action-controller>
```

　　至此，整个开发过程基本完毕。

## 4 效果图

　　数据库中存着的用户的用户名是：admin,密码是：admin。效果也是和之前一样。
![登录界面](http://i.imgur.com/ji1svnO.png)
![登陆成功](http://i.imgur.com/6r7xHao.png)
![登录界面，错误的用户名密码](http://i.imgur.com/NSp1xFl.png)
![登录失败](http://i.imgur.com/CyaA6F1.png)

　　然后可以看到，log.xml文件中的日志记录如下图：
![登录成功](http://i.imgur.com/eDTosQ7.png)
![登录失败](http://i.imgur.com/Y9B0vjv.png)



