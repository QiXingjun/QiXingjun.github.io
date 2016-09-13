---
layout: post
title: 访问权限控制
categories: JAVA编程思想(第四版)
description: JAVA编程思想(第四版)读书笔记。
keywords: Java 
---

## 1. 包：库单元

当编写一个Java源代码文件时，次文件常常被称为编译单元。在编译单元中只能有一个public类，否则编译器不接受。并且该类的名称必须与文件的名称相同。
注意：Java包的命名规则是全部使用小写字母。package语句必须是文件中的第一行非注释程序代码。

## 2. Java访问权限修饰词

|   作用域   | 当前类   |   同一package | 子孙类 | 其他package |
|:----------:|:--------:|:-------------:|:------:|:-----------:|
|　public    |     √    |         √     |     √  |      √      |
|　protected |     √    |         √     |     √  |      ×　　  |
|　friendly  |     √    |         √     |     ×  |      ×      |
|　private   |     √    |         ×     |     ×  |      ×      |

注意：不写时默认为friendly

