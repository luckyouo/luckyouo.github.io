# Eureka 服务注册中心


## 前言

Springcloud 封装了Netflix 公司开发的 Eureka 模块来实现服务注册与发现。

Eureka采用了C-S的架构设计，EurekaServer 作为服务注册功能的服务器，他是服务注册中心

## Eureka 的基本配置和使用

1. 服务端

   因为 Eureka 采用的是 C-S 架构，所以需要区分客户端和服务端，服务端需要导入服务端的依赖并进行配置

   1. 依赖导入

   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-eureka-server</artifactId>
     <version>1.4.6.RELEASE</version>
   </dependency>
   ```

   2. 配置启动类

   ```java
   @SpringBootApplication
   // @EnableEurekaServer 服务端的启动类，可以接受别人注册进来~
   @EnableEurekaServer
   public class EurekaServer_7001 {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer_7001.class,args);
       }
   }
   ```

   3. 配置文件配置

   ```yaml
   server:
     port: 7001
   # Eureka配置
   eureka:
     instance:
       # Eureka服务端的实例名字
       hostname: 127.0.0.1
     client:
       # 表示是否向 Eureka 注册中心注册自己(这个模块本身是服务器,所以不需要)，表示该服务为 Eureka 的服务端
       register-with-eureka: false
       # fetch-registry如果为false,则表示自己为注册中心,客户端的化为 ture
       fetch-registry: false
       # Eureka监控页面~
       service-url:
       	# 如果多个注册中心，使用 ，符号进行隔开，则不包含自己
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ 
   ```

2. 客户端

1. 依赖导入

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

2. 启动类配置

```java
@SpringBootApplication
// @EnableEurekaClient 开启Eureka客户端注解，在服务启动后自动向注册中心注册服务
// @EnableDiscoveryClient 和 @EnableEurekaClient 共同点就是都是能够让注册中心能够发现，扫描到改服务。
// 不同点：@EnableEurekaClient 只适用于Eureka作为注册中心，@EnableDiscoveryClient 可以是其他注册中心。
// @EnableEurekaClient 开启服务发现客户端的注解，可以用来获取一些配置的信息，得到具体的微服务
@EnableEurekaClient
public class DeptProvider_8001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider_8001.class,args);
    }
}
```

3. 配置文件配置注册中心

```yaml
# Eureka配置：配置服务注册中心地址
eureka:
  client:
    service-url:
    	# 如果有多台服务机器，可以配置多台注册中心
      defaultZone: http://localhost:7001/eureka/
  instance:
    instance-id: springcloud-provider-dept-8001 #修改Eureka上的默认描述信息
```

4. 监控信息配置

需要监控微服务的注册状态，需要导入监控信息的依赖

```xml
<!--actuator完善监控信息-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

并在配置文件中配置

```yaml
# info配置
info:
# 项目的名称
app.name: app-name
# 公司的名称
company.name: company-name
```

5. 服务提供者（使用 RestTemplate 实现）

```java
/**
 * @Auther: csp1999
 * @Date: 2020/05/17/22:44
 * @Description:
 */
@RestController
public class DeptConsumerController {
    /**
     * 理解：消费者，不应该有service层~
     * RestTemplate .... 供我们直接调用就可以了！ 注册到Spring中
     * (地址：url, 实体：Map ,Class<T> responseType)
     * <p>
     * 提供多种便捷访问远程http服务的方法，简单的Restful服务模板~
     */
    @Autowired
    private RestTemplate restTemplate;
    /**
     * 服务提供方地址前缀
     * <p>
     * Ribbon:我们这里的地址，应该是一个变量，通过服务名来访问
     */
    private static final String REST_URL_PREFIX = "http://localhost:8001";
    //private static final String REST_URL_PREFIX = "http://SPRINGCLOUD-PROVIDER-DEPT";
    /**
     * 消费方添加部门信息
     * @param dept
     * @return
     */
    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        // postForObject(服务提供方地址(接口),参数实体,返回类型.class)
        return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
    }
    /**
     * 消费方根据id查询部门信息
     * @param id
     * @return
     */
    @RequestMapping("/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
        // getForObject(服务提供方地址(接口),返回类型.class)
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
    }
    /**
     * 消费方查询部门信息列表
     * @return
     */
    @RequestMapping("/consumer/dept/list")
    public List<Dept> list() {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
    }
}
```

该方法直接使用 RestTemplate 来构造请求发送数据，并访问远程服务，使用 RestTemplate 之前需要向容器注入 RestTemplate

```java
@Configuration
public class ConfigBean {
    //@Configuration -- spring  applicationContext.xml
    //配置负载均衡实现RestTemplate
    // IRule
    // RoundRobinRule 轮询
    // RandomRule 随机
    // AvailabilityFilteringRule ： 会先过滤掉，跳闸，访问故障的服务~，对剩下的进行轮询~
    // RetryRule ： 会先按照轮询获取服务~，如果服务获取失败，则会在指定的时间内进行，重试
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

## Zookeeper 和 Eureka 的区别

**Zookeeper保证的是CP**

**Eureka保证的是AP** 

## 负载均衡

当一个服务有多个应用进行注册时，注册中心需要挑选应用来提供服务，此时需要一个策略，Ribbon 则可以提供负载均衡策略。

1. 客户端导入依赖

```xml
<!--Ribbon-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
<!--Eureka: Ribbon需要从Eureka服务中心获取要拿什么-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

