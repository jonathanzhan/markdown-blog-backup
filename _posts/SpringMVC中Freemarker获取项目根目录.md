---
title:  SpringMVC中Freemarker获取项目根目录
date: 2016-05-30 16:34:54
tags:
- SpringMvc
categories:
- Spring
toc: true
---
在SpringMVC框架中使用Freemarker试图时，要获取根路径的方式如下:
<!-- more -->
```xml
<!-- FreeMarker视图解析 如返回userinfo。。在这里配置后缀名ftl和视图解析器。。 -->
<bean id="viewResolverFtl"
    class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="viewClass"
        value="org.springframework.web.servlet.view.freemarker.FreeMarkerView" />
    <property name="suffix" value=".ftl" />
    <property name="contentType" value="text/html;charset=UTF-8" />
    <property name="exposeRequestAttributes" value="true" />
    <property name="exposeSessionAttributes" value="true" />
    <property name="exposeSpringMacroHelpers" value="true" />
    <property name="requestContextAttribute" value="request" />
    <property name="cache" value="true" />
    <property name="order" value="0" />
</bean>
```
其中**property name="requestContextAttribute" value="request"**是关键。
意思是把Spring的RequestContext对象暴露为变量request
利用${request.contextPath}来获取应用程序的contextPath

如果是集成了Springboot，在配置文件中，只需要设置
spring.freemarker.request-context-attribute=request 即可


ftl中的页面设置如下：
```html
<#assign ctx=request.contextPath />
<!DOCTYPE html>
<html lang="zh">
<head>
    <base id="ctx" href="${ctx}">
    <title>首页</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <link href="${ctx}/static/bootstrap-3.3.4/css/bootstrap.min.css" rel="stylesheet">
    <script src="${ctx}/static/bootstrap-3.3.4/js/bootstrap.min.js"></script>
```
js文件中获取path
```js
var base = document.getElementById("ctx").href;
// 与后台交互
$.ajax({
        url : base + '/' + url,
        data : value,
        dataType : 'json',
        type : 'post',
        success : function(data) {
            success(data);
        },
        error : function(data) {
            error(data);
        }
    });

```
