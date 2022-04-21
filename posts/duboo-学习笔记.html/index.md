# Duboo 学习笔记


## 前言

以往开发项目，都是将项目部署在一个单体的应用当中，但随着业务需求量的增多，内部耦合严重，扩展性差。为了解决这个问题，引入了微服务概念，即将若干个模块作为微服务独立运行。在之前的单体项目是通过内部的 api 实现各个模块资源访问，而微服务因为将模块分开部署，则必须解决各个业务模块之间的通信问题。

微服务主要分为两个部分，服务提供者（Provider）和服务消费者（Consumer），一个可能即是服务提供者也是服服务消费者。

Java 微服务常用两个微服务有 Dubbo 和 Spring Cloud。Dubbo 使用 RPC（Reomote Process Call） 框架，性能更好，通常应用与大型服务上，而 Spring Cloud 是基于Spring  Boot 快捷开发，使用 http 通信，适用于中小型企业当中。

{{< figure src="/images/dubbo.png" title="Dubbo 工作流程图" >}}

## 使用流程

### 导入依赖

1. spring 框架导入基本依赖 [Dubbo 依赖](https://dubbo.apache.org/zh/docs/v2.7/user/dependencies/)

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.5.3</version>
</dependency>
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.6.Final</version>
</dependency>
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.21.0-GA</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.3.RELEASE</version>
</dependency>
```

2. spring boot 导入基本依赖

```xml
<dependency>
   <groupId>com.alibaba.spring.boot</groupId>
   <artifactId>dubbo-spring-boot-starter</artifactId>
   <version>2.0.0</version>
</dependency>
```

### 配置服务提供者

因为是服务提供者，需要定义服务接口，并实现该接口以提供给其他服务消费者

#### 定义服务接口

```java
package org.apache.dubbo.demo;

public interface DemoService {
    String sayHello(String name);
}
```

#### 实现服务接口

```java
package org.apache.dubbo.demo.provider;
 
import org.apache.dubbo.demo.DemoService;
 
public class DemoServiceImpl implements DemoService {
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```

#### 暴露服务

1. Spring 配置文件暴露。

在 spring 的配置文件 application.xml 中注册对象，并配置相关接口属性，服务提供者使用该配置以提供服务。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <!--<dubbo:registry address="multicast://224.5.6.7:1234" /> -->
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service registry="N/A" interface="org.apache.dubbo.demo.DemoService" ref="demoService"/>
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
</beans>
```

 ```java
 import org.springframework.context.support.ClassPathXmlApplicationContext;
  
 public class Provider {
     public static void main(String[] args) throws Exception {
         ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/dubbo-demo-provider.xml"});
         context.start();
         System.in.read(); // 按任意键退出
     }
 }
 ```

2. Spring boot 配置注解

在服务提供者的类上配置 Service 注解，并在 spring boot 的配置文件上配置 dubbo 的设置

```java
import com.alibaba.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;

@Service(interfaceClass = DemoService.class)
@Component
public class DemoServiceImpl implements DemoService {
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```

```properties
spring.application.name=dubbo-spring-boot-starter
spring.dubbo.server=true
spring.dubbo.registry=N/A
```

### 配置服务消费者

#### 定义服务接口

定义与服务提供者相同的服务接口，全类名相同

#### 配置引用远程服务

1. Spring 配置文件配置远程服务

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <!-- <dubbo:registry address="multicast://224.5.6.7:1234" /> -->
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService url="dubbo://localhost:20880" />
</beans>
```

2. Spring boot 注解配置

在消费者类中使用 Reference 注解配置远程地址

```java
import com.alibaba.dubbo.config.annotation.Reference;
import com.cskaoyan.dubbo.provider.api.demoService;
import org.springframework.stereotype.Component;

@Component
public class Consumer {
    @Refrence(url = "dubbo://localhost:20880")
    DemoService demoService;
    
    public void sendMsg(String name){
        System.out.println(demoService.sayHello(name));
    }
}
```

## 服务注册和服务发现

上述过程都是直连提供者，是一种点对点的调用模式，这种模式主要是用于测试环境，而在线上环境中，服务请求更多，因此需要使用服务中心来协调服务。

ZooKeeper 是一个中间件，负责分布式系统提供协调服务

### 安装 ZooKeeper

archlinux 使用 aur 提供的包安装即可，并启动服务

```bash
yay -S zookeeper
sudo systemctl start zookeeper.service
```

### 导入依赖

1. Spring 框架导入依赖

```xml
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>1.0</version>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
    <type>pom</type>
</dependency>
```

2. Spring boot 导入依赖

```xml
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
```

### 配置

##### Spring 整合配置

1. 服务提供者配置

修改服务端的注册地址

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://localhost:2181" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService"/>
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
</beans>
```

2. 服务消费者配置

修改服务消费者注册中心

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://localhost:2181" />
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService"/>
</beans>
```

##### Spring boot 整合配置

1. 服务提供者配置

在服务提供者的 spring boot 配置文件中配置 dubbo 的注册地址

```properties
spring.application.name=dubbo-spring-boot-starter
spring.dubbo.server=true
spring.dubbo.registry=zookeeper://localhost:2181
```

2. 服务消费者配置

在服务消费者的 spring boot 配置文件中配置 dubbo 的注册地址，同时去掉服务消费者类的 url 设置。

```properties
pring.dubbo.registry=zookeeper://localhost:2181
```

## 参考资料

[Dubbo 文档](https://dubbo.apache.org/zh/docs/)




