---
layout: post
title: 关于IDEA报could not autowire的错误
categories: JavaWeb
description: 关于IDEA报could not autowire的错误
keywords: Java，IDEA
---

　　问题：在使用spring注解方式的时候 Intellij IDEA报could not autowire，但是之前在eclipse却没有任何的问题，在stackOverFlow上找到一个答案。

　　解决方法：删除项目的iml文件，然后在pom.xml文件上通过右键-->maven-->reimport的方式reimport项目的pom.xml 文件，重建后错误提示消失，问题解决。
