---
title: java设计模式之策略模式
date: 2019-07-29 14:55:23
tags:
- 设计模式
categories:
- 设计模式
toc: true
---
在现实生活中常常遇到实现某种目标存在多种方案可供选择的情况,例如出行旅游可以乘坐火车,飞机,或者自驾等。
在软件开发中,也常常遇到类似的情况,当实现某个功能存在多种算法或者策略,我们可以根据不同的条件选择不同的算法来实现该功能。比如排序算法,我们可以选择冒泡排序,选择排序,插入排序,二叉树排序等
如果使用多重条件转移语句实现(即硬编码),不但使条件语句变得复杂，而且增加了代码的复杂程度,不易维护,违背开闭原则。采用策略模式能很好的解决这个问题。
<!-- more -->
### 概述

#### 定义

策略(strategy)模式属于对象行为型模式,主要针对一组算法,将每一个算法封装到具有公共接口的独立实现类中,
从而使他们可以相互替换,策略模式可以使程序在不影响客户端的情况下发生变化。通常,策略模式适用于
当一个程序需要实现一种特定的服务或者功能,而且该程序有多种的实现方法时使用。

#### 主要优点

1. 多重语句不易维护,使用策略模式可以避免使用多重条件语句。
2. 策略模式提供了一系列的可供重用的算法族,恰当使用继承可以把算法族中的公共代码转移到父类里面,避免重复代码。
3. 策略模式可以提供相同行为的不同实现,客户可以根据不同条件选择不同的算法
4. 策略模式提供了对开闭原则的完美支持,可以在不修改源代码的情况下,灵活增加新的算法
5. 策略模式把算法的使用放到环境类中,而算法的实现转移到具体策略类中,实现了二者的分离。

#### 主要缺点

1. 客户端必须理解所有的策略算法的区别,用于选择恰当的算法。
2. 策略模式会造成很多的策略类。

### 策略模式的结构与实现

#### 结构

策略模式是准备一组算法,并将这组算法封装到一系列的策略类里面,作为一个抽象策略类的子类。策略模式的重点不是如何实现算法,而是如何组织这些算法,从而让程序结构更加灵活,具有更好的扩展性和可维护性。
策略模式的主要角色如下:

1. 抽象策略类(strategy):定义了一个公共接口,各种不同的算法以不同的方法实现这个接口。环境类使用这个接口调用不同的算法,一般使用接口或者抽象类实现。
2. 具体策略类(concrete strategy):实现了抽象策略定义的接口,提供具体的算法实现
3. 环境类(context):持有一个策略类的引用,最终给客户端调用。

其结构图如下:
![策略模式结构图](/files/dp/strategy.png)

#### 实现

```java
/**
 * 策略模式的环境类
 *
 * @author Jonathan
 * @version 1.0.0
 * @since 1.0.0+
 */
public class Context {

    private Strategy strategy;

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public void execMethod(){
        strategy.method();
    }

}



/**
 * 抽象策略接口类
 *
 * @author Jonathan
 * @version 1.0.0
 * @since 1.0.0+
 */
public interface Strategy {

    /**
     * 策略执行方法
     */
    void method();
}

/**
 * 接口实现类1
 *
 * @author Jonathan
 * @version 1.0.0
 * @since 1.0.0+
 */
public class StrategyImp1 implements Strategy {
    @Override
    public void method() {
        System.out.println("execute method 1");
    }
}

/**
 * 接口实现类2
 *
 * @author Jonathan
 * @version 1.0.0
 * @since 1.0.0+
 */
public class StrategyImp2 implements Strategy {

    @Override
    public void method() {
        System.out.println("execute method 2");
    }
}


public class StrategyTest {
    public static void main(String[] args) {
        Context context = new Context();
        context.setStrategy(new StrategyImp1());
        context.execMethod();

        context.setStrategy(new StrategyImp2());
        context.execMethod();
    }

}


```

### 具体案例

策略模式在很多地方都有用到,如java SE中的容器布局就是一个典型的实例,JAVA SE中的每个容器都存在多种布局供用户选择。在程序设计中,通常在以下几种情况使用策略模式较多。

1. 系统需要动态的在几种算法中选择一种,可将每个算法封装到策略类中。
2. 一个类定义了多种行为,并且这些行为在这个类的操作中以多个条件语句的形式出现，可将每个条件分支转移到他们各自的策略类中代替这些条件语句。
3. 系统中各算法彼此完全独立,并要求对客户隐藏具体的实现细节
4. 系统中要求使用算法的客户不应该知道其操作的数据时,可使用策略模式来隐藏与算法相关的数据结构
5. 多个类只区别在表现行为上的不同,可以使用策略模式,在运行时动态选择具体要执行的行为。

### 策略模式的扩展

1. Lambda表达式其实可以归纳到策略模式中
2. Spring源码的Resource接口的实现(参看下图,来自网络)

![Resource接口](/files/dp/spring-resource.jpg)
