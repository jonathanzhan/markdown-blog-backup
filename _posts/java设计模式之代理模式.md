---
title: java设计模式之代理模式
date: 2019-08-29 14:12:04
tags:
- 设计模式
categories:
- 设计模式
toc: true
---
在有些情况下,客户不能或者不想直接访问另一个对象,这时需要找一个中介帮忙来完成某项任务,这个中介就是代理对象.比如租房子,不一定直接去找现房,可以找中介帮忙,找工作可以通过猎头等等.
<!-- more -->

### 代理模式的定义与特点

代理模式的定义:由于某些原因需要给某对象提供一个代理以控制对该对象的访问,这时访问对象不适合或者不能直接引用目标对象,代理对象作为访问对象和目标对象之间的中介.

代理模式的主要优点有:

* 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用.
* 代理对象可以拓展目标对象的功能
* 代理模式能将客户端与目标对象分离,在一定程序上降低了系统的耦合度.

其主要缺点是:

* 在客户端和目标对象之间增加了一个代理对象,会造成请求处理速度变慢
* 增加了系统的复杂度

### 静态代理

#### 静态代理模式的结构

代理模式的结构比较简单,主要是通过定义一个继承抽象主题的代理来包含真实主题,从而实现对真实主题的访问.

代理模式的主要角色如下:

1. 抽象主题(subject)角色:通过接口或抽象类声明真实主题和代理对象实现的业务方法.
2. 真实主题(real subject)角色:也叫被委托角色,被代理角色,实现了抽象主题中的具体业务,是代理对象所代表的真实对象,是最终要引用的对象.
3. 代理(proxy)角色:提供了与真实主题相同的接口,其内部含有对真实主题的引用,它可以访问、控制、扩展真实主题的功能,并在真实主题角色处理完毕前后做预处理和善后处理工作.

其结构图如下:
![代理模式结构图](/files/dp/proxy.png)

#### 静态代理模式的实现

```java
/**
 * 抽象主题
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 10:07
 * @since 1.0.0+
 */
public interface Subject {
    /**
     * 待具体实现的方法
     */
    void request();
}

/**
 * 真实主题对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 10:08
 * @since 1.0.0+
 */
public class RealSubject implements Subject {

    /**
     * 待具体实现的方法
     */
    @Override
    public void request() {
        System.out.println("访问真实主题方法");
    }
}

/**
 * 代理对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 10:09
 * @since 1.0.0+
 */
public class Proxy implements Subject {

    private RealSubject realSubject;

    /**
     * 待具体实现的方法
     */
    @Override
    public void request() {
        if(realSubject == null) {
            realSubject = new RealSubject();
        }
        preRequest();
        realSubject.request();
        postRequest();
    }

    private void postRequest() {
        System.out.println("访问真实主题以后的后续处理");
    }

    private void preRequest() {
        System.out.println("访问真实主题之前的预处理");
    }

}


/**
 * 静态代理的客户端调用
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 10:12
 * @since 1.0.0+
 */
public class ProxyClient {
    public static void main(String[] args) {
        Proxy proxy = new Proxy();
        proxy.request();
    }
}

```

### 代理模式扩展

#### 普通代理

普通代理要求客户端自能访问代理角色,不能访问真实角色.也就是不能直接new RealSubject对象,必须由
proxy角色来进行.普通代理比较常见,上述的静态代理实现方式其实就是普通代理.

#### 强制代理

我们一般的思维是通过代理模式找到真实的角色,但是强制代理反其道行之,必须强制通过真实角色去查找到其代理角色.
总而言之强制代理就是通过new 或者其他方式创建一个真实角色对象,真实角色对象会返回它的代理对象,然后通过代理
才可能进行一系列的操作.现实中最常见的例子就是某个明星和经纪人.好吧,你如果知道明星的联系方式,找到了他,结果他告诉你,
你去找我的经纪人XX吧,他会负责处理的.

具体实现如下:

