# 线段树


## 前言

线段树擅长处理区间，形状如下图 。线段树是一颗完美二叉树，其叶子存储元素值值，非叶子节点存储对应区间的值，根节点维护整个区间，对区间的操作可以在 `O(log n)` 时间复杂度完成。

{{< image src="/images//Segment_tree.svg" alt="conter" caption="`Transformer 模型框架`)" height="100" width="600">}}

## 算法思想

如果要对连续区间重复多次操作时，比如求区间的最值/和等，利用线段树是可以从 `O(m * n)` 降低至 `O(m * logn)` ，`m` 为操作次数，  `n` 为总区间长度。算法基本过程：

1. 构造线段树。利用所给的区间元素，构造线段树。由于父节点需要存储两个子节点运算后的值，可以采用 ` 自底向上` 构造线段树，比如使用递归方法。
2. 更改线段树。更改某元素的值时，也需要更改父节点以上的值，和构造阶段一样，采用 ` 自底向上` 的方法依次更改。
3. 求区间值。求区间则使用 `自顶向下` 方法实现取值，根据父节点的区间范围与求取的范围，不断的递归向下取值，同时根据所求范围将区间进行拆分为若干个极大区间，直到左右区间达到所给而范围。

### 再优化

当需要改变区间内某些元素值时，可以在所对应区间内（包括父节点的区间）进行标记，从而记录更改，而当需要查询到该区间时，在依次递归进行更改，减少因为更改而增加的复杂度。

## 代码实现

