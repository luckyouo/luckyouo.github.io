# Java 常用数据结构


## LinkedList

`LinkedList` 链表，除了 `队列` 接口，还实现了 `双端队列` 

- `void add(int index, E element)` : 在特定位置位置插入元素
- `boolean add(E e)` ： 在链表尾部加入元素
- ` E remove(int index)` ： 删除链表给定序号的元素
- `E removeLast()` : 删除链表尾部元素

## HashMap

`HashMap` 哈希表，可以存 `空值` ，不保证映射后的遍历顺序，也不保证该顺序会随着时间变化而保持，迭代方式

1. 键值迭代

```java
import java.util.HashMap;
import java.util.Map;
 
public class Test {
	public static void main(String[] args) throws IOException {
		Map<Integer, Integer> map = new HashMap<Integer, Integer>();
		map.put(1, 10);
		map.put(2, 20);
 
		// Iterating entries using a For Each loop
		for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
			System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
		}
 
	}
}
```

2. 键迭代

```java

import java.util.HashMap;
import java.util.Map;
 
public class Test {
	public static void main(String[] args) throws IOException {
		Map<Integer, Integer> map = new HashMap<Integer, Integer>();
		map.put(1, 10);
		map.put(2, 20);
 
		// 迭代键
		for (Integer key : map.keySet()) {
			System.out.println("Key = " + key);
		}
 
		// 迭代值
		for (Integer value : map.values()) {
			System.out.println("Value = " + value);
		}
	}
}
```

- `boolean isEmpty()` : 判断哈希表是否为空
- `boolean containsKet(Object Key)` : 查询哈希表是否存在 `Key` ，存在返回 `true`
- `Set<Map.Entry <K, V>> entrySet()` ： 获取 `HashMap` 的迭代器，可以同时获得 `Key` 和 `Value`
- `Set<K> keySet` ：获取哈希表的 `Key` 的迭代器
- `V remove(Object Key)` : 溢出哈希表的 `Key`和对于的 `Value`
- `int size()` : 获取哈希标的实际大小

## LinkedHashMap

`LinkedHashMap` 与 `HashMap` 不同的是，该哈希表可以预测迭代顺序，因为实现了双向链表，该链表定义了迭代顺序，`api` 同上

## TreeSet

`TreeSet` 为有序集合，实现了 `NavigableSet` 接口，默认排序顺序为升序排序，内部结构为平衡二叉树(红黑树)。可以在 `O(h)` 的时间复杂度完成查找、增加、删除操作。由于实现了 `NavigableSet` 接口，可以实现查找小于、大于某元素的结果，如没有找到则返回 `null` 。所以，需要使用包装类接受返回值，而不能使用基本类型接受返回数据。

- ` E ceiling(E e)` : 返回此集合中大于或等于给定元素的最小元素，如果没有这样的元素，则返回 `null` 
- `E floor(E e)` : 返回此集合中小于或等于给定元素的最大元素，如果没有这样的元素，则返回 `null`
- `E greater(E e)` : 返回此集合中严格大于给定元素的最小元素，如果没有这样的元素，则返回 `null`
- `E lower(E e)` : 返回此集合中严格小于给定元素的最小元素，如果没有这样的元素，则返回 `null`
- `boolean contains(Object o)` : 查询元素是否存在有序集合当中

### TreeMap

基于红黑树的 `NavigableMap` 实现。根据其键的自然顺序排序，或者由地图创建时提供的 `Comparator` 排序，具体取决于使用的构造函数，和 `TreeSet` 类似。

- `Map.Entry<K, V> floorEntry(Key key)` : 返回与小于或等于给定键的最大键关联的键值映射，如果没有这样的键，则返回 null
- `K floorKey(Key key)` ：返回小于或等于给定键的最大键，如果没有这样的键，则返回 null
- `Map.Entry<K, V> ceilingEntry (Key key)` ： 返回与大于或等于给定键的最小键关联的键值映射，如果没有这样的键，则返回 null
- `K ceilingKey(Key key)`  ：返回大于或等于给定键的最小键，如果没有这样的键，则返回 null
- `Map.Entry<K, V> lowerEntry(Key key)` ： 返回与严格小于给定键的最大键关联的键值映射，如果没有这样的键，则返回 null
- `K lowerKey(Key key)`  ：返回严格小于给定键的最大键，如果没有这样的键，则返回 null

- `Map.Entry<K, V> higherEntry (Key key)`  ： 返回与严格大于给定键的最小键关联的键值映射，如果没有这样的键，则返回 null
- `K highKey(Key key)` ：返回严格大于给定键的最小键，如果没有这样的键，则返回 null

### PriorityQueue

`PriorityQueue` 默认实现为小顶堆，与`C++` 的优先队列默认为大顶堆不一样。

`E peek()` : 查看堆顶元素

`E poll()` : 删除顶堆元素

