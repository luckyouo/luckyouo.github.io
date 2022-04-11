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

### Comparator类的compare方法重写

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


