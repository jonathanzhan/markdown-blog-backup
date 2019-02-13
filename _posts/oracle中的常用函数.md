---
title:  oracle中的常用函数
date: 2016-05-05 17:43:43
tags:
- oracle
categories:
- 数据库
toc: true
---

oracle中常用函数的介绍,持续更新中
<!-- more -->
### decode
* 含义
decode(条件,值1,返回值1,值2,返回值2,...值n,返回值n,缺省值)
if(条件==值1) return 返回值1
else if(条件==值2) return 返回值2
。。。。
else return 缺省值
* 举例
```sql
select decode(1+1,0,1,2,3,4,5,-1) from dual
```
如果1+1=0，则返回1，=2，返回3，=4，返回5，全部不等，返回-1

排序:
某个表，两个字段(name，score)，分别存课程名，分数。
需要根据课程名排序。

```sql
select * from t order by decode(name, '语文', 1, '数学', 2,3,'英语',4)
```
使用decode，可以设定想要的排序规则

### sign
* 含义
sign()函数根据某个值是0、正数还是负数，分别返回0、1、-1
* 举例
```sql
select sign(1-1) from dual
```

----------
未完待续
