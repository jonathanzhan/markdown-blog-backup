---
layout: spring
title: Spring IOC容器的实现
date: 2018-06-11 11:27:17
tags:
- Spring core
categories:
- Spring
---
## 什么是IOC/DI？
IOC(Inversion of Control)控制反转，就是把原来我们代码里需要实现的对象创建，依赖的代码，反转给容易来实现。简单的说就是将原始类A使用类B时需要创建类B的操作交给容器来创建。
在传统的开发模式中对象之间是互相依赖的，但是在IOC开发模式中，IOC容器来安排对象之间的依赖。
IOC的另外名字叫做依赖注入(dependency Injection),所谓的依赖注入，就是由IOC容器在运行期间，动态的将某种依赖关系注入到对象之中。所以依赖注入(DI)和控制反转(IOC)是从不同的角度描述的同一件事情，就是指通过引入IOC容器，利用依赖关系注入的方式，实现对象之间的解耦。

## 注入方法
Spring依赖注入的方式有:
* set注入方式
* 静态工厂注入方式
* 构造方法注入方式
* 基于注解的注入方式

### set注入方式
具体代码：
```java
 private UserDao userDao;

 public void setUserDao(UserDao userDao) {
     this.userDao = userDao;
 }
```

Spring的配置文件:其中配置声明的userService类存在属性：userDao,程序在运行时，会将已经实例化的userDao对象调用setUserDao方法注入。
```xml
<bean name = "userService" class="xxxxx.UserService">
    <property name="userDao" ref="userDao"/>
</bean>

<bean name = "userDao" class="XXXXX.UserDao"/>
```

### 构造器注入方式
具体代码：
```java
private UserDao userDao;

public UserService(UserDao userDao) {
    this.userDao = userDao;
}
```
Spring配置文件:
```xml
<bean name = "userService" class="xxxxx.UserService">
    <constructor-arg ref="userDao" />
</bean>

<bean name = "userDao" class="XXXXX.UserDao"/>
```

### 基于注解方式(推荐使用，比较便捷)
```java
@Service("userService")
public class UserService {
    @Autowired
    //@Resource
    private UserDao userDao;
}
```

## @Autowired与@Resource的区别
1. 两个都可以用来装配bean,都可以写在字段上，或者写在setter方法上
2. @Autowired默认按照类型装配，默认情况下必须要求依赖对象必须存在。如果允许null值，可以设置他的required属性为false.
3. @Resource注解属于J2EE的，默认按照名称进行装配，名称可以通过name属性来指定。如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称进行查找。如果注解写在setter方法上，默认取属性名进行装配。当找不到与名称匹配的bean时，会按照类型进行装配。但需要注意的是，如果name属性一旦指定，就只会按照名称进行匹配。

