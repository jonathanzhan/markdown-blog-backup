---
title: activiti入门一(简单介绍)
date: 2016-10-10 18:22:45
categories:
- activiti
tags:
- activiti
---
接触activiti已经1年多了，最近因为项目需要，把activiti又重新整合了下，把碰到的一些问题以及心得记下来，仅做参考。
<!-- more -->
### 什么是Activiti?

    在我们生活工作中,到处都是流程式的操作，比如上班过程中请假的审批，网购过程中的整个订单过程，从下单起，我们就可以跟踪整个订单的状态。

    工作流总是以任务(Task)的形式驱动人处理业务或者驱动业务系统自动完成作业。有了工作流引擎后，我们不必一直等待其他人的工作进度，只需要关注我们自己的待办任务即可。

    activiti是一个针对企业用户,开发人员，系统管理员的轻量级工作流业务管理平台，其核心是使用Java开发的快速，稳定的BPMN2.0流程引擎。

#### 相关资料

* [activiti官网](http://activiti.org/)
* [activti源码](https://github.com/Activiti/Activiti)
* [咖啡兔(Activiti实战作者)](http://www.kafeitu.me/)

#### 入门以及参考实例

1. activiti-explorer
    在activiti源码根目录下，有wars/activiti-explorer.war文件，将该文件放到tomcat/webapps下，启动tomcat，输入http://localhost:8080/activiti-explorer即可看到官网demo的实例
2. kft-activiti-demo
    咖啡兔写了个参考实例,参看[kft-activiti-demo](http://www.kafeitu.me/activiti/2012/08/04/resources-index-of-learn-activiti.html)

### activiti基本架构以及服务组件介绍

#### 基本架构图
![activiti架构图](/files/activiti/架构图.jpg)

#### 七大service接口
##### RepositoryService
Activiti 中每一个不同版本的业务流程的定义都需要使用一些定义文件，部署文件和支持数据 ( 例如 BPMN2.0 XML 文件，表单定义文件，流程定义图像文件等 )，这些文件都存储在 Activiti 内建的 Repository 中。Repository Service 提供了对 repository 的存取服务,流程仓库service，用于管理流程仓库，例如部署，删除，读取流程资源

##### RunTimeService
在Activiti中，每个流程定义被启动一次之后，都会生成相应的流程对象实例。RunTimeService提供启动流程，查询流程实例，设置获取流程实例变量等功能。此外还提供对流程部署，流程定义和流程实例存取的服务。

##### TaskService
在activiti业务流程定义的每一个执行节点被称为一个task，对流程中的数据存取，状态变更等操作都需要在task中完成。TaskService提供了对用户task和form的相关操作。提供了运行时任务的查询、领取、完成、删除以及变量设置等功能。

##### IdentityService
Activiti内置了用户以及用户组的概念以及功能，必须使用用户或者用户组才能获取到相应的task。IdentityService提供了对用户和用户组的管理功能。

##### HistoryService
主要用于获取正在运行或者已经运行结束的流程实例信息，与RunTimeService获取的流程信息不同，历史信息包含已经持久储存化的信息，并已经针对查询做出优化。

##### FormService
Activiti中的流程和状态Task均可关联相关的业务数据，通过FormService可以存取启动和完成任务所需的表单数据并根据需要来渲染表单。

##### ManagementService
ManagementService提供对流程引擎的管理和维护的功能，这些功能不在工作流驱动的应用程序中使用，主要运用activiti的日常维护。
