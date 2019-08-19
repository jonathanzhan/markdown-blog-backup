---
title: java设计模式之单例模式
date: 2019-08-19 15:18:05
tags:
- 设计模式
categories:
- 设计模式
toc: true
---
单例模式是设计模式中最简单的模式之一。通常，普通类的构造函数是公有的，外部类可以通过 ` new 构造函数()`来生成多个实例，但是，如果将类的构造函数设置为私有，外部类就无法调用该构造函数，也就无法生成多个实例。这时该类自身必须定义一个静态私有实例，并向外提供一个静态的公有函数用于创建或获取该静态私有实例。
<!-- more -->
### 单例模式的定义与特点

单例(singleton)模式的定义:指一个类只有一个实例,且该类能自行创建这个实例的一种模式，例如，windows中只能打开一个任务管理器,这样可以避免因打开多个任务管理器窗口而造成内存资源的浪费或出现各个窗口显示内容不一致的问题。
在计算机系统中,还有windows的回收站、操作系统中的文件系统、多线程中的线程池、显卡的驱动程序对象、打印机的后台处理服务、应用程序的日志对象、数据库连接池、网站计数器、web应用的配置对象、应用程序中的对话框、系统缓存等常常都被设计成单例
单例模式有三个特点:

* 单例类只有一个实例对象
* 该单例对象必须由单例类自行创建
* 单例类对外提供一个访问该单例的全局访问点

### 单例模式的结构

单例模式的主要角色如下:

* 单例类:包含一个实例且能自行创建这个实例的类
* 访问类：使用单例的类

其结构图如下:
![单例模式结构图](/files/dp/singleton.png)

### 单例模式的实现

#### 饿汉式单例

```java
/**
 * 饿汉式单例
 * 缺点，无法延时加载,没有使用就已经加载了
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/19 18:21
 * @since 1.0.0+
 */
public class HungrySingleton {

    private static final HungrySingleton singleton = new HungrySingleton();

    private HungrySingleton(){

    }
    /**
     * 静态工厂方法
     * @return HungrySingleton
     */
    public static HungrySingleton getInstance() {
        return singleton;
    }
}
```

#### 懒汉式单例(线程不同步)

优化了饿汉式无法延迟加载的问题

```java
/**
 * 懒汉式单例
 * 多线程并发的时候会失效，getInstance不同步，
 * 例子：一个线程在创建mInstance时，还未创建完成，另一个线程访问mInstance此时还是为空，又创建了一个
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/19 18:07
 * @since 1.0.0+
 */
public class SafeLazySingleton {

    private static SafeLazySingleton single = null;

    /**
     * 构造函数私有化
     */
    private SafeLazySingleton(){
    }

    /**
     * 静态工厂方法,线程安全，同步
     * @return Singleton
     */
    public static synchronized SafeLazySingleton getInstance() {
        if(single == null) {
            single = new SafeLazySingleton();
        }
        return single;
    }
}
```

以上懒汉式的实现没有考虑线程安全问题,在并发环境下可能出现多个Singleton实例，要实现线程安全,以下三种方法，都是对getInstance方法进行改造，保证了懒汉式单例的线程安全。

#### 懒汉式单例(线程安全)

在getInstance方法加上`synchronized`

```java
/**
 * 静态工厂方法,线程安全，同步
 * @return Singleton
 */
public static synchronized Singleton getInstance() {
    if(single == null) {
        single = new Singleton();
    }
    return single;
}
```

或者通过同步代码块来实现

```java
/**
    * 静态工厂方法,线程安全，同步代码块
    * @return SafeLazySingleton
    */
public static SafeLazySingleton getInstance() {
    synchronized (SafeLazySingleton.class) {
        if (single == null) {
            single = new SafeLazySingleton();
        }
        return single;
    }
}
```

#### DCL双重检查锁

```java
/**
 * DLC双重检查单例
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/19 18:30
 * @since 1.0.0+
 */
public class DLCSingleton {

    private static volatile DLCSingleton single = null;

    private DLCSingleton() {}

    public static DLCSingleton getInstance() {
        //第一次检查,先检查实例是否存在，如果不存在才进入下面的同步块
        if(single == null) {
            //同步块，线程安全的创建实例
            synchronized (DLCSingleton.class) {
                //再次检查实例是否存在，如果不存在才真正的创建实例
                if(single == null) {
                    single = new DLCSingleton();
                }
            }
        }
        return single;
    }
}
```

关键字`volatile`的意思是:被它修饰的变量的只，将不会被本地线程缓存,所有对该变量的读写都是直接操作共享内存,从而确保多个线程能正确的处理该变量。但是可能会屏蔽虚拟机中一些必要的代码优化，因为它进制编译器对`volatie`变量重排序,并且保证变量的读操作发生在写操作之后,所以运行效率并不是很高。因此一般情况下，没有特别的需要，不要使用。

#### 静态内部类

这是最为推崇的写法,利用static final 关键字的同步机制,初始化后就无法修改，保证了线程安全，使用`SingletonHolder.singleton`方法保证了只有被调用的时候才会装载,从而实现了延迟加载

```java
/**
 * 静态内部类的实现方式,完成了懒汉式的延迟加载，同时static保证了线程安全
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/19 18:39
 * @since 1.0.0+
 */
public class StaticInnerSingleton {

    private StaticInnerSingleton(){}

    public static StaticInnerSingleton getInstance() {
        return SingletonHolder.singleton;
    }

    /**
     * 私有静态类,初始化的时候,不调用getInstance()方法则不会加载
     */
    private static class SingletonHolder{
        /**
         * 利用static final关键字的同步机制，初始化后就无法修改保证了线程安全
         */
        private static final StaticInnerSingleton singleton = new StaticInnerSingleton();
    }
}
```

### 单例模式的应用场景

* 某类只要求生成一个对象的时候，如每个人的身份证号等。
* 当对象需要被共享时,由于单例模式只允许创建一个对象,共享该对象可以节省内存,并加快对象访问速度，比如数据库连接池，web配置对象等
* 某类需要频繁实例化,而创建的对象又频繁被销毁的时候,如多线程的线程池,网络连接池等

