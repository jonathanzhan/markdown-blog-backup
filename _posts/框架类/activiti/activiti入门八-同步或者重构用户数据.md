---
title: activiti入门八(同步或者重构用户数据)
date: 2016-10-28 17:34:25
tags:
 - activiti
categories:
 - activiti
toc: true
---
在咖啡兔的blog中有一篇文章[同步或者重构activiti用户数据的比较](http://www.kafeitu.me/activiti/2012/04/23/synchronize-or-redesign-user-and-role-for-activiti.html),在这里再做简单的介绍以及补充
<!-- more -->

### 方案1:通过identityService接口进行同步。

在上篇文章[identityService介绍](/2016/10/28/activiti入门七-identityService介绍/)中，讲解了用户，用户组，用户与用户组关系的使用。
那么在我们业务系统中，在新增用户，角色，角色分配等功能中，增加activiti用户，用户组等操作即可实现用户数据的同步。

比如保存系统用户信息:
1. 调用系统业务代码保存用户
2. 调用identityService接口将用户数据保存到activiti中。注意的userId唯一性。
3. 同步用户组，用户与用户组关系等操作

简单点说就是两套表结构，都是用来存储用户，用户组信息的。只要保证数据一致性即可。

### 方案二:自定义sessionFactory

引擎内部与数据库交互使用的是MyBatis，对不同的表的CRUD操作均由一个对应的实体管理器（XxxEntityManager，有接口和实现类），引擎的7个Service接口在需要CRUD实体时会根据接口获取注册的实体管理器实现类（初始化引擎时用Map对象维护两者的映射关系），并且引擎允许我们覆盖内部的实体管理器，查看源码后可以知道有关Identity操作的两个接口分别为：UserIdentityManager和GroupIdentityManager。


查看引擎配置对象ProcessEngineConfigurationImpl类可以找到一个名称为“customSessionFactories”的属性，该属性可以用来自定义SessionFactory（每一个XXxManager类都是一个Session<实现Session接口>，由SessionFactory来管理），为了能替代内部的实体管理器的实现我们可以自定义一个SessionFactory并注册到引擎。

这种自定义SessionFactory的方式适用于公司内部有独立的身份系统或者公共的身份模块的情况，所有和用户、角色、权限的服务均通过一个统一的接口获取，而业务系统则不保存这些数据，此时引擎的身份模块表就没必要存在（ACT_ID_*）。

具体配置如下:

```xml
    <!-- 定义基于Spring引擎配置对象bean -->
    <bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
        <property name="dataSource" ref="dataSource" />
        <property name="transactionManager" ref="transactionManager" />
        <!--<property name="deploymentResources" value="classpath*:/act/deployments/**/*.bar"/>-->
        <property name="databaseSchemaUpdate" value="true" />
        <property name="jobExecutorActivate" value="true" />
        <property name="history" value="full" />
        <property name="processDefinitionCacheLimit" value="10"/>

        <!-- UUID作为主键生成策略 -->
        <property name="idGenerator" ref="idGen" />

        <!-- 生成流程图的字体 -->
        <property name="activityFontName" value="${activiti.diagram.activityFontName}"/>
        <property name="labelFontName" value="${activiti.diagram.labelFontName}"/>

        <!-- 自定义用户权限 -->
        <property name="customSessionFactories">
            <list>
                <bean class="com.mfnets.workfocus.modules.act.service.ext.ActUserEntityServiceFactory"/>
                <bean class="com.mfnets.workfocus.modules.act.service.ext.ActGroupEntityServiceFactory"/>
            </list>
        </property>
    </bean>
```


以下是ActGroupEntityServiceFactory与ActUserEntityServiceFactory的代码具体实现

```java
public class ActGroupEntityServiceFactory implements SessionFactory {

    @Autowired
    private ActGroupEntityService actGroupEntityService;

    public Class<?> getSessionType() {
        // 返回原始的GroupIdentityManager类型
        return GroupIdentityManager.class;
    }

    public Session openSession() {
        // 返回自定义的GroupEntityManager实例
        return actGroupEntityService;
    }

}
```


```
public class ActUserEntityServiceFactory implements SessionFactory {

    @Autowired
    private ActUserEntityService actUserEntityService;

    public Class<?> getSessionType() {
        // 返回原始的UserIdentityManager类型
        return UserIdentityManager.class;
    }

    public Session openSession() {
        // 返回自定义的GroupEntityManager实例
        return actUserEntityService;
    }

}
```

在ActGroupEntityService中，我们继承GroupEntityManager类，重写其中的代码即可。
```java
@Service
public class ActGroupEntityService extends GroupEntityManager {

    private SystemService systemService;

    public SystemService getSystemService() {
        if (systemService == null){
            systemService = SpringContextHolder.getBean(SystemService.class);
        }
        return systemService;
    }

    public Group createNewGroup(String groupId) {
        return new GroupEntity(groupId);
    }

    public void insertGroup(Group group) {
//      getDbSqlSession().insert((PersistentObject) group);
        throw new RuntimeException("not implement method.");
    }

    public void updateGroup(GroupEntity updatedGroup) {
//      CommandContext commandContext = Context.getCommandContext();
//      DbSqlSession dbSqlSession = commandContext.getDbSqlSession();
//      dbSqlSession.update(updatedGroup);
        throw new RuntimeException("not implement method.");
    }

    public void deleteGroup(String groupId) {
//      GroupEntity group = getDbSqlSession().selectById(GroupEntity.class, groupId);
//      getDbSqlSession().delete("deleteMembershipsByGroupId", groupId);
//      getDbSqlSession().delete(group);
        throw new RuntimeException("not implement method.");
    }

    public GroupQuery createNewGroupQuery() {
//      return new GroupQueryImpl(Context.getProcessEngineConfiguration().getCommandExecutorTxRequired());
        throw new RuntimeException("not implement method.");
    }

//  @SuppressWarnings("unchecked")
    public List<Group> findGroupByQueryCriteria(GroupQueryImpl query, Page page) {
//      return getDbSqlSession().selectList("selectGroupByQueryCriteria", query, page);
        throw new RuntimeException("not implement method.");
    }

    public long findGroupCountByQueryCriteria(GroupQueryImpl query) {
//      return (Long) getDbSqlSession().selectOne("selectGroupCountByQueryCriteria", query);
        throw new RuntimeException("not implement method.");
    }

    public List<Group> findGroupsByUser(String userId) {
//      return getDbSqlSession().selectList("selectGroupsByUserId", userId);
        List<Group> list = Lists.newArrayList();
        User user = getSystemService().getUserByLoginName(userId);
        if (user != null && user.getRoleList() != null){
            for (Role role : user.getRoleList()){
                list.add(ActUtils.toActivitiGroup(role));
            }
        }
        return list;
    }

    public List<Group> findGroupsByNativeQuery(Map<String, Object> parameterMap, int firstResult, int maxResults) {
//      return getDbSqlSession().selectListWithRawParameter("selectGroupByNativeQuery", parameterMap, firstResult, maxResults);
        throw new RuntimeException("not implement method.");
    }

    public long findGroupCountByNativeQuery(Map<String, Object> parameterMap) {
//      return (Long) getDbSqlSession().selectOne("selectGroupCountByNativeQuery", parameterMap);
        throw new RuntimeException("not implement method.");
    }

}
```

在ActUserEntityService中，我们继承UserEntityManager类，重写其中的代码即可。

```java
@Service
public class ActUserEntityService extends UserEntityManager {

    private SystemService systemService;

    public SystemService getSystemService() {
        if (systemService == null){
            systemService = SpringContextHolder.getBean(SystemService.class);
        }
        return systemService;
    }

    public User createNewUser(String userId) {
        return new UserEntity(userId);
    }

    public void insertUser(User user) {
        throw new RuntimeException("not implement method.");
    }

    public void updateUser(UserEntity updatedUser) {
        throw new RuntimeException("not implement method.");
    }


    public UserEntity findUserById(String userId) {
        return ActUtils.toActivitiUser(getSystemService().getUserByLoginName(userId));
    }

    public void deleteUser(String userId) {
        User user = findUserById(userId);
        if (user != null) {
            getSystemService().deleteUser(new com.mfnets.workfocus.modules.sys.entity.User(user.getId()));
        }
    }

    public List<User> findUserByQueryCriteria(UserQueryImpl query, Page page) {
        throw new RuntimeException("not implement method.");
    }

    public long findUserCountByQueryCriteria(UserQueryImpl query) {
        throw new RuntimeException("not implement method.");
    }

    public List<Group> findGroupsByUser(String userId) {
        List<Group> list = Lists.newArrayList();
        for (Role role : getSystemService().findRole(new Role(new com.mfnets.workfocus.modules.sys.entity.User(null, userId)))){
            list.add(ActUtils.toActivitiGroup(role));
        }
        return list;
    }

    public UserQuery createNewUserQuery() {
        throw new RuntimeException("not implement method.");
    }

    public IdentityInfoEntity findUserInfoByUserIdAndKey(String userId, String key) {
        throw new RuntimeException("not implement method.");
    }

    public List<String> findUserInfoKeysByUserIdAndType(String userId, String type) {
        throw new RuntimeException("not implement method.");
    }

    public Boolean checkPassword(String userId, String password) {
        throw new RuntimeException("not implement method.");
    }

    public List<User> findPotentialStarterUsers(String proceDefId) {
        throw new RuntimeException("not implement method.");

    }

    public List<User> findUsersByNativeQuery(Map<String, Object> parameterMap, int firstResult, int maxResults) {
        throw new RuntimeException("not implement method.");
    }

    public long findUserCountByNativeQuery(Map<String, Object> parameterMap) {
        throw new RuntimeException("not implement method.");
    }

}
```

这种方法，具体的适用环境参看上面，所以我们在继承XXEntityManager后，主要的核心代码在于查询，在调用activiti中findGroupsByUser等方法时，仅仅是从我们的业务系统中取出数据，并没有使用新增，修改，删除等影响数据库操作的操作。也就是说act_id_*中并没有数据。

我实例中的代码只写了部分的实现，在实际项目中，我们可以根据具体的需求，去重写具体需要的方法。


### 方法三:用视图覆盖相应的act_id_*表
此方案和方法二类似，放弃使用act_id_*的表，改用视图。
#### 删除已创建的ACT_ID_*表
创建视图必须删除引擎自动创建的ACT_ID_*表，否则不能创建视图。
#### 创建视图
. ACT_ID_GROUP
. ACT_ID_INFO
. ACT_ID_MEMBERSHIP
. ACT_ID_USER
创建的视图要保证数据类型一致，例如用户的ACT_ID_MEMBERSHIP表的两个字段都是字符型，一般系统中都是用NUMBER作为用户、角色的主键类型，所以创建视图的时候要把数字类型转换为字符型。

#### 修改引擎默认配置
在引擎配置中设置属性dbIdentityUsed为false即可。
```xml
<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
    ...
    <property name="dbIdentityUsed" value="false">
    ...
</property></bean>

```

### 总结
1. 方案一：通过数据推送方式同步数据到引擎的身份表，需要把数据备份到引擎的身份表或者公司有平台或者WebService推送用户数据的推荐使用，两边数据需要同步，工作流和业务系统耦合在一起，个人觉得不太好，不太适合中大型项目
2. 方案二：自定义SessionFactory，非侵入式替换接口实现，对于公司内部有统一身份访问接口的推荐使用，只要修改下接口实现即可
3. 方案三：不需要编写代码，只需要创建同名视图即可，但是对于权限比较复杂的业务数据，仅仅靠视图，效率以及功能，够用吗？？
