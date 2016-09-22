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

可能这几个概念不是那么好理解，各位可以先看粗略的有个了解，不必深究，看完下面的文章，再回头看看这几个概念就好理解了。

## 4. Git常用命令

有些命令并不是Git所特有的，但是在Git中要经常用的到的，都列了出来，各位可以大略的过一遍，后面的文章会一一进行详细的说明。

   `mkdir XXX`:创建一个空目录，XXX指目录名。
   
   `pwd`:显示当前目录的路径。
   
   `git init`:把当前的目录变成可以管理的git仓库，生成隐藏.git文件。
   
   `git add XXX`:把XXX文件添加到暂存区去。
   
   `git commit –m "XXX"`:提交文件到版本库，–m 后面的XXX是注释。
   
   `git status`:查看仓库状态

   `git diff XXX`:查看XXX文件修改了那些内容

   `git log`:查看历史记录

   `git reset –hard HEAD^` 或者 `git reset –hard HEAD~`:回退到上一个版本，如果想要退回到某个版本（使用`git log`查询），可以使用命令`git reset -hard HEAD^100`（退回到100个版本）。

   `cat XXX`:查看XXX文件的内容。

   `git reflog`:查看历史记录的版本号id。

   `git checkout -- XXX`:把XXX文件在工作区的修改全部撤销。

   `git rm XXX`:删除XXX文件。

   `git remote add origin https://github.com/QiXingjun/ECollaboration`:关联一个远程库。

   `git push –u origin master`:把当前master分支推送到远程库(第一次要用 -u 以后不需要)。

   `git clone https://github.com/QiXingjun/ECollaboration`:从远程库中克隆项目。

   `git checkout –b dev`:创建dev分支，并切换到dev分支上。

   `git branch`:查看当前所有的分支。

   `git checkout master`:切换回master分支。

   `git merge dev`:在当前的分支上合并dev分支。

   `git branch –d dev`:删除dev分支。

   `git branch name`:创建分支。

   `git stash`:把当前的工作隐藏起来，等以后恢复现场后继续工作。

   `git stash list`:查看所有被隐藏的文件列表。

   `git stash apply`:恢复被隐藏的文件，但是内容不删除。

   `git stash drop`:删除文件

   `git stash pop`:恢复文件的同时也删除文件。

   `git remote`:查看远程库的信息。

   `git remote –v`：查看远程库的详细信息。

   `git push origin master`:Git会把master分支推送到远程库对应的远程分支上。

## 5. Git创建版本库

### 5.1 初始化Git仓库

#### 5.1.1 使用当前目录初始化

`git init`

#### 5.1.2 使用指定目录初始化

`git init XXX`，其中XXX代表你要创建版本库的目录。

### 5.2 添加文件到Git仓库

`git add XXX`，这一步就是将XXX添加到暂存区，其中XXX代表你要往Git仓库中添加的文件。

`git commit -m "XXXX"`，这一步就是将暂存区中的修改提交到仓库，其中XXX代表本次提交的说明信息。

经过以上两部，文件就被添加进了Git仓库中了。还有一点需要说明的是可以进行多次`add`，然后一并进行`commit`。

### 5.3 查看仓库状态

使用`git status`用于查看你上次提交之后仓库当前的状态。例如：

```
$ git status
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

上面的示例说明上次提交之后本地仓库中新添加了一个名为123.txt的文件,想要进行提交的话，需要现进行add，commit。如果在123.txt中添加一些文字。再次运行`git status`,如下例所示：

```
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   123.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

这段文字说明了123.txt被修改了。

### 5.4 查看文件修改了哪些东西

通过5.3中知道了123.txt被修改了，但是具体修改了什么呢？可以通过`git diff 123.txt`查看。

```
$ git diff 123.txt
diff --git a/123.txt b/123.txt
index e69de29..97b5955 100644
--- a/123.txt
+++ b/123.txt
@@ -0,0 +1 @@
+12345678
warning: LF will be replaced by CRLF in 123.txt.
The file will have its original line endings in your working directory.
```

通过上面的文字可以看出，123.txt中添加了12345678，然后就可以进行add和commit操作了。

## 6. 版本回退：

使用`git log`命令可以查看历史记录，比如：

```
$ git log
Author: QiXingjun <18862141550@163.com>
Date:   Thu Sep 22 22:46:40 2016 +0800

    add nihao ,shijie ! in the file of 123.txt

commit d2619b5da9ff710bf6bcfcad3c67932908484e26
Author: QiXingjun <18862141550@163.com>
Date:   Thu Sep 22 22:25:33 2016 +0800

    add hello world! in the file of 123.txt

commit 02c61c86c168b555c211c034c006a164a74d019f
Author: QiXingjun <18862141550@163.com>
Date:   Thu Sep 22 22:24:19 2016 +0800

    add the file of 123.txt
```

可以看出，最近的一次更新是：`add nihao ,shijie ! in the file of 123.txt`,前一次更新是：`add nihao ,shijie! in the file of 123.txt`,前一次更新是：`add the file of 123.txt`。现在如果想使用版本回退操作，想把当前的版本回退到上一个版本，要使用什么命令呢？可以使用如下2种命令：第一种是：`git reset  --hard HEAD^`。 那么如果要回退到上上个版本，只需把`HEAD^` 改成 `HEAD^^` 以此类推。。。那如果要回退到前100个版本的话，使用上面的方法肯定不方便，我们可以使用下面的简便命令操作：`git reset --hard HEAD~100` 即可。未回退之前的123.txt内容如下：
```
hello world!

nihao ,shiijie !
```
想退回到上一个版本，使用命令`git reset --hard HEAD^`，例如：

````
$ git reset --hard HEAD^
HEAD is now at d2619b5 add hello world! in the file of 123.txt
```

再次查看123.txt中的内容，如下：

```
hello world!
```

再次使用`git log`

```
$ git log
commit d2619b5da9ff710bf6bcfcad3c67932908484e26
Author: QiXingjun <18862141550@163.com>
Date:   Thu Sep 22 22:25:33 2016 +0800

    add hello world! in the file of 123.txt

commit 02c61c86c168b555c211c034c006a164a74d019f
Author: QiXingjun <18862141550@163.com>
Date:   Thu Sep 22 22:24:19 2016 +0800

    add the file of 123.txt
```

发现已经没有了`add nihao ,shijie ! in the file of 123.txt`。



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










