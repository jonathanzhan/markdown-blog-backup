---
title: thymeleaf中的fragment使用
date: 2017-10-24 18:21:59
tags:
- thymeleaf
categories:
- java
---
## fragment介绍
fragment类似于JSP的tag，在html中文件中，可以将多个地方出现的元素块用fragment包起来使用。
## fragment使用

### 定义fragment

所有的fragment可以写在一个文件里面，也可以单独存在，例如
```html
<footer th:fragment="copy">  
   the content of footer 
</footer>
```

### fragment的引用
1. th:insert:保留自己的主标签，保留th:fragment的主标签。
2. th:replace:不要自己的主标签，保留th:fragment的主标签。
3. th:include:保留自己的主标签，不要th:fragment的主标签。（官方3.0后不推荐）

``` html
导入片段：
<div th:insert="footer :: copy"></div>  
  
<div th:replace="footer :: copy"></div>  
  
<div th:include="footer :: copy"></div>

结果为:
<div>  
    <footer>  
       the content of footer   
    </footer>    
</div>    
  
<footer>  
    the content of footer 
</footer>    
  
<div>  
    the content of footer 
</div>  
```

在Springboot中，默认读取thymeleaf文件的路径是：src/main/resource/templates，静态文件为src/main/resource/static，这个默认值可以在配置文件中修改：spring.thymeleaf.prefix=classpath:/templates/

所有在调用fragment时，是默认从thymeleaf的根路径开始设置的：
例如
``` <head th:replace="/include/header::head" > ```
从读取templates/include/header.html中 fragment=head的代码块

### fragment的参数设置

```html
<div th:fragment="frag (onevar,twovar)">
    <p th:text="${onevar} + ' - ' + ${twovar}">...</p>
</div>


<div th:replace="::frag (${value1},${value2})">...</div>
<div th:replace="::frag (onevar=${value1},twovar=${value2})">...</div>
可以不按照参数定义的顺序
<div th:replace="::frag (twovar=${value2},onevar=${value1})">...</div>

```

### fragment的lexible layouts

定义(文件地址:include/header.html)：
```html
<head th:fragment="head(title,links,scripts)">  
<title th:replace="${title}">The awesome application</title>  
  
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />  
  
<th:block th:replace="${links}" />  
<th:block th:replace="${scripts}" />  
  
</head>  

```
调用：
```html
<head th:include="include/header :: head(~{::title},~{::link},~{::script})">   
<title>html的title</title>  
<link rel="stylesheet" th:href="@{/css/bootstrap.css}">  
<script th:src="@{/js/bootstrap.js}"></script>  
</head>  
```
注意是link 和script，不是links 和scripts
如果调用的页面没有link或者script ，则指定传入的参数为~{}即可。
```html
<head th:include="include/header :: head(~{::title},~{},~{})">   
```













