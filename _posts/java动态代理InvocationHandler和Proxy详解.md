---
title: java动态代理InvocationHandler和Proxy详解
date: 2019-08-29 15:52:40
tags:
- java基础
categories:
- java
---
今天在整理代理模式时,发现以前对于`InvocationHandler`中的`invoke()`方法理解很肤浅,所以重新梳理学习了下.
<!-- more -->

### InvocationHandler接口

`InvocationHandler`接口是proxy代理实例的调用处理程序实现的一个接口,每一个proxy代理实例都有一个关联的调用处理程序.
在代理实例调用方法时,方法调用被编码分派到调用处理程序的`invoke()`方法.

每一个`动态代理类调用处理程序`都必须实现`InvocationHandler`接口,并且每个代理类的实例都关联到了实现该接口的动态代理类调用处理程序中.
我们通过动态代理对象调用方法时,这个方法的调用就会被转发到实现了`InvocationHandler`接口类的`invoke()`方法来调用.
方法参看

```java
    /**
     * invoke方法
     * @param proxy 代理类代理的真实代理对象com.sun.proxy.$Proxy0
     * @param method 所要调用某个真实对象的方法的Method对象
     * @param args 所要调用某个真实对象的方法的传递的参数
     * @return 所要调用某个真实对象的方法的返回值或者返回代理对象
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    }
```

#### proxy参数

查了各种资料,个人总结出proxy参数的解释如下:

1. 可以使用反射获取代理对象的信息(也就是proxy.getClass().getName())
2. 可以将代理对象返回用来进行连续调用.

proxy是真实对象的真实代理对象,invoke方法可以返回调用代理对象方法的返回结果,也可以返回对象的真实代理对象(com.sun.proxy.$Proxy0)

参看以下例子

```java
/**
 * 代理模式中的抽象主题对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 18:14
 * @since 1.0.0+
 */
public interface Subject {

    /**
     * 方法无具体含义,为了测试参数及返回值类型
     * @param name String
     * @return Subject
     */
    Subject request(String name);

    /**
     * 方法无具体含义,为了测试方法的返回值
     * @return String
     */
    String getTime();

    /**
     * 方法无具体含义,为了测试方法无返回值
     */
    void test();

}
/**
 * 具体实现
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 18:17
 * @since 1.0.0+
 */
public class RealSubject implements Subject {
    /**
     * 方法无具体含义,为了测试参数及返回值类型
     *
     * @param name String
     * @return Subject
     */
    @Override
    public Subject request(String name) {
        System.out.println("调用request方法");
        return this;
    }

    /**
     * 方法无具体含义,为了测试方法的返回值
     *
     * @return String
     */
    @Override
    public String getTime() {
        System.out.println("调用getTime方法");
        return new Date().toString();
    }

    /**
     * 方法无具体含义,为了测试方法无返回值
     */
    @Override
    public void test() {
        System.out.println("调用test方法");
    }
}


/**
 * 代理类的调用
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 18:19
 * @since 1.0.0+
 */
public class ProxyHandler implements InvocationHandler {

    private Object object;

    public ProxyHandler(Object object) {
        this.object = object;
    }

    /**
     * invoke方法
     * @param proxy 代理类代理的真实代理对象com.sun.proxy.$Proxy0
     * @param method 所要调用某个真实对象的方法的Method对象
     * @param args 所要调用某个真实对象的方法的传递的参数
     * @return 所要调用某个真实对象的方法的返回值
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before 动态代理");
        System.out.println("invoke()方法参数一的类型为:"+proxy.getClass().getName());
        if(method.getName().equals("request")){
            method.invoke(this.object, args);
            System.out.println("after 动态代理");
            return proxy;
        } else if (method.getName().equals("getTime")){
            System.out.println("after 动态代理");
            return method.invoke(this.object, args);
        } else {
            System.out.println("after 动态代理");
            method.invoke(this.object, args);
            return proxy;
        }
    }
}

/**
 * JDK动态代理调用测试
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/29 18:25
 * @since 1.0.0+
 */
public class HandlerTest {
    public static void main(String[] args) {
        Subject subject = new RealSubject();
        InvocationHandler handler = new ProxyHandler(subject);
        Class<?> clazz = subject.getClass();
        Subject proxy = (Subject) Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(),handler);
        proxy.request("name").getTime();
        proxy.test();
        System.out.println("代理对象的类型为:"+ proxy.getClass().getName());
        System.out.println(Proxy.getProxyClass(clazz.getClassLoader(), clazz.getInterfaces()));
        System.out.println(Proxy.isProxyClass(proxy.getClass()));
        System.out.println(Proxy.getInvocationHandler(proxy));
    }
}

```

输入结果为
> before 动态代理
invoke()方法参数一的类型为:com.sun.proxy.$Proxy0
调用request方法
after 动态代理
before 动态代理
invoke()方法参数一的类型为:com.sun.proxy.$Proxy0
after 动态代理
调用getTime方法
before 动态代理
invoke()方法参数一的类型为:com.sun.proxy.$Proxy0
after 动态代理
调用test方法
代理对象的类型为:com.sun.proxy.$Proxy0
class com.sun.proxy.$Proxy0
true
com.clexel.codetree.java.proxy.ProxyHandler@27c170f0

注意代码中的真实对象中三个方法的返回值(String,返回当前对象,void等).
由以上代码可以得到结论:

1. invoke()中的第一个参数运行时的类型是:com.sun.proxy.$Proxy0真实的代理对象
2. invoke()方法我们既可以返回真实对象方法的返回结果(int,String,等等),也可以将proxy返回,以用来连续调用,参看测试调用代码中的`proxy.request("name").getTime()`,其中request()方法的返回结果就是`proxy`,然后我们可以连续进行调用`getTime()`方法.
3. 如果返回this的话,this的类型指的是实现InvocationHandler接口的类(代理类的调用处理程序),并不是代理类$Proxy0

### Proxy类说明

Proxy类就是用来创建一个真实对象的真实代理对象,提供了很多方法,最常用的就是`newProxyInstance`方法

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

方法的作用就是创建一个代理类对象,总共有三个参数,含义如下:

1. loader:ClassLoader对象,定义了由哪个ClassLoader对象对生成的代理类进行加载.一般指的是真实对象的ClassLoader对象
2. interfaces:指的是将要给我们的代理对象提供一组什么样的接口,也就是声明代理类实现哪些接口,这样代理类才可以调用接口中声明的方法.一般指的是真实对象所实现的接口.
3. h: 指的就是实现了`InvocationHandler`接口的代理类调用处理类,表示的是当动态代理对象调用方法时会关联到哪个实现了`InvocationHandler`接口的对象上,并进行转发到`invoke()`方法中

Proxy类还有其他的方法:

1. Proxy.getProxyClass():给定类加载器和接口数组的代理类的java.lang.Class对象。
2. Proxy.isProxyClass():当且仅当使用getProxyClass方法或newProxyInstance方法将指定的类动态生成为代理类时，才返回true。
3. Proxy.getInvocationHandler():返回指定代理实例的调用处理程序
