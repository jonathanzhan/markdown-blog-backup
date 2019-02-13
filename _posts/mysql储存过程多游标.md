---
title: mysql储存过程多游标
date: 2016-03-31 16:25:14
categories:
- 数据库
tags:
- mysql
---
对于mysql，储存过程使用的不是很多，往往过了一段时间就忘记如何使用，本文主要对mysql存储过程的一些信息做记录
<!-- more -->
下面是一个简单存储过程的例子:

```sql
drop procedure IF EXISTS test_proc;
DELIMITER $$
create procedure test_proc()
begin
    -- 声明一个标志done， 用来判断游标是否遍历完成
    DECLARE done INT DEFAULT 0;
    -- 声明一个变量，用来存放从游标中提取的数据
    -- 特别注意这里的名字不能与由游标中使用的列明相同，否则得到的数据都是NULL
    DECLARE tname varchar(50) DEFAULT NULL;
    DECLARE tpass varchar(50) DEFAULT NULL;
    -- 声明游标对应的 SQL 语句
    DECLARE cur CURSOR FOR
        select name, password from netingcn_proc_test;

    -- 在游标循环到最后会将 done 设置为 1
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    -- 执行查询
    open cur;
    -- 遍历游标每一行
    REPEAT
        -- 把一行的信息存放在对应的变量中
        FETCH cur INTO tname, tpass;
        if not done then
            -- 这里就可以使用 tname， tpass 对应的信息了
            select tname, tpass;
        end if;
    UNTIL done END REPEAT;
    CLOSE cur;
END$$
delimiter ;

-- 执行存储过程
call test_proc();
```
> 需要注意的是变量的声明、游标的声明和HANDLER声明的顺序不能搞错，必须是先声明变量，再申明游标，最后声明HANDLER。

上述存储过程的例子中只使用了一个游标，那么如果要使用两个或者更多游标怎么办，其实很简单，可以这么说，一个怎么用两个就是怎么用的。例子如下：

```sql
drop procedure IF EXISTS test_proc_1;
delimiter $$
create procedure test_proc_1()
begin
    DECLARE done INT DEFAULT 0;
    DECLARE tid int(11) DEFAULT 0;
    DECLARE tname varchar(50) DEFAULT NULL;
    DECLARE tpass varchar(50) DEFAULT NULL;

    DECLARE cur_1 CURSOR FOR
        select name, password from netingcn_proc_test;

    DECLARE cur_2 CURSOR FOR
        select id, name from netingcn_proc_test;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    open cur_1;
    REPEAT
        FETCH cur_1 INTO tname, tpass;
        if not done then
            select tname, tpass;
        end if;
    UNTIL done END REPEAT;
    CLOSE cur_1;

    -- 注意这里，一定要重置done的值为 0
    set done = 0;

    open cur_2;
    REPEAT
        FETCH cur_2 INTO tid, tname;
        if not done then
            select tid, tname;
        end if;
    UNTIL done END REPEAT;
    CLOSE cur_2;
end$$
delimiter ;

call test_proc_1();
```

上述代码和第一个例子中基本一样，就是多了一个游标声明和遍历游标。这里需要注意的是，在遍历第二个游标前使用了set done = 0，因为当第一个游标遍历玩后其值被handler设置为1了，如果不用set把它设置为 0 ，那么第二个游标就不会遍历了。当然好习惯是在每个打开游标的操作前都用该语句，确保游标能真正遍历。当然还可以使用begin语句块嵌套的方式来处理多个游标,例如：

```sql
drop procedure IF EXISTS test_proc_2;
delimiter $$
create procedure test_proc_2()
begin
    DECLARE done INT DEFAULT 0;
    DECLARE tname varchar(50) DEFAULT NULL;
    DECLARE tpass varchar(50) DEFAULT NULL;

    DECLARE cur_1 CURSOR FOR
        select name, password from netingcn_proc_test;

    DECLARE cur_2 CURSOR FOR
        select id, name from netingcn_proc_test;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    open cur_1;
    REPEAT
        FETCH cur_1 INTO tname, tpass;
        if not done then
            select tname, tpass;
        end if;
    UNTIL done END REPEAT;
    CLOSE cur_1;

    begin
        DECLARE done INT DEFAULT 0;
        DECLARE tid int(11) DEFAULT 0;
        DECLARE tname varchar(50) DEFAULT NULL;

        DECLARE cur_2 CURSOR FOR
            select id, name from netingcn_proc_test;

        DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

        open cur_2;
        REPEAT
            FETCH cur_2 INTO tid, tname;
            if not done then
                select tid, tname;
            end if;
        UNTIL done END REPEAT;
        CLOSE cur_2;
    end;
end$$
delimiter ;
call test_proc_2();
```
