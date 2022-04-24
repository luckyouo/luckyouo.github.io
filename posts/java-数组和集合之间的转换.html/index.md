# Java 数组和集合之间的转换


## 前言

算法题目中，经常出现集合和数组之间、集合和集合之间的转换。

## 内容

### List 和 数组

#### List 转 数组

1. 通过 `toArray(T[] a)` 方法 （推荐）

toArray(T[] a) 方法是 `Collection` 接口中的方法，用来将集合转换为数组，参数为转换后的类型，**不推荐使用 `toArray()` 方法，该方法返回的是 Object 数组，需要强制类型转换**。

```java
    public static void main(String[] args) {
        //1. 通过 toArray()
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add("value" + i);
        }
        String[] arrays = list.toArray(new String[0]);
        System.out.println(Arrays.toString(arrays));
    }
```

2. 通过 `stream` (jdk 1.8 提供)

```java
public static void main(String[] args) {
    //2. jdk1.8 stream
    List<String> list = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        list.add("value" + i);
    }
    String[] arrays = list.stream().toArray(String[]::new);
    System.out.println(Arrays.toString(arrays));
}
```

#### 数组转 List

1. 通过 `Arrays.asList()` 方法

**因为生成的 List 是 Arrays 的内部类，继承了 AbstractList 但没有实现 add() 那些方法，所以生成 List 长度固定，需谨慎使用**

```java
public static void main(String[] args) {
    /*
         * 此种方法生成的List不可进行add和remove操作
         * 因为它是一个定长的List集合，跟数组长度一致
         */
    String[] array = new String[]{"value1", "value2", "value3"};
    List<String> stringList = Arrays.asList(array);
    System.out.println(stringList.toString());
}
```

2. 通过 `Collections.addAll(list, arrays)`

```java
public static void main(String[] args) {
    //2.通过Collections.addAll(list, arrays);
    List<String> stringList=new ArrayList<>();
    String[] array=new String[]{"value1","value2","value3"};
    Collections.addAll(stringList, array);
    System.out.println(stringList.toString());
}
```

3. 通过 `stream` (jdk 1.8 提供)

```java
public static void main(String[] args) {
    //3. jdk1.8 通过Stream
    String[] arrays = new String[]{"value1", "value2", "value3"};
    List<String> listStrings = Stream
        .of(arrays)
        .collect(Collectors.toList());
    System.out.println(listStrings.toString());
}
```

### Set 和 数组

#### Set 转数组

1. 通过 `toArray(T[] a)` 方法

```java
private static void setToArray1() {
    Set<String> set = new HashSet<>();
    set.add("value1");
    set.add("value2");
    set.add("value3");
    //Set-->数组
    String[] array=set.toArray(new String[0]);
    System.out.println(Arrays.toString(array));
}
```

2. 通过 `stream` (jdk 1.8 提供)

```java
private static void setToArray2() {
        Set<String> set = new HashSet<>();
        set.add("value1");
        set.add("value2");
        set.add("value3");
        //Set-->数组
        String[] array= set.stream().toArray(String[]::new);
        System.out.println(Arrays.toString(array));
    }
```

#### 数组转 Set

1. 先转 List，在转 Set

```java
 //数组-->Set
private static void arrayToSet() {
    String[] array = {"value1","value2","value3"};
    Set<String> set = new HashSet<>(Arrays.asList(array));
    System.out.println(set);
}
```

2. 通过 `stream` (jdk 1.8 提供)

```java
private static void arrayToSet2() {
    String[] array = {"value1","value2","value3"};
    Set<String> set = Stream.of(array).collect(Collectors.toSet());
    System.out.println(set);
}
```

### Set 和 List

#### Set 转 List

1. List 构造方法

List 为 null 时，抛出异常

```java
private static void setToList() {
    Set<String> set = new HashSet<>();
    set.add("value1");
    set.add("value2");
    set.add("value3");
    set=null;
    List<String> list=new ArrayList<>(set);
    System.out.println(list.toString());
}
```

2. 通过 `stream` 

```java
private static void setToList2() {
    Set<String> set = new HashSet<>();
    set.add("value1");
    set.add("value2");
    set.add("value3");
    List list=Stream.of(set.toArray(new String[0])).collect(Collectors.toList());
    System.out.println(list.toString());
}
```

#### List 转 Set

1. 通过 Set 构造方法

```java
private static void listToSet() {
    List<String> list = new ArrayList<>();
    list.add("value1");
    list.add("value2");
    list.add("value3");
    Set<String> set=new HashSet<>(list);
    System.out.println(set.toString());
}
```

## 参考资料

[Java中List，Set，数组的互相转换](https://www.jianshu.com/p/717bc27141c4)
