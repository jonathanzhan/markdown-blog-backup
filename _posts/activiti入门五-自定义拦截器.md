---
title: activiti入门五(自定义拦截器)
date: 2016-10-11 15:08:44
tags:
- activiti
categories:
- activiti
---
在上章引擎与引擎配置对象的介绍中,主要介绍了activiti的默认常用的几种引擎配置对象,接下来主要讲下自定义引擎对象以及拦截器.
<!-- more -->

### 自定义引擎配置
查看上章的ProcessEngineConfiguration类结构图,我们定义属于自已的引擎配置,只要继承抽象类ProcessEngineConfigurationImp即可.

```java
public class MyProcessEngineConfiguration extends ProcessEngineConfigurationImpl {


    public MyProcessEngineConfiguration() {
        // 做自定义设置
    }

    //测试属性，需要在processEngineConfiguration注入
    private String userName;

    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getUserName() {
        return this.userName;
    }

    @Override
    protected CommandInterceptor createTransactionInterceptor() {
        return new MyInterceptorA();
    }


    @Override
    protected Collection<? extends CommandInterceptor> getDefaultCommandInterceptors() {
        List<CommandInterceptor> interceptors = new ArrayList<CommandInterceptor>();
        interceptors.add(new LogInterceptor());
        CommandInterceptor transactionInterceptor = createTransactionInterceptor();
        if(transactionInterceptor!=null) {
            interceptors.add(transactionInterceptor);
        }
        interceptors.add(new MyInterceptorB());
        interceptors.add(new CommandContextInterceptor(commandContextFactory,this));
        return interceptors;
    }
}

```
如果注意重写的方法名的话，就可以发现这就是activiti中的拦截器。其拦截器的原理采用了设计模式中的命令模式和责任链模式。

查看org.activiti.engine.impl.cfg.ProcessEngineConfigurationImpl源码可以发现, 当初始化流程引擎的时候,会执行下面方法
```java
//org.activiti.engine.impl.cfg.ProcessEngineConfigurationImpl

public ProcessEngine buildProcessEngine() {
    init();
    return new ProcessEngineImpl(this);
  }
```

以下是init()方法的源码
```java
protected void init() {
    initConfigurators();
    configuratorsBeforeInit();
    initProcessDiagramGenerator();
    initHistoryLevel();
    initExpressionManager();
    initDataSource();
    initVariableTypes();
    initBeans();
    initFormEngines();
    initFormTypes();
    initScriptingEngines();
    initClock();
    initBusinessCalendarManager();
    initCommandContextFactory();
    initTransactionContextFactory();
    initCommandExecutors();
    initServices();
    initIdGenerator();
    initDeployers();
    initJobHandlers();
    initJobExecutor();
    initAsyncExecutor();
    initTransactionFactory();
    initSqlSessionFactory();
    initSessionFactories();
    initJpa();
    initDelegateInterceptor();
    initEventHandlers();
    initFailedJobCommandFactory();
    initEventDispatcher();
    initProcessValidator();
    initDatabaseEventLogging();
    configuratorsAfterInit();
  }
```
可以发现，init()方法，初始化了很多操作，其中包括initCommandExecutors(),添加拦截器的操作.
接下来查看return new ProcessEngineImpl(this)，来查看拦截器的作用时间。

```java
public ProcessEngineImpl(ProcessEngineConfigurationImpl processEngineConfiguration) {
    this.processEngineConfiguration = processEngineConfiguration;
    this.name = processEngineConfiguration.getProcessEngineName();
    this.repositoryService = processEngineConfiguration.getRepositoryService();
    this.runtimeService = processEngineConfiguration.getRuntimeService();
    this.historicDataService = processEngineConfiguration.getHistoryService();
    this.identityService = processEngineConfiguration.getIdentityService();
    this.taskService = processEngineConfiguration.getTaskService();
    this.formService = processEngineConfiguration.getFormService();
    this.managementService = processEngineConfiguration.getManagementService();
    this.dynamicBpmnService = processEngineConfiguration.getDynamicBpmnService();
    this.jobExecutor = processEngineConfiguration.getJobExecutor();
    this.asyncExecutor = processEngineConfiguration.getAsyncExecutor();
    this.commandExecutor = processEngineConfiguration.getCommandExecutor();
    this.sessionFactories = processEngineConfiguration.getSessionFactories();
    this.transactionContextFactory = processEngineConfiguration.getTransactionContextFactory();

    commandExecutor.execute(processEngineConfiguration.getSchemaCommandConfig(), new SchemaOperationsProcessEngineBuild());

    if (name == null) {
      log.info("default activiti ProcessEngine created");
    } else {
      log.info("ProcessEngine {} created", name);
    }

    ProcessEngines.registerProcessEngine(this);

    if (jobExecutor != null && jobExecutor.isAutoActivate()) {
      jobExecutor.start();
    }

    if (asyncExecutor != null && asyncExecutor.isAutoActivate()) {
      asyncExecutor.start();
    }

    if (processEngineConfiguration.getProcessEngineLifecycleListener() != null) {
      processEngineConfiguration.getProcessEngineLifecycleListener().onProcessEngineBuilt(this);
    }

    processEngineConfiguration.getEventDispatcher().dispatchEvent(
            ActivitiEventBuilder.createGlobalEvent(ActivitiEventType.ENGINE_CREATED));
  }
```
 可以发现
