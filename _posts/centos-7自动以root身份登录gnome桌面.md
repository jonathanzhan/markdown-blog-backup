---
title: centos_7自动以root身份登录gnome桌面
date: 2016-08-29 17:10:38
tags:
- linux
categories:
- 运维实施
---
本文主要介绍在contos7下，如何以root身份登录gnome桌面
<!-- more -->
## CentOS 7自动以root身份登录gnome桌面
刚刚在虚拟机中成功的安装上了CentOS 7 64位，发现在登录gnome桌面时必须创建一个普通用户，否则不让登录。

重启CentOS发现下方藏有一个使用其他用户登录选项，可以输入用户名使用root登录。
### 1. 配置root自动登录gnome
在配置的时候会遇到GDM：

GDM是什么？
> GDM (The GNOME Display Manager)是GNOME显示环境的管理器，并被用来替代原来的X Display Manager。与其竞争者(X3DM,KDM,WDM)不同，GDM是完全重写的，并不包含任何XDM的代码。GDM可以运行并管理本地和远程登录的X服务器(通过XDMCP)。
>
> gdm仅仅是一个脚本，实际上是通过他来运行GDM二进制可执行文件。
>
> gdm-stop是用来迅速终止当前正在运行的gdm守护进程的一个脚本。
>
> gdm-restart脚本将迅速重启当前守护进程。
>
> gdm-safe-restart会当所有人都注销后再重启。
>
> gdmsetup是一种可以很简单的修改多数常用选项的图形化界面工具。

百度上搜到的都是RHEL、CentOS 6的配置方法，并不适用于CentOS 7

> 在CentOS 6较新版本的Linux发行版中预设不允许以root账号登入gnome图形用户桌面，因此一般使用者登入后，可以在终端机以su root，暂时取得root权限；
>
> 如果一定要以root登入图形界面，可以修改/etc/pam.d/gdm以及 /etc/pam.d/gdm-passwd，把这行auth required pam_succeed_if.so user != root quiet加上#注释掉，保存后就可以用root账号了。


对于CentOS 7的用户,简单来说就是：vi /etc/gdm/custom.conf

然后在[daemon]下面添加：

```
[daemon]
AutomaticLoginEnable=True
AutomaticLogin=root  #你想自动登录的用户名
```
保存并重启，重启的时候已经以root用户登录了。

