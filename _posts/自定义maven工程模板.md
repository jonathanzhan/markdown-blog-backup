---
title: 自定义maven工程模板
date: 2016-03-31 16:20:17
categories:
- java
tags:
- maven
- java
---

以前研究Springside的时候觉得白衣大大很强大。尤其是根据其中demo的quickstart快速创建一个maven项目，感觉很强大。刚好最近应该会有这个需求，大概研究了下。
<!-- more -->
##### 具体步骤如下：
1. 首先需要有一个maven项目，需要定义groupId，artifactId，version
2. 在项目跟目录下执行
   ```
   mvn archetype:create-from-project
   ```
3. 在创建的targetgenerated-sourcesarchetype目录下执行

   ```
   mvn install
   ```
4. 执行完后会在本地maven仓库创建maven的项目模板，至此，已经可以根据这个maven模板创建项目
5. 在某个目录下执行

    ```
    call mvn archetype:generate -DarchetypeGroupId= 模板的groupid -DarchetypeArtifactId=模板的artifactId-archetype -DarchetypeVersion=模板的version
    ```
    或者<br/>

	```
    mvn archetype:generate -DarchetypeCatalog=local
	```
	就会在这个目录下创建一个基于maven模板的项目
