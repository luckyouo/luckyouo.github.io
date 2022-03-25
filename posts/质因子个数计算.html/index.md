# 质因子个数计算


## 前言

[172. 阶乘后的零](https://leetcode-cn.com/problems/factorial-trailing-zeroes/) 该题中计算阶乘结果 `0` 的个数，可转换为计算 `n` 中含有 `5` 质因子的个数问题

## 算法思想

[1,*n*] 中 *p* 的倍数有 $n_1 = \lfloor \frac{n}{p} \rfloor$ ，以此类推可以得到 $n_i = \lfloor \frac{n}{p^i} \rfloor$  可以得到 $p^i$ 质因子的个数。又因为若 `n` 是  $p^i$ 的倍数，则 `n` 必定也是 $n^{i-1}$ 的倍数，故 [1, n]的中的 *p* 的质因子个数为 $\sum \lfloor \frac{n}{p^i} \rfloor$ 

计算结果的 `0` 的个数即统计 `10` 因子的个数，质因素分解为 `2` 和 `5` ，根据上述公式，可以知道 `2` 的 个数多余 `5` 的个数，故只要计算 `5` 的质因子个数。

## 代码

```java
class Solution {
    public int trailingZeroes(int n) {
        int res = 0;
        while(n >= 5){
            res += n / 5;
            n /= 5;
        }

        return res;
    }
}
```



## 参考资料

[阶乘后的零](https://leetcode-cn.com/problems/factorial-trailing-zeroes/solution/jie-cheng-hou-de-ling-by-leetcode-soluti-1egk/)

