---
title: java反射机制
date: 2016-05-04 18:10:20
tags:
- java基础
categories:
- java
---


通俗地说,反射机制就是可以把一个类,类的成员(函数,属性),当成一个对象来操作，也就是说 类,类的成员,我们在运行的时候还可以动态地去操作他们.

<!-- more -->
具体看以下的demo

```java
import org.junit.Test;

import java.lang.reflect.*;

/**
 * java反射机制的测试
 * @author Jonathan
 * @date 2016/5/4 15:25
 * @since V1.0
 */
public class ReflectTest {


    /**
     * 通过一个对象，获取完整的包名和对象名
     */
    @Test
    public void demo1(){
        Tree tree = new Tree();
        //获取包名
        System.out.println(tree.getClass().getPackage().getName());
        //获取完整的类名
        System.out.println(tree.getClass().getName());
    }

    /**
     * 验证所有的类的对象都是Class类的实例
     *
     */
    @Test
    public void demo2(){
        Class<?> class1 = null;
        Class<?> class2 = null;
        Class<?> class3 = null;

        try {
            class1 = Class.forName("com.mfnets.workfocus.reflect.Tree");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        class2 = new Tree().getClass();

        class3 = Tree.class;

        System.out.println(class1.getName());
        System.out.println(class2.getName());
        System.out.println(class3.getName());
    }

    /**
     * 通过Class实例化其他类的对象
     * 前提是对象中包含有无参数的构造函数
     */
    @Test
    public void demo3(){
        Class<?> class1 = null;
        Tree tree = null;
        try {
            class1 = Class.forName("com.mfnets.workfocus.reflect.Tree");
            tree = (Tree) class1.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        tree.setId("1");
        tree.setName("name");
        System.out.println(tree.toString());
    }


    /**
     * 获取类对象的所有构造函数
     * 通过Class类调用其他的构造函数
     */
    @Test
    public void demo4(){
        Class<?> class1 = null;

        try {
            class1 = Class.forName("com.mfnets.workfocus.reflect.Tree");

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        //获取类的构造函数
        Constructor<?> constructor[] = class1.getConstructors();
        try {
            Tree tree1 = (Tree)constructor[0].newInstance();
            Tree tree2 = (Tree)constructor[1].newInstance("1","name");
            System.out.println(tree1.toString());
            System.out.println(tree2.toString());
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

    }


    /**
     * 获取类的父类，接口，函数，成员，类型等信息
     */
    @Test
    public void demo5(){
        Class<?> class1 = null;
        try {
            class1 = Class.forName("com.mfnets.workfocus.reflect.Tree");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        System.out.println("父类的信息");
        //获取类继承的父类的信息
        Class<?>  superClass = class1.getSuperclass();
        System.out.println(superClass.getName());


        System.out.println("接口的信息");
        //获取类实现的接口
        Class<?> interfaces[] = class1.getInterfaces();
        for (Class<?> inter:interfaces){
            System.out.println(inter.getName());
        }


        System.out.println("构造函数的信息");
        //获取类的构造函数
        Constructor<?> constructor[] = class1.getConstructors();
        for (Constructor<?> con:constructor){
            System.out.println(con);
        }


        System.out.println("类属性信息");
        //获取类的属性
        Field[] fields = class1.getDeclaredFields();
        for (Field field:fields){
            System.out.println(Modifier.toString(field.getModifiers())+" "+field.getType()+" "+field.getName());
        }

        //获取实现的接口或者父类中的属性
        System.out.println("类接口或者父类中的属性信息");
        Field[] fields1 = class1.getFields();
        for (Field field:fields1){
            System.out.println(Modifier.toString(field.getModifiers())+" "+field.getType()+" "+field.getName());
        }

        System.out.println("类中方法的信息");
        //获取类中的方法
        Method[] methods = class1.getDeclaredMethods();
        for (Method method:methods){
            StringBuffer sb = new StringBuffer();
            for (Class<?> param:method.getParameterTypes()){
                sb.append(param.getName()+",");
            }
            System.out.println(Modifier.toString(method.getModifiers())+" "+method.getReturnType()+" "+method.getName()+"("+sb.toString()+")");
            Class<?> exc[] = method.getExceptionTypes();
            for (Class<?> e:exc){
                System.out.println("throws:"+e.getName());
            }

        }
    }

    /**
     * 通过反射调用类的其他方法
     */
    @Test
    public void demo6(){
        Class<?> class1 = null;
        try {
            class1 = Class.forName("com.mfnets.workfocus.reflect.Tree");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        try {
            Method method1 = class1.getMethod("printTree");
            method1.invoke(class1.newInstance());

            Method method2 = class1.getMethod("printTree",String.class,String.class);
            method2.invoke(class1.newInstance(), "1222", "Jonathan");

        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }

    }


    /**
     * 通过反射调用set get方法
     */
    @Test
    public void demo7(){
        Class<?> class1 = null;
        try {
            class1 = Class.forName("com.mfnets.workfocus.reflect.Tree");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        try {
            Object obj = class1.newInstance();

            Field field = class1.getDeclaredField("name");
            field.setAccessible(true);
            field.set(obj, "张三");

            System.out.println(field.get(obj));
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }


    }

    /**
     * 通过Java反射机制得到类加载器信息
     * 在java中有三种类类加载器
     * 1）Bootstrap ClassLoader 此加载器采用c++编写，一般开发中很少见。
     * 2）Extension ClassLoader 用来进行扩展类的加载，一般对应的是jre\lib\ext目录中的类
     * 3）AppClassLoader 加载classpath指定的类，是最常用的加载器。同时也是java中默认的加载器。
     */
    @Test
    public void demo8(){
        Class<?> class1 = null;
        try {
            class1 = Class.forName("com.mfnets.workfocus.reflect.Tree");
            String name = class1.getClassLoader().getClass().getName();
            System.out.println(name);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 通过反射取得并修改数组的信息
     */
    @Test
    public void demo9(){
        int[] temp={1,2,3,4,5};
        Class<?> class1 = temp.getClass().getComponentType();
        System.out.println("数组类型： "+class1.getName());
        System.out.println("数组长度  "+Array.getLength(temp));
        System.out.println("数组的第一个元素: "+Array.get(temp, 0));
        Array.set(temp, 0, 100);
        System.out.println("修改之后数组第一个元素为： "+Array.get(temp, 0));

        Object newArr=Array.newInstance(class1, 7);

        int co=Array.getLength(temp);
        System.arraycopy(temp, 0, newArr, 0, co);

        for (int i = 0; i < Array.getLength(newArr); i++) {
            System.out.print(Array.get(newArr, i)+" ");
        }

    }

}

```