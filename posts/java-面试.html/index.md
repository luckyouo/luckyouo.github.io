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



## 参考资料

[Java基础入门80问，适合新手，老鸟直接跳过](http://coderleixiaoshuai.gitee.io/java-eight-part/#/docs/java/base/Java基础入门80问)

