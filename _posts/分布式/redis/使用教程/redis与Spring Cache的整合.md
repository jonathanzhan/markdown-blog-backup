---
title: redis与Spring Cache的整合
date: 2016-07-05 17:41:25
tags:
- redis
categories:
- java
toc: true
---
本文主要介绍redis与Spring Cache的整合教程
<!-- more -->
## redis安装
此步骤略过，网上教程很多。

## reds与Spring Cache整合

###主要依赖的jar包有:

```xml
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-redis</artifactId>
	<version>${spring-data-redis.version}</version>
</dependency>

```
### Spring配置

```xml
<!-- Jedis线程 -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
	<property name="maxIdle" value="${redis.maxIdle}" />
	<property name="minIdle" value="${redis.minIdle}" />
	<property name="maxTotal" value="${redis.maxTotal}" />
	<property name="testOnBorrow" value="true" />
</bean>
<bean id="jedisShardInfo" class="redis.clients.jedis.JedisShardInfo">
	<constructor-arg index="0" value="${redis.host}" />
	<constructor-arg index="1" value="${redis.port}" type="int" />
</bean>
<!-- Redis连接 -->
<bean id="jedisConnectionFactory"
	class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="shardInfo" ref="jedisShardInfo"/>
		<property name="poolConfig" ref="jedisPoolConfig"/>
</bean>
<!-- 缓存序列化方式 -->
<bean id="keySerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />
<bean id="valueSerializer" class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" />
<!-- 缓存 -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
	<property name="connectionFactory" ref="jedisConnectionFactory" />
	<property name="keySerializer" ref="keySerializer" />
	<property name="valueSerializer" ref="valueSerializer" />
	<property name="hashKeySerializer" ref="keySerializer" />
	<property name="hashValueSerializer" ref="valueSerializer" />
</bean>
<bean id="redisCacheManager" class="org.springframework.data.redis.cache.RedisCacheManager">
	<constructor-arg index="0" ref="redisTemplate" />
	<property name="defaultExpiration" value="${redis.expiration}" />
</bean>
```

完成以上工作后，就可以在service方法中使用@Cacheable,@CachePut,@CacheEvict注解了
## 缓存名词的介绍
### 缓存命中率
即从缓存中读取数据的次数 与 总读取次数的比率，命中率越高越好：
命中率 = 从缓存中读取次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])
Miss率 = 没有从缓存中读取的次数 / (总读取次数[从缓存中读取次数 + 从慢速设备上读取的次数])
 
这是一个非常重要的监控指标，如果做缓存一定要健康这个指标来看缓存是否工作良好；
### 缓存策略
Eviction policy
移除策略，即如果缓存满了，从缓存中移除数据的策略；常见的有LFU、LRU、FIFO：
FIFO（First In First Out）：先进先出算法，即先放入缓存的先被移除；
LRU（Least Recently Used）：最久未使用算法，使用时间距离现在最久的那个被移除；
LFU（Least Frequently Used）：最近最少使用算法，一定时间段内使用次数（频率）最少的那个被移除；
 
TTL（Time To Live ）
存活期，即从缓存中创建时间点开始直到它到期的一个时间段（不管在这个时间段内有没有访问都将过期）
 
TTI（Time To Idle）
空闲期，即一个数据多久没被访问将从缓存中移除的时间。
 