2. 启动类配置注解 `@EnableEurekaClient `

```java
//Ribbon 和 Eureka 整合以后，客户端可以直接调用，不用关心IP地址和端口号
@SpringBootApplication
@EnableEurekaClient //开启Eureka 客户端
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class, args);
    }
}
```

3. 配置文件配置

```yaml
# Eureka配置
eureka:
  client:
    register-with-eureka: false # 不向 Eureka注册自己
    service-url: # 从三个注册中心中随机取一个去访问
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

4. 配置负载均衡类

```java
@Configuration
public class ConfigBean {
    //@Configuration -- spring  applicationContext.xml
    @LoadBalanced //Ribbon 配置负载均衡实现RestTemplate
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

后续客户端则负载均衡的访问集群 Eureka

5. 远程调用配置

```java
//Ribbon:我们这里的地址，应该是一个变量，通过服务名来访问
//private static final String REST_URL_PREFIX = "http://localhost:8001";
private static final String REST_URL_PREFIX = "http://SPRINGCLOUD-PROVIDER-DEPT";
```

直接使用远程服务的名字进行调用

6. 使用 Ribbon 实现负载均衡

```java
@Configuration
public class ConfigBean {
    //@Configuration -- spring  applicationContext.xml
    /**
     * IRule:
     * RoundRobinRule 轮询策略
     * RandomRule 随机策略
     * AvailabilityFilteringRule ： 会先过滤掉，跳闸，访问故障的服务~，对剩下的进行轮询~
     * RetryRule ： 会先按照轮询获取服务~，如果服务获取失败，则会在指定的时间内进行，重试
     */
    @Bean
    public IRule myRule() {
        return new RandomRule();//使用随机策略
        //return new RoundRobinRule();//使用轮询策略
        //return new AvailabilityFilteringRule();//使用轮询策略
        //return new RetryRule();//使用轮询策略
    }
}
```

配置负载均衡规则 IRule 

7. 启动类上配置 Ribbon 的规则

```java
//Ribbon 和 Eureka 整合以后，客户端可以直接调用，不用关心IP地址和端口号
@SpringBootApplication
@EnableEurekaClient
//在微服务启动的时候就能加载自定义的Ribbon类(自定义的规则会覆盖原有默认的规则)
@RibbonClient(name = "SPRINGCLOUD-PROVIDER-DEPT",configuration = MyRule.class)//开启负载均衡,并指定自定义的规则
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class, args);
    }
}
```

## 远程接口调用

Feign 提供了面向接口编程的方法，远程调用不再需要通过 RestTemplate 来构造请求实现，Feign 内部帮助我们构造一个请求进行远程调用

**Feign默认集成了Ribbon**：利用**Ribbon**维护了MicroServiceCloud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡，而与**Ribbon**不同的是，通过**Feign**只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用。

1. 依赖导入

```xml
<!--Feign的依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

2. 启动类配置 `@EnableFeignClients` 注解

```java
@SpringBootApplication
@EnableEurekaClient
// feign客户端注解,并指定要扫描的包以及配置接口DeptClientService
@EnableFeignClients(basePackages = {"com.***.feign"})
// 切记不要加这个注解，不然会出现404访问不到
//@ComponentScan("com.haust.springcloud")
public class FeignDeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(FeignDeptConsumer_80.class, args);
    }
}
```

3. Feign 接口

```java
@FeignClient(value = “SPRINGCLOUD-PROVIDER-DEPT”)
public interface DeptClientService {
  @GetMapping("/dept/get/{id}")
  public Dept queryById(@PathVariable("id") Long id);
  @GetMapping("/dept/list")
  public Dept queryAll();
  @GetMapping("/dept/add")
  public Dept addDept(Dept dept);
}
```

其中 `value` 表示远程服务的名称

4. 调用者注入 Service 调用远程服务

```java
@RestController
public class DeptConsumerController {
  @Autowired
  private DeptClientService deptClientService;
  /**
     * 消费方添加部门信息
     * @param dept
     * @return
     */
  @RequestMapping("/consumer/dept/add")
  public boolean add(Dept dept) {
    return deptClientService.addDept(dept);
  }
  /**
     * 消费方根据id查询部门信息
     * @param id
     * @return
     */
  @RequestMapping("/consumer/dept/get/{id}")
  public Dept get(@PathVariable("id") Long id) {
    return deptClientService.queryById(id);
  }
  /**
     * 消费方查询部门信息列表
     * @return
     */
  @RequestMapping("/consumer/dept/list")
  public List<Dept> list() {
    return deptClientService.queryAll();
  }
}
```

## 参考资料

[狂神说SpringCloud学习笔记](https://www.kuangstudy.com/bbs/1374942542566551554)

[【狂神说Java】SpringCloud最新教程IDEA版](https://www.bilibili.com/video/BV1jJ411S7xr/?p=10&spm_id_from=333.788.top_right_bar_window_history.content.click&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)

