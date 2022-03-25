# Java 常用数据结构


## LinkedList

`LinkedList` 链表，除了 `队列` 接口，还实现了 `双端队列` 

- `void add(int index, E element)` : 在特定位置位置插入元素
- `boolean add(E e)` ： 在链表尾部加入元素
- ` E remove(int index)` ： 删除链表给定序号的元素
- `E removeLast()` : 删除链表尾部元素

## HashMap

`HashMap` 哈希表，可以存 `空值` ，不保证映射后的遍历顺序，也不保证该顺序会随着时间变化而保持

- `boolean isEmpty()` : 判断哈希表是否为空
- `boolean containsKet(Object Key)` : 查询哈希表是否存在 `Key` ，存在返回 `true`
- `Set<Map.Entry <K, V>> entrySet()` ： 获取 `HashMap` 的迭代器，可以同时获得 `Key` 和 `Value`
- `Set<K> keySet` ：获取哈希表的 `Key` 的迭代器
- `V remove(Object Key)` : 溢出哈希表的 `Key`和对于的 `Value`
- `int size()` : 获取哈希标的实际大小

## LinkedHashMap

`LinkedHashMap` 与 `HashMap` 不同的是，该哈希表可以预测迭代顺序，因为实现了双向链表，该链表定义了迭代顺序，`api` 同上


