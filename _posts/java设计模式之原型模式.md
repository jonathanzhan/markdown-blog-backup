---
title: java设计模式之原型模式
date: 2019-08-20 11:50:45
tags:
- 设计模式
categories:
- 设计模式
toc: true
---
在有些系统中，存在大量的相同或相似对象的创建问题，如果用传统的构造函数来创建对象，会比较复杂且耗时耗资源，用原型模式生成对象就很高效，就像孙悟空拔下猴毛变出很多孙悟空一样简单。
<!-- more -->
### 原型模式的定义与特点

原型(prototype)模式的定义如下:用一个已创建的实例作为原型，通过复制该原型对象来创建一个和原型相同或相似的新对象。在这里，原型实例指定了要创建的对象的种类。用这种方法创建对象非常高效，根本无须知道对象创建的细节。例如windows操作系统安装通常较耗时，但是通过复制镜像就快了很多。

### 原型模式的结构

由于java提供了对象的clone()方法,所以用java实现原型模式很简单
原型模式包含以下主要角色:

1. 抽象原型类:规定了具体原型对象必须实现的接口。
2. 具体原型类:实现抽象原型类的clone()方法，它是可被复制的对象
3. 访问类:使用具体原型类中的clone()来复制新的对象

### 浅度克隆的实现

浅度克隆只负责克隆按值传递的数据(比如基本数据类型，String类型)，而不复制它所引用的对象。换言之，所有的对其他对象的引用都仍然指向原来的对象。java中的object类提供了浅克隆的clone()方法，具体原型类只要实现cloneable接口就可以实现对象的浅克隆。cloneable接口就是抽象原型类。
代码如下:

```java
/**
 * 抽象原型之浅克隆
 *
 * @author Jonathan
 * @version 1.0.0
 * @date 2019/8/20 15:08
 * @since 1.0.0+
 */
public class RealizeType implements Cloneable {
    RealizeType()
    {
        System.out.println("具体原型创建成功！");
    }
    private String name;
    private List<String> list = new ArrayList<>();
    @Override
    public Object clone() throws CloneNotSupportedException
    {
        System.out.println("具体原型复制成功！");
        return (RealizeType)super.clone();
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    public List<String> getList() {
        return list;
    }

    public void setList(List<String> list) {
        this.list = list;
    }
}

public static void main(String[] args) throws CloneNotSupportedException {
    RealizeType realizetype = new RealizeType();
    realizetype.setName("test1");

    List<String> list = new ArrayList<String>();
    list.add("1111");
    list.add("2222");
    realizetype.setList(list);

    RealizeType cloneType = (RealizeType) realizetype.clone();
    list.add("3333");
    cloneType.setName("test2");
    cloneType.setList(list);
    System.out.println(realizetype.getName());
    System.out.println(cloneType.getName());
    System.out.println(realizetype.getList());
    System.out.println(cloneType.getList());
}
```

浅度克隆对于基本类型和String类型的数据前后都是指向不同的地址空间的，改变一个不会影响其他的对象
但是如果包含引用类型，比如对象，数组，集合等，就只会克隆引用，结果指向同一个引用地址

### 深度克隆

由于浅度克隆的引用类型问题，我们引入了深度克隆的方式。
要实现深度克隆，我们必须修改clone方法

```java
/**
 * 深度克隆
 * @return Object
 * @throws CloneNotSupportedException
 */
@Override
public Object clone() throws CloneNotSupportedException
{
    System.out.println("具体原型复制成功！");
    RealizeType cloneType = (RealizeType)super.clone();
    if(cloneType.getList() != null) {
        List<String> list = new ArrayList<>();
        for (String str:cloneType.getList()) {
            list.add(str);
        }
        cloneType.setList(list);
    }
    return cloneType;
}
```

### 原型模式的应用场景

原型模式通常使用于以下场景:

* 对象之间相同或相似,即只是个别的几个属性不同的时候
* 对象的创建过程比较麻烦,但复制比较简单的时候