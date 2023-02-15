# 最大公约数相关算法


## 前言

最大公约数指的是能够整除多个整数的最大正整数，且多个整数不能都为零。

### 裴蜀定理

裴蜀定理也称为贝祖定理，说明了对任何整数 a, b和它们的最大公约数 d ，关于未知数 x 和 y 的存在如下线性不定方程：

$$ ax + by = d, d = gcd(a, b)$$

重要推论：a，b 互质的充分必要条件是存在整数 x，y 使得 ax + by = 1

### 代码

[1250. 检查「好数组」](https://leetcode.cn/problems/check-if-it-is-a-good-array/description/)

```markdown
给你一个正整数数组 nums，你需要从中任选一些子集，然后将子集中每一个数乘以一个 任意整数，并求出他们的和。

假如该和结果为 1，那么原数组就是一个「好数组」，则返回 True；否则请返回 False。
```

该题目可以转换为求数组的最大公约数，若为 1 时，则必定存在结果为 1。

```java
class Solution {
    public boolean isGoodArray(int[] nums) {
        int divisor = nums[0];
        for (int num : nums) {
            divisor = gcd(divisor, num);
            if (divisor == 1) {
                break;
            }
        }
        return divisor == 1;
    }

    public int gcd(int num1, int num2) {
        while (num2 != 0) {
            int temp = num1;
            num1 = num2;
            num2 = temp % num2;
        }
        return num1;
    }
}
```



### 参考资料

[1250. 检查「好数组」](https://leetcode.cn/problems/check-if-it-is-a-good-array/description/)

[裴蜀定理](https://oi-wiki.org/math/number-theory/bezouts/)

[裴蜀定理-百度百科](https://baike.baidu.com/item/裴蜀定理/5186593)

