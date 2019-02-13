---
title: oracle中的rank() over,dense_rank(),row_number()的区别
date: 2016-12-25 18:14:32
tags:
- oracle
categories:
- 数据库
---

oracle中的rank() over,dense_rank(),row_number()的各自使用方法以及区别
<!-- more -->
### 语法
`rank() over([partition by col1] order by col2) `
`dense_rank() over([partition by col1] order by col2) `
`row_number() over([partition by col1] order by col2) `
其中`[partition by col1]`可省略。
三个分组函数都是按照col1分组内从1开始排序,区别在于:
**row_number() 是没有重复值的排序(即使两天记录相等也是不重复的) **
**dense_rank() 是连续排序，两个第二名仍然跟着第三名**
**rank()       是跳跃排序，两个第二名下来就是第四名 **
### 案例
数据准备:
```sql
create table t(
name varchar2(10),
score number(3)
);

insert into t(name,score) 
select '语文',60 from dual union all
select '语文',90 from dual union all
select '语文',80 from dual union all
select '语文',80 from dual union all
select '数学',67 from dual union all
select '数学',77 from dual union all
select '数学',78 from dual union all
select '数学',88 from dual union all
select '数学',99 from dual union all
select '语文',70 from dual;

select * from t;
```
![](/files/article/oracle/1.png)
----------

**row_number()**

```
select name,score,row_number() over(partition by name order by score) tt from t;
```
![row_number()](/files/article/oracle/2.png)


----------

**rank()**

```
select name,score,rank() over(partition by name order by score) tt from t;
```
![rank()](/files/article/oracle/3.png)

----------
**dense_rank()**

```
select name,score,dense_rank() over(partition by name order by score) tt from t;
```
![dense_rank()](/files/article/oracle/4.png)

