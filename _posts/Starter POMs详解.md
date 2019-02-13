﻿---
title:  Starter POMs详解
date: 2016-05-13 10:47:17
tags:
- SpringBoot
categories:
- Spring
toc: true
---
Starter POMs是可以包含到应用中的一个方便的依赖关系描述符集合。你可以获取所有Spring及相关技术的一站式服务，而不需要翻阅示例代码，拷贝粘贴大量的依赖描述符。例如，如果你想使用Spring和JPA进行数据库访问，只需要在你的项目中包含`spring-boot-starter-data-jpa`依赖，然后你就可以开始了。

该starters包含很多你搭建项目，快速运行所需的依赖，并提供一致的，管理的传递依赖集。
<!-- more -->
**名字有什么含义**：所有的starters遵循一个相似的命名模式：`spring-boot-starter-*`，在这里`*`是一种特殊类型的应用程序。该命名结构旨在帮你找到需要的starter。很多IDEs集成的Maven允许你通过名称搜索依赖。例如，使用相应的Eclipse或STS插件，你可以简单地在POM编辑器中点击`ctrl-space`，然后输入"spring-boot-starter"可以获取一个完整列表。

下面的应用程序starters是Spring Boot在`org.springframework.boot`组下提供的：

**Spring Boot application starters** 

|名称|描述|
|------|:-----|
|spring-boot-starter|核心Spring Boot starter，包括自动配置支持，日志和YAML|
|spring-boot-starter-actuator|生产准备的特性，用于帮你监控和管理应用|
|spring-boot-starter-amqp|对"高级消息队列协议"的支持，通过`spring-rabbit`实现|
|spring-boot-starter-aop|对面向切面编程的支持，包括`spring-aop`和AspectJ|
|spring-boot-starter-batch|对Spring Batch的支持，包括HSQLDB数据库|
|spring-boot-starter-cloud-connectors|对Spring Cloud Connectors的支持，简化在云平台下（例如，Cloud Foundry 和Heroku）服务的连接|
|spring-boot-starter-data-elasticsearch|对Elasticsearch搜索和分析引擎的支持，包括`spring-data-elasticsearch`|
|spring-boot-starter-data-gemfire|对GemFire分布式数据存储的支持，包括`spring-data-gemfire`|
|spring-boot-starter-data-jpa|对"Java持久化API"的支持，包括`spring-data-jpa`，`spring-orm`和Hibernate|
|spring-boot-starter-data-mongodb|对MongoDB NOSQL数据库的支持，包括`spring-data-mongodb`|
|spring-boot-starter-data-rest|对通过REST暴露Spring Data仓库的支持，通过`spring-data-rest-webmvc`实现|
|spring-boot-starter-data-solr|对Apache Solr搜索平台的支持，包括`spring-data-solr`|
|spring-boot-starter-freemarker|对FreeMarker模板引擎的支持|
|spring-boot-starter-groovy-templates|对Groovy模板引擎的支持|
|spring-boot-starter-hateoas|对基于HATEOAS的RESTful服务的支持，通过`spring-hateoas`实现|
|spring-boot-starter-hornetq|对"Java消息服务API"的支持，通过HornetQ实现|
|spring-boot-starter-integration|对普通`spring-integration`模块的支持|
|spring-boot-starter-jdbc|对JDBC数据库的支持|
|spring-boot-starter-jersey|对Jersey RESTful Web服务框架的支持|
|spring-boot-starter-jta-atomikos|对JTA分布式事务的支持，通过Atomikos实现|
|spring-boot-starter-jta-bitronix|对JTA分布式事务的支持，通过Bitronix实现|
|spring-boot-starter-mail|对`javax.mail`的支持|
|spring-boot-starter-mobile|对`spring-mobile`的支持|
|spring-boot-starter-mustache|对Mustache模板引擎的支持|
|spring-boot-starter-redis|对REDIS键值数据存储的支持，包括`spring-redis`|
|spring-boot-starter-security|对`spring-security`的支持|
|spring-boot-starter-social-facebook|对`spring-social-facebook`的支持|
|spring-boot-starter-social-linkedin|对`spring-social-linkedin`的支持|
|spring-boot-starter-social-twitter|对`spring-social-twitter`的支持|
|spring-boot-starter-test|对常用测试依赖的支持，包括JUnit, Hamcrest和Mockito，还有`spring-test`模块|
|spring-boot-starter-thymeleaf|对Thymeleaf模板引擎的支持，包括和Spring的集成|
|spring-boot-starter-velocity|对Velocity模板引擎的支持|
|spring-boot-starter-web|对全栈web开发的支持，包括Tomcat和`spring-webmvc`|
|spring-boot-starter-websocket|对WebSocket开发的支持|
|spring-boot-starter-ws|对Spring Web服务的支持|

除了应用程序的starters，下面的starters可以用于添加[生产准备](../V. Spring Boot Actuator/README.md)的特性。

**Spring Boot生产准备的starters**

|名称|描述|
|----|:----|
|spring-boot-starter-actuator|添加生产准备特性，比如指标和监控|
|spring-boot-starter-remote-shell|添加远程`ssh` shell支持|

最后，Spring Boot包含一些可用于排除或交换具体技术方面的starters。

**Spring Boot technical starters**

|名称|描述|
|------|:------|
|spring-boot-starter-jetty|导入Jetty HTTP引擎（作为Tomcat的替代）|
|spring-boot-starter-log4j|对Log4J日志系统的支持|
|spring-boot-starter-logging|导入Spring Boot的默认日志系统（Logback）|
|spring-boot-starter-tomcat|导入Spring Boot的默认HTTP引擎（Tomcat）|
|spring-boot-starter-undertow|导入Undertow HTTP引擎（作为Tomcat的替代）|

**注**：查看GitHub上位于`spring-boot-starters`模块内的[README文件](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/README.adoc)，可以获取到一个社区贡献的其他starter POMs列表。
