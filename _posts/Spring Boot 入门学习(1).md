---
title:  Spring Boot 入门学习(1)
date: 2016-05-12 17:43:43
tags:
- SpringBoot
categories:
- Spring
toc: true
---
### 简介
Spring官方网站本身使用Spring框架开发，随着功能以及业务逻辑的日益复杂，应用伴随着大量的XML配置文件以及复杂的Bean依赖关系。随着Spring 3.0的发布，Spring IO团队逐渐开始摆脱XML配置文件，并且在开发过程中大量使用“约定优先配置”（convention over configuration）的思想来摆脱Spring框架中各类繁复纷杂的配置（即时是Java Config）。

<!-- more	 -->
Spring Boot正是在这样的一个背景下被抽象出来的开发框架，它本身并不提供Spring框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于Spring框架的应用程序。也就是说，它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具。同时它集成了大量常用的第三方库配置（例如Jackson, JDBC, Mongo, Redis, Mail等等），Spring Boot应用中这些第三方库几乎可以零配置的开箱即用（out-of-the-box），大部分的Spring Boot应用都只需要非常少量的配置代码，开发者能够更加专注于业务逻辑。根据官网文档的描述,他的主要目标是

 * 为所有的Spring开发提供一个从根本上更快的和广泛使用的入门经验
 列表内容
 * 开箱即用,但你可以通过不采用默认设置来摆脱这种方式 。
 * 提供一系列大型项目常用的非功能性特征(比如,内嵌服务器，安全，指标，健康监测，外部化配置)。
 * 绝对不需要代码生成及xml配置
### 入门实例
创建一个maven的项目
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.3.5.RELEASE</version>
</parent>

<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```
*通过mvn dependency:tree 命令以树形的结构查看项目的依赖情况,其中已经包含了tomcat web服务器以及Spring boot自身*
编写一个类包含处理HTTP请求的方法以及一个main()函数：

```java
@RestController
@EnableAutoConfiguration
public class HelloWorldController {

	@RequestMapping(value = "helloWorld")
	public String helloWorld(){
		return "hello world";
	}

	public static void main(String[] args) {
		SpringApplication.run(HelloWorldController.class,args);
	}
}
```
启动main函数后，在控制台中可以发现启动了一个Tomcat容器，一个基于Spring MVC的应用也同时启动起来，这时访问http://localhost:8080就可以看到Hello World出现在浏览器中了。
	
@RestController与@RequestMapping注解是Spring MVC的常用注解。
@RestController

```java
@Target(value=TYPE)  
 @Retention(value=RUNTIME)  
 @Documented  
 @Controller  
 @ResponseBody  
public @interface RestController 
```

@RestController，它继承自@Controller注解，是spring 4.0引入的。查看源码可知其包含了 @Controller 和 @ResponseBody 注解。专门为响应内容式的 Controller 而设计的，可以直接响应对象为JSON(RestController下面的action返回都默认为ResponseBody)。 而 @Controller 用来响应页面

### @EnableAutoConfiguration注解
这个注解告诉Spring Boot根据添加的jar依赖猜测你想如何配置Spring。由于spring-boot-starter-web添加了Tomcat和Spring MVC，所以auto-configuration将假定你正在开发一个web应用并相应地对Spring进行设置。
Starter POMs和Auto-Configuration：设计auto-configuration的目的是更好的使用"Starter POMs"，但这两个概念没有直接的联系。你可以自由地挑选starter POMs以外的jar依赖，并且Spring Boot将仍旧尽最大努力去自动配置你的应用。

### main()方法
main方法通过调用run，将业务委托给了Spring Boot的SpringApplication类。SpringApplication将引导我们的应用，启动Spring，相应地启动被自动配置的Tomcat web服务器。我们需要将Example.class作为参数传递给run方法来告诉SpringApplication谁是主要的Spring组件。为了暴露任何的命令行参数，args数组也会被传递过去。

同样的，因为是个maven项目，我们也可以通过mvn spring-boot:run来启动应用。
