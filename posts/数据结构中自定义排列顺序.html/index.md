# 数据结构中自定义排列顺序


## 前言

做算法题时，经常遇到需要对某些数据结构的 `compare` 方法重写，比如 `PriorityQueue`，但 `Java` 默认实现的优先队列为小顶堆，即最小的优先选择。但有的时候需要大顶堆或者其他数据类型比较，比如数组，类等，所以，必须对 `Comparator` 的 `compare` 方法进行重写

## 基本步骤

以下都以 `PriorityQueue` 优先队列实现作为参考，其他数据结构类似

### 基本数据类型

当对基本数据类型重写时，只需要对方法简单重写即可。重写方法有创建 `Comparator` 的子类、匿名类或者 `Lambda` 表达式。以下均为改写为升序优先队列，即大顶堆实现方式。

1. 子类

继承 `Comparator` 子类并重写父类的方法，是最基本的实现方式 

```java
class Main{
    public static void main(String[] args){
        PriorityQueue<Integer> numbers = new PriorityQueue<>(new myComparator());
    }
}

class MyComparator implements Comparator<Integer> {
    @Override
    public int compare(Integer number1, Integer number2) {
        return number2 - number1;
    }
}
```

2. 匿名类

为了减少创建不必要的类，可以直接使用匿名类完成

```java
class Main{
    public static void main(String[] args){
        PriorityQueue<Integer> numbers = new PriorityQueue<>(new Comparator(){
            @Override
        	public int compare(Integer number1, Integer number2) {
            	return number2 - number1;
    		}
        });
    }
}
```

3. Lambda 表达式

为了减少使用匿名类的代码，可以直接使用 Lambda 表达式

```java
class Main{
    public static void main(String[] args){
        PriorityQueue<Integer> numbers = new PriorityQueue<>((a, b) -> b - a);
    }
}
```

### 数组

数组重写方法基本与基本数据类似，只以匿名类写法作为参考

```java
class Main{
    public static void main(String[] args){
        PriorityQueue<int[]> numbers = new PriorityQueue<>(new Comparator<int[]>(){
            @Override
            public int compare(int[] o1, int[] o2){
                return o1[1] - o2[1];
            }
        });
    }
}
```

该优先队列存储数据结构类型为 `int` 数组，且以下数组的索引 `1` 为对应数据降序排序

## 参考资料

[Java PriorityQueue](https://www.cainiaojc.com/java/java-priorityqueue.html)

[1606. 找到处理最多请求的服务器](https://leetcode-cn.com/problems/find-servers-that-handled-most-number-of-requests/)


