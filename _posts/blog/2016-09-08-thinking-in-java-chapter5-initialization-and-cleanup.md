---
layout: post
title: 初始化和清理
categories: JAVA编程思想（第四版）
description: JAVA编程思想（第四版）读书笔记。
keywords: Java 
---

## 1. 用构造器确保初始化

在Java中，通过提供构造器，类的设计者可确保每个对象都会得到初始化。构造器采用和类相同的名字，所以“每个方法首字母小写”的编码风格并不适用于
构造器。构造器是一种特殊类型的方法，它没有返回值。

`
  class Rock{
    //这是一个默认构造器
    Rock(){
      System.out.print("Rock");
    }
    //这是一个含参构造器
    Rock(int i){
      System.out.print("Rock"+i+" ");
    }
  }
`
