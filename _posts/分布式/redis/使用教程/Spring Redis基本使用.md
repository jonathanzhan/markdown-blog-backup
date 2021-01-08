---
title: Spring Redis基本使用
date: 2016-07-08 17:16:42
tags:
- redis
categories:
- java
toc: true
---
本文主要讲解通过Spring data redis(SDR)进行Spring与redis的整合使用过程以及redisTemplate的简单使用
<!-- more -->

## 依赖jar包的引入

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.7.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.8.1</version>
</dependency>
```
## Spring的配置

```xml
<!-- Jedis连接池的配置对象 -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
	<!--最大空闲数-->
	<property name="maxIdle" value="${redis.maxIdle}" />
	<!--最小空闲数-->
	<property name="minIdle" value="${redis.minIdle}" />
	<!--最大连接数-->
	<property name="maxTotal" value="${redis.maxTotal}" />
	<property name="testOnBorrow" value="true" />
	<!--最大建立连接等待时间-->
	<property name="maxWaitMillis" value="${redis.maxWaitMillis}"/>
</bean>

<!--jedis服务器信息-->
<bean id="jedisShardInfo" class="redis.clients.jedis.JedisShardInfo">
	<constructor-arg index="0" value="${redis.host}" />
	<constructor-arg index="1" value="${redis.port}" type="int" />
	<constructor-arg index="2" value="${redis.timeout}" type="int"/>
</bean>

<!--jedis连接池-->
<bean id="shardedJedisPool" class="redis.clients.jedis.ShardedJedisPool">
	<constructor-arg index="0" ref="jedisPoolConfig" />
	<constructor-arg index="1">
		<list>
			<ref bean="jedisShardInfo" />
		</list>
	</constructor-arg>
</bean>

<!-- Redis连接工厂 -->
<bean id="jedisConnectionFactory"
	class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="shardInfo" ref="jedisShardInfo"/>
		<property name="poolConfig" ref="jedisPoolConfig"/>
</bean>
<!-- 缓存序列化方式 -->
<!--对key的默认序列化器。默认值是StringSerializer-->
<bean id="keySerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />
<!--是对value的默认序列化器，默认值是取自DefaultSerializer的JdkSerializationRedisSerializer。-->
<bean id="valueSerializer" class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" />

<!-- redis操作模板,对Jedis进行的通用API操作 -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
	<property name="connectionFactory" ref="jedisConnectionFactory" />
	<property name="keySerializer" ref="keySerializer" />
	<property name="valueSerializer" ref="valueSerializer" />
	<property name="hashKeySerializer" ref="keySerializer" />
	<property name="hashValueSerializer" ref="valueSerializer" />
</bean>
```
redis的配置参数如下:

```Properties
#redis的服务器地址
redis.host=127.0.0.1
#redis的服务端口
redis.port=6379
#客户端超时时间单位是毫秒
redis.timeout=100000
#最大建立连接等待时间
redis.maxWaitMillis=1000
#最小空闲数
redis.minIdle=5
#最大空闲数
redis.maxIdle=20
#最大连接数
redis.maxTotal=100
#指明是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
redis.testOnBorrow=true
```

Spring 配置中，具体的配置说明参看注释。

## redisTemplate的基本使用
RedisTemplate提供了很多使用redis的api，而不需要自己来维护连接，事务。
同时提供了一系列的operation,比如valueOperation,HashOperation,ListOperation,SetOperation等，用来操作不同数据类型的Redis。
并且，RedisTemplate还提供了对应的*OperationsEditor，用来通过RedisTemplate直接注入对应的Operation。

下面是简单的一个单元测试类，后续会详细讲。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:Spring-config.xml" })
public class RedisTemplateTest {

	@Resource(name = "redisTemplate")
	private RedisTemplate<String, String> template;

	//RedisTemplate还提供了对应的*OperationsEditor，用来通过RedisTemplate直接注入对应的Operation。
	@Resource(name = "redisTemplate")
	private ValueOperations<String, Object> vOps;

	@Test
	public void test(){
		template.execute(new RedisCallback<Boolean>() {
			@Override
			public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
				byte [] key = "tempkey".getBytes();
				byte[] value = "tempvalue".getBytes();
				connection.set(key, value);
				return true;
			}
		});
	}

	/**
	 * valueOperation数据类型的操作
	 */
	@Test
	public void test1(){
		vOps.set("value", "code");
	}


}
```
这个是对String类型插入的两个测试。test方法中，使用了模版类提交回调(RedisCallBack)的方法来使用jedis connection操作数据。
test1使用valueOperation模板接口操作数据

