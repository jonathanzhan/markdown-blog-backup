---
title:  Java中处理时区的转换
date: 2016-06-14 17:44:53
tags:
- java基础
categories:
- java
toc: true
---
java中的时区处理
<!-- more -->

格林威治时间(21世纪的世界标准时间)转中国时间

```java
TimeZone timeZone = TimeZone.getTimeZone("GMT+8:00");
// dateTime是格林威治时间
long chineseMills = dateTime.getTime() + timeZone.getRawOffset();
Date chineseDateTime = new Date(chineseMills);
```

其他时区转中国时间

```java
TimeZone timeZone = TimeZone.getTimeZone("GMT+8:00");
TimeZone HollandTimeZone = TimeZone.getTimeZone("GMT+1:00");
// dateTime是荷兰时间
long chineseMills = dateTime.getTime() + timeZone.getRawOffset() - HollandTimeZone.getRawOffset();
Date chineseDateTime = new Date(chineseMills);
```
