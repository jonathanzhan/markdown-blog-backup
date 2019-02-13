---
title: activiti入门九(repositoryService介绍)
date: 2016-10-31 16:48:29
tags:
 - activiti
categories:
 - activiti
toc: true
---
在*activiti入门一(简单介绍中)*,简单介绍了repositoryService的主要作用是管理流程仓库，例如部署，删除，读取流程资源等。本文就repositoryService再做下详细的说明
<!-- more -->

### 流程的部署
常用的流程定义文件扩展名有:bpmn20.xml和bpmn。流程定义的图片一般用png格式。

部署流程资源的实现方式有很多种，如classPath，InputStream，字符串，zip压缩包格式等。下面简单介绍下各种方式的使用。

#### 通过classPath方式部署

classPath方式，就是以class目录为根路径来读取相应的资源文件并进行部署。路径的命名方式是用斜杠"/"来分割包名。
具体实现看代码
```java
@Test
public void classPathDeploymentTest(){

    //定义资源文件的classPath
    String bpmnClassPath = "/bpmn/leave-dynamic-form.bpmn20.xml";
    String pngClassPath = "/bpmn/leave-dynamic-form.png";
    //创建部署构建器
    DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();

    //添加资源
    deploymentBuilder.addClasspathResource(bpmnClassPath);
    deploymentBuilder.addClasspathResource(pngClassPath);
    //执行部署
    deploymentBuilder.deploy();
    //验证部署是否成功
    ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();
    long count = processDefinitionQuery.processDefinitionKey("leave-dynamic-from").count();
    Assert.isTrue(count == 1);
    //读取图片文件
    ProcessDefinition processDefinition = processDefinitionQuery.singleResult();
    String diagramResourceName = processDefinition.getDiagramResourceName();
    System.out.println(diagramResourceName);

}
```

该方式的每行的代码作用参看注释。classPath一般用于开发阶段，在真正的产品使用中，很少会用到这种方式。

#### InputStream方式部署
InputStream方式，简单点说就是把文件转换为输入流，来进行部署。输入流的来源，可以从classPath读取，也可以从一个绝对路径读取，也可以根据上传下载的文件来。

```java
@Test
public void inputStreamFromAbsolutePathTest() throws Exception{
    String filePath = "/Users/whatlookingfor/code/workfocus/src/test/resources/bpmn/leave-dynamic-form.bpmn20.xml";
    //读取filePath文件作为一个输入流
    FileInputStream fileInputStream = new FileInputStream(filePath);
    repositoryService.createDeployment().addInputStream("leave-dynamic-form.bpmn20.xml",fileInputStream).deploy();
    //验证部署是否成功
    ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();
    long count = processDefinitionQuery.processDefinitionKey("leave-dynamic-form").count();
    Assert.isTrue(count == 1);
}
```

使用inputStream部署时，需要额外的参数是资源的名称。以上的代码是从一个绝对路径去读取文件，生成inputStream输入流。
这种方式可能是在项目中最广泛使用的，比如通过上传bpmn文件来部署，或者给出文件URL地址来部署。

#### 字符串方式部署
字符串的方式部署，就是把bpmn或者xml中的内容转换为纯文本来作为资源部署的。
```java
@Test
public void stringDeploymentTest(){
    String text = "xml内容";
    repositoryService.createDeployment().addString("leave-dynamic-form.bpmn20.xml",text);
    //验证是否部署成功
    ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();
    long count = processDefinitionQuery.processDefinitionKey("leave-dynamic-form").count();
    Assert.isTrue(count == 1);
}
```

#### zip/bar格式的压缩包方式部署
无论是classPath，InputStream，还是字符串方式，每次部署，都只能部署一个流程资源。而压缩包方式的部署，则可以同时部署多个流程资源。压缩包中有多少个资源文件，则就可以部署多少个，其实类似于classPath的批量
```java
@Test
    public void zipStreamDeploymentTest(){
        //读取资源
        InputStream zipStream = getClass().getClassLoader().getResourceAsStream("bpmn/leave.zip");
        repositoryService.createDeployment().addZipInputStream(new ZipInputStream(zipStream)).deploy();
        //统计已部署流程定义的数量
        long count = repositoryService.createProcessDefinitionQuery().count();
        Assert.isTrue(count == 1);
        //查询部署记录
        Deployment deployment = repositoryService.createDeploymentQuery().singleResult();
        Assert.notNull(deployment);
        String deploymentId = deployment.getId();
        //验证资源文件是否都部署成功
        Assert.notNull(repositoryService.getResourceAsStream(deploymentId,"leave-dynamic-form.bpmn"));
        Assert.notNull(repositoryService.getResourceAsStream(deploymentId,"leave-dynamic-form.png"));
    }
```

看代码可以发现，压缩包方式，也是转换为输入流的。所以，在项目中，最常用的就是inputStream方式和压缩包方式的整合。
通过上传功能(格式包含bpmn,bpmn20.xml,zip,bar)，然后根据文件格式，使用不同的部署方式。

### 读取流程资源

#### 读取已部署流程定义
其实在上面的代码中，验证是否部署成功，已经实现了读取已部署的流程定义
```java
@Test
public void findDefinitionTest(){
    ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();
    List<ProcessDefinition> processDefinitionList = processDefinitionQuery.listPage(0,10);
}
```


#### 通过接口读取资源流
` InputStream getResourceAsStream(String deploymentId, String resourceName); `
通过此方式，可以读取资源流，然后根据各种不同的情况，输入资源流即可。

` List<String> getDeploymentResourceNames(String deploymentId); `
读取某个部署ID对应的文件资源名称


### 删除部署
具体参看代码
```java
public void deleteDeployment(){
    String deploymentId = "leave-dynamic-form:1:5004";
    repositoryService.deleteDeployment(deploymentId,true);
}
```
deleteDeployment方法有两个参数，第一个是部署ID，第二个参数true表示删除时，会同时把流程相关的流程数据(包括运行中的和已经结束的流程实例)一并删除掉。

### 额外的API方法

#### 流程模型的激活和挂起
流程模型的激活
` repositoryService.activateProcessDefinitionById(processDefinitionId, true, null); `

流程模型的挂起
` repositoryService.suspendProcessDefinitionById(processDefinitionId, true, null); `
第二个参数就是用来控制在挂起流程定义时，是否同时挂起对应的流程实例。同样的，流程实例也有激活和挂起，这里不做详细说明，参看runtimeService介绍。

其实repositoryService的常用方法并不止上面那些。比如流程模型的创建，设置，删除等等。这方面没有使用过，因为模型设计主要通过的是Activiti Modeler。当然如果想要自己设计一套流程模型界面的话，repositoryService中的很多方法，你需要详细研究下。



