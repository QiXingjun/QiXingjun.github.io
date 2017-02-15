---
layout: post
title: 在Web中查询到的时间和mysql中查询到的不一致
categories: J2EE
description: 在Web中查询到的时间和mysql中查询到的不一致
keywords: J2EE，mysql
---

在Web中查询到的时间和mysql中查询到的不一致，找了好一会，发现原来是因为pattern格式的大小写有误：

![time-in-web-and-in-mysql](http://i.imgur.com/e6IrCR9.png)

将小写的mm改成大写的MM就可以了。