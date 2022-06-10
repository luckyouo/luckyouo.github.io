# 动态规划


## 前言

动态规划与分治方法类似，通过组合子问题解决原问题。动态规划应用与子问题重叠的情况，及不同的子问题具有公共 的子子问题的解。动态规划只能应用于有**最优子结构**的问题。最优子结构的意思是局部最优解能决定全局最优解（对有些问题这个要求并不能完全满足，故有时需要引入一定的近似）。简单地说，问题能够分解成子问题来解决。

适用情况 (摘自 [维基百科 动态规划](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92)) ：

1. 最优子结构性质。如果问题的最优解所包含的子问题的解也是最优的，我们就称该问题具有最优子结构性质（即满足最优化原理）。最优子结构性质为动态规划算法解决问题提供了重要线索。
2. 无后效性。即子问题的解一旦确定，就不再改变，不受在这之后、包含它的更大的问题的求解决策影响。
3. 子问题重叠性质。子问题重叠性质是指在用递归算法自顶向下对问题进行求解时，每次产生的子问题并不总是新问题，有些子问题会被重复计算多次。动态规划算法正是利用了这种子问题的重叠性质，对每一个子问题只计算一次，然后将其计算结果保存在一个表格中，当再次需要计算已经计算过的子问题时，只是在表格中简单地查看一下结果，从而获得较高的效率，降低了时间复杂度。

## 解题思路

动态规划算法主要通过四个步骤来设计

1. 刻画一个最优解的结构特征
2. 递归定义最优解
3. 计算最优解的值（自底向上或带备忘的自顶向下，大部分使用自底向上)
4. 利用计算的信息构造最优解

## 代码实现

### [剑指 Offer II 093. 最长斐波那契数列](https://leetcode-cn.com/problems/Q91FMA/) 

```markdown
如果序列 X_1, X_2, ..., X_n 满足下列条件，就说它是 斐波那契式 的：

n >= 3
对于所有 i + 2 <= n，都有 X_i + X_{i+1} = X_{i+2}
给定一个严格递增的正整数数组形成序列 arr ，找到 arr 中最长的斐波那契式的子序列的长度。如果一个不存在，返回  0 。

（回想一下，子序列是从原序列  arr 中派生出来的，它从 arr 中删掉任意数量的元素（也可以不删），而不改变其余元素的顺序。例如， [3, 5, 8] 是 [3, 4, 5, 6, 7, 8] 的一个子序列）
```

斐波纳契数列长度主要观察最后两位，若后续值可以等于最后两位的和，则表示可以将该数加入对应的数列当中。于是可以使用`dp[i][j]`：表示以arr[i] , arr[j]结尾的斐波那契数列的最大长度。若 arr[j] - arr[i] 存在数列当中，且索引小于 `i` 时，记为 `k` ，则 `dp[i][j]` 需要进行更新，`dp[i][j] = max(dp[i][j], dp[k][i] + 1)` 。

为了降低确认 **差值** 是否存在数列当中且索引低于 `i` ，可以先将数列存入哈希表当中，其中 `Key` 为值， `Value` 为索引。

```java
class Solution {
    public int lenLongestFibSubseq(int[] arr) {
        int n = arr.length;
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < n; i++){
            map.put(arr[i], i);
        }

        int[][] dp = new int[n][n]；    
        int max = 0;
	    for(int i = 0; i < n - 1; i++) {
	    	for(int j = i + 1; j < n; j++) {
	    		int temp = arr[j] - arr[i];
	    		int index = map.getOrDefault(temp, -1);
	    		if(index > -1 && index < i) {
	    			dp[i][j] = dp[index][i] + 1;
	    			max = Math.max(max, dp[i][j] + 2);
	    		}
	    	}
	    }

        return max > 2 ? max : 0;
    }
}
```

注：解该题的重要思路就是确认 `dp` 数组表示的内容，即表示到达 `i`  和 `j` 的斐波纳契数列最大长度。

### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

```markdown
给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。

子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列
```

求严格递增序列，当前位置若大于前面元素，则对应长度 + 1，否则最大长度不变，所以只需要求当前位置大于前面位置元素的最大长度。由此可以得到如下递推式子：

