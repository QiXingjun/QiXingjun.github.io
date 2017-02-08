---
layout: post
title: IDEA中Maven依赖不可以自动补全
categories: Maven
description: IDEA中Maven依赖不可以自动补全
keywords: IDEA，Maven
---

最近在idea上使用maven插件时，发现在pom.xml编写项目依赖的jar包时，已经下载到本地的jar，无法自动补全，需要手动写出来。非常影响效率。

对于这个问题，可以在maven的setting中手动更新本地仓库的jar索引来解决。操作如下
打开设置界面，选中本地的仓库，点击右上角的update，更新maven仓库索引。
这样对于已经下载到本地的jar都可以自动进行补全了。

![problem-about-maven-in-idea](http://i.imgur.com/1KnP6jP.png)