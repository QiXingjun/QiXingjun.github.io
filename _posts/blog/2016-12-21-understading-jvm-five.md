---
layout: post
title: 深入理解Java虚拟机（五）——对象内存分配与回收
categories: 深入理解JVM
description: 深入理解Java虚拟机（五）——对象内存分配与回收
keywords: Java虚拟机，JVM
---

## 1. 对象优先在Eden上分配

大多数情况下，对象优先在新生代Eden区域中分配。当Eden内存区域没有足够的空间进行分配时，虚拟机将触发一次 Minor GC(新生代GC)。

Minor GC期间虚拟机将Eden区域的对象移动到其中一块Survivor区域。

## 2. 大对象直接进入老年代

所谓大对象是指需要大量连续空间的对象。

虚拟机提供了一个**-XX:PretenureSizeThreshold**参数，令大于这个值的对象直接在老年代中分配。

## 3. 长期存活的对象将进入老年代

虚拟机采用分代收集的思想管理内存，那内存回收时就必须能识别哪些对象该放到新生代，哪些该到老年代中。为了做到这点，虚拟机为每个对象定义了一个对象年龄Age，每经过一次新生代GC后仍然存活，将对象的年龄Age增加1岁，当年龄到一定程度（默认为15）时，将会被晋升到老年代中。对象晋升老年代的年龄限定值，可通过**-XX:MaxTenuringThreshold**来设置。

## 4. Minor GC 和Full GC区别

新生代GC(Minor GC)：指发生在新生代的垃圾收集动作，因为对象大多都具备朝生夕灭特性，所以Minor GC非常频繁，回收速度也比较快。

老年代GC(Major GC / Full GC)：指发生在老年代中的GC，出现Major GC后，经常会伴随至少一次的 Minor GC。

Major GC的速度一般会比Minor GC慢10倍以上。