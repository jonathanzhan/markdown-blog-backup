---
title: Jedis使用说明
date: 2016-07-07 17:16:42
tags:
- redis
categories:
- java
toc: true
---
Jedis 是 Redis 官方首选的 Java 客户端开发包。使用Jedis提供的Java API对Redis进行操作，是Redis官方推崇的方式；并且，使用Jedis提供的对Redis的支持也最为灵活、全面；不足之处，就是编码复杂度较高。
<!-- more -->
## jedis基本使用
引入jedis的依赖包
```xml
<dependency>
   <groupId>redis.clients</groupId>
   <artifactId>jedis</artifactId>
   <version>2.8.1</version>
   <type>jar</type>
</dependency>
```
建立一个到Redis的连接，并进行操作

```java
@Test
public void BasicJedisTest(){
	//连接redis
	Jedis jedis = new Jedis("127.0.0.1",6379);
	//键值存储
	jedis.set("name","hello world");
	//取值
	String name = jedis.get("name");
	logger.debug(name);
	//删除
	jedis.del("name");
}
```
## redis连接池的使用
在Jedis中，管理Redis连接的类是JedisPool。

```java
@Test
public void JedisPoolTest() {
	//连接池初始化
	JedisPool jedisPool = new JedisPool("127.0.0.1", 6379);
	Jedis jedis = null;
	try {
		//获取链接
		jedis = jedisPool.getResource();
		//键值存储
		jedis.set("name", "hello world");
		//取值
		String name = jedis.get("name");
		logger.debug(name);
		//删除
		jedis.del("name");
	} finally {
		//关闭连接
		if (jedis != null) {
			jedis.close();
		}
	}
	//连接池销毁
	jedisPool.destroy();
}
```
## jedis 常用操作
jedis 常用的操作对象有：字符串，列表(list)、集合(set)、有序集合(sorted set)、哈希表(hash)等数据结构

 - 字符串的相关操作
```java
//键值存储
jedis.set("name", "hello world");
//取值
String name = jedis.get("name");
logger.debug(name);
//删除
jedis.del("name");
```

 - 列表的相关操作
```java
//右边入队
jedis.rpush("userList", "James");
jedis.rpush("userList", "Jonathan");
jedis.rpush("userList", "Steve");
//返回列表范围：从0开始，到最后一个(-1),包含截止
List<String> userList = jedis.lrange("userList", 0, -1);
//设置：位置1处为新值
jedis.lset("userList", 1, "Nick Xu");
//左边出队
jedis.lpop("userList");
//返回长度：
Long size = jedis.llen("userList");
//进行裁剪,从0开始,到最后一个(-1),包含截止
jedis.ltrim("userList",0,0);
```

 - 集合的相关操作
集合和列表不同，集合中的元素是无序的，因此元素也不能重复
```java
//集合数据的添加,一次可添加多个
jedis.sadd("code", "java");
jedis.sadd("code", "node", "python", "scala");
jedis.sadd("code", "java", "swift");
//集合的遍历
Set<String> codeSet = jedis.smembers("code");
//元素的删除(一次可删除多个)
jedis.srem("code", "node");
//返回集合的长度
Long size = jedis.scard("code");
//判断集合是否包含某个元素
Boolean isMember = jedis.sismember("code", "java");
//集合的运算
jedis.sadd("code1", "java", "C#", "C");
//集合的交运算
Set<String> sinterSet = jedis.sinter("code", "code1");
//集合的差集
Set<String> sdiffSet = jedis.sdiff("code", "code1");
//集合的并集
Set<String> sunionSet = jedis.sunion("code", "code1");
```
 - 有序集合的相关操作
 有序集合在集合的基础上，增加了一个用于排序的参数。
```java
//元素的添加,根据第二个参数进行排序
jedis.zadd("userSet", 1, "James");
jedis.zadd("userSet", 3, "John");
jedis.zadd("userSet", 2, "Jonathan");
//元素相同时，更新当前的权重。
jedis.zadd("userSet", 4, "Jonathan");
//找到从0到-1的所有元素。
Set<String> userSet = jedis.zrange("userSet", 1, 2);
```
 - 哈希表的相关操作

```java
//初始化map
Map<String, String> map = new HashMap<String, String>();
map.put("age", "20");
map.put("weight", "90KG");
map.put("height", "180cm");
//存放hashMap
jedis.hmset("people", map);
//获取hashMap
List<String> data = jedis.hmget("people", "age", "height");
```

 - 其他操作
```java
//对key的模糊查询
Set<String> keys = jedis.keys("co*");
//判断某个key是否存在
Boolean isExists = jedis.exists("code2");
//设置过期时间
jedis.expire("code",60);
//获取剩余存活时间(-1代表永久)
long seconds = jedis.ttl("code");
//去掉key的expire设置：不再有失效时间
jedis.persist("code");

//为指定的 key 设置值及其过期时间。如果key已经存在,将会替换旧的值
jedis.setex("name",100,"hello");

//int类型采用string类型的方式存储
jedis.set("amount", 2 + "");
//递增或递减：incr()/decr()
jedis.incr("amount");
//增加或减少：incrBy()/decrBy()
jedis.incrBy("amount", 5);
//清空当前库
jedis.flushDB();
//清空所有库
jedis.flushAll();
```

