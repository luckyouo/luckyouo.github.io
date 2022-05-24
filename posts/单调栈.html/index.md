# 单调栈


## 前言

极值和最值不一样，极值是由区间范围所决定的，最值是由整个区间所决定的，当求极值的一个区间的影响范围时，由于涉及多个区间，如果直接暴力求，则需要 $O(n^2)$ 时间复杂度，而使用单调栈进行求解，时间复杂度为 $O(n)$ 。

{{< figure src="/images/mono_stack.jpg" title=" shiro 工作流程" >}}

## 算法思想

单调栈是指一个栈的值顺序是自增或自减的，比如求值最小值所影响的区间范围是，则使用单调递增的栈。从左至右遍历数组 $arr$，当遍历当数组元素 $a_i$  时，栈中第一个小于数组元素的值即为当前元素 $a_i$ 作为最小值的左区间，若栈空或者栈中无元素小于当前元素，则左区间表示整个区间的左端。同理，也可以从右至左求该元素的最为最小值的右区间。

至此，数组的每个元素作为最值的影响范围都已经求出

## 代码实现

### [1856. 子数组最小乘积的最大值](https://leetcode.cn/problems/maximum-subarray-min-product/)

```markdown
一个数组的 最小乘积 定义为这个数组中 最小值 乘以 数组的 和 。

比方说，数组 [3,2,5] （最小值是 2）的最小乘积为 2 * (3+2+5) = 2 * 10 = 20 。
给你一个正整数数组 nums ，请你返回 nums 任意 非空子数组 的最小乘积 的 最大值 。由于答案可能很大，请你返回答案对  109 + 7 取余 的结果。

请注意，最小乘积的最大值考虑的是取余操作 之前 的结果。题目保证最小乘积的最大值在 不取余 的情况下可以用 64 位有符号整数 保存。

子数组 定义为一个数组的 连续 部分。
```

由于题目需要求最大值，所以区间范围应该为极值的最大影响范围，这与单调栈解决的方向完美契合，而区间和可以通过前缀和解决，因此该题是前缀和 + 单调栈的模板题目。

```java
class Solution {
    public int maxSumMinProduct(int[] nums) {
        final int mod = (int) (1e9 + 7);
        int n = nums.length;
        int[] left = new int[n];
        int[] right = new int[n];

        ArrayDeque<Integer> st = new ArrayDeque<>();
        left[0] = 0;
        right[n-1] = n;
        st.push(0);
        for (int i = 1; i < n; i++) {
            while (!st.isEmpty() && nums[st.peek()] >= nums[i]){
                st.pop();
            }
            left[i] = st.isEmpty() ? 0 : st.peek() + 1;
            st.push(i);
        }

        st = new ArrayDeque<>();
        st.push(n-1);
        for (int i = n - 2; i >= 0; i--) {
            while (!st.isEmpty() && nums[st.peek()] >= nums[i]){
                st.pop();
            }
            right[i] = st.isEmpty() ? n: st.peek();
            st.push(i);
        }

        long[] prefix = new long[n+1];
        for (int i = 0; i < n; i++) {
            prefix[i+1] = prefix[i] + nums[i];
        }

        long max = 0;
        for (int i = 0; i < n; i++) {
            long product = nums[i] * (prefix[right[i]] - prefix[left[i]]) ;
            max = Math.max(max, product);
        }
        return (int) (max % mod);
    }
}
```

单调栈求左右区间范围时，可以通过一次遍历实现同时求左右区间，当一次遍历求左右区间范围时，需要初始化默认左右区间值，上题默认左区间范围为 $0$，默认右区间范围为 $n-1$ 。

```java
class Solution {
    public int maxSumMinProduct(int[] nums) {
        final int mod = (int) (1e9 + 7);
        int n = nums.length;
        int[] left = new int[n];
        int[] right = new int[n];

        ArrayDeque<Integer> st = new ArrayDeque<>();
        left[0] = 0;
        Arrays.fill(right, n-1);
        for (int i = 0; i < n; i++) {
            while (!st.isEmpty() && nums[st.peek()] >= nums[i]) {
                right[st.pop()] = i - 1;
            }
            if (!st.isEmpty()) {
                left[i] = st.peek() + 1;
            }
            st.push(i);
        }

        long[] prefix = new long[n + 1];
        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        long max = 0;
        for (int i = 0; i < n; i++) {
            long product = nums[i] * (prefix[right[i] + 1] - prefix[left[i]]);
            max = Math.max(max, product);
        }
        return (int) (max % mod);
    }
}
```

