---
title: java设计模式之工厂模式
date: 2019-07-30 17:31:27
tags:
- 设计模式
categories:
- 设计模式
toc: true
---

在现实生活中社会分工越来越细，越来越专业化。各种产品有专门的工厂生产，彻底告别了自给自足的小农经济时代，这大大缩短了产品的生产周期，提高了生产效率。
同样，在软件开发中能否做到软件对象的生产和使用相分离呢？
能否在满足“开闭原则”的前提下，客户随意增删或改变对软件相关对象的使用呢？
<!-- more -->
### 定义

工厂(factory)模式属于创建型模式,定义了一个创建对象的工厂接口,将产品对象的实际创建工作推迟到具体的工厂类中。
我们把被创建的对象成为`产品`，把创建产品的对象成为`工厂`

如果要创建的产品不多,只要一个工厂类就可以完成，这种模式叫`简单工厂模式`，它不属于23种设计模式,缺点是增加新产品时会违背`开闭原则`

### 工厂分类

以家电为例，区分说明工厂模式的几个概念

1. 在还有工厂时代:如果客户需要一台冰箱，一般的做法是自己去造一台冰箱，然后拿来用,也就是我们常用的通过new操作符自己创建对象实例。
2. 简单工厂模式:用户不用自己造冰箱，会有一个工厂来帮他造海尔冰箱，想要什么型号的海尔冰箱，这个工厂就可以建造。这个工厂就是创建海尔系列的冰箱。
3. 工厂方法模式:为了满足客户的需求，海尔集团的产品越来越多，有冰箱，空调，热水器等。一个工厂无法创建所有的产品，于是单独分出很多具体的工厂，每个工厂创建一种产品，即具体工厂只能创建一个具体的同类的产品。
4. 抽象工厂模式:随着家电的发展，出现了格力，美的等等。我们对产品再次进行分类，比如海尔的冰箱，格力的冰箱，美的的冰箱等同归于一个工厂，都可以生产，而空调则在另外一个工厂生产。每个工厂都能生产出一组的产品来。

综上所述:工厂模式总体分为三类

* 简单工厂模式(simple factory)
* 工厂方法模式(factory method)
* 抽象工厂模式(abstract factory)

三种模式从上到下逐步抽象,同时也可将工厂模式分为两类：工厂方法模式和抽象工厂模式。将简单工厂模式看做为工厂方法的一种特例。

### 简单工厂模式

#### 简单工厂描述
简单工厂模式又称静态工厂模式.它存在的目的就是定义一个用于创建对象的接口。
它的组成:

* 工厂类角色:本模式的核心,含有一定的逻辑或者判断。由一个具体类实现。
* 抽象产品角色:是具体产品继承的父类或者实现的接口。由抽象类或者接口来实现。
* 具体产品角色: 工厂类所创建的对象就是此角色的实例，由具体类实现。

#### 简单工厂实现

代码实现如下:

```java
/**
 * 家电的接口类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 11:37
 * @since 1.0.0+
 */
public interface Appliance {
    /**
     * 家电的描述显示接口方法
     */
    void display();

}

/**
 * 型号1的冰箱
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 11:40
 * @since 1.0.0+
 */
public class Refrigerator1 implements Appliance {
    /**
     * 冰箱1的描述显示
     */
    @Override
    public void display() {
        System.out.println("型号1的冰箱");
    }
}

/**
 * 型号2的冰箱
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 11:42
 * @since 1.0.0+
 */
public class Refrigerator2 implements Appliance {
    /**
     * 冰箱2的描述显示
     */
    @Override
    public void display() {
        System.out.println("型号2的冰箱");
    }
}

/**
 * 家电的工厂类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 11:44
 * @since 1.0.0+
 */
public class RefrigeratorFactory {
    public static Appliance build(String type) {
        if(type.equals("1")) {
            return new Refrigerator1();
        } else {
            return new Refrigerator2();
        }
    }
}

public static void main(String[] args) {
    Refrigerator refrigerator = RefrigeratorFactory.build("1");
    refrigerator.display();
}
```

