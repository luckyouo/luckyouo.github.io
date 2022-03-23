# 前缀和


## 前言

在算法中，如果需要求一些连续数组成的集合，可以使用前缀和对结果进行存储，从而避免暴力搜索，利用空间复杂度换取时间复杂度

例如leetcode中的 [剑指 Offer II 008. 和大于等于 target 的最短子数组](https://leetcode-cn.com/problems/2VG8Kg/)

## 算法思想

创建一个前缀数组 `pre[n]` ，其中 `pre[i]` 表示前 `i` 个数组所构成的数据结果，例如对于 [剑指 Offer II 008. 和大于等于 target 的最短子数组](https://leetcode-cn.com/problems/2VG8Kg/) 题目，则可以表示为前 `i` 个元素和。因为是求连续数组，则可以通过 `pre[j] - pre[i]` 表示任意第 `i` 个元素到第 `j` 个元素的的连续数组，即使用前缀和来减少暴力搜索的时间复杂度，这样算法的时间复杂度为`O(N^2)`，空间复杂度为 `O(N)` 

## 再优化

尽管使用了前缀和思想，可以将时间复杂度降至`O(N^2)`，但在 `n`足够大时，仍然会出现超时情况。为此，可以根据题目的不同要求，在求前缀和的过程中，使用其他数据结构将所需要的数据进行保存，比如使用 `HashMap` 。这在后续求解时，可以使用 `O(1)` 的时间复杂度进行求解，例如在 [剑指 Offer II 011. 0 和 1 个数相同的子数组](https://leetcode-cn.com/problems/A1NYOS/) 题目当中，使用 `Key` 将结果作为索引 ，位置信息作为 `Value` 进行保存

## 代码

[剑指 Offer II 008. 和大于等于 target 的最短子数组](https://leetcode-cn.com/problems/2VG8Kg/) 示例参考代码如下

```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        int n = nums.length;
        if (n == 0) {
            return 0;
        }
        int ans = Integer.MAX_VALUE;
        int[] sums = new int[n + 1]; 
        // 为了方便计算，令 size = n + 1 
        // sums[0] = 0 意味着前 0 个元素的前缀和为 0
        // sums[1] = A[0] 前 1 个元素的前缀和为 A[0]
        // 以此类推
        for (int i = 1; i <= n; i++) {
            sums[i] = sums[i - 1] + nums[i - 1];
        }
        for (int i = 1; i <= n; i++) {
            int target = s + sums[i - 1];
            int bound = Arrays.binarySearch(sums, target);
            if (bound < 0) {
                bound = -bound - 1;
            }
            if (bound <= n) {
                ans = Math.min(ans, bound - (i - 1));
            }
        }
        return ans == Integer.MAX_VALUE ? 0 : ans;
    }
}
```



## 参考链接

[剑指 Offer II 008. 和大于等于 target 的最短子数组](https://leetcode-cn.com/problems/2VG8Kg/solution/he-da-yu-deng-yu-target-de-zui-duan-zi-s-ixef/)




