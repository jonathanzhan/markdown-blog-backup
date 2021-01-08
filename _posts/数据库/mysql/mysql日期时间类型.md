---
title: mysql日期时间类型
date: 2020-05-23 16:04:35
tags:
- mysql
categories:
- 数据库
---
最近在写mysql语句时,总是想不起来相关的SQL函数,或者容易与Oracle,Sql Server等函数混淆,所以下定决心自己整理下
第一步,先整理下mysql中的相关日期时间类型
<!-- more -->
### 几种类型比较

类型|大小(bytes)|范围|格式|用途|零值表示
:-:|-|-|-|-|-
DATE|4 bytes|1000-01-01/9999-12-31|YYYY-MM-DD|日期值|0000-00-00
TIME|4 bytes|'-838:59:59'/'838:59:59'|HH:MM:SS| 时间或持续时间(HHH:MM:SS)|00:00:00
DATETIME| 8 bytes| 1000-01-01 00:00:00/9999-12-31 23:59:59| YYYY-MM-DD HH:MM:SS	| 混合日期和时间值| 0000-00-00 00:00:00
TIMESTAMP| 4 bytes | 1970-01-01 00:00:00/2038某个时间(北京时间 2038-1-19 11:14:07) | YYYYMMDD HHMMSS |混合日期和时间值，时间戳 |00000000000000
YEAR| 1 byets| 1901/2155 |YYYY| 年份值 |0000

以上的零值格式在启用`NO_ZERO_DATE` SQL模式下，这些值会产生警告

当使用日期和时间类型时应记住以下几点：

* MySQL以标准输出格式检索给定日期或时间类型的值，但它尽力解释你指定的各种输入值格式(例如，当你指定一个分配给或与日期或时间类型进行比较的值时)
* 包含两位年值的日期转换分为:70-99范围的年值转换为1970-1999。00-69范围的年值转换为2000-2069。
* 尽管MySQL尝试解释几种格式的值，日期总是以年-月-日顺序(例如，'98-09-04')，而不是其它地方常用的月-日-年或日-月-年顺序(例如，'09-04-98'，'04-09-98')
* 如果值用于数值上下文中，MySQL自动将日期或时间类型的值转换为数字，反之亦然
* 当 MySQL遇到一个日期或时间类型的超出范围或对于该类型不合法的值时，它将该值转换为该类的“零”值。一个例外是超出范围的TIME值被裁剪到TIME范围的相应端点

### DATE

支持的范围为'1000-01-01'到'9999-12-31'。MySQL以'YYYY-MM-DD'格式显示DATE值，但允许使用字符串或数字为DATE列分配值。

### TIME

MySQL以'HH:MM:SS'格式检索和显示TIME值(或对于大的小时值采用'HHH:MM:SS'格式)。TIME值的范围可以从'-838:59:59'到'838:59:59'。小时部分会因此大的原因是TIME类型不仅可以用于表示一天的时间(必须小于24小时)，还可能为某个事件过去的时间或两个事件之间的时间间隔(可以大于24小时，或者甚至为负)。

可以用以下格式指定TIME值

