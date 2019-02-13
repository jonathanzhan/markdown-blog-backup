---
title: linux常用命令
date: 2016-11-16 17:47:30
tags:
- linux
categories:
- 运维实施
toc: true
---
本文记录了在开发部署过程中的一些常用命令，持续更新中
<!-- more -->
#### 创建文件
` touch fileName `

#### 解压文件
`  tar -zxvf jdk-8u121-linux-x64.tar.gz `

说明
<pre>
tar
-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件

这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个


-z：有gzip属性的
-j：有bz2属性的
-Z：有compress属性的
-v：显示所有过程
-O：将文件解开到标准输出

-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。


压缩
tar –cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg
tar –czf jpg.tar.gz *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar –cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
tar –cZf jpg.tar.Z *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for Linux
zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux

解压
tar –xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2   //解压 tar.bz2
tar –xZvf file.tar.Z   //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip
</pre>


* *.tar 用 tar –xvf 解压
* *.gz 用 gzip -d或者gunzip 解压
* *.tar.gz和*.tgz 用 tar –xzf 解压
* *.bz2 用 bzip2 -d或者用bunzip2 解压
* *.tar.bz2用tar –xjf 解压
* *.Z 用 uncompress 解压
* *.tar.Z 用tar –xZf 解压
* *.rar 用 unrar e解压
* *.zip 用 unzip 解压

#### 查看进程
如下:
```
[root@localhost local]# ps -ef |grep tomcat
root      2917  2655  0 14:45 pts/0    00:00:00 grep tomcat
```
字段含义如下：

|UID |PID| PPID|C|STIME|TTY|TIME|CMD|
| -- |-- | ----|--|----|---|----|---|
|root|2917|2655| 0|14:45|pts/0|00:00:00|grep tomcat|

ps:将某个进程显示出来
-A 　显示所有程序。 
-e 　此参数的效果和指定"A"参数相同。
-f 　显示UID,PPIP,C与STIME栏位。 
grep命令是查找
中间的|是管道命令 是指ps命令与grep同时执行
这条命令的意思是显示有关tomcat有关的进程

* UID 程序被该 UID 所拥有
* PID 就是这个程序的 ID 
* PPID 则是其上级父程序的ID
* C CPU 使用的资源百分比
* STIME 系统启动时间
* TTY 登入者的终端机位置
* TIME 使用掉的 CPU 时间。
* CMD 所下达的指令为何

> 因为ps -ef是显示所有进程的消息，包括tomcat和grep tomcat这两个甚至包括ps -ef本身，而grep是查找输出包含想要的字符串的行，也就是说grep tomcat是在所有运行的进程中查找输出包含“tomcat”字符串的输出行，这里面就包含tomcat，和grep tomcat 两个进程。如果运行了会显示两条输出一条是tomcat的，令一条是grep tomcat的,如果没运行只会显示grep tomcat的。


### 防火墙端口设置
打开8080端口，vi /etc/sysconfig/iptables，
在默认的22端口这条规则的下面添加(第一条是默认的22端口)
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 8080 -j ACCEPT
```
重启防火墙
` /etc/init.d/iptables restart `
