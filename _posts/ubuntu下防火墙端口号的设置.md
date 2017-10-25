---
title: ubuntu下防火墙端口号的设置
date: 2016-08-25 17:50:07
tags:
- linux
categories:
- 运维实施
---

iptables是linux下的防火墙，同时也是服务名称。
### 关闭所有的INPUT FORWARD OUTPUT 只对某些端口开放。
```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP
```
再用命令 iptables -L -n 查看 是否设置好
还要使用 service iptables save 进行保存
### 打开某个特定的端口号
```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
```
再使用 iptables -L -n ,查看是否添加上去
最后别忘记了保存 对防火墙的设置
通过命令：service iptables save 进行保存

-A 参数就看成是添加一条 INPUT 的规则
-p 指定是什么协议 我们常用的tcp 协议，当然也有udp 例如53端口的DNS
而 --dport 就是目标端口 当数据从外部进入服务器为目标端口
反之 数据从服务器出去 则为数据源端口 使用 --sport
-j 就是指定是 ACCEPT 接收 或者 DROP 不接收
### 禁用某个IP访问
1台Linux服务器,2台windows xp 操作系统进行访问
Linux服务器ip 192.168.1.99
xp1 ip: 192.168.1.2
xp2 ip: 192.168.1.8

通过命令 iptables -A INPUT -p tcp -s 192.168.1.2 -j DROP禁用192.168.1.2的访问。
### 如何删除规则
首先我们要知道 这条规则的编号，每条规则都有一个编号
通过 ` iptables -L -n --line-number ` 可以显示规则和相对应的编号

num target     prot opt source               destination
1    DROP       tcp -- 0.0.0.0/0            0.0.0.0/0           tcp dpt:3306
2    DROP       tcp -- 0.0.0.0/0            0.0.0.0/0           tcp dpt:21
3    DROP       tcp -- 0.0.0.0/0            0.0.0.0/0           tcp dpt:80

多了num这一列， 这样我们就可以 看到刚才的规则对应的是 编号2
那么我们就可以进行删除了` iptables -D INPUT 2`删除INPUT链编号为2的规则。
再 ` iptables -L -n` 查看一下已经被清除了。
### Ubuntu下保存iptables规则并开机自动加载的方法
机器重启后，iptables中的配置信息会被清空。您可以将这些配置保存下来，让iptables在启动时自动加载，省得每次都得重新输入。iptables-save和iptables-restore 是用来保存和恢复设置的。

先将防火墙规则保存到/etc/iptables.up.rules文件中
` iptables-save > /etc/iptables.up.rules  #需要sudo su - root切换用户后执行，直接sudo cmd是不行的 `

然后修改脚本/etc/network/interfaces，使系统能自动应用这些规则
```Ini
auto eth0
iface eth0 inet dhcp
pre-up iptables-restore < /etc/iptables.up.rules
```
大多数人并不需要经常改变他们的防火墙规则，因此只要根据前面的介绍，建立起防火墙规则就可以了。但是如果您要经常修改防火墙规则，以使其更加完善，那么您可能希望系统在每次重启前将防火墙的设置保存下来。为此您可以在/etc/network/interfaces文件中添加一行：
post-down iptables-save > /etc/iptables.up.rules

