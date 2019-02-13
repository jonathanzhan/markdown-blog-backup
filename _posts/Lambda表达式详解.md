---
title: Lambda表达式详解
date: 2018-09-03 18:14:39
thumbnail: /files/thumb/3.jpg
tags:
- java基础
categories:
- java
toc: true
---
Lambda表达式是Java SE 8中一个重要的新特性。lambda表达式允许你通过表达式来代替功能接口。 lambda表达式就和方法一样,它提供了一个正常的参数列表和一个使用这些参数的主体(body,可以是一个表达式或一个代码块)。
<!--more-->
Lambda表达式的语法
基本语法:
(parameters) -> expression
或
(parameters) ->{ statements; }

Lambda表达式一共有三部分组成：
![lambda表达式](/files/java/lambda.png)


下面是Java lambda表达式的简单例子:

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)  
```

**Lambda 表达式需要 “函数式接口” 的支持**

函数式接口 : 接口中只有一个抽象方法的接口，称为函数式接口，可以通过 Lambda 表达式来创建该接口的对象 (若 Lambda表达式抛出一个受检异常，那么该异常需要在目标接口的抽象方法上进行声明) 可以使用注解 @FunctionalInterface 修饰可以检查是否是函数式接口，同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口
```java
@FunctionalInterface
public interface fun {
    Integer getValue(Integer num);
}

函数式接口中使用泛型 :
@FunctionalInterface
public interface fun<T> {
    T getValue(T t);
}
```

### 基本的Lambda例子

雇员对象如下，有一个包含许多员工信息的对象 employees，要求获取当前公司中员工年龄大于 35 的员工信息

```java
public class Employee {
    private String name;
    private int age;
    public Employee() {
    }
    public Employee(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // ...
    @Override
    public String toString() {
        return "Employee{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
}
```

最简单的方式是采用 foreach 循环遍历，以下是各种优化方式
优化方式一 : 策略设计模式
```java
@FunctionalInterface
public interface MyPredicate<T> {
    boolean test(T t);
}
public class FilterEmployeeByAge implements MyPredicate<Employee> {
    @Override
    public boolean test(Employee employee) {
        return employee.getAge() >= 35;
    }
}
public List<Employee> filterEmployees3(List<Employee> employees, MyPredicate<Employee> myPredicate) {
    List<Employee> emps = new ArrayList<>();
    for (Employee employee : employees) {
        if (myPredicate.test(employee)) {
            emps.add(employee);
        }
    }
    return emps;
}
```
调用
```java
List<Employee> emps = filterEmployees3(employees, new FilterEmployeeByAge());
for (Employee employee : emps) {
    System.out.println(employee.toString());
}
```

优化方式二 : 匿名内部类
```java
List<Employee> emps = filterEmployees3(employees, new MyPredicate<Employee>() {
    @Override
    public boolean test(Employee employee) {
        return employee.getAge() >= 35;
    }
});
for (Employee employee : emps) {
    System.out.println(employee.toString());
}
```

优化方式三 : Lambda
```java
filterEmployees3(employees, employee -> employee.getAge() >= 35).forEach(System.out::println);
```

### 方法引用
若Lambda体中的功能，已经有方法提供实现，可以使用方法引用 (可以将方法引用理解为Lambda表达式的另外一种表现形式)

1. 对象的引用 :: 实例方法名
2. 类名 :: 静态方法名
3. 类名 :: 实例方法名

#### 对象的引用 :: 实例方法名

```java
Employee emp = new Employee(101, "张三", 18, 9999.99);
Supplier<String> sup = () -> emp.getName();
System.out.println(sup.get());
Supplier<String> sup2 = emp::getName;
System.out.println(sup2.get());
```

#### 类名 :: 静态方法名
```java
Comparator<Integer> comparator1 = (x, y) -> Integer.compare(x, y);
Comparator<Integer> comparator2 = Integer::compare;

BiFunction<Double, Double, Double> fun = (x, y) -> Math.max(x, y);
System.out.println(fun.apply(1.5, 22.2));
BiFunction<Double, Double, Double> fun2 = Math::max;
System.out.println(fun2.apply(1.2, 1.5));
```


#### 类名 :: 实例方法名
```java
BiPredicate<String, String> bp = (x, y) -> x.equals(y);
System.out.println(bp.test("abcde", "abcde"));
BiPredicate<String, String> bp2 = String::equals;
System.out.println(bp2.test("abc", "abc"));

Function<Employee, String> fun = (e) -> e.show();
System.out.println(fun.apply(new Employee()));
Function<Employee, String> fun2 = Employee::show;
System.out.println(fun2.apply(new Employee()));
```

#### 构造器引用
构造器的参数列表，需要与函数式接口中参数列表保持一致 (就是函数签名一致)

```java
Supplier<Employee> sup = () -> new Employee();
System.out.println(sup.get());
Supplier<Employee> sup2 = Employee::new;
System.out.println(sup2.get());

Function<String, Employee> fun = Employee::new;
System.out.println(fun.apply("张三"));
BiFunction<String, Integer, Employee> fun2 = Employee::new;
System.out.println(fun2.apply("张三",20));
```

#### 数组引用

```java
Function<Integer, String[]> fun = (args) -> new String[args];
String[] strs = fun.apply(10);
System.out.println(strs.length);

Function<Integer, Employee[]> fun2 = Employee[]::new;
Employee[] emps = fun2.apply(20);
System.out.println(emps.length);
```


### Java8 内置的四大核心函数式接口

| 函数式接口 | 参数类型 | 返回类型 | 用途 |
| :- | :- | :- | :- |
| Consumer<T> 消费型接口 | T | void | 对类型为T的对象应用操作，包含方法 :void accept(T t)| 
| Consumer<T> 供给型接口 | 无 | T | 返回类型为T的对象，包含方法 :T get()| 
| Function<T, R> 函数型接口 | T | R | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象，包含方法 :R apply(T t)| 
| Predicate<T> 断定型接口 | T | boolean | 确定类型为T的对象是否满足某约束，并返回boolean 值，包含方法 :boolean test(T t)| 


### 其他接口

| 函数式接口 | 参数类型 | 返回类型 | 用途 |
| :- | :- | :- | :- |
| BiFunction<T, U, R> | T, U | R | 对类型为T, U参数应用操作,返回R类型的结果，包含方法 :R apply(T t, U u))| 
| UnaryOperator<T> | T | T | 对类型为T的对象进行一元运算， 并返回T类型的结果，包含方法 :T apply(T t)| 
| BinaryOperator<T> | T,T | T | 对类型为T的对象进行二元运算， 并返回T类型的结果，包含方法 :T apply(T t1, T t2)| 
| BiConsumer<T, U> | T, U | void | 对类型为T, U 参数应用操作，包含方法 :void accept(T t, U u)| 
| ToIntFunction<T> ToLongFunction<T> ToDoubleFunction<T> | T | int long double | 分别计算int 、long 、double值的函数| 
| IntFunction<T> LongFunction<T> DoubleFunction<T> | int long double | T | 参数分别为int、 long、double 类型的函数| 