---
title: JVM常用总结3之常量池
date: 2020-07-07 13:35:05
tags:
- java基础
categories:
- Java
toc: true
---

本文主要介绍常量池的一些概念定义以及常见的问题,基于JDK1.8+

<!-- more -->

### class常量池,运行时常量池,字符串常量池

首先看下JVM的内存模型

![JVM内存模型](/files/java/JVM内存模型.png)

#### class常量池

Class常量池可以理解为class文件中的资源仓库,class文件中除了包含类的版本,字段,方法,接口等描述信息外,还有一项信息就是常量池,用于存放编译器生成的各种字面量和符号引用.

##### 字面量

字面量就是指有字母,数字等构成的字符串或者数值常量.字面量只可以出现在=右边.
如以下代码,=右边的都是字面量

```java
int a = 1;
int b = 2;
String c = "asd";
String d = "sdf";
```

字面量包括以下几种：

1. 文本字符串
2. 八种基本类型的值
3. 被声明为final的常量等

##### 符号引用

符号引用是编译原理的概念,主要包括以下三类常量:

* 类和接口的全限定名
* 字段的名称和描述符
* 方法的名称和描述符

上面代码中的 a,b,c,d 就是字段名称,是一种符号引用.

比如某个class类通过Javap反编译后,在常量池里的`Lcom/test/test` 是类的全限定名,里面的方法名,还有一些UTF8的格式描述符等,都是符号引用.

#### 运行时常量池

以上的这些常量池(符号引用,字面量)现在是静态信息,只有到运行时被加载到内存后,这些符号才有对应的内存地址信息,这些常量池一旦被加载到内存中,就会变为`运行时常量池`, 对应的符号引用在程序加载或运行时会被转换为被加载到内存区域的代码的直接引用,也就是我们说的动态链接例如方法test()这个符号引用在运行时就会被转换为test()方法具体代码在内存中的地址,主要通过对象头里的类型指针去转换直接引用.

#### 字符串常量池

##### 字符串常量池的设计思想

JVM为了提高性能和减少内存开销,在实例化字符串常量时进行了一些优化

* 为字符串单独开辟一个字符串常量池,类似于缓存区
* 创建字符串常量池时,首先查询字符串常量池中是否有该字符串
* 存在该字符串,则返回引用实例,不存在,实例化该字符串并放入字符串常量池中.

在jdk1.6及之前内存区域中有永久代, 运行时常量池在永久代，运行时常量池包含字符串常量池

Jdk1.7：有永久代，但已经逐步“去永久代”，字符串常量池从永久代里的运行时常量池分离到堆里

Jdk1.8及之后:无永久代，运行时常量池在元空间，字符串常量池里依然在堆里

##### 字符串常用操作

(1) 直接赋值字符串

`String s = "helloworld" // s指向常量池中的引用`
这种方式创建的字符串对象,只会在常量池中,因为有`helloworld` 这个字面量,创建s对象时,JVM会先去常量池中通过`equals(key)`方法,判断是否有相同对象,如果有,直接返回该对象在常量池的引用,如果没有,则在常量池中创建一个新对象,再返回引用

(2) new String()操作

`String s1 = new String("helloworld") //s1指向内存中的对象引用`

这种方式保证字符串常量池和堆中都有这个对象,没有就创建,最后返回堆内存中的对象引用.

步骤大致如下:

* 因为有`helloworld`这个字面量,所以先会检查常量池中是否存在字符串`helloworld`
* 如果不存在,先在字符串常量池中创建一个字符串对象,再去堆内存中创建一个字符串对象`helloworld`
* 存在的话,直接去堆内存中创建一个字符串对象`helloworld`
* 最后返回内存中的引用

(3) intern方法

```java
String s1 = new String("helloworld");
String s2 = s1.intern();
System.out.print(s1==s2); //false
```

String 中的intern方法是一个native方法,当调用intern方法时,如果常量池中已经包含了一个等于此String 对象的字符串(用equals方法确定),则返回常量池中的字符串.否则,将intern返回的引用指向当前字符串s1.(JDK1.7 1.8会将堆内存中的s1的引用放到常量池中,JDK1.6会将s1复制到常量池中)

参考以下代码

```java
String s1 = new String("he")+new String("llo");
String s2 = s1.intern();
System.out.print(s1==s2);

// 在 JDK 1.6 下输出是 false，创建了 6 个对象
// 在 JDK 1.7 及以上的版本输出是 true，创建了 5 个对象
```
首先在常量池中创建了两个对象`he`和`llo`,接下来在堆中同样创建了两个对象`he`和`llo`,然后在堆中创建了`hello`的对象.

在JDK1.6中,因为此时常量池中没有`hello`,所以会先将`hello`对象复制到常量池中,然后返回的是常量池中的对象.

在JDK1.7及以上,常量池中没有`hello`,所以s2直接指向了s1字符串的引用,因此s1==s2

(4) 常见实例

```java
String s0 = "asdf";
String s1 = "asdf";
String s2 = "as"+"df";
System.out.println( s0==s1 ); //true
System.out.println( s0==s2 ); //true
```

因为例子中的 s0和s1中的”asdf”都是字符串常量，它们在编译期就被确定了,所以s0=s1,而”as”和”df”也都是字符串常量，当一个字符串由多个字符串常量连接而成时，它自己肯定也是字符串常量，所以s2也同样在编译期就被优化为一个字符串常量"asdf"，所以s2也是常量池中” asdf”的一个引用。所以我们得出s0==s1==s2

