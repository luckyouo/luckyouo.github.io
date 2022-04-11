# Spring AOP学习笔记


## 前言

AOP 全称为 `Aspect Oriented Programming`，即面向切片编程。与 OOP (Object Oriented Programming) 面向对象编程不同的是，AOP将系统看作为多个对象的交互，通过切面，对系统的不同关注点，切分为多个平面，利用 AOP ，可以更好的对过往代码进行复用。

## AOP基础

### 术语

AOP 的基本用语：

1. `Target`：目标类，需要代理的类
2. `JointPoint`：连接点，被代理的对象中需要增强的点，比如方法
3. `PointCut`：切入点，已经被增强的连接点
4. `Advice`：通知，代理对象执行到 连接点所需要执行的操作
5. `Weaver`：植入，把通知应用到代理对象当中的过程
6. `Proxy` ：代理类，动态代理

### 实现方式

AOP 的底层采用代理机制方式实现。AOP 功能实现方式主要有三种

1. 手动实现。使用 `jdk` 或者 `cglib` 动态代理实现

2. 使用 Spring AOP api 实现。该方法需要手动实现 AOP 的接口，并对指定方法进行增加

3. 使用 `aspectj` 实现。常用方式，实现过程如下

   - 导入 `aspectjweaver` 包，在 `application.xml` 中引入 `aop` 的头文件（后续可以使用配置类完成）

   ```java
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!-- bean definitions here -->
   
   </beans>
   ```

   - 配置切入面、切入点和通知

   ```xml
   <aop:config>
   <!--  -->
   <aop:aspect ref="myAspect（切片的id）">
   <aop:pointcut id="mypointcut" expression="execution(public boolean com.cskaoyan.dao.impl.UserServiceImpl.register(String,String))"/>
   <aop:pointcut id="mypointcut2" expression="execution(* com..impl..*(..))"/>
   <aop:before method="before13" pointcut-ref="mypointcut2"/>
   <aop:around method="myAround" pointcut-ref="mypointcut2"/>
   <aop:after method="myAfter" pointcut-ref="mypointcut2"/>
   <aop:after-throwing method="myAfterThrowing" pointcut-ref="mypointcut2" throwing="myThrowable"/>
   <aop:after-returning method="myAfterReturning" pointcut-ref="mypointcut2" returning="myReturnResult"/>
   </aop:aspect>
   <!-- <aop:aspect ref="myAspect2">
               <aop:before method="before13" pointcut-ref="mypointcut"/>
           </aop:aspect> -->
   </aop:config>
   ```

## AspectJ

基于 AOP 框架，Spring2.0 之后支持了基于 AspectJ 的切入点表达式，可以在不改动原代码的的前提下，自定义开发，植入代码。

{{< figure src="/images/AspectJ.png" title="AspectJ 流程图" >}}

语法：`execution（修饰符 返回值 包 类 方法名(参数类型) throws 异常 `，其中修饰符一般省略，修饰符可以使用 `*` 表示任意; 返回值不可以省略，但是可以使用 `*` 表示任意，若是非 `java.lang` 下的类型，需要写全类名; 方法不可以省略，可以使用 `*` 表示任意; 参数可以使用 `(..)` 表示参数任意; throws 异常可以省略。

通知类型：

1. `Before` ： 在切入点之前执行。 参数类型的校验。 前置通知
2. `AtterReturning` ： 周在切入点之后执行。后置通知，可以对结果进行检查，增加log
3. `Around` ：环绕通知 在切入点之前和之后都会执行。增加事务等等。
4. `AfterThrowing` ：  抛出异常的时候执行通知。正常情况下走不到。只有发生异常的情况下才会去通知，比如记录一些日志。
5. `After` ： 在finally语句里。不管切入点是否有异常发生都会执行。

实现方式：

1. 在配置文件 xml 中书写

```xml
<aop:before method="MyBefore" pointcut-ref="myPointCut"/>
    
<aop:after-returning method="MyAfterReturning" pointcut-ref="myPointCut"/>

<aop:around method="MyAround" pointcut-ref="myPointCut"/>

<aop:afterthrowing method="MyAfterThrowing" pointcut-ref="myPointCut"/>

<aop:after method="MyAfter" pointcut-ref="myPointCut"/>
```

2. 使用注解实现，需要现在配置文件中引入自动代理

```xml
<aop:aspectj-autoproxy/>
```

```java
@Before("execution(public java.utils.list.com..*.addUser(..))")

@AfterReturning(value = "execution(public java.utils.list.com..*.addUser(..))"", returning = "res)

@Around("execution(public java.utils.list.com..*.addUser(..))")

@AfterThrowing(value = "execution(public java.utils.list.com..*.addUser(..))", throwing = "exception")

@After("execution(public java.utils.list.com..*.addUser(..))")
```


