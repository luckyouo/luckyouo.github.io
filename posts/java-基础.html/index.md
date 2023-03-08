# Java 基础


## 前言

主要总结一些 `java` 的基本使用

持续总结中 `ing`

## 内容

### 字符串拼接

字符串的拼接优先使用StringBuilder和StringBuffer类，同时在使用拼接的过程中，尽量使用一个一个的字符串进行 `append` ，而不是将某些字符串拼接后再 `append` ，例如

```java
int n = 100;
StringBuilder sb = new StringBuilder();
for(int i = 0; i < n; i++){
    # sb.append(i + "#");
    sb.append(i);
    sb.append("#");
}
```

不要将 `i + "#"` 拼接好后在 `append` ，字符串拼接后会产生的新的字符串从而产生更多的开销

### Comparator 类的 compare 方法重写

当需要在某数组某区间进行排序并且需要逆序是，则可以重写 `static <T> void sort(T[] a, int fromIndex, int toIndex, Comparator<? super T> c)` 方法，该方法返回类型为泛型，所以不能使用基本类型数组进行接受（数组泛型不会自动解包操作），需要使用包装类完成类型接受。

```java
public static void mySort(Integer[] nums, int idx) {
	Arrays.sort(nums, 0, idx, (a, b) -> Integer.compare(b, a));
}
```

### 数组填充

int double long 数组的默认初始化是 `0`， boolean 数组默认初始化为 `false` ，类的数组初始化为 `null`。 若需要对数组全部进行填充时，可以使用Arrays `api`完成，比如将 int 数组 arr 初始化为 -1。

```java
Arrays.fill(arr, -1)
```

### 集合元素添加

当往集合中添加非基本类型元素时，因为非基本类型元素是引用类型，若引用类型发生改变，则集合元素也会发生改变。所以，没有特别要求时，集合添加元素时，需要额外创建一个引用类型进行添加，以区分之前的引用元素。

```java
set.add(new ArrayList<>(list));
```

当直接添加引用元素时，引用元素修改，集合元素也发生修改

```java
set.add(list);
list.add(12);
// set 中的 list 会增加 12 元素
```

### Arrays.asList 注意事项

1. 将数组转换为 List 时，`Arrays.asList` 生成的 `list` 长度是固定的，内容结构不可变更（不可以使用add、remove，但是可以使用 set），但内部数据可以发生变更，且生成的 list 底层结构仍然是数组，是对数组的引用，若生成的 list 数据发生变更，则数组对应的数组也会变更。
2. 当基本类型转换时，需要使用包装类完成。若使用基本类型时，比如 `int` ，则生成的 `list` 的类型为 `int[]` 。因为 `Arrays.asList` 的实现是通过泛型完成，而基本类型是不可以泛化的，而基本类型的数组是可以泛化的，所以生成的的 `list` 的类型为基本类型数组。若需要使用基本类型的 `list` ，则应该使用基本类型对应的包装类数组完成转换，比如 `Integer[]` 数组。

```java

import java.util.Arrays;
import java.util.List;
 
public class Test {
    public static void main(String[] args){
        int[] nums = {1, 2, 3};
        List<int[]> list1 = Arrays.asList(nums);
        
        Integer[] nums1 = new Integer[]{1,2,3};
 		List<Integer> list1 = Arrays.asList(nums1);
    }
}
```