### 区间求和

 [307. 区域和检索 - 数组可修改](https://leetcode-cn.com/problems/range-sum-query-mutable/)

```markdown
给你一个数组 nums ，请你完成两类查询。

其中一类查询要求 更新 数组 nums 下标对应的值
另一类查询要求返回数组 nums 中索引 left 和索引 right 之间（ 包含 ）的nums元素的 和 ，其中 left <= right
实现 NumArray 类：

NumArray(int[] nums) 用整数数组 nums 初始化对象
void update(int index, int val) 将 nums[index] 的值 更新 为 val
int sumRange(int left, int right) 返回数组 nums 中索引 left 和索引 right 之间（ 包含 ）的nums元素的 和 （即，nums[left] + nums[left + 1], ..., nums[right]）
```

线段树各层数量为等比数量，最大需要 `4n` 空间。根据二叉树性质，父节点 `i` 的两个子节点对应的索引分别为 `2 * i + 1 ` 和 ` 2  * i + 2` ，`i` 从0开始编号，从而可以使用数组来表示线段树。

```java
class NumArray {
    private int[] segmentTree;
    private int n;

    public NumArray(int[] nums) {
        n = nums.length;
        segmentTree = new int[nums.length * 4];
        build(0, 0, n - 1, nums);
    }

    public void update(int index, int val) {
        change(index, val, 0, 0, n - 1);
    }

    public int sumRange(int left, int right) {
        return range(left, right, 0, 0, n - 1);
    }

    private void build(int node, int s, int e, int[] nums) {
        if (s == e) {
            segmentTree[node] = nums[s];
            return;
        }
        int m = s + (e - s) / 2;
        build(node * 2 + 1, s, m, nums);
        build(node * 2 + 2, m + 1, e, nums);
        segmentTree[node] = segmentTree[node * 2 + 1] + segmentTree[node * 2 + 2];
    }

    private void change(int index, int val, int node, int s, int e) {
        if (s == e) {
            segmentTree[node] = val;
            return;
        }
        int m = s + (e - s) / 2;
        if (index <= m) {
            change(index, val, node * 2 + 1, s, m);
        } else {
            change(index, val, node * 2 + 2, m + 1, e);
        }
        segmentTree[node] = segmentTree[node * 2 + 1] + segmentTree[node * 2 + 2];
    }

    private int range(int left, int right, int node, int s, int e) {
        if (left == s && right == e) {
            return segmentTree[node];
        }
        int m = s + (e - s) / 2;
        if (right <= m) {
            return range(left, right, node * 2 + 1, s, m);
        } else if (left > m) {
            return range(left, right, node * 2 + 2, m + 1, e);
        } else {
            return range(left, m, node * 2 + 1, s, m) + range(m + 1, right, node * 2 + 2, m + 1, e);
        }
    }
}
```

链表实现，因为构造线段树采用 `自顶向下` 实现，所以使用了前缀和来构造线段树。

```java
class TreeNode{
    int val;
    int low;
    int high;
    TreeNode left;
    TreeNode right;
    TreeNode parent;
}
class NumArray {
    int n;
    TreeNode root;
    int[] prefix;
    int sum;
    public NumArray(int[] nums) {
        n = nums.length;
        root = new TreeNode();
        prefix = new int[n];
        prefix[0] = nums[0];
        for(int i = 1; i < n; i++){
            prefix[i] = prefix[i - 1] + nums[i];
        }

        root.val = prefix[n-1];
        root.low = 0;
        root.high = n -1;
        root.left = init(root, nums, 0, (n-1) / 2);
        root.right = init(root, nums, (n-1) / 2 + 1, n - 1);
    }

    public TreeNode init(TreeNode root, int[] nums, int low, int high){
        if(low > high){
            return null;
        }
        TreeNode child = new TreeNode();
        child.parent = root;
        child.low = low;
        child.high = high;
        if(low == 0){
            child.val = prefix[high];
        }else{
            child.val = prefix[high] - prefix[low - 1];            
        }
        
        if(low == high){                
            return child;
        }else{
            child.left = init(child, nums, low, (low + high) / 2);
            child.right = init(child, nums, (low + high) / 2 + 1, high);
            return child;
        }      
    }
    
    public void update(int index, int val) {
        TreeNode p = root;
        int mid = (p.low + p.high) / 2;
        while(p.low != index || p.high != index){
            if(mid < index){
                p = p.right;
            }else{
                p = p.left;
            }
            mid = (p.low + p.high) / 2;
        }

        p.val = val;
        while(p.parent != null){
            if(p.parent.right != null){
                p.parent.val = p.parent.left.val + p.parent.right.val;
            }else{
                p.parent.val = p.parent.left.val;
            }   
            p = p.parent;
        }
    }
    
    public int sumRange(int left, int right) {
        sum = 0;
        TreeNode p = root;
        if(left == root.low && right == root.high){
            return root.val;
        }       
        return getSum(p, left, right);
    }
    
    public void getSum(TreeNode root, int left, int right) {
        if(root == null){
            return;
        }
    	int mid = (root.low + root.high) / 2;
    	if(root.low == left && root.high == right) {
    		sum = root.val;
            return;
    	}
    	
    	if(root.low == left && root.high < right) {
    		sum += root.val;
    		return ;
    	}
    	
    	if(root.low > left && root.high < right) {
    		sum += root.val;
    		return;
    	}
    	
    	if(root.low > left && root.high == right) {
    		sum += root.val;
    		return;
    	}
    	
    	if(mid <= right) {    		
    		getSum(root.right, left, right);
    	}
    	getSum(root.left, left, right);

    }
}
```

## 动态开点线段树

当离散点数量过大时，比如 $1e9$ 级别，直接创建线段树可能会内存超出，所以需要动态开点创建线段树，此时直接使用数组无法进行表示，需要使用节点对象数组。该节点需要维护信息的如下：

- `ls/rs`: 分别代表当前节点的左右子节点在线段树数组 `tr` 中的下标；
- `add`: 懒标记；
- `val`: 为当前区间的所包含的点的数量

[729. 我的日程安排表 I](https://leetcode.cn/problems/my-calendar-i/) 

动态开点相比于原始的线段树实现，本质仍是使用「满二叉树」的形式进行存储，只不过是按需创建区间，如果我们是按照连续段进行查询或插入，最坏情况下仍然会占到 $O(4*n)$ 的空间，因此盲猜 $\log{n}$ 的常数在 4 左右，保守一点可以直接估算到 6，因此我们可以估算点数为 $6 * m * \log{n}$ ，其中 $n = 1e9$  和 $m = 1e3$ 分别代表值域大小和查询次数。

当然一个比较实用的估点方式可以「尽可能的多开点数」，利用题目给定的空间上界和我们创建的自定义类（结构体）的大小，尽可能的多开（ Java 的 128M 可以开到 $5 * 10^6$ 以上）

代码实现如下：

```java
class MyCalendar {
    class Node {
        int ls, rs, add, val;
    }
    int N = (int)1e9, M = 120010, cnt = 1;
    Node[] tr = new Node[M];
    void update(int u, int lc, int rc, int l, int r, int v) {
        if (l <= lc && rc <= r) {
            tr[u].val += (rc - lc + 1) * v;
            tr[u].add += v;
            return ;
        }
        lazyCreate(u);
        pushdown(u, rc - lc + 1);
        int mid = lc + rc >> 1;
        if (l <= mid) update(tr[u].ls, lc, mid, l, r, v);
        if (r > mid) update(tr[u].rs, mid + 1, rc, l, r, v);
        pushup(u);
    }
    int query(int u, int lc, int rc, int l, int r) {
        if (l <= lc && rc <= r) return tr[u].val;
        lazyCreate(u);
        pushdown(u, rc - lc + 1);
        int mid = lc + rc >> 1, ans = 0;
        if (l <= mid) ans = query(tr[u].ls, lc, mid, l, r);
        if (r > mid) ans += query(tr[u].rs, mid + 1, rc, l, r);
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
    void pushdown(int u, int len) {
        tr[tr[u].ls].add += tr[u].add; tr[tr[u].rs].add += tr[u].add;
        tr[tr[u].ls].val += len / 2 * tr[u].add; tr[tr[u].rs].val += len / 2 * tr[u].add;
        tr[u].add = 0;
    }
    void pushup(int u) {
        tr[u].val = tr[tr[u].ls].val + tr[tr[u].rs].val;
    }
    public boolean book(int start, int end) {
        if (query(1, 1, N + 1, start + 1, end) > 0) return false;
        update(1, 1, N + 1, start + 1, end, 1);
        return true;
    }
}
```

## 参考资料

[线段树](https://zh.wikipedia.org/wiki/%E7%B7%9A%E6%AE%B5%E6%A8%B9)

[OI Wiki 线段树](https://oi-wiki.org/ds/seg/)

[【宫水三叶】一题双解 :「模拟」&「线段树（动态开点](https://leetcode.cn/problems/my-calendar-i/solution/by-ac_oier-1znx/) 

[算法学习笔记(49): 线段树的拓展](https://zhuanlan.zhihu.com/p/246255556) 

[动态开点线段树](https://chenshouao.github.io/mkdocs/pages/Algorithm/ds/dynamicTree.html) 




