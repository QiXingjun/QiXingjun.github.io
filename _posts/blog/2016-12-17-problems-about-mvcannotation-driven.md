---
layout: post
title: 通配符的匹配很全面, 但无法找到元素 'mvc:annotation-driven' 的声明
categories: JavaWeb
description: 通配符的匹配很全面, 但无法找到元素 'mvc:annotation-driven' 的声明
keywords: Java，IDEA，MVC
---

　　问题：报错信息：通配符的匹配很全面, 但无法找到元素 'mvc:annotation-driven' 的声明。

　　原因是：虽然在xml文件上方声明了mvc，但没有配置此声明对应的文件信息，正确配置如下：

```java
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd  ">
```
　　意思就是，mvc声明用http://www.springframework.org/schema/mvc/spring-mvc.xsd这个文件来解析
