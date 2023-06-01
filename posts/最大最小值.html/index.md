# 最大最小值


## 前言

算法中经常遇到求最小值的最大值或者最大值的最小值，在该问题中，常用方法为二分查找

## 算法思想

当求最大的最小值时，只需要对可能的结果进行结果判断是否满足条件，而可能的结果则可以使用二分法进行操作，因为可能的结果满足单调性。

## 问题

```markdown
给你一个正整数数组 price ，其中 price[i] 表示第 i 类糖果的价格，另给你一个正整数 k 。

商店组合 k 类 不同 糖果打包成礼盒出售。礼盒的 甜蜜度 是礼盒中任意两种糖果 价格 绝对差的最小值。

返回礼盒的 最大 甜蜜度
```

## 代码实现

```java
class Solution {
    public int maximumTastiness(int[] price, int k) {
        Arrays.sort(price);

        // 二分模板·其三（开区间写法）https://www.bilibili.com/video/BV1AP41137w7/
        int left = 0, right = (price[price.length - 1] - price[0]) / (k - 1) + 1;
        while (left + 1 < right) { // 开区间不为空
            // 循环不变量：
            // f(left) >= k
            // f(right) < k
            int mid = left + (right - left) / 2;
          	// 进行数组长度判断
            if (f(price, mid) >= k) left = mid; // 下一轮二分 (mid, right)
            else right = mid; // 下一轮二分 (left, mid)
        }
        return left;
    }
	
  	// 可能的结果所对应的数组最大长度
    private int f(int[] price, int d) {
        int cnt = 1, pre = price[0];
        for (int p : price) {
            if (p >= pre + d) {
                cnt++;
                pre = p;
            }
        }
        return cnt;
    }
}
```

## 参考资料

[【套路】二分答案！附题单！](https://leetcode.cn/problems/maximum-tastiness-of-candy-basket/solutions/2031994/er-fen-da-an-by-endlesscheng-r418/)

