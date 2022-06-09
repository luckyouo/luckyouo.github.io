# 贪心算法


## 前言

当最优子结构可以决定全局最优解时，则可以使用贪心算法，在保证每一步都是当前最优子结构，从而得到全局最优解。和动态规划相比，动态规划则将每一步的子结构存储起来，根据先前的子结构来获取当前的结构，即可以根据各个子结构进行回退。

## 算法

### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

```markdown
给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。

子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。
```

该题目是面试常问题目，除了可以使用动态规划在 $O(n^2)$ 的时间复杂度解决，也可以使用贪心算法在 $O(nlogn)$ 的时间复杂度解决

如果我们要使上升子序列尽可能的长，则我们需要让序列上升得尽可能慢，因此我们希望每次在上升子序列最后加上的那个数尽可能的小。

基于上面的贪心思路，我们维护一个数组 d[i] ，表示长度为 i 的最长上升子序列的末尾元素的最小值，用 $len$ 记录目前最长上升子序列的长度，起始时 $len$ 为 1，$d[1] = nums[0]$。

同时我们可以注意到 d[i]d[i] 是关于 ii 单调递增的。因为如果 $d[j] ≥ d[i]$ 且 j < i，我们考虑从长度为 i 的最长上升子序列的末尾删除 $i - j$ 个元素，那么这个序列长度变为 $j$ ，且第 $j$ 个元素 $x$（末尾元素）必然小于 $d[i]$，也就小于 $d[j]$ 。那么我们就找到了一个长度为 $j$ 的最长上升子序列，并且末尾元素比 $d[j]$ 小，从而产生了矛盾。因此数组 $d$ 的单调性得证。

我们依次遍历数组 $nums$ 中的每个元素，并更新数组 $d$ 和 $len$ 的值。如果 $nums[i] > d[len]$ ， 则更新 $len = len + 1$，否则在 $d[1…len]$ 中找满足 $d[i - 1] < nums[j] < d[i]$ 的下标 $i$，并更新 $d[i] =nums[j]$。

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int len = 1, n = nums.length;
        if (n == 0) {
            return 0;
        }
        int[] d = new int[n + 1];
        d[len] = nums[0];
        for (int i = 1; i < n; ++i) {
            if (nums[i] > d[len]) {
                d[++len] = nums[i];
            } else {
                int l = 1, r = len, pos = 0; // 如果找不到说明所有的数都比 nums[i] 大，此时要更新 d[1]，所以这里将 pos 设为 0
                while (l <= r) {
                    int mid = (l + r) >> 1;
                    if (d[mid] < nums[i]) {
                        pos = mid;
                        l = mid + 1;
                    } else {
                        r = mid - 1;
                    }
                }
                d[pos + 1] = nums[i];
            }
        }
        return len;
    }
}
```

**最后的序列并不是实际最长递增序列，当长度等于最长递增序列的长度，因为每个数组元素可能替换之前元素的位置，导致顺序错误，但长度不受影响**

### [673. 最长递增子序列的个数](https://leetcode.cn/problems/number-of-longest-increasing-subsequence/)

```markdown
给定一个未排序的整数数组 nums ， 返回最长递增子序列的个数 。