参考：[Arrays.asList() 详解](https://www.jianshu.com/p/52cdcec633bd) 

### 基本数据类型排序

java 的内部排序默认是快排，数据是从大到小，但当需要从小到大时，需要重写 `Comparator` 接口方法。但如果是基本数据类型数组，因为基本类型不属于类，所有没有办法利用泛型重写 `Comparator` 接口方法。解决方法如下：

1. 自己手写一个排序
2. 利用 java8 的流特性进行反转
3. 内部排序之后再逆转
4. 将 int[] 数组转换为 Integer[] 数组，利用包装类进行排序。（不能直接强制类型转换，只能循环复制，利用自动装箱和拆箱机机制）

参考：[怎么进行java数组逆序排序？](https://segmentfault.com/q/1010000011728359) 

### HashSet 去重

HashSet 是无序不可重复的集合，线程不安全的，底层基于HashMap，基于hashCode()。判断重复机制首先判断添加的元素的 hashCode() 是否相同，再通过 equals() 方法判断元素是否相等。

当使用 HashSet 去重 Integer[] 等数组元素时，如果每次都是通过 new 添加元素，而Integr[] 数组对象没有重写 hashCode 和 equals 方法，故无法达到去重的效果

```java
HashSet<Integer[]> set = new HashSet<>();
set.add(new Integer[]{1, 2});
set.add(new Integer[]{1,2 }); // 无法去重，因为 new 出来的对象 hashcode 不同，而 equals 方法默认是比较两个对象的地址，故也不同。
```

而 HashSet 去重 Collection 元素时，即使通过 new 创建一个对象，但 Collection 都已经重写了 hashCode 和 equals 方法，故可以去重。

```java
HashSet<List<Integer>> set = new HashSet<>();
ArrayList<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
set.add(new ArrayList<>(list));
set.add(new ArrayList<>(list)); // set size 仍为 1
```

### StringBuilder 插入

使用 StringBuilder 底层的数据结构为字符数组，所以当执行频繁的插入操作时，尽量使用 append 方法在尾部插入。

```java
Deque<Character> stack = new LinkedList<>();
while (!stack.isEmpty()){
    sb.append(stack.pollFirst());
    // sb.insert(0, stack.pollLast());
}
```

### final 关键字

final 定义的变量是不可以变的量，指的是变量不能更改对象引用，但是引用的对象内容是可以变更的。比如用 final 修饰 list 时，list的内容仍然可以发生变更，但使用 final 修饰基本类型或者包装类时，则变量对应的就是基本数据，不可以发生变更，否则编译异常。

```java
final Integer i = 1;

i += 1; // 编译不通过

final ArrayList<Integer> list = new ArrayList<>();
list.add(2); // 编译通过
```

### toString

在将对象输出时，会自动调用对象的 toString 方法。如果想利用 toString 方法打印对象地址，不能使用 this 来打印对象地址，而需要使用 `Object.toString()` 方法，即使用 `super.toString()` ，否则会出现循环调用，因为 this 对象也表示该对象，即对象再次调用自己的 toString 方法。

### 数组转 String

Arrays.toString 方法是用来将一维数组转换 String。多为数组使用 Arrays.deepToString。

### 双浮点数除法精确度

当使用两个浮点数进行除法时，可能会出现结果与预期不一致的问题，其原因是使用浮点数无法正确表示某些小数，所以尽量减少使用浮点数做除法。

```java
double v1 = 0.7;
double v2 = 0.025;
double result = v1 / v2;
System.out.println(result);
// 27.999999999999996 而不是预期的 28
```

### Math.ceil 

该方法主要作用是对一个浮点数向上取整。但是如果是对除法结果向上取整时，在 java 中除法的结果已经对结果向下取整了，即结果已经是一个整数，故与预期结果不一致。

```java
int x = 16;
int y = 3;
int ans =(int) Math.ceil(x / y);
System.out.println(ans);
// 5 而不是预期的 6
```

解决办法有三种：

1. 改为浮点数除法，这样结果即为浮点数，在对其向上取整。弊端是存在双浮点数除法存在精度异常问题，需要注意使用

```java
int ans =(int) Math.ceil((double)x / y);
System.out.println(ans);
// 6
```

2. 利用模运算，若取余结果不为 0，则对除法结果进一

```java
int ans = x / y + ((x % y) == 0 ? 0 : 1)
System.out.println(ans);
// 6
```

3. 利用加法的一些技巧

```java
int ans = (x + y - 1) / y; // 该情况不适用于 y < 0

int ans = (x - 1) / y + 1; // 该情况不适用于 a=0 和 b<1
```

[Java rounding up to an int using Math.ceil](https://stackoverflow.com/questions/7139382/java-rounding-up-to-an-int-using-math-ceil) 

### System.arraycopy 和 Arrays.copyOf

二者对数组的拷贝方式都是浅拷贝。调用 `list.toArray(T[])` 方法时，当传递的数组大小小于集合长度时，调用 `Arrays.copyOf` 方法进行拷贝，反之，则调用 `System.arraycopy` 进行拷贝。`Arrays.copyOf` 底层仍然是调用 `System.copyarray` 方法进行拷贝。故集合的 `toArray(T[])` 方法也为浅拷贝。

**注意集合中的 toArray() 方法也是浅拷贝** 

### 内存存储

使用 new 关键字创建的对象都是存储在堆内存中的，而基本类型都是存储在栈上的，分配和清理堆相对于栈要花费更多的时间，但 java 内部已经进行了足够的优化，

### 数组

不管使用什么类型的数组，存储的都是堆上对象的引用，因为数组可以显示的通过 new 进行创建，也可也隐式的通过 new 创建。对象数组和基元数组在使用上是完全相同的。唯一的不同之处就是对象数组存储的是对象的引用，而**基元数组则直接存储基本数据类型的值**。

### 局部变量

java 和 C++ 不一样，不支持同名的局部变量，即没有同名变量

### Arrays.binarySearch

$java$ 内部 $Arrays$ 默认实现的二分查找是根据**递增顺序** 的二分查找，若需要使用逆序或者自定义排序时，则需要传入 $Comparator$ 对象，但这只支持泛型数组，由于泛型数组不能接受基本数据类型数组，所以基本类型数组不能使用该方法，基本类则可以使用。

该二分查找使用的是中间随机二分查找，因此，若数组出现重复元素时，不保证查询顺序。若查询的数据不再数组中时，则返回 $-(1 + index) $ ，其中 index 会插入位置。

之所以返回 $-(1 + index) $ ，而不是 $-index$ ，是为了区别 $0$，即区别查找的元素正好为第一个元素和需要插入在一个元素前，否则二者返回结果都是 $0$。

### int 乘法溢出问题

当两个 int 类型相乘时，如果结果溢出，需要先将 int 作为 long，在进行乘法，否则两个 int 相乘结果已经溢出，再转 long，结果仍然为溢出结果。

```java
n = 100000;
System.out.println(n * (n - 1));
System.out.println((long) n * (n -1));

// 1409965408
// 9999900000
```

### for 循环应尽量遵循外小内大原则

```java
stratTime = System.nanoTime();  
for (int i = 0; i <10000 ; i++) {  
    for (int j = 0; j < 100; j++) {
        for (int k = 0; k < 10; k++) {  
            testFunction(i, j, k);
        }
    }
}
System.out.println("外大内小耗时："+ (endTime - stratTime));  

stratTime = System.nanoTime();  
for (int i = 0; i <10 ; i++) {  
    for (int j = 0; j < 100; j++) {
        for (int k = 0; k < 10000; k++) {  
            testFunction(i, j, k);
        }
    }
}
endTime = System.nanoTime();  
System.out.println("外小内大耗时："+(endTime - stratTime)); 

// 外大内小耗时：1582127649  
// 外小内大耗时：761666633  
```

由于内循环通常会存在创建对象对象的情况，所以外循环越小，创建对象的次数也就越少，也可以将对象创建放置在循环外，这样可以进一步减少时间。

```java
stratTime = System.nanoTime();  
int i, j, k;
for (i = 0; i <10 ; i++) {  
    for (j = 0; j < 100; j++) {
        for (k = 0; k < 10000; k++) {  
            testFunction(i, j, k);
        }
    }
}
endTime = System.nanoTime();  
System.out.println("提取出循环内变量后耗时："+(endTime - stratTime))
// 外小内大耗时：761666633  
// 提取出循环内变量后耗时：748479323
```

### for 循环应该消除判断终止时的方法调用

```java
stratTime = System.nanoTime();  
for (int i = 0; i < list.size(); i++) {  
      
}  
endTime = System.nanoTime();  
System.out.println("未优化list耗时："+(endTime - stratTime));

// 优化后
stratTime = System.nanoTime();  
int size = list.size();  
for (int i = 0; i < size; i++) {  
      
}  
endTime = System.nanoTime();  
System.out.println("优化list耗时："+(endTime - stratTime));
```

[java性能优化笔记 —— for循环](https://segmentfault.com/a/1190000012688298)

### Java 的取模运算

常用的取模运算符号为 %，比如 x % y，但该取模的结果正负与 x 相同，若 x 为负，则结果也为负。如果要与 y 保持相同符号，则应该使用 `Math.floorMod` 方法

### java 超 int

超 int 结果时，会将高位截断，如果需要取模时，需要在强制类型转换前进行取模运算，否则结果运算错误。

```java
long a = Long.MAX_VALUE:
int ans = (int) (a % mod);
```

### ArrayDeque

Java 内部的 ArrayDeque 是通过循环数组实现时，所以作为队列其效率高于 LinkedList。

### HashMap

jdk 1.7 之前是通过冲突链表实现的，这种情况不适合长链表需要查找的的情况。1.8 之后，当冲突的链表元素超过 8 个时，则会转换为红黑树

### HashSet

HashSet 只是简单的对 HashMap 进行了包装，具体实现方法都会调用 HashMap 方法

