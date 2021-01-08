---
title: spring面试题
date: 2018-06-11 11:27:17
thumbnail: /files/thumb/4.jpg
tags:
- Spring
categories:
- Spring
toc: true
---

### 什么是IOC/DI？

IOC(Inversion of Control)控制反转，就是把原来我们代码里需要实现的对象创建，依赖的代码，反转给容器来实现,同时需要一种描述来让容器知道需要创建的对象与对象的关系。简单的说就是将原始类A使用类B时需要创建类B的操作交给容器来创建。
在传统的开发模式中对象之间是互相依赖的，但是在IOC开发模式中，IOC容器来安排对象之间的依赖。

IOC的另外名字叫做依赖注入(dependency Injection),所谓的依赖注入，就是由IOC容器在运行期间，动态的将某种依赖关系注入到对象之中。所以依赖注入(DI)和控制反转(IOC)是从不同的角度描述的同一件事情，就是指通过引入IOC容器，利用依赖关系注入的方式，实现对象之间的解耦。

依赖注入的方式有三种:构造器注入,setter方法注入,接口注入

### 解释Spring框架中的IOC

Spring IOC负责创建对象,管理对象(通过依赖注入),配置对象,并管理这些对象的生命周期.
Spring中的org.springframework.beans包和org.springframework.context包构成了Spring框架IOC容器的基础.
BeanFactory是Spring IOC容器的具体实现,用来管理对象.
IOC把对象的创建,初始化,销毁交给Spring管理.而不是开发者管理,从而实现控制反转.

### BeanFactory 和ApplicationContext的区别

beanfactory可以理解为含有bean集合的工厂,包含了各种bean的定义,以便在接收到请求时将对应的bean实例化.
applicationContext继承beanfactory,除了beanfactory的功能以外,还有其他的功能,比如国际化消息文本,资源文件读取,监听器中注册bean的事件等.

### Spring的几种配置方式

1. 基于XML的配置
2. 基于注解的配置
3. 基于java的配置

Spring对java配置的支持主要是由@Configuration和@bean注解来实现的.
由@bean注解的方法将会实例化,配置和初始化一个新的对象,这个对象将给IOC容器去管理.
@Configuration所注解的类主要目的是作为bean定义的资源.

### Spring bean的生命周期

1. Spring容器从配置中读取bean的定义,并实例化bean
2. Spring根据bean的定义填充属性
3. 如果bean实现了BeanNameAware接口,Spring传递bean的id到SetBeanName方法
4. 如果bean实现了BeanFactoryAware接口,Spring传递beanFactory给setBeanFactory
5. 如果bean实现了beanPostProcessors接口,Spring会在postProcessorBeforeInitialzation()方法调用
6. 如果bean实现了IntializingBean,调用afterPropertySet方法.
7. 调用初始化方法init
8. 如果有任何与bean关联的beanPostProcessors,这些bean的postProcessorAfterInitialzation()方法将被调用
9. 如果bean实现了DisposeableBean,将调用destory()方法
10. 调用指定销毁方法customerDestory

Spring 框架提供了四种方式来管理bean的生命周期

* InitializingBean和disposeableBean回调接口
* 针对特殊行为的Aware接口
* bean配置文件中定义的Custom init()方法和destory()方法
* @PostConstruct 和@PreDestory注解方法

bean的生命周期由两组回调方法组成

1. 初始化之后调用的回调方法
2. 销毁之前调用的回调方法

### Spring bean的作用域之间的区别

1. singleton: 这种bean范围是默认的.这种范围确保不管接收到多少请求,每个容器中只有一个bean的实例.
2. prototype: 原型范围与单例范围相反,为每一个bean请求提供一个实例.
3. request:在请求范围内,每一个来自客户端网络请求都会创建一个实例,在请求完成之后,bean失效并被垃圾回收器回收.
4. session:与request相似,是在一个session中有一个bean的实例,session过期后,bean失效.
5. global-session:在一个全局的 HTTP Session 中，一个bean定义对应一个实例.仅在使用prtlet context的时候有效,该作用域仅在基于web的SpringApplicationContext的情形下有效.

### 什么是bean装配

bean装配是指在Spring容器中把bean组装到一起,前期是Spring容器知道bean之间的依赖关系.
Spring容器还可以通过bean工厂自动装配bean之间的关联关系

自动装配的方式有:

* no:默认方式是不进行自动装配,通过显示设置ref属性来进行装配
* byName:通过设置autowired =byName 参数名自动装配,之后容器视图匹配,装配和该bean的属性具有相同名称的bean
* byType:通过参数类型自动装配，Spring 容器在配置文件中发现bean的 autowire 属性被设置成 byType， 之后容器试图匹配、装配和该bean的属性具有相同类型的 bean
* constructor:类似于byType,但是要提供构造函数
* autodetect:首先尝试用constructor装配,如果无法工作,则使用byType

开启基于注解的自动装配@Autowired,需要注册AutowiredAnnotationBeanPostProcessor.
<bean>
<context:annotation-config />
</bean>

### @Required注解

这个注解表明 bean 的属性必须在配置的时候设置，通 过一个 bean 定义的显式的属性值或通过自动装配，若 @Required 注解的 bean 属性未被设置，容器将抛出 BeanInitializationException。

### @Autowired 注解

