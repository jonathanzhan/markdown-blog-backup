---
title: thymeleaf自定义工具对象详解
date: 2017-11-02 10:39:38
tags:
- thymeleaf
categories:
- java
toc: true
---
Thymeleaf还提供了一系列Utility对象，通过#来访问，比如strings，dates等等，但在一些特殊的情况下，内置的对象并不能满足我们的使用。所以本文主要介绍下如何自定义工具对象表达式。
<!-- more -->
## 简介
在thymeleaf中,提供了很多的工具对象来帮助我们完成一些常规的操作，比如#uris,#strings,#numbers等，我个人把他们定义为工具对象，可能定义的不太准确。简单点说也就是我们项目中比较常用的一些工具类了，比如google.guave,commons-lang等等。接下来，就讲下如何在thymeleaf中使用commons-lang中的StringUtils类。


## 定义Dialect
我们接触到的Dialect有StandardDialect,以及SpringStandardDialect。如何仔细研究这两个类的话，那么基本上也就可以完成自定义工具对象的操作了。
首先，通过继承AbstractDialect来定义一个dialect对象。
```java
public class WorkFocusDialect extends AbstractDialect implements IExpressionObjectDialect {
    ...
}
```
**通过实现IExpressionObjectDialect接口，可以完成自定义的工具对象的工厂类。**

```java
private final IExpressionObjectFactory EXPRESSION_OBJECTS_FACTORY = new WorkFocusExpressionFactory();

@Override
public IExpressionObjectFactory getExpressionObjectFactory() {
    return this.EXPRESSION_OBJECTS_FACTORY;
}
```

在这里 我们定义了一个表达式对象的工厂类，主要用来提供我们后续将要继承的工具对象，比如StringUtils.

## 定义IExpressionObjectFactory

```java
public class WorkFocusExpressionFactory implements IExpressionObjectFactory {
    ...
}
```
通过实现IExpressionObjectFactory接口来完成表达式对象工厂类的定义。

该接口提供了以下方法
```java
// 返回该工厂类能创建的工具类对象的集合。
public Set<String> getAllExpressionObjectNames();

// 根据表达式的名称,创建工具类对象
public Object buildObject(final IExpressionContext context, final String expressionObjectName);

// 返回该工具对象是否可缓存。(可能理解的不太到位)
public boolean isCacheable(final String expressionObjectName);
```

接下来看下WorkFocusExpressionFactory类的具体的实现：

```java
public static final String STRING_UTILS_EXPRESSION_OBJECT_NAME = "stringUtils";

private static final StringUtils stringUtils = new StringUtils();


public static final Set<String> ALL_EXPRESSION_OBJECT_NAMES;


static {

    final Set<String> allExpressionObjectNames = new LinkedHashSet<String>();
    allExpressionObjectNames.add(STRING_UTILS_EXPRESSION_OBJECT_NAME);
    ALL_EXPRESSION_OBJECT_NAMES = Collections.unmodifiableSet(allExpressionObjectNames);

}
public WorkFocusExpressionFactory(){
    super();
}

@Override
public Set<String> getAllExpressionObjectNames() {
    return ALL_EXPRESSION_OBJECT_NAMES;
}

@Override
public Object buildObject(IExpressionContext context, String expressionObjectName) {
    return STRING_UTILS_EXPRESSION_OBJECT_NAME.equals(expressionObjectName) ? stringUtils : null;

}

public boolean isCacheable(String expressionObjectName) {
    return expressionObjectName != null && "stringUtils".equals(expressionObjectName);
}

```

## 最后将自定义的Dialect添加到模板引擎中。
```java
@Bean
public SpringTemplateEngine templateEngine(){
    SpringTemplateEngine templateEngine = new SpringTemplateEngine();
    templateEngine.setTemplateResolver(templateResolver());
    templateEngine.addDialect(new WorkFocusDialect());
    return templateEngine;
}
```

addDialect方法将我们自定义的方言WorkFocusDialect添加到模板引擎中，然后我们就可以在模板中通过
` #stringUtils.isEmpty(str) `来完成对StringUtils类的调用。

如果使用的是SpringBoot,并且有使用spring-boot-starter-thymeleaf的话，只需要配置该方言对象即可。
```java
    @Bean
    @ConditionalOnMissingBean
    public WorkFocusDialect wlfDialect() {
        return new WorkFocusDialect();
    }
```
想了解具体为什么不需要通过addDialect方法来添加的话，可以查看Spring-boot的源码中的ThymeleafAutoConfiguration类。






