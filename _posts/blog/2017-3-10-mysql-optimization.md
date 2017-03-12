---
layout: post
title: MySQL优化（一） 数据库优化简介及SQL语句优化
categories: MySQL 
description: MySQL优化（一） 数据库优化简介及SQL语句优化
keywords: MySQL 优化 SQL
---

对于大多数的后端程序员来说，Mysql应该是很熟悉的。MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件。

MySQL是一种关系数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。
MySQL所使用的 SQL 语言是用于访问数据库的最常用标准化语言。

但是，大多数的程序员仅仅是使用mysql做一些增删改查的操作，没有或者很少想过做一些数据库方面的优化。下面是我的一些关于mysql优化的学习心得。

## 1 数据库优化的目的

### 1.1 避免出现页面访问错误

* 由于数据库连接timeout，产生5xx错误
* 由于慢查询造成页面无法加载
* 由于阻塞造成数据无法提交

### 1.2 增加数据库的稳定性

* 很多数据库问题都是由于低效的查询引起的

### 1.3 优化用户体验

* 流畅的页面访问速度
* 良好的网站功能体验

## 2 数据库优化的方面

数据库的优化可以从以下几个方面考虑：

1.SQL及索引--> 2.数据库的表结构--> 3.系统的配置--> 4.硬件

从1->4 的优化成本越来越高，且效果越来越差，所以，尽量在1和2两方面进行优化，这样既可以得到很好的效果，又可以节省很大的成本。

## 3 SQL语句优化

### 3.1 sakila的安装

为了更好地演示SQL及索引的优化，mysql提供了sakila数据库，[下载地址](https://dev.mysql.com/doc/sakila/en/sakila-installation.html)。

在Ubuntu下的安装sakila数据库：

1. tar zxf sakila-db.zip #解压

2. mysql -u root -p ##登录

3. 在MYSQL 命令下： SOURCE 下载路径/sakila-schema.sql #建立表结构

4. 在MYSQL 命令下： SOURCE 下载路径/sakila-data.sql #插入数据

### 3.2 mysql慢查询日志

#### 3.2.1 慢查询日志的相关命令

1. 查看mysql是否开启慢查询日志

	`show variables like 'slow_query_log';`

2. 设置没有索引的记录到慢查询日志

	`set global log_queries_not_using_indexes=on;`

3. 查看超过多长时间的sql进行记录到慢查询日志

	`show variables like 'long_query_time';`

4. 开启慢查询日志

	`set global slow_query_log=on;`

5. 查看日志记录位置

	`show variables like 'slow_query_log_file;`

6. 查看日志

	`sudo tail -50 第五步查询出来的记录日志的位置`

需要注意的是：前5条是在登录mysql之后使用，而第6条是在退出mysql之后使用。

#### 3.2.2 慢查询日志的格式

```
sql的执行时间

Time: 170312 13:09:10

执行sql的主机信息

User@Host: root[root] @ localhost []

sql的执行信息

Query_time: 0.000361  Lock_time: 0.000101 Rows_sent: 2 Rows_examined: 2

sql的执行时间

SET timestamp=1489295350;

sql的具体内容

select * from store;
```

#### 3.2.3 慢查询日志的分析工具

* mysqldumpslow

mysqldumpslow的具体使用命令，可以使用`mysqldumpslow -h`查看，例如,使用如下命令查看慢查询日志的前3条：
`sudo mysqldumpslow -t 3 /var/lib/mysql/VM-25-22-ubuntu-slow.log | more`

* pt-query-digest

pt-query-digestde 的查询结果会更加的完善和详细。例如，使用如下命令进行慢查询日志分析：
`sudo pt-query-digest /var/lib/mysql/VM-25-22-ubuntu-slow.log | more`

### 3.3 通过慢查询日志进行问题定位

有问题的SQL的特征：

* **查询次数多，且每次查询占用的时间长的SQL**：通常为pt-query-digest分析的前几个查询
* **IO大的SQL**：注意pt-query-digest分析中的Rows examine项
* **未命中索引的SQL**：注意pt-query-digest分析中的Rows examine和Rows Send的对比

 



