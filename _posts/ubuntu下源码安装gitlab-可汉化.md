---
title: 'ubuntu下源码安装gitlab,可汉化'
date: 2016-10-25 18:03:04
tags:
- linux
categories:
- 运维实施
---
此方法也适用于其他的linux版本。gitlab中有中文的源码汉化包，通过源码安装，一方面后面比较好配置，另外一方面，省去了汉化的步骤。不要相信网上说的汉化补丁，因为汉化的版本与你安装的版本几乎是不一致的，你需要找相同版本的。另外版本相同的情况下，补丁打好后，也只有那么不到百分之一的生效了。
<!-- more -->
具体步骤如下：
参考[gitlab官网安装说明](http://docs.gitlab.com/ce/install/installation.html)

前面的七步操作全部按照官网的来,数据库我使用的是mysql，在安装pgsql的时候报错了，对pgsql也不熟悉，所以改装mysql了。

在通过gitlab下载的时候需要注意更换源码的地址。因为我需要中文汉化包的源代码。
` sudo -u git -H git clone https://gitlab.com/larryli/gitlab.git -b 8-8-zh gitlab`

### 配置gitlab.yml说明：
在下载完gitlab，配置参数的时候需要注意
gitlab:
    host:  IP地址或者域名，不加http://

email_from: _配饰smtp使用的，设置为邮箱。

plain_url: "http://gravatar.duoshuo.com/avatar/%{hash}?s=%{size}&d=identicon"_  配置头像(默认的地址被墙了)
### Nginx配置
文档上说必须是要写域名的，我尝试了下，其实走IP也是可以的。

### 通知邮箱的配置(基于smtp)
首先是拷贝Gitlab自带的example
```bash
cd /home/git/gitlab
sudo -u git -H cp config/initializers/smtp_settings.rb.sample config/initializers/smtp_settings.rb
```

然后用我们自己的邮箱替换example中的邮箱
这里提供163和腾讯企业邮箱两种配置方式，注意将下面的123456替换成自己的密码
163：
```yml
address: "smtp.163.com",
    port: 25,
    user_name: "gitlab",
    password: "123456",
    domain: "163.com",
    authentication: :plain,
    enable_starttls_auto: true
```

腾讯企业邮箱
```yml
address: "smtp.exmail.qq.com",
    port: 25,
    user_name: “asdsd@asds.com",
    password: "123456",
    domain: "smtp.qq.com",
    authentication: :plain,
    enable_starttls_auto: true,
```

修改gitlab.yml
将默认邮箱修改为自己的邮箱
```yml
    email_from: asdsd@asds.com
```
重启
sudo /etc/init.d/gitlab restart

注意事项：
1. 不需要修改 config/environments/production.rb，网上的其他文章说要修改这个文件可能是针对老版本的，gitlab7.0不需要修改这个文件
2. 腾讯企业邮箱不能使用其帮助网页上所写的465端口。设置了openssl_verify_mode也没有作用。反正我没有设置成功。

