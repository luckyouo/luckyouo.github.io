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

1. 将数组转换为 List 时，`Arrays.asList` 生成的 `list` 长度是固定的，内容不可变更
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


