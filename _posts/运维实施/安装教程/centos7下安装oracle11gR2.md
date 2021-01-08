---
title: centos7下安装oracle11gR2
date: 2016-08-30 17:35:58
tags:
- linux
- oracle
categories:
- 运维实施
toc: true
---
本文介绍在centos7下安装oracle11g的操作步骤
<!-- more -->

## Centos7安装oracle11gR2说明

### 环境准备
安装包:

1. CentOS-7-x86_64-DVD
2. linux.x64_11gR2_database_1of2.zip
3. linux.x64_11gR2_database_2of2.zip

本教程是在VMware下安装的，注意设置内存的时候，不要设置动态内存。
### 安装Oracle前准备

#### 创建运行oracle数据库的系统用户和用户组
```Bash
[jonathan@localhost ~]$ su root　　#切换到root
Password:
[root@localhost]# groupadd oinstall　　#创建用户组oinstall
[root@localhost]# groupadd dba　　#创建用户组dba
[root@localhost]# useradd -g oinstall -g dba -m oracle　　#创建oracle用户，并加入到oinstall和dba用户组
[root@localhost]# passwd oracle　　#设置用户oracle的登陆密码，不设置密码，在CentOS的图形登陆界面没法登陆
Changing password for user oracle.
New password: 　　# 密码
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 　　# 确认密码
passwd: all authentication tokens updated successfully.
[root@localhost]# id oracle # 查看新建的oracle用户
uid=1001(oracle) gid=1002(dba) groups=1002(dba)

```
为啥要创建oinstall用户组及dba组？参考[http://www.oracle.com/technetwork/cn/articles/hunter-rac11gr2-iscsi-2-092412-zhs.html#13](http://www.oracle.com/technetwork/cn/articles/hunter-rac11gr2-iscsi-2-092412-zhs.html#13)

#### 创建oracle数据库安装目录
```Bash
[jonathan@localhost ~]$ su root
Password:
[root@localhost]# mkdir -p /data/oracle　　#oracle数据库安装目录
[root@localhost]# mkdir -p /data/oraInventory　　#oracle数据库配置文件目录
[root@localhost]# mkdir -p /data/database　　#oracle数据库软件包解压目录
[root@localhost]# cd /data
[root@localhost data]# ls　　#创建完毕检查一下（强迫症）
database  oracle  oraInventory
[root@localhost data]# chown -R oracle:oinstall /data/oracle　　#设置目录所有者为oinstall用户组的oracle用户
[root@localhost data]# chown -R oracle:oinstall /data/oraInventory
[root@localhost data]# chown -R oracle:oinstall /data/database
[root@localhost data]#
```

#### 修改OS系统标识
oracle默认不支持CentOS系统安装，Oracle Database 11g Release 2 的 OS要求参考：[https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#LADBI1106](https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#LADBI1106)

修改文件 /etc/RedHat-release

```shell
[root@localhost data]# cat /proc/version
Linux version 3.10.0-327.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.3 20140911 (Red Hat 4.8.3-9) (GCC) ) #1 SMP Thu Nov 19 22:10:57 UTC 2015
[root@localhost data]# cat /etc/redhat-release　　
CentOS Linux release 7.1.1503 (Core)
[root@localhost data]# vi /etc/redhat-release
[root@localhost data]# cat /etc/redhat-release
redhat-7
[root@localhost data]#
```

#### 安装oracle数据库所需要的软件包
Oracle Database Package Requirements for Linux x86-64 如下：（参考：[https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#BABCFJFG](https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#BABCFJFG)）

```shell
yum install gcc* gcc-* gcc-c++-* glibc-devel-* glibc-headers-* compat-libstdc* libstdc* elfutils-libelf-devel* libaio-devel* sysstat* unixODBC-* pdksh-*
```

根据具体情况去安装,上面只是提供了一个大概的内容，不是很全

#### 关闭防火墙

CentOS 7.2默认使用的是firewall作为防火墙

```
[root@localhost /]# systemctl status firewalld.service　　#查看防火墙状态，运行中
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2016-04-07 18:54:29 PDT; 2h 20min ago
 Main PID: 802 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─802 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

Apr 07 18:54:25 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Apr 07 18:54:29 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
[root@localhost /]# systemctl stop firewalld.service　　#关闭防火墙
[root@localhost /]# systemctl status firewalld.service　　#再次查看防火墙状态，发现已关闭
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Thu 2016-04-07 21:15:34 PDT; 9s ago
 Main PID: 802 (code=exited, status=0/SUCCESS)

Apr 07 18:54:25 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Apr 07 18:54:29 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
Apr 07 21:15:33 localhost systemd[1]: Stopping firewalld - dynamic firewall daemon...
Apr 07 21:15:34 localhost systemd[1]: Stopped firewalld - dynamic firewall daemon.
[root@localhost /]# systemctl disable firewalld.service　　#禁止使用防火墙（重启也是禁止的）
Removed symlink /etc/systemd/system/dbus-org.Fedoraproject.FirewallD1.service.
Removed symlink /etc/systemd/system/basic.target.wants/firewalld.service.
[root@localhost /]#
```

#### 关闭selinux（需重启生效）

[root@localhost /]# vi /etc/selinux/config
[root@localhost /]# cat /etc/selinux/config


将 SELINUX=disabled   #此处修改为disabled

#### 修改内核参数

[root@localhost /]# vi /etc/sysctl.conf

在最下面添加以下内容:

```
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
fs.file-max = 6815744 #设置最大打开文件数
fs.aio-max-nr = 1048576
kernel.shmall = 2097152 #共享内存的总量，8G内存设置：2097152*4k/1024/1024
kernel.shmmax = 2147483648 #最大共享内存的段大小
kernel.shmmni = 4096 #整个系统共享内存端的最大数
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500 #可使用的IPv4端口范围
net.core.rmem_default = 262144
net.core.rmem_max= 4194304
net.core.wmem_default= 262144
net.core.wmem_max= 1048576
```

使配置参数生效

```
[root@localhost /]# sysctl -p
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
sysctl: setting key "fs.file-max": Invalid argument
fs.file-max = 6815744 #设置最大打开文件数
fs.aio-max-nr = 1048576
sysctl: setting key "kernel.shmall": Invalid argument
kernel.shmall = 2097152 #共享内存的总量，8G内存设置：2097152*4k/1024/1024
sysctl: setting key "kernel.shmmax": Invalid argument
kernel.shmmax = 2147483648 #最大共享内存的段大小
sysctl: setting key "kernel.shmmni": Invalid argument
kernel.shmmni = 4096 #整个系统共享内存端的最大数
kernel.sem = 250 32000 100 128
sysctl: setting key "net.ipv4.ip_local_port_range": Invalid argument
net.ipv4.ip_local_port_range = 9000 65500 #可使用的IPv4端口范围
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
[root@localhost /]#
```

#### 对oracle用户设置限制，提高软件运行性能

[root@localhost /]# vi /etc/security/limits.conf

在最下面部分添加内容(粗体为添加的内容)


@student        -       maxlogins       4

** oracle soft nproc 2047 **

** oracle hard nproc 16384 **

** oracle soft nofile 1024 **

** oracle hard nofile 65536 **


End of file


#### 配置用户的环境变量

[root@localhost /]# vi /home/oracle/.bash_profile
添加以下内容:

```
export ORACLE_BASE=/data/oracle #oracle数据库安装目录
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1 #oracle数据库路径
export ORACLE_SID=orcl #oracle启动数据库实例名
export ORACLE_TERM=xterm #xterm窗口模式安装
export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH #添加系统环境变量
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib #添加系统环境变量
export LANG=en_US #防止安装过程出现乱码
export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK  #设置Oracle客户端字符集
```
生效

[root@localhost /]# source /home/oracle/.bash_profile

#### 解压安装包

```
[oracle@localhost /]$ cd /usr/local/src　　#进入/usr/local/src目录
[oracle@localhost src]$ ls
linux.x64_11gR2_database_1of2.zip  linux.x64_11gR2_database_2of2.zip
[oracle@localhost src]$ unzip linux.x64_11gR2_database_1of2.zip -d /data/database/　　#解压
(省略...)
[oracle@localhost src]$ unzip linux.x64_11gR2_database_2of2.zip -d /data/database/　　#解压
(省略...)
[oracle@localhost src]$ su root
Password:
[root@localhost src]# chown -R oracle:oinstall /data/database/database/
```

### oracle安装
#### 登录oracle用户
通过图形界面登录oracle用户

#### 启动oralce安装
到/data/database/database/目录下，执行./runInstaller

#### 按照步骤进行安装
#### 安装中出现的问题

安装过程中连接库时,在进度68%时会出现两个错误：

第一个：
/lib64/libstdc++.so中memcpy@GLIBC_2.4找不到。
问题：glibc是2.17的库，连接找的是2.14的库。
解决办法：改成静态链接。
查看 /usr/lib64/libc.a是否存在。
修改oracle安装目录下：$ORACLE_HOME/ctx/lib/ins_ctx.mk
ctxhx: $(CTXHXOBJ)
        $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK)
修改为：
ctxhx: $(CTXHXOBJ)
        -static $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK) /usr/lib64/libc.a
然后点击retry通过。

第二个：
问题：undefinied reference symbol'B_DestroyKeyObject'，查看日志，实际就是没有找到nnz11这个库。
解决办法：
修改$ORACLE_HOME/sysman/lib/ins_emagent
$(MK_EMAGENT_NMECTL)
修改为：
$(MK_EMAGENT_NMECTL) -lnnz11
然后点击retry通过。


