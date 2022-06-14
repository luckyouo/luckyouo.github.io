# 位运算常用技巧


## 前言

位运算总共有六种，与、或、取反、异或、左移和右移，在面试和算法题目当中，偶尔会出现考察位运算的题目，故总结一些在算法中常用的位运算的操作技巧。

## 位运算常用技巧

### 幂乘

左移和右移运算主要是用来取代 2 的幂次方，由于计算机内部是使用二进制存储的数据，所以位运算比乘法速度相对更快。

此外，利用位运算可以判断一个整数的某个二进制位是否为 0 或者 1，此时还需要结合与操作。

### 判断奇偶

利用与操作，判断最后移位是否为 1，从而判断奇偶

```java
if(num & 1){
    // 奇数
}else{
    // 偶数
}
```

### 交换两个数

利用异或操作可以不需要中间变量就可以实现交换两个数

```java
// 交换 a， b
a ^= b;
b ^= a; // 此时 b = a
a ^= b; // 此时 a = b
```

### 遍历所有子集

使用状态压缩时，一个二进制表示的是一个集合，可以使用与操作遍历该集合的所有子集（包含本身）

```java
// n 表示集合
for(int i = n; i != 0; i = (i - 1) & n){
    // i 为 n 集合的子集
}
```

### 只出现一次的数字

只有一个数字出现一次，其他所有数字都出现两次。则可以使用异或求出只出现一次的数字。**两个相同的数字异或为0，异或内部满足交换律和结合率**

```java
// nums 数组包含一个只出现一次的数字
int ans = 0;
for(int i = 0; i < nums.length; i++){
    ans ^= nums[i];
}
```

该题目还有变种题目，比如，其他数字都出现了三次。则需要统计各个数字的位上 $1$ 的个数，3 的倍数的位上，则是其他数字，对 $3$ 取 mod 余 $1$ 的位上则为所求的数字。

```java
public int[] singleNumbers(int[] nums) {
    int value[]=new int[2];
    if(nums.length==2)
        return  nums;
    int val=0;//异或求的值
    for(int i=0;i<nums.length;i++)
    {
        val^=nums[i];
    }
    int index=getFirst1(val);
    int num1=0,num2=0;
    for(int i=0;i<nums.length;i++)
    {
        if(((nums[i]>>index)&1)==0)//如果这个数第index为0 和num1异或
            num1^=nums[i];
        else//否则和 num2 异或
            num2^=nums[i];
    }
    value[0]=num1;
    value[1]=num2;
    return  value;
}

private int getFirst1(int val) {
    int index=0;
    while (((val&1)==0&&index<32))
    {
        val>>=1;// val=val/2
        index++;
    }
    return index;
}
```

上述题目还有根据门电路优化改进的空间

### 运算法则

常用的运算法则如下：

1. 交换律、结合率。除了位移运算，内部都满足该规律
2. 分配律：异或运算对与运算满足分配律、与运算对或运算满足分配律、或运算对与运算满足分配律

## 参考资料

[位运算](https://blog.sukiu.top/Knowledge/Bit-Operation/)

[超全的位运算介绍与总结](https://segmentfault.com/a/1190000039101602)

