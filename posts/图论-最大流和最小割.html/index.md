# 图论 最大流和最小割


## 前言

在图论中，找到单点 `s` 到单点 `t` 的最大流，称为最大流问题，其中 `s-t` 的**最大流**等于 `s-t` 的**最小割**，使用 `Ford–Fulkerson` 算法可以确认最大流。

{{< figure src="/images//Max_flow.svg" title="维基百科 最大流" >}}

## 最大流

求由 `s` 发出，`t` 每秒接受的最大流量，称为最大流问题。首先可以利用贪心算法，确认 `s->t` 的一条路径，但该路径不一定是最大流。`Ford–Fulkerson` 算法提出了一个**残余网络**，也叫**增广路**，将之前的`s->t`路径的流进行反转，同时原网络减去 `s->t` 的流量，二者合并构成残余网络。同时在根据残余网络找到 `s->t` 的路径。根据上述规则，不断的重复迭代，直到 `s->t` 没有到达路径流，则可以得到最大流（最大流量在每次迭代的过程中都是累加的，因为只需要计算 `s` 到 `t` 的流量，终点 `t` 每次迭代后流入流量都是在累加）。

将流量路径反转的目的是在于可以重新规划边的流量流向，若后续还存在 `s->t` 路径，且需要更改反转的路径大小，则表面需要调整之前的流量大小，以此提高 `s` 到 `t` 的流量大小。

## 最小割

图的割，指的是对于某个顶点集合  $S \subseteq V$ ，从 S 出发指向外部那些边的集合，记为割(S, V\S)。这些边的流量容量之和称为割的容量。通俗来说，就是将一个连通图，划分为两个不连通的子图，而割代表的是再划分时，所需要去掉的边。为了让 `s` 和 `t` 分别处于不同的子图，所需要的最小边的容量，记为最小割。当 `s` 和 `t` 处于不同的子图时，`s` 和 `t` 即不存在相连通的路径。

## 基本公式

记$f$为`s-t`的流量，则$f = (s出边的总流量)$ ，对于$v \in S${s} 时，又有 $(v的出边的总流量) = (v的入边的总流量)$，所以 $f = (S的出边的总流量) - (S的入边的总流量)$ ，所以 $f \le (割的流量)$。

对于残余网络来说，$f^‘$ 对应的残余网路中`s` 到可达 `v` 的的集合为 `S` ，因为$f^‘$ 不存在 `s->t` 的路径，所以(S, V\S)就是一个s-t割。根据 $f^‘ = (S的出边的总流量) - (S的入边的总流量) = (割的流量)$ ，所以$f^‘$ 为最大流。

## 代码实现

可以利用DFS或者BFS进行路径搜索，并不断进行迭代，直到两点之间没有路径，即可以达到最大流量。

```java
// Ford-Fulkerson algorith in Java

import java.util.LinkedList;

class FordFulkerson {
  static final int V = 6;

  // Using BFS as a searching algorithm 
  boolean bfs(int Graph[][], int s, int t, int p[]) {
    boolean visited[] = new boolean[V];
    for (int i = 0; i < V; ++i)
      visited[i] = false;

    LinkedList<Integer> queue = new LinkedList<Integer>();
    queue.add(s);
    visited[s] = true;
    p[s] = -1;

    while (queue.size() != 0) {
      int u = queue.poll();

      for (int v = 0; v < V; v++) {
        if (visited[v] == false && Graph[u][v] > 0) {
          queue.add(v);
          p[v] = u;
          visited[v] = true;
        }
      }
    }

    return (visited[t] == true);
  }

  // Applying fordfulkerson algorithm
  int fordFulkerson(int graph[][], int s, int t) {
    int u, v;
    int Graph[][] = new int[V][V];

    for (u = 0; u < V; u++)
      for (v = 0; v < V; v++)
        Graph[u][v] = graph[u][v];

    int p[] = new int[V];

    int max_flow = 0;

    # Updating the residual calues of edges
    while (bfs(Graph, s, t, p)) {
      int path_flow = Integer.MAX_VALUE;
      for (v = t; v != s; v = p[v]) {
        u = p[v];
        path_flow = Math.min(path_flow, Graph[u][v]);
      }

      for (v = t; v != s; v = p[v]) {
        u = p[v];
        Graph[u][v] -= path_flow;
        Graph[v][u] += path_flow;
      }

      // Adding the path flows
      max_flow += path_flow;
    }

    return max_flow;
  }

  public static void main(String[] args) throws java.lang.Exception {
    int graph[][] = new int[][] { { 0, 8, 0, 0, 3, 0 }, { 0, 0, 9, 0, 0, 0 }, { 0, 0, 0, 0, 7, 2 },
        { 0, 0, 0, 0, 0, 5 }, { 0, 0, 7, 4, 0, 0 }, { 0, 0, 0, 0, 0, 0 } };
    FordFulkerson m = new FordFulkerson();

    System.out.println("Max Flow: " + m.fordFulkerson(graph, 0, 5));

  }
}
```

##  参考资料

[图论：最大流最小割详解](https://seineo.github.io/%E5%9B%BE%E8%AE%BA%EF%BC%9A%E6%9C%80%E5%A4%A7%E6%B5%81%E6%9C%80%E5%B0%8F%E5%89%B2%E8%AF%A6%E8%A7%A3.html)

[Ford-Fulkerson Algorithm](https://www.programiz.com/dsa/ford-fulkerson-algorithm)


