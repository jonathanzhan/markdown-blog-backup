---
title: activiti入门三(引擎配置)
date: 2016-10-10 19:18:00
tags:
- activiti
categories:
- activiti
---
本文主要介绍基于maven的pom文件的配置，另外就是activiti的Spring配置。
<!-- more -->
### maven配置文件
activiti主要的依赖包如下:

```xml
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-engine</artifactId>
    <version>${activiti.version}</version>
</dependency>
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring</artifactId>
    <version>${activiti.version}</version>
</dependency>
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-json-converter</artifactId>
    <version>${activiti.version}</version>
</dependency>
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-explorer</artifactId>
    <version>${activiti.version}</version>
    <exclusions>
        <exclusion>
            <artifactId>vaadin</artifactId>
            <groupId>com.vaadin</groupId>
        </exclusion>
        <exclusion>
            <artifactId>dcharts-widget</artifactId>
            <groupId>org.vaadin.addons</groupId>
        </exclusion>
        <exclusion>
            <artifactId>activiti-simple-workflow</artifactId>
            <groupId>org.activiti</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-modeler</artifactId>
    <version>${activiti.version}</version>
</dependency>
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-rest</artifactId>
    <version>${activiti.version}</version>
</dependency>
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-diagram-rest</artifactId>
    <version>${activiti.version}</version>
</dependency>
```

### 标准的activiti配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration">
        <property name="databaseType" value="mysql"/>
        <property name="databaseSchemaUpdate" value="true"/>
        <property name="jobExecutorActivate" value="true" />
        <property name="history" value="full"/>
    </bean>
</beans>
```

1. 定义了一个id为processEngineConfiguration的bean对象,其中processEngineConfiguration即为Activiti默认的引擎配置管理器名称
2. databaseSchemaUpdate用来声明数据库脚本的更新策略,和bibernate的机制类似。取值如下：
   - false:什么都不做
   - true:当Activiti的表不存在的情况下,自动创建表;当Activiti的jar文件定义中的定义版本号与数据库中记录的版本号不一致的时候会自动执行相应的升级脚本，并且会记录升级过程。
   - create-drop:创建引擎时执行初始化脚本,引擎销毁时,执行删除数据库脚本的操作
3. jobExecutorActivate:用来设置是否启用作业执行功能,默认为false。设置为true后,引擎会不间断的刷新数据库的作业表,检查是否存在需要执行的作业,有则触发作业的执行。作业的来源有多重,例如各种时间事件或异步任务执行。
4. history:用来设置记录历史的级别,默认为audit.支持以下几种类型：
    - none:不保存任何历史记录,可以提高系统性能。
    - activity:保存所有的流程实例,任务,活动信息
    - audit:也是activiti的默认级别,保存所有的流程实例,任务,活动,表单属性.
    - full:是最完成的历史记录,除了包含audit级别的信息以外,还保存详细,例如流程变量,表单属性

### 基于Spring的配置
Activiti从最开始设计时就考虑了与Spring的集成，其实它的默认配置文件activiti-cfg.xml就是Spring格式的。所以它与Spring的集成应该是很好处理的一件事情,具体如下:
```xml
<!-- 数据源 -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
    <property name="driverClass" value="org.h2.Driver" />
    <property name="url" value="jdbc:h2:file:~/activiti-in-action-chapter7;AUTO_SERVER=TRUE" />
    <property name="username" value="sa" />
    <property name="password" value="" />
</bean>
<!-- 定义事务 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
<!-- 定义基于Spring引擎配置对象bean-->
<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
    <property name="dataSource" ref="dataSource" />
    <property name="transactionManager" ref="transactionManager" />
    <property name="databaseSchemaUpdate" value="true" />
    <property name="jobExecutorActivate" value="true" />
    <property name="history" value="full" />
    <property name="processDefinitionCacheLimit" value="10"/>

    <!-- 生成流程图的字体 -->
    <property name="activityFontName" value="宋体"/>
    <property name="labelFontName" value="宋体"/>
</bean>

<!--定义引擎工厂bean-->
<bean id="processEngineFactory" class="org.activiti.spring.ProcessEngineFactoryBean">
    <property name="processEngineConfiguration" ref="processEngineConfiguration" />
</bean>

<!-- Activiti的七大service接口 -->
<bean id="repositoryService" factory-bean="processEngineFactory" factory-method="getRepositoryService" />
<bean id="runtimeService" factory-bean="processEngineFactory" factory-method="getRuntimeService" />
<bean id="formService" factory-bean="processEngineFactory" factory-method="getFormService" />
<bean id="identityService" factory-bean="processEngineFactory" factory-method="getIdentityService" />
<bean id="taskService" factory-bean="processEngineFactory" factory-method="getTaskService" />
<bean id="historyService" factory-bean="processEngineFactory" factory-method="getHistoryService" />
<bean id="managementService" factory-bean="processEngineFactory" factory-method="getManagementService" />

```

具体的配置含义在注释中都有解释

### 基于Spring配置的单元测试
#### 传统方式的测试
```java
/**
 * 基于传统的单元测试,测试Spring配置是否可以创建引擎对象
 *
 * @author Jonathan
 * @version 2016/10/11 12:07
 * @since JDK 7.0+
 */
public class CreateEngineUseSpringProxy {


    @Test
    public void createEngineUseSpring() {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext-test.xml");
        ProcessEngineFactoryBean factoryBean = context.getBean(ProcessEngineFactoryBean.class);
        assertNotNull(factoryBean);

        RuntimeService bean = context.getBean(RuntimeService.class);
        assertNotNull(bean);
    }
}
```

#### 基于注解方式的测试
```java
/**
 * 基于注解的单元测试,测试Spring配置是否可以创建引擎对象
 *
 * @author Jonathan
 * @version 2016/10/11 14:33
 * @since JDK 7.0+
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext-test.xml")
public class CreateEngineUseSpringProxyAnnotation {

    @Autowired
    RuntimeService runtimeService;

    @Autowired
    ProcessEngineFactoryBean processEngineFactoryBean;


    @Test
    public void testService() throws Exception {
        assertNotNull(runtimeService);
        ProcessEngine processEngine = processEngineFactoryBean.getObject();
        assertNotNull(processEngine.getRuntimeService());
    }
}

```