@Autowired 注解对自动装配何时何处被实现提供了更多细粒度的控制。修饰setter方法,构造器,属性

### @Qualifier 注解

当有多个相同类型的bean,但是只有一个需要自动装配时,将@Qualifier注解和@Autowire注解结合使用以消除这种混淆，指定需要装配的确切的bean

### AOP
面向切面编程.它是一种方法论,AOP的主要编程对象是切面aspect.比如日志和事务管理.

AOP 核心就是切面，它将多个类的通用行为封装成可重用的模块，该模块含有一组API提供横切功能.
比如， 一个日志模块可以被称作日志的 AOP 切面。根据需求的 不同，一个应用程序可以有若干切面
在 Spring AOP 中，切面通过带有@Aspect 注解的类实现。

切面(Aspect):一个关注点的模块化，这个关 注点可能会横切多个对象。事务管理是 J2EE应 用中一个关于横切关注点的很好的例子。 在 Spring AOP 中，切面可以使用通用类(基于模 式的风格) 或者在普通类中以 @Aspect 注解 (@AspectJ 风格)来实现。

关注点是应用中一个模块的行为，一个关注点可能会被 定义成一个我们想实现的一个功能。

连接点(Joinpoint):连接点代表一个应用程序的某个位置，在这个位置我们 可以插入一个 AOP 切面，它实际上是个应用程序执行 Spring AOP 的位置。

切入点(Pointcut):匹配连接点 (Joinpoint)的断言。通知和一个切入点表达 式关联，并在满足这个切入点的连接点上运行 (例如，当执行某个特定名称的方法时)。 切 入点表达式如何和连接点匹配是 AOP的核心: Spring 缺省使用 AspectJ 切入点语法。

通知是个在方法执行前或执行后要做的动作，实际上是 程序执行时要通过 SpringAOP 框架触发的代码段。

Spring切面通知有:before,after,after returning,after throwing,around

切点:切入点是一个或一组连接点，通知将在这些位置执行。 可以通过表达式或匹配的方式指明切入点。

引入允许我们在已存在的类中增加新的方法和属性

目标对象:被一个或者多个切面所通知的对象。它通常是一个代理 对象。也指被通知(advised)对象。

### Spring MVC 运行流程

1. 用户发送请求到DispatchServlet
2. DispatchServlet根据请求路径查询具体的Handler
3. HandlerMapping返回一个HandlerExcutionChain给DispatchServlet HandlerExcutionChain:Handler和 Interceptor集合
4. DispatchServlet调用HandlerAdapter适配器
5. HandlerAdapter调用具体的Handler处理业务
6. Handler处理结束返回一个具体的ModelAndView给适配器
7. 适配器将ModelAndView给DispatchServlet
8. DispatchServlet把视图名称给ViewResolver视图解析器
9. ViewResolver返回一个具体的视图给DispatchServlet
10. 渲染视图
11. 展示给用户

### Spring事务隔离级别

1. ISOLATION_DEFAULT：用底层数据库的默认隔离级别，数据库管理员设置什么就是什么
2. ISOLATION_READ_UNCOMMITTED（未提交读）：最低隔离级别、事务未提交前，就可被其他事务读取（会出现幻读、脏读、不可重复读）
3. ISOLATION_READ_COMMITTED（提交读）：一个事务提交后才能被其他事务读取到（该隔离级别禁止其他事务读取到未提交事务的数据、所以还是会造成幻读、不可重复读）、sql server默认级别
4. ISOLATION_REPEATABLE_READ（可重复读）：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（该隔离基本可防止脏读，不可重复读（重点在修改），但会出现幻读（重点在增加与删除））（MySql默认级别，更改可通过set transaction isolation level 级别）
5. ISOLATION_SERIALIZABLE（序列化）：代价最高最可靠的隔离级别（该隔离级别能防止脏读、不可重复读、幻读）

### Spring传播行为

* PROPAGATION_REQUIRED：支持当前事务，如当前没有事务，则新建一个。
* PROPAGATION_SUPPORTS：支持当前事务，如当前没有事务，则以非事务性执行（源码中提示有个注意点，看不太明白，留待后面考究）。
* PROPAGATION_MANDATORY：支持当前事务，如当前没有事务，则抛出异常（强制一定要在一个已经存在的事务中执行，业务方法不可独自发起自己的事务）。
* PROPAGATION_REQUIRES_NEW：始终新建一个事务，如当前原来有事务，则把原事务挂起。
* PROPAGATION_NOT_SUPPORTED：不支持当前事务，始终已非事务性方式执行，如当前事务存在，挂起该事务
* PROPAGATION_NEVER：不支持当前事务；如果当前事务存在，则引发异常。
* PROPAGATION_NESTED：如果当前事务存在，则在嵌套事务中执行，如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操

### 如何自定义注解实现功能

可以结合 spring 的 AOP，对注解进行拦截，提取注解。
大致流程为:
1. 新建一个注解@MyLog，加在需要注解申明的方法上面
2. 新建一个类 MyLogAspect，通过@Aspect 注解使该类成为切面类。
3. 通过@Pointcut 指定切入点 ，这里指定的切入点为 MyLog 注解类型，也就是被@MyLog 注解修饰的方法，进入该切入点。
4. MyLogAspect 中的方法通过加通知注解(@Before、@Around、@AfterReturning、 @AfterThrowing、@After 等各种通知)指定要做的业务操作。
