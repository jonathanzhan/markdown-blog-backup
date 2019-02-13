---
title: 安装ubuntu(16.04)后要做的事
date: 2016-08-17 15:06:27
tags:
- linux
categories:
- 运维实施
---
安装完ubuntu后需要做以下操作
<!-- more -->
### 删除libreoffice
`
sudo apt-get remove libreoffice-common
`
### 删除Amazon的链接
`
sudo apt-get remove unity-webapps-common
`
### 删掉基本不用的自带软件（用的时候再装也来得及）
`
sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku  landscape-client-ui-install
`
`
sudo apt-get remove onboard deja-dup
`
###  安装Vim
`
sudo apt-get install vim
`
### 设置时间使用UTC
`
sudo vim /etc/default/rcS
`
将UTC=no改为UTC=yes