---
title:  Spring(AbstractRoutingDataSource)实现动态数据源切换
date: 2016-07-05 17:41:25
tags:
- Spring
categories:
- Spring
toc: true
---
原始出处：http://linhongyu.blog.51cto.com/6373370/1615895
### 前言

近期一项目A需实现数据同步到另一项目B数据库中，在不改变B项目的情况下，只好选择项目A中切换数据源，直接把数据写入项目B的数据库中。这种需求，在数据同步与定时任务中经常需要。
<!-- more -->
那么问题来了，该如何解决多数据源问题呢？不光是要配置多个数据源，还得能灵活动态的切换数据源。以spring+hibernate框架项目为例
![](/files/article/java/4.jpg)
单个数据源绑定给sessionFactory，再在Dao层操作，若多个数据源的话，那不是就成了下图：
![](/files/article/java/5.jpg)
可见，sessionFactory都写死在了Dao层，若我再添加个数据源的话，则又得添加一个sessionFactory。所以比较好的做法应该是下图：

![](/files/article/java/6.jpg)

接下来就为大家讲解下如何用spring来整合这些数据源，同样以spring+hibernate配置为例。

### 实现原理
1、 扩展Spring的AbstractRoutingDataSource抽象类（该类充当了DataSource的路由中介, 能有在运行时, 根据某种key值来动态切换到真正的DataSource上。）
从AbstractRoutingDataSource的源码中：

```java
public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean
```
我们可以看到，它继承了AbstractDataSource，而AbstractDataSource不就是javax.sql.DataSource的子类，So我们可以分析下它的getConnection方法：

```java
public Connection getConnection() throws SQLException {  
    return determineTargetDataSource().getConnection();  
}  
   
public Connection getConnection(String username, String password) throws SQLException {  
    return determineTargetDataSource().getConnection(username, password);  
}
```
获取连接的方法中，重点是determineTargetDataSource()方法，看源码：

```java
protected DataSource determineTargetDataSource() {  
    Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");  
    Object lookupKey = determineCurrentLookupKey();  
    DataSource dataSource = this.resolvedDataSources.get(lookupKey);  
    if (dataSource == null && (this.lenientFallback || lookupKey == null)) {  
        dataSource = this.resolvedDefaultDataSource;  
    }  
    if (dataSource == null) {  
        throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");  
    }  
    return dataSource;  
    }
```
上面这段源码的重点在于determineCurrentLookupKey()方法，这是AbstractRoutingDataSource类中的一个抽象方法，而它的返回值是你所要用的数据源dataSource的key值，有了这个key值，resolvedDataSource（这是个map,由配置文件中设置好后存入的）就从中取出对应的DataSource，如果找不到，就用配置默认的数据源。
看完源码，应该有点启发了吧，没错！你要扩展AbstractRoutingDataSource类，并重写其中的determineCurrentLookupKey()方法，来实现数据源的切换：

```java
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
 
/**
 * 获取数据源（依赖于spring）
 */
public class DynamicDataSource extends AbstractRoutingDataSource{
    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceHolder.getDataSource();
    }
}
```
DataSourceHolder这个类则是我们自己封装的对数据源进行操作的类：

```java
public class DataSourceHolder {
    //线程本地环境
    private static final ThreadLocal<String> dataSources = new ThreadLocal<String>();
    //设置数据源
    public static void setDataSource(String customerType) {
        dataSources.set(customerType);
    }
    //获取数据源
    public static String getDataSource() {
        return (String) dataSources.get();
    }
    //清除数据源
    public static void clearDataSource() {
        dataSources.remove();
    }
}
```
2、有人就要问，那你setDataSource这方法是要在什么时候执行呢？当然是在你需要切换数据源的时候执行啦。手动在代码中调用写死吗？这是多蠢的方法，当然要让它动态咯。所以我们可以应用spring aop来设置，把配置的数据源类型都设置成为注解标签，在service层中需要切换数据源的方法上，写上注解标签，调用相应方法切换数据源咯（就跟你设置事务一样）：
```java
@DataSource(name=DataSource.slave1)
public List getProducts(){}
```
当然，注解标签的用法可能很少人用到，但它可是个好东西哦，大大的帮助了我们开发：

```java
import java.lang.annotation.*;
 
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSource {
    String name() default DataSource.master;
 
    public static String master = "dataSource1";
 
    public static String slave1 = "dataSource2";
 
    public static String slave2 = "dataSource3";
 
}
```

