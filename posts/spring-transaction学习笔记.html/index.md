# Spring Transaction学习笔记


## 前言

在 Spring 框架中，需要使用事务管理对数据库进行操作，以实现事务的 `ACID` 特性。

## 事务的基本实现

Spring 框架提供了三个对象来管理事务。

1. `PlatformTransactionManager` ：平台事务管理器，spring要管理事务，必须使用事务管理器。事务配置的过程中，也必须配置事务管理器，常见的事务管理器有：

   - `DataSourceTransactionManager` ，在jdbc开发中采用 `JdbcTemplate` 。(常用)
   - `HibernateTransactionManage` ，hibernate开发时事务管理器，整合hibernate。

   常用 `api` 有三个：

   - `TransactionStatus getTransaction(TransactionDefinition definition) ` ： 事务管理器 通过“事务详情”，获得“事务状态”，从而管理事务。 获取事务状态后，Spring根据传播行为来决定如何开启事务
   - `void commit(TransactionStatus status) ` ： 根据状态提交
   - `void rollback(TransactionStatus status)` ： 根据状态回滚

2. `TransactionDefinition` ：这个接口的作用就是定义事务的名称、隔离级别、传播行为、超时时间长短、只读属性等。

3. `TransactionStatus` ： 这个接口的作用就是获取事务的状态（回滚点、是否完成、是否新事物、是否回滚）属性

### 基本流程

首先需要导入 `spring-tx` 包。

实现方式有四种：

1. 使用事务模板完成

   - 向容器中注册 `TransactionTemplate` 对象，然后在`Service ` 类中引入事务管理器

   ```xml
   <bean id="myUserServiceImpl" class="com.cskaoyan.service.UserServiceImpl">
       <property name="dao" ref="accountdaoImpl"></property>
       <property name="transactionTemplate" ref="myTransactionTemplate"></property>
   </bean>
   
   <bean id="myTransactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
       <property name="transactionmanager" ref="myTransactionmanager"></property>
   </bean>
   
   <bean id="myTransactionmanager" class="org.springframework.transaction.support.TransactionManager">
       <property name="datasource" ref="mydatasource"></property>
   </bean>
   ```

   ```java
   @Autowired
   TransactionTemplate transactionTemplate;
   
   public void setTransactionTemplate(TransactionTemplate transactionTemplate){
       this.transactionTemplate = transactionTemplate;
   }
   ```

   - 在需要执行的事务业务中，使用 `transactionTemplate` 对象执行 `execute(new TransactionCallbackWithoutResult(){})`方法。在匿名类中写事务代码。

   ```java
   transactionTemplate.execute(new TransactionCallbackWithoutResult(){
       // 执行所需要的事务
   })
   ```

2. 使用配置文件配置事务

```xml
<bean id="myServiceProxyFactoryBean" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="proxyInterface" value="com.cskaoyan.service.UserService"></property>
    <property name="target" ref="myServiceImpl"></property>
    <property name="transactionManager" ref="myTransactionManager"></property>
    <preoerty name="transactionAttribues">
        <props>
            <prop ket="transferMoney">PROPAGATION_REQUIRED,ISOLATION_REPEATABLE_READ</prop>
        </props>
    </preoerty>
</bean>

<bean id="myTransactionManager" class="org.springframework.transaction.support.TransactionManager">
    <property name="datasource" ref="mydatasource"></property>
</bean>
```

3. 使用事务模板实现，通过使用 `spring-tx` 配置，首先使用 `spring-tx` 的schema

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx" 
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx https://www.springframework.org/schema/tx/spring-tx.xsd 
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- bean definitions here -->

</beans>
```

接着配置事务

```xml
<tx:advice id="myadvice" transaction-manager="myTransactionManager">
    <tx:attribues>
        <tx:method name="transferMoney" propagation="REQUIRED" isolation="DEFAULT"/>
    </tx:attribues>
</tx:advice>
```

4. 通过spring tx 注解实现，在需要执行事务而方法中使用注解。（常用方式）

```java
@Transactional(isolation=Isolation.Default, propagation=Propagation.REQUIRED)
```


