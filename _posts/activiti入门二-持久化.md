---
title: activiti入门二(持久化)
date: 2016-10-10 19:12:52
categories:
- activiti
tags:
- activiti
---
- Activiti 使用 Mybatis3 做持久化工作,可以在配置中设置流程引擎启动时创建表。
- Activiti 使用到的表都是 ACT_开头的。
- ACT_RE_*:流程定义存储。
- ACT_RU_*:流程执行记录,记录流程启动到结束的所有动作,流程结束后会清除相关记录。
- ACT_ID_*:用户记录,流程中使用到的用户和组。
- ACT_HI_*:流程执行的历史记录。
- ACT_GE_*:通用数据及设置。
<!-- more -->

使用到的表:


- ACT_GE_BYTEARRAY:流程部署的数据。二进制数据表
- ACT_GE_PROPERTY:通用设置。 属性数据表存储整个流程引擎级别的数据,初始化表结构时，会默认插入三条记录
- ACT_HI_ACTINST:流程活动的实例，历史节点表
- ACT_HI_ATTACHMENT: 历史附件表
- ACT_HI_COMMENT:历史意见表
- ACT_HI_DETAIL: 历史详情表，提供历史变量的查询
- ACT_HI_PROCINST:历史流程实例。
- ACT_HI_TASKINST:历史任务实例。
- ACT_ID_GROUP:用户组。
- ACT_ID_INFO:用户扩展信息表
- ACT_ID_MEMBERSHIP: 用户组与用户对应信息表
- ACT_ID_USER:用户。
- ACT_RE_DEPLOYMENT:部署记录。
- ACT_RE_PROCDEF:流程定义数据表。
- ACT_RU_EXECUTION:流程执行记录。
- ACT_RU_IDENTITYLINK:运行时的流程人员表，主要储存任务节点与参与者的相关信息
- ACT_RU_JOB: 定时任务数据表
- ACT_RU_TASK:执行的任务节点记录。
- ACT_RU_VARIABLE:执行中的变量记录。
