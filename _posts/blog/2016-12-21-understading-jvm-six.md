---
layout: post
title: 深入理解Java虚拟机（六）——对象引用强度
categories: 深入理解JVM
description: 深入理解Java虚拟机（六）——对象引用强度
keywords: Java虚拟机，JVM
---

无论是通过计数算法判断对象的引用数量，还是通过根搜索算法判断对象引用链是否可达，判定对象是否存活都与“引用”相关。

引用主要分为 ：**强引用(Strong Reference)、软引用(Soft Reference)、弱引用(Weak Reference)、虚引用(PhantomReference)** 四种，引用的强度依次骤减。

## 1.强引用

就是指在代码之中普遍存在的，类似：`“Object objectRef = new Obejct”`，这种引用，只要强引用还存在，永远不会被GC清理。

## 2.软引用

用来描述一些还有用，但并非必须存在的对象，当Jvm内存不足时（内存溢出之前）会被回收，如果执行GC后，还是没有足够的空间，才会抛出内存溢出异常。

通过SoftReference类来实现软引用，SoftReference很适合用于实现缓存。另外，当GC认为扫描的SoftReference不经常使用时，可会进行回收。

使用方法：

```java
User user = new User();  
SoftReference<Object> softReference  = new SoftReference<Object>(user);  
softReference.get();
```
  
## 3.弱引用

弱引用也是用来描述一些还有用，但并非必须存在的对象，它的强度会比软引用弱些，被弱引用关联的对象，只能生存到下一次GC前，当GC工作时，无论内存是否足够，都会回收掉弱引用关联的对象。

JDK通过WeakReference类来实现。

当获取时，可通过weakReference.get方法获取，可能返回null。

可传入一个ReferenceQueue对象到WeakReference构造，当引用对象被表示为可回收时，isEnqueued返回true

```java
User user = new User();  
WeakReference<User> weakReference = new WeakReference<User>(user);  
weakReference.get();  
  
ReferenceQueue<User> referenceQueue = new ReferenceQueue<User>();  
WeakReference<User> weakReference2 = new WeakReference<User>(user, referenceQueue);  
//当引用对象被标识为可回收时  返回true,  即当user对象标识为可回收时，返回true  
weakReference.isEnqueued();
```
  
## 4.虚引用

虚引用称为“幻影引用”，它是最弱的一种引用关系，一个对象是否有虚引用的存在，完全不会对生存时间构成影响。

为一个对象设置虚引用关联的唯一目的就是希望能在这个对象被GC回收时收到一个系统通知。

通过PhantomReference类实现。

值得注意的是：phantomReference.get方法永远返回null, 当user从内存中删除时，调用isEnqueued会返回true

```java
User user = new User();  
ReferenceQueue<User> referenceQueue = new ReferenceQueue<User>();  
PhantomReference<User>  phantomReference = new PhantomReference<User>(user, referenceQueue);  
//即当user对象标识为可回收时，返回true  
System.out.println(phantomReference.isEnqueued());  
//永远返回null  
System.out.println(phantomReference.get());
```