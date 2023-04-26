# Feign 远程调用


## 前言

远程调用服务常用的协议有 http 和 rpc 协议，其中 openfign 采用的是 http 协议。

Feign 是 netflix 开发的，但不在维护。Spring 官方在 Feign 的基础上开发了 OpenFeign，后续统一称为 Feign

## Feign 的基本配置和使用

1. 导入依赖

```xml
<dependency> 
  <groupId>org.springframework.cloud</groupId> 
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2. 启动类使用注解进行配置 `@EnableFeignClients`
3. 远程调用接口使用注解 `@FeignClient` 配置

```java
@FeignClient("stores") 
public interface OrderFeignService {
  @RequestMapping("/order/***")
  List<Order> getOrder(User user);
}
```

@FeignClient 后续标注所需要请求的服务名字

接口内的方法的要求：

1. 请求方式一样，Get/Post。。。
2. 请求路径一样：绝对路径需要保持一致
3. 请求参数所对应的 json 对象需要一致：参数可以为不同的类，因为数据传输的过程中，会转换为 json 对象，所以只需要保证 json 对象中的 key 值都要一样即可。

## 常见异常

请求头丢失：当远程调用时，Feign 会创建一个新的请求，但是该请求不包含之前的请求头信息，若远程请求需要携带 cookie 或者 token 时，则会鉴权异常，此时需要对 Feign 的请求头进行改造。

1. 单线程

```java
@Configuration
public class FeignConfig {

    @Bean("requestInterceptor")
    public RequestInterceptor requestInterceptor(){
        return new RequestInterceptor(){
            @Override
            public void apply(RequestTemplate template) {
                //1、RequestContextHolder拿到刚进来的这个请求
                ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                if(attributes!=null){
                    HttpServletRequest request = attributes.getRequest(); //老请求
                    if(request != null){
                        //同步请求头数据，Cookie
                        String cookie = request.getHeader("Cookie");
                        //给新请求同步了老请求的cookie
                        template.header("Cookie",cookie);
                    }
                }
            }
        };
    }
}
```

2. 异步远程调用

```java
//1.问题主要出在 RequestContextHolder.getRequestAttributes();上，点进这个方法 看下源码
 	@Nullable
  public static RequestAttributes getRequestAttributes() {
    //它是从requestAttributesHolder这里面取出来的
    RequestAttributes attributes = (RequestAttributes)requestAttributesHolder.get();
    if (attributes == null) {
      attributes = (RequestAttributes)inheritableRequestAttributesHolder.get();
    }

    return attributes;
  }

//2.接着追 我们发现requestAttributesHolder是一个NamedThreadLocal对象
private static final ThreadLocal<RequestAttributes> requestAttributesHolder = new NamedThreadLocal("Request attributes");

//3.我们发现NamedThreadLocal继承自ThreadLocal
//而ThreadLocal是一个线程局部变量，在不同线程之间是独立的所以我们获取不到 原先主线程的请求属性，即给请求头添加cookie失败
  public class NamedThreadLocal<T> extends ThreadLocal<T> {
    private final String name;

    public NamedThreadLocal(String name) {
      Assert.hasText(name, "Name must not be empty");
      this.name = name;
    }

    public String toString() {
      return this.name;
    }
  }

//4.解决方法(这里只提供了一种 变量复制）
  //获取之前的请求
  RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();

  CompletableFuture<Void> getAddressFuture = CompletableFuture.runAsync(() -> {
    //每一个线程都共享之前的请求数据
    RequestContextHolder.setRequestAttributes(requestAttributes);
    //1.远程查询所有的收货地址列表
    List<MemberAddressVo> address = memberFeignService.getAddress(memberRespVo.getId());
    confirmVo.setAddress(address);
  }, executor);
```

## 参考资料

[Java项目《谷粒商城》Java架构师 | 微服务 | 大型电商项目](https://www.bilibili.com/video/BV1np4y1C7Yf?p=1&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)

[Feign远程调用请求头丢失问题](https://juejin.cn/post/7060120244643168263)


