---
title: thymeleaf方言和处理器简介
date: 2017-11-02 15:43:23
thumbnail: files/thumb/1.jpg
tags:
- thymeleaf
categories:
- java
---
thymeleaf是一个很容易扩展的库，大部分面向用户的功能不是直接构建在他的核心中，而是通过打包和组件化到一个称谓Dialect(方言)的功能集合中。
由于Spring-boot间接式的主推了thymeleaf模板，所以本文主要介绍下thymeleaf的一些扩展功能，尽管thymeleaf3的性能相比2来说提升了很大，但是相比别的，还是有一点差距。
<!-- more -->

## 方言(Dialects)
如果我们研究过thymeleaf的一些基础使用的话，应该能意识到我们之前了解的准确的说不是thymeleaf，而是thymeleaf的标准方言,例如th:text，仅仅只是可以立即使用的标准方言。
同时我们可以自定义一组attribute或者tag在thymeleaf中用来处理我们的模板。

Dialects是实现了` org.thymeleaf.dialect.IDialect`接口的对象,具体如下：
```java
public interface IDialect {

    public String getName();

}
```
同时最基础的接口有：
* IProcessorDialect 处理器方言
* IPreProcessorDialect 预处理方言
* IPostProcessorDialect 后处理方言
* IExpressionObjectDialect 表达式对象方言
* IExecutionAttributeDialect 可执行属性方言

### IProcessorDialect 处理器方言
参看接口代码
```java
public interface IProcessorDialect extends IDialect {
    public String getPrefix();
    public int getDialectProcessorPrecedence();
    public Set<IProcessor> getProcessors(final String dialectPrefix);
}
```
processor是负责执行thymeleaf模板中的大部分逻辑的对象。也是最重要的扩展方言。
定义了三个主要是属性方法。
* prefix 应用于匹配元素和属性的前缀，类似于th:if,thLtext中的th。如果希望处理器在未定义的标签或者属性上执行，则可以将prefix定义为null
* getDialectProcessorPrecedence 定义方言的优先级。
* getProcessors 定义一组由该方言提供的处理器集合。

### IPreProcessorDialect 预处理方言
```java
public interface IPreProcessorDialect extends IDialect {
    public int getDialectPreProcessorPrecedence();
    public Set<IPreProcessor> getPreProcessors();
}
```
预处理和后处理与处理器不同。处理器是在单个时间或者模板片段上执行。而预处理和后处理是作为引擎处理过程中的附加步骤，应用在整个模板的执行过程中。
因此他们遵循与处理器完成不同的API，他们更加面向事件。
预处理在特定的情况下，是在为特定模板执行处理器之前应用的。后处理器则相反，是在执行处理器之后应用。

###  IPostProcessorDialec 后处理方言
```java
public interface IPostProcessorDialect extends IDialect {
    public int getDialectPostProcessorPrecedence();
    public Set<IPostProcessor> getPostProcessors();
}
```

### IExpressionObjectDialect 表达式对象方言
通过实现此接口，Dialect可以提供新的表达式对象或者表达式应用程序对象，例如#strings,#numbers等
```java
public interface IExpressionObjectDialect extends IDialect {
    public IExpressionObjectFactory getExpressionObjectFactory();
}
```
通过代码可以看出，IExpressionObjectDialect返回了一个工厂类，原因是一些表达式对象需要处理上下文的数据才能被构建，所以在我们真正处理模板之前不可能构建他们。此外，大多数表达式并不需要表达式对象，所以只有在特定表达式真正需要的时候才能按需构建他们。

```java
public interface IExpressionObjectFactory {
    public Map<String,ExpressionObjectDefinition> getObjectDefinitions();
    public Object buildObject(final IProcessingContext processingContext, final String expressionObjectName);
}
```

### IExecutionAttributeDialect 可执行属性方言
实现这个接口的方言被允许提供执行属性，即在模板处理期间执行的每个处理器可用的对象。
例如StandardDialect实现这个接口，以便为每个处理器提供以下功能：
* Thymeleaf标准表达式解析器，以便可以解析和执行任何属性中的标准表达式
* 变量表达式计算器` ${...}`表达式在OGNL或SpringEL中执行
注意，这些对象在上下文中不可用，因此它们不能在模板表达式中使用。它们的可用性仅限于扩展点的实现，例如处理器，预处理器等。

```java
public interface IExecutionAttributeDialect extends IDialect {
    public Map<String,Object> getExecutionAttributes();
}
```

## Processors处理器
处理器的对象全部实现` org.thymeleaf.processor.IProcessor`接口。
接口代码如下：
```java
public interface IProcessor {
    public TemplateMode getTemplateMode();
    public int getPrecedence();
}
```
接下来介绍下几种常见类型的处理器

### IElementProcessor
元素处理器是在open element 或者独立元素上执行。
```java
public interface IElementProcessor extends IProcessor {
    public MatchingElementName getMatchingElementName();
    public MatchingAttributeName getMatchingAttributeName();

}
```

IElementProcessor处理器并不是直接实现这个接口，它还包含了两个子接口
* IElementTagProcessor
* IElementModelProcessor

### ITemplateBoundariesProcessor
### 其他处理器
以下是Thymeleaf 3.0允许声明处理器的其他事件，它们中的每一个都实现了相应的接口：
* Text events: 接口 ` ITextProcessor`
* Comment events: 接口 ` ICommentProcessor`
* CDATA Section events: 接口 ` ICDATASectionProcessor`
* DOCTYPE Clause events: 接口 ` IDocTypeProcessor`
* XML Declaration events: 接口 ` IXMLDeclarationProcessor`
* Processing Instruction events: 接口 ` IProcessingInstructionProcessor`

**处理器并没有详细介绍，后续通过具体的实际使用来说明**