```java
/**
 * 抽象主题
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 10:07
 * @since 1.0.0+
 */
public interface Subject {
    /**
     * 待具体实现的方法
     */
    void request();

    /**
     * 获取每个具体实现对应的代理对象实例
     * @return 返回对应的代理对象
     */
    Subject getProxy();
}

/**
 * 强制代理对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 14:36
 * @since 1.0.0+
 */
public class ForceProxy implements Subject {

    private Subject subject = null;

    public ForceProxy(Subject subject) {
        this.subject = subject;
    }

    /**
     * 待具体实现的方法
     */
    @Override
    public void request() {
        preRequest();
        this.subject.request();
        postRequest();
    }

    /**
     * @return 返回对应的代理对象就是自己
     */
    @Override
    public Subject getProxy() {
        return this;
    }

    private void postRequest() {
        System.out.println("访问真实主题以后的后续处理");
    }

    private void preRequest() {
        System.out.println("访问真实主题之前的预处理");
    }
}

/**
 * 具体的实现对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 14:35
 * @since 1.0.0+
 */
public class RealSubject implements Subject {

    /**
     * 该具体实现对象的代理对象
     */
    private Subject proxy = null;

    @Override
    public Subject getProxy(){
        this.proxy = new ForceProxy(this);
        return this.proxy;
    }

    /**
     * 待具体实现的方法
     */
    @Override
    public void request() {
        System.out.println("访问真实主题方法");
    }
}


/**
 * 强制代理客户端调用
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 14:40
 * @since 1.0.0+
 */
public class ForceProxyClient {
    public static void main(String[] args) {
        Subject subject = new RealSubject();
        Subject proxy = subject.getProxy();
        proxy.request();
    }
}

```

### 动态代理

什么是动态代理?动态代理是在实现阶段不用关心代理谁,而在运行阶段由JVM指定代理哪一个对象.相对来说,自己写代理类的方式就是静态代理,动态代理中,代理类并不是在java代码中定义的,而是在运行时根据我们在java代码中的指示动态生成.相对于静态代理,动态代理的优势在于可以很方便的对代理类的函数进行统一的处理,而不用修改每个代理类中的方法.

有一个名词叫做面向切面编程,也就是AOP(Aspect Oriented Programming),其核心就是采用了动态代理机制.

#### JDK动态代理

因为动态代理的灵活特性,我们在设计动态代理类(dynamicProxy)时,不用显示的让它的实现与真实主题类(realSubject)
相同的接口,而是把这个实现推迟到运行时.

为了能让DynamicProxy类能够在运行时才去实现RealSubject类已实现的一系列接口并执行接口中的相关方法操作,
需要让dynamicProxy类实现JDK自带的`java.lang.reflect.InvocationHandler`接口,该接口中的`invoke()`方法能够
让DynamicProxy实例在运行时调用被代理类需要对外实现的所有接口中的方法,也就是完成对真实主题类方法的调用.

所以我们必须先把真实主题类(realSubject)中已实现的所有接口都加载到JVM中,然后我们就可以创建一个
真实主题类的实例,获取该实例的类加载器ClassLoader(所谓的类加载器,即使具有某个类的定义,内部相关结构,包括继承树,方法区等等).

java动态代理机制中有两个重要的类和接口InvocationHandler（接口）和Proxy（类），这一个类Proxy和接口InvocationHandler是我们实现动态代理的核心

因此JDK动态代理的步骤如下:

1. 定义一个事件管理器,实现`InvocationHandler`接口,并重写invoke(被代理类,被代理的方法,方法的参数列表)方法
2. 实现具体主题对象类(realSubject)
3. 调用`Proxy.newProxyInstance(真实主题类加载器,真实主题类实现的接口,事件管理器对象)`生成一个代理实例.
4. 通过该代理实例调用方法

具体实现如下:

