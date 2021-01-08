---
title: maven安装使用说明
date: 2017-01-25 16:45:02
tags:
- maven
categories:
- java
toc: true
---
### maven下载地址
maven的[下载地址](http://maven.apache.org/download.cgi)
![](/files/article/java/1.jpg)

<!-- more -->
### maven的安装
如果大概看了maven下载地址的说明了后，maven3.3以后的版本是需要JDK1.7的支持的。因此确保电脑上安装了JDK1.7+


#### 安装
将下载的maven解压到电脑的某个文件夹下，如D:\tools\apache-maven-3.3.9
#### 环境变量的设置
系统变量：`M2_HOME = D:\tools\apache-maven-3.3.9`
系统变量：`path = %M2_HOME%\bin`

设置完后，在cmd中输入命令：mvn -version
如果能打印出maven的相关信息，则设置没有问题。

#### 设置maven本地仓库
 在maven安装文件夹中，找到conf/settings.xml
 修改

```
 <localRepository>D:\tools\mavenRepo</localRepository>
```
默认本行代码的配置是注释掉的。其作用是设置maven的本地仓库地址，如果不设置，maven会默认将本地仓库建到/.m2/repository文件夹下。

#### 设置.m2文件夹
如果是windows，在C:\用户\用户名下，也就是系统默认的用户文件夹下，建立名为.m2的文件夹。
将conf/settings.xml拷贝到该文件夹下。则完成全部maven的安装。
很多情况下，.m2文件夹是不必要的，但是使用eclipse或者IDEA的话，会默认生成该文件夹，此文件夹中除了setttings.xml以外，还有一个配置文件，主要是设置项目本地的maven模板，一般情况下不会使用到的。

如果由于权限的问题，不能创建该文件夹，还可以通过eclipse或者IDEA来创建。

#### 开发工具下的maven配置
如果是eclipse的话，需要需要下载maven的插件（最新版的eclipse中已经集成）
如果是IDEA的话，则只需要设置maven的地址即可，如下图：
![](/files/article/java/2.jpg)
根据说明，设置maven的安装地址，配置文件地址，以及本地仓库地址。

### maven项目的导入
使用IDEA导入项目
目前IDEA已经做得很成熟了，可以导入maven项目，eclipse项目等。
1、import project
2、选择项目所在路径
3、如下图选择maven项目
4、然后就可以一直下一步，直到完成了。
eclipse其实也类似。
![](/files/article/java/3.jpg)
