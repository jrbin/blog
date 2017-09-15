---
title: COMP9024 Graph Algorithm
date: 2017-09-15T06:17:41.000Z
tags:
  - comp9024
---

# Graph Algorithm

## Path finding

### DFS

Depth first search 深度优先

![dfs path finding](/img/traversal.png)

```no-highlight
0
0, 1
0, 1, 5
0, 1, 5, 3
0, 1, 5, 3, 2
0, 1, 5, 3, 4
0, 1, 5, 3, 4, 7
0, 1, 5, 3, 4, 7, 8
0, 1, 5, 3, 4, 7, 8, 9
0, 1, 5, 6
```

伪代码

```no-highlihg
function dfs(src, dst) {
    visited[src] = true
    for edge from src {
        if not visited[edge.dst] {
            if edge.dst == dst {
                return true
            }
            if dfs(edge.dst, dst)  {
                return true
            }
        }
    }
    return false
}
```

### BFS

Breadth first search 广度优先

![dfs path finding](/img/traversal.png)

```no-highlight
0
1, 2, 5
2, 5
5, 3
3, 4, 6, 7
4, 6, 7, 8
6, 7, 8
7, 8
8, 9
9
```

伪代码

```no-highlight
function bfs(src, dst) {
    q = queue_init()
    enqueue(q, src)
    visited[src] = true
    while queue is not empty {
        node = dequeue(q)
        for edge from node {
            if not visited[edge.dst] {
                visited[edge.dst] = true
                enqueue(q, edge.dst)
            }
        }
    }
}
```

### 计算连通分量数量

假设一个图不是连通的，也就是说它至少有2个连通分量，用一种算法计算其中连通分量个数。

提示：用dfs填充每一个分量，在外层调用dfs的次数就是连通分量的个数，因为每次dfs会填满一个连通分量。

### Hamiltonian path

一条经过所有点的路径，并且路径中没有重复顶点。如果开始顶点和结束顶点是同一个，则是Hamiltonian circuit

代码不用记，记思路，就是dfs找遍图中每一条路径。

练习

![Hamiltonian path](/img/traversal3.png)

### Euler Path

一条刚好经过图中所有边的路径。如果开始顶点和结束顶点是同一个，则是Euler circuit

> 理论：有Euler circuit <-> 是连通图 并且 所有顶点度数是偶数

> 理论：有Euler path <-> 是连通图 并且 有且只有2个顶点度数是奇数（就是开始和结束的那两个顶点）

## Transitive Closure

意思就是从每个点出发，最终能到另外哪些点。最后也会得到一个matrix叫reachability matrix

![Transitive Closure](/img/tc.png)

算法是Warshall algorithm，其实很好记，三个for循环

```c
int **tc_mat = copy(adjacency_mat); // pseudo code
for (int k = 0; k < n_vertices; k++) {
    for (int i = 0; i < n_vertices; i++) {
        for (int j = 0; j < n_vertices; j++) {
            if (tc_mat[i][k] && tc_mat[k][j]) {
                tc_mat[i][j] = 1;
            }
        }
    }
}

// calculate shortest path using warshall's algorithm
// 少用因为是O(V^3)的，后面讲的比较常用的dijkstra是O(V^2)的
// 但dijkstra不能处理有负边的图，这种算法能判断图中有没有负边
for (int k = 0; k < n_vertices; k++) {
    for (int i = 0; i < n_vertices; i++) {
        for (int j = 0; j < n_vertices; j++) {
            if (tc_mat[i][j] > tc_mat[i][k] + tc_mat[k][j]) {
                tc_mat[i][j] = tc_mat[i][k] + tc_mat[k][j];
            }
        }
    }
}
```

## Shortest path

单源最短路，常用的Dijkstra, Bellman Ford, Floyd Warshall

### Dijkstra

![Dijkstra](/img/dijkstra.png)


| Iteration | 0 | 1  | 2 | 3 | 4  | 5  | visit     |
|-----------|---|----|---|---|----|----|-----------|
| 0         | 0 | +  | + | + | +  | +  | 0         |
| 1         | 0 | 14 | 9 | 7 | +  | +  | 3         |
| 2         | 0 | 14 | 9 | 7 | +  | 22 | 2         |
| 3         | 0 | 13 | 9 | 7 | +  | 12 | 5         |
| 4         | 0 | 13 | 9 | 7 | 20 | 12 | 1         |
| 5         | 0 | 13 | 9 | 7 | 18 | 12 | 4         |
| 6         | 0 | 13 | 9 | 7 | 18 | 12 | no update |


```c
void dijkstra(int src) {
    int dist[n_vertices] = {INT_MAX};
    int pred[n_vertices] = {-1};
    int visited[n_vertices] = {0};
    dist[src] = 0;
    while (1) {
        // find unvisited vertex with min dist
        int v = -1;
        int min_dist = INT_MAX;
        for (int i = 0; i < n_vertices; i++) {
            if (!visited[v] && dist[v] < min_dist) {
                min_dist = dist[v];
                v = i;
            }
        }
        if (v == -1) {
            // no update, end the algorithm
            break;
        }
        visited[v] = 1;
        // update v's neighbours
        for (int u = 0; u < n_vertices; u++) {
            if (mat[v][u] && dist[u] > dist[v] + mat[v][u]) {
                dist[u] = dist[v] + mat[v][u];
                pred[u] = v;
            }
        }
    }
}
```

\\(O(V^2)\\)

用堆优化可以更快，lecture slide里也只是提了一下

## Minimum spanning tree

最小生成树，一个图的生成树，并且这个生成树的所有边的和是在这个图的所有生成树中最小的

### Prim

思路有点类似Dijkstra，从任意点出发，每次选取当前能够选取的最小的边。可以把Dijkstra的代码稍微改一下就得到这个算法。


```c
int prim(int src) {
    int dist[n_vertices] = {INT_MAX};
    int visited[n_vertices] = {0};
    dist[src] = 0;
    int result = 0;
    while (1) {
        // find unvisited vertex with min dist
        int v = -1;
        int min_dist = INT_MAX;
        for (int i = 0; i < n_vertices; i++) {
            if (!visited[v] && dist[v] < min_dist) {
                min_dist = dist[v];
                v = i;
            }
        }
        if (v == -1) {
            // no update, end the algorithm
            break;
        }
        result += dist[v];
        visited[v] = 1;
        // update v's neighbours
        for (int u = 0; u < n_vertices; u++) {
            if (mat[v][u] && dist[u] > dist[v]) {
                dist[u] = dist[v];
            }
        }
    }
    return result;
}
```

### Kruskal

每次从边中选取最小的加入到生成树中，只要不与已经选择的边形成cycle就行了，直到这个生成树有n-1条边，停止算法。

用Array of edges实现起来比较简单。

两种算法最后得到的最小生成树可能不一样，但最小生成树的边的和是相等的。

![spanning tree](/img/spanning-tree.png)
