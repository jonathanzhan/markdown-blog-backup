---
title: 如何升级nodejs版本
date: 2016-03-31 16:10:29
tags:
- node
categories:
- 前端设计
---

Node.js的开发非常活跃,他的最新稳定版本也频繁变化,所以需要经常的升级Node.具体方法如下：
<!-- more -->
具体可以用Node Binary管理模块“n”
1. 检查当前node版本
    ``` bash
    node -v
    ```
2. 清除npm cache
     ``` bash
     sudo npm cache clean -f
     ```
3.  安装N模块
	``` bash
    sudo npm install -g n
    ```
4.  升级到最新版本,可以自定义升级到的版本，用以下命令<br/>
    ``` bash
    sudo n 0.8.11
    ```
    或者默认升级到最新文档的版本
    ``` bash
    sudo n stable
    ```
5.  检查版本是否更新完成
    ``` bash
    node -v
    ```