$$dp[i] = max(dp[i], dp[j])$$

其中 $ 0 <= j < i $

所以实现代码如下：

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        int max = 1;
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if(nums[i] > nums[j]){
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            max = Math.max(max, dp[i]);
        }
        return max;
    }
}
```

### 再优化

更新 `dp` 数组时，上述代码采用的是暴力更新。因为数列是已经排序好了的，可以采用双指针进行遍历更新，从而减少时间复杂度，同时也不在需要哈希表了。

```java
class Solution {
    public int lenLongestFibSubseq(int[] arr) {
        int n = arr.length, max = 0;
        int[][] dp = new int[n][n];
        for(int i = 2 ; i < n ; i++){
            int j = 0, k = i-1;
            while(j < k){
                if(arr[j] + arr[k] == arr[i]){
                    if(dp[j][k] == 0){
                        dp[k][i] = 3;
                    }else{
                        dp[k][i] = Math.max(dp[j][k]+1, dp[k][i]);
                    }
                    max = Math.max(max, dp[k][i]);
                    j++;k--;
                }
                else if(arr[j] + arr[k] < arr[i]){
                    j++;
                }else {
                    k--;
                }
            }
        }
        return max;
    }
}
```

### [516. 最长回文子序列](https://leetcode.cn/problems/longest-palindromic-subsequence/)

```markdown
给你一个字符串 s ，找出其中最长的回文子序列，并返回该序列的长度。

子序列定义为：不改变剩余字符顺序的情况下，删除某些字符或者不删除任何字符形成的一个序列。
```

回文序列和回文子串都是动态规划的经典题目。子串指的是连续字符回文，而序列则不需要连续。

**字串除了使用动态规划，还可以使用中心扩展法或者 Manacher 算法。**

使用 $dp[i][j]$ 表示从 $i$ 到 $j$ 的最长回文序列，当 $s[i] = s[j]$ 时，则 $dp[i][j] = dp[i+1][j-1] + 2$ ，否则 $dp[i][j] = max(dp[i+1][j], dp[i][j-1])$ ，根据状态转移公式可知，$i$ 需要从大到下遍历，而 $j$ 需要从小到大遍历，初始 $dp[i][i] = 1$。 

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        int n = s.length();
        int[][] dp = new int[n][n];
        for (int i = n-1; i >= 0; i--) {
            dp[i][i] = 1;
            for (int j = i + 1; j < n; j++) {
                if (s.charAt(i) == s.charAt(j)) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                } else {
                    dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[0][n - 1];
    }
}
```

### [647. 回文子串](https://leetcode.cn/problems/palindromic-substrings/)

```markdown
给你一个字符串 s ，请你统计并返回这个字符串中 回文子串 的数目。

回文字符串 是正着读和倒过来读一样的字符串。

子字符串 是字符串中的由连续字符组成的一个序列。

具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。
```

使用 $dp[j]$ 表示 $j$ 到当前遍历 $i$ 是否为回文序列，若是，则计数 +1。因为 $i$ 和 $j$ 都为从左往右遍历，所以当前遍历的 $dp[j+1]$ 表示上一轮遍历值，即 $j+1$ 到 $i-1$ 是否为回文序列，则当前遍历轮次只需要判断 $s[i] == s[j] $ 和 $dp[j+1]$ 即可以判断 $j$ 至 $i$ 是否为回文序列。

```java
class Solution {
    public int countSubstrings(String s) {
        int n = s.length();
        int[] dp = new int[n];
        int cnt = 0;
        for (int i = 0; i < n; i++) {
            dp[i] = 1;
            cnt++;
            for (int j = 0; j < i; j++) {
                if(s.charAt(i) == s.charAt(j) && dp[j+1] == 1){
                    dp[j] = 1;
                    cnt++;
                }else{
                    dp[j] = 0;
                }
            }
        }
        return cnt;
    }
}
```

## 参考资料

[动态规划](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92)

[状态定义很是重要！](https://leetcode-cn.com/problems/length-of-longest-fibonacci-subsequence/solution/zhuang-tai-ding-yi-hen-shi-zhong-yao-by-christmas_/)

[算法导论](https://book.douban.com/subject/20432061/)

