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

发现已经没有了`nihao ,shijie !`，再次使用`git log`

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

发现已经没有了`add nihao ,shijie ! in the file of 123.txt`。但是如果现在又想恢复`nihao ,shijie !`,该怎么办呢？`git log`中已经没有了这部分信息，这时需要使用另外一个命令：`git relog`。

```
$ git reflog
d2619b5 HEAD@{0}: reset: moving to HEAD^
47303ff HEAD@{1}: commit: add nihao ,shijie ! in the file of 123.txt
d2619b5 HEAD@{2}: commit: add hello world! in the file of 123.txt
02c61c8 HEAD@{3}: commit: add the file of 123.txt
```

从上面的命令，可以看到：`add nihao ,shijie ! in the file of 123.txt`的版本号是`47303ff`，这时可以使用`git reset --hard 47303ff`,如下：

```
$ git reset --hard 47303ff
HEAD is now at 47303ff add nihao ,shijie ! in the file of 123.txt
```

这时再看123.txt中的内容：

```
hello world!

nihao ,shiijie !
```

发现`nihao ,shijie !`已经回来了。

## 7. 再讲工作区，暂存区和版本库

工作区就是我们电脑中的目录，通过`add`操作可以将文件放到暂存区，通过`commit`可以将文件放到版本库。其中`add`操作是可以多次进行的，也就是说可以多次将多个文件放到暂存区，然后一次性进行`commit`,即一次性将之前多次放到暂存区的文件一次性放到版本库。

## 8. Git撤销修改和删除文件

### 8.1 Git撤销修改

现在我在123.txt中新添加了一行`hello moto！`,123.txt中的文件内容为：

```
hello world!

nihao ,shiijie !

hello moto!
```

现在如果想要撤销这些修改该怎么办呢？有如下两种方法：第一：如果我知道要删掉那些内容的话，直接手动更改去掉那些需要的文件，然后add添加到暂存区，最后commit掉。第二：可以按以前的方法直接恢复到上一个版本，使用 `git reset --hard HEAD^`。当然，如果你不想用这两种方法，还有第三种：在`4. Git常用命令`中，我说到了`git checkout -- XXX`命令可以丢弃工作区文件XXX的修改。例如：

```
$ git checkout -- '123.txt'
```

现在再来看123.txt中的文件内容：

```
hello world!

nihao ,shiijie !
```

发下已经没有了`hello moto!`。
需要强调的是：对于已经放到暂存区的修改，`$ git checkout -- XXX`是不可以回退的。例如，在123.txt中添加内容hahaha，然后add到版本库，再次在123.txt中添加内容lalala。此时123.txt中的内容是：

```
hello world!

nihao ,shiijie !

hahaha

lalala
```

使用`git checkout -- 123.txt`之后，123.txt中的内容为：

```
hello world!

nihao ,shiijie !

hahaha
```

发现已经被提交到暂存区的内容hahaha并没有撤销。

### 8.2 Git删除和恢复删除的文件

#### 8.2.1 删除文件

通过`git rm XXX`，来删除文件XXX。
例如：

```
$ ls
1.txt  123  123.txt  789.txt  hello  README.md
```

原来的工作区中有如上的文件，其中1.txt的内容为：

```
$ cat 1.txt
123123
```

现在通过命令`rm 1.txt`来删除1.txt，

```
$ ls
123  123.txt  789.txt  hello  README.md
```

发现工作区中已经没有了1.txt。

#### 8.2.2 恢复文件

通过命令`$ git checkout -- 1.txt`来恢复被删除的文件，现在发现1.txt又回来了。

```
$ ls
1.txt  123  123.txt  789.txt  hello  README.md
```

## 9. 远程仓库

我们使用的大多数的远程仓库是`GitHub`，关于GitHub的使用，下面做一些简单的介绍。

### 9.1 创建并添加 SSH Key

第一步：通过命令`ssh-keygen -t rsa –C "youremail@example.com"`来创建SSH Key，我们通过访问C盘用户目录下的`.ssh`文件夹，发现多了两个文件：`id_rsa`，`id_rsa.pub`。其中id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。第二步：登录github,打开`settings`中的`SSH Keys`页面，然后点击`Add SSH Key`,填上任意title，在Key文本框里黏贴id_rsa.pub文件的内容即可。

### 9.2 关联远程仓库

假设有如下情景：我们已经在本地创建了一个Git仓库后，又想在github创建一个Git仓库，并且希望这两个仓库进行远程同步，这样github的仓库可以作为备份，又可以其他人通过该仓库来协作。这种情况该如何处理呢？

#### 9.2.1 在GitHub上创建一个新的仓库

