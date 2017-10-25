---
title: thymeleaf进阶使用
date: 2017-10-25 11:28:50
tags:
- thymeleaf
categories:
- java
---
thymeleaf毕竟是一个脚本语言,在生成html时有一些特殊的字符串需要通过特定的拼接才能完成，本文主要介绍下在thymeleaf下字符串的常用操作
<!-- more -->

[TOC]

## 字符串拼接
```html
<span th:text="'Welcome to our application, ' + ${user.name} + '!'">
<span th:text="|Welcome to our application, ${user.name}!|">
<span th:text="${onevar} + ' ' + |${twovar}, ${threevar}|">
```

## url地址拼接
```html
<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3&name=asd' (plus rewriting) -->
<a href="details.html"
   th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id},name=${o.name})}">view</a>
<!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>
<!-- Will produce '/user/3/details?name=123&type=1' (plus rewriting) -->
<a href="details.html" th:href="@{/user/{id}/details(id=${user.id},name=${user.name},type=${user.userType})}">view</a>
<!-- /details/123?orderId=1 -->
<a th:href="@{'/details/'+${user.login}(orderId=${o.id})}">view</a>
```
## 判断
```html
<span th:text="*{age}?: '(no age specified)'">27</span
<span th:text="*{age != null}? *{age} : '(no age specified)'">27</span>
```

以前两种是等价的 ?:表示在不等于null的情况下，输出值为判断条件,否则为定义的默认值 
具体参考上篇文章

* If-then: (if) ? (then)
* If-then-else: (if) ? (then) : (else)
* Default: (value) ?: (defaultvalue)


## appending 和prepending
```html
<input type="button" value="Do it!" class="btn" th:attrappend="class=${' ' + cssStyle}" />
<!-- If you process this template with the cssStyle variable set to "warning" , you will get: -->
<input type="button" value="Do it!" class="btn warning" />

<tr th:each="prod : ${prods}" class="row" th:classappend="${prodStat.odd}? 'odd'">

```

## th:remove
当html作为模板使用时，会根据模板中定义的变量等等来生成html页面，但是如果直接作为静态页面来查看,他的显示效果并不好，所以出现了th:remove的标签
例如：
```html
<table> <tr>
    <th>NAME</th>
    <th>PRICE</th>
    <th>IN STOCK</th>
    <th>COMMENTS</th>
  </tr>
  <tr th:each="prod : ${prods}" th:class="${prodStat.odd}? 'odd'">
    <td th:text="${prod.name}">Onions</td>
    <td th:text="${prod.price}">2.41</td>
    <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    <td>
      <span th:text="${#lists.size(prod.comments)}">2</span> comment/s
      <a href="comments.html"
         th:href="@{/product/comments(prodId=${prod.id})}"
         th:unless="${#lists.isEmpty(prod.comments)}">view</a>
    </td>
  </tr>
</table>
```
在作为静态页面显示时，并没有显示任何数据。为了前端的一些显示，我们需要在静态展示时，增加一些数据，则可以这样做
```html
<table>
  <thead>
    <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
      <th>COMMENTS</th>
    </tr>
  </thead>
  <tbody th:remove="all-but-first">
    <tr th:each="prod : ${prods}" th:class="${prodStat.odd}? 'odd'">
      <td th:text="${prod.name}">Onions</td>
      <td th:text="${prod.price}">2.41</td>
      <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
      <td>
        <span th:text="${#lists.size(prod.comments)}">2</span> comment/s
        <a href="comments.html"
           th:href="@{/product/comments(prodId=${prod.id})}"
           th:unless="${#lists.isEmpty(prod.comments)}">view</a>
      </td>
    </tr>
    <tr class="odd">
      <td>Blue Lettuce</td>
      <td>9.55</td>
      <td>no</td>
      <td>
        <span>0</span> comment/s
      </td>
</tr> <tr>
      <td>Mild Cinnamon</td>
      <td>1.99</td>
      <td>yes</td>
      <td>
        <span>3</span> comment/s
        <a href="comments.html">view</a>
      </td>
    </tr>
  </tbody>
</table>
```

th:remove的具体作用如下:

* all-but-first 删除所有包含标签的孩子,除了第一个。
* all 删除包含标签和所有的孩子。
* body 不包含标记删除，但删除所有的孩子
* tag 包含标记删除,但是不删除他的孩子
* none 什么也不做

th:remove属性可以采取任何Thymeleaf标准表达式,只要允许它返回一个字符串值(all, tag, body, all-but-first or none),意味着删除可能是有条件的

```html
<a href="/something" th:remove="${condition}? tag : none">Link text not to be removed</a>

<a href="/something" th:remove="${condition}? tag">Link text not to be removed</a>
```

## 注释

### 标准 HTML/XML注释  

语法：` <!--     --> `

### 解析器级注释块（Parser-level comment blocks）
语法：` <!--/*    */--> `
thymeleaf解析时会移除代码    
单行： ` <!--/*  xxxxx  */-->  `  
多行： 
```html
<!--/*-->          
     Xxxxxx          
     Xxxxxx         
<!--*/-->
```
### 针对原型的注释
语法：` <!--/*/    /*/--> ` 

```html
<span>hello!</span>
<!--/*/
  <div th:text="${...}">
    ...
  </div>
/*/-->
<span>goodbye!</span>

```


thymealeaf解析时会移除掉此标签对，但不会移除其中的内容。解析完成后如下

```html
<span>hello!</span>
 
  <div th:text="${...}">
    ...
  </div>
 
<span>goodbye!</span>

```

### th:block
thymealeaf解析时会移除掉此标签对，但不会移除其中的内容。
th:block是一个属性容器，允许模板开发人员指定他们想要的任何属性。Thymeleaf将执行这些属性，然后简单地制作块，而不是其内容消失。

```html
<table>
  <th:block th:each="user : ${users}">
    <tr>
        <td th:text="${user.login}">...</td>
        <td th:text="${user.name}">...</td>
    </tr>
    <tr>
        <td colspan="2" th:text="${user.address}">...</td>
    </tr>
  </th:block>
</table>
```

## 内联inline

内联：th:inline，值有三种：text,javascript,none

### 文本内联
```html
<p th:inline="text">Hello, [[${session.user.name}]]!</p>
```

### 脚本内联
```javascript
<script th:inline="javascript">
var user = [[${user.username}]];
alert(user);

var msg  = 'Hello, ' + [[${user.username}]];
alert(msg);

var username = /*[[${session.user.name}]]*/ "Gertrud Kiwifruit";
</script>    
```
/*[[${session.user.name}]]*/在thymeleaf解释器解析代码时会解析里面的[[${session.user.name}]]

加载静态页时会解析注释后面的语句 var username = 'Gertrud Kiwifruit';

js内联代码中需要加入/*<![CDATA[*/    ......    /*]]>*/代码块，thymeleaf才能正确解析一些运算符(<等)和操作符号&/&&等。

javascript内联时有以下两个特性:

* javascript附加代码
语法：` /*[+   +]*/  `

```javascript
/*[+
var msg  = 'This is a working application';
+]*/
```

* javascript移除代码
语法：` /*[- */    /* -]*/`
```javascript
/*[- */
var msg  = 'This is a non-working template';
/* -]*/
```

### 样式内联

```css
<style th:inline="css">
    .[[${classname}]] {
      text-align: [[${align}]];
    }
</style>
```
两个参数的值分别是
classname = 'main elems'
align = 'center'

and result is 
```css
<style th:inline="css">
    .main\ elems {
      text-align: center;
    }
</style>
```

