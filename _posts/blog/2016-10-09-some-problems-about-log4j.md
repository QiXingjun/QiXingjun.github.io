---
layout: post
title: log4j:WARN No appenders could be found for logger
categories: Java
description: log4j:WARN No appenders could be found for logger
keywords: log4j,Java
---

在SSM整合开发的时候，遇到问题如下：

```
log4j:WARN No appenders could be found for logger (org.springframework.web.context.ContextLoader).  
log4j:WARN Please initialize the log4j system properly. 
```

解决方法 : 在WEB-INF/classes/路径下加上文件 `log4j.properties` 其参考内容如下：

```
log4j.rootLogger=DEBUG, Console
#Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
log4j.logger.java.sql.ResultSet=INFO
log4j.logger.org.apache=INFO
log4j.logger.java.sql.Connection=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```
