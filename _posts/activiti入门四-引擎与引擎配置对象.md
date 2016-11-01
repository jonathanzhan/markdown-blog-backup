---
title: activiti入门四(引擎与引擎配置对象)
date: 2016-10-11 14:39:48
tags:
- activiti
categories:
- activiti
---
在上文中,主要讲解了activiti的引擎配置,本文主要分析activiti的引擎与引擎配置对象的使用,以及中文乱码的解决方案。
<!-- more -->
### 引擎配置对象ProcessEngineConfiguration
引擎配置是配置Activiti的第一步，无论你使用Standalone还是Spring管理引擎都可以在配置文件中配置参数，虽然目前Activiti支持多种引擎配置对象，但是均继承自一个基础的配置对象（抽象类）org.activiti.engine.ProcessEngineConfiguration。

除了基础的引擎配置对象之外还有一下几个具体的实现，不同的场合使用不用的引擎实现，均继承自org.activiti.engine.impl.cfg.ProcessEngineConfigurationImpl

1. ** StandaloneProcessEngineConfiguration ** 标准的单机引擎配置对象，默认读取activiti.cfg.xml文件的配置
2. ** StandaloneInMemProcessEngineConfiguration ** 用于测试环境,jdbcUrl配置为jdbc:h2:mem:activiti，数据库的DDL操作配置：create-drop，在日常的快速测试中经常用到
3. ** JtaProcessEngineConfiguration ** 顾名思义,用于JPA
4. ** SpringProcessEngineConfiguration ** 也就是我们最常用的一个,有Spring代理创建引擎,最主要的是如果把引擎嵌入的业务系统中,可以做到业务事物与引擎事物的统一管理。
![ProcessEngineConfiguration类结构图](/files/activiti/ProcessEngineConfiguration类结构图.png)
### 配置引擎的别名以及获取引擎对象
Activiti允许创建多个引擎，每个引擎用别名区分，可以在引擎配置对象中设置一下属性，默认的引擎别名为：default。
#### 标准方式
```xml
<property name="processEngineName" value="myProcessEngine"></property>
```
其中的myProcessEngine即为引擎的别名，当需要获取引擎对象时可以通过下面的代码获取：
```java
ProcessEngine myProcessEngine = ProcessEngines.getProcessEngine("myProcessEngine");
```

当然如果只有一个引擎获取就更简单了，下面的代码可以直接获取一个默认的引擎对象。

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
// 等价于 ProcessEngines.getProcessEngine("default");
```

#### Spring方式
如果使用了Spring代理引擎可以使用“Spring”风格方式获取引擎对象，例如下面的配置(参考上篇文章)：
```xml
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
```

```java
@Autowired
    ProcessEngineFactoryBean processEngineFactoryBean;


    @Test
    public void testService() throws Exception {
        assertNotNull(runtimeService);
        ProcessEngine processEngine = processEngineFactoryBean.getObject();
        assertNotNull(processEngine.getRuntimeService());
    }
```

#### 在引擎外部设置引擎配置对象
原因是这样的，众所周知，在默认的配置情况下部署包含中文的流程文件会导致中文乱码（Linux、Windows，Mac平台没问题），所以需要覆盖引擎默认的字体配置属性（活动的字体与输出流文字字体），参看Spring方式中的生成流程图字体。

字体名称根据平台的不同其值也不同，例如在Windows平台下可以使用诸如宋体、微软雅黑等，但是在Linux平台下引擎没有这些字体（或者名称不同）需要特殊设置

流程图生成工具类ProcessDiagramGenerator会从当前的ThreadLocal中获取引擎配置对象，但是目前仅仅是引擎自动在内部实现时把引擎配置对象设置到ThreadLocal中，所以很多人遇到在Struts(2）或者Spring MVC中直接调用下面的代码得到的图片是乱码
```java
InputStream imageStream = ProcessDiagramGenerator.generateDiagram(bpmnModel, "png", activeActivityIds);
```
既然知道引擎从ThreadLocal中获取引擎配置对象，而且我们已经获取了引擎对象（也就是说可以从中获取引擎配置对象），解决问题的办法很简单，手动把引擎配置对象设置到ThreadLocal中就解决问题了；下面的代码在kft-activiti-demo的ActivitiController类中。
```java
@RequestMapping(value = "/process/trace/auto/{executionId}")
public void readResource(@PathVariable("executionId") String executionId, HttpServletResponse response)
throws Exception {
  ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
  .processInstanceId(executionId).singleResult();
  BpmnModel bpmnModel = repositoryService.getBpmnModel(processInstance.getProcessDefinitionId());
  List<string> activeActivityIds = runtimeService.getActiveActivityIds(executionId);
  // 不使用spring请使用下面的两行代码
  //    ProcessEngineImpl defaultProcessEngine = (ProcessEngineImpl) ProcessEngines.getDefaultProcessEngine();
  //    Context.setProcessEngineConfiguration(defaultProcessEngine.getProcessEngineConfiguration());
  // 使用spring注入引擎请使用下面的这行代码
  Context.setProcessEngineConfiguration(processEngine.getProcessEngineConfiguration());

  InputStream imageStream = ProcessDiagramGenerator.generateDiagram(bpmnModel, "png", activeActivityIds);

  // 输出资源内容到相应对象
  byte[] b = new byte[1024];
  int len;
  while ((len = imageStream.read(b, 0, 1024)) != -1) {
    response.getOutputStream().write(b, 0, len);
  }
}
```

关键就在于在调用生成流程图的方法之前调用一次Context.setProcessEngineConfiguration()方法即可，这样引擎就可以获取到引擎配置对象，从而获取到自定义的字体名称属性，乱码问题自然没有了。

