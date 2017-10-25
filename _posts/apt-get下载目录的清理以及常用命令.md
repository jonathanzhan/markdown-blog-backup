---
title: apt-get下载目录的清理以及常用命令
date: 2016-08-17 17:09:11
tags:
- linux
categories:
- 运维实施
---

### 使用说明
apt-get install 这样的命令会下载文件放在 /var/cache/apt/archives目录下，然后安装。这样这个目录所占空间会越来越大，幸运的是apt提供了相应的管理工具apt-get clean删除/var/cache/apt/archives/ 和 /var/cache/apt/archives/partial/目录下所有包(锁定的除外)。

apt-get autoclean仅删除不再能被下载的包. 另外aptitude clean也可删除/var/cache/apt/archives/ 和 /var/cache/apt/archives/partial/目录下所有包(锁定的除外)。

### apt-get autoclean:
如果你的硬盘空间不大的话，可以定期运行这个程序，将已经删除了的软件包的.deb安装文件从硬盘中删除掉。如果你仍然需要硬盘空间的话，可以试试apt-get clean，这会把你已安装的软件包的安装包也删除掉，当然多数情况下这些包没什么用了，因此这是个为硬盘腾地方的好办法。

### apt-get clean:
类似上面的命令，但它删除包缓存中的所有包。这是个很好的做法，因为多数情况下这些包没有用了。但如果你是拨号上网的话，就得重新考虑了。


### apt-get autoremove:
删除为了满足其他软件包的依赖而安装的，但现在不再需要的软件包。

### apt-get remove 软件包名称：
删除已安装的软件包（保留配置文件）。
### apt-get --purge remove 软件包名称：
删除已安装包（不保留配置文件)。
