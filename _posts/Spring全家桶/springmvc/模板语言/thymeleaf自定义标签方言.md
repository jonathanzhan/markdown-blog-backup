---
title: thymeleaf自定义标签方言
date: 2017-11-06 15:12:50
thumbnail: /files/thumb/2.jpg
tags:
- thymeleaf
categories:
- java
toc: true
---
在上几篇文章中，讲解了thymeleaf的方言定义以及处理器等，接下来通过一个具体的使用来深度了解下thymeleaf方言和处理器的具体使用
<!-- more -->
## 实例1
table表格中某一列显示内容为是否启用，具体的值为0和1.如果是0，该单元格是红色,否则为绿色。

### 定义方言
首先，我们需要定义一个thymeleaf的方言。具体如下:
```java
public class WorkFocusDialect extends AbstractProcessorDialect {

    private final IExpressionObjectFactory EXPRESSION_OBJECTS_FACTORY = new WorkFocusExpressionFactory();

    private static final String DIALECT_NAME = "workfocus";

    private static final String PREFIX = "wlf";

    public static final int PROCESSOR_PRECEDENCE = 1000;

    public WorkFocusDialect() {
        // We will set this dialect the same "dialect processor" precedence as
        // the Standard Dialect, so that processor executions can interleave.
        super(DIALECT_NAME, PREFIX, PROCESSOR_PRECEDENCE);
    }
    @Override
    public Set<IProcessor> getProcessors(final String dialectPrefix) {
        final Set<IProcessor> processors = new HashSet<IProcessor>();
        ... 在这里增加自定义的处理器
        processors.add(new SampleAttributeTagProcessor(dialectPrefix));
        processors.add(new SampleElementTagProcessor(dialectPrefix));
        return processors;
    }
}
```


### 定义处理器
在上篇文章中，我们了解到如果需要自定义标签的话，其实本质上是需要定义thymeleaf处理器。具体的实现如下
```java
public class SampleAttributeTagProcessor extends AbstractAttributeTagProcessor {

    private static final String ATTR_NAME = "sample1";
    private static final int PRECEDENCE = 10000;

    public SampleAttributeTagProcessor(final String dialectPrefix) {
        super(
            TemplateMode.HTML, // This processor will apply only to HTML mode
            dialectPrefix,     // Prefix to be applied to name for matching
            null,              // No tag name: match any tag name
            false,             // No prefix to be applied to tag name
            ATTR_NAME,         // Name of the attribute that will be matched
            true,              // Apply dialect prefix to attribute name
            PRECEDENCE,        // Precedence (inside dialect's own precedence)
            true);             // Remove the matched attribute afterwards
    }


    @Override
    protected void doProcess(
            final ITemplateContext context, final IProcessableElementTag tag,
            final AttributeName attributeName, final String attributeValue,
            final IElementTagStructureHandler structureHandler) {

        final IEngineConfiguration configuration = context.getConfiguration();
        /*
         * Obtain the Thymeleaf Standard Expression parser
         */
        final IStandardExpressionParser parser =
                StandardExpressions.getExpressionParser(configuration);

        /*
         * Parse the attribute value as a Thymeleaf Standard Expression
         */
        final IStandardExpression expression = parser.parseExpression(context, attributeValue);
        /*
         * Execute the expression just parsed
         */
        final Integer position = (Integer) expression.execute(context);

        if(position.equals(1)) {
            structureHandler.setAttribute("style", "background:green");
        } else {
            structureHandler.setAttribute("style", "background:red");
        }
    }
```

### 添加到thymeleaf引擎中
```java
@Bean
@ConditionalOnMissingBean
public WorkFocusDialect wlfDialect() {
    return new WorkFocusDialect();
}
```

### 具体使用
```html
<table>
<tr>
    <td>....</td>
    <td wlf:sample1="${user.status}" th:text="${user.status}">状态</td>
</tr>
</table>
```

## 实例2
在上面的实例中，我们仅仅是根据具体的值，改变了td元素的属性值而已，所以在处理器中继承的是` AbstractAttributeTagProcessor`类。
接下来我们在上面的例子的基础上继续进行扩展。
如果user.status==1 ，则显示 
```html
<td style="background:green">启用</td>
````
else，则显示 

```html
<td style="background:red">停用</td>
```
### 定义处理器
本次继承的类是` AbstractElementTagProcessor`
```java
public class Sample3ElementTagProcessor extends AbstractElementTagProcessor {

    private static final String TAG_NAME = "sample3";
    private static final int PRECEDENCE = 1000;

    public Sample3ElementTagProcessor(final String dialectPrefix) {
        super(
            TemplateMode.HTML, // This processor will apply only to HTML mode
            dialectPrefix,     // Prefix to be applied to name for matching
            TAG_NAME,          // Tag name: match specifically this tag
            true,              // Apply dialect prefix to tag name
            null,              // No attribute name: will match by tag name
            false,             // No prefix to be applied to attribute name
            PRECEDENCE);       // Precedence (inside dialect's own precedence)
    }


    @Override
    protected void doProcess(
            final ITemplateContext context, final IProcessableElementTag tag,
            final IElementTagStructureHandler structureHandler) {

        /*
         * Read the 'order' attribute from the tag. This optional attribute in our tag 
         * will allow us to determine whether we want to show a random headline or
         * only the latest one ('latest' is default).
         */
        final String statusValue = tag.getAttributeValue("status");

        final IEngineConfiguration configuration = context.getConfiguration();
        /*
         * Obtain the Thymeleaf Standard Expression parser
         */
        final IStandardExpressionParser parser = StandardExpressions.getExpressionParser(configuration);

        final IStandardExpression expression = parser.parseExpression(context, statusValue);

        final Integer parseStatus = (Integer) expression.execute(context);
        /*
         * Create the DOM structure that will be substituting our custom tag.
         */
        final IModelFactory modelFactory = context.getModelFactory();
        final IModel model = modelFactory.createModel();
        if(parseStatus.equals(0)) {
            model.add(modelFactory.createOpenElementTag("td", "style", "background:green"));
            model.add(modelFactory.createText(HtmlEscape.escapeHtml5("停用")));
        }else {
            model.add(modelFactory.createOpenElementTag("td", "style", "background:red"));
            model.add(modelFactory.createText(HtmlEscape.escapeHtml5("启用")));
        }
        model.add(modelFactory.createCloseElementTag("td"));


        /*
         * Instruct the engine to replace this entire element with the specified model.
         */
        structureHandler.replaceWith(model, false);
        
    }

}

```

### 具体使用
```html
<table id="contentTable" class="table table-striped table-bordered table-hover table-condensed">
    <thead>
    <tr>
        <th>序号</th>
        <th>登录名</th>
        <th>用户名</th>
        <th>状态</th>
    </tr>
    </thead>
    <tbody>
        <tr th:each="user: ${userList}" >
            <td th:text="${userStat.count}">1</td>
            <td wlf:sample1="${user.status}" th:text="${user.status}"></td>
            <wlf:sample3 status="${user.status}"/>
            <td th:text="${user.status}"></td>
        </tr>

    </tbody>
</table>
```

## 补充
在处理器中,我们可以根据上下文获取Spring对象，比如获取某个Service.
```java
final ApplicationContext appCtx = SpringContextUtils.getApplicationContext(ITemplateContext context);

final XXXService XXXService = appCtx.getBean(XXXService.class);
```