* 'D HH:MM:SS.fraction'格式的字符串。还可以使用下面任何一种“非严格”语法：'HH:MM:SS.fraction'、'HH:MM:SS'、'HH:MM'、'D HH:MM:SS'、'D HH:MM'、'D HH'或'SS'。这里D表示日，可以取0到34之间的值。请注意MySQL还不保存分数。
* 'HHMMSS'格式的没有间割符的字符串，假定是有意义的时间。例如，'101112'被理解为'10:11:12'，但'109712'是不合法的(它有一个没有意义的分钟部分)，将变为'00:00:00'。
* HHMMSS格式的数值，假定是有意义的时间。例如，101112被理解为'10:11:12'。下面格式也可以理解：SS、MMSS、HHMMSS、HHMMSS.fraction。请注意MySQL还不保存分数
* 函数返回的结果，其值适合TIME上下文，例如CURRENT_TIME
* 对于指定为包括时间部分间割符的字符串的TIME值，如果时、分或者秒值小于10，则不需要指定两位数。'8:3:2'与'08:03:02'相同。
* 为TIME列分配简写值时应注意。没有冒号，MySQL解释值时假定最右边的两位表示秒。(MySQL解释TIME值为过去的时间而不是当天的时间）。例如，你可能认为'1112'和1112表示'11:12:00'(11点过12分)，但MySQL将它们解释为'00:11:12'(11分，12 秒)。同样，'12'和12 被解释为 '00:00:12'。相反，TIME值中使用冒号则肯定被看作当天的时间。也就是说，'11:12'表示'11:12:00'，而不是'00:11:12'。
* 超出TIME范围但合法的值被裁为范围最接近的端点。例如，'-850:00:00'和'850:00:00'被转换为'-838:59:59'和'838:59:59'。

### DATETIME

 `DATETIME` 用于表示 年月日 时分秒，是 `DATE` 和 `TIME` 的组合，并且记录的年份（见上表）比较长久。如果实际应用中有这样的需求，就可以使用 `DATETIME` 类型。

### TIMESTAMP

* `TIMESTAMP` 用于表示 年月日 时分秒，但是记录的年份（见上表）比较短暂。
* `TIMESTAMP` 和时区相关，更能反映当前时间。当插入日期时，会先转换为本地时区后再存放；当查询日期时，会将日期转换为本地时区后再显示。所以不同时区的人看到的同一时间是不一样的。
* 表中的第一个 `TIMESTAMP` 列自动设置为系统时间（`CURRENT_TIMESTAMP`,如果表中有第二个 `TIMESTAMP`列，则默认值设置为0000-00-00 00:00:00。
* TIMESTAMP 的属性受 Mysql 版本和服务器 SQLMode 的影响较大

如果记录的日期需要让不同时区的人使用，最好使用 TIMESTAMP

`TIMESTAMP`列的显示格式与`DATETIME`列相同。换句话说，显示宽度固定在19字符，并且格式为YYYY-MM-DD HH:MM:SS。

TIMESTAMP初始化规则如下:

* 表内的任何一个`TIMESTAMP`列可以设置为自动初始化为当前时间戳和/或更新
* 用`DEFAULT CURRENT_TIMESTAMP`和`ON UPDATE CURRENT_TIMESTAMP`子句，列为默认值使用当前的时间戳，并且自动更新。
* 用`DEFAULT CURRENT_TIMESTAMP`子句不用ON UPDATE子句，列为默认值使用当前的时间戳但是不自动更新。
* 在DEFAULT和ON UPDATE子句中可以使用CURRENT_TIMESTAMP、CURRENT_TIMESTAMP()或者NOW()。它们均具有相同的效果.

### YEAR

 `YEAR` 用于表示 年份，`YEAR` 有 2 位（最好使用4位）和 4 位格式的年。 默认是4位。如果实际应用只保存年份，那么用 1 bytes 保存 `YEAR` 类型完全可以。不但能够节约存储空间，还能提高表的操作效率。

 SELECT * FROM cms_book_statistics WHERE substring(Convert(char(10),update_time ,112),1,8)='20170927' 
 
SELECT * FROM cms_book_statistics WHERE update_time between '2017-09-27 00:00:00' and '2017-09-27 23:59:59' 
 
SELECT * FROM cms_book_statistics WHERE year(update_time ) = 2017 and month(update_time )= 09 and day(update_time ) = 27
 
SELECT * FROM cms_book_statistics WHERE update_time > '2017-09-27' and update_time < '2017-09-28'
 
SELECT * FROM cms_book_statistics WHERE ( datediff ( update_time , '2017-09-27' ) = 0 )



#今天
 
select * from 表名 where to_days(时间字段名) = to_days(now());
 
#昨天
 
SELECT * FROM 表名 WHERE DATEDIFF(字段,NOW())=-1;
 
#本周
 
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now());
 
#上周
 
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now())-1;
 
#近7天
 
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(时间字段名)
 
#近30天
 
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(时间字段名)
 
#本月
 
SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' )
 
#上一个月
 
SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1
 
#本季度
 
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(now());
 
#上季度
 
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));
 
#本年
 
select * from `ht_invoice_information` where YEAR(create_date)=YEAR(NOW());
 
#去年
 
select * from `ht_invoice_information` where year(create_date)=year(date_sub(now(),interval 1 year));

mysql 查询指定日期格式，使用 DATE_FORMAT(date,format) 函数
select DATE_FORMAT(create_date,"%Y-%m-%d %H:%i%s") AS create_date from t_sys

%M 月名字(January……December)
%W 星期名字(Sunday……Saturday)
%D 有英语前缀的月份的日期(1st, 2nd, 3rd, 等等。）
%Y 年, 数字, 4 位
%y 年, 数字, 2 位
%a 缩写的星期名字(Sun……Sat)
%d 月份中的天数, 数字(00……31)
%e 月份中的天数, 数字(0……31)
%m 月, 数字(01……12)
%c 月, 数字(1……12)
%b 缩写的月份名字(Jan……Dec)
%j 一年中的天数(001……366)
%H 小时(00……23)
%k 小时(0……23)
%h 小时(01……12)
%I 小时(01……12)
%l 小时(1……12)
%i 分钟, 数字(00……59)
%r 时间,12 小时(hh:mm:ss [AP]M)
%T 时间,24 小时(hh:mm:ss)
%S 秒(00……59)
%s 秒(00……59)
%p AM或PM
%w 一个星期中的天数(0=Sunday ……6=Saturday ）
%U 星期(0……52), 这里星期天是星期的第一天
%u 星期(0……52), 这里星期一是星期的第一天
%% 一个文字“%”
