---
title: Mac OS安装Redis
date: 2016-07-06 09:26:00
tags:
- redis
categories:
- 运维实施
toc: true
---

Mac OS X安装Redis的步骤说明
<!-- more -->
### 下载、解压、重命名并且编译安装Redis

```bash
~ wget http://download.redis.io/releases/redis-3.0.5.tar.gz 
~ tar xzf redis-3.0.5.tar.gz
~ mv redis-3.0.5 redis
~ cd redis
~ make
~ make test
~ make install
```

### 配置文件redis.conf
redis解压目录里有一个配置文件redis.conf ，编辑此配置文件，找到 dir  ./  这一行。redis会将内存中的数据写入文件中，而此配置就是指定数据文件保存的路径。我本机指定的目录为：

```bash
dir /Users/whatlookingfor/tools/redis_data/
```
编辑过后，将配置文件移动到 /usr/local/etc 目录下

```
~ sudo mv redis.conf /usr/local/etc
```

### 启动Redis
在终端输入：

```bash
~ /usr/local/bin/redis-server /usr/local/etc/redis.conf
```
出现服务启动成功画面。

### 测试连通性 

```bash
~ cd /usr/local/bin
~ ./redis-cli
127.0.0.1:6379> set jackiehff hi
OK
127.0.0.1:6379> get jackiehff
"hi"
```
### 设置开机自动启动redis server
新建plist文件

```bash
~ sudo vi /Library/LaunchDaemons/io.redis.redis-server.plist
```
文件内容如下 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>io.redis.redis-server</string>
  <key>ProgramArguments</key>
  <array>
        <string>/usr/local/bin/redis-server</string>
        <string>/usr/local/etc/redis.conf</string>
  </array>
  <key>RunAtLoad</key><true/> </dict>
</plist>
```
使用launchctl设置开机自动启动

```bash
~ sudo launchctl load /Library/LaunchDaemons/io.redis.redis-server.plist
```

使用launchctl启动redis server

```bash
~ sudo launchctl start io.redis.redis-server
```

使用launchctl停止redis server

```bash
~ sudo launchctl stop io.redis.redis-server
```
