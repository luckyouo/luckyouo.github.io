# 异步和线程池


## 前言

同步是指当方法一旦被调用就必须等待方法执行完才可以继续向下执行其他操作，而异步不关心方法的执行过程，触发要调用的方法后就继续向下执行其他操作，不会像同步那样必须阻塞至方法结束才继续。

实现异步则需要开启一个新的线程。

## 线程

Java 初始化线程主要有四种方式：

1. 继承 Thread
2. 实现 Runnable 接口
3. 实现 Callable 接口 + FutureTask
4. 线程池

前两种方法无法获取新的线程结果，第三种方法可以获取线程的结果

线程的创建和销毁都需要占用系统资源，所以，在实际应该过程中，应该通过线程池来获取新的线程。

## 线程池

### 线程池的7大参数

1. @param corePoolSize：池中一直保持的线程的数量，即使线程空闲。除非设置了 allowCoreThreadTimeOut
2. @param maximumPoolSize：池中允许的最大的线程数
3. @param keepAliveTime：当线程数大于核心线程数的时候，线程在最大多长时间没有接到新任务就会终止释放， 最终线程池维持在 corePoolSize 大小
4. @param unit：时间单位
5. @param workQueue：阻塞队列，用来存储等待执行的任务，如果当前对线程的需求超过了 corePoolSize 大小，就会放在这里等待空闲线程执行。
6. @param threadFactory：创建线程的工厂，比如指定线程名等
7. @param handler：拒绝策略，如果线程满了，线程池就会使用拒绝策略。

运行流程：

1. 线程池创建，准备好 core 数量的核心线程，准备接受任务
2. 新的任务进来，用 core 准备好的空闲线程执行。
   1. core 满了，就将再进来的任务放入阻塞队列中。空闲的 core 就会自己去阻塞队列获取任务执行
   2. 阻塞队列满了，就直接开新线程执行，最大只能开到 max 指定的数量
   3. max 都执行好了。Max-core 数量空闲的线程会在 keepAliveTime 指定的时间后自动销毁，最终保持到 core 大小
   4. 如果线程数开到了 max 的数量，还有新任务进来，就会使用 reject 指定的拒绝策 略进行处理

所有的线程创建都是由指定的 factory 创建的。

### 内置四大线程池（阿里巴巴规范不建议使用，有 OOM 风险）

- newCachedThreadPool：创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若 无可回收，则新建线程。
- newFixedThreadPool：创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
- newScheduledThreadPool：创建一个定长线程池，支持定时及周期性任务执行
- newSingleThreadExecutor：创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务 按照指定顺序(FIFO, LIFO, 优先级)执行。

上述四个线程池最大线程池数量为 Integer.MAX_VALUE ，所以可能会出现 OOM 情况

根据阿里巴巴的规范，建议使用7大参数自己构建

## 异步

### 异步任务创建

使用 CompletableFuture 工具类进行异步任务编排

该工具类提供了四个静态方法创建来创建异步任务：

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
public static CompletableFuture<Void> runAsync(Runnable runnable);
```

前两个使用自己的线程池创建异步任务，而后者则根据 CPU 的核数创建同步线程池，推荐使用自己的线程池。

run 开头的方法创建的异步任务都没有返回值，而 supply 创建的带有返回结果。

任务完成后可指定回调方法

```java
CompletionStage<T> whenComplete(BiConsumer<? super T,? super Throwable> action);
CompletionStage<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)；
```

使用 whenComplete 开头的方法可以指定某个任务在一个任务结束之后（包括异常结束）。

已 Async 结束的方法表示后续任务已开线新的线程进行，而没有 Async 的则在同一个线程中继续进行，**后续方法类似该规则命名**

可以通过 handle 方法修改线程返回的结果

```java
<U> CompletionStage<U> handle(BiFunction<? super T,Throwable,? extends U> fn)
<U> CompletionStage<U> handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor);
```

### 异步任务编排

1. 顺序编排

```java
CompletionStage<Void> thenRun(Runnable action);
CompletionStage<Void> thenRunAsync(Runnable action);
CompletionStage<Void> thenRunAsync(Runnable action, Executor executor);

CompletionStage<Void> thenAccept(Consumer<? super T> action);
CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor);

<U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);
<U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn);
<U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor);
```

run 开头方法不需要前任务的返回值，accept 开头方法需要前任务的返回值，但后续没有新的返回值，而 apply 开头方法需要前任务的返回值，并提供新的返回值。

2. 组合编排

可以编排各个异步任务之前的前后顺序，在 `CompletionStage ` 类中提高了各式 api 接口，具体可以文档参考

两个常见的组合编排，CompletableFuture 工具类提供如下两个方法

```java
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs);
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs);
```

allOf：等待所有任务完成

anyOf：只要有一个任务完成

## 参考资料

[CompletableFuture避坑1——需要自定义线程池](https://www.jianshu.com/p/8c9dc192fa63)

[Java项目《谷粒商城》Java架构师 | 微服务 | 大型电商项目](https://www.bilibili.com/video/BV1np4y1C7Yf?p=1&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)


