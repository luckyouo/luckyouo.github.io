# 差分


## 前言

差分是前缀和的相对操作，前缀和主要是用在不改变数组的情况下，相反，差分主要用在需要多次改变数组的情况下/

## 算法基本思想

计算数组两两元素之间的差，构成差分数组，其中第一个元素取数组第一个元素的值。

$$b_i = a_i - a_{i-1}$$

且后续可以通过差分数组的前缀和恢复数组

 $$a_i = \sum_1^i b_i$$

当需要对某个区间的元素统一进行操作时，比如对区间 $[c, d)$的所有元素增加 k，则只需要进行如下操作：

 $$b_c = b_c + k$$

$$b_d = b_d - k$$

即可以完成对区间内所有元素操作。将 $O(k)$ 的操作的时间复杂度降低至 $O(1)$ 。若存在频繁的更改区间操作时，则可以大幅降低时间复杂度。

可以将差分数组转换为前缀和数组

$$sum = \sum_i^na_i = \sum_{i=1}^n\sum_{j=1}^ib_j=\sum_{i=1}^n(n-i+1)*b_i$$ 

## 代码

### 例题一

[731. 我的日程安排表 II](https://leetcode.cn/problems/my-calendar-ii/) 

每次预定一个时间段，意味着那个时间段的访问次数 $+1$ ，所以判断三重预定，则只需要将该区间段 $+1$ 再统计所有区间是否有次数超过三次的，有则预定失败，删除，否则返回 $true$ 。利用 java 的 $TreeMap$ 顺序遍历既可以实现上述要求。

```java
class MyCalendarTwo {
    TreeMap<Integer, Integer> delta;

    public MyCalendarTwo() {
        delta = new TreeMap();
    }

    public boolean book(int start, int end) {
        delta.put(start, delta.getOrDefault(start, 0) + 1);
        delta.put(end, delta.getOrDefault(end, 0) - 1);

        int active = 0;
        for (int d: delta.values()) {
            active += d;
            if (active >= 3) {
                delta.put(start, delta.get(start) - 1);
                delta.put(end, delta.get(end) + 1);
                // if (delta.get(start) == 0)
                //     delta.remove(start);
                return false;
            }
        }
        return true;
    }
}
```

### 例题二

[732. 我的日程安排表 III](https://leetcode.cn/problems/my-calendar-iii/)  

和上题类似，由于不需要判断是否成功，所以只需要将所给的区间段 $+1$ ，在顺序统计区间段的最大的访问次数。

```java
class MyCalendarThree {

    TreeMap<Integer, Integer> map;
    int max = 0;

    public MyCalendarThree() {
        map = new TreeMap<>();
    }

    public int book(int start, int end) {
        map.put(start, map.getOrDefault(start, 0) + 1);
        map.put(end, map.getOrDefault(end, 0) - 1);
        
        int count = 0;
        for (Integer value : map.values()) {
            count += value;
            max = Math.max(count , max);
        }
        return max;
    }
}
```

### 上述题目的线段树解法

差分和线段树都是应用于区间更改操作的数据结构，因此也可以使用线段树解决。但题目给的范围过大，直接初始化线段树内存会爆，因此可以采用动态开点的线段树。上述两题的线段树代码如下：

例题一的线段树解法：

```java
class MyCalendarTwo {
    class Node {
        int ls, rs, add, max;
    }
    int N = (int)1e9, M = 120010, cnt = 1;
    Node[] tr = new Node[M];
    void update(int u, int lc, int rc, int l, int r, int v) {
        if (l <= lc && rc <= r) {
            tr[u].add += v;
            tr[u].max += v;
            return ;
        }
        lazyCreate(u);
        pushdown(u);
        int mid = lc + rc >> 1;
        if (l <= mid) update(tr[u].ls, lc, mid, l, r, v);
        if (r > mid) update(tr[u].rs, mid + 1, rc, l, r, v);
        pushup(u);
    }
    int query(int u, int lc, int rc, int l, int r) {
        if (l <= lc && rc <= r) return tr[u].max;
        lazyCreate(u);
        pushdown(u);
        int mid = lc + rc >> 1, ans = 0;
        if (l <= mid) ans = Math.max(ans, query(tr[u].ls, lc, mid, l, r));
        if (r > mid) ans = Math.max(ans, query(tr[u].rs, mid + 1, rc, l, r));
        return ans;
    }
    void lazyCreate(int u) {
        if (tr[u] == null) tr[u] = new Node();
        if (tr[u].ls == 0) {
            tr[u].ls = ++cnt;
            tr[tr[u].ls] = new Node();
        }
        if (tr[u].rs == 0) {
            tr[u].rs = ++cnt;
            tr[tr[u].rs] = new Node();
        }
    }
    void pushup(int u) {
        tr[u].max = Math.max(tr[tr[u].ls].max, tr[tr[u].rs].max);
    }
    void pushdown(int u) {
        tr[tr[u].ls].add += tr[u].add; tr[tr[u].rs].add += tr[u].add;
        tr[tr[u].ls].max += tr[u].add; tr[tr[u].rs].max += tr[u].add;
        tr[u].add = 0;
    }
    public boolean book(int start, int end) {
        if (query(1, 1, N + 1, start + 1, end) >= 2) return false;
        update(1, 1, N + 1, start + 1, end, 1);
        return true;
    }
}
```

例题二的线段树解法：

```java
class MyCalendarThree {
    class Node {
        int ls, rs, add, max;
    }
    int N = (int)1e9, M = 4 * 400 * 20, cnt = 1;
    Node[] tr = new Node[M];
    void update(int u, int lc, int rc, int l, int r, int v) {
        if (l <= lc && rc <= r) {
            tr[u].add += v;
            tr[u].max += v;
            return ;
        }
        lazyCreate(u);
        pushdown(u);
        int mid = lc + rc >> 1;
        if (l <= mid) update(tr[u].ls, lc, mid, l, r, v);
        if (r > mid) update(tr[u].rs, mid + 1, rc, l, r, v);
        pushup(u);
    }
    int query(int u, int lc, int rc, int l, int r) {
        if (l <= lc && rc <= r) return tr[u].max;
        lazyCreate(u);
        pushdown(u);
        int mid = lc + rc >> 1, ans = 0;
        if (l <= mid) ans = Math.max(ans, query(tr[u].ls, lc, mid, l, r));
        if (r > mid) ans = Math.max(ans, query(tr[u].rs, mid + 1, rc, l, r));
        return ans;
    }
    void lazyCreate(int u) {
        if (tr[u] == null) tr[u] = new Node();
        if (tr[u].ls == 0) {
            tr[u].ls = ++cnt;
            tr[tr[u].ls] = new Node();
        }
        if (tr[u].rs == 0) {
            tr[u].rs = ++cnt;
            tr[tr[u].rs] = new Node();
        }
    }
    void pushdown(int u) {
        tr[tr[u].ls].add += tr[u].add; tr[tr[u].rs].add += tr[u].add;
        tr[tr[u].ls].max += tr[u].add; tr[tr[u].rs].max += tr[u].add;
        tr[u].add = 0;
    }
    void pushup(int u) {
        tr[u].max = Math.max(tr[tr[u].ls].max, tr[tr[u].rs].max);
    }
    public int book(int start, int end) {
        update(1, 1, N + 1, start + 1, end, 1);
        return query(1, 1, N + 1, 1, N + 1);
    }
}
```

## 总结

差分的适用情况与线段树、柱状数组情况类似，但代码更加简洁。差分难点主要在和其他算法一起结合使用，而不是单独的使用差分思想。

## 参考资料

[前缀和 & 差分](https://oi-wiki.org/basic/prefix-sum/#_6) 

[浅谈差分和树上差分算法的基本应用](https://www.cnblogs.com/zhwer/p/12800475.html) 

[小而美的算法技巧：差分数组](https://labuladong.gitee.io/algo/2/18/23/) 

[731. 我的日程安排表 II](https://leetcode.cn/problems/my-calendar-ii/) 

[732. 我的日程安排表 III](https://leetcode.cn/problems/my-calendar-iii/) 

[【宫水三叶】线段树（动态开点）运用题](https://leetcode.cn/problems/my-calendar-ii/solution/by-ac_oier-okkc/) 

[【宫水三叶】线段树（动态开点）运用题](https://leetcode.cn/problems/my-calendar-iii/solution/by-ac_oier-cv31/)