### 配置文件
项目中单独分离出application-database.xml，关于数据源配置的文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Spring 数据库相关配置 放在这里 -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
 
    <bean id = "dataSource1" class = "com.mysql.jdbc.jdbc2.optional.MysqlDataSource">   
        <property name="url" value="${db1.url}"/>
        <property name = "user" value = "${db1.user}"/>
        <property name = "password" value = "${db1.pwd}"/>
        <property name="autoReconnect" value="true"/>
        <property name="useUnicode"  value="true"/>
        <property name="characterEncoding" value="UTF-8"/>
    </bean>
 
    <bean id = "dataSource2" class = "com.mysql.jdbc.jdbc2.optional.MysqlDataSource">
        <property name="url" value="${db2.url}"/>
        <property name = "user" value = "${db2.user}"/>
        <property name = "password" value = "${db2.pwd}"/>
        <property name="autoReconnect" value="true"/>
        <property name="useUnicode"  value="true"/>
        <property name="characterEncoding" value="UTF-8"/>
    </bean>
 
    <bean id = "dataSource3" class = "com.mysql.jdbc.jdbc2.optional.MysqlDataSource">
        <property name="url" value="${db3.url}"/>
        <property name = "user" value = "${db3.user}"/>
        <property name = "password" value = "${db3.pwd}"/>
        <property name="autoReconnect" value="true"/>
        <property name="useUnicode"  value="true"/>
        <property name="characterEncoding" value="UTF-8"/>
    </bean>
    <!-- 配置多数据源映射关系 -->
    <bean id="dataSource" class="com.datasource.test.util.database.DynamicDataSource">
        <property name="targetDataSources">
            <map key-type="java.lang.String">
        <entry key="dataSource1" value-ref="dataSource1"></entry>
                <entry key="dataSource2" value-ref="dataSource2"></entry>
                <entry key="dataSource3" value-ref="dataSource3"></entry>
            </map>
        </property>
    <!-- 默认目标数据源为你主库数据源 -->
        <property name="defaultTargetDataSource" ref="dataSource1"/>
    </bean>
 
    <bean id="sessionFactoryHibernate" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">com.datasource.test.util.database.ExtendedMySQLDialect</prop>
                <prop key="hibernate.show_sql">${SHOWSQL}</prop>
                <prop key="hibernate.format_sql">${SHOWSQL}</prop>
                <prop key="query.factory_class">org.hibernate.hql.classic.ClassicQueryTranslatorFactory</prop>
                <prop key="hibernate.connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</prop>
                <prop key="hibernate.c3p0.max_size">30</prop>
                <prop key="hibernate.c3p0.min_size">5</prop>
                <prop key="hibernate.c3p0.timeout">120</prop>
                <prop key="hibernate.c3p0.idle_test_period">120</prop>
                <prop key="hibernate.c3p0.acquire_increment">2</prop>
                <prop key="hibernate.c3p0.validate">true</prop>
                <prop key="hibernate.c3p0.max_statements">100</prop>
            </props>
        </property>
    </bean>
 
    <bean id="hibernateTemplate" class="org.springframework.orm.hibernate3.HibernateTemplate">
        <property name="sessionFactory" ref="sessionFactoryHibernate"/>
    </bean>
 
    <bean id="dataSourceExchange" class="com.datasource.test.util.database.DataSourceExchange"/>
 
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactoryHibernate"/>
    </bean>
 
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="insert*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="add*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="update*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="modify*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="edit*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="del*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="save*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="send*" propagation="NESTED" rollback-for="Exception"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="query*" read-only="true"/>
            <tx:method name="search*" read-only="true"/>
            <tx:method name="select*" read-only="true"/>
            <tx:method name="count*" read-only="true"/>
        </tx:attributes>
    </tx:advice>
 
    <aop:config>
        <aop:pointcut id="service" expression="execution(* com.datasource..*.service.*.*(..))"/>
        <!-- 关键配置，切换数据源一定要比持久层代码更先执行（事务也算持久层代码） -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="service" order="2"/>
        <aop:advisor advice-ref="dataSourceExchange" pointcut-ref="service" order="1"/>
    </aop:config>
 
</beans>
```
### 疑问
多数据源切换是成功了，但牵涉到事务呢？单数据源事务是ok的，但如果多数据源需要同时使用一个事务呢？这个问题有点头大，网络上有人提出用atomikos开源项目实现JTA分布式事务处理。你怎么看？
