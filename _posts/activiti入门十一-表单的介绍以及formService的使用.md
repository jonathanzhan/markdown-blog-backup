---
title: activiti入门十一(表单的介绍以及formService使用)
date: 2016-11-01 17:32:43
tags:
 - activiti
categories:
 - activiti
toc: true
---
本来这章是要讲解FormService的使用，在准备资料的过程中，发现还是需要先介绍下activiti的不同表单类型的应用。同时附带把FormService中常用的操作做个介绍
在Activiti中总共有三种表单，动态表单，普通表单和外置表单。
<!-- more -->

### 动态表单

#### 流程定义文件的代码
先看下流程定义文件(bpmn20.xml)的部分代码
```xml
<startEvent activiti:initiator="applyUserId" id="start" name="start">
  <extensionElements>
    <activiti:formProperty datePattern="yyyy-MM-dd" id="startDate" name="请假开始日期" required="true" type="date"/>
    <activiti:formProperty datePattern="yyyy-MM-dd" id="endDate" name="请假结束日期" required="true" type="date"/>
    <activiti:formProperty id="reason" name="请假原因" required="true" type="string"/>
  </extensionElements>
</startEvent>
<userTask activiti:assignee="admin" activiti:exclusive="true" id="deptLeaderAudit" name="部门领导审批">
  <extensionElements>
    <activiti:formProperty datePattern="yyyy-MM-dd" id="startDate" name="请假开始日期" type="date" writable="false"/>
    <activiti:formProperty datePattern="yyyy-MM-dd" id="endDate" name="请假结束日期" type="date" writable="false"/>
    <activiti:formProperty id="reason" name="请假原因" type="string" writable="false"/>
    <activiti:formProperty id="deptLeaderPass" name="审批意见" required="true" type="enum">
      <activiti:value id="true" name="同意"/>
      <activiti:value id="false" name="不同意"/>
    </activiti:formProperty>
  </extensionElements>
</userTask>
```

用activiti:formProperty属性定义，可以在开始事件(startEvent)和任务(Task)上设置表单的动态内容，表单的内容都是以key和value的形式数据保存在引擎表中
因此，动态表单的布局在顺序的显示在页面上，基本上毫无布局可言，但是同时这种方式对于不需要布局的流程，在开发和实施的过程中，是最为方便的，我在写activiti的各种demo时，最常用的就是动态表单。

#### 获取动态表单的定义内容
那么问题来了，如何去获取在流程定义中定义的表单内容。activiti提供了FormService的API操作可以获取到流程启动节点的表单内容以及任务节点的表单内容
##### 根据流程定义Id获取流程启动节点的表单内容
` StartFormData getStartFormData(String processDefinitionId);`
` StartFormData startFormData = formService.getStartFormData(流程定义id)`
查看startFormData的实体类定义，可以发现它集成的父类是FormData，而FormData中有一项属性List<FormProperty> formProperties
以下是FormProperty的源码结构
```java
public interface FormProperty extends Serializable {
  String getId();

  String getName();

  FormType getType();

  String getValue();

  boolean isReadable();

  boolean isWritable();

  boolean isRequired();
}
```
看到这里，我们已经可以知道流程启动节点的表单内容了,那么接下来根据这些内容去维护一个公共的动态表单页面即可。

可能在写页面代码时，会碰到一个问题，就是日期格式。因为我们在流程定义文件中是有维护日期格式的，但是在FormProperty中并没有这一属性项。
同时还有枚举类型的枚举项。
参看以下代码
```java
@RequestMapping(value = "startForm/{procDefId}")
public String startForm(@PathVariable(value = "procDefId") String procDefId, Model model) {

    Map<String, Map<String, String>> result = Maps.newHashMap();
    Map<String,String> datePatterns = Maps.newHashMap();
    StartFormData startFormData = formService.getStartFormData(procDefId);
    List<FormProperty> formProperties = startFormData.getFormProperties();
    for (FormProperty formProperty : formProperties) {
    	if("enum".equals(formProperty.getType().getName())){
    		Map<String, String> values;
    		values = (Map<String, String>) formProperty.getType().getInformation("values");
    		if (values != null) {
    			for (Map.Entry<String, String> enumEntry : values.entrySet())
    				logger.debug("enum, key: {}, value: {}", enumEntry.getKey(), enumEntry.getValue());
    			result.put("enum_" + formProperty.getId(), values);
    		}

    	}else if("date".equals(formProperty.getType().getName())){
    		datePatterns.put("pattern_"+formProperty.getId(), (String)formProperty.getType().getInformation("datePattern"));
    		logger.debug("date,key:{},pattern:{}",formProperty.getId(),formProperty.getType().getInformation("datePattern"));
    	}

    }
    model.addAttribute("datePatterns",datePatterns);
    model.addAttribute("result", result);
    model.addAttribute("list", formProperties);
    model.addAttribute("formData", startFormData);

    return "modules/act/dynamicStartForm";
}
```
这段代码是获取动态表单启动界面的信息。其中` (String)formProperty.getType().getInformation("datePattern") `是获取日期格式的，` (Map<String, String>) formProperty.getType().getInformation("values"); `是获取枚举类型的枚举项的
接下来就是页面的事情了。
##### 根据taskId获取流程任务中的表单内容

