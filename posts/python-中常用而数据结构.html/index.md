# Python 中常用而数据结构


## 前言

使用 python 刷算法时，记录一些常用的数据结构

## 数据结构

### Array-Like(List/Tuple)

python 内置没有 array 数组，但提供了 list 列表和 tuple 元组，二者底层都是通过数组实现。list 为长度可变数组，而 tuple 为长度固定的数组，一般来说 list 更加常用。列表 **没有** `isempty` 判空方法，通过 `if list:` 语句直接判断即可，若存在元素则返回 True ，否则返回 False。 

列表/数组的内置方法：

- `append(val)` ： 在列表末尾出添加元素
- `clear()` ： 清空列表
- `copy()` ： 数组复制
- `count(val)` ：统计列表中 `val` 出现的次数
- `extend(iterable)` : 将可迭代的对象逐一添加至列表末尾
- `index(val)` : 返回列表中 `val` 第一次出现的索引，不存在则抛出异常
- `insert(index, val)` : 在指定索引 `index` 位置插入`val` 
- `pop(index)` : 移除 `index` 位置的元素，默认索引为 0
- `remove(val)` : 移除列表中 `val` 第一次出现的元素，若不存在，则抛出异常
- `reverse()` : 列表反转，原地反转
- `sort()` : 列表自然排序，原地排序

### LinkedList-Like(Deque)

python 内置的 LinkedList 链表有 `Deque` 双端队列。`Deque` 的底层实现为链表，属于 `collections` 模块下的类，双端队列**没有** `isempty` 判空方法，通过 `if deque:` 语句直接判断即可，若存在元素则返回 True ，否则返回 False。 

双端队列的常用方法：

- `append(val)` : 添加到双端列表的末端
- `appendleft(val)` : 添加到双端队列的头部
- `clear()` ： 清除双端队列
- `copy()` : 拷贝双端队列，浅拷贝
- `count(val)` : 统计双端队列 `val` 元素出现的次数
- `extend(iterable)` : 将可迭代的对象逐一添加至双端队列的末尾
- `extendleft(iterable)` : 将可迭代的对象逐一添加至双端队列的头部
- `index(val)` : 返回双端列表从头部至尾部第一次出现 `val` 的索引，不存在则抛出异常
- `insert(index, val)` : 在索引位置出插入元素
- `pop`() : 删除双端队列的尾部的一个元素，队列为空则抛出异常
- `popleft()` 删除双端队列头部的一个元素，队列为空则抛出异常
- `remove(val)` ： 从头部至尾部删除双端队列 `val` 第一次出现的元素，不存在则抛出异常
- `reverse()` : 原地逆序
- `rotate()` : 右循环 n 步，即删除尾部元素，添加至头部

### Set

python 内置 set 集合，属于无序集合，创建集合可以使用花括号或者 set()，但是**创建空集合不可以使用花括号**，空的花括号表示字典。集合 **没有** `isempty` 判空方法，通过 `if deque:` 语句直接判断即可，若存在元素则返回 True ，否则返回 False。

集合常用方法：

- `copy()` : 集合拷贝，浅拷贝
- `issubset(other)` ：判断集合元素是否都在 other 之中
- `isdisjoint(other)` : 判断集合与 other 是否有交集
- `add(val)` ： 将 `val` 元素添加至集合当中
- `remove(val)` ：删除集合中 `val` 元素，不存在则抛出异常
- `pop()` : 返回集合中的任意一个元素，为集合为空，则抛出异常
- `clear()` : 清空集合

### HashTable(Dict)

python 内置字典为哈希表，为无序哈希表。字典 **没有** `isempty` 判空方法，通过 `if deque:` 语句直接判断即可，若存在元素则返回 True ，否则返回 False。

字典常用方法：

- `del dict[key]` : 删除字典中 `key` 对应的键值对，不存在 `key` 在抛出异常
- `keys()` : 返回字典 key 的新视图
- `item()` ： 返回字典 key-val 的新视图
- `values()`: 返回字典 values 的新视图
- `pop(key[, default])` : 删除字典的中 `key` 的键值对，否则返回 default，没有指定 default 则抛出异常
- `reversed(dict)` : 逆序获取字典的健的迭代器， 这是 `reversed(d.keys())` 的快捷方式。 

