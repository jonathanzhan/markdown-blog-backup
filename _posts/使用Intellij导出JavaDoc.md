---
title: thymeleaf进阶使用
date: 2015-10-25 16:44:13
tags:
- 开发工具
categories:
- java
---
本文主要介绍如何用Intellij IDEA生成JavaDoc文档，以及常见的一些注释操作。
<!-- more -->

常用的注释标签:

> @author 作者
> @version 版本
> @see 参考转向
> @param 参数说明
> @return 返回值说明
> @exception 异常说明
> @since jdk版本


导出文档方法:

在菜单栏选择Tools->Gerenate JavaDoc;

在弹出框内填写输出路径;

点击确认;

中文乱码解决方法:

在 Other command line arguments 中输入:
`-encoding utf-8 -charset utf-8`