```java
String s0="hello";
String s1=new String("hello");
String s2="he" + new String("llo");
System.out.println( s0==s1 ); //false
System.out.println( s0==s2 ); //false
System.out.println( s1==s2 ); // false
```

用`new String()`创建的字符串不是常量，不能在编译期就确定，所以`new String()`创建的字符串不放入常量池中，它们有自己的地址空间

```java
String a = "a1";
String b = "a" + 1;
System.out.println(a == b); // true

String a = "atrue";
String b = "a" + "true";
System.out.println(a == b); // true

String a = "a3.4";
String b = "a" + 3.4;
System.out.println(a == b); // true
```

JVM对于字符串常量的"+"号连接，将在程序编译期，JVM就将常量字符串的"+"连接优化为连接后的值

```java
String a = "ab";
String bb = "b";
String b = "a" + bb;

System.out.println(a == b); // false
```

JVM对于字符串引用，由于在字符串的"+"连接中，有字符串引用存在，而引用的值在程序编译期是无法确定的，即"a" + bb无法被编译器优化，只有在程序运行期来动态分配并将连接后的新地址赋给b。所以上面程序的结果也就为false。

```java
String a = "ab";
final String bb = "b";
String b = "a" + bb;
System.out.println(a == b); // true
```

和上面示例中唯一不同的是bb字符串加了final修饰，对于final修饰的变量，它在编译时被解析为常量值的一个本地拷贝存储到自己的常量池中或嵌入到它的字节码流中。所以此时的"a" + bb和"a" + "b"效果是一样的。故上面程序的结果为true。

```java
String a = "ab";
final String bb = getBB();
String b = "a" + bb;

System.out.println(a == b); // false

private static String getBB()
{  
    return "b";  
}
```

JVM对于字符串引用bb，它的值在编译期无法确定，只有在程序运行期调用方法后，将方法的返回值和"a"来动态连接并分配地址为b，故上面程序的结果为false

```java
String  s  =  "a" + "b" + "c";  //就等价于String s = "abc";
String  a  =  "a";
String  b  =  "b";
String  c  =  "c";
String  s1  =   a  +  b  +  c;
```

通过观察其JVM指令码发现s1的"+"操作会变成如下操作：

```java
StringBuilder temp = new StringBuilder();
temp.append(a).append(b).append(c);
String s = temp.toString();
```

```java

//字符串常量池："计算机"和"技术"     堆内存：str1引用的对象"计算机技术"  
//堆内存中还有个StringBuilder的对象，但是会被gc回收，StringBuilder的toString方法会new String()，这个String才是真正返回的对象引用
String str2 = new StringBuilder("计算机").append("技术").toString();   //没有出现"计算机技术"字面量，所以不会在常量池里生成"计算机技术"对象
System.out.println(str2 == str2.intern());  //true
//"计算机技术" 在池中没有，但是在heap中存在，则intern时，会直接返回该heap中的引用

//字符串常量池："ja"和"va"     堆内存：str1引用的对象"java"  
//堆内存中还有个StringBuilder的对象，但是会被gc回收，StringBuilder的toString方法会new String()，这个String才是真正返回的对象引用
String str1 = new StringBuilder("ja").append("va").toString();    //没有出现"java"字面量，所以不会在常量池里生成"java"对象
System.out.println(str1 == str1.intern());  //false
//java是关键字，在JVM初始化的相关类里肯定早就放进字符串常量池了

String s1=new String("test");  
System.out.println(s1==s1.intern());   //false
//"test"作为字面量，放入了池中，而new时s1指向的是heap中新生成的string对象，s1.intern()指向的是"test"字面量之前在池中生成的字符串对象

String s2=new StringBuilder("abc").toString();
System.out.println(s2==s2.intern());  //false 同上
```

#### 八种基本类型的包装类和对象池

java中基本类型的包装类的大部分都实现了常量池技术(严格来说应该叫对象池，在堆上)，这些类是Byte,Short,Integer,Long,Character,Boolean,另外两种浮点数类型的包装类则没有实现.另外Byte,Short,Integer,Long,Character这5种整型的包装类也只是在对应值小于等于127时才可使用对象池，也即对象不负责创建和管理大于127的这些类的对象。

```java
//5种整形的包装类Byte,Short,Integer,Long,Character的对象，  
//在值小于127时可以使用对象池  
Integer i1 = 127;  //这种调用底层实际是执行的Integer.valueOf(127)，里面用到了IntegerCache对象池
Integer i2 = 127;
System.out.println(i1 == i2);//输出true  

//值大于127时，不会从对象池中取对象  
Integer i3 = 128;
Integer i4 = 128;
System.out.println(i3 == i4);//输出false  

//用new关键词新生成对象不会使用对象池
Integer i5 = new Integer(127);  
Integer i6 = new Integer(127);
System.out.println(i5 == i6);//输出false

//Boolean类也实现了对象池技术  
Boolean bool1 = true;
Boolean bool2 = true;
System.out.println(bool1 == bool2);//输出true  

//浮点类型的包装类没有实现对象池技术  
Double d1 = 1.0;
Double d2 = 1.0;
System.out.println(d1 == d2);//输出false  
```
