# RabbitMQ 消息队列


## 前言

在微服务中，可以通过消息中间件来给服务之间传递数据，并提升系统的异步能力。

## 消息队列

消息队列主要有如下四种实现方式：

1. 点对点式：消息只有唯一的发送者和接受者，但并不是说只能有一个接收者
2. 发布订阅式：发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个 主题，那么就会在消息到达时同时收到消息
3. JMS（Java Message Service）：基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现
4. AMQP（Advanced Message Queuing Protocol）：高级消息队列协议，也是一个消息代理的规范，兼容JMS，RabbitMQ是AMQP的实现

## RabbitMQ

### 核心概念：

1. Message：消息，消息是不具名的，它由消息头和消息体组成Publisher

2. Publisher：消息的生产者，也是一个向交换器发布消息的客户端应用程序。

3. Exchange：交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列

   四种类型交换机：

   1. direct（默认）：消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器 就将消息发到对应的队列中。
   2. fanout：每个发到 fanout 类型交换器的消息都 会分到所有绑定的队列上去。
   3. topic：topic 交换器通过模式匹配分配消息的 路由键属性，将路由键和某个模式进行 匹配，此时队列需要绑定到一个模式上。
   4. headers：headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了

4. Queue：消息队列，用来保存消息直到发送给消费者。

5. Binding：绑定，用于消息队列和交换器之间的关联。

6. Connection：网络连接，比如一个TCP连接。

7. Channel：信道，多路复用连接中的一条独立的双向数据流通道。

8. Consumer：消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

9. Virtual Host：虚拟主机，表示一批交换器、消息队列和相关对象，RabbitMQ 默认的 vhost 是 / 。

10. Broker 表示消息队列服务器实体

### 确认机制

![image-20230425202753896](https://p.ipic.vip/ces8oa.png)

### 延时队列和死信

可以通过 RabbitMQ 的延时队列和私信来实现定时任务，延时队列设置了超过某个 ttl 时，其中消息被丢弃到某个指定的队列当中，其中的消息被称为死信，而指定的队列则有任务一直在监听并处理其中的消息。

### 基本配置和使用

1. 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

2. 注解配置启动类

```java
@EnableRabbit
```

3. 可以通过 `@Bean` 注解注册 Queue 、Exchange 和 Binding

```java
@Configuration
public class MyMQConfig {
  	// 延时队列
  	@Bean
    public Queue orderDelayQueue() {

        Map<String,Object> arguments = new HashMap<>();
        /**
         * x-dead-letter-exchange: order-event-exchange
         * x-dead-letter-routing-key: order.release.order
         * x-message-ttl: 60000
         */
        arguments.put("x-dead-letter-exchange","order-event-exchange");
        arguments.put("x-dead-letter-routing-key","order.release.order");
        arguments.put("x-message-ttl",60000); // 延迟时间
        //String name, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        Queue queue = new Queue("order.delay.queue", true, false, false,arguments);
        return queue;
    }
		
  	// 死信消费队列
    @Bean
    public Queue orderReleaseOrderQueue() {
        Queue queue = new Queue("order.release.order.queue", true, false, false);
        return queue;
    }
		
  	// 交换机
    @Bean
    public Exchange orderEventExchange() {
        //String name, boolean durable, boolean autoDelete, Map<String, Object> arguments
        return new TopicExchange("order-event-exchange",true,false);
    }
		
  	// 延时队列和交换机的绑定关系
    @Bean
    public Binding orderCreateOrderBingding() {
        //String destination, DestinationType destinationType, String exchange, String routingKey,
        //			Map<String, Object> arguments
        return new Binding("order.delay.queue",
                Binding.DestinationType.QUEUE,
                "order-event-exchange",
                "order.create.order",
                null);

    }
		
  	// 死信消费队列和交换机的绑定关系
    @Bean
    public Binding orderReleaseOrderBingding() {
        return new Binding("order.release.order.queue",
                Binding.DestinationType.QUEUE,
                "order-event-exchange",
                "order.release.order",
                null);
    }
}
```

4. 配置监听方法，监听队列

可以在配置类上配置 `@RabbitListener(queues = "order.release.order.queue")` 注解监听队列，类中方法使用 `@RabbitHandler` 注解表示改方法生效进行监听

```java
@RabbitListener(queues = "order.release.order.queue")
@Service
public class OrderCloseListener {

    @Autowired
    OrderService orderService;

    @RabbitHandler
    public void listener(OrderEntity entity, Channel channel, Message message) throws IOException {
        try {
            orderService.closeOrder(entity);
            //手动调用支付宝收单；
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
        }
    }
}
```

也可以直接在方法上直接使用 `@RabbitListener` 注解进行监听队列

### 手动 ack

第三个确认过程中是默认自动 ack 的，也就是只要监听到该队列，其消息都会被接受，即使中途发生了异常崩溃。所以，应该在程序中手动确认改过程。

首先在配置文件中设置手动 ack

```xml
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

前两个确认配置文件修改方式如下：

```xml
spring.rabbitmq.publisher-returns=true 
spring.rabbitmq.template.mandatory=true
```

在服务完成后，使用 Channel 对象确认 ack

```java
channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
```

## 参考资料

[Java项目《谷粒商城》Java架构师 | 微服务 | 大型电商项目](https://www.bilibili.com/video/BV1np4y1C7Yf?p=1&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)

