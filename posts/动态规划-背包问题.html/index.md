# 动态规划 背包问题


## 前言

背包问题是动态规划的经典问题，也是最为常见的二维动态的规划。背包问题最简常见的有 0-1背包问题和完全背包问题。

0-1背包问题：

```markdown
我们有n种物品，物品j的重量为wj，价格为pj。
我们假定所有物品的重量和价格都是非负的。背包所能承受的最大重量为W。
如果限定每种物品只能选择0个或1个，则问题称为0-1背包问题。
```

完全背包问题：

```markdown
和上题类似，但可选择的物品是无限的，不需要考虑物品的数量
```

根据上述两个问题，还存在多重背包等变形问题。

## 思路历程

背包的解法最为简单也是最暴力的方法，就是回溯搜索，但时间复杂度也是二次方的。考虑到所搜的过程中，有大量重复子结构，因此可以使用记忆搜索来减少不必要的重复搜索，最后考虑的状态转移方程，则可以将记忆搜索转换为动态规划。

$$暴力搜索 -> 记忆化搜索 -> 动态规划$$

## 算法思想

### 0-1 背包问题

由于每个物品都存在两个状态，即选择与未选择，因此可以根据物品的不同状态进行转移。记 $dp[i][j]$ 为 前 $i$ 各物品选择时，容量 为 $j$ 是最大的价值，则存在如下状态转移方程。

$$dp[i][j] = max(dp[i-1][j], dp[i-1][j - w[i]] + v[i])$$

其逆推方程如下：

$$dp[i][j] = max(dp[i+1][j], dp[i+1][j - w[i]] + v[i])$$

因此使用两层循环既可以求解，时间复杂度为 $O(mn)$，其中 $n$ 为物品总件数，$m$ 为容量大小

```java
// for(int i = 0; i < n; i++) {
//     for(int j = 0; j <= m; j++) {
//         if(j - weight[i] >= 0){
//             dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j - weight[i]] + value[i]);
//         }else{
//             dp[i][j] = dp[i-1][j];
//         } 
//     }
// }



for(int i = 0; i < n; i++) {
    for(int j = w[i]; j <= m; j++) {
        dp[i][j] = Math.max(dp[i-1][j], dp[i-1][j - weight[i]] + value[i]);
    }
}
```

### 完全背包问题

由于每个物品都有无限个，因此按照 0-1 背包思想可知，

$$dp[i][j] = max(dp[i-1][j], dp[i-1][j - k * w[i]] + k * v[i])$$

其中 $j - k * w[i] >= 0$，需要遍历所有 $k$ ，因此按照上述递推，则需要三重循环。将上述变形如下

$$dp[i][j] = max(dp[i-1][j], dp[i][j - w[i] + v[i])$$

```java
// for(int i = 0; i < n; i++) {
//     for(int j = 0; j <= m; j++) {
//         if(j - weight[i] >= 0){
//             dp[i][j] = Math.max(dp[i-1][j], dp[i][j - weight[i]] + value[i]);
//         }else{
//             dp[i][j] = dp[i-1][j];
//         } 
//     }
// }


for(int i = 0; i < n; i++) {
    for(int j = w[i]; j <= m; j++) {
        dp[i][j] = Math.max(dp[i-1][j], dp[i][j - weight[i]] + value[i]);
    }
}
```

## 其他问题

### 种类问题

在上述两个经典问题中，求解的是价值最大问题，因此与物品顺序选择无关，所以两个循环可以更换顺序，当是求解种类时，则必须考虑顺序，及外层循环为物品，内层循环为背包容量，例如 

[518. 零钱兑换 II](https://leetcode.cn/problems/coin-change-2/)

```markdown
给你一个整数数组 coins 表示不同面额的硬币，另给一个整数 amount 表示总金额。

请你计算并返回可以凑成总金额的硬币组合数。如果任何硬币组合都无法凑出总金额，返回 0 。

假设每一种面额的硬币有无限个。 

题目数据保证结果符合 32 位带符号整数。
```

由于计算的是选择种类，所以必须考虑顺序，不能将循环顺序更换

```java
class Solution {
    public int change(int amount, int[] coins) {
        int n = coins.length;
        int[][] dp = new int[n + 1][amount + 1];

        for (int i = 0; i <= n; i++) {
            dp[i][0] = 1;
        }

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= amount; j++) {
                if (j >= coins[i - 1]) {
                    dp[i][j] = dp[i - 1][j] + dp[i][j - coins[i - 1]];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[n][amount];
    }
}


class Solution {
    public int change(int amount, int[] coins) {
        HashSet<Integer> set = new HashSet<>();
        int n = coins.length;
        int[] dp = new int[amount + 1];
        dp[0] = 1;


        for (int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
        }
        return dp[amount];
    }
}
```

### 背包容量太大问题

如果背包的容量 $m$ 远大于各个 0-1 背包各个物品的重量 $w_i$ 时，则可以取 $max(m, n * w_{max}(i))$ 为最大容量，从而减少时间复杂度和空间复杂度。

## 参考资料

[动态规划：关于01背包问题，你该了解这些！](https://programmercarl.com/%E8%83%8C%E5%8C%85%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%8001%E8%83%8C%E5%8C%85-1.html#_01-%E8%83%8C%E5%8C%85) 

[完全背包](https://programmercarl.com/%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80%E5%AE%8C%E5%85%A8%E8%83%8C%E5%8C%85.html#%E5%AE%8C%E5%85%A8%E8%83%8C%E5%8C%85) 


