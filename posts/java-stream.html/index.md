# Java Stream


## 前言

`Stream` 是 `java 8` 增加的新特性。流提供了一种可以让我们在更高的概念级别上指定计算任务的数据视图，通过流可以说明我们想要完成什么任务，而不是说明如何去实现它，操作的实现调度由具体实现解决。

```markdown
+--------------------+       +------+   +------+   +---+   +-------+
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
+--------------------+       +------+   +------+   +---+   +-------+
```

## 基本使用

### 流的创建

集合可以通过 `Collection`  接口中的 `stram` 或者 `parallelStram` 方法中创建，前者是串行流，后者是并行流。

```java
list.stream();
set.paralletStream();
```

数组可以通过 `Steam` 类中的静态方法 `Stream.of` 方法创建。**注意，和 Array.asList 方法一样，如果使用基本类型数组创建的流的类型为基本类型数组类型，使用包装类数组转换的流的类型为包装类**

```java
int[] arr = {1, 2};
Stream<int[]> stream1 = Stream.of(arr);

Integer arr2 = {1, 2};
Stream<Integer> stream1 = Stream.of(arr2);
```

### 过滤

使用 `filter` 方法对了流中的元素进行过滤，`filter` 方法的引元是 `Predicate<T>` ，即从 T 到 boolean 的函数。

```java
list.stream().filter(num -> num > 1);
```

### 映射

通过 `map` 方法将原有元素按照映射规则映射到需要的值。即通过函数进行元素转换，可以通过函数引用或者 lambda 表达式完成。

### 部分抽取

#### 抽取前段部分

通过 `limit` 方法抽取前 n 个元素，如果流长度小于 n，则提前终止。

```
list.stream().limit(100)
```

#### 抽取后段部分

通过 `skip` 方法跳过前 n 个元素，获取后段部分流元素

```java
list.stream().skip(100)
```

#### 抽取满足条件的元素

通过 `takeWhile` 方法，获取满足条件的元素

```java
list.stream().takeWhile(s -> "1234567890".contains(s));
```

#### 丢弃满足条件的元素

通过 `dropWhile` 方法，丢弃满足条件的元素

```java
list.stream().dropWhile(s -> "1234567890".contains(s));
```

### 排序

`stream` 提供了 `sorted` 方法进行排序，有默认提供的排序的方法（流中的元素必须是 Comparable），也可以提供一个 `Comparator` 进行排序。排序会产生一个新的流。

```java
list.stream.sorted(Comparator.comparing(String::length).reversed());
```

### 遍历

`stream` 提供了 `forEach` 方法对流的元素进行遍历。

```java
list.steam.forEach(System.out::println)
```

### 简单约简

上述的基本操作只是对流进行转换和操作，最终需要获取想要的答案，则需要进行简单约简，即终结操作，将流转换为非流值。

#### 计数

使用 `count` 方法统计流中的元素个数

```java
list.stream().count();
```

#### 最值获取

`max` 和 `min` 方法它们主要用于int、double、long等基本类型上。

```java
int max = Arrays.stream(nums).max().getAsInt();
int min = Arrays.stream(nums).min().getAsInt();
```

同时也可以使用 `Int(Long/Double)SummaryStatistics` 类中的方法实现更多的功能。

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
```

## 参考资料

[Java 8 Stream](https://www.runoob.com/java/java8-streams.html)

