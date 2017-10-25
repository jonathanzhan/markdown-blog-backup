---
title: mysql字段替换
date: 2016-04-11 11:17:51
categories:
- 数据库
tags:
- mysql
description: "mysql字段替换"
---
replace的用法（替换某字段部分内容）
<!-- more -->
#### replace into
```sql
replace into table (id,name) values('1','aa'),('2','bb')
```
此语句的作用是向表table中插入两条记录。如果主键id为1或2不存在就相当于
insert into table (id,name) values('1','aa'),('2','bb')
如果存在相同的值则不会插入数据

#### replace(object,search,replace)
把object中出现search的全部替换为replace
select replace('www.163.com','w','Ww')--->WwWwWw.163.com
例：把表table中的name字段中的aa替换为bb
update table set name=replace(name,'aa','bb')

#### UPDATE更新一个字段中的的部分内容
现在有一条记录的字段是“abcdefg",现在我只想将该字段中的c改为C，update语句应该怎么写

update 表名 set 字段1 = replace(字段1,'c','C')
