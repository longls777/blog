---
title: leetcode827
date: 2022-09-18 13:56:43
tags: 并查集
categories: Algorithm Problem
math: true
---

#### [827. 最大人工岛](https://leetcode.cn/problems/making-a-large-island/)

给你一个大小为 `n x n` 二进制矩阵 `grid` 。**最多** 只能将一格 `0` 变成 `1` 。

返回执行此操作后，`grid` 中最大的岛屿面积是多少？

**岛屿** 由一组上、下、左、右四个方向相连的 `1` 形成。

 

**示例 1:**

```
输入: grid = [[1, 0], [0, 1]]
输出: 3
解释: 将一格0变成1，最终连通两个小岛得到面积为 3 的岛屿。
```

**示例 2:**

```
输入: grid = [[1, 1], [1, 0]]
输出: 4
解释: 将一格0变成1，岛屿的面积扩大为 4。
```

**示例 3:**

```
输入: grid = [[1, 1], [1, 1]]
输出: 4
解释: 没有0可以让我们变成1，面积依然为 4。
```

 

**提示：**

- `n == grid.length`
- `n == grid[i].length`
- `1 <= n <= 500`
- `grid[i][j]` 为 `0` 或 `1`

#### 题解

```java
class Solution {
    static int N = 510;
    static int[] p = new int[N * N], sz = new int[N * N];
    int[][] dirs = new int[][]{{1,0},{-1,0},{0,1},{0,-1}};
    int find(int x) {
        if (p[x] != x) p[x] = find(p[x]);
        return p[x];
    }
    void union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return ;
        if (sz[ra] > sz[rb]) {
            union(b, a);
        } else {
            sz[rb] += sz[ra]; p[ra] = p[rb];
        }
    }
    public int largestIsland(int[][] g) {
        int n = g.length;
        for (int i = 1; i <= n * n; i++) {
            p[i] = i; sz[i] = 1;
        }
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (g[i][j] == 0) continue;
                for (int[] di : dirs) {
                    int x = i + di[0], y = j + di[1];
                    if (x < 0 || x >= n || y < 0 || y >= n || g[x][y] == 0) continue;
                    union(i * n + j + 1, x * n + y + 1);
                }
            }
        }
        int ans = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (g[i][j] == 1) {
                    ans = Math.max(ans, sz[find(i * n + j + 1)]);
                } else {
                    int tot = 1;
                    Set<Integer> set = new HashSet<>();
                    for (int[] di : dirs) {
                        int x = i + di[0],y = j + di[1];
                        if (x < 0 || x >= n || y < 0 || y >= n || g[x][y] == 0) continue;
                        int root = find(x * n + y + 1);
                        if (set.contains(root)) continue;
                        tot += sz[root];
                        set.add(root);
                    }
                    ans = Math.max(ans, tot);
                }
            }
        }
        return ans;
    }
}
```

**并查集** + **枚举**

为了方便，我们令 grid 为 g。

根据题意，容易想到通过「并查集」来维护所有连通块大小，再通过「枚举」来找最优翻转点。

具体的，我们可以先使用「并查集」维护所有 g[i] [j] = 1的块连通性，并在维护连通性的过程中，使用 sz[idx] 记录下每个连通块的大小（注意：只有连通块根编号，sz[idx] 才有意义，即只有 sz[find(x)] 才有意义）。

随后我们再次遍历 g，根据原始的 g[i] [j]的值进行分别处理：

若 g[i] [j]= 1，该位置不会作为翻转点，但真实最大面积未必是由翻转后所导致的（可能取自原有的连通块），因此我们需要将 sz[root] 参与比较，其中 root 为 (i, j) 所属的连通块根节点编号；

若 g[i] [j] = 0，该位置可作为翻转点，我们可以统计其四联通位置对应的连通块大小总和 tot（注意若四联通方向有相同连通块，只统计一次），那么 tot + 1即是翻转该位置所得到的新连通块大小。

最后对所有连通块大小取最大值即是答案。

一些细节：为了方便，我们令点 (i, j)的编号从 1 开始；

同时由于我们本身就要用 sz 数组，因此我们可以随手把并查集的「按秩合并」也加上。体现在 union 操作时，我们总是将小的连通块合并到大的连通块上，从而确保我们并查集单次操作即使在最坏情况下复杂度仍为O(log* n)（近似O(1)）。需要注意只有同时应用「路径压缩」和「按秩合并」，并查集操作复杂度才为 O(log* n)。

- 时间复杂度：$O(n^2)$
- 空间复杂度：$O(n^2)$

> https://leetcode.cn/problems/making-a-large-island/solution/by-ac_oier-1kmp/
>
> https://blog.csdn.net/weixin_38279101/article/details/112546053