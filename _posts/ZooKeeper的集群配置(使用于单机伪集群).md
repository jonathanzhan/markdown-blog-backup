---
title:  ZooKeeper的集群配置(使用于单机伪集群)
date: 2016-06-20 14:28:34
tags:
- zookeeper
categories:
- 微服务
toc: true
---

### 前提
ZooKeeper 是为了解决分布式应用场景的，所以经常运行与集群模式下。
但是可能由于没有多余的机子或者仅仅是对ZooKeeper做个了解，那么可以在一个机器上部署三个ZooKeeper服务来组成一个集群(ZooKeeper要求最少需要三个服务)
<!-- more -->
### 安装
 zookeeper下载地址为[http://www.apache.org/dyn/closer.cgi/zookeeper/](http://www.apache.org/dyn/closer.cgi/zookeeper/)

1. 新建zookeeper文件夹，并在其下建立server1,server2,server3文件夹，分别用于放zookeeper的安装文件。
2. 将zookeeper的安装包解压分别放到server1,server2,server3三个文件夹中。
3. 新建目录data：/zookeeper/server1/zookeeper/data
新建目录logs：/zookeeper/server1/zookeeper/logs
新建文件myid：/zookeeper/server1/zookeeper/data/myid
myid文件无后缀，内容为1，与serverX的X保持一致。server1内容为1，server2内容为2
4. 编辑zookeeper目录下的conf/zoo.cfg,如果没有，则将zoo_sample.cfg复制一份，重命名即可。(注意:clientPort这个端口如果你是在1台机器上部署多个server,那么每台机器都要不同的clientPort，比如我server1是3181（2181这个端口好像被占用了）,server2是2182，server3是2183，dataDir和dataLogDir也需要区分下，如果是在不同机器上部署，则不用考虑端口号不一致的问题)
server1的zoo.cfg如下： 
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/zookeeper/server1/zookeeper/data
clientPort=2181
server.1=127.0.0.1:2881:3881
server.2=127.0.0.1:2882:3882
server.3=127.0.0.1:2883:3883
```
server2的zoo.cfg如下:
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/zookeeper/server2/zookeeper/data
clientPort=2182
server.1=127.0.0.1:2881:3881
server.2=127.0.0.1:2882:3882
server.3=127.0.0.1:2883:3883
```

server3的zoo.cfg如下:

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/zookeeper/server3/zookeeper/data
clientPort=2183
server.1=127.0.0.1:2881:3881
server.2=127.0.0.1:2882:3882
server.3=127.0.0.1:2883:3883
```

### 启动服务
 分别进入各个zookeeper的bin目录，然后运行“./zkServer.sh start”来启动一个ZooKeeper服务。
 
### 客户端连接
 随便进入一个zookeeper的bin目录，启动客户端服务。“./zkCli.sh –server 127.0.0.1:2181"端口号根据自己在zoo.cfg中设置的来

然后在其中的一个client上进行一个写操作

```
＄bin/zkCli.sh -server 127.0.0.1:2181
[127.0.0.1:2181(CONNECTED) 1] create /mytest test 
[zk: 127.0.0.1:2181(CONNECTED) 3] ls / [mytest, zookeeper]
```
在其他机器上(服务)查询：

```
＄bin/zkCli.sh -server 127.0.0.1:2182
[zk: 127.0.0.1:2182(CONNECTED) 3] ls / [mytest, zookeeper]
```
### 查看ZooKeeper状态
 通过运行 “./zkServer.sh status”来查看各个ZooKeeper的状态
 
