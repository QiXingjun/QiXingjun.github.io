---
layout: post
title: tar压缩解压缩命令详解
categories: Ubuntu
description: tar压缩解压缩命令详解
keywords: Ubuntu,tar,压缩
---

如果经常使用Ubuntu的同学一定对tar命令不陌生，我也经常使用此命令，但是一直没有系统的对tar的使用进行系统的总结，现在就对tar的使用做一个小结。

## tar命令

- -c: 建立压缩档案 

- -x：解压

- -t：查看内容

- -r：向压缩归档文件末尾追加文件

- -u：更新原压缩包中的文件

**注意：这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。**


下面的参数是根据需要在压缩或解压档案时可选的。

- -z：有gzip属性的

- -j：有bz2属性的

- -Z：有compress属性的

- -v：显示所有过程

- -O：将文件解开到标准输出

**注意：参数-f是必须的 -f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。**

- tar -cf all.tar *.jpg 这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。

- tar -rf all.tar *.gif 这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。 

- tar -uf all.tar logo.gif 这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。 

- tar -tf all.tar 这条命令是列出all.tar包中所有文件，-t是列出文件的意思。

- tar -xf all.tar 这条命令是解出all.tar包中所有文件，-x是解开的意思。

### 查看

- tar -tf aaa.tar.gz   在不解压的情况下查看压缩包的内容

### 压缩

- tar –cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg

- tar –czf jpg.tar.gz *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz

- tar –cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2

- tar –cZf jpg.tar.Z *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z

### 解压

- tar –xvf file.tar //解压 tar包

- tar -xzvf file.tar.gz //解压tar.gz

- tar -xjvf file.tar.bz2   //解压 tar.bz2tar –xZvf file.tar.Z //解压tar.Z


## 总结

- *.tar 用 tar –xvf 解压

- *.gz 用 gzip -d或者gunzip 解压

- *.tar.gz和*.tgz 用 tar –xzf 解压

- *.bz2 用 bzip2 -d或者用bunzip2 解压

- *.tar.bz2用tar –xjf 解压

- *.Z 用 uncompress 解压

- *.tar.Z 用tar –xZf 解压

## by the way

如果是解压zip文件，直接使用命令`unzip 文件名　目录名`　其中目录名是可以省略的。
