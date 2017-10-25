---
title: mysql快速复制数据库
date: 2017-10-25 18:08:57
tags:
- mysql
categories:
- 数据库
---
为了方便快速复制一个数据库，可以用以下命令,将db1数据库的数据以及表结构复制到newdb数据库
<!-- more -->
创建新的数据库
` #mysql -u root -p123456 `

``` mysql>CREATE DATABASE `newdb` DEFAULT CHARACTER SET UTF8 COLLATE UTF8_GENERAL_CI; ```
复制数据库，使用mysqldump及mysql的命令组合，一次性完成复制
``` #mysqldump db1 -u root -p123456 --add-drop-table | mysql newdb -u root -p123456 ```
注意-p123456参数的写法：-p后面直接跟密码，中间没有空格)

以上是在同一台MySQL服务器上复制数据库的方法。如果要复制到远程另一台MySQL服务器上，可以使用mysql的“ -h 主机名/ip”参数。前提是mysql允许远程连接，且远程复制的传输效率和时间可以接受。

不在同一个mysql服务器上
``` #mysqldump db1 -uroot -p123456 --add-drop-table | mysql -h 192.168.1.22 newdb -u root -p123456 ```
