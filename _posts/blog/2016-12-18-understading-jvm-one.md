---
layout: post
title: 深入理解Java虚拟机（一）——Java内存结构划分
categories: 深入理解JVM
description: 深入理解Java虚拟机（一）——Java内存结构划分
keywords: Java虚拟机，JVM
---

　　![Java内存结构化分](http://i.imgur.com/a46rxku.png)

图一：Java内存结构划分

由上图可知，java内存主要分为6部分，分别是：**程序计数器，虚拟机栈，本地方法栈，堆，方法区和直接内存**,下面将逐一详细描述。其中，方法区和堆是线程共享的，而程序计数器，虚拟机栈，本地方法栈是线程私有的。

## 1、程序计数器

线程私有，即每个线程都会有一个，线程之间互不影响，独立存储。

代表着当前线程所执行字节码的行号指示器。

## 2、虚拟机栈

线程私有，它的生命周期和线程相同。

描述的是java方法执行的内存模型：每个方法在执行的同时多会创建一个栈帧用于存储局部变量表、操作数栈、动态链表、方法出口等信息。

每一个方法从调用直至完成的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程。

局部变量表存放了编译期可知的各种基本数据类型和对象引用，所需内存空间在编译期确定。

> -Xoss参数设置本地方法栈大小（对于HotSpot无效）
>
> -Xss参数设置栈容量 例： -Xss128k

## 3、本地方法栈

同虚拟机栈，只不过本地方法栈为虚拟机使用到的native方法服务。

Sun HotSpot虚拟机把本地方法栈和虚拟机栈合二为一。

## 4、java堆

线程共享

主要用于分配对象实例和数组。

> -Xms参数设置最小值
> 
> -Xmx参数设置最大值 例：VM Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError

> 若-Xms=-Xmx,则可避免堆自动扩展。

> -XX:+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出时dump出当前的内存堆转储快照。

## 5、方法区

线程共享

用于存储已被虚拟机加载的类信息、常量、静态变量、即使编译后的代码等数据。

别名:永久代（Permanent Generation）

> -XX:MaxPermSize设置上限
> 
> -XX:PermSize设置最小值 例：VM Args:-XX:PermSize=10M -XX:MaxPermSize=10M

运行时常量池(Runtime Constant Pool)是方法区的一部分。

Class文件中除了有类的版本、字段、方法、接口等信息外，还有一项是常量池（Constant Pool Table）,用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

运行时常量池相对于Class文件常量池的一个重要特征是具备动态性：即除了Class文件中常量池的内容能被放到运行时常量池外，运行期间也可能将新的常量放入池中，比如String类的intern()方法。

## 6、直接内存

直接内存并不是虚拟机运行时数据区的一部分。

在NIO中，引入了一种基于通道和缓冲区的I/O方式，它可以使用native函数直接分配堆外内存，然后通过一个存储在java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。

> -XX:MaxDirectMemorySize设置最大值，默认与java堆最大值一样。
> 
> 例：-XX:MaxDirectMemorySize=10M -Xmx20M
