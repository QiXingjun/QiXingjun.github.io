---
layout: post
title: JavaWeb实现简单的登录验证(controller采用配置文件)
categories: JavaWeb
description: JavaWeb实现简单的登录验证(controller采用配置文件)
keywords: Java，Servlet，JSP，controller，configuration
---

　　将[JavaWeb实现简单的登录验证](http://qixingjun.tech/2016/11/26/simple-controller/)这篇文章中的示例进行修改。将其中的控制器部分改为使用配置文件来完成。还有一个小的修改，将登录的验证方式，从以用户名查询密码然后进行比较变为了通过用户名密码查询user，如果查出来就返回true，否则返回false。

***

　　附，源码地址：[simplecontroller](https://github.com/QiXingjun/simplecontroller)

## 1 开发环境

- IDE：Intellij IDEA
- JDK：1.7
- Tomcat：7

## 2 项目结构
![程序结构](http://i.imgur.com/lEJEKIi.png)

## 3 详细开发

　　将[JavaWeb实现简单的登录验证](http://qixingjun.tech/2016/11/26/simple-controller/)这篇文章中的示例进行修改，将其中的控制器部分改为使用配置文件来完成。还有一个小的修改，将登录的验证方式，从以用户名查询密码然后进行比较变为了通过用户名密码查询user，如果查出来就返回true，否则返回false。

　　具体来讲，dao层和service层的变化不大，只是登录验证方式进行简单的更改。将之前的Controller更改为全局的控制器，并采用配置文件的方式来完成。这个ActionController解析url，将不同的事情交给不同的Action去处理。比如，之前实现的是登录，这里解析出login请求后，就会交给LoginAction进行处理。LoginAction调用login()方法之后返回一个成功或者失败的flag，Controller根据返回的flag来进行结果的跳转。由于是采用配置文件的方式，还要建立一个解析配置文件的工具类ParseXml，根据配置文件的结构，还需要一个封装action节点的ActionXmlBean和一个封装视图的ResultXmlBean。

### 3.1 简单java对象(Bean）

#### 3.1.1 UserBean 

```java
package com.qixingjun.pojo;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 用户实体类
 */
public class UserBean {
    private String userName;
    private String passWord;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassWord() {
        return passWord;
    }

    public void setPassWord(String passWord) {
        this.passWord = passWord;
    }
}
```

#### 3.1.2 ActionXmlBean

```java
package com.qixingjun.pojo;

import java.util.HashMap;
import java.util.Map;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/12/2
 * @Description 封装action节点
 */
public class ActionXmlBean {
    // 请求路径名称
    private String name;
    // 处理aciton类的全名
     private String className;
    // 处理方法
    private String method;
    // 结果视图集合
     private Map<String,ResultXmlBean> results;

    public Map<String, ResultXmlBean> getResults() {
        return results;
    }

    public void setResults(Map<String, ResultXmlBean> results) {
        this.results = results;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }

    public String getMethod() {
        return method;
    }

    public void setMethod(String method) {
        this.method = method;
    }
}
```

#### 3.1.3 ResultXmlBean

```java
package com.qixingjun.pojo;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/12/2
 * @Description 封装结果视图
 */
public class ResultXmlBean {
    // 跳转的结果标记
    private String name;
    // 跳转类型，默认为转发； "redirect"为重定向
    private String type;
    // 跳转的页面
    private String page;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getPage() {
        return page;
    }

    public void setPage(String page) {
        this.page = page;
    }
}
```

### 3.2 数据访问层(dao、dao.impl)

#### 3.2.1 数据访问层(dao)--IUserDao

```java
package com.qixingjun.dao;

import com.qixingjun.pojo.UserBean;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 根据用户名,密码来查找用户，找到返回true，否则返回false
 */
public interface IUserDao {
    Boolean findUser(UserBean userBean);
}
```

#### 3.2.2 数据访问层(dao.impl)--UserDaoImpl

```java
package com.qixingjun.dao.impl;

import com.qixingjun.dao.IUserDao;
import com.qixingjun.pojo.UserBean;

import java.sql.*;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description IUserDao接口的实现类，数据的持久化
 */
public class UserDaoImpl implements IUserDao {
    @Override
    public Boolean findUser(UserBean userBean) {
        Connection con =null;
        PreparedStatement pstmt =null;
        ResultSet rs = null;
        try {
            String driver ="com.mysql.jdbc.Driver";
            String url ="jdbc:mysql://localhost:3306/simplecontroller";
            String user ="root";
            String password ="root";
            Class.forName(driver);
            con = DriverManager.getConnection(url, user, password);
            String sql = "select * from t_user where username=? AND password=?";
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1,userBean.getUserName());
            pstmt.setString(2,userBean.getPassWord());
            rs = pstmt.executeQuery();
            if(rs==null){
                return false;
            }
            if(rs.next()){
                return true;
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            try {
                if(pstmt!=null)pstmt.close();
                if(con!=null)con.close();
            }
            catch (SQLException e) {
            }
        }
        return false;
    }
}
```

### 3.2 业务处理层(service、service.impl)

#### 3.2.1 业务处理层(service)

```java
package com.qixingjun.service;

import com.qixingjun.pojo.UserBean;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 登录接口
 */
public interface ILoginService {
    Boolean findUser(UserBean userBean);
}
```

#### 3.2.2 业务处理层(service.impl)

```java
package com.qixingjun.service.impl;

import com.qixingjun.dao.impl.UserDaoImpl;
import com.qixingjun.pojo.UserBean;
import com.qixingjun.service.ILoginService;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.1.0
 * @Date 2016/11/26
 * @Description  登录接口的实现
 */
public class LoginServiceImpl implements ILoginService {
    @Override
    public Boolean findUser(UserBean userBean) {
        UserDaoImpl userDaoImpl = new UserDaoImpl();
        return userDaoImpl.findUser(userBean);
    }
}
```

### 3.3 控制层(ActionController)

　　这个类的功能，就是统筹全局请求的分配，针对比如像http://localhost:8080/login.scaction的请求，ActionController解析出login，把这个请求交给LoginAction类处理。　　

1. 收到http:localhost:8080/login.scaction的请求，解析出login。

2. 转发给LoginAction处理，LoginAction调用login()方法，这个方法执行完成之后返回一个returnFlag标志，success表示登录成功，fail表示登录失败。

3. ActionController根据返回的returnFlag判断跳转的页面，先查询配置文件（controller.xml），获取对应的跳转page，以及跳转的方式（转发或者重定向）。

4. 对于一个项目而言，要处理的请求有很多，也就说有很多的XxxAction那么对应的controller.xml文件就会很大，我们不会每次都来查找解析这个xml文件，而且对于需要同时管理很多信息时，常见的做法是把这若干信息封装到一些bean中。每个`<action>`封装到对应的ActionXmlBean类中，每个`<result>`封装到ResultXmlBean类中，因为每个`<action>`中包含若干的`<result>`所以每个ActionXmlBean总包含若干的`<Result>`。 

```java
package com.qixingjun.controller;

import com.qixingjun.action.LoginAction;
import com.qixingjun.pojo.ActionXmlBean;
import com.qixingjun.pojo.ResultXmlBean;
import com.qixingjun.util.ParseXml;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Method;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.1.0
 * @Date 2016/11/26
 * @Description  全局的控制器，拦截所有满足以"*.scaction"为后缀的请求
 */

@WebServlet(name = "ActionController",urlPatterns = "*.scaction")
public class ActionController extends HttpServlet {
    private ParseXml parseXml;

    //启动时候执行执行一次
    @Override
    public void init() throws ServletException {
        parseXml = new ParseXml();
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        try {
            //1.获取请求的uri,得到请求路径名称
            String uri = request.getRequestURI();
            //得到login
            String actionName = uri.substring(uri.lastIndexOf("/")+1, uri.indexOf(".scaction"));

            //2.根据路径名称，读取配置文件，得到类的全名
            ActionXmlBean actionXmlBean = parseXml.getAction(actionName);
            String className = actionXmlBean.getClassName();
            //当前请求的处理方法
            String method = actionXmlBean.getMethod();

            //3.反射：创建对象，调用方法，获取方法返回的标记
            Class<?> clazz = Class.forName(className);
            Object object = clazz.newInstance();
            Method m = clazz.getDeclaredMethod(method,HttpServletRequest.class,HttpServletResponse.class);
            //调用方法的的标记
            String returnFlag = (String) m.invoke(object, request,response);

            //4.拿到标记，读取配置文件得到标记对应的页面，跳转类型
            ResultXmlBean result = actionXmlBean.getResults().get(returnFlag);
            //类型
            String type = result.getType();
            //页面
            String page = result.getPage();

            if ("redirect".equals(type)) {
                response.sendRedirect(request.getContextPath()+page);
            }else {
                request.getRequestDispatcher(page).forward(request, response);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {
        doGet(req, res);
    }
}
```

### 3.4 请求处理层(LoginAction)

```java
package com.qixingjun.action;

import com.qixingjun.pojo.UserBean;
import com.qixingjun.service.impl.LoginServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.1.0
 * @Date 2016/11/30
 * @Description 负责处理login请求的Action
 */
public class LoginAction{
    LoginServiceImpl loginService = new LoginServiceImpl();
    //登录的处理数据的方法
    public String login(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException {
        UserBean userBean = new UserBean();
        userBean.setUserName(req.getParameter("username"));
        userBean.setPassWord(req.getParameter("password"));
        if (loginService.findUser(userBean)){
            return "success";
        }else{
            return "fail";
        }
    }
}
```

### 3.5 XML文件的解析类(ParseXml)

```java
package com.qixingjun.util;

import com.qixingjun.pojo.ActionXmlBean;
import com.qixingjun.pojo.ResultXmlBean;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import java.io.InputStream;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/12/2
 * @Description 解析XML文件
 */
public class ParseXml {
    private Map<String,ActionXmlBean> allActions;

    public ParseXml(){
        allActions = new HashMap<String, ActionXmlBean>();
        this.init();
    }

    public ActionXmlBean getAction(String ActionName){
        if (ActionName==null) {
            throw new RuntimeException("传入参数有误，请查看配置文件配置路径");
        }
        ActionXmlBean actionXmlBean = allActions.get(ActionName);
        if (actionXmlBean==null) {
            throw new RuntimeException("在配置文件中找不到路径");
        }
        return actionXmlBean;
    }

    // 初始化allActions集合
    private void init(){
        //读取配置文件
        try {
            // 1.得到解析器
            SAXReader reader = new SAXReader();

            // 2. 加载文件
            InputStream inputStream = this.getClass().getResourceAsStream("/controller.xml");
            Document doc = reader.read(inputStream);

            //3.获取根
            Element root = doc.getRootElement();

            //4.得到所有的action子节点
            List<Element> listActions = root.elements("action");

            //5.遍历，封装
            for (Element ele_action : listActions) {
                ActionXmlBean actionXmlBean = new ActionXmlBean();
                actionXmlBean.setName(ele_action.element("name").getText());
                actionXmlBean.setClassName(ele_action.element("class").element("name").getText());
                actionXmlBean.setMethod(ele_action.element("class").element("method").getText());

                //封装当前aciton节点下的results
                HashMap<String, ResultXmlBean> results = new HashMap<String, ResultXmlBean>();

                //得到当前action节点下所有的result子节点
                Iterator<Element> iterator = ele_action.elementIterator("result");

                while(iterator.hasNext()){
                    // 当前迭代的每一个元素都是 <result...>
                    Element ele_result = iterator.next();
                    ResultXmlBean resultXmlBean = new ResultXmlBean();
                    resultXmlBean.setName(ele_result.element("name").getText());
                    resultXmlBean.setType(ele_result.element("type").getText());
                    resultXmlBean.setPage(ele_result.element("value").getText());
                    results.put(resultXmlBean.getName(), resultXmlBean);
                }
                actionXmlBean.setResults(results);
                allActions.put(actionXmlBean.getName(), actionXmlBean);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 3.6 配置文件(controller.xml)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<action-controller>
    <action>
        <name>login</name>
        <class>
            <name>com.qixingjun.action.LoginAction</name>
            <method>login</method>
        </class>
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

### 3.7 登录页
　　登录页基本没有变化，除了form的action部分修改为了`action="/login.scaction"`,之所以这么修改是因为在ActionController的WebServlet部分定义了`@WebServlet(name = "ActionController",urlPatterns = "*.scaction")`。


　　至此，整个开发过程基本完毕。还有一点需要注意的是，由于要解析xml文件，采用的是dom4j。因此要引入dom4j的jar包。
[点击这里获取dom4j的jar包](http://pan.baidu.com/s/1pL6hSiJ)

## 4 效果图

　　数据库中存着的用户的用户名是：admin,密码是：admin。效果也是和之前一样。
![登录界面](http://i.imgur.com/ji1svnO.png)
![登陆成功](http://i.imgur.com/6r7xHao.png)
![登录界面，错误的用户名密码](http://i.imgur.com/NSp1xFl.png)
![登录失败](http://i.imgur.com/CyaA6F1.png)

