# 凸包


## 前言

点集 Q 的凸包是一个最小的凸多边形 P，满足 Q 中每个点都在 P 的边界上或在 P 的内部。

{{< figure src="/images/ConvexHull.png" title="凸包" >}}

## 算法

### Jarvis 算法

算法思路：

先找到一个必定在凸包的点，比如最左边的点。在找到下一个点，该点要求如下：除了凸包上的点，其他点和凸包上最后的一个点所构成的向量都在该点和凸包最后一个点构成的向量的左边或者右边（后续都选择一边即可，比如一直选左边），并将该点作为凸包的最后一个点。按照上述规则循环，直到回到开始的第一个点。

判断一个向量是否在另一个向量的左侧还是右侧时，可以通过向量叉乘，两个二维向量的叉乘结果为 $z$ 轴向量，通过其值正负判断左右（根据右手螺旋法则）。若结果为正，则向量在其左侧，若结果为0，则同向，若结果为负，则向量在其右侧。

算法因为每个点都需要判断其他点是否在其左侧还是右侧，时间复杂度为 $O(n*h)$，h 为凸包上点的数量，n 为总顶点数量。

```java
class Solution {
    public int[][] outerTrees(int[][] trees) {
        int n = trees.length;
        if (n < 4) {
            return trees;
        }
        int leftMost = 0;
        for (int i = 0; i < n; i++) {
            if (trees[i][0] < trees[leftMost][0]) {
                leftMost = i;
            }
        }

        List<int[]> res = new ArrayList<int[]>();
        boolean[] visit = new boolean[n];
        int p = leftMost;
        do {
            int q = (p + 1) % n;
            for (int r = 0; r < n; r++) {
                /* 如果 r 在 pq 的右侧，则 q = r */ 
                if (cross(trees[p], trees[q], trees[r]) < 0) {
                    q = r;
                }
            }
            /* 是否存在点 i, 使得 p 、q 、i 在同一条直线上 */
            for (int i = 0; i < n; i++) {
                if (visit[i] || i == p || i == q) {
                    continue;
                }
                if (cross(trees[p], trees[q], trees[i]) == 0) {
                    res.add(trees[i]);
                    visit[i] = true;
                }
            }
            if  (!visit[q]) {
                res.add(trees[q]);
                visit[q] = true;
            }
            p = q;
        } while (p != leftMost);
        return res.toArray(new int[][]{});
    }

    public int cross(int[] p, int[] q, int[] r) {
        return (q[0] - p[0]) * (r[1] - q[1]) - (q[1] - p[1]) * (r[0] - q[0]);
    }
}
```

### Graham 算法

算法思路：

首先选择一个凸包上的初始点 $bottom$ 。我们选择 $y$ 坐标最小的点为起始点，如果有多个，则选择最最左边的点，我们可以肯定 $bottom$ 一定在凸包上，将给定点集按照相对的以 $bottom$ 为原点的极角大小进行排序。

这一排序过程大致给了我们在逆时针顺序选点时候的思路。为了将点排序，我们使用上一方法使用过的函数 $cross$ 。极角顺序更小的点排在数组的前面。如果有两个点相对于点 $bottom$ 的极角大小相同，则按照与点 $bottom$ 的距离排序。

我们还需要考虑另一种重要的情况，如果共线的点在凸壳的最后一条边上，我们需要从距离初始点最远的点开始考虑起。所以在将数组排序后，我们从尾开始遍历有序数组并将共线且朝有序数组尾部的点反转顺序，因为这些点是形成凸壳过程中尾部的点，所以在经过了这些处理以后，我们得到了求凸壳时正确的点的顺序。

现在我们从有序数组最开始两个点开始考虑。我们将这条线上的点放入栈中。然后我们从第三个点开始遍历有序数组 。如果当前点与栈顶的点相比前一条线是一个「左拐」或者是同一条线段上，我们都将当前点添加到栈顶，表示这个点暂时被添加到凸壳上。

如果当前点与上一条线之间的关系是右拐的，说明上一个点不应该被包括在凸壳里，因为它在边界的里面，所以我们将它从栈中弹出并考虑倒数第二条线的方向。重复这一过程，弹栈的操作会一直进行，直到我们当前点在凸壳中出现了右拐。这表示这时凸壳中只包括边界上的点而不包括边界以内的点。在所有点被遍历了一遍以后，栈中的点就是构成凸壳的点。

