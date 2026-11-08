# 最近公共祖先（LCA）

> 一句话概括：**最近公共祖先（Lowest Common Ancestor）** 是树上两个节点的所有公共祖先中深度最大的那个。它是树上路径、距离、子树统计等大量问题的基础工具。

---

## 目录

1. [概念与直观理解](#1-概念与直观理解)
2. [性质与定理](#2-性质与定理)
3. [暴力求法](#3-暴力求法)
4. [倍增 LCA](#4-倍增-lca)
5. [欧拉序 + ST 表](#5-欧拉序--st-表)
6. [Tarjan 离线 LCA](#6-tarjan-离线-lca)
7. [树剖 LCA](#7-树剖-lca)
8. [经典例题](#8-经典例题)
9. [易错点](#9-易错点)
10. [练习建议](#10-练习建议)

---

## 1. 概念与直观理解

### 1.1 定义

给定一棵有根树，根为 $r$。对任意两个节点 $u,v$，它们的**最近公共祖先** $\operatorname{LCA}(u,v)$ 是同时为 $u$ 和 $v$ 祖先的节点中，深度最大的那个。

约定：一个节点也是它自己的祖先。

### 1.2 文字示意图

```
               1
             / | \
            2  3  4
           / \    |
          5   6   7
             / \
            8   9
```

- $\operatorname{LCA}(8,9)=6$
- $\operatorname{LCA}(8,5)=2$
- $\operatorname{LCA}(7,9)=1$
- $\operatorname{LCA}(3,3)=3$
- $\operatorname{LCA}(4,6)=1$

### 1.3 为什么需要 LCA

LCA 是树上问题的「万能钥匙」：

- **树上两点距离**：$\operatorname{dist}(u,v)=\operatorname{dep}(u)+\operatorname{dep}(v)-2\operatorname{dep}(\operatorname{LCA}(u,v))$；
- **判断祖先关系**：$u$ 是 $v$ 的祖先 $\iff$ $\operatorname{LCA}(u,v)=u$；
- **树上差分**：路径 $(u,v)$ 上点权加 $1$，等价于 $d_u\!+\!+,\ d_v\!+\!+,\ d_{\operatorname{lca}}\!-\!-,\ d_{\operatorname{fa(lca)}}\!-\!-$；
- **虚树构建**：按 DFS 序排序后相邻取 LCA；
- **路径最大值/最小值**：配合倍增维护路径信息。

---

## 2. 性质与定理

### 2.1 基本性质

1. $\operatorname{LCA}(u,v)=\operatorname{LCA}(v,u)$（对称性）；
2. $\operatorname{LCA}(u,u)=u$（幂等性）；
3. 若 $u$ 是 $v$ 的祖先，则 $\operatorname{LCA}(u,v)=u$；
4. $\operatorname{LCA}(\operatorname{LCA}(u,v),w)=\operatorname{LCA}(u,\operatorname{LCA}(v,w))$（结合律）；
5. $\operatorname{dep}(\operatorname{LCA}(u,v))\le \min(\operatorname{dep}(u),\operatorname{dep}(v))$。

### 2.2 重要定理：欧拉序性质

设对树做 DFS，**每进入一个节点记录一次、每次从儿子返回时再记录一次父节点**，得到长度为 $2n-1$ 的序列称为**欧拉序**，记 $\operatorname{tour}[1..2n-1]$。设 $\operatorname{first}(u)$ 为节点 $u$ 在欧拉序中第一次出现的位置。

**定理**：对任意两个节点 $u,v$（不妨设 $\operatorname{first}(u)<\operatorname{first}(v)$），有

$$
\operatorname{LCA}(u,v)=\arg\min_{\operatorname{first}(u)\le i\le \operatorname{first}(v)}\operatorname{dep}\bigl(\operatorname{tour}[i]\bigr)
$$

即：**欧拉序中从 $u$ 第一次出现到 $v$ 第一次出现这一段里，深度最小的节点就是它们的 LCA**。

这个性质是「欧拉序 + ST 表」求 LCA 的基础。

> 注意：单靠 DFS 入栈时间戳（`dfn`）的区间最小值并不能直接得到 LCA，必须使用欧拉序（含回溯记录）。DFS 序常用于「子树对应连续区间」，而欧拉序用于「路径 LCA 对应区间」，两者用途不同。

### 2.3 树上距离公式

$$
\operatorname{dist}(u,v)=\operatorname{dep}(u)+\operatorname{dep}(v)-2\operatorname{dep}(\operatorname{LCA}(u,v))
$$

根的深度记为 $0$ 或 $1$ 均可（只要全树统一；本文模板取 $1$），这是 LCA 最常用的推论。

---

## 3. 暴力求法

最朴素的做法：先把 $u$、$v$ 中较深的那个往上跳到与另一个同深度，然后两个一起往上跳直到相遇。单次询问 $O(n)$，$q$ 次询问 $O(nq)$，只能应对 $n,q\le 10^3$ 的小数据。

```cpp
int lca_brute(int u, int v) {
    while (dep[u] > dep[v]) u = fa[u];
    while (dep[v] > dep[u]) v = fa[v];
    while (u != v) { u = fa[u]; v = fa[v]; }
    return u;
}
```

暴力法的价值在于：理解「先对齐深度，再一起上跳」这一核心思想，所有优化版本都是在这个思想上加速。

---

## 4. 倍增 LCA

### 4.1 核心思想

预处理 $f[u][k]$ 表示节点 $u$ 向上跳 $2^k$ 步到达的祖先：

$$
f[u][0]=\operatorname{fa}(u),\qquad f[u][k]=f\bigl[f[u][k-1]\bigr][k-1]
$$

查询时：

1. 把较深的点跳到与较浅点同深度（二进制拆分深度差）；
2. 若此时 $u=v$，答案是 $u$；
3. 从大到小枚举 $k$：若 $f[u][k]\ne f[v][k]$，则 $u\gets f[u][k]$，$v\gets f[v][k]$；
4. 最后 $\operatorname{LCA}=\operatorname{fa}(u)$。

### 4.2 正确性说明

第 3 步中「从大到小尝试跳」的直觉：如果两个点跳 $2^k$ 步后仍不同，说明 LCA 在更上面，可以安全地跳；最终两个点会停在 LCA 的两个不同儿子上，再跳一步就是 LCA。

### 4.3 文字示意

沿用 1.2 节的树，查询 $\operatorname{LCA}(8,9)$（`dep[8]=dep[9]=4`，无需对齐深度）：

```
k=1: f[8][1]=2, f[9][1]=2，相同，不跳
k=0: f[8][0]=6, f[9][0]=6，相同，不跳
循环结束，返回 f[8][0] = 6      （正确答案）
```

再查询 $\operatorname{LCA}(8,5)$：8 的深度为 4、5 的深度为 3，先把 8 向上跳 1 步到 6（`k=0` 时 `dep[8]-1=3>=dep[5]`）；此时 `u=6, v=5` 深度相同但不同点，`f[6][0]=f[5][0]=2` 相同不跳，循环结束返回 `f[6][0]=2`（正确答案）。

> 注意「相同就不跳」：`f[u][k] == f[v][k]` 说明跳 $2^k$ 步后会到达公共祖先，可能已经越过 LCA，所以不能跳；只有不同时才安全地跳。

### 4.4 完整模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 500010;
const int LOG = 20;          // 2^19 = 524288 > 5e5

vector<int> g[MAXN];
int f[MAXN][LOG], dep[MAXN];
int n, m, root;

void dfs(int u, int p) {
    f[u][0] = p;
    dep[u] = dep[p] + 1;     // 根的 dep 设为 1，则 dep[0]=0
    for (int k = 1; k < LOG; ++k) f[u][k] = f[f[u][k - 1]][k - 1];
    for (int v : g[u]) if (v != p) dfs(v, u);
}

int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);
    for (int k = LOG - 1; k >= 0; --k)
        if (dep[u] - (1 << k) >= dep[v]) u = f[u][k];
    if (u == v) return u;
    for (int k = LOG - 1; k >= 0; --k)
        if (f[u][k] != f[v][k]) { u = f[u][k]; v = f[v][k]; }
    return f[u][0];
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m >> root;
    for (int i = 1; i < n; ++i) {
        int u, v; cin >> u >> v;
        g[u].push_back(v); g[v].push_back(u);
    }
    dfs(root, 0);
    while (m--) {
        int u, v; cin >> u >> v;
        cout << lca(u, v) << '\n';
    }
    return 0;
}
```

### 4.5 复杂度

预处理 $O(n\log n)$ 时间、$O(n\log n)$ 空间；单次查询 $O(\log n)$；支持在线查询，是最常用的 LCA 实现。

倍增还能维护路径上的聚合信息（最大值、最小值、和等）：预处理时同步合并 `mx[u][k] = max(mx[u][k-1], mx[f[u][k-1]][k-1])`，查询时与 `lca` 结构完全一致，只是每次移动顺带 `res = max(res, mx[...])`。

```cpp
int mx[MAXN][LOG];                 // 从 u 向上 2^k 条边路径上的最大值

void dfs2(int u, int p) {
    f[u][0] = p; dep[u] = dep[p] + 1; mx[u][0] = w[u];
    for (int k = 1; k < LOG; ++k) {
        f[u][k] = f[f[u][k - 1]][k - 1];
        mx[u][k] = max(mx[u][k - 1], mx[f[u][k - 1]][k - 1]);
    }
    for (int v : g[u]) if (v != p) dfs2(v, u);
}
```

---

## 5. 欧拉序 + ST 表

### 5.1 欧拉序定义

对树做 DFS，**每进入一个节点记录一次，每次从儿子返回时再记录一次父节点**，得到的长度为 $2n-1$ 的序列称为欧拉序（Euler tour）。

```
        1
       / \
      2   3
     / \
    4   5

欧拉序：1 2 4 2 5 2 1 3 1
```

### 5.2 关键性质

对任意 $u,v$，设它们在欧拉序中第一次出现的位置分别为 $l,r$（$l<r$），则：

$$
\operatorname{LCA}(u,v) = \text{区间 } [l,r] \text{ 中深度最小的节点}
$$

因此问题转化为**静态区间最小值查询（RMQ）**，用 ST 表 $O(1)$ 回答。

### 5.3 代码

```cpp
const int MAXN = 500010;
vector<int> g[MAXN];
int dep[MAXN], dfn[MAXN], st[20][MAXN << 1];  // st 存节点编号
int lg[MAXN << 1];
int timer = 0, cnt = 0;

void dfs(int u, int p) {
    dep[u] = dep[p] + 1;
    dfn[u] = ++timer;       // 第一次出现的位置
    st[0][++cnt] = u;       // 欧拉序第 cnt 个元素
    for (int v : g[u]) {
        if (v == p) continue;
        dfs(v, u);
        st[0][++cnt] = u;   // 回溯时再记一次
    }
}

int better(int u, int v) { return dep[u] < dep[v] ? u : v; }

void buildST() {
    for (int i = 2; i <= cnt; ++i) lg[i] = lg[i >> 1] + 1;
    for (int k = 1; (1 << k) <= cnt; ++k)
        for (int i = 1; i + (1 << k) - 1 <= cnt; ++i)
            st[k][i] = better(st[k - 1][i], st[k - 1][i + (1 << (k - 1))]);
}

int lca(int u, int v) {
    int l = dfn[u], r = dfn[v];
    if (l > r) swap(l, r);
    int k = lg[r - l + 1];
    return better(st[k][l], st[k][r - (1 << k) + 1]);
}
```

### 5.4 复杂度

- 预处理：$O(n\log n)$；单次查询：$O(1)$；
- 不支持动态加点，适合静态树 + 大量查询的场景。

---

## 6. Tarjan 离线 LCA

### 6.1 思想

把所有询问离线，做一次 DFS，用**并查集**维护「当前节点所在的、已回溯完毕的子树集合」：

1. DFS 进入 $u$，标记 `vis[u] = true`；
2. 递归处理每个儿子 $v$，回溯后 `merge(u, v)`（把 $v$ 的集合合并到 $u$）；
3. 处理所有与 $u$ 相关的询问 $(u,v)$：若 `vis[v]` 为真，则 $\operatorname{LCA}(u,v)=\operatorname{find}(v)$。

### 6.2 正确性直觉

`vis[v]` 为真说明 $v$ 的 DFS 已完成。此时 $v$ 所在集合的代表是「$v$ 的祖先中，那些儿子已经全部处理完的最深节点」，恰好就是 $\operatorname{LCA}(u,v)$（因为 $u$ 正在被处理，说明 $u$ 是当前 DFS 栈上的一员）。

### 6.3 代码

```cpp
const int MAXN = 500010;
vector<int> g[MAXN];
vector<pair<int,int>> qry[MAXN];   // (另一个点, 询问编号)
int fa[MAXN], ans[MAXN];
bool vis[MAXN];

int find(int x) { return fa[x] == x ? x : fa[x] = find(fa[x]); }

void tarjan(int u) {
    vis[u] = true;
    fa[u] = u;
    for (int v : g[u]) {
        if (vis[v]) continue;
        tarjan(v);
        fa[v] = u;                 // 回溯后把子树接到 u
    }
    for (auto &q : qry[u]) {
        int v = q.first, id = q.second;
        if (vis[v]) ans[id] = find(v);
    }
}
```

### 6.4 复杂度

$O(n\alpha(n)+q)$，近似线性；但只能离线处理。

---

## 7. 树剖 LCA

树链剖分（见「树链剖分」一章）也可以 $O(\log n)$ 求 LCA，且常数比倍增小：

```cpp
int lca(int u, int v) {
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        u = fa[top[u]];
    }
    return dep[u] < dep[v] ? u : v;
}
```

原理：每次把所在重链头较深的那个点跳到链头的父节点，直到两点在同一条重链上，深度小的就是 LCA。

---

## 8. 经典例题

### 例题 1：【模板】最近公共祖先（Luogu P3379）

**题目大意**：给定一棵 $n$ 个点的有根树，$m$ 次询问 $u,v$ 的最近公共祖先。$n,m\le 5\times 10^5$。

**思路**：倍增 LCA 模板，预处理 $O(n\log n)$，每次 $O(\log n)$。也可以用欧拉序 + ST 表做到 $O(1)$ 查询。

**复杂度**：$O(n\log n + m\log n)$。

---

### 例题 2：树上距离（Luogu P1351 联合权值 / 距离类问题）

**题目大意**：给定一棵 $n$ 个点的树，$m$ 次询问两点间距离，边权为 $1$。

**思路**：直接套用距离公式

$$
\operatorname{dist}(u,v)=\operatorname{dep}(u)+\operatorname{dep}(v)-2\operatorname{dep}(\operatorname{LCA}(u,v))
$$

**复杂度**：预处理 $O(n\log n)$，每次 $O(\log n)$。

```cpp
int dist(int u, int v) {
    return dep[u] + dep[v] - 2 * dep[lca(u, v)];
}
```

---

### 例题 3：货车运输（Luogu P1967 [NOIP2013]）

**题目大意**：$n$ 个城市、$m$ 条道路，每条道路有限重。$q$ 次询问，每次问从 $u$ 到 $v$ 的路径上最小边权的最大值。

**思路**：这是典型的「最大瓶颈路径」问题：

1. 原图可能不连通，先求**最大生成树**（按边权从大到小 Kruskal）；
2. 在最大生成树上，任意两点路径的最小边权就是答案；
3. 用倍增 LCA 同时维护「向上 $2^k$ 步路径上的最小边权」。

查询时一边跳一边取最小值。注意不连通时输出 $-1$。

**复杂度**：$O(m\log m + (n+q)\log n)$。

---

### 例题 4：紧急疏散（Luogu P1600 [NOIP2016] 天天爱跑步）

**题目大意**：一棵 $n$ 个点的树，第 $i$ 个点有一个观察员，会在时刻 $w_i$ 观察。有 $m$ 个人从 $s_i$ 跑到 $t_i$，每秒跑一条边。问每个观察员能看到多少人。

**思路**：经典的「树上差分 + LCA」难题。

把每个人的路径拆成 $s\to \operatorname{lca}$ 与 $\operatorname{lca}\to t$ 两段：

- 上行段：在点 $u$ 被看到当且仅当 $w_u = \operatorname{dep}(s)-\operatorname{dep}(u)$，即 $\operatorname{dep}(s)=w_u+\operatorname{dep}(u)$；
- 下行段：在点 $u$ 被看到当且仅当 $w_u = \operatorname{dep}(s)-2\operatorname{dep}(\operatorname{lca})+\operatorname{dep}(u)$。

用两个桶分别统计，配合树上差分在 DFS 时维护桶，注意 LCA 处会被重复计算需要减去。复杂度 $O((n+m)\log n)$。

**复杂度**：$O(n\log n + m\log n)$。

---

## 9. 易错点

1. **根节点的父节点**：`f[root][0] = 0`，同时 `dep[0] = 0`，否则对齐深度时会越界。
2. **LOG 开小**：$n\le 5\times10^5$ 时 `LOG` 至少 20；保险起见写 `LOG = 20` 或用 `__lg(n)+1`。
3. **倍增跳深度时写错**：`dep[u] - (1<<k) >= dep[v]`，不要写成 `dep[f[u][k]] >= dep[v]`（当 $f[u][k]=0$ 时 `dep[0]=0` 可能造成误判）。
4. **多组数据不清空**：`g`、`f`、`dep` 都要清空。
5. **递归爆栈**：$n=5\times10^5$ 时 DFS 递归可能爆栈，建议手写栈或使用 BFS 预处理。
6. **欧拉序长度**：欧拉序长度是 $2n-1$，ST 表要开到 `2*n`。
7. **Tarjan 离线**：并查集初始化和 `vis` 标记的时机容易写错；询问要存双向。
8. **树剖 LCA 的 `dep[top[u]]`**：比较的是**链头**的深度，不是当前点的深度。
9. **倍增维护路径信息**：合并顺序与边界（是否包含 LCA 本身）要按题目要求仔细处理。

---

## 10. 练习建议

由易到难：

1. **模板**：Luogu P3379 【模板】最近公共祖先 —— 掌握倍增写法；
2. **距离应用**：Luogu P1351 联合权值 —— 树形 DP + LCA 思想；
3. **瓶颈路**：Luogu P1967 货车运输 —— 最大生成树 + 倍增维护最小值；
4. **树上差分**：Luogu P3128 [USACO15DEC] Max Flow P —— 路径加、单点查询；
5. **欧拉序 + ST**：Luogu P3379 的另一种写法，或 SPOJ QTREE2；
6. **Tarjan 离线**：Luogu P3379 用 Tarjan 实现，体会离线算法；
7. **进阶**：Luogu P1600 天天爱跑步 —— 树上差分 + LCA 综合；
8. **动态树**：Luogu P4211 [LNOI2014] LCA —— 树剖 + 离线询问，适合学完树剖后挑战。

学习建议：

- 务必先手动画图模拟倍增的跳法，理解「为什么最后停在 LCA 的两个儿子上」；
- 建议把倍增、欧拉序 ST、Tarjan、树剖四种写法都实现一遍，体会不同场景下的取舍；
- LCA 本身不难，难的是「看出题目需要 LCA」以及与之配合的差分、倍增信息维护。
