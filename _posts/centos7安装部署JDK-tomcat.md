---
title: 'centos7安装部署JDK,tomcat'
date: 2017-07-12 17:12:10
tags:
- linux
categories:
- 运维实施
---

## Centos7安装JDK，tomcat以及部署简单程序的操作说明
### Centos7安装JDK

1. 在/usr/local下创建java文件夹
```bash
cd /usr/local 
mkdir java
cd java
```
2. 将JDK文件拷贝到/usr/local/java下
` cp jdk-8u121-linux-x64.tar.gz /usr/local/java ` 
` tar -zxvf jdk-8u121-linux-x64.tar.gz `
3. 设置环境变量
vi /etc/profile，在文件末尾添加环境变量
```bash
export JAVA_HOME=/usr/local/java/jdk1.8.0_121
export JRE_HOME=/usr/local/java/jdk1.8.0_121/jre
export PATH=$PATH:/usr/local/java/jdk1.8.0_121/bin
export CLASSPATH=./:/usr/local/java/jdk1.8.0_121/lib:/usr/local/java/jdk1.8.0_121/jre/lib
```
按 Esc 键、输入 :wq 回车，保持并退出。
` source /etc/profile ` 使配置生效
输入javac/java/java -version测试使用正常


### 安装tomcat

1. 拷贝tomcat至/usr/local下
```bash
cp apache-tomcat-7.0.75.tar.gz /usr/local 
tar -xvzf apache-tomcat-7.0.75.tar.gz
```
2. 设置tomcat编码格式
vi /apache-tomcat-7.0.75/conf/server.xml,增加URIEncoding参数配置
```xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="utf-8"/>
```
3. 常用命令
* 启动tomcat
` [root@localhost local]# ./apache-tomcat-7.0.75/bin/startup.sh `
* 关闭tomcat
` [root@localhost local]# ./apache-tomcat-7.0.75/bin/shutdown.sh `
* 查看日志
` tail -f catalina.out `
* 查看tomcat进程
` ps -ef |grep tomcat `
```
### mysql的配置
vi /etc/my.cnf文件
```Ini
[mysqld]
datadir=/home/data/mysql
socket=/home/data/mysql/mysql.sock
lower_case_table_names=1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character_set_server = utf8
default-storage-engine=INNODB
# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES,ONLY_FULL_GROUP_BY,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER
```

修改sql_mode为最后一行的配置。

然后重启mysql
```bash
/etc/init.d/mysqld stop
/etc/init.d/mysqld restart

## 或者
service mysql stop
service mysql start

```