` commandExecutor.execute(processEngineConfiguration.getSchemaCommandConfig(), new SchemaOperationsProcessEngineBuild()); `
触发了拦截器的操作

### 拦截器
在定义的引擎中有InterceptorA的拦截器,其定义如下：
```java
/**
 * 继承CommandContextInterceptor,实现execute方法--使用的是责任链设计模式
 *
 * @author Jonathan
 * @version 2016/10/11 15:44
 * @since JDK 7.0+
 */
public class MyInterceptorA extends CommandContextInterceptor {
    @Override
    public <T> T execute(CommandConfig config, Command<T> command) {
        System.out.println("this is interceptor A "+command.getClass().getName());
        return next.execute(config, command);
    }
}
```

### 自定义拦截器的测试

在上面两节,分别自定义了流程引擎以及拦截器,接下来我们配置对应的activiti的配置文件，以及测试代码
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置自定义属性 -->
    <bean id="processEngineConfiguration" class="com.mfnets.workfocus.activiti.MyProcessEngineConfiguration">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/workfocus" />
        <property name="jdbcDriver" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="123456" />
        <property name="databaseSchemaUpdate" value="false"></property>
    </bean>

</beans>
```

测试代码如下:
```java
@Test
    public void createMyEngine(){
        //创建配置对象
        MyProcessEngineConfiguration configuration = (MyProcessEngineConfiguration) ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("my-config.xml");
        //初始化流程引擎
        ProcessEngine engine = configuration.buildProcessEngine();
        Assert.assertTrue("Jonathan".equals(configuration.getUserName()));
        //部署一个简单的流程(注释是因为项目中已经部署,可以直接使用)
//      engine.getRepositoryService().createDeployment().addClasspathResource("bpmn/leave-normal-from.bpmn20.xml").deploy();

        //开始流程
        Map<String, Object> vars = new HashMap<String, Object>();
        vars.put("title", "12312");
        engine.getRuntimeService().startProcessInstanceByKey("leave-normal-from",vars);

    }
```

输出结果主要内容如下:

```
16:41:08.011 [main] INFO  org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [my-config.xml]
log4j:WARN No appenders could be found for logger (org.apache.ibatis.thread.Runnable).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
this is interceptor A org.activiti.engine.impl.SchemaOperationsProcessEngineBuild
this is interceptor B org.activiti.engine.impl.SchemaOperationsProcessEngineBuild
16:41:09.527 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntity.selectDbSchemaVersion - ==>  Preparing: select VALUE_ from ACT_GE_PROPERTY where NAME_ = 'schema.version'
16:41:09.552 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntity.selectDbSchemaVersion - ==> Parameters:
16:41:09.565 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntity.selectDbSchemaVersion - <==      Total: 1
16:41:09.571 [main] INFO  org.activiti.engine.impl.ProcessEngineImpl - ProcessEngine default created
this is interceptor A org.activiti.engine.impl.cmd.StartProcessInstanceCmd
this is interceptor B org.activiti.engine.impl.cmd.StartProcessInstanceCmd
16:41:09.581 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntity.selectLatestProcessDefinitionByKey - ==>  Preparing: select * from ACT_RE_PROCDEF where KEY_ = ? and (TENANT_ID_ = '' or TENANT_ID_ is null) and VERSION_ = (select max(VERSION_) from ACT_RE_PROCDEF where KEY_ = ? and (TENANT_ID_ = '' or TENANT_ID_ is null))
16:41:09.582 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntity.selectLatestProcessDefinitionByKey - ==> Parameters: leave-normal-from(String), leave-normal-from(String)
16:41:09.585 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntity.selectLatestProcessDefinitionByKey - <==      Total: 1
16:41:09.586 [main] DEBUG org.activiti.engine.impl.persistence.entity.DeploymentEntity.selectDeploymentById - ==>  Preparing: select * from ACT_RE_DEPLOYMENT where ID_ = ?

......下面省略.......
```



