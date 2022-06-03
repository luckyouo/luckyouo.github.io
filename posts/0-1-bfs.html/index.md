# 0-1 BFS


## 前言

若要寻找两个节点路径最小权重，最为常用的方法是 Dijkstra， 当一个图各个节点的权重只有 0 和 1 （k）时，则可以使用 0-1 BFS 进行搜索

## 算法思想

0-1 BFS 算法是通过 BFS 搜索，当节点的权重为 0 时，则放置在队头，若为 1 时，则放置在队尾。初始时，默认从源节点到各个节点路径权重为无穷大，开始将源节点放置队列，并将权重置为 0。BFS 搜索节点，若节点路径权重发生改变时，则按上述规则入队，否则搜索队列的下一个节点，直到队列为空时，搜索结束，此时路径权重收缩至最小值。时间复杂度为 $O(m*n)$

## 代码实现

[2290. 到达角落需要移除障碍物的最小数目](https://leetcode.cn/problems/minimum-obstacle-removal-to-reach-corner/) 

```markdown
给你一个下标从 0 开始的二维整数数组 grid ，数组大小为 m x n 。每个单元格都是两个值之一：

0 表示一个 空 单元格，
1 表示一个可以移除的 障碍物 。
你可以向上、下、左、右移动，从一个空单元格移动到另一个空单元格。

现在你需要从左上角 (0, 0) 移动到右下角 (m - 1, n - 1) ，返回需要移除的障碍物的 最小 数目。
```

```java
class Solution {
    final static int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

    public int minimumObstacles(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        var dis = new int[m][n];
        for (var i = 0; i < m; i++) Arrays.fill(dis[i], Integer.MAX_VALUE);
        dis[0][0] = 0;
        var q = new ArrayDeque<int[]>();
        q.addFirst(new int[]{0, 0});
        while (!q.isEmpty()) {
            var p = q.pollFirst();
            int x = p[0], y = p[1];
            for (var d : dirs) {
                int nx = x + d[0], ny = y + d[1];
                if (0 <= nx && nx < m && 0 <= ny && ny < n) {
                    var g = grid[nx][ny];
                    if (dis[x][y] + g < dis[nx][ny]) {
                        dis[nx][ny] = dis[x][y] + g;
                        if (g == 0) q.addFirst(new int[]{nx, ny});
                        else q.addLast(new int[]{nx, ny});
                    }
                }
            }
        }
        return dis[m - 1][n - 1];
    }
}
```

## 参考资料

[0-1 BFS](https://leetcode.cn/problems/minimum-obstacle-removal-to-reach-corner/solution/0-1-bfs-by-endlesscheng-4pjt/)

[0-1 BFS Tutorial](https://codeforces.com/blog/entry/22276) 


