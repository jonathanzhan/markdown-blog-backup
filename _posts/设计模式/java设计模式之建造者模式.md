---
title: java设计模式之建造者模式
date: 2019-08-20 09:59:31
tags:
- 设计模式
categories:
- 设计模式
toc: true
---
在软件的开发过程中，有时需要创建一个复杂的对象，这个对象通常由多个子部件按照一定的步骤组合而成。例如计算机有CPU,主板，内存，硬盘，显卡，机箱，显示器等组装而成，采购员不可能自己去组装计算机，而是将计算机的配置要求告诉给计算机销售，销售安排技术人员去组装计算机，然后交付。
生活中有很多这样的例子，比如房屋的建造，其空间，装修，家具等特性都有所差异，汽车的各个配件等等。
以上提到的产品都是由多个部件构成，每个部件都可以灵活选择,但是其创建的步骤都大同小异。这类产品可以用建造者模式很好的描述出产品的创建过程。
<!-- more -->
### 建造者模式的定义及特点

建造者(builder)模式，属于创建型模式的一种，指的是将一个复杂对象的构造与它的表示分离，使同样的构建过程可以创建不同的表示。它是将一个复杂的对象分解为多个简单的对象或者部件，然后一步一步构建而成。它将变与不变相分离，即部件的组成是不变得，但每一个部分是可以灵活选择的。

该模式的主要优点如下:

* 各个具体的建造者相互独立，有利于系统的扩展
* 客户端不需要知道产品内部组成的细节，便于控制细节风险

其缺点如下:

* 产品的组成部分必须相同，这就限制了其使用的范围
* 如果产品的内部变化复杂，该模式会增加很多的建造者类

注意:
**建造者模式和工厂模式关注点不同，建造者模式注重零部件的组装过程，工厂方法模式注重零部件的创建过程，两者可以结合使用**

### 建造者模式的结构与实现

#### 模式的结构

建造者(builder)模式由产品,抽象建造者，具体建造者，指挥者四个要素组成。

* 产品角色(product):它是包含多个组成部件的复杂对象，由具体的建造者来创建各个组件
* 抽象建造者(builder):它是一个包含创建产品各个子部件的抽象方法或接口,通常包含一个返回复杂对象的方法(retriveResult)。一般来说，产品角色所包含的零件数量与建造方法的数量相符，总而言之，就是有多少个零件，就有多少相应的建造方法。
* 具体建造者(concrete builder)：实现builder接口，完成复杂产品的零部件的具体创建方法和返回产品对象。
* 指挥者(director)角色:调用具体建造者角色来创建产品对象。注意的是导演者角色并没有产品类的具体知识，真正拥有产品类的具体知识的是具体建造者角色。

其结构图如下:
![建造者模式结构图](/files/dp/builder.png)

#### 模式的实现

##### 产品角色

```java
/**
 * 产品角色
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/20 10:35
 * @since 1.0.0+
 */
public class Product {
    //以下定义产品的零部件
    /**
     * 产品的零部件1
     */
    private String part1;

    /**
     * 产品的零部件2
     */
    private String part2;

    /**
     * 产品的零部件2
     */
    private String part3;

    ////////省略get set方法
}
```

##### 抽象建造者角色

```java
/**
 * 抽象建造者角色
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/20 10:38
 * @since 1.0.0+
 */
public interface Builder {
    //产品零部件的建造方法

    /**
     * 产品零部件1的建造方法
     */
    void buildPart1();

    /**
     * 产品零部件1的建造方法
     */
    void buildPart2();

    /**
     * 产品零部件1的建造方法
     */
    void buildPart3();

    /**
     * 返回产品对象的方法
     * @return Product
     */
    Product retrieveResult();
}
```

##### 具体建造者角色

```java
/**
 * 具体的建造者对象
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/20 10:40
 * @since 1.0.0+
 */
public class ConcreteBuilder implements Builder {
    private Product product = new Product();
    /**
     * 产品零部件1的建造方法
     */
    @Override
    public void buildPart1() {
        product.setPart1("part1");
    }

    /**
     * 产品零部件1的建造方法
     */
    @Override
    public void buildPart2() {
        product.setPart2("part2");
    }

    /**
     * 产品零部件1的建造方法
     */
    @Override
    public void buildPart3() {
        product.setPart3("part3");
    }

    /**
     * 返回产品对象的方法
     *
     * @return Product
     */
    @Override
    public Product retrieveResult() {
        return product;
    }
}

```

##### 指挥者角色

```java
/**
 * 指挥者角色
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/20 10:41
 * @since 1.0.0+
 */
public class Director {
    private Builder builder;

    /**
     * 指挥者角色构造方法，传入建造者对象
     * @param builder Builder
     */
    public Director(Builder builder) {
        this.builder = builder;
    }

    /**
     * 产品的构造方法,负责调用各个零件的建造方法，比如产品的生产熟悉怒等
     */
    public void construct(){
        builder.buildPart1();
        builder.buildPart2();
        builder.buildPart3();
    }
}
```

##### 客户端调用

```java
public class ClientTest {

    public static void main(String[] args) {
        Builder builder = new ConcreteBuilder();
        Director director = new Director(builder);
        director.construct();
        Product product = builder.retrieveResult();
        System.out.println(product.getPart1());
        System.out.println(product.getPart2());
    }
}
```

### 模式的应用场景

建造者模式创建的是复杂的对象，其产品的各个部分经常面临着剧烈的变化，但将他们组合在一起的算法是相对稳定的，所以通常用在以下场景:

* 创建的对象较复杂，由多个部件构成，各部件面临着复杂的变化，但构件间的建造顺序是稳定的
* 创建复杂对象的算法独立于该对象的组成部分以及他们的装备方式，即产品的构建过程和最终的表示是独立的
