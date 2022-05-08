# 重复数据查找


## 前言

查找数组中的重复数据，是非常经典的算法题。故总结一下查找重复数据的各种算法实现。

## 算法

### 题型一

 题目：在一个数组 `nums` 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

1. 方法一

利用位，因为其他数字都出现三次，故对应的位上 $1$ 的个数都是 $3$ 的倍数，而出现一次的数字，则对 $3$ 取余结果为 $1$ 。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int ans = 0;
        for (int i = 0; i < 32; ++i) {
            int total = 0;
            for (int num: nums) {
                total += ((num >> i) & 1);
            }
            if (total % 3 != 0) {
                ans |= (1 << i);
            }
        }
        return ans;
    }
}
```

2. 方法二

方法一过程优化，根据位运算结果进行优化

```java
class Solution {
    public int singleNumber(int[] nums) {
        int a = 0, b = 0;
        for (int num : nums) {
            int aNext = (~a & b & num) | (a & ~b & ~num), bNext = ~a & (b ^ num);
            a = aNext;
            b = bNext;
        }
        return b;
    }
}
```

3. 方法三

方法二再优化

```java
class Solution {
    public int singleNumber(int[] nums) {
        int a = 0, b = 0;
        for (int num : nums) {
            b = ~a & (b ^ num);
            a = ~b & (a ^ num);
        }
        return b;
    }
}
```

### 题型二

给你一个长度为 $n$ 的整数数组 `nums` ，其中 `nums` 的所有整数都在范围 $[1, n]$ 内，且每个整数出现 一次 或 两次 。请你找出所有出现 两次 的整数，并以数组形式返回。

1. 方法一

利用交换，将数据元素交换至索引对应的位置。后遍历数组，若索引对应的元素不是该值，则表面元素出现重复。

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        int n = nums.length;
        for (int i = 0; i < n; ++i) {
            while (nums[i] != nums[nums[i] - 1]) {
                swap(nums, i, nums[i] - 1);
            }
        }
        List<Integer> ans = new ArrayList<Integer>();
        for (int i = 0; i < n; ++i) {
            if (nums[i] - 1 != i) {
                ans.add(nums[i]);
            }
        }
        return ans;
    }
}
```

2. 方法二

利用取反映射，因为数组元素固定范围，将元素值对应的索引位置进行取反，相同的元素对应的索引位置相同，若对应索引数据是正的，则是第一次映射，反之，则彼岸面已经映射过一次，即为重复元素对应的索引位置。

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        int n = nums.length;
        List<Integer> ans = new ArrayList<Integer>();
        for (int i = 0; i < n; ++i) {
            int x = Math.abs(nums[i]);
            if (nums[x - 1] > 0) {
                nums[x - 1] = -nums[x - 1];
            } else {
                ans.add(x);
            }
        }
        return ans;
    }
}
```

## 参考

[剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/) 

[442. 数组中重复的数据](https://leetcode-cn.com/problems/find-all-duplicates-in-an-array/) 

