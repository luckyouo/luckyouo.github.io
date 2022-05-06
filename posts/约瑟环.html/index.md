# 约瑟环


## 前言

**阿橋问题**（有时也称为**约瑟夫斯置换**），是一个出现在[计算机科学](https://zh.wikipedia.org/wiki/计算机科学)和[数学](https://zh.wikipedia.org/wiki/数学)中的问题。在计算机[编程](https://zh.wikipedia.org/wiki/编程)的算法中，类似问题又称为**约瑟夫环**。

人们站在一个等待被处决的圈子里。 计数从圆圈中的指定点开始，并沿指定方向围绕圆圈进行。 在跳过指定数量的人之后，处刑下一个人。 对剩下的人重复该过程，从下一个人开始，朝同一方向跳过相同数量的人，直到只剩下一个人，并被释放。

## 算法

### 基本内容

用 $f(n, k)$ 表示 $n$ 个人，每 $k$ 个淘汰一人，则可以得到如下递归式

$$f(n, k) = (f(n - 1, k) + k - 1) \mod n + 1$$

当 n = 1 时，显然 1 获胜

当 n = k 时，则在一个人 $k^{`}$ 离开之后，剩下 n - 1 个人，且从 $（k + 1）^{`}$ 之后开始计数。所以可以递推可得$f(n, k) = (f(n - 1, k) + k - 1) \mod n + 1$ 。其中最后 $+1$ 表示获胜的顺序的下一位（编号从 1 开始）

### 代码

[1823. 找出游戏的获胜者](https://leetcode-cn.com/problems/find-the-winner-of-the-circular-game/) 

```java
class Solution {
    public int findTheWinner(int n, int k) {
        int ans = 1;
        int i = 2;
        while(i <= n){
           ans = (ans + k - 1) % i + 1;
           i += 1;
        }
        return ans;
    }
}
```

## 参考

[阿橋问题](https://zh.wikipedia.org/wiki/%E7%BA%A6%E7%91%9F%E5%A4%AB%E6%96%AF%E9%97%AE%E9%A2%98) 

[约瑟夫环——公式法（递推公式](https://blog.csdn.net/u011500062/article/details/72855826) 

