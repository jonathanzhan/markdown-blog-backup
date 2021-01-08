---
title: java设计模式之观察者模式
date: 2019-08-12 16:33:47
tags:
- 设计模式
categories:
- 设计模式
toc: true
---
在现实世界中,很多对象并不是独立存在的,其中一个对象的行为发生改变可能会导致一个或者多个的其他对象的行为也发生改变。
比如微信公众号,不定时的发布一些消息,关注公众号就可以收到消息,取消关注就收不到消息。还有当我们开车到路口时，
遇到红灯会停下,绿灯则通行等等
在软件世界也是这样,比如图表中的数据与折线图,饼状图,柱状图之间的关系，MVC模式中的模型与视图的关系，比如发布订阅模式等等。
<!-- more -->
### 概述

#### 定义

观察者(observer)模式的定义是指多个对象间存在一对多的依赖关系,当一个对象的状态发生改变,所有依赖于它的对象都得到通知并被自动更新。
这种模式又被称作发布-订阅(publish-subscribe)模式,模型-视图(model/view)模式,源-监听器(source/listener)模式,它是对象行为型模式。
**观察者模式和发布订阅模式是有稍微的区别的**

#### 主要优点

1. 降低了目标与观察者之间的耦合关系,两者之间是抽象耦合关系。
2. 目标与观察者之间建立了一套触发机制。

#### 主要缺点

1. 目标与观察者之间的依赖关系并没有完全解除,而且很有可能出现循环引用。
2. 当观察者对象很多时,通知的发布会花费很多时间,影响程序效率。

### 观察者模式的结构与实现

实现观察者模式时要注意具体目标对象和具体观察者对象之间不能直接调用,否则将使两者之间紧密耦合起来,违反了面向对象的设计原则。

#### 结构

观察者模式的主要角色如下:

1. 抽象主题(subject)角色:也叫抽象目标类,它提供了一个用于保存观察者对象的聚集类和增加,删除观察者对象的方法,以及通知所有观察者的抽象方法。
2. 具体主题(concrete subject)角色:也叫具体目标类,它实现抽象目标中的通知方法,当具体主题的内部状态发生改变时,通知所有注册过的观察者对象。
3. 抽象观察者(observer)角色:它是一个抽象类或接口,包含了一个更新自己的抽象方法,当接收到具体主题的更改通知时被调用。
4. 具体观察者(concrete observer)角色:实现了抽象观察者定义的抽象方法,以便在得到目标的更改通知时更新自身的状态。

其结构图如下:
![观察者模式结构图](/files/dp/observer.png)

同时,在观察者模式,又分为推模型和拉模型两种方法。

* 推模型:主题对象向观察者对象推送主题的详细信息,不管观察者是否需要,推送的信息通常是主题对象的全部或部分数据。
* 拉模型:主题对象在通知观察者的时候,只传递少量信息,如果观察者需要更具体的信息,由观察者主动到主题对象中获取,相当于是从主题对象中拉数据。这种模型的实现中,会把主题对象自身通过实现的通知方法传递给观察者,这样观察者在需要获取数据的时候,就可以通过这个引用来获取。

两种模式的比较:
推模型是假定主题对象知道观察者需要的数据,而拉模型是主题对象不知道观察者具体需要什么数据,没有办法的情况下,干脆把自身传递给观察者,让观察者自己去按需要取值。

#### 推模型实例实现

* 定义抽象主题

```java
/**
 * 抽象主题
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/6/12 19:04
 * @since 1.0.0+
 */
public abstract class Subject {

    /**
     * 定义保存观察者对象的聚集
     */
    protected List<Observer> observers = new ArrayList<>();

    /**
     * 增加观察者对象
     * @param o 观察者对象
     */
    public void add(Observer o) {
        observers.add(o);
    }

    /**
     * 删除观察者对象
     * @param o 观察者对象
     */
    public void remove(Observer o) {
        observers.remove(o);
    }

    /**
     * 定义通知所有观察者对象的抽象方法
     * @param info 通知给观察者对象的消息数据
     */
    protected abstract void notifyObserver(String info);

}

```

* 定义具体主题对象

