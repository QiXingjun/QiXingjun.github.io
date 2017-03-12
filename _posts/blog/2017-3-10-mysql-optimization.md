---
layout: post
title: MySQL数据库优化
categories: MySQL 
description: MySQL数据库优化
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

### 3.4 使用explain查询SQL的执行计划

![explain命令](http://i.imgur.com/Z0WYMR6.png)

参数说明：

* **table** ：显示这一行的数据是关于哪张表的
* **type** ：这一列很重要，显示连接使用了哪种类型。从最好到最差的连接类型为：const，eq_reg，ref，range，index和ALL。
* **possible_keys** :显示可能应用在这张表中的索引。如果为空，说明没有可能的索引。
* **key** ：显示实际使用的索引。如果为空，说明没有使用索引。
* **key_len** ：使用的索引的长度。在不损失精度的条件下，长度越短越好。
* **ref** ：显示索引的哪一列被使用了，如果可能的话，是一个常数。
* **rows** ：mysql认为必须检查的用来返回请求数据的行数。
* **extra** ：扩展列，有两个值，看到这两个值，说明查询需要优化了：
1. Using fileSort：MySQL需要额外的步骤来发现如何对返回的行排序，它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行。
2. Using temporary：MySQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行order by上，而不是group by上。

### 3.5 Count()和Max()的优化

如果要求某个字段的Max(),例如：`select Max(payment_date) from payment;`,如果此时payment_date上没有建立索引，执行上面那条语句的效率是很低的，如果建立了索引，查询效率就会变得非常高，因为索引是递增的。

Count(\*)与Count(某个字段名)的结果有时候是不一样的，那是因为Count(\*)会把NULL也算进去，而Count(某个字段名)不会算进去。

### 3.6 子查询的优化

通常情况下，需要将子查询优化为join查询，但是在优化时需要注意，关联键是否有一对多的关系，如果有的话，需要注意重复数据，使用`distinct`关键字进行去重。

## 4 索引优化

### 4.1 如何选择合适的列建立索引

* 在where从句，group by从句，order by从句，on从句中出现的列
* 索引字段越小越好
* 离散度大的列放到联合索引的前面，比如：如果a的离散度比b要大，建立联合索引的时候应该是：`index(a,b)`，而不是`index(b,a)`;

索引的引入会加快查询到效率，但是会降低写入的效率。

### 4.2 重复索引以及冗余索引

重复索引是指：相同的列以相同的顺序建立的同类型的索引，例如，下面例子中的primary key和id列上的索引就是重复索引。

```
create table test(
	id int not null primary key,
	name varchar(20) not null,
	unique(id)
)engine=innodb
```

冗余索引是指： 多个索引的前缀列相同，或者是在联合索引中包含了主键的索引，下面例子中，key(name,id)就是一个冗余索引。
```
create table test(
	id int not null primary key,
	name varchar(20) not null,
	key(name,id)
)engine=innodb
```

由于innodb的特性，每个索引的后面会自动加上主键，所以会有冗余。

重复及冗余索引的查找工具：**pt-duplicate-key-checker**，使用方法如下：
```
pt-duplicate-key-checker -u 用户名 -p 密码 -h ip
```

## 5 数据库结构优化

### 5.1 选择合适的数据类型

合适的含义：

* 选择可以存下你的数据的最小的数据类型
* 使用简单的数据类型。int要比varchar类型在mysql处理上简单。
* 尽可能的使用not null定义字段
* 尽量少的使用text类型，非用不可时，考虑分表。

### 5.2 数据库表的反范式化的设计

反范式化指的是：为了查询效率的考虑，把原本符合第三范式的表适当的增加冗余，以达到优化查询效率的目的，反范式化是一种空间换时间的操作。因为，范式化导致分的表过多，查询的时候会进行很多表的关联查询，所以会影响效率。

### 5.3 数据库表的垂直拆分

所谓表的垂直拆分，就是把原来一个有很多列的表拆分成多个表，这就解决了表的宽度问题。通常垂直拆分可以按一下原则进行：
* 把不常用的字段单独存到一个表中
* 把大字段单独存到一个表中
* 把经常一起使用的字段放到一个表中

### 5.4 数据库表的水平拆分

表的水平拆分是为了解决表的数据量过大的问题，比如一张表中有上亿条的数据，水平拆分的表每一个表的结构都是完整一致的。





