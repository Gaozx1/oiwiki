# 最小生成树

> 一句话概括：在连通无向带权图中，选出 $n-1$ 条边把所有点连起来且总权值最小，这样的边集就是**最小生成树（MST）**。

最小生成树是最经典的「全局最优」图论模型：铺路、架线、电网、聚类都直接对应它。本节讲清 MST 的两大核心性质（环性质与割性质），推导 Kruskal 与 Prim 两种算法的正确性，并补充瓶颈生成树、次小生成树等常见变形。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 生成树的定义
   - 1.2 两个核心性质
   - 1.3 Kruskal 算法
   - 1.4 Prim 算法
   - 1.5 常见变形
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 生成树的定义

给定连通无向图 $G=(V,E)$，$|V|=n$。若子图 $T=(V,E')$ 满足：

- $E'$ 包含 $n-1$ 条边；
- $T$ 连通（等价地：无环）；

则称 $T$ 是 $G$ 的一棵**生成树（Spanning Tree）**。生成树恰好用最少的边把所有点连起来。

若边带权，令 $w(T)=\sum_{e\in E'}w(e)$，使 $w(T)$ 最小的生成树即**最小生成树（Minimum Spanning Tree, MST）**。

```
原图（数字为边权）：                  一棵 MST（选 1-2、2-4、3-4，总权 6）：

   1 ---1--- 2                        1 ---1--- 2
   | \       |                        |         |
   4   2     2                        |         2
   |     \   |                        |         |
   3 ---3--- 4                        3 ---3--- 4
```

左边原图共有 5 条边：$1\text{-}2$ 权 1、$2\text{-}4$ 权 2、$3\text{-}4$ 权 3、$1\text{-}3$ 权 4、$1\text{-}4$ 权 2。右边是其中一棵 MST，代价为 $1+2+3=6$，恰好 $n-1=3$ 条边。

**基本事实**：

- MST 不一定唯一（当存在等权边时可能有多棵），但**最小总权值唯一**。
- 只有连通图才有生成树；不连通时求的是**最小生成森林**（每个连通块各求一棵 MST）。
- MST 的边权和是 $n-1$ 条边权值的最小可能和。

### 1.2 两个核心性质

MST 的所有算法都建立在下面两条性质上。

#### 割性质（Cut Property）

把点集 $V$ 任意分成两个非空集合 $S$ 与 $V\setminus S$，跨越这两个集合的边称为**割边**。则**权值最小的割边一定属于某棵 MST**。

直观理解：要把 $S$ 和外面连通，至少得用一条割边；用最小的那条永远不会吃亏。

#### 环性质（Cycle Property）

对图中任意一个环，**环上权值最大的边一定不属于任何 MST**（若最大边不唯一，则至少有一条最大边不在某棵 MST 中）。

直观理解：环上删掉最大的一条边，剩下的点仍然连通，总权值只会变小。

```
环性质示意：
     1
   /   \
  2     3
   \   /
     9        ← 9 是环上最大边，任何 MST 都不会选它
```

#### 推论

由上面两条性质可以得到一个非常重要的结论：**任意时刻，若已选边集 $A$ 是某棵 MST 的子集，则从图中选一条不与 $A$ 成环的最小边加入 $A$，$A$ 仍是某棵 MST 的子集**。Kruskal 与 Prim 都是这条推论的不同实现方式。

### 1.3 Kruskal 算法

**思路（加边法）**：把所有边按权值升序排序，依次尝试加入每条边；若加入后不形成环（即两个端点当前不在同一连通块），就选它；否则跳过。选够 $n-1$ 条边即结束。

用**并查集**维护连通性：`find(u) != find(v)` 说明可以加边，加完后 `union(u,v)`。

**流程示意**：

```
边按权排序: (1,2,1) (2,4,2) (1,3,4) (3,4,3) (1,4,2)
① 加 (1,2,1)  → {1,2}
② 加 (2,4,2)  → {1,2,4}
③ 加 (3,4,3)  → {1,2,3,4}，已选 3 = n-1 条，结束
```

**正确性**：每次选的边都是某个割的最小割边（割 = 当前连通块与外界），由割性质它属于某棵 MST，归纳即得。

**复杂度**：排序 $O(m\log m)$，并查集近似 $O(m\alpha(n))$，总

$$
T=O(m\log m),\qquad S=O(n+m).
$$

Kruskal 的优点是实现简单、能处理稀疏图、天然支持「只加部分边」的变形；缺点是需要对边排序，且不易做在线加点的处理。

### 1.4 Prim 算法

**思路（加点法）**：从任意点 $s$ 开始，维护集合 $S$（已在树中的点）。每次找一条「一端在 $S$、一端不在 $S$」的最小边，把外面的点加入 $S$，直到所有点都加入。

**流程示意**：

```
从 1 出发: S={1}
可选边: (1,2,1) (1,3,4) (1,4,2) → 选 (1,2,1), S={1,2}
可选边: (2,4,2) (1,3,4) (1,4,2) → 选 (2,4,2), S={1,2,4}
可选边: (4,3,3) (1,3,4)        → 选 (4,3,3), S={1,2,3,4}
```

**正确性**：每次选的边就是割 $(S, V\setminus S)$ 的最小割边，由割性质成立。

**复杂度**：暴力实现 $O(n^2)$，适合稠密图；用堆优化后 $O(m\log n)$，适合稀疏图。**在完全图上 Prim 的 $O(n^2)$ 比 Kruskal 的 $O(n^2\log n)$ 更优**。

### 1.5 常见变形

| 变形 | 做法 |
|---|---|
| 最小生成森林 | 对每个连通块分别跑 Kruskal/Prim，统计边数是否达到 $n-\text{块数}$ |
| 最大生成树 | 边权取相反数，或把排序改成降序 |
| 瓶颈生成树 | 使「最大边权」最小的生成树；**MST 一定是一棵瓶颈生成树** |
| 次小生成树 | 先求 MST，再枚举非树边 $(u,v,w)$，用树上 $u\to v$ 路径的最大边替换 |
| 最小度限制生成树 | 先求 MST，若某点度超限则做调整（较复杂，可暂缓） |
| 最小生成树计数 | 按权值分组，用 Matrix-Tree 定理分别计数后相乘 |

---

## 2. 模板代码

### 2.1 Kruskal（并查集实现）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const int MAXN = 200005;
const int MAXM = 500005;

struct Edge {
    int u, v, w;
    bool operator<(const Edge &o) const { return w < o.w; }
} e[MAXM];

int fa[MAXN];

int find(int x) {
    // 路径压缩
    return fa[x] == x ? x : fa[x] = find(fa[x]);
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; ++i) scanf("%d%d%d", &e[i].u, &e[i].v, &e[i].w);
    for (int i = 1; i <= n; ++i) fa[i] = i;
    sort(e + 1, e + m + 1);           // 按边权升序

    ll total = 0;
    int used = 0;
    for (int i = 1; i <= m; ++i) {
        int ru = find(e[i].u), rv = find(e[i].v);
        if (ru == rv) continue;        // 成环，跳过
        fa[ru] = rv;                   // 合并
        total += e[i].w;
        ++used;
        if (used == n - 1) break;      // 已选够 n-1 条边
    }
    if (used < n - 1) printf("orz\n"); // 图不连通
    else printf("%lld\n", total);
    return 0;
}
```

### 2.2 Prim（堆优化版）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const int MAXN = 5005;
const int INF = 0x3f3f3f3f;

struct Edge { int to, w; };
vector<Edge> g[MAXN];
bool inTree[MAXN];
int minEdge[MAXN];            // minEdge[v]：v 到当前树集合的最短边

ll prim(int n, int s) {
    fill(minEdge, minEdge + n + 1, INF);
    minEdge[s] = 0;
    ll total = 0;
    for (int iter = 1; iter <= n; ++iter) {
        int u = -1;
        // 暴力找不在树中且 minEdge 最小的点
        for (int v = 1; v <= n; ++v)
            if (!inTree[v] && (u == -1 || minEdge[v] < minEdge[u])) u = v;
        if (u == -1 || minEdge[u] == INF) return -1;   // 不连通
        inTree[u] = true;
        total += minEdge[u];
        for (auto &e : g[u])
            if (!inTree[e.to] && e.w < minEdge[e.to])
                minEdge[e.to] = e.w;
    }
    return total;
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; ++i) {
        int u, v, w; scanf("%d%d%d", &u, &v, &w);
        g[u].push_back({v, w});
        g[v].push_back({u, w});
    }
    ll ans = prim(n, 1);
    if (ans == -1) printf("orz\n");
    else printf("%lld\n", ans);
    return 0;
}
```

### 2.3 堆优化 Prim（稀疏图版）

```cpp
ll primHeap(int n, int s) {
    fill(minEdge, minEdge + n + 1, INF);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    minEdge[s] = 0;
    pq.push({0, s});
    ll total = 0;
    int cnt = 0;
    while (!pq.empty() && cnt < n) {
        int d = pq.top().first, u = pq.top().second;
        pq.pop();
        if (inTree[u]) continue;
        inTree[u] = true;              // 取出即加入树
        total += d;
        ++cnt;
        for (auto &e : g[u])
            if (!inTree[e.to] && e.w < minEdge[e.to]) {
                minEdge[e.to] = e.w;
                pq.push({e.w, e.to});
            }
    }
    return cnt == n ? total : -1;
}
```

### 2.4 次小生成树（非严格）

```cpp
// 先跑 Kruskal 得到 MST，同时建出这棵树；预处理树上两点路径的最大边权
// maxOnPath[u][v] 可用 O(n^2) DFS 或倍增 LCA 得到
// 枚举每条非树边 (u,v,w)，候选值 = mst + w - maxOnPath[u][v]
// 取所有候选值中的最小值即次小生成树（非严格）

vector<int> tree[MAXN];        // MST 的邻接表
int wOnTree[MAXN][MAXN];       // 树上边的权值
int maxOnPath[MAXN][MAXN];

void dfsMax(int root, int u, int fa, int curMax) {
    maxOnPath[root][u] = curMax;
    for (int v : tree[u]) {
        if (v == fa) continue;
        dfsMax(root, v, u, max(curMax, wOnTree[u][v]));
    }
}
```

---

## 3. 经典例题

### 例题 1：最小生成树（洛谷 P3366 模板题）

**题意**：给出 $n$ 个点 $m$ 条边的无向图，若连通求 MST 边权和，否则输出 `orz`。$n\le 5000$，$m\le 2\times10^5$。

**思路**：直接套 Kruskal 模板（2.1 节）。注意判断最终选边数是否为 $n-1$，不足说明图不连通。

**复杂度**：$O(m\log m)$。

### 例题 2：局域网（洛谷 P2820）

**题意**：有 $n$ 台电脑和 $m$ 条网线，每条网线有连接费用。要求删掉一些网线，使剩下的网线把电脑连成一个连通块且总费用最小，求**删掉的网线费用之和**。

**思路**：剩下的网线就是 MST，删掉的费用 = 总费用 − MST 的总权值。

```cpp
long long sumAll = 0, mst = 0;
int used = 0;
for (int i = 1; i <= m; ++i) sumAll += e[i].w;
sort(e + 1, e + m + 1);
for (int i = 1; i <= m && used < n - 1; ++i) {
    int ru = find(e[i].u), rv = find(e[i].v);
    if (ru == rv) continue;
    fa[ru] = rv;
    mst += e[i].w;
    ++used;
}
printf("%lld\n", sumAll - mst);       // 删掉的边权和 = 总和 - MST
```

**复杂度**：$O(m\log m)$。

### 例题 3：繁忙的都市（洛谷 P2330 / SCOI2005）

**题意**：城市有 $n$ 个路口 $m$ 条道路，选一些道路把城市连通，要求**边数最少**且**最大边权最小**，输出边数和最大边权。

**思路**：边数一定是 $n-1$（树）。最大边权最小 = 瓶颈生成树问题，而 **MST 一定是瓶颈生成树**。因此直接求 MST，答案为 $n-1$ 和 MST 中的最大边权。

```cpp
int maxW = 0;
for (int i = 1; i <= m; ++i) {
    int ru = find(e[i].u), rv = find(e[i].v);
    if (ru == rv) continue;
    fa[ru] = rv;
    maxW = max(maxW, e[i].w);      // Kruskal 升序选边，最后选的边最大
    if (++used == n - 1) break;
}
printf("%d %d\n", n - 1, maxW);
```

**复杂度**：$O(m\log m)$。

### 例题 4：新的开始（洛谷 P1550 / USACO08OCT）

**题意**：$n$ 块田地需要灌溉，可以直接打井（在第 $i$ 块地打井费用 $w_i$），也可以从别的田地修水渠（费用 $p_{ij}$）。求让所有田地都有水的最小总费用。

**思路**：经典「虚拟源点」建模。新建一个超级源点 0，从 0 向每个点 $i$ 连一条权为 $w_i$ 的边，表示「在 $i$ 打井」。原图中田地之间的边保留。则答案为这张 $n+1$ 点图的最小生成树。

```cpp
// 建图：点 0 为超级源点
for (int i = 1; i <= n; ++i) {
    int w; scanf("%d", &w);
    e[++m] = {0, i, w};
}
// 读入 p[i][j]（只读上三角，避免重边）
for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= n; ++j) {
        int p; scanf("%d", &p);
        if (i < j) e[++m] = {i, j, p};
    }
// 对 n+1 个点跑 Kruskal，选够 n 条边
```

**复杂度**：$O(m\log m)$。

---

## 4. 易错点与练习建议

### 易错点

1. **图不连通仍输出总权**：Kruskal 结束时要检查 `used == n-1`；Prim 要检查是否所有点都加入了树。
2. **并查集忘记初始化**：`fa[i] = i` 必须在读边后、Kruskal 前做。
3. **重边处理**：Kruskal 天然允许重边（排序后取小的即可），无需特判；但邻接矩阵存图时要取 `min`。
4. **边权溢出**：MST 总权可能达到 $10^{14}$，`total` 要用 `long long`。
5. **Kruskal 排序从 0 开始**：若边数组从 1 开始存，排序要写 `sort(e + 1, e + m + 1)`，写成 `e + m` 会漏掉最后一条边。
6. **Prim 初始化 `minEdge[s] = 0`**：不是 `INF`，否则第一个点无法加入。
7. **Prim 的 `inTree` 判断时机**：堆优化版应在**出堆时**判 `inTree` 并标记，入堆时标记会导致错误。
8. **MST 唯一性问题**：有等权边时 MST 可能不唯一，但题目问总权值时答案唯一；若问方案数要另做计数。
9. **次小生成树求成了非严格**：若题目要求严格次小（权值和严格大于 MST），当替换边与原边等权时需要换次大边，处理更复杂。
10. **虚拟源点模型**：加了超级源点后，树的边数是 $(n+1)-1=n$，不是 $n-1$。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P3366 最小生成树 | Kruskal/Prim 模板 |
| 入门 | 洛谷 P2820 局域网 | MST 与总费用差 |
| 入门 | 洛谷 P2330 繁忙的都市 | 瓶颈生成树 |
| 提高 | 洛谷 P1550 新的开始 | 虚拟源点建模 |
| 提高 | 洛谷 P1194 买礼物 | 特殊建图 + MST |
| 提高 | 洛谷 P1396 营救 | 瓶颈路 + 并查集/二分 |
| 提高 | 洛谷 P4180 严格次小生成树 | 倍增 + 次小生成树 |
| 进阶 | 洛谷 P4208 最小生成树计数 | 矩阵树定理 |
| 进阶 | 洛谷 P5633 最小度限制生成树 | 度限制 MST |

学习建议：Kruskal 与 Prim 至少各手写三遍，做到不看书能写对。遇到「最小代价连通」「最大边最小」「删边最多」等字眼，第一反应应是 MST。虚拟源点是非常高频的建模技巧，务必掌握。
