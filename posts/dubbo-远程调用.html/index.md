# Dubbo 远程调用


## 前言

Apache Dubbo 是一款 RPC 服务开发框架，用于解决微服务架构下的服务治理与通信问题

![image-20230427143418660](https://raw.githubusercontent.com/luckyouo/pictures/main/image-20230427143418660.png)

- 服务消费者 (Dubbo Consumer)，发起业务调用或 RPC 通信的 Dubbo 进程
- 服务提供者 (Dubbo Provider)，接收业务调用或 RPC 通信的 Dubbo 进程

## Dubbo 基本配置和使用

配置中心可以使用 nacos、eureka 和 zookeeper 等，测试采用 zookeeper 作为注册中心。

1. 依赖导入

```xml
<!-- dubbo -->
<dependency>
  <groupId>org.apache.dubbo</groupId>
  <artifactId>dubbo-spring-boot-starter</artifactId>
</dependency>
<dependency>
  <groupId>org.apache.dubbo</groupId>
  <artifactId>dubbo-dependencies-zookeeper</artifactId>
  <type>pom</type>
  <exclusions>
    <exclusion>
      <artifactId>slf4j-reload4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>
```

导入 Dubbo 和 Zookeeper 的相关依赖

2. 启动类配置注解 `@EnableDubbo` 

```java
@SpringBootApplication
@EnableDubbo
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

```java
@SpringBootApplication
@EnableDubbo
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

 `@EnableDubbo` 注解表示该服务向注册中心进行注册

3. 配置文件配置 Dubbo 的注册中心

```yaml
dubbo:
  application:
    name: dubbo-springboot-app-name
  protocol:
    name: dubbo
    port: -1
  registry:
    address: zookeeper://${zookeeper.address:127.0.0.1}:2181
```

4. 定义远程接口

```java
public interface DemoService {

    String sayHello(String name);
}
```

消费者和生产者都需要远程接口，且类的绝对路径必须一致，因此应该将远程接口定义在公共模块内

5. 服务生产者提供接口实现

```java
@DubboService
public class DemoServiceImpl implements DemoService {

    @Override
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```

使用 `@DubboService` 注解向注册中心表示对应的远程接口实现。（Dubbo 3.0 之前使用 @Service 注解，因与 Spring 官方 @Service 名字相同，改为 @DubboService，前者被废弃）

6. 服务消费者引用接口服务对象

```java
@Component
public class Task implements CommandLineRunner {
    @DubboReference
    private DemoService demoService;

    @Override
    public void run(String... args) throws Exception {
        String result = demoService.sayHello("world");
        System.out.println("Receive result ======> " + result);

        new Thread(()-> {
            while (true) {
                try {
                    Thread.sleep(1000);
                    System.out.println(new Date() + " Receive result ======> " + demoService.sayHello("world"));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    Thread.currentThread().interrupt();
                }
            }
        }).start();
    }
}
```

使用注解 `@DubboReference` 注入接口对象。（Dubbo 3.0 之前使用 @Reference 注解，因与 Spring 官方 @Reference 名字相同，改为 `@DubboReference`，前者被废弃）

## 参考资料

[Dubbo 文档](https://cn.dubbo.apache.org/zh-cn/overview/home/)