使用了简单工厂模式后，我们不需要再关注产品创建的过程，仅仅负责获取产品即可。
参考上面的工厂方法简介，假如这个时候，我们需要开始生产电视，空调了，这个工厂该怎么办？再次打造出一个生产线，甚至重新定义产品，带来的代价太大了，因此我们可以考虑工厂模式了。

### 工厂模式

#### 工厂模式结构

工厂模式去掉了简单工厂模式中的静态方法，使得它可以被子类继承。这样在简单工厂模式中集中在工厂方法的压力就可以由工厂方法模式里的不同工厂子类来分担。

工厂模式的结构如下:

* 抽象工厂(abstract factory)角色:这是工厂模式的核心，提供创建产品的接口，调用者通过它访问具体工厂的工厂方法来创建产品。
* 具体工厂(concrete factory)角色: 主要实现抽象工厂的抽象方法。完成具体产品的创建
* 抽象产品(product):定义了产品的规范,描述了产品的主要特征和功能
* 具体产品(concrete product):实现了抽象产品角色所定义的接口,由具体工厂创建，同具体工厂之间一一对应。

#### 工厂模式实现

代码实现如下:

```java
/**
 * 家电的接口类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 11:37
 * @since 1.0.0+
 */
public interface Appliance {
    /**
     * 家电的描述显示接口方法
     */
    void display();

}


/**
 * 电视的具体实现对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 18:56
 * @since 1.0.0+
 */
public class Television implements Appliance {
    /**
     * 家电的描述显示接口方法
     */
    @Override
    public void display() {
        System.out.println("电视");
    }
}

/**
 * 冰箱的具体对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 18:55
 * @since 1.0.0+
 */
public class Refrigerator implements Appliance {
    /**
     * 冰箱的描述显示接口方法
     */
    @Override
    public void display() {
        System.out.println("冰箱");
    }
}

/**
 * 家电的抽象工厂接口
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 18:57
 * @since 1.0.0+
 */
public interface ApplianceFactory {
    /**
     * 家电的生产方法接口
     * @return appliance
     */
    Appliance build();
}

/**
 * 生产冰箱的具体工厂类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 18:59
 * @since 1.0.0+
 */
public class RefrigeratorFactory implements ApplianceFactory {
    /**
     * 冰箱的生产方法实现
     *
     * @return appliance
     */
    @Override
    public Appliance build() {
        return new Refrigerator();
    }
}

/**
 * 生产电视的具体工厂类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 19:00
 * @since 1.0.0+
 */
public class TelevisionFactory implements ApplianceFactory {
    /**
     * 电视的生产方法实现
     *
     * @return appliance
     */
    @Override
    public Appliance build() {
        return new Television();
    }
}

public static void main(String[] args) {
    ApplianceFactory refrigeratorFactory = new RefrigeratorFactory();
    Appliance refrigerator = refrigeratorFactory.build();

    ApplianceFactory televisionFactory = new TelevisionFactory();
    Appliance television = televisionFactory.build();

    refrigerator.display();
    television.display();
}
```

### 抽象工厂方法

#### 抽象工厂模式描述

随着家电市场的扩大,出现了很多品牌的家电，比如海尔，格力等等，工厂此时也接到了更多的订单，比如生产海尔的电视，冰箱，格力的电视，冰箱等，我们此时就需要对业务线再次进行升级，于是用到了抽象工厂方法。

首先认识下什么是产品族:位于不同产品等级结构中，功能相关联的产品组成的家族。比如海尔的电视和海尔的冰箱都属于海尔品牌的家电，格力的电视和格力冰箱都属于格力品牌的家电。

抽象工厂模式和工厂方法的区别在于需要创建对象的复杂程度上。抽象工厂模式是三个里面最为抽象的，最具一般性的，抽象工厂模式的定义为:是一种为访问类提供一个创建一组相关或者相互依赖对象的接口，且访问类无须指定所要产品的具体类就能得到同族的不同等级的产品的模式结构。

要使用抽象工厂模式,一般需要满足以下条件:

* 系统中有多个产品族，每个具体工厂创建同一族但属于不同等级结构的产品。
* 系统一次只可能消费其中某一族产品,即同族的产品一起使用。

