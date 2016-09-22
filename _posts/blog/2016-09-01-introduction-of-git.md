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

## 3. Git中的基本概念

### 3.1 工作区（Working Directory）

所谓的工作区，就是在本地电脑中所能看到的目录，但是不包含隐藏的`.git`目录。

### 3.2 版本库（Repository）

在已经初始化的工作区中有一个隐藏目录`.git`，这个就是Git的版本库。这个目录里面的所有文件都可以被Git管理起来，每个文件的修改，删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻还可以将文件”还原”。在版本库中有两个比较重要的东西：一个是暂存区，另一个是分支master以及指向master的指针HEAD。

### 3.2 暂存区（stage/index）

Git之父Linus当初设计暂存区的初衷是由于每次在SVN中commit的时候都需要选择需要提交到版本库的文件，发现这个功能太鸡肋了。于是他想如果能够在真正commit之前做任意的修改，这些修改可以先放在暂存区中，如果后悔了不仅可以非常方便撤销，而且不会影响到现有的版本库。如果发现没什么问题，可以一次性将多次`add`操作进行提交。

可能这几个概念不是那么好理解，读者







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

### 4.1 Git clone

使用`git clone`拷贝一个Git仓库到本地，让自己能够查看该项目，或者进行修改。
如果你需要与他人合作一个项目，或者想要复制一个项目，看看代码，你就可以克隆那个项目。 执行命令：

`git clone XXX`，其中XXX代表要克隆的项目的Git库地址。例如，clone一个名为ECollaboration的项目：

```
~$ git clone https://github.com/QiXingjun/ECollaboration.git
Cloning into 'ECollaboration'...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), done.
Checking connectivity... done.
```

### 4.2 Git status

使用`git status`用于查看你上次提交之后仓库当前的状态。例如：

```
~/ECollaboration$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

nothing to commit, working directory clean
```

上面的示例说明上次提交之后没有对本地仓库作任何的修改。而如果如下例所示：

```
~/ECollaboration$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	123.txt

nothing added to commit but untracked files present (use "git add" to track)
```

上面的示例说明上次提交之后本地仓库中新添加了一个名为123.txt的文件,想要进行提交的话，需要现进行add，执行`git add 123.txt`之后的结果为：

```
~/ECollaboration$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   123.txt
```

这段示例说明，已经进行了add操作，现在就可以进行commit操作了，执行`git commit -m "add a file named 123.txt"`之后的结果为：

```
$ git commit -m "add a file named 123.txt"
[master 6b4ad8f] add a file named 123.txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 123.txt
```

这段说明已经将123.txt进行了commit，可以进行push操作了。其中，执行语句中的`-m "XXXXX"`是对于你当前提交的东西进行一个说明。执行`git push`之后你所修改的东西就真正的提交到远程的服务器了。

### 4.2 Git commit










