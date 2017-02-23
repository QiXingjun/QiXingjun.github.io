---
layout: post
title: 深入理解Java虚拟机（四）——hotspot垃圾收集器
categories: 深入理解JVM
description: 深入理解Java虚拟机（四）——hotspot垃圾收集器
keywords: Java虚拟机，JVM
---

本文所讲的是sun hotspot虚拟机实现，主要讲解Serial，ParNew，Parallel Scavenge,Serial Old,CMS(Concurrent Mark Sweep),Parallel Old,G1（garbage first）垃圾收集器。

先看Java堆内存结构，适用于非G1收集器外的垃圾收集器：
![](http://i.imgur.com/wMqvxVo.png)

首先根据java对象的生存周期长短把java堆内存分成老年代和新生代，新生代大小可以通过参数 **-Xmn10M**来控制。

然后新生代又被分成3块，一个Eden区，两个大小相等的survivor区，Eden区和Survivor区的大小可以通过参数 **-XX:SurvivorRatio=8**来进行控制。
![](http://i.imgur.com/rTTTk89.jpg)

上面有7中收集器，分为两块，上面为新生代收集器，下面是老年代收集器。如果两个收集器之间存在连线，就说明它们可以搭配使用

在垃圾收集领域有个很有意思的语句“Stop the world”，原因是当执行垃圾回收时需要停止所有的用户线程。

## 1、Serial收集器

 新生代收集器。

 采用复制算法。

 顾名思义它是个单线程的收集器。Client模式下虚拟机新生代默认收集器。

## 2、ParNew收集器

 年轻代收集器。

 采用复制算法。

 其实就是Serial收集器的多线程版本。

 Server模式下虚拟机新生代默认收集器。

 ParNew收集器也是使用 **-XX:+UseConcMarkSwepGC**选项后默认的新生代收集器，也可以使用**-XX:+UseParNewGC**选项来强制指定它。

 它默认开启的收集线程与CPU的数量相同，可以通过**-XX:ParallelGCThreads**参数来限制垃圾收集线程数。

## 3、Parallel Scavenge收集器

 年轻代收集器。

 采用复制算法。

 并行收集器。

 关注点是达到一个可控制的吞吐量（吞吐量优先），而非其他收集器的尽可能缩短垃圾收集时用户线程的等待时间。

 所谓吞吐量就是CPU运行用户代码的时间和总耗时的比值，即吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间）。

> 参数 -XX:MaxGCPauseMillis参数用于控制最大垃圾收集停顿时间。
> 
> 参数 -XX:GCTimeRatio直接设置吞吐量大小（垃圾收集时间比率，吞吐量的倒数）。

## 4、Serial Old收集器

 老年代收集器。

 采用标记-整理算法。

 单线程，类似于Serial收集器。

 Client模式下虚拟机默认的老年代收集器。

## 5、Parallel Old收集器

 老年代收集器。

 采用标记-整理算法。

 多线程，类似于Parallel Scavenge收集器。

 与Parallel Scavenge收集器搭配使用。

## 6、CMS收集器

 老年代收集器。

 采用标记-清除算法。

 获取最短回收停顿时间为目的。

 整个过程分为4个步骤:

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

 其中，初始标记和重新标记这两步骤仍然需要“Stop the world”。

 默认启用的回收线程数是（CPU数量+3）/4。

> 参数 -XX:CMSInitiatingOccupancyFraction的值来触发垃圾收集。

## 7、G1(garbage first)收集器

 当今收集器技术的发展最前沿技术。
 从jdk1.7发布。
 