```java
/**
 * 主题的具体实现对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/6/12 19:21
 * @since 1.0.0+
 */
public class ConcreteSubject extends Subject {

    @Override
    protected void notifyObserver(String info) {
        System.out.println("开始广播。。。。");
        for (Observer o : observers) {
            o.response(info);
        }
    }
}
```

* 定义抽象观察者对象(接口或者抽象类)

```java
/**
 * 观察者接口
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/6/12 19:04
 * @since 1.0.0+
 */
public interface Observer {
    /**
     * 观察者响应
     * @param info 数据内容
     */
    void response(String info);
}

```

* 定义具体观察者对象

```java
/**
 * 具体观察者对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/6/12 19:52
 * @since 1.0.0+
 */
public class ObserverObject implements Observer {
    /**
     * 观察者名称
     */
    private String name;

    /**
     * 被通知的消息数据
     */
    private String dataInfo;

    public ObserverObject(String name) {
        this.name = name;
    }

    /**
     * 观察者响应
     *
     * @param info 数据内容
     */
    @Override
    public void response(String info) {
        this.dataInfo = info;
        System.out.println(name + " 收到推送消息： " + dataInfo);
    }

}
```

* 编写测试类

```java
public class ObserverTest {

    public static void main(String[] args) {
        ConcreteSubject subject = new ConcreteSubject();
        ObserverObject object1 = new ObserverObject("张三");
        ObserverObject object2 = new ObserverObject("李四");
        subject.add(object1);
        subject.add(object2);

        subject.notifyObserver("mess1");

        subject.notifyObserver("mess2");

        subject.remove(object1);
        subject.notifyObserver("mess3");
    }
}

```

测试结果如下:

> 开始广播。。。。
张三 收到推送消息： mess1
李四 收到推送消息： mess1
开始广播。。。。
张三 收到推送消息： mess2
李四 收到推送消息： mess2
开始广播。。。。
李四 收到推送消息： mess3

#### 拉模型实例实现

拉模型通常都是把主题对象当做参数传递。

* 定义抽象主题

```java
/**
 * 抽象主题对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/12 16:50
 * @since 1.0.0+
 */
public abstract class Subject {
    /**
     * 定义保存观察者对象的聚集
     */
    protected List<Observer> observers = new ArrayList<>();

    /**
     * 增加观察者对象
     * @param o 观察者对象
     */
    protected void add(Observer o) {
        observers.add(o);
    }

    /**
     * 删除观察者对象
     * @param o 观察者对象
     */
    protected void remove(Observer o) {
        observers.remove(o);
    }

    /**
     * 定义通知所有观察者对象的抽象方法
     */
    protected void notifyObserver() {
        for (Observer observer : observers) {
            observer.response(this);
        }
    }

}
```

* 定义具体主题对象

```java
/**
 * 具体的主题对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/12 16:55
 * @since 1.0.0+
 */
public class ConcreteSubject extends Subject {

    /**
     * 传递的少量消息
     */
    private String version;

    /**
     * 更多消息内容
     */
    private String description;

    public String getVersion() {
        return version;
    }

    public String getDescription() {
        return description;
    }

    protected void changeInfo(String version, String description) {
        this.version = version;
        this.description = description;
        System.out.println("版本变更为:" + this.version + ",更新内容为:" + this.description);
        //消息发生改变,通知给各个观察者
        this.notifyObserver();
    }
}
```

* 定义抽象观察者对象

```java
/**
 * 观察者抽象接口
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/12 16:52
 * @since 1.0.0+
 */
public interface Observer {
    /**
     * 观察者响应
     * @param subject 抽象主题对象
     */
    void response(Subject subject);
}
```

* 定义具体观察者对象

