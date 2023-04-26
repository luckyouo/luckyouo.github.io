# Sentinel 熔断降级


## 前言

在微服务应用下，如果某个微服务崩溃，调用他的服务若在很长时间才能收到异常消息，则会导致资源浪费，且如果在高并发的情况下，可能出现雪崩的情况。因此，当一个服务出现异常时，应该让其他调用者快速失败或者降级进行其他操作。

## 熔断降级

熔断降级常用组件有 Sentinel 和 Hystrix，其区别如下：

![image-20230426181915198](https://raw.githubusercontent.com/luckyouo/pictures/main/image-20230426181915198.png)

## Sentinel 基本配置使用

1. 导入依赖

```xml
<dependency> 
  <groupId>org.springframework.cloud</groupId> 
  <artifactId>spring-cloud-starter-openfeign</artifactId> 
</dependency>

<dependency> 
  <groupId>com.alibaba.cloud</groupId> 
  <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId> 
</dependency>

<dependency> 
  <groupId>com.alibaba.cloud</groupId> 
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId> 
</dependency>
```

2. 配置文件设置

开始 Feign 远程接口熔断配置

```properties
feign.sentinel.enabled=true
```

3. 配置熔断方法

当出现 Feign 接口熔断时，执行 fallback 方法

```java
@FeignClient(value = "gulimall-seckill",fallback = SeckillFeignServiceFallBack.class)
public interface SeckillFeignService {

    @GetMapping("/sku/seckill/{skuId}")
    R getSkuSeckillInfo(@PathVariable("skuId") Long skuId);
}
```

```java
@Slf4j
@Component
public class SeckillFeignServiceFallBack implements SeckillFeignService {
    @Override
    public R getSkuSeckillInfo(Long skuId) {
        log.info("熔断方法调用...getSkuSeckillInfo");
        return R.error(BizCodeEnume.TOO_MANY_REQUEST.getCode(),BizCodeEnume.TOO_MANY_REQUEST.getMsg());
    }
}
```

## 降级

1. 对特定方法进行限流或降级：在方法的接口上添加 `@SentinelResource` 注解，表示该方法需要进行限流管控

2. 可以在 Sentinel 的 web 页面进行流量管控，超出管控范围，则进行降级。

配置文件配置 web 设置

```properties
spring.cloud.sentinel.transport.dashboard=localhost:8080
```

3. 自定义降级后的方法

配置 WebCallbackManage 的相关设置

```java
@Configuration
public class ProductSentinelConfig {

    public ProductSentinelConfig(){
        WebCallbackManager.setUrlBlockHandler(new UrlBlockHandler(){
            @Override
            public void blocked(HttpServletRequest request, HttpServletResponse response, BlockException ex) throws IOException {
                R error = R.error(BizCodeEnume.TOO_MANY_REQUEST.getCode(), BizCodeEnume.TOO_MANY_REQUEST.getMsg());
                response.setCharacterEncoding("UTF-8");
                response.setContentType("application/json");
                response.getWriter().write(JSON.toJSONString(error));
            }
        });
    }
}
```

## 参考资料

[Java项目《谷粒商城》Java架构师 | 微服务 | 大型电商项目](https://www.bilibili.com/video/BV1np4y1C7Yf?p=1&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)