#### 抽象工厂模式结构

抽象工厂模式的主要角色如下:

* 抽象工厂(abstract factory):提供了创建产品的接口,包含了多个创建产品的方法，可以创建多个不同等级的产品。
* 具体工厂(concrete factory):主要实现抽象工厂中的多个抽象方法,完成具体产品的创建
* 抽象产品(product):定义了产品的规范,描述了产品的主要特征和功能，抽象工厂模式有多个抽象产品.
* 具体产品(concrete product):实现了抽象产品角色定义的接口,由具体工厂来创建，同具体工厂之间是多对一的关系。

#### 抽象工厂模式实现

具体实现如下:

```java
/**
 * 电视的抽象接口类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/16 11:49
 * @since 1.0.0+
 */
public interface Television {

    /**
     * 电视的描述显示接口方法
     */
    void display();
}

/**
 * 冰箱的接口类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/16 11:49
 * @since 1.0.0+
 */
public interface Refrigerator {
    /**
     * 冰箱的描述显示接口方法
     */
    void display();
}



/**
 * 家电的抽象工厂接口
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 18:57
 * @since 1.0.0+
 */
public interface ApplianceFactory {
    /**
     * 电视的生产方法接口
     * @return appliance
     */
    Television buildTelevision();


    /**
     * 冰箱的生产方法接口
     * @return appliance
     */
    Refrigerator buildRefrigerator();
}


/**
 * 格力冰箱的具体产品对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/16 11:48
 * @since 1.0.0+
 */
public class GeliRefrigerator implements Refrigerator {
    /**
     * 格力冰箱的描述显示接口方法
     */
    @Override
    public void display() {
        System.out.println("格力冰箱");
    }
}

/**
 * 格力电视的具体产品对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/16 11:48
 * @since 1.0.0+
 */
public class GeliTelevision implements Television {
    /**
     * 格力电视的描述显示接口方法
     */
    @Override
    public void display() {
        System.out.println("格力电视");
    }
}


/**
 * 海尔冰箱的具体产品对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/16 11:48
 * @since 1.0.0+
 */
public class HaierRefrigerator implements Refrigerator {
    /**
     * 海尔冰箱的描述显示接口方法
     */
    @Override
    public void display() {
        System.out.println("海尔冰箱");
    }
}


/**
 * 海尔电视的具体产品对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/16 11:48
 * @since 1.0.0+
 */
public class HaierTelevision implements Television {
    /**
     * 电视的描述显示接口方法
     */
    @Override
    public void display() {
        System.out.println("海尔电视");
    }
}

/**
 * 海尔家电的具体工厂类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 18:59
 * @since 1.0.0+
 */
public class HaierFactory implements ApplianceFactory {
    /**
     * 海尔电视的生产方法接口
     *
     * @return appliance
     */
    @Override
    public Television buildTelevision() {
        return new HaierTelevision();
    }

    /**
     * 海尔冰箱的生产方法接口
     *
     * @return appliance
     */
    @Override
    public Refrigerator buildRefrigerator() {
        return new HaierRefrigerator();
    }
}

/**
 * 格力家电的具体工厂类
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/14 18:59
 * @since 1.0.0+
 */
public class GeliFactory implements ApplianceFactory {
    /**
     * 格力电视的生产方法接口
     *
     * @return appliance
     */
    @Override
    public Television buildTelevision() {
        return new GeliTelevision();
    }

    /**
     * 格力冰箱的生产方法接口
     *
     * @return appliance
     */
    @Override
    public Refrigerator buildRefrigerator() {
        return new GeliRefrigerator();
    }
}


public static void main(String[] args) {
    ApplianceFactory haierFactory = new HaierFactory();
    ApplianceFactory geliFactory = new GeliFactory();

    Television haierTel = haierFactory.buildTelevision();
    Refrigerator haierRe = haierFactory.buildRefrigerator();

    Television geliTel = geliFactory.buildTelevision();
    Refrigerator geliRe = geliFactory.buildRefrigerator();

    haierTel.display();
    haierRe.display();
    geliTel.display();
    geliRe.display();
}


```