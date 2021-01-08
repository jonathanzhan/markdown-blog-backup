---
title: RedisTemplate的使用说明(序列化以及事物)
date: 2016-07-08 18:47:02
tags:
- redis
categories:
- java
toc: true
---

SDR官方文档中对RedisTemplate的介绍：the template is in fact the central class of the Redis module due to its rich feature set. The template offers a high-level abstraction for Redis interactions.
通过RedisTemplate可以调用ValueOperations,ListOperations,SetOperations,ZSetOperations等方法，分别是对Redis命令的高级封装。
<!-- more -->
### RedisTemplate的序列化

以下是RedisTemplate的部分代码片段:
```java
if (defaultSerializer == null) {
	defaultSerializer = new JdkSerializationRedisSerializer(classLoader != null ? classLoader : this.getClass().getClassLoader());
}
```
从代码可以看出，Spring提供了默认的StringSerializer和JdkSerializer。
StringSerializer就是通过String.getBytes()来实现的，而且在Redis中，所有存储的值都是字符串类型的。所以这种方法保存后，通过Redis-cli控制台，是可以清楚的查看到我们保存了什么key,value是什么。

JdkSerializationRedisSerializer，这个序列化方法是Idk提供的，要求要被序列化的类继承自Serializeable接口，然后通过Jdk对象序列化的方法保存，这个序列化保存的对象，即使是个String类型的，在redis控制台，也是看不出来的，因为它保存了一些对象的类型什么的额外信息。

所以在上篇文章中的Spring配置如下

```xml
<!--对key的默认序列化器。默认值是StringSerializer-->
<bean id="keySerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />
<!--是对value的默认序列化器，默认值是取自DefaultSerializer的JdkSerializationRedisSerializer。-->
<bean id="valueSerializer" class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" />
```
keySerializer:这个是对key的默认序列化器。默认值是StringSerializer。

valueSerializer:这个是对value的默认序列化器，默认值是取自DefaultSerializer的JdkSerializationRedisSerializer。

hashKeySerializer:对hash结构数据的hashkey序列化器，默认值是取自DefaultSerializer的JdkSerializationRedisSerializer。

hashValueSerializer：对hash结构数据的hashvalue序列化器，默认值是取自DefaultSerializer的JdkSerializationRedisSerializer

### RedisTemplate的事务
通过`private boolean enableTransactionSupport = false;` 可以看出RedisTemplate是默认事物关闭的。
在RedisTemplate源码中搜索该变量，发现
```java
if (enableTransactionSupport) {
	synchronization conn = RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
} else {
	conn = RedisConnectionUtils.getConnection(factory);
}
```
也就是如果设置参数enableTransactionSupport=true,则系统自动帮我们拿到了事务中绑定的连接。可以在一个方法的多次对Redis增删该查中，始终使用同一个连接。
同样的，在Spring中@Transactional 应该也是可以进行事物控制的。
