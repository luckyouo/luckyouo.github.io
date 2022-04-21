# Spring IOC 学习笔记


## 前言

IOC 全称为 `Inverse of control` ，即控制反转。通过向 Spring 的IOC容器中注册，从而使得获得对象的过程进行反转，即不再手动生成所需要的对象，而是IOC容器向应用程序注入某个对象。

## IOC基础

### 对象注册

首先需要导入 `spring-contex` 包，该包包含四个核心和一个依赖，剩余三个核心分别为 `spring-aop` 、`spring-bean` 和 `spring-core`，一个依赖为 `spring-expression`。 同时在配置中引入头文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- bean definitions here -->

</beans>
```

 接着在配置文件中进行对象注册

```xml
<bean name="userService" class="com.cskaoyan.service.impl.UserServiceImpl"/>
```

此时，IOC容器中就包含了所该类的一个的对象。

### 对象设置

在配置文件中设置对象。利用 `property` 标签设置，`name` 表示为类中的 `set` 方法的后缀。如果设置的为对象，则使用 `ref=` 来设置对象，若是值，则使用 `value=` 来设置。

```xml
<bean name="userService" class="com.cskaoyan.service.impl.UserServiceImpl">
    <property name="userDao" ref="UserDao"/>
<bean>
```

默认是通过无参构造方法创建对象（也是主要使用方法），如没有无参构造方法时，需要使用有参构造方法，配置 `<constructor-arg>` 标签来通过调用有参构造方法来创建对象。

```xml
<bean name="userService" class="com.cskaoyan.service.impl.UserServiceImpl">
    <constructor-arg name="id" value="001"/>
   	<constructor-arg name="name" value="java"/>
<bean>
```

也可以通过实例工厂或静态工厂来获得对象（了解即可）

**在IOC容器注册的对象是单例模式，可以通过不同的ID来创建同一个类的不同的对象，也可以更改Bean的作用域（singleton更改为prototype），使得每次获得的对象都是一个新的实例。**

### 注解装配对象

通过配置文件，自动扫描带有注解的类，将类进行注册。配置文件设置如下

```xml
<context:component-scan base-package="com.cskaoyan"></context:component-scan>
```

1. @Component取代<bean class="">，@Component("id") 取代 <bean id="" class="">
2. 在 web 开发中，使用衍生注解取代<bean class="">
   - @Repository ：dao层
   - @Service：service层
   - @Controller：web层

3. 属性注入
   - 普通值：@Value("")
   - 引用值：@Autowired、 @Autowired  @Qualifier("名称") 或 @Resource(name = "名称")

### Test配置

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:application.xml")
```


