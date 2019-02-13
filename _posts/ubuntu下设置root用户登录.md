---
title: ubuntu下设置root用户登录
date: 2016-08-18 17:51:03
tags:
- linux
categories:
- 运维实施
---
安装完成如需使用root身份登录，可打开终端输入以下命令：
```
#设置root密码
sudo passwd root
#切换到root用户
sudo -s
```
<!-- more -->
想要在登录界面使用root身份登录，可编辑/etc/lightdm/目录下的lightdm.conf文件，如没有此文件，直接创建

在terminal下输入
`
sudo gedit /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
`
输入以下内容
```
greeter-show-manual-login=true
```

使用reboot重启即可


如果重启后，读取/root/.profile时发现错误:mesg:ttyname

启动ubuntu，以root用户登陆，打开命令行终端

输入命令:#vi /root/.profile

找到.profile文件中的mesg n
将其替换成tty -s && mesg n
重启ubuntu，问题解决

