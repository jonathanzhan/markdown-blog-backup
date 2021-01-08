---
title: mysql关于锁表的监控
date: 2016-10-25 18:07:24
tags:
- mysql
categories:
- 数据库
---
1、查看锁表的命令
` show open tables where in_use>0; `
2、查看当前进程

` show processlist; `
3、杀掉进程

` kill process_id; `