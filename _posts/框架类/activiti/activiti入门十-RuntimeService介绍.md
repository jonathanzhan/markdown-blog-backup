---
title: activiti入门十(RuntimeService介绍)
date: 2016-11-01 11:03:33
tags:
 - activiti
categories:
 - activiti
toc: true
---
上篇文章*repositoryService介绍*中讲解的repositoryService主要用于管理流程部署的数据，而RuntimeService则主要用于管理流程在运行时产生的数据(流程参数，事件，流程实例，以及执行流)以及对正在运行的流程进行操作的API。
<!--more -->

### 流程实例与执行流的概念
在Activiti中，启动了一个流程后，就会创建一个流程实例(ProcessInstance),简单来说流程实例就是根据一次（一条）业务数据用流程驱动的入口
Execution的含义就是一个流程实例（ProcessInstance）具体要执行的过程对象。
两者的对象映射关系：ProcessInstance（1）--->Execution(N)，其中N >= 1。
每个流程实例至少会有一个执行流(execution)，如果流程中没有分支，则N=1，如果流程中出现了分支，则N>1

### 流程启动
RuntimeService提供了很多启动流程的API，并且全部的命名规则为startProcessInstanceByXX，比如按照流程定义key值启动的，按照流程定义Id启动的等等。
同时在formService中也提供了流程启动的方法：
` ProcessInstance submitStartFormData(String processDefinitionId, Map<String, String> properties); `
和
` ProcessInstance submitStartFormData(String processDefinitionId, String businessKey, Map<String, String> properties); `
这两个方法主要是在动态表单中启动流程使用的，这个在后面章节会讲到。
接下来看个简单的流程启动的代码
```Java
/**
 * 启动流程的测试
 */
@Test
public void startProcessTest(){
	//系统已经提前部署了一个processDefinitionKey='timer-serviceTask'的流程

	//启动流程
	Map<String,Object> vars = new HashMap<String, Object>();
	vars.put("title","启动流程");
	ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processDefinitionKey,vars);
	//验证是否启动成功
	Assert.assertNotNull(processInstance);
	//通过查询正在运行的流程实例来判断
	ProcessInstanceQuery processInstanceQuery = runtimeService.createProcessInstanceQuery();
	//根据流程实例ID来查询
	List<ProcessInstance> runningList = processInstanceQuery.processInstanceId(processInstance.getProcessInstanceId()).list();
	Assert.assertTrue(runningList.size()>0);
}
```
以上代码是按照流程定义的key值，并且加入变量来启动流程。

### 流程的激活和挂起
假如要暂时停止某个流程实例的执行，那么就将其挂起，反之激活。
```Java
/**
 * 流程的挂起和激活
 */
@Test
public void suspendAndActivateTest(){
	ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processDefinitionKey(processDefinitionKey).variableValueEquals("title","启动流程").singleResult();
	String processInstanceId = processInstance.getProcessInstanceId();

	System.out.println(processInstance.isSuspended());
	//挂起流程实例
	runtimeService.suspendProcessInstanceById(processInstanceId);
	//验证是否挂起
	Assert.assertTrue(runtimeService.createProcessInstanceQuery().processInstanceId(processInstanceId).singleResult().isSuspended());

	//激活流程实例
	runtimeService.activateProcessInstanceById(processInstanceId);
	//验证是否激活
	Assert.assertTrue(!runtimeService.createProcessInstanceQuery().processInstanceId(processInstanceId).singleResult().isSuspended());
}
```


### 执行流的查询
RuntimeService中有createExecutionQuery方法可以得到一个ExecutionQuery对象，该对象就可以根据执行流的相关数据查询执行流
```Java
/**
 * 执行流的查询
 */
@Test
public void executionQueryTest(){
  List<Execution> executionList = runtimeService.createExecutionQuery().processDefinitionKey(processDefinitionKey).list();
  Assert.assertTrue(executionList.size()>0);
}
```


### 流程实例的查询
与执行流类似的，RuntimeService提供了一个createProcessInstanceQuery的对象，可以查询对应的流程实例信息
```Java
/**
 * 流程实例的查询
 */
@Test
public void processInstanceQueryTest(){
	//根据流程定义Key值查询正在运行的流程实例
	List<ProcessInstance> processInstanceList = runtimeService.createProcessInstanceQuery().processDefinitionKey(processDefinitionKey).list();
	Assert.assertTrue(processInstanceList.size()>0);

	//查询激活的流程实例
	List<ProcessInstance> activateList = runtimeService.createProcessInstanceQuery().processDefinitionKey(processDefinitionKey).active().list();
	Assert.assertTrue(activateList.size()>0);

	//相反 查询挂起的流程则是
	List<ProcessInstance> suspendList = runtimeService.createProcessInstanceQuery().processDefinitionKey(processDefinitionKey).suspended().list();
	Assert.assertTrue(suspendList.size()==0);

	//根据变量来查询
	// 根据title='启动流程',以及processDefinitionKey来作为查询条件进行查询
	List<ProcessInstance> varList = runtimeService.createProcessInstanceQuery().variableValueEquals("title","启动流程").list();
	Assert.assertTrue(varList.size()>0);
}
```

### 流程实例的删除
```Java
/**
 * 流程实例的删除
 */
@Test
public void deleteProcessInstanceTest(){
	ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processDefinitionKey(processDefinitionKey).variableValueEquals("title","启动流程").singleResult();
	String processInstanceId = processInstance.getProcessInstanceId();
	runtimeService.deleteProcessInstance(processInstanceId,"删除测试");
}
```
