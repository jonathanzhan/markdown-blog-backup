---
title: activiti入门七(identityService介绍)
date: 2016-10-28 16:19:20
tags:
 - activiti
categories:
 - activiti
toc: true
---
在Activiti中内置了一套简单的对用户和用户组的支持，用于满足基本的业务需求。org.activiti.engine.identity该包用来进行身份管理和认证，其功能依托于IdentityService接口。本文主要介绍下如何通过IdentityService实现用户，用户组的增删改查等常用功能。
<!-- more -->

### 用户管理
```java
/**
 * 用户管理
 */
@Test
public void testUser(){
    User user = identityService.newUser("Jonathan");
    user.setFirstName("Jonathan");
    user.setLastName("chang");
    user.setEmail("whatlookingfor@gmail.com");
    user.setPassword("123");
    //保存用户到数据库
    identityService.saveUser(user);
    //用户的查询
    User userInDb = identityService.createUserQuery().userId("Jonathan").singleResult();
    Assert.notNull(userInDb);
    //验证用户名和密码
    Assert.isTrue(identityService.checkPassword("Jonathan","123"));

    //删除用户
    identityService.deleteUser("Jonathan");

    //验证是否删除成功
    userInDb = identityService.createUserQuery().userId("Jonathan").singleResult();
    Assert.isNull(userInDb);
}
```
本段代码主要实现了以下功能
1. 创建用户对象  `identityService.newUser(String userId);`
2. 保存用户对象到数据库 `identityService.saveUser(User user);`
3. 创建查询对象   `identityService.createUserQuery();`
4. 用户对象的删除  `identityService.deleteUser(Strign userId);`

### 用户组管理
用户组，顾名思义，即可一组用户，他们拥有操作某些功能的权限。
在activiti中，用户组的类型分为两种，即assignment和security-role.前者代表一种普通的岗位角色，是用户分配业务中的功能权限。后者是安全角色，可以从全局管理用户组织以及整个流程的状态。
不过在项目中，貌似按照这种类型设置的很少，都是根据具体情况进行扩展的。

```java
    /**
     * 用户组管理
     */
    @Test
    public void testGroup(){
        //创建用户组对象
        Group group = identityService.newGroup("hr");
        group.setName("hr用户组");
        group.setType("assignment");
        //保存用户组
        identityService.saveGroup(group);
        //验证是否保存成功
        Group groupInDb = identityService.createGroupQuery().groupId("hr").singleResult();
        Assert.notNull(groupInDb);
        //删除用户组
        identityService.deleteGroup("hr");
        //验证是否删除成功
        groupInDb = identityService.createGroupQuery().groupId("hr").singleResult();
        Assert.isNull(groupInDb);
    }
```

用户组的代码功能与用户的功能类似，只是把用户改为用户组对象而已。这里不做介绍。


### 用户与用户组的关系
参考上面用户组的介绍，那么用户与用户组应该是个N：N的关系。如果系统仅仅有用户和用户组，那是远远不够的，还需要将用户与用户组关联起来。

```java
    /**
     * 用户 用户组管理
     */
    @Test
    public void testUserAndGroupMemership(){
        //创建并保存用户组
        Group group = identityService.newGroup("hr");
        group.setName("hr用户组");
        group.setType("assignment");
        //保存用户组
        identityService.saveGroup(group);

        User user = identityService.newUser("Jonathan");
        user.setFirstName("Jonathan");
        user.setLastName("chang");
        user.setEmail("whatlookingfor@gmail.com");
        user.setPassword("123");
        //保存用户到数据库
        identityService.saveUser(user);

        //将用户Jonathan加入到用户组hr中
        identityService.createMembership("Jonathan","hr");

        //查询属于HR用户组的用户
        User userInGroup = identityService.createUserQuery().memberOfGroup("hr").singleResult();
        Assert.notNull(userInGroup);
        Assert.isTrue(userInGroup.getId().equals("Jonathan"));
        //查询用户所属组
        Group groupContainsUser = identityService.createGroupQuery().groupMember("Jonathan").singleResult();
        Assert.notNull(groupContainsUser);
        Assert.isTrue(groupContainsUser.getId().equals("hr"));
    }
```

### 额外说明
identityService主要是提供基础的用户管理以及身份认证。用户任务中的用户和组的设定，参看后面章节的taskService的介绍。
