# Nacos 注册中心和配置中心


## 前言

在微服务当中，各个微服务需要进行相互调用，一般是通过注册中心来完成。

## 注册中心

常用的注册中心有 Nacos 和 Eureka，前者是 alibaba 中的组件，后者是 Spring 官方中的组件。

### 注册中心配置

下载 nacos 服务端，并启动

客户端配置流程如下：

1. 导入依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2. 配置启动类，启动类添加注解 `@EnableDiscoveryClient`
3. 配置文件设置注册中心地址，设置微服务名字

```yml
spring:  
	cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # 注册中心地址
  application:
    name: app-name
```

## 配置中心

Nacos 除了可以当中注册外，还可以当作配置中心，而 Eureka 则不可以当作配置中心

配置流程如下：

1. 导入依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2. 在启动配置文件 `bootstrap.properties` 中配置相应参数

```properties
spring.cloud.nacos.config.server-addr=127.0.0.1:8848 // 配置中心地址
spring.cloud.nacos.config.namespace=31098de9-fa28-41c9-b0bd-c754ce319ed4  // 配置中心的命名空间id
// 扩展配置文件
spring.cloud.nacos.config.ext-config[0].data-id=gulimall-datasource.yml 
spring.cloud.nacos.config.ext-config[0].refresh=false // 不动态刷新配置
spring.cloud.nacos.config.ext-config[0].group=dev

```

Nacos 作为配置中心时，其 web 界面可以设置 Namespace 或者 Group 来区分不同微服务采用的不同配置 

### 配置动态加载

当使用静态文件作为配置文件时，应用启动后，修改配置文件并不会让服务马上发生变化，只能等待下次服务重新启动才能生效，而使用配置中心时，则可以动态加载配置文件。

如果需要对配置类进行动态刷新时，可以给配置添加 `@RefreshScop` 或 `@ConfigurationProperties` 注解，

## 参考资料

[Java项目《谷粒商城》Java架构师 | 微服务 | 大型电商项目](https://www.bilibili.com/video/BV1np4y1C7Yf?p=1&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)

