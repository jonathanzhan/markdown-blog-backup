---
title: maven命令导入jar包到本地仓库
date: 2018-09-03 15:02:01
categories:
- java
tags:
- maven
---
在使用maven做版本控制时，通常会有一部分jar包没有maven版本，有两种解决方案
<!-- more -->
### 导入jar包到本地仓库
```bash
mvn install:install-file -DgroupId=com.xx -DartifactId=xx-sdk -Dversion=1.0 -Dfile=/Users/xx/xx-sdk-1.0.jar -Dpackaging=jar -DgeneratePom=true
```

### pom文件增加scope增加system属性
```xml
<dependency>
    <groupId>com.xx</groupId>
    <artifactId>xxi-sdk</artifactId>
    <version>1.0</version>
    <systemPath>${project.basedir}/lib/xx-1.0.jar</systemPath>
    <scope>system</scope>
</dependency>
```