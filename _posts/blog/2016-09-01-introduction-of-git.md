---
layout: post
title: Git快速上手
categories: Git
description: Git学习笔记。
keywords: Git,版本控制 
---

## 1. Git是什么

如果你是一个Coder，你就应该知道Git。如果你以前不知道，那么现在你一定知道了。Git是一个开源的分布式版本控制系统，很多人也许使用过SVN，
至于SVN和Git的区别，这里不做探讨，毕竟先学会Git的使用，才可以更明白的理解SVN和Git的区别，不是吗？

## 2. Git的安装和配置

### 2.1 Ubuntu下安装Git

Ubuntu下直接使用命令：

`sudo apt-get install git`

就OK了。

### 2.2 Windows下安装Git

Windows平台，直接下载Git的安装包，和安装普通软件一样安装就OK了，在开始菜单找到Git-->Git Bash，就会弹出Git的命令行窗口，然后就可以直接进行Git操作了。

### 2.3 Git配置

配置个人的用户名称和电子邮件地址：

`$ git config --global user.name "QiXingjun"`

`$ git config --global user.email 18862141550@163.com`

## 3. Git创建版本库

版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

### 3.1 初始化Git仓库

#### 3.1.1 使用当前目录初始化

`git init`

#### 3.1.2 使用指定目录初始化

`git init XXX`，其中XXX代表你要创建版本库的目录。

### 3.2 添加文件到Git仓库

`git add XXX`，其中XXX代表你要往Git仓库中添加的文件。

`git commit -m "XXXX"`，其中XXX代表本次提交的说明信息。

经过以上两部，文件就被添加进了Git仓库中了。还有一点需要说明的是可以进行多次`add`，然后一并进行`commit`。

## 4. Git基本操作




