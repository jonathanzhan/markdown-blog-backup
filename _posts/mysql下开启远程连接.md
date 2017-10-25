---
title: mysql下开启远程连接
date: 2016-10-25 18:09:41
tags:
- mysql
- linux
categories:
- 数据库
---
mysql开始远程连接账号，有两步需要注意的：
1、确定服务器上的防火墙没有阻止 3306 端口
2、增加允许远程连接 MySQL 用户并授权。

### 防火墙端口号的设置
具体参看另外一篇文章，linux下防火墙端口号的设置。
telnet 192.168.1.111 3306
###  允许远程连接 MySQL 用户并授权
检查MySQL配置
如果开启了防火墙，telnet还是失败，通过netstat查看3306的端口状态：
netstat -apn|grep 3306
tcp6    0    0 127.0.0.1:3306    :::*    LISTEN        13524/mysqld*
注意，这说明3306被绑定到了本地。检查一下my.cnf的配置，这里可以配置绑定ip地址。
bind-address=addr或者注释掉bind-address
不配置或者IP配置为0.0.0.0，表示监听所有客户端连接。

```bash
mysql -u root -p
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
flush privileges;
```
最后，别忘了重启mysql使配置生效。
` /etc/init.d/mysql restart`


MySQL建用户的时候会指定一个host，默认是127.0.0.1/localhost，那么这个用户就只能本机访问，其它机器用这个用户帐号访问会提示没有权限，host改为%，表示允许所有机器访问。

```sql
mysql> use mysql;
mysql> select host,user from user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | root             |
| localhost | debian-sys-maint |
| localhost | git              |
| localhost | mysql.sys        |
+-----------+------------------+
4 rows in set (0.00 sec)

mysql> update user set host ='%' where user = 'root';
```
