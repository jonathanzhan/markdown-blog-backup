---
title: RedisTemplate使用说明(常用接口方法)
date: 2016-07-08 18:47:02
tags:
- redis
categories:
- java
toc: true
---
在RedisTemplate中提供了几个常用的接口方法的使用，分别是:

```java
private ValueOperations<K, V> valueOps;
private ListOperations<K, V> listOps;
private SetOperations<K, V> setOps;
private ZSetOperations<K, V> zSetOps;
```
本文主要讲解几个接口的使用。
<!-- more -->

### RedisOperations
这个接口的实现类就是RedisTemplate，提供了一些对Redis命令的一些操作。

### ValueOperations
这个接口的实现类为:DefaultValueOperations.
在RedisTemplate中，已经提供了一个工厂方法:opsForValue()。这个方法会返回一个默认的操作类。另外，我们可以直接通过注解@Resource(name = "redisTemplate")来进行注入。

```java
//声明
@Resource(name = "redisTemplate")
private RedisTemplate<String, String> template;

//调用方法
template.opsForValue().set("key","value");
```

```java
//RedisTemplate还提供了对应的*OperationsEditor，用来通过RedisTemplate直接注入对应的Operation。
//声明
@Resource(name = "redisTemplate")
private ValueOperations<String, Object> vOps;

//调用方法
vOps.set("key","value");
```
除了可以通过template注入ValueOperations，还可以注入 上面的其他几种operations以及HashOperations

DefaultValueOperations提供了所有Redis字符串类型的操作api。比如set，get，incr等等。使用这些方法，可以方便的直接存储任意的java类型，而不需要自己去将存储的东西序列化以及反序列化

ListOperations,SetOperations,ZSetOperations除了提供的操作API不一样以外，其他的调用方法与DefaultValueOperations一致。

### HashOperations接口说明
这个接口并没有定义成员变量,但是直接提供了方法。
```java
public <HK, HV> HashOperations<K, HK, HV> opsForHash() {
	return new DefaultHashOperations<K, HK, HV>(this);
}
```

具体的调用如下：
#### 方法1
```java
//注入HashOperations对象
@Resource(name = "redisTemplate")
private HashOperations<String,String,Object> hashOps;

//具体调用
Map<String,String> map = new HashMap<String, String>();
map.put("value","code");
map.put("key","keyValue");
hashOps.putAll("hashOps",map);
```

#### 方法2

```java
//注入RedisTemplate对象
@Resource(name = "redisTemplate")
private RedisTemplate<String, String> template;


//具体调用
Map<String,String> map = new HashMap<String, String>();
map.put("value","code");
map.put("key","keyValue");
template.opsForHash().putAll("hashOps",map);
```
