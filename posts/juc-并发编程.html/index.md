# JUC 并发编程


## 前言

JUC 是 "java.util.concurrent" 的简称，主要指的就是该包下一些常用的并发工具类，包括  "java.util.concurrent" 、 "java.util.concurrent.atomic" 和  "java.util.concurrent.locks" 。

## 基本内容

### 线程

**Java 无法开启线程** ，start 方法是通过调用 native 方法（ C/C++ 方法）实现的线程。

两种方法创建线程：继承 Thread 类和实现 Runnable 接口。

线程存在6种状态：NEW（新生）、RUNNABLE（运行）、BLOCKED（阻塞）、WAITTING（等待，死等）、TIMED-WAITTING（超时等待）和 TERMINATED（终止）

线程的常用操作：wait/sleep/notify，其中 wait 方法是 Object 类的，而 sleep 是 Thread 类。wait 会释放锁，且必须在同步块中使用，而 sleep 不会释放锁，可以在任何地方使用，wait 和 notify 配对使用。

### Lock

传统上锁方式是使用 Synchronized ，在 jdk 1.5 引入了 Lock 接口。Lock 配合 Condition 接口可以更好的实现同步。

Synchronized  和 Lock 的区别

-  Synchronized 内置的Java关键字， Lock 是一个Java类 

-  Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁 

- Synchronized 会自动释放锁，lock 必须要手动释放锁！如果不释放锁，死锁 

- Synchronized 线程 1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下 去；

- Synchronized 可重入锁，不可以中断的，非公平；Lock ，可重入锁，可以 判断锁，非公平（可以 自己设置）；

- Synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码！

一个 Lock 可以有多个 Condition 来监视，并可以使用多个 Condition 来实现线程之间的同步问题

常用锁：

ReadWriteLock（读写锁，读锁可以多个线程占有，而写锁只能一个线程）

### 原子类

JDK 提供了一些常用的原子类，包括常用的集合、一些辅助类和基本包装类

线程安全的集合： CopyOnWriteArrayList、CopyOnWriteArraySet、ConcurrentHashMap

辅助类：CountDownLatch（向下计数器）、CyclicBarrier（向上计数器）和 Semaphore（信号量）

### 线程池

线程不应该自己手动创建，应该通过线程池来获取线程。这样可以线程复用、控制最大并发数、管理线程。不推荐使用 Executors 来创建线程池，可能会导致 OOM ，Executors 内部是通过 ThreadPoolExecutor 来创建线程池，所以我们应该直接使用 ThreadPoolExecutor  来创建线程池。

创建线程池的常用7大参数：

```java
ThreadPoolExecutor(
int corePoolSize, // 核心线程池大小
int maximumPoolSize, // 最大核心线程池大小 
long keepAliveTime, // 超时了没有人调用就会释放 
TimeUnit unit, // 超时单位 
BlockingQueue<Runnable> workQueue, // 阻塞队列 
ThreadFactory threadFactory, // 线程工厂：创建线程的，一般不用动
RejectedExecutionHandler handle // 拒绝策略) 

/** * 
四种拒绝策略
new ThreadPoolExecutor.AbortPolicy() // 银行满了，还有人进来，不处理这个人的，抛出异 常 * 
new ThreadPoolExecutor.CallerRunsPolicy() // 哪来的去哪里！ 
new ThreadPoolExecutor.DiscardPolicy() //队列满了，丢掉任务，不会抛出异常！ 
new ThreadPoolExecutor.DiscardOldestPolicy() //队列满了，尝试去和最早的竞争，也不会 抛出异常！ */
```

### ForkJoin

ForkJoin 在 JDK 1.7 ， 并行执行任务！提高效率。大数据量！

### Volatile

Volatile 是 Java 虚拟机提供轻量级的同步机制

1、保证可见性 2、不保证原子性 3、禁止指令重排

JMM 是 Java Memory Model，即 java 的内存模型。

![image-20230320213610720](https://p.ipic.vip/9u0a0l.png)

Volatile 在 JMM 的基础，可以保证可见性和禁止指令重排，但不能保证原子性。

### CAS

CAS 是 Compare and Swap 的缩写，是计算机内部常用的原子指令，java 的 unsafe 可以调用 native 的来进行 cas 操作。

缺点：

1、 循环会耗时

2、一次性只能保证一个共享变量的原子性 

3、ABA问题

可以通过原子引用（AtomicStampedReference）来解决 ABA 问题，其思想采用的是乐观锁方式来实现

### 其他

BlockingQueue：阻塞队列

SynchronousQueue： 同步队列

## 参考资料

[【狂神说Java】JUC并发编程最新版通俗易懂](https://www.bilibili.com/video/BV1B7411L7tE/?spm_id_from=333.337.search-card.all.click&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)


