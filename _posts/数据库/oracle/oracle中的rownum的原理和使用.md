---
title: oracle中的rownum的原理和使用
date: 2016-06-25 18:17:20
tags:
- oracle
categories:
- 数据库
---

# oracle中的rownum的原理和使用

闲着无事，整理和研究了下oracle中关于rownum的使用。

对于oracle中的rownum,不支持>,>=,between .. and,=。只能用<,& lt;=,!=。但并不是提示sql语法错误，而是查询出来的数据显示不准确。

<!-- more -->
### 举例说明 
```sql
select id,name,rownum from tb_test where rownum<5;
select id,name,rownum from tb_test where rownum>=5;
```
其中第一条sql能够查询出希望的数据，而第二条sql，应该是没有记录的吧。

### rownum的说明
** rownum是对结果集增加的一个伪列,即先查到结果集之后再加上去的一个列 （强调：先要有结果集）。直白点说rownum是对符合条件结果的序列号,并且永远是从1开始。所以查询的结果中不可能没有1,而有大于1的数据 **

ROWNUM是一个序列，是oracle数据库从数据文件或缓冲区中读取数据的顺序。它取得第一条记录则rownum值为1，第二条为2，依次类推。如 果你用>，>=，=，between……and这些条件，因为从缓冲区或数据文件中得到的第一条记录的rownum为1，则被删除，接着取下 条，可是它的rownum还是1，又被删除，依次类推，便没有了数据。
### rownum的几种常用使用
1. ` select id,name from tb_test where rownum!=10 `
为何返回的是9条数据，与`select id,name from tb_test where rownum<10`返回结果一样
因为是在查询到结果集后，显示完第9条记录后，之后的记录全部都不等于10，或者 >=10，所以只显示前面9条记录。
也可以这样理解，`rownum` 为9后的记录的rownum为10，因条件为 !=10，所以去掉，其后记录补上，`rownum`又是10，也去掉，如果下去也就只会显示前面9条记录了
2. 为什么 `rownum >1` 时查不到一条记录，而 `rownum >0` 或 `rownum >=1` 却总显示所有的记录
 因为 `rownum `是在查询到的结果集后加上去的，它总是从1开始
3. 为什么` between 1 and 10` 或者 `between 0 and 10` 能查到结果，而用 `between 2 and 10` 却得不到结果,原因同上一样，因为 `rownum` 总是从 1 开始
4. 无ORDER BY排序的分页写法
```sql
select * from (
select id,name,rownum as rn from tb_test where rownum<=10
) 
where rn>=5
```
5. 有ORDER BY排序的写法。(效率最高)
```sql
select * from 
(
  select a.*,rownum as rn from 
  (
    select id,name from tb_test order by id desc 
  ) a where rownum<=10
) b where b.rn>=5
```

