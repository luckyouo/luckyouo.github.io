# Java 面试


## 前言

记录一些常见 Java 面试题

## Java 基础篇

1. Java 如何跳出多重循环

```java
ok:
for (int i = 0; i < 10; i++) {
  for (int j = 0; j < 10; j++) {
    System.out.println("i=" + i + ",j=" + j);
    if (j == 5) break ok; // 利用 tag
  }
}

int[][] arr = {{1, 2, 3}, {4, 5, 6, 7}, {9}};
boolean found = false; // 利用二重判断条件，优先选择该方法
for (int i = 0; i < arr.length && !found; i++) {
  for (int j = 0; j < arr[i].length; j++) {
    System.out.println("i = " + i + ", j = " + j);
    if (arr[i][j] == 5) {
      found = true;
      break;
    }
  }
}
```

2. switch语句能否作用在byte上，能否作用在long上，能否作用在String上?

在switch（expr1）中，expr1只能是一个整数表达式或者枚举常量（更大字体），整数表达式可以是int基本类型或Integer包装类型，由于，byte,short,char都可以隐含转换为int，所以，这些类型以及这些类型的包装类型也是可以的。

switch 不支持 long 类型；从 java1.7开始 switch 开始支持 String，这是 Java 的语法糖。

3. short s = 1; s1 = s1 + 1; 有什么错误，s1 += 1 有没有错？

`+=` 在 java 为内置运算符，会进行强制转换，而 s1 = s1 + 1 则会出现强制类型转换异常

4. 使用final关键字修饰一个变量时，是引用不能变，还是引用的对象不能变?

使用final关键字修饰一个变量时，是指引用变量不能变，引用变量所指向的对象中的内容还是可以改变的。例如，对于如下语句：

```java
final StringBuffer a=new StringBuffer("immutable");
a = new StringBuffer(""); // 错误
a.append(" broken!"); // 正常
```

5. Math.round(11.5)等于多少? Math.round(-11.5)等于多少?

Math类中提供了三个与取整有关的方法：ceil、floor、round，这些方法的作用与它们的英文名称的含义相对应，例如，ceil 方法表示向上取整，Math.ceil(11.3)的结果为12，Math.ceil(-11.3)的结果是-11；floor 方法表示向下取整，Math.floor(11.6)的结果为11,Math.floor(-11.6)的结果是-12；round方法，它表示“四舍五入”，算法为Math.floor(x+0.5)，即将原来的数字加上0.5后再向下取整，所以，Math.round(11.5)的结果为12，Math.round(-11.5)的结果为-11。

6. 移位操作支持的数据类型有哪些

移位操作符实际上支持的类型只有`int`和`long`，编译器在对`short`、`byte`、`char`类型进行移位前，都会将其转换为`int`类型再操作。由于 `double`，`float` 在二进制中的表现比较特殊，因此不能来进行移位操作。

当 int 类型左移/右移位数大于等于 32 位操作时，会先求余（%）后再进行左移/右移操作。也就是说左移/右移 32 位相当于不进行移位操作（32%32=0），左移/右移 42 位相当于左移/右移 10 位（42%32=10）。当 long 类型进行左移/右移操作时，由于 long 对应的二进制是 64 位，因此求余操作的基数也变成了 64。

7. 包装类型的缓存机制了解么

Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `True` or `False`。

![image-20230503181037473](https://raw.githubusercontent.com/luckyouo/pictures/main/image-20230503181037473.png)

8. Java 中有哪些常见的语法糖？

Java 中最常用的语法糖主要有泛型、自动拆装箱、变长参数、枚举、内部类、增强 for 循环、try-with-resources 语法、lambda 表达式等

9. Optional 要解决什么问题

在调用一个方法得到了返回值却不能直接将返回值作为参数去调用别的方法，我们首先要判断这个返回值是否为null，只有在非空的前提下才能将其作为其他方法的参数。Java 8引入了一个新的Optional类：这是一个可以为null的容器对象，如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

## 参考资料

[Java基础入门80问，适合新手，老鸟直接跳过](http://coderleixiaoshuai.gitee.io/java-eight-part/#/docs/java/base/Java基础入门80问)

[基础概念与常识](https://github.com/Snailclimb/JavaGuide/blob/main/docs/java/basis/java-basic-questions-01.md)

[Java 全栈知识点问题汇总（上）](https://www.pdai.tech/md/interview/x-interview.html)


