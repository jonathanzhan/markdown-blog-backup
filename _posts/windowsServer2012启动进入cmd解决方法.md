---
title: windows Server 2012启动进入cmd解决方法
date: 2017-01-25 18:05:04
tags:
- windows
categories:
- 运维实施
---
```bash
Dism /online /enable-feature /all /featurename:Server-Gui-Mgmt /featurename:Server-Gui-Shell /featurename:ServerCore-FullServer 
```