```java
class Solution {
    public int[][] outerTrees(int[][] trees) {
        int n = trees.length;
        if (n < 4) {
            return trees;
        }
        int bottom = 0;
        /* 找到 y 最小的点 bottom*/
        for (int i = 0; i < n; i++) {
            if (trees[i][1] < trees[bottom][1]) {
                bottom = i;
            }
        }
        swap(trees, bottom, 0);
        /* 以 bottom 原点，按照极坐标的角度大小进行排序 */
        Arrays.sort(trees, 1, n, (a, b) -> {
            int diff = cross(trees[0], a, b) - cross(trees[0], b, a);
            if (diff == 0) {
                return distance(trees[0], a) - distance(trees[0], b);
            } else {
                return -diff;
            }
        });
        /* 对于凸包最后且在同一条直线的元素按照距离从小到大进行排序 */
        int r = n - 1;
        while (r >= 0 && cross(trees[0], trees[n - 1], trees[r]) == 0) {
            r--;
        }
        for (int l = r + 1, h = n - 1; l < h; l++, h--) {
            swap(trees, l, h);
        }
        Deque<Integer> stack = new ArrayDeque<Integer>();
        stack.push(0);
        stack.push(1);
        for (int i = 2; i < n; i++) {
            int top = stack.pop();
            /* 如果当前元素与栈顶的两个元素构成的向量顺时针旋转，则弹出栈顶元素 */
            while (!stack.isEmpty() && cross(trees[stack.peek()], trees[top], trees[i]) < 0) {
                top = stack.pop();
            }
            stack.push(top);
            stack.push(i);
        }

        int size = stack.size();
        int[][] res = new int[size][2];
        for (int i = 0; i < size; i++) {
            res[i] = trees[stack.pop()];
        }
        return res;
    }

    public int cross(int[] p, int[] q, int[] r) {
        return (q[0] - p[0]) * (r[1] - q[1]) - (q[1] - p[1]) * (r[0] - q[0]);
    }

    public int distance(int[] p, int[] q) {
        return (p[0] - q[0]) * (p[0] - q[0]) + (p[1] - q[1]) * (p[1] - q[1]);
    }

    public void swap(int[][] trees, int i, int j) {
        int temp0 = trees[i][0], temp1 = trees[i][1];
        trees[i][0] = trees[j][0];
        trees[i][1] = trees[j][1];
        trees[j][0] = temp0;
        trees[j][1] = temp1;
    }
}
```

### Andrew 扫描法

算法思路：

将所有的点排序，先 $x$ 坐标，后 $y$ 坐标。排序的第一一个点和最后一个点必定为凸包上的顶点，他们之间的部分可以分为上下两条链进行求解。求下侧链时，从小到大处理排序上的节点，逐步构造凸包，构造过程中的凸包末尾加上的节点可能会破坏凸性，使用向量叉乘判断凹凸性，将凹的部分节点从末尾提出即可。求上侧的链则从大到小处理。

算法因为需要排序，排序的时间复杂度为 `O(nlogn)`  ，剩下部分处理的时间复杂度为 `O(n)`  ，总的时间复杂度为 `O(nlogn)` 。

```java
class Solution {
    public int[][] outerTrees(int[][] trees) {
        int n = trees.length;
        if (n < 4) {
            return trees;
        }
        /* 按照 x 大小进行排序，如果 x 相同，则按照 y 的大小进行排序 */
        Arrays.sort(trees, (a, b) -> {
            if (a[0] == b[0]) {
                return a[1] - b[1];
            }
            return a[0] - b[0];
        });
        List<Integer> hull = new ArrayList<Integer>();
        boolean[] used = new boolean[n];
        /* hull[0] 需要入栈两次，不进行标记 */
        hull.add(0);
        /* 求出凸包的下半部分 */
        for (int i = 1; i < n; i++) {
            while (hull.size() > 1 && cross(trees[hull.get(hull.size() - 2)], trees[hull.get(hull.size() - 1)], trees[i]) < 0) {
                used[hull.get(hull.size() - 1)] = false;
                hull.remove(hull.size() - 1);
            }
            used[i] = true;
            hull.add(i);
        }
        int m = hull.size();
        /* 求出凸包的上半部分 */
        for (int i = n - 2; i >= 0; i--) {
            if (!used[i]) {
                while (hull.size() > m && cross(trees[hull.get(hull.size() - 2)], trees[hull.get(hull.size() - 1)], trees[i]) < 0) {
                    used[hull.get(hull.size() - 1)] = false;
                    hull.remove(hull.size() - 1);
                }
                used[i] = true;
                hull.add(i);
            }
        }
        /* hull[0] 同时参与凸包的上半部分检测，因此需去掉重复的 hull[0] */
        hull.remove(hull.size() - 1);
        int size = hull.size();
        int[][] res = new int[size][2];
        for (int i = 0; i < size; i++) {
            res[i] = trees[hull.get(i)];
        }
        return res;
    }

    public int cross(int[] p, int[] q, int[] r) {
        return (q[0] - p[0]) * (r[1] - q[1]) - (q[1] - p[1]) * (r[0] - q[0]);
    }
}
```



## 参考资料

[安装栅栏](https://leetcode-cn.com/problems/erect-the-fence/)