```java
/**
 * 抽象主题
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 10:07
 * @since 1.0.0+
 */
public interface Subject {
    /**
     * 待具体实现的方法
     */
    void request();
}

/**
 * 真实主题类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 11:13
 * @since 1.0.0+
 */
public class RealSubject implements Subject {
    /**
     * 具体实现的方法
     */
    @Override
    public void request() {
        System.out.println("访问真实主题方法");
    }
}

/**
 * 事件管理器,代理类的调用程序
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 11:14
 * @since 1.0.0+
 */
public class DynamicProxy implements InvocationHandler {

    /**
     * 代理类中的真实对象
     */
    private Object target;

    /**
     * 构造函数
     * @param target 真实对象
     */
    public DynamicProxy(Object target) {
        super();
        this.target = target;
    }

    /**
     *
     * @param proxy 被代理的类(真实代理对象)
     * @param method 被代理的类的方法
     * @param args 方法的参数列表
     * @return 代理对象方法的返回结果或者真实代理对象(com.sun.proxy.$Proxy0)
     * @throws Throwable Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before 动态代理");
        method.invoke(target, args);
        System.out.println("after 动态代理");
        return null;
    }
}

/**
 * JDK动态代理调用
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 11:59
 * @since 1.0.0+
 */
public class DynamicTest {

    public static void main(String[] args) {
        RealSubject realSubject = new RealSubject();
        InvocationHandler dynamicProxy = new DynamicProxy(realSubject);
        Class<?> cls = realSubject.getClass();

        Subject subject = (Subject) Proxy.newProxyInstance(cls.getClassLoader(), cls.getInterfaces(), dynamicProxy);
        subject.request();
    }
}

```

#### cglib动态代理

AOP的源码中用到了两种动态代理来实现拦截切入的功能:JDK动态代理和cglib动态代理
JDK的动态代理机制只能代理实现了接口的类,否则不能实现JDK的动态代理.因此JDK动态代理有一定的局限性,cglib是针对类来实现代理的,
它的原理是对指定的目标类生成一个子类,并覆盖其中方法实现增强,因为采用的是继承,所以不能对final修饰符的类进行代理.

因为JAVA只允许单继承,而JDK生成的代理类本身就继承了Proxy类,因此使用cglib来实现继承式的动态代理

CGLIB(Code Generation Library)是一个开源项目,是一个强大的，高性能，高质量的Code生成类库，它可以在运行期扩展Java类与实现Java接口，通俗说cglib可以在运行时动态生成字节码。

具体实现如下:

```java
/**
 * 具体主题对象,注意没有实现任何接口
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 12:16
 * @since 1.0.0+
 */
public class RealSubject {

    public void request() {
        System.out.println("访问真实主题方法");
    }
}


import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * 动态代理类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 12:17
 * @since 1.0.0+
 */
public class CglibProxy implements MethodInterceptor {
    /**
     * 通过Enhancer 创建代理对象
     */
    private Enhancer enhancer = new Enhancer();

    /**
     * 通过class对象获取代理对象
     * @param clazz class对象
     * @return 代理对象
     */
    public Object getProxy(Class<?> clazz) {
        //设置enhancer对象的父类
        enhancer.setSuperclass(clazz);
        //设置enhancer的回调
        enhancer.setCallback(this);
        return enhancer.create();
    }

    /**
     *
     * @param object 被代理对象
     * @param method 被代理对象的方法
     * @param args 被代理对象的方法参数
     * @param methodProxy 代理方法
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("before 动态代理");
        //调用新生成的cglib的代理对象,所属的父类的被代理方法
        Object result = methodProxy.invokeSuper(object, args);
        System.out.println("after 动态代理");
        return result;
    }
}

/**
 * cglib动态代理的调用
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 12:44
 * @since 1.0.0+
 */
public class CglibTest {

    public static void main(String[] args) {
        CglibProxy proxy = new CglibProxy();
        RealSubject realSubject = (RealSubject) proxy.getProxy(RealSubject.class);
        realSubject.request();
    }
}

```

### 代理模式的扩展

大到一个系统框架、企业平台，小到代码片段、事务处理，很多地方都会用到代理模式,该模式应该是我们接触最多的模式,而且有了AOP我们写代理就更加简单了,有类似于Spring AOP 和AspectJ这样的优秀工具,拿来就可以使用.

在学习AOP框架时,我们只需要弄清楚几个名词即可:切面(Aspect)、切入点(pointCut)、通知(advice)、目标(target)、 代理(proxy) 、连接点(joinPoint)