` TaskFormData getTaskFormData(String taskId); `
FormService中的这个API方法，提供了流程任务中的表单内容的定义。
上面详细讲解了FormProperty的使用，那么相似的，我们可以按照类似的操作获取到对应的信息

#### 动态表单定义额外的变量类型
在流程定义文件中，每个字段都有一个指定的类型，目前Activiti默认支持的类型有String,long,enum,date,boolean,collection。而在实际的应用中，默认的类型往往是不够用的，如果需要其他类型的字段，我们该如何处理呢？activiti是允许自定义表单字段类型的。具体实现如下:
`<activiti:formProperty id="user" name="用户名" required="true" type="users"/>`
比如，我们定义一个一个类型是users，我们的需求是可以输入多个用户名，以逗号隔开。
接下来，去定义一个表单类型users的解析类，用来转换表单值。

##### 定义表单类型的解析类
**注意所有自定义的表单字段都需要继承一个表达类型抽象类"org.activiti.engine.form.AbstractFormType"**

```Java
/**
 * 多用户的表单类型,loginName中间以','隔开
 *
 * @author Jonathan
 * @version 2016/9/22 17:00
 * @since JDK 7.0+
 */
public class UsersFormType extends AbstractFormType {

	/**
	 * 把字符串的值转换为集合对象
	 * @param propertyValue
	 * @return
	 */
	@Override
	public Object convertFormValueToModelValue(String propertyValue) {
		String[] split = StringUtils.split(propertyValue, ",");
		return Arrays.asList(split);
	}

	/**
	 * 把集合对象的值转换为字符串
	 * @param modelValue
	 * @return
	 */
	@Override
	public String convertModelValueToFormValue(Object modelValue) {
		if(modelValue==null){
			return null;
		}
		return modelValue.toString();
	}

  /**
	 * 定义表单类型的标识符
	 * @return
	 */
	@Override
	public String getName() {
		return "users";
	}
}
```

##### 在流程引擎中注册解析类
自定义表单类型的解析类定义完成后，还需要在流程引擎中注册，否则引擎会提示未找到对应的类型转换类。注册表单字段类型的配置如下:
```xml
<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
  ......
  <!-- 自定义表单字段类型 -->
	<property name="customFormTypes">
		<list>
			<bean class="com.mfnets.workfocus.modules.act.utils.UsersFormType"/>
		</list>
	</property>

</bean>  
```

到此，我们就可以使用名为users的表单字段类型了。

#### 动态表单的流程启动和处理
在上章RuntimeService介绍中，提到了一个FormService中的两个API操作:
` ProcessInstance submitStartFormData(String processDefinitionId, Map<String, String> properties); `
和
` ProcessInstance submitStartFormData(String processDefinitionId, String businessKey, Map<String, String> properties); `
都是用来启动流程的。
其中最后一个参数Map<String, String> properties则需要我们维护流程启动中自定义的表单内容的key值和value值。


同理，动态表单的任务处理，也是类似的
` void submitTaskFormData(String taskId, Map<String, String> properties); `
` void saveFormData(String taskId, Map<String, String> properties); `

### 普通表单
#### 流程定义代码
```xml
<startEvent id="begin" name="请假申请" activiti:initiator="applyUserId" activiti:formKey="/demo/leave/startForm"></startEvent>
<userTask id="leaderAudit" name="部门经理审批" activiti:candidateGroups="test" activiti:formKey="/demo/leave/completeForm"></userTask>
```
其中activiti:formKey就是来定义我们启动界面或者某个任务环节处理界面的对应的表单标识的key
所以，普通表单的启动界面和任务处理界面都是需要我们自己来定义维护的，因此布局肯定比动态表单要优美很多。

那么如何去获取activiti:formKey中对应的内容呢，仍旧是FormService提供了对应的API
` String getStartFormKey(String processDefinitionId) `获取流程启动的formKey
` String getTaskFormKey(String processDefinitionId, String taskDefinitionKey); `获取任务环节的formKey

至于具体的流程启动和任务处理的实现，参看RuntimeService和TaskService的介绍。

### 外置表单
这种方式常用于基于工作流平台开发的方式，代码写的很少，开发人员只要把表单内容写好保存到.form文件中即可，然后配置每个节点需要的表单名称（form key），实际运行时通过引擎提供的API读取Task对应的form内容输出到页面。
