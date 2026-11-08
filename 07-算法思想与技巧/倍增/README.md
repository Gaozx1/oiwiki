# 倍增（Binary Lifting）

> 倍增 = 每次以 $2$ 的幂次跳跃。它把「一步一步走」变成「一跳一大段」，是 LCA、快速幂、稀疏表等算法的共同思想。

---

## 目录

1. [核心思想](#1-核心思想)
2. [倍增求 LCA](#2-倍增求-lca)
3. [倍增的其他应用](#3-倍增的其他应用)
4. [模板代码](#4-模板代码)
5. [经典例题](#5-经典例题)
6. [易错点与练习](#6-易错点与练习)

---

## 1. 核心思想

任何一个正整数都可以拆成若干个 **2 的幂**之和（二进制分解）。类似地，任何「步数」也可以拆成若干个 2 的幂的跳跃。

**预处理**：$up[k][i]$ 表示从 $i$ 出发，跳 $2^k$ 步到达的位置。

**转移**：

$$
up[k][i] = up[k-1][\,up[k-1][i]\,]
$$

即「跳 $2^{k-1}$ 步再跳 $2^{k-1}$ 步」。

**查询**：把步数 $d$ 二进制分解，从高位到低位跳。

```cpp
for (int k = LOG; k >= 0; --k)
    if (d >> k & 1) u = up[k][u];
```

**复杂度**：预处理 $O(n \log n)$，查询 $O(\log n)$。

---

## 2. 倍增求 LCA

### 2.1 问题

给定一棵树，多次询问 $u, v$ 的最近公共祖先（LCA）。

### 2.2 算法

1. **预处理**：
   - DFS 求每个点的深度 `dep[u]` 和 $2^k$ 级祖先 `up[k][u]`；
2. **查询**：
   - 先把较深的点跳到和较浅的点同一深度；
   - 若此时两点相同，返回该点；
   - 否则从高位到低位同时跳，直到两点的父节点相同；
   - 返回 `up[0][u]`。

```cpp
int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);
    int d = dep[u] - dep[v];
    for (int k = 0; k < LOG; ++k)
        if (d >> k & 1) u = up[k][u];
    if (u == v) return u;
    for (int k = LOG - 1; k >= 0; --k)
        if (up[k][u] != up[k][v]) { u = up[k][u]; v = up[k][v]; }
    return up[0][u];
}
```

### 2.3 预处理

```cpp
void dfs(int u, int p) {
    dep[u] = dep[p] + 1;
    up[0][u] = p;
    for (int k = 1; k < LOG; ++k)
        up[k][u] = up[k-1][up[k-1][u]];
    for (int v : g[u])
        if (v != p) dfs(v, u);
}
```

---

## 3. 倍增的其他应用

### 3.1 快速幂

二进制分解指数，$O(\log n)$ 求 $a^n$。

### 3.2 ST 表（稀疏表）

预处理 $st[k][i]$ 表示从 $i$ 开始长度为 $2^k$ 的区间最值，查询 $O(1)$。

### 3.3 树上第 $k$ 个祖先

直接跳 $k$ 步，$O(\log n)$。

```cpp
int kthAncestor(int u, int k) {
    for (int i = 0; i < LOG; ++i)
        if (k >> i & 1) u = up[i][u];
    return u;
}
```

### 3.4 跳表 / 倍增 DP

某些 DP 的转移可以加速：$f[k][i]$ 表示从 $i$ 走 $2^k$ 步的最优值。

### 3.5 倍增求区间覆盖

「用最少的区间覆盖 $[1,n]$」：预处理每个位置跳 $2^k$ 次能到的最远处。

---

## 4. 模板代码

### 4.1 完整 LCA

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 500005;
const int LOG = 20;

int n, m;
vector<int> g[N];
int dep[N], up[LOG][N];

void dfs(int u, int p) {
    dep[u] = dep[p] + 1;
    up[0][u] = p;
    for (int k = 1; k < LOG; ++k)
        up[k][u] = up[k-1][up[k-1][u]];
    for (int v : g[u])
        if (v != p) dfs(v, u);
}

int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);
    int d = dep[u] - dep[v];
    for (int k = 0; k < LOG; ++k)
        if (d >> k & 1) u = up[k][u];
    if (u == v) return u;
    for (int k = LOG - 1; k >= 0; --k)
        if (up[k][u] != up[k][v]) { u = up[k][u]; v = up[k][v]; }
    return up[0][u];
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i < n; ++i) {
        int u, v;
        scanf("%d%d", &u, &v);
        g[u].push_back(v);
        g[v].push_back(u);
    }
    dfs(1, 0);   // 1 为根，0 为虚拟根
    while (m--) {
        int u, v;
        scanf("%d%d", &u, &v);
        printf("%d\n", lca(u, v));
    }
    return 0;
}
```

### 4.2 倍增求树上距离

$$
dist(u, v) = dep[u] + dep[v] - 2 \cdot dep[lca(u, v)]
$$

---

## 5. 经典例题

### 5.1 【模板】最近公共祖先

**题意**：$n, m \le 5 \times 10^5$，$m$ 次询问 LCA。

**复杂度**：$O((n+m) \log n)$。

### 5.2 树上距离

**题意**：多次询问树上两点距离。

**做法**：LCA + 深度。

### 5.3 树上第 k 个祖先

**题意**：询问某个点的第 $k$ 个祖先。

**做法**：二进制分解跳。

### 5.4 货车运输

**题意**：图上多次询问两点间路径最小边权的最大值。

**做法**：最大生成树 + 倍增维护路径最小值。

```cpp
int mn[LOG][N];
// 查询时同时取 min
```

### 5.5 区间覆盖最少次数

**题意**：$n$ 个区间覆盖 $[1, m]$，求最少区间数。

**做法**：预处理每个位置跳 $2^k$ 次的最远位置。

---

## 6. 易错点与练习

### 易错点

1. **`LOG` 开够**：$\log_2(5 \times 10^5) \approx 19$，开 20 或 21；
2. **根节点 `up[0][root] = root`**：否则会跳到 0 越界；
3. **深度差二进制分解从 0 到 LOG-1**，而 LCA 同时跳从 LOG-1 到 0；
4. **多组数据重置**：`dep`、`up` 数组要清；
5. **树的深度太大**：DFS 可能爆栈，用迭代或 BFS 预处理。

### 练习建议

1. 快速幂；
2. ST 表求区间最值；
3. 倍增求 LCA；
4. 树上第 $k$ 祖先；
5. 最大生成树 + 倍增（货车运输）；
6. 倍增 DP。
