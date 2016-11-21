---
layout: post
title: 在ubuntu下安装Chrome
categories: Ubuntu
description: 在ubuntu下安装Chrome，解决firefox和Chromium下flash无法使用的问题。
keywords: Ubuntu,Flash,chrome,firefox 
---

之前使用ubuntu一直是用来Coding,现在由于一些原因，需要看些视频，可是之前安装的firefox和Chromium都不是默认支持flash的，然后就找了好多方法。
比如：拷贝libflashplayer.so到浏览器的插件目录下，还有就是从软件中心安装flash，可是这几种方法使用了之后都没有用。所以打算直接安装chrome，因为
chrome默认是支持flash的，并不需要额外安装。

## 1. 安装PPA

从[Google Linux Repository](http://www.google.com/linuxrepositories/)下载Key，如果链接打不开，[点击这里](https://pan.baidu.com/s/1mhQm0nY),
密码是`1xbr`。
下载之后，在终端执行

`wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -`

安装完毕之后，执行

`sudo sh -c "echo 'deb http://dl.google.com/linux/chrome/deb/ stable main' >> /etc/apt/sources.list`

这样PPA就安装完毕，接下来执行以下命令进行更新。

`sudo apt-get update`

## 2. Chrome的安装

在终端输入：

`sudo apt-get install google-chrome-stable`

命令执行完毕之后，Chrome就安装好了，无需安装Flash就可以播放视频，再也不用为了Ubuntu下为firefox和Chromium安装Flash而费心了。