首先，登录github上，然后在右上角找到“create a new repo”创建一个新的仓库。比如，新建一个名为test的远程仓库,这时的仓库是空的。

#### 9.2.2 本地与远程关联

通过命令`git remote add origin https://github.com/QiXingjun/test`，将本地的仓库与远程的仓库相关联。把本地库的内容推送到远程，使用 `git push -u origin master`命令，实际上是把当前分支master推送到远程。
由于远程库是空的，我们第一次推送master分支时，加上了`-u`参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令,使用不带`-u`的命令`git push origin master`。推送成功后，可以立刻在github页面中看到远程库的内容已经和本地一模一样了。

#### 9.2.3 克隆远程仓库

通过命令`git clone https://github.com/QiXingjun/test`就可以克隆远程仓库到本地了。

## 10. 创建与合并分支

### 10.1 创建分支

首先，创建dev分支，然后切换到dev分支：
```
$ git checkout -b dev
Switched to a new branch 'dev'
```

git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：

```
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
```

然后，用`git branch`命令查看当前分支：

```
$ git branch
* dev
  master
```
git branch命令会列出所有分支，当前分支前面会标一个`*`号。

然后，我们就可以在dev分支上正常提交，比如对1.txt做个修改，加上一行：

```
this is dev 1.txt
```
然后提交：

```
$ git commit -m "dev 1.txt"
[dev c0b9cc4] dev 1.txt
 1 file changed, 1 deletion(-)
```
现在，dev分支的工作完成，我们就可以切换回master分支：

```
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
```

切换回master分支后，再查看1.txt文件，

```
$ cat 1.txt
123123
```

刚才添加的内容不见了！因为那个提交是在dev分支上，而master分支此刻的提交点并没有改变。

### 10.2 合并分支

现在，我们把dev分支的工作成果通过命令`git merge dev`合并到master分支上：

```
$ git merge dev
Updating 8a348fb..c0b9cc4
Fast-forward
 1.txt | 2 ++
 1 file changed, 2 insertions(+)
```
git merge命令用于合并指定分支到当前分支。合并后，再查看1.txt的内容,

```
$ cat 1.txt
123123

this is dev 1.txt
```

可以看到，和dev分支的最新提交是完全一样的。

注意到上面的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。

当然，也不是每次合并都能Fast-forward，我们后面会讲其他方式的合并。

合并完成后，就可以放心地通过命令`git branch -d dev`删除dev分支了：

```
$ git branch -d dev
Deleted branch dev (was c0b9cc4).
```

删除后，查看branch，就只剩下master分支了：

```
$ git branch
* master
```

因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。

### 10.3 冲突解决

先看一下master分支下的文件：

```
XingJun Qi@XingJunQi-PC MINGW64 /g/ECollaboration (master)
$ ls
hello.txt  hi.txt
```

然后看一下dev分支下的文件：

```
XingJun Qi@XingJunQi-PC MINGW64 /g/ECollaboration (dev)
$ ls
hello.txt
```

现在不管是master分支中的hello.txt文件还是dev分支下的hello.txt文件，内容都是

```
$ cat hello.txt
123
```

首先在master分支下修改hello.txt文件，在下一行添加文字`master add1`,然后进行add和commit将修改提交到版本库。此时，master分支下的hello.txt文件中的内容是：

```
$ cat hello.txt
123

master add1
```

然后，切换到dev分支下，查看dev分支下的hello.txt文件，内容仍为：

```
$ cat hello.txt
123
```

编辑dev分支下的hello.txt文件，在下一行添加文字`dev add1`，然后进行add和commit将修改提交到版本库。此时dev分支下的hello.txt文件中的内容为：

```
$ cat hello.txt
123

dev add1
```

现在，回到master分支中，使用命令`git merge dev`想要将dev分支合并带master分支，但是此时会报错。

```
$ git merge dev
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Automatic merge failed; fix conflicts and then commit the result.
```

这时，查看hello.txt文件,会显示如下内容：

```
$ cat hello.txt
123

<<<<<<< HEAD
master add1
=======
dev add1
>>>>>>> dev
```

Git用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容。这时，修改master分支下的hello.txt文件为：

```
$ cat hello.txt
123
```

再次执行`git merge dev`操作，发现hello.txt文件的内容已经变成了：

```
$ cat hello.txt
123

dev add1
```
至此，冲突已经解决。
通过命令`git log --graph --pretty=oneline --abbrev-commit`可以查看分支合并图。

### 10.4 分支策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到`master`上，在`master`分支发布1.0版本；

你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，比如:zhangsandev,lisidev，时不时地往dev分支上合并就可以了。

### 10.5 bug分支

在软件开发过程中，我们会遇到各种各样的bug。有了bug就需要修复，在Git中，由于分支是如此的强大，所以，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。

