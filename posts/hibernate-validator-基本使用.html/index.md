# Hibernate Validator 基本使用


## 前言

Spring MVC 框架帮我们做了参数封装，动作解耦，结果视图的转向，json结果的生成等，但参数和合法性验证没有实现，需要自己手动实现。而参数校验属于业务逻辑以外的项目，为了将这些代码解耦出来，可以使用 Hibernate Validator 框架提供的参数校验注解功能。

{{< figure src="/images/hibernate_validator.png" title=" hibernate_validator 验证逻辑" >}}

## 使用流程

### 导入依赖

```xml
<!-- validation -->
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-validator</artifactId>
  <version>5.1.0.Final</version>
</dependency>
<dependency>
  <groupId>javax.validation</groupId>
  <artifactId>validation-api</artifactId>
  <version>1.1.0.Final</version>
</dependency>
```

### 注册

在 application.xml （或者application-mvc.xml）配置文件中添加配置，即向 Spring MVC 容器中注册 validator 对象，并添加验证驱动。

```xml
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
</bean>
<mvc:annotation-driven validator="validator"/>
```

### 数据验证

在向需要验证的 Bean 的属性字段上添加所需要验证方式的注解。

```java
@Size(min = 6, max = 8, message = "size error")
String password;
```

其中 message 为验证错误时给出的错误信息，并在需要验证的传入参数添加 `@Valid` 注解

```java
@Controller
public class UserController {

    @RequestMapping("login")
    public String login(@Valid User user){
        return "/WEB-INF/view/show.jsp";
    }
}
```

## 常用注解

@Null  被注释的元素必须为 null   

@NotNull   被注释的元素必须不为 null   

@Size(max=, min=)  被注释的元素的大小必须在指定的范围内   

@AssertTrue   被注释的元素必须为 true   

@AssertFalse   被注释的元素必须为 false   

@Min(value)   被注释的元素必须是一个数字，其值必须大于等于指定的最小值   

@Max(value)   被注释的元素必须是一个数字，其值必须小于等于指定的最大值   

@DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值   

@DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值   

@Digits (integer, fraction)   被注释的元素必须是一个数字，其值必须在可接受的范围内   

@Past  被注释的元素必须是一个过去的日期   

@Future   被注释的元素必须是一个将来的日期   

@Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式  

**Hibernate Validator 附加的 constraint**   

@NotBlank(message =)  验证字符串非null，且长度必须大于0   

@Email  被注释的元素必须是电子邮箱地址   

@Length(min=,max=)  被注释的字符串的大小必须在指定的范围内   

@NotEmpty  被注释的字符串的必须非空   

@Range(min=,max=,message=)  被注释的元素必须在合适的范围内


