# 状态压缩动态规划


## 前言

对集合进行动态规划时，一般采用状态压缩进行。此处集合指的是一个状态的一系列子状态（不包含其本身），并且本身状态还由其子状态所决定。

## 算法基本思想

使用一个整数的二进制来表示一个一个状态，比如 $101$，其中 $1$ 表示处于状态中，而 $0$ 表示没有处于状态中，则该状态的子集有如下情况：$100$ 、 $001$ 。

剩下情况和普通状态规划一样，找到状态转移公式，求最优解。

## 代码

### [5289. 公平分发饼干](https://leetcode.cn/problems/fair-distribution-of-cookies/)

```markdown
给你一个整数数组 cookies ，其中 cookies[i] 表示在第 i 个零食包中的饼干数量。另给你一个整数 k 表示等待分发零食包的孩子数量，所有 零食包都需要分发。在同一个零食包中的所有饼干都必须分发给同一个孩子，不能分开。

分发的 不公平程度 定义为单个孩子在分发过程中能够获得饼干的最大总数。

返回所有分发的最小不公平程度。
```

该题目为典型的 **消耗** 类型问题，即每个饼干都只能属于一个人，所有饼干必须由 k 个人消耗完毕。使用 $dp[k][1 << n]$ 表示状态转移矩阵，其中 $k$ 表示由消耗人数，而 $1<<n$ 表示的所有消耗状态，$n$ 为饼干数量，即 n  个饼干只能处于消耗或未消耗的状态中，因此可以得到如下状态转移方程：

$$dp[i][j] = min(max(dp[i][j/s], sum[s]))$$

其中 $j/s$ 与 $s$ 是集合 $j$ 的互为补集的集合。

求一个集合的任意一个子集可以使用位运算完成，例如 $k = (k - 1) \& j$ ，其中 k 初始化为 $j$ ，一直迭代至 $0$ 时，则可以遍历所以子集。

```java
class Solution {
    public int distributeCookies(int[] cookies, int k) {
        int n = cookies.length;
        int[][] dp = new int[k][1 << n];
        int[] sum = new int[1 << n];
		
        // 初始化 sum，和 dp
        for (int i = 1; i < (1 << n); i++) {
            int idx = Integer.numberOfTrailingZeros(i);
            sum[i] = sum[i - (1 << idx)] + cookies[idx];
            dp[0][i] = sum[i];
        }
		
        for (int i = 1; i < k; i++) {
            for (int j = 0; j < (1 << n); j++) {
                int min = Integer.MAX_VALUE;
                // 遍历所有子集
                for (int l = j; l != 0; l = (l - 1) & j) {
                    // l 为子集时，则 j - l 为对应的补集
                    min = Math.min(min, Math.max(dp[i - 1][j - l], sum[l]));
                }
                dp[i][j] = min;
            }
        }
        return dp[k - 1][(1 << n) - 1];
    }
}
```

## 参考资料

[子集状压 DP](https://leetcode.cn/problems/fair-distribution-of-cookies/solution/by-endlesscheng-80ao/)


