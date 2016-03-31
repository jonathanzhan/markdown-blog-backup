---
title: fmt标签(formatDate)详解
date: 2016-03-31 16:26:44
categories:
- java
tags:
- jstl
- java
description: "jstl标签中的fmt标签中的formatDate方法的使用说明"
---

##### 标签引用

```
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
```

##### 参数说明

1. value：需要格式化的数据，类型为java.util.Date

2. type:需要格式化的样式，参见下面

3. dateStyle: 具体样式，比type更具体的描述,可以不写，具体参看下面例子

4. pattern：输出的格式

##### 格式模式

* d&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;月中的某一天。一位数的日期没有前导零。
* dd&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;月中的某一天。一位数的日期有一个前导零。
* ddd&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;周中某天的缩写名称，在   AbbreviatedDayNames   中定义。
* dddd&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;周中某天的完整名称，在   DayNames   中定义。
* M&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;月份数字。一位数的月份没有前导零。
* MM&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;月份数字。一位数的月份有一个前导零。
* MMM&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;月份的缩写名称，在AbbreviatedMonthNames   中定义。
* MMMM&nbsp;&nbsp;月份的完整名称，在MonthNames中定义。
* y&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不包含纪元的年份。如果不包含纪元的年份小于   10，则显示不具有前导零的年份。
* yy&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不包含纪元的年份。如果不包含纪元的年份小于   10，则显示具有前导零的年份。
* yyyy&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;包括纪元的四位数的年份。
* gg&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;时期或纪元。如果要设置格式的日期不具有关联的时期或纪元字符串，则忽略该模式。
* h&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;12小时制的小时。一位数的小时数没有前导零。从1到12，分上下午 范围：01：00 AM~12:59AM
* hh&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;12小时制的小时。一位数的小时数有前导零。
* H&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;24 小时制的小时。一位数的小时数没有前导零。从0到23，范围：00：00 AM~23:59AM
* HH&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;24 小时制的小时。一位数的小时数有前导零。
* m&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分钟。一位数的分钟数没有前导零。
* mm&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分钟。一位数的分钟数有一个前导零。
* s&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;秒。一位数的秒数没有前导零。
* ss&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;秒。一位数的秒数有一个前导零。


```
<fmt:formatDate value="<%=new Date() %>" type="both"/>
//2014-8-20 11:21:16

<fmt:formatDate value="<%=new Date() %>" type="date"/>
//2014-8-5 注意如果是小于10的话，显示的是1位数字

<fmt:formatDate value="<%=new Date() %>" type="time"/>
//8:30:38

<fmt:formatDate value="<%=new Date() %>" type="date" dateStyle="default"/>
//2014-8-5

<fmt:formatDate value="<%=new Date() %>" type="date" dateStyle="short"/>
//14-8-5

<fmt:formatDate value="<%=new Date() %>" type="date" dateStyle="long"/>
//2014年8月5日

<fmt:formatDate value="<%=new Date() %>" type="date" dateStyle="full"/>
//2014年8月5日 星期二

<fmt:formatDate value="<%=new Date() %>" type="date" dateStyle="medium"/>
//2014-8-5

<fmt:formatDate value="<%=new Date() %>" type="time" timeStyle="default"/>
//8:36:49

<fmt:formatDate value="<%=new Date() %>" type="time" timeStyle="short"/>
//上午8:36

<fmt:formatDate value="<%=new Date() %>" type="time" timeStyle="medium"/>
//8:36:49

<fmt:formatDate value="<%=new Date() %>" type="time" timeStyle="long"/>
//上午08时36分49秒

<fmt:formatDate value="<%=new Date() %>" type="time" timeStyle="full"/>
//上午08时36分49秒 CST

<fmt:formatDate value="<%=new Date() %>" type="both" pattern="EEEE, MMMM d, yyyy HH:mm:ss Z"/>
星期二, 八月 5, 2014 08:39:02 +0800
```