例如：当你接到一个修复一个代号101的bug的任务时，很自然地，你想创建一个分支issue-101来修复它，但是，等等，当前正在dev上进行的工作还没有提交。
并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？
幸好，Git还提供了一个`git stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：
现在，用git status查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。

首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支。
修复完成后，切换到master分支，并完成合并，最后删除issue-101分支。
然后用`git stash list`命令看看,
工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；

另一种方式是用`git stash pop`，恢复的同时把stash内容也删了。

### 10.6 多人协作开发

当你从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`。

要查看远程库的信息，用`git remote`：

```
$ git remote
origin
```

或者，用`git remote -v`显示更详细的信息：

```
$ git remote -v
origin  https://github.com/QiXingjun/ECollaboration.git (fetch)
origin  https://github.com/QiXingjun/ECollaboration.git (push)
```

上面显示了可以抓取和推送的origin的地址，如果没有推送权限，就看不到push的地址。

#### 10.6.1 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```
$ git push origin master
Username for 'https://github.com': QiXingjun
Everything up-to-date
```
如果要推送其他分支，比如dev，首先要通过命令`git checkout dev`切换到dev分支，然后进行推送：

```
$ git push origin dev
Username for 'https://github.com': QiXingjun
Everything up-to-date
```

但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？

1. master分支是主分支，因此要时刻与远程同步；

2. dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；

3. bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；


#### 10.6.2 多人协作

多人协作时，大家都会往master和dev分支上推送各自的修改。

因此，多人协作的工作模式通常是这样：

首先，可以试图用`git push origin branch-name`推送自己的修改(其中branch-name是需要推送的分支)；

如果推送失败，则因为远程分支版本比你的本地版本更新，需要先用`git pull`试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用`git push origin branch-name`推送就能成功！

如果git pull提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream branch-name origin/branch-name`。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

## 11. 标签管理

发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。
Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针，所以，创建和删除标签都是瞬间完成的。

### 11.1 创建标签

在Git中打标签非常简单，首先，切换到需要打标签的分支上，
然后，敲命令`git tag <name>`就可以打一个新标签：

```
$ git tag v1.1
```

可以用命令`git tag`查看所有标签：

```
$ git tag
v1.0
v1.1
```

默认标签是打在最新提交的commit上的。有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打，怎么办？

方法是通过`git reflog`命令找到历史提交的`commit id`，然后打上就可以了：

```
$ git reflog
6301c0f HEAD@{0}: checkout: moving from dev to master
76e4f36 HEAD@{1}: checkout: moving from master to dev
6301c0f HEAD@{2}: checkout: moving from dev1 to master
6301c0f HEAD@{3}: checkout: moving from master to dev1
6301c0f HEAD@{4}: merge dev: Fast-forward
658530b HEAD@{5}: checkout: moving from dev to master
6301c0f HEAD@{6}: commit: add come words
```

比方说要对`merge dev: Fast-forward`这次提交打标签，它对应的`commit id`是6301c0f，敲入命令：

```
$ git tag v0.9 6301c0f
```

再用命令`git tag`查看标签：

```
$ git tag
v0.9
v1.0
v1.1
```
注意，标签不是按时间顺序列出，而是按字母排序的。可以用`git show <tagname>`查看标签信息：

```
$ git show v1.0
commit 6301c0f675a9508eb18aed97f08d724725be571b
Author: QiXingjun <18862141550@163.com>
Date:   Tue Sep 27 12:43:00 2016 +0800

    add come words

diff --git a/hello.txt b/hello.txt
index 190a180..e1046c2 100644
--- a/hello.txt
+++ b/hello.txt
@@ -1 +1,3 @@
 123
+
+dev add1
```

### 11.2 删除分支

如果标签打错了，也可以删除：

```
$ git tag -d v0.9
Deleted tag 'v0.9' (was 6301c0f)
```
因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。
如果要推送某个标签到远程，使用命令`git push origin <tagname>`：

```
$ git push origin v1.0
Username for 'https://github.com': QiXingjun
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/QiXingjun/ECollaboration.git
 * [new tag]         v1.0 -> v1.0
```

或者，一次性推送全部尚未推送到远程的本地标签：

```
$ git push origin --tags
Username for 'https://github.com': QiXingjun
Counting objects: 1, done.
Writing objects: 100% (1/1), 159 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To https://github.com/QiXingjun/ECollaboration.git
 * [new tag]         v1.1 -> v1.1
 * [new tag]         v1.2 -> v1.2
```

## 12. 总结

至此，Git的学习就基本差不多了，如果想要更加深入的理解其中的一些命令，还需要在实践中慢慢体会。
