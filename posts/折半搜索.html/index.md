# 折半搜索


## 前言

折半搜索方法，也称为 `meet in the mid` 。算法的主要过程是将搜索过程分为两个更小的部分，最后在将这两者合并，得到最终结果，时间复杂度为 $2^{n/2} * k $ ，其中 k 表示后续合并的时间复杂度。一般这种算法主要是解决 ` n <= 40` 的算法问题。

## 算法思想

当暴力解决一个问题的所需要的时间复杂度为 `2^n` 时，且 `30<=n <= 40` ，若直接暴力，则一般会超出时间限制。当问题可以分解为两个不同的问题，并且将两个问题合并时，可以得到远题目的答案，则可以使用折半搜索。

## 代码实现

[805. 数组的均值分割](https://leetcode.cn/problems/split-array-with-same-average/description//)

```markdown
给定你一个整数数组 nums

我们要将 nums 数组中的每个元素移动到 A 数组 或者 B 数组中，使得 A 数组和 B 数组不为空，并且 average(A) == average(B) 。

如果可以完成则返回true ， 否则返回 false  。

注意：对于数组 arr ,  average(arr) 是 arr 的所有元素的和除以 arr 长度。
```

一个数字分为 A 或 B 组时，有两种选择，直接回溯暴力搜索时，时间复杂度为 $2 ^n$，会超时。此时将数组对半分，分别计算一半的结果，并将结果存储下来，最后两组结果合并，得到最终的结果，时间复杂度为 $n * 2 ^ {n/2}$

```java
class Solution {
    public boolean splitArraySameAverage(int[] nums) {
        if (nums.length == 1) {
            return false;
        }
        int n = nums.length, m = n / 2;
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        for (int i = 0; i < n; i++) {
            nums[i] = nums[i] * n - sum;
        }

        Set<Integer> left = new HashSet<Integer>();
        for (int i = 1; i < (1 << m); i++) {
            int tot = 0;
            for (int j = 0; j < m; j++) {
                if ((i & (1 << j)) != 0) {
                    tot += nums[j];
                }
            }
            if (tot == 0) {
                return true;
            }
            left.add(tot);
        }
        int rsum = 0;
        for (int i = m; i < n; i++) {
            rsum += nums[i];
        }
        for (int i = 1; i < (1 << (n - m)); i++) {
            int tot = 0;
            for (int j = m; j < n; j++) {
                if ((i & (1 << (j - m))) != 0) {
                    tot += nums[j];
                }
            }
            if (tot == 0 || (rsum != tot && left.contains(-tot))) {
                return true;
            }
        }
        return false;
    }
}
```



## 参考资料

[数组的均值分割](https://leetcode.cn/problems/split-array-with-same-average/solutions/1966163/shu-zu-de-jun-zhi-fen-ge-by-leetcode-sol-f630/)

[【朝夕的ACM笔记】搜索-折半搜索（Meet In The Middle](https://zhuanlan.zhihu.com/p/112563549#:~:text=%E6%8A%98%E5%8D%8A%E6%90%9C%E7%B4%A2%EF%BC%8C%E5%8F%88%E7%A7%B0%E4%B8%BA,%E5%BE%80%E5%BE%80%E6%98%AF%E6%8C%87%E6%95%B0%E7%BA%A7%E5%88%AB%E7%9A%84%E3%80%82)

