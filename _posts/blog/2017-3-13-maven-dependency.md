---
layout: post
title: Maven依赖冲突
categories: maven 
description: Maven依赖冲突
keywords: maven 依赖冲突
---

## 1 冲突的原因

1. 依赖是使用Maven坐标来定位的，而Maven坐标主要由GAV（groupId, artifactId, version）构成。如果两个相同的依赖包，如果groupId, artifactId, version不同，那么maven也认为这两个是不同的。
2. 依赖会传递，A依赖了B，B依赖了C，那么A的依赖中就会出现B和C。
3. Maven对同一个groupId, artifactId的冲突仲裁，不是以version越大越保留，而是依赖路径越短越优先，然后进行保留。
4. 依赖的scope会影响依赖的影响范围。

## 2 定位冲突与解决冲突

1. 通过`mvn dependency:tree` 命令可以查看全部的依赖。
2. 如果使用的是Intellij IDEA开发工具的话，可以打开pom.xml，右键->Diagrams->show dependences或者右键->Maven->show dependences来进行查看全部依赖。
3. 在想要删除的冲突jar上右键->exclude即可删除冲突的依赖。

