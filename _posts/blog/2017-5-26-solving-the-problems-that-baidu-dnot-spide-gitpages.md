---
layout: post
title: 同时同步托管到github和coding.net,解决百度不收录GitPages问题
categories: Git 
description: 同时同步托管到github和coding.net,解决百度不收录GitPages问题
keywords: Git Tools
---

## 问题：
由于github禁止百度爬虫访问，造成托管在github Pages上的博客无法被百度收录。相关问题可以通过百度站长平台的抓取诊断再现，每次都是403 Forbidden的错误…

## 解决方案：
将hexo同时同步到github和coding.net

### 具体如下：

#### （1）在coding.net开启pages功能

在coding.net上注册后，创建一个与用户名相同的项目，如用户名是henryqi，则项目名称也是henryqi。创建项目后，在项目的Pages页面里，修改Pages服务的关联分支为master，然后开启Pages服务。再填入你要绑定的域名。

#### （2）在本机建立coding.net的ssh key

首先本地需要安装git客户端，在任意路径右键->`git bash here`

`ssh-keygen -t rsa -C "yourmail@domain.com"`
可以一路按enter键，然后可在C:\Users\主机名.ssh下找到，文件名：id_rsa(私钥)，id_rsa.pub(公钥)

最后在coding.net，刚建立的pages项目中输入这个公钥

#### （3）修改博客目录的_config.yml，设置多个仓库同步

```
deploy:
  type: git
  repo: 
    github: https://github.com/QiXingjun/QiXingjun.github.io.git
    coding: git@git.coding.net:henryqi/henryqi.git
  branch: master
  message: just do it
```

#### （4）修改博客目录.git目录下的config文件

```
[remote "origin"]
	url = https://github.com/QiXingjun/QiXingjun.github.io
	url = https://git.coding.net/henryqi/henryqi.git
	fetch = +refs/heads/*:refs/remotes/origin/*
```
这样，每次push都会同时推送到github和coding。

#### （5）在DNSPOD设置CNAME

首先要在coding里你的pages项目绑定一个域名，然后会提示你需要将你的域名cname到pages.coding.me，照着做就好。设置完之后将讲这个cname的线路类型选择为百度，这样百度爬虫就会去找你在coding上面的镜像了。

## 总结：

按照上面的方法设置之后，你的博客就可以被百度收录了，如果有什么问题，欢迎在下面留言！