### [2281. 巫师的总力量和](https://leetcode.cn/problems/sum-of-total-strength-of-wizards/)

```markdown
作为国王的统治者，你有一支巫师军队听你指挥。

给你一个下标从 0 开始的整数数组 strength ，其中 strength[i] 表示第 i 位巫师的力量值。对于连续的一组巫师（也就是这些巫师的力量值是 strength 的 子数组），总力量 定义为以下两个值的 乘积 ：

1. 巫师中 最弱 的能力值。
2. 组中所有巫师的个人力量值 之和 。
请你返回 所有 巫师组的 总 力量之和。由于答案可能很大，请将答案对 109 + 7 取余 后返回。

子数组 是一个数组里 非空 连续子序列。
```

连续的一组巫师代表的就是一个连续区间范围，最弱的值则为极小值，因此该题目求的是极值影响的范围区间的所有包含该极值的子区间和与该极值的乘积的和。区间的范围可以通过单调栈求解，包含极值的子区间可以通过前缀和实现，而所有包含该极值的子区间的和与该极值乘积的和，通过公式求和，确认为前缀和的前缀和。

在范围 [L,R][*L*,*R*] 内的所有子数组的元素和的和可以表示为

$$\sum_{r=i+1}^{R+1}\sum_{l=L}^{i}s[r] - s[l]$$

通过变化求和可得

$$(i - L + 1)\sum_{r=i+1}^{R+1}s[r] - (R- i +1)\sum_{l=L}^is[l]$$

```java
class Solution {
    public int totalStrength(int[] strength) {
        final var mod = (int) 1e9 + 7;

        var n = strength.length;
        var left = new int[n];  // left[i] 为左侧严格小于 strength[i] 的最近元素位置（不存在时为 -1）
        var right = new int[n]; // right[i] 为右侧小于等于 strength[i] 的最近元素位置（不存在时为 n）
        Arrays.fill(right, n);
        var st = new ArrayDeque<Integer>();
        for (var i = 0; i < n; i++) {
            while (!st.isEmpty() && strength[st.peek()] >= strength[i]) right[st.pop()] = i;
            left[i] = st.isEmpty() ? -1 : st.peek();
            st.push(i);
        }

        var s = 0L; // 前缀和
        var ss = new int[n + 2]; // 前缀和的前缀和
        for (var i = 1; i <= n; ++i) {
            s += strength[i - 1];
            ss[i + 1] = (int) ((ss[i] + s) % mod); // 注意取模后，下面计算两个 ss 相减，结果可能为负
        }

        var ans = 0L;
        for (var i = 0; i < n; ++i) {
            int l = left[i] + 1, r = right[i] - 1; // [l,r] 左闭右闭
            var tot = ((long) (i - l + 1) * (ss[r + 2] - ss[i + 1]) - (long) (r - i + 1) * (ss[i + 1] - ss[l])) % mod;
            ans = (ans + strength[i] * tot) % mod; // 累加贡献
        }
        return (int) (ans + mod) % mod; // 防止算出负数
    }
}

```

## 参考资料

[1856. 子数组最小乘积的最大值](https://leetcode.cn/problems/maximum-subarray-min-product/) 

[2281. 巫师的总力量和](https://leetcode.cn/problems/sum-of-total-strength-of-wizards/) 

[单调栈 + 前缀和的前缀和（Python/Java/C++/Go）](https://leetcode.cn/problems/sum-of-total-strength-of-wizards/solution/dan-diao-zhan-qian-zhui-he-de-qian-zhui-d9nki/) 

[【灵茶山艾府】第 294 场力扣周赛讲题 + 算法练习方法](https://leetcode.cn/link/?target=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1RY4y157nW) 


