---
layout: post
title: JavaWeb实现简单的登录验证
categories: JavaWeb
description: JavaWeb实现简单的登录验证
keywords: Java，Servlet，JSP，登录
---

　　使用Java实现简单的用户登录功能，代码是按照【简单java对象(pojo)】→【数据访问层(dao、dao.impl)】→【业务处理层(service、service.impl)】→ 【控制层(controller)】的顺序进行编写的。 前端使用的是BootStrap。 数据库使用的是mysql。

　　附，源码地址：[simplecontroller](https://github.com/QiXingjun/simplecontroller)

## 1 开发环境

- IDE：Intellij IDEA
- JDK：1.7
- Tomcat：7

## 2 项目结构
![simplecontroller的项目结](http://i.imgur.com/xemKL0j.png)

## 3 详细开发

　　因为代码是按照【简单java对象(pojo)】→【数据访问层(dao、dao.impl)】→【业务处理层(service、service.impl)】→ 【控制层(controller)】的顺序进行编写的，所以也按照这个顺序介绍。

### 3.1 简单java对象(pojo）--UserBean

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

### 3.2 数据访问层(dao、dao.impl)

#### 3.2.1 数据访问层(dao)--IUserDao

```java
package com.qixingjun.dao;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 根据用户名来查找用户的密码
 */
public interface IUserDao {
    String findUser(String userName);
}
```

#### 3.2.1 数据访问层(dao.impl)--UserDaoImpl

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
    public String findUser(String userName) {
        String psw = null;
        Connection con =null;
        PreparedStatement pstmt =null;
        ResultSet rs = null;
        try {
            String driver ="com.mysql.jdbc.Driver";   //数据库采用mysql，要加载mysql的驱动
            String url ="jdbc:mysql://localhost:3306/simplecontroller";  //数据库名称为simplecontroller
            String user ="root";   //mysql的用户名
            String password ="root";    //mysql的密码
            Class.forName(driver);
            con = DriverManager.getConnection(url, user, password);
            String sql = "select * from t_user where username=?";   //表名为t_user
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, userName);
            rs = pstmt.executeQuery();
            if(rs==null){
                return null;
            }
            if(rs.next()){
                psw=rs.getString("password");
            }else{
                psw=null;
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
        return psw;
    }
}
```

### 3.2 业务处理层(service、service.impl)

```java
package com.qixingjun.service;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 登录接口
 */
public interface ILoginService  {
    String findUser(String userName);
}
```

### 3.3 业务处理层(service、service.impl)

#### 3.3.1 业务处理层(service)

```java
package com.qixingjun.service;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 登录接口
 */
public interface ILoginService  {
    String findUser(String userName);
}
```

#### 3.3.2 业务处理层(service.impl)

```java
package com.qixingjun.service.impl;

import com.qixingjun.dao.impl.UserDaoImpl;
import com.qixingjun.service.ILoginService;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 登录接口的实现
 */
public class LoginServiceImpl implements ILoginService {
    @Override
    public String findUser(String userName) {
        UserDaoImpl userDaoImpl = new UserDaoImpl();
        return userDaoImpl.findUser(userName);
    }
}
```

### 3.4 控制层(controller)--LoginController

```java
package com.qixingjun.controller;

import com.qixingjun.service.impl.LoginServiceImpl;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @Author XingJun Qi
 * @MyBlog www.qixingjun.tech
 * @Version 1.0.0
 * @Date 2016/11/26
 * @Description 用户登录的控制器
 */
@WebServlet(name = "LoginController",urlPatterns = "/loginController")
public class LoginController extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String userName = request.getParameter("username");
        String passWord = request.getParameter("password");
        LoginServiceImpl loginServiceImpl = new LoginServiceImpl();
        if (passWord.equals(loginServiceImpl.findUser(userName))){
            request.getRequestDispatcher("/jsp/login_success.jsp").forward(request,response);
        }else{
            request.getRequestDispatcher("/jsp/login_fail.jsp").forward(request,response);
        }
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 3.5 登录页

```java
<%--
  Created by IntelliJ IDEA.
  User: XingJun Qi
  Date: 2016/11/26
  Time: 14:01
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<head>
    <title>用户登录</title>

    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">

</head>
<body>
    <div style="width:30%">
        <form action="/loginController" method="post" align="center" class="form-signin">
            <h2 align="center">登  录</h2>
            <input type="text" class="form-control" name="username" placeholder="UserName" required autofocus><br>
            <input type="password" class="form-control" name="password" placeholder="Password" required><br>
            <button class="btn btn-lg btn-primary btn-block" type="submit">登  录</button>
        </form>
    </div>
</body>
</html>
```

　　至此，整个开发过程基本完毕，login_sucess.jsp和login_fail.jsp就不贴出来了，很简单，输出成功或者失败就行。还有一点需要注意的是，由于需要数据库，要引入mysql的驱动jar包。![msyql-driver](http://i.imgur.com/SehUI7K.png) 

[点击这里获取mysql驱动包](http://pan.baidu.com/s/1pLCnhzH)

## 4 效果图

　　数据库中存着的用户的用户名是：admin,密码是：admin。
![登录界面](http://i.imgur.com/ji1svnO.png)
![登陆成功](http://i.imgur.com/6r7xHao.png)
![登录失败](http://i.imgur.com/CyaA6F1.png)