```java
/**
 * 具体的观察者对象(拉模式)
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/12 16:54
 * @since 1.0.0+
 */
public class ConcreteObserver implements Observer {
    /**
     * 观察者名称
     */
    private String name;

    /**
     * 主题对象
     */
    private Subject subject;

    public ConcreteObserver(String name) {
        this.name = name;
    }

    /**
     * 观察者响应
     *
     * @param subject 抽象主题对象
     */
    @Override
    public void response(Subject subject) {
        this.subject = subject;
        // 将主题对象传给观察者,观察者就可以在响应方法中获取想要的信息,比如version,description
        String version = ((ConcreteSubject) subject).getVersion();
        String description = ((ConcreteSubject) subject).getDescription();
        // 疑问
        // ((ConcreteSubject) subject).changeInfo("1.0.0","观察者自己更改了通知内容");
        System.out.println(name + " 接收到了新版本: " + version + "通知,变更记录为:"+description);
    }
}

```
*在这里有个疑问:拉模式中直接把主题对象传给观察者对象后,是不是会暴露出一些不应该观察者能取到的数据或者方法,比如主题对象推送消息的方法调用？？*

* 编写测试类

```java
public static void main(String[] args) {
        ConcreteSubject subject = new ConcreteSubject();
        ConcreteObserver object1 = new ConcreteObserver("张三");
        ConcreteObserver object2 = new ConcreteObserver("李四");
        subject.add(object1);
        subject.add(object2);
        subject.changeInfo("1.0","1.0变更记录");
        subject.changeInfo("1.1","1.1变更记录");

        subject.remove(object1);
        subject.changeInfo("1.2","1.2变更记录");
    }
```

测试结果如下:

> 版本变更为:1.0,更新内容为:1.0变更记录
张三 接收到了新版本: 1.0通知,变更记录为:1.0变更记录
李四 接收到了新版本: 1.0通知,变更记录为:1.0变更记录
版本变更为:1.1,更新内容为:1.1变更记录
张三 接收到了新版本: 1.1通知,变更记录为:1.1变更记录
李四 接收到了新版本: 1.1通知,变更记录为:1.1变更记录
版本变更为:1.2,更新内容为:1.2变更记录
李四 接收到了新版本: 1.2通知,变更记录为:1.2变更记录

### 模式的应用场景

通过前面的分析与应用实例可知观察者模式适合以下几种情形。

1. 对象间存在一对多关系，一个对象的状态发生改变会影响其他对象。
2. 当一个抽象模型有两个方面，其中一个方面依赖于另一方面时，可将这二者封装在独立的对象中以使它们可以各自独立地改变和复用。

### 模式的扩展

在 Java 中，通过 java.util.Observable 类和 java.util.Observer 接口定义了观察者模式，只要实现它们的子类就可以编写观察者模式实例。

1. Observable类
Observable 类是抽象目标类，它有一个 Vector 向量，用于保存所有要通知的观察者对象，下面来介绍它最重要的 3 个方法。
    * void addObserver(Observer o) 方法：用于将新的观察者对象添加到向量中。
    * void notifyObservers(Object arg) 方法：调用向量中的所有观察者对象的 update。方法，通知它们数据发生改变。通常越晚加入向量的观察者越先得到通知。
    * void setChange() 方法：用来设置一个 boolean 类型的内部标志位，注明目标对象发生了变化。当它为真时，notifyObservers() 才会通知观察者。
2. Observer 接口
Observer 接口是抽象观察者，它监视目标对象的变化，当目标对象发生变化时，观察者得到通知，并调用 void update(Observable o,Object arg) 方法，进行相应的工作。

具体实现如下:

```java
import java.util.Observable;

/**
 * 定义微信公众号推送服务
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/12 18:25
 * @since 1.0.0+
 */
public class WebChatServer extends Observable {
    public void push(String mess) {
        setChanged();
        notifyObservers(mess);
    }
}


import java.util.Observable;
import java.util.Observer;

/**
 * 观察者,关注公众号的用户
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/12 18:29
 * @since 1.0.0+
 */
public class User implements Observer {
    private String name;

    private String mess;

    public User(String name) {
        this.name = name;
    }

    @Override
    public void update(Observable observable, Object mess) {
        this.mess = mess.toString();
        read();
    }

    public void read() {
        System.out.println(name + " 收到推送消息： " + mess);
    }
}





public static void main(String[] args) {
    WebChatServer webChatServer = new WebChatServer();
    User user1 = new User("张三");
    User user2 = new User("李四");
    webChatServer.addObserver(user1);
    webChatServer.addObserver(user2);
    webChatServer.push("推送消息了");
}

```

测试结果如下:

> 李四 收到推送消息： 推送消息了
张三 收到推送消息： 推送消息了
