# 蓄水池抽样法


## 前言

[382. 链表随机节点](https://leetcode-cn.com/problems/linked-list-random-node/) 给你一个单链表，随机选择链表的一个节点，并返回相应的节点值。每个节点 **被选中的概率一样** 。

实现 `Solution` 类：

- `Solution(ListNode head)` 使用整数数组初始化对象。
- `int getRandom()` 从链表中随机选择一个节点并返回该节点的值。链表中所有节点被选中的概率相等。

## 算法思想

以下内容摘自[蓄水池抽样算法（Reservoir Sampling）](https://www.jianshu.com/p/7a9ea6ece2af)

**给定一个数据流，数据流长度N很大，且N直到处理完所有数据之前都不可知，请问如何在只遍历一遍数据（O(N)）的情况下，能够随机选取出m个不重复的数据。**

### 原理

**第i个接收到的数据最后能够留在蓄水池中的概率**=**第i个数据进入过蓄水池的概率\**\**之后第i个数据不被替换的概率**（第i+1到第N次处理数据都不会被替换）。

1. 当i<=m时，数据直接放进蓄水池，所以**第i个数据进入过蓄水池的概率=1**。
2. 当i>m时，在[1,i]内选取随机数d，如果d<=m，则使用第i个数据替换蓄水池中第d个数据，因此**第i个数据进入过蓄水池的概率=m/i**。
3. 当i<=m时，程序从接收到第m+1个数据时开始执行替换操作，第m+1次处理会替换池中数据的为m/(m+1)，会替换掉第i个数据的概率为1/m，则第m+1次处理替换掉第i个数据的概率为(m/(m+1))*(1/m)=1/(m+1)，不被替换的概率为1-1/(m+1)=m/(m+1)。依次，第m+2次处理不替换掉第i个数据概率为(m+1)/(m+2)...第N次处理不替换掉第i个数据的概率为(N-1)/N。所以，之后**第i个数据不被替换的概率=m/(m+1)*(m+1)/(m+2)*...*(N-1)/N=m/N**。
4. 当i>m时，程序从接收到第i+1个数据时开始有可能替换第i个数据。则参考上述第3点，**之后第i个数据不被替换的概率=i/N**。
5. 结合第1点和第3点可知，当i<=m时，第i个接收到的数据最后留在蓄水池中的概率=1*m/N=m/N。结合第2点和第4点可知，当i>m时，第i个接收到的数据留在蓄水池中的概率=m/i*i/N=m/N。综上可知，**每个数据最后被选中留在蓄水池中的概率为m/N**。

## 拓展

1. 假设有K台机器，将大数据集分成K个数据流，每台机器使用单机版蓄水池抽样处理一个数据流，抽样m个数据，并最后记录处理的数据量为N1, N2, ..., Nk, ..., NK(假设m<Nk)。N1+N2+...+NK=N。
2. 取[1, N]一个随机数d，若d<N1，则在第一台机器的蓄水池中等概率不放回地（1/m）选取一个数据；若N1<=d<(N1+N2)，则在第二台机器的蓄水池中等概率不放回地选取一个数据；一次类推，重复m次，则最终从N大数据集中选出m个数据。

验证如下：

1. 第k台机器中的蓄水池数据被选取的概率为m/Nk。
2. 从第k台机器的蓄水池中选取一个数据放进最终蓄水池的概率为Nk/N。
3. 第k台机器蓄水池的一个数据被选中的概率为1/m。（不放回选取时等概率的）
4. 重复m次选取，则每个数据被选中的概率为**m\*(m/Nk\*Nk/N\*1/m)=m/N**

## 代码

### [382. 链表随机节点](https://leetcode-cn.com/problems/linked-list-random-node/)

```markdown
给你一个单链表，随机选择链表的一个节点，并返回相应的节点值。每个节点 被选中的概率一样 。

实现 Solution 类：

Solution(ListNode head) 使用整数数组初始化对象。
int getRandom() 从链表中随机选择一个节点并返回该节点的值。链表中所有节点被选中的概率相等。
```

蓄水池大小选择为 1，若遍历当前节点的概率为 $1 / i$ 时，则替换为当前节点，所以替换的概率比为 $1/i$，没有被替换的概率为 $(i - 1/i$。第 i 个点被选择的概率公式如下：

$$ p = 1 / i * i / (i + 1) * (i + 1) / (i + 2) * ... (n - 1)/n = 1 / n$$

第一项为在第 i 个节点被选择的概率，后续为在 i 个节点后没有被替换的概率

```java
class Solution {
    ListNode head;
    Random random;

    public Solution(ListNode head) {
        this.head = head;
        random = new Random();
    }

    public int getRandom() {
        int i = 1, ans = 0;
        for (ListNode node = head; node != null; node = node.next) {
            if (random.nextInt(i) == 0) { // 1/i 的概率选中（替换为答案）
                ans = node.val;
            }
            ++i;
        }
        return ans;
    }
}
```

### [497. 非重叠矩形中的随机点](https://leetcode.cn/problems/random-point-in-non-overlapping-rectangles/)

```markdown
给定一个由非重叠的轴对齐矩形的数组 rects ，其中 rects[i] = [ai, bi, xi, yi] 表示 (ai, bi) 是第 i 个矩形的左下角点，(xi, yi) 是第 i 个矩形的右上角角点。设计一个算法来随机挑选一个被某一矩形覆盖的整数点。矩形周长上的点也算做是被矩形覆盖。所有满足要求的点必须等概率被返回。

在一个给定的矩形覆盖的空间内任何整数点都有可能被返回。

请注意 ，整数点是具有整数坐标的点。

实现 Solution 类:

Solution(int[][] rects) 用给定的矩形数组 rects 初始化对象。
int[] pick() 返回一个随机的整数点 [u, v] 在给定的矩形所覆盖的空间内。
```

由于矩形中的点数多，所以可能全部存储，因此可以维持一个大小为 1 的蓄水池进行采样，若概率等于 $1/i$ 时，则选择矩形对应点。

**注意每个矩形大小不一样，而题目要求选择点的概率一致，因此此处 i 应为替换为矩形的点的数量，即 count / total** 

```java
class Solution {
    int[][] rects;
    Random rand;

    public Solution(int[][] rects) {
        this.rects = rects;
        this.rand = new Random();
    }

    public int[] pick() {
        int[] ans = new int[2];
        int total = 0;
        for (int i = 0; i < rects.length; i++) {
            int a = rects[i][0];
            int b = rects[i][1];
            int x = rects[i][2];
            int y = rects[i][3];
            int count = (x - a + 1) * (y - b + 1);
            // 当前遍历总的点的数量
            total += count;
            // 若概率等于当前矩形的点的数量的百分比 i / total时，则进行替换
            if(rand.nextInt(total) < count){
                int x1 = rand.nextInt(x - a + 1) + a;
                int y1 = rand.nextInt(y - b + 1) + b;
                ans = new int[]{x1, y1};
            }
        }
        return ans;
    }
}
```

## 其他

[398. 随机数索引](https://leetcode-cn.com/problems/random-pick-index/)

因为蓄水池抽样法的概率为等概率时间，通常可以用来等概率采样。等概率采用方法还可以通过 ArrayList 实现，但需要额外而空间复杂度 $O(n)$，而蓄水池采样的空间复杂度为固定常数 $O(1)$ ，可以利用时间复杂度换取空间复杂度。

## 参考链接

[蓄水池抽样算法（Reservoir Sampling）](https://www.jianshu.com/p/7a9ea6ece2af)

[382. 链表随机节点](https://leetcode-cn.com/problems/linked-list-random-node/)

[398. 随机数索引](https://leetcode-cn.com/problems/random-pick-index/)

