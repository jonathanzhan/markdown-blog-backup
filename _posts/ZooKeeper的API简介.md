---
title:  ZooKeeper的API简介
date: 2016-06-21 16:11:41
tags:
- zookeeper
categories:
- 微服务
toc: true
---

## 简介
ZooKeeper是 ZooKeeper 客户端库的主要类文件。如果要使用 ZooKeeper 服务，应用程序首先必须创建一个 Zookeeper 实例，这时就需要使用此类。一旦客户端和 ZooKeeper 服务建立起连接， ZooKeeper 系统将会分配给此连接回话一个 ID 值，并且客户端将会周期地向服务器发送心跳来维持会话的连接。只要连接有效，客户端就可以调用 ZooKeeper API 来做相应的处理。

<!-- more -->

ZooKeeper API 共包含 5 个包，分别为： 
org.apache.zookeeper,
org.apache.zookeeper.data,
org.apache.zookeeper.server,
org.apache.zookeeper.server.quorum,
org.apache.zookeeper.server.upgrade
其中 org.apache.zookeeper 包含 ZooKeeper 类，它我们编程时最常用的类文件。

官网的API DOC地址：[https://zookeeper.apache.org/doc/r3.4.8/api/index.html](https://zookeeper.apache.org/doc/r3.4.8/api/index.html)


## 实例

具体的客户端使用案例参看以下代码

```java
public class ZooKeeperDemo {
	Logger logger = LoggerFactory.getLogger(ZooKeeperDemo.class);

	private static final int SESSION_TIMEOUT = 30000;

	ZooKeeper zk;

	Watcher watch = new Watcher() {
		public void process(WatchedEvent event) {
			System.out.println(event.toString());
		}
	};

	/**
		* 初始化zookeeper
		* @throws IOException
		*/
	private void createZkInstance() throws IOException {
		zk = new ZooKeeper("127.0.0.1:2181", SESSION_TIMEOUT, this.watch);
	}

	/**
		* zookeeper的一些操作
		* @throws KeeperException
		* @throws InterruptedException
		*/
	private void zkOperations() throws KeeperException, InterruptedException {
		logger.debug("创建节点：{}，数据：{}，权限：{}，节点类型：{}", "test", "test1", Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		zk.create("/test", "test1".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

		//获取节点信息
		System.out.println(new String(zk.getData("/test", false, null)));

		//修改节点
		zk.setData("/test", "test2".getBytes(), -1);
		System.out.println(new String(zk.getData("/test", false, null)));

		System.out.println("删除节点 ");
		zk.delete("/test", -1);

		//判断节点是否存在
		System.out.println(" 节点状态： [" + zk.exists("/test", false) + "]");

	}

	/**
		* zookeeper关闭
		* @throws InterruptedException
		*/
	private void zkClose() throws InterruptedException {
		zk.close();
	}


	public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
		ZooKeeperDemo demo = new ZooKeeperDemo();
		demo.createZkInstance();
		demo.zkOperations();
		demo.zkClose();
	}

}
```