注意 这个数列必须是 严格 递增的。
```

我们将数组 $d$ 扩展成一个二维数组，其中 $d[i]$ 数组表示所有能成为长度为 $i$ 的最长上升子序列的末尾元素的值。具体地，我们将更新 $d[i]=nums[j]$ 这一操作替换成将 $nums[j]$ 置于 d[i]d[i] 数组末尾。这样 $d[i]d[i]$ 中就保留了历史信息，且 $d[i]$ 中的元素是有序的（单调非增）

类似地，我们也可以定义一个二维数组 $cnt$，其中 $cnt[i][j]$ 记录了以 $d[i][j]$ 为结尾的最长上升子序列的个数。为了计算$cnt[i]$，我们可以考察 $d[i−1]$ 和 $cnt[i−1]$，将所有满足 $d[i-1][k]<d[i][j]$ 的 $cnt[i−1][k]$累加到 $cnt[i][j]$，这样最终答案就是 $cnt[maxLen]$ 的所有元素之和。

由于 $d[i]$ 中的元素是有序的，我们可以二分得到最小的满足 $d[i-1][k]<d[i][j]$ 的下标 $k$。另一处优化是将 $cnt$ 改为其前缀和，并在开头填上 0，此时 $d[i][j]$ 对应的最长上升子序列的个数就是 $cnt[i−1][−1]−cnt[i−1][k]$，这里 $[-1]$ 表示数组的最后一个元素。

```java
class Solution {
    public int findNumberOfLIS(int[] nums) {
        List<List<Integer>> d = new ArrayList<List<Integer>>();
        List<List<Integer>> cnt = new ArrayList<List<Integer>>();
        for (int v : nums) {
            int i = binarySearch1(d.size(), d, v);
            int c = 1;
            if (i > 0) {
                int k = binarySearch2(d.get(i - 1).size(), d.get(i - 1), v);
                c = cnt.get(i - 1).get(cnt.get(i - 1).size() - 1) - cnt.get(i - 1).get(k);
            }
            if (i == d.size()) {
                List<Integer> dList = new ArrayList<Integer>();
                dList.add(v);
                d.add(dList);
                List<Integer> cntList = new ArrayList<Integer>();
                cntList.add(0);
                cntList.add(c);
                cnt.add(cntList);
            } else {
                d.get(i).add(v);
                int cntSize = cnt.get(i).size();
                cnt.get(i).add(cnt.get(i).get(cntSize - 1) + c);
            }
        }

        int size1 = cnt.size(), size2 = cnt.get(size1 - 1).size();
        return cnt.get(size1 - 1).get(size2 - 1);
    }

    public int binarySearch1(int n, List<List<Integer>> d, int target) {
        int l = 0, r = n;
        while (l < r) {
            int mid = (l + r) / 2;
            List<Integer> list = d.get(mid);
            if (list.get(list.size() - 1) >= target) {
                r = mid;
            } else {
                l = mid + 1;
            }
        }
        return l;
    }

    public int binarySearch2(int n, List<Integer> list, int target) {
        int l = 0, r = n;
        while (l < r) {
            int mid = (l + r) / 2;
            if (list.get(mid) < target) {
                r = mid;
            } else {
                l = mid + 1;
            }
        }
        return l;
    }
}
```

贪心 + 树状数组

```java
class Solution {
    int n;
    int[][] tr = new int[2010][2];
    int lowbit(int x) {
        return x & -x;
    }
    int[] query(int x) {
        int len = 0, cnt = 0;
        for (int i = x; i > 0; i -= lowbit(i)) {
            if (len == tr[i][0]) {
                cnt += tr[i][1];
            } else if (len < tr[i][0]) {
                len = tr[i][0];
                cnt = tr[i][1];
            }
        }
        return new int[]{len, cnt};
    }
    void add(int x, int[] info) {
        for (int i = x; i <= n; i += lowbit(i)) {
            int len = tr[i][0], cnt = tr[i][1];
            if (len == info[0]) {
                cnt += info[1];
            } else if (len < info[0]) {
                len = info[0];
                cnt = info[1];
            }
            tr[i][0] = len; tr[i][1] = cnt;
        }
    }
    public int findNumberOfLIS(int[] nums) {
        n = nums.length;
        // 离散化
        int[] tmp = nums.clone();
        Arrays.sort(tmp);
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0, idx = 1; i < n; i++) {
            if (!map.containsKey(tmp[i])) map.put(tmp[i], idx++);
        }
        // 树状数组维护 (len, cnt) 信息
        for (int i = 0; i < n; i++) {
            int x = map.get(nums[i]);
            int[] info = query(x - 1);
            int len = info[0], cnt = info[1];            
            add(x, new int[]{len + 1, Math.max(cnt, 1)});
        }
        int[] ans = query(n);
        return ans[1];
    }
}
```

## 参考资料

[最长上升子序列](https://leetcode.cn/problems/longest-increasing-subsequence/solution/zui-chang-shang-sheng-zi-xu-lie-by-leetcode-soluti/)

​	

