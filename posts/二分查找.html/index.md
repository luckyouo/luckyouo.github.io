# 二分查找


## 前言

二分查找一般是用来在有序序列中查找某个范围的数据，通过不断缩小范围来求解，但也可以搭配其他算法来求解其他问题。

## 二分搜索问题

这是学习二分查找时，解决的最基本的问题。求解有序数组小于等于 target 的最大值

```java
public int binarySearch(int[] nums, int left, int right, int taget){
    while(left < right){
        int mid = (left + right) / 2;
        if(nums[mid] >= target){
            right = mid;
        }else{
            left = mid+1;
        }
    }
    return left;
}
```

## 其他问题

### k 值的问题

[668. 乘法表中第k小的数](https://leetcode.cn/problems/kth-smallest-number-in-multiplication-table/) 

题目中，需要求解乘法表中第 k 大的值。由于乘法表的数据范围过大，直接使用数组存储的化，会导致内存溢出，而计算一个数大于等于乘法表的数字的数量则相对简单，因此可以通过二分查找进行搜索。

乘法表中的某一行数据小于等于该数个个数为：$min(\frac{x}{i}, n)$ ，其中 $i$ 表示行数，且第 $i$ 行都是 $i$ 的倍数。

有由于当 $i < \frac{x}{n}$ 时，$\frac{x}{i} > n$ ，故可以得到如下计算公式

$$\lfloor\frac{x}{n}\rfloor*n + \sum_{\frac{x}{n} + 1}^{m}\lfloor\frac{x}{i}\rfloor$$ 

```java
class Solution {
    public int findKthNumber(int m, int n, int k) {
        int left = 1, right = m * n;
        while (left < right) {
            int x = left + (right - left) / 2;
            int count = x / n * n;
            for (int i = x / n + 1; i <= m; ++i) {
                count += x / i;
            }
            if (count >= k) {
                right = x;
            } else {
                left = x + 1;
            }
        }
        return left;
    }
}
```

### 可行性问题

[Cable master](http://poj.org/problem?id=1064) 

```markdown
有 N 条绳子，他们的长度分为 L_i。如果从中切割出 K 条长度相同的绳子的话，则 K 条绳子的每条最长能多长。
```

该问题看似与二分查找无关。但从最后的结果来看，若长度越长，则必然导致 K 条数量不够，因此可以通过二分查找来判断长度 $l$ 是否可以满足获取 K 条长度相同的绳子。

```java
public void solve(int[] lengths, int k){
    int max = lengths[0];
    for(int len: lengths){
        max = Math.max(max, len);
    }
    
    int left = 0;
    int right = max;
    while(left < right){
        int mid = (left + right) / 2;
        int count = 0;
        for(int len : lengths){
            count += len / mid;
        }
        if(count >= k){
            left = mid;
        }else{
            right= mid + 1;
        }
    }
    return left;
}
```

## 总结

以上问题可以使用二分查找来解决问题，通常都是其解都是 ”单向“ 的。这个单向指的是解的结果影响的方向。比如对于 k 值问题，若解增大时，则必然导致小于等于该数的数量非严格递增。而对于可行性问题来说，若解的结果越大，则获取的绳子的数量必然是非严格递减。

## 参考资料

[668. 乘法表中第k小的数](https://leetcode.cn/problems/kth-smallest-number-in-multiplication-table/) 

[Cable master](http://poj.org/problem?id=1064) 

