---
title:  ZooKeeper的常用命令
date: 2016-06-20 17:28:34
tags:
- zookeeper
categories:
- 微服务
toc: false
---
本文主要介绍zookeeper的常用命令
<!-- more -->
## ZooKeeper服务命令 ##

 - 启动ZK服务:       sh bin/zkServer.sh start
 - 查看ZK服务状态: sh bin/zkServer.sh status
 - 停止ZK服务:       sh bin/zkServer.sh stop
 - 重启ZK服务:       sh bin/zkServer.sh restart

## ZooKeeper客户端命令 ##

 - 连接到 ZooKeeper 服务: ./zkCli.sh -server 127.0.0.1:2181
 - 显示根目录下、文件(使用 ls 命令来查看当前 ZooKeeper 中所包含的内容)： ls /
 - 显示根目录下、文件(查看当前节点数据并能看到更新次数等数据): ls2 /
 - 创建一个新的 znode： create /zk myData 。这个命令创建了一个新的 znode 节点“ zk ”以及与它关联的字符串
 - 获取文件内容： get /zk 确认 znode 是否包含我们所创建的字符串
 - 修改文件内容： set /zk "zkbak" 对 zk 所关联的字符串进行设置
 - 删除文件： delete /zk 将刚才创建的 znode 删除
 - 退出客户端： quit
 - 帮助命令： help
