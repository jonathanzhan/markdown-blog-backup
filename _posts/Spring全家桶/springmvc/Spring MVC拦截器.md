---
title: SpringMvc拦截器
date: 2016-07-07 13:02:01
tags:
- SpringMvc
categories:
- Spring
toc: true
---
## 简介
SpringMVC 中的Interceptor 拦截器也是相当重要和相当有用的，它的主要作用是拦截用户的请求并进行相应的处理。比如通过它来进行权限验证，或者是来判断用户是否登陆,或者来监控系统的性能等等。
<!-- more -->
## 常见的应用场景
 1. 日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。
 2. 权限验证，如登录检测，如果没有登录，则返回登录页面。
 3. 性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；
 4. 通用行为的记录，提取Locale、Theme信息等，只要是多个处理器都需要的即可使用拦截器实现。

## 拦截器接口HandlerInterceptor

```java
public interface HandlerInterceptor {
	boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception;
	
	void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception;
}
```
HandlerInterceptor 接口中定义了三个方法，我们就是通过这三个方法来对用户的请求进行拦截处理的。

**preHandle**:
	该方法将在请求处理之前进行调用。SpringMVC 中的Interceptor 是链式的调用的，在一个应用中或者说是在一个请求中可以同时存在多个Interceptor 。每个Interceptor 的调用会依据它的*声明顺序依次执行*，而且最先执行的都是Interceptor 中的preHandle 方法
	在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。该方法的返回值是布尔值Boolean 类型的，当它返回为false 时，表示请求结束，后续的Interceptor 和Controller 都不会再执行；当返回值为true 时就会继续调用下一个Interceptor 的preHandle 方法，如果已经是最后一个Interceptor 的时候就会是调用当前请求的Controller 方法。

**postHandle**:
后处理回调方法，在当前请求进行处理之后，也就是Controller 方法调用之后执行，但是它会在DispatcherServlet 进行视图返回渲染之前被调用，所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作
*postHandle 方法被调用的方向跟preHandle 是相反的，也就是说先声明的Interceptor 的postHandle 方法反而会后执行*

**afterCompletion**:
该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中preHandle返回true的拦截器的afterCompletion。

## 拦截器适配器HandlerInterceptorAdapter
有时候我们可能只需要实现三个回调方法中的某一个，如果实现HandlerInterceptor接口的话，三个方法必须实现，不管你需不需要.
因此spring提供了一个HandlerInterceptorAdapter适配器，允许我们只实现需要的回调方法。

```java
public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {
		return true;
	}

	@Override
	public void postHandle(
			HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
			throws Exception {
	}

	@Override
	public void afterCompletion(
			HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
	}

	@Override
	public void afterConcurrentHandlingStarted(
			HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
	}

}
```
AsyncHandlerInterceptor提供了一个afterConcurrentHandlingStarted()方法, 这个方法会在Controller方法异步执行时开始执行。而Interceptor的postHandle方法则是需要等到Controller的异步执行完才能执行

## DispatcherServlet的内部实现
具体参看方法

```java
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response)


void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv)

void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, Exception ex)

void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) 

```

## 简单实现

```java
public class HandlerInterceptor1 extends HandlerInterceptorAdapter{
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		System.out.println("===========HandlerInterceptor1 preHandle");
		return true;
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
		System.out.println("===========HandlerInterceptor1 postHandle");
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
		System.out.println("===========HandlerInterceptor1 afterCompletion");
	}

	@Override
	public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		System.out.println("===========HandlerInterceptor1 afterConcurrentHandlingStarted");
	}
}
```

## Spring MVC 拦截器配置
在SpringMVC的配置文件中加上支持MVC的schema

```xml
xmlns:mvc="http://www.springframework.org/schema/mvc"
xsi:schemaLocation="http://www.springframework.org/schema/mvc  http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd"
```
这样在SpringMVC的配置文件中就可以使用mvc标签了，mvc标签中有一个mvc:interceptors是用于声明SpringMVC的拦截器的。

使用mvc:interceptors标签来声明需要加入到SpringMVC拦截器链中的拦截器
```xml
<!--配置拦截器, 多个拦截器,顺序执行 -->
<mvc:interceptors>  
	<mvc:interceptor>  
		<!-- 匹配的是url路径， 如果不配置或/**,将拦截所有的Controller -->
		<mvc:mapping path="/" />
		<mvc:mapping path="/user/**" />
		<mvc:mapping path="/test/**" />
		<bean class="com.whatlookingfor.HandlerInterceptor1"/> 
	</mvc:interceptor>
	<!-- 当设置多个拦截器时，先按顺序调用preHandle方法，然后逆序调用每个拦截器的postHandle和afterCompletion方法 -->
</mvc:interceptors>
```
