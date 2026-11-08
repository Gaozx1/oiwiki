# 基环树（环套树）

> 一句话概括：**基环树**是「恰好含有一个环」的连通图，形如在一棵树上加一条边。它兼具树与环的性质，核心技巧是「找环 + 断环 + 环上处理」。

---

## 目录

1. [概念与性质](#1-概念与性质)
2. [基环树的分类与识别](#2-基环树的分类与识别)
3. [找环的三种方法](#3-找环的三种方法)
4. [基环树 DP 的基本框架](#4-基环树-dp-的基本框架)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 概念与性质

### 1.1 定义

**基环树（pseudoforest / unicyclic graph）**：一个连通的无向图，若其边数等于点数，即 $m=n$，则它恰好含有一个环。

更一般地，一个**基环树森林**是若干棵基环树的并。

### 1.2 为什么是「树上加一条边」

一棵树有 $n-1$ 条边。若在树上任意加一条边，必然形成恰好一个环，得到 $n$ 个点、$n$ 条边，这就是基环树；反过来，基环树删除环上任意一条边就得到一棵树。

```
            1                 挂树 T1        挂树 T2
           / \                  |              |
          2   3            a1 --+-- a2 --------+
         /     \            \       \
        4 ----- 5             \       a3 ---- a4
                              \             /
环：1-2-4-5-3-1                 +-----------+
                             环：a1 - a2 - a3 - a4 - a1
```

左图是最小的基环树（5 个点全在环上）；右图是更一般的情形——环上每个点还可以额外连出一棵（或多棵）树，树上的点不在环上。

### 1.3 关键性质

1. **边数等于点数**：$m=n$（连通情形）；
2. **环唯一**：恰有一个简单环；
3. **内向基环树**：若每个点恰好有一条出边（函数图 / functional graph），则每个连通分量恰有一个环，其余点沿出边走最终都进入该环；
4. **删环上一边得树**：这是最常用的转化手段；
5. **任意两点路径**：要么不经过环，要么经过环的某一段（两种走法取最优）。

### 1.4 与树、仙人掌的区别

| 结构 | 特征 |
|---|---|
| 树 | 无环，$m=n-1$ |
| 基环树 | 恰一个环，$m=n$ |
| 仙人掌 | 每条边至多属于一个环，可有多个环 |

---

## 2. 基环树的分类与识别

### 2.1 无向基环树

- $n$ 个点、$n$ 条边的连通无向图；
- 用并查集加边时，若某条边的两端已经在同一集合，则这条边就是「环上的边」（多余边）；
- 或 DFS 时记录父边，遇到已访问的点即发现环。

### 2.2 内向基环树（每个点出度为 1）

**典型场景**：每个点 $i$ 有一条出边指向 $f(i)$。这种图叫「内向基环树森林」（也叫函数图 / functional graph）。

- 每个连通分量恰好有一个环；
- 环上的点沿出边走会绕圈；
- 其他点沿出边走最终进入环。

**识别**：拓扑排序删去入度为 0 的点，剩下的点就是所有环上的点。

### 2.3 外向基环树（每个点入度为 1）

对称情形，把边反向即可转化为内向基环树。

---

## 3. 找环的三种方法

### 3.1 拓扑排序法（适用于内向基环树）

```cpp
// 每个点出度为 1，求所有环上的点
queue<int> q;
for (int i = 1; i <= n; ++i)
    if (inDeg[i] == 0) q.push(i);
while (!q.empty()) {
    int u = q.front(); q.pop();
    int v = to[u];              // u 的唯一出边
    if (--inDeg[v] == 0) q.push(v);
}
// 结束后 inDeg[i] > 0 的点就是环上的点
```

### 3.2 DFS 时间戳法（通用）

```cpp
int dfn[MAXN], fa[MAXN], timer = 0;
bool onRing[MAXN];

void dfs(int u, int p) {
    dfn[u] = ++timer;
    for (int v : g[u]) {
        if (v == p) continue;          // 无向图跳过父边（重边需用边编号）
        if (!dfn[v]) { fa[v] = u; dfs(v, u); }
        else if (dfn[v] < dfn[u]) {    // 发现一条回边，沿 fa 回溯标记整个环
            onRing[v] = true;
            for (int x = u; x != v; x = fa[x]) onRing[x] = true;
        }
    }
}
```

### 3.3 并查集法（无向图）

```cpp
// 加边时若两端已连通，则该边是环上的边
for (int i = 1; i <= m; ++i) {
    int u = eu[i], v = ev[i];
    if (find(u) == find(v)) { ringFlag = i; continue; }
    merge(u, v);
    g[u].push_back(v); g[v].push_back(u);
}
// 断掉第 ringFlag 条边，剩下的图是一棵树
```

---

## 4. 基环树 DP 的基本框架

### 4.1 通用套路：断环成树

以「基环树上的最大独立集」为例：

1. **找环**：确定环上所有点；
2. **断边**：任选环上一条边 $(x,y)$ 断开，得到一棵树；
3. **两次树形 DP**：
   - 强制 $x$ 不选，做一次树形 DP；
   - 强制 $x$ 选，则 $y$ 必须不选，再做一次树形 DP；
4. 两次取最大值。

### 4.2 为什么需要两次

断边后，$(x,y)$ 这条边的约束丢失了。原问题要求 $(x,y)$ 不能同时被选，所以分两类讨论：

- $x$ 不选：$y$ 任意，正常 DP；
- $x$ 选：$y$ 必须不选，把 $y$ 的状态强制为不选。

### 4.3 内向基环树上的 DP：先树后环

对于「每个点出度为 1」的基环树，先对每棵挂到环上的子树做树形 DP，把结果汇总到对应的环点，再在环上做一次「环形 DP」（通常把环复制一倍，配合单调队列 / 断环成链）。

### 4.4 文字示意

```
              T1      T2
               |       |
      a1 —— a2 —— a3 —— a4
       \                 /
        +---- a5 ------+

环：a1 - a2 - a3 - a4 - a5 - a1；T1、T2 是挂在 a2、a3 上的子树（可有多层）

步骤：① 找出 a1..a5 构成的环；② 对挂到每个环点上的子树做树形 DP，把结果汇总到对应环点；
      ③ 在环上做 DP（断环成链，枚举起点或分类讨论）。
```

---

## 5. 模板代码

### 5.1 无向基环树：找环 + 断边 + 树形 DP

下面是「基环树最大独立集」的完整代码（可编译；对每个连通分量做一次，本题保证连通）。读入为 `n`、$n$ 个点权、$n$ 条边（0-indexed 端点）。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100010;
const long long NEG = -1e18;      // 「不合法」状态的标记

int n;
int a[MAXN];                      // 点权
vector<pair<int,int>> g[MAXN];    // (邻点, 边编号)

int dfn[MAXN], fa[MAXN], timer;
int ringU, ringV;                 // 被断开的环边 (ringU, ringV)
bool found = false;
long long f[MAXN][2];             // f[u][0/1]：u 不选 / 选 时的最大权值和

// ---------- 找环：DFS 遇到「已访问且不是父节点」的点，即发现一条回边 ----------
void findRing(int u, int pEdge) {
    dfn[u] = ++timer;
    for (auto &pr : g[u]) {
        int v = pr.first, id = pr.second;
        if (id == pEdge) continue;           // 用边编号跳过父边（可正确处理重边）
        if (!dfn[v]) {
            fa[v] = u;
            findRing(v, id);
            if (found) return;
        } else if (dfn[v] < dfn[u]) {         // 只从「后访问」的一端记录一次
            ringU = u;
            ringV = v;
            found = true;
            return;
        }
    }
}

// ---------- 树形 DP：ban 节点强制不选 ----------
void treeDP(int u, int p, int ban) {
    f[u][0] = 0;
    f[u][1] = (u == ban ? NEG : (long long)a[u]);
    for (auto &pr : g[u]) {
        int v = pr.first;
        if (v == p) continue;
        // 跳过被断开的环边
        if ((u == ringU && v == ringV) || (u == ringV && v == ringU)) continue;
        treeDP(v, u, ban);
        f[u][0] += max(f[v][0], f[v][1]);
        f[u][1] += f[v][0];
        if (f[u][1] < NEG / 2) f[u][1] = NEG;   // 防止负无穷累加溢出
        if (f[u][0] < NEG / 2) f[u][0] = NEG;
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n;
    for (int i = 1; i <= n; ++i) cin >> a[i];
    for (int i = 1; i <= n; ++i) {
        int u, v;
        cin >> u >> v;
        ++u; ++v;                          // 题目 0-indexed，转 1-indexed
        g[u].push_back({v, i});
        g[v].push_back({u, i});
    }

    findRing(1, 0);

    long long ans = 0;

    // 情况 1：ringU 不选（ringV 任意）
    treeDP(ringU, 0, ringU);
    ans = max(ans, f[ringU][0]);           // f[ringU][1] 必为 NEG

    // 情况 2：ringV 不选（此时允许 ringU 选）
    treeDP(ringU, 0, ringV);
    ans = max(ans, max(f[ringU][0], f[ringU][1]));

    cout << ans << '\n';
    return 0;
}
```

> 两种情况合起来恰好覆盖所有合法方案：任一合法方案不可能同时选 `ringU` 与 `ringV`，因此它要么属于「ringU 不选」，要么属于「ringV 不选」。
>
> 若题目是基环树森林，则对所有 `dfn[i]==0` 的点分别调用 `findRing` 并做上述两次 DP，把各连通分量的答案累加（或按题目要求取 max）。

### 5.2 内向基环树：拓扑找环 + 每个点到环的距离

「内向基环树」（每个点出度为 $1$）最常见的需求是：判断哪些点在环上、每个点沿出边走多少步进入环。下面是一份完整可编译的程序，输入 $n$ 和每个点的出边 `to[i]`，输出每个点所属环的长度与到环的距离。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100010;

int n;
int to[MAXN], inDeg[MAXN];
int ringId[MAXN];        // 所属环编号（0 表示还没算）
int ringLen[MAXN];       // 该环的长度
int distToRing[MAXN];    // 沿出边走多少步进入环
vector<int> ringNodes[MAXN];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> to[i];
        ++inDeg[to[i]];
    }

    // 1. 拓扑排序，剥掉所有不在环上的点
    queue<int> q;
    for (int i = 1; i <= n; ++i)
        if (inDeg[i] == 0) q.push(i);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        int v = to[u];
        if (--inDeg[v] == 0) q.push(v);
    }
    // 结束后 inDeg[i] > 0 的点就是环上的点

    // 2. 给每个环编号并记录环长（沿出边走一圈）
    int cnt = 0;
    for (int i = 1; i <= n; ++i) {
        if (inDeg[i] > 0 && ringId[i] == 0) {
            ++cnt;
            int u = i, len = 0;
            do {
                ringId[u] = cnt;
                ringNodes[cnt].push_back(u);
                u = to[u];
                ++len;
            } while (u != i);
            for (int x : ringNodes[cnt]) ringLen[cnt] = len;
        }
    }

    // 3. 反图 BFS：从所有环点出发，向外扩散求距离
    vector<vector<int>> rev(n + 1);
    for (int i = 1; i <= n; ++i)
        if (ringId[i] == 0) rev[to[i]].push_back(i);
    queue<int> bfs;
    for (int i = 1; i <= n; ++i) {
        if (ringId[i] > 0) { distToRing[i] = 0; bfs.push(i); }
    }
    while (!bfs.empty()) {
        int u = bfs.front(); bfs.pop();
        for (int v : rev[u]) {
            ringId[v] = ringId[u];              // 继承所在环
            distToRing[v] = distToRing[u] + 1;
            bfs.push(v);
        }
    }

    for (int i = 1; i <= n; ++i)
        cout << ringLen[ringId[i]] << ' ' << distToRing[i] << '\n';
    return 0;
}
```

复杂度 $O(n)$。核心技巧是**反图 BFS**：从所有环点同时出发，沿反边扩散，第一次到达某个点时的距离就是它到环的最短距离（因为它只有一条出边，反图上就是唯一的入边路径）。

### 5.3 复杂度

找环 $O(n)$，反图 BFS 或树形 DP 均为 $O(n)$；总复杂度 $O(n)$。若环上需要做单调队列优化（如基环树直径），仍为 $O(n)$。

---

## 6. 经典例题

### 例题 1：城市环路（Luogu P1453）

**题目大意**：一个城市有 $n$ 个点、$n$ 条边，构成基环树。每个点有一个权值，要求选出一个点集，使得没有相邻的点同时被选，求最大权值和。

**思路**：基环树最大独立集：

1. 找环上的一条边 $(u,v)$；
2. 断掉它，得到一棵树；
3. 分两种情况：$u$ 不选 / $u$ 选（此时 $v$ 必须不选）；
4. 两次树形 DP 取最大值。

**复杂度**：$O(n)$。

---

### 例题 2：骑士（Luogu P2607 [ZJOI2008]）

**题目大意**：有 $n$ 个骑士，每个骑士有一个战斗力 $a_i$，且有一个「最讨厌的人」$b_i$。要求选出一些骑士，使得任意两个骑士不互相讨厌，且总战斗力最大。

**思路**：把「$i$ 讨厌 $b_i$」看成一条无向边 $i-b_i$，每个点恰好连一条边（可能自环），形成**基环树森林**。

对每棵基环树：

- 找环、断边；
- 分两种情况（断边的一端选/不选）做树形 DP；
- 求和。

注意自环的处理：若 $i$ 讨厌自己，则 $i$ 不能选。

**复杂度**：$O(n)$。

---

### 例题 3：岛屿（Luogu P4381 [IOI2008] Island）

**题目大意**：给定一个基环树森林，每条边有长度，求所有连通分量的直径之和。

**思路**：基环树直径 = max(不经过环的路径, 经过环的路径)：

1. 对每棵挂在环上的子树求直径 $d_1$；
2. 对环上点 $i$ 求「从 $i$ 出发向子树走的最远距离」$f_i$；
3. 经过环的路径：在环上枚举两点 $i<j$，答案为
   $$
   f_i+f_j+\min(\operatorname{dist}(i,j),\ L-\operatorname{dist}(i,j))
   $$
   其中 $L$ 为环长。把环复制一倍，用单调队列维护 $f_i-\operatorname{pre}_i$ 的最大值，$O(\text{环长})$ 解决。

**复杂度**：$O(n)$。

---

### 例题 4：旅行（Luogu P5022 [NOIP2018]）

**题目大意**：给定一棵树或基环树（$m=n-1$ 或 $m=n$），从 1 号点出发，每次只能走向未访问的相邻点，求字典序最小的 DFS 序。

**思路**：若 $m=n-1$，直接 DFS，每次选编号最小的未访问儿子；若 $m=n$，枚举环上每一条边断开，对得到的树做「字典序最小 DFS」，取所有方案中字典序最小者。

**复杂度**：$O(n^2)$（$n\le 5000$），可以通过本题。

---

## 7. 易错点

1. **混淆边数与点数**：基环树是 $m=n$，树是 $m=n-1$，读入后先判断。
2. **自环**：内向基环树中可能出现 $i\to i$ 的自环，找环和 DP 都要特殊处理。
3. **重边**：无向图找环时用「父节点」跳过父边，遇到重边可能误判，建议用边编号。
4. **断边后忘了恢复**：树形 DP 前跳过断边，DP 后要清理状态再跑第二次。
5. **两次 DP 的状态污染**：第二次 DP 前要重新初始化 `f` 数组。
6. **多棵基环树**：题目可能是基环树森林，要对每个连通分量分别处理再汇总。
7. **不连通**：注意 `findRing` 只从一个点出发，需要对所有未访问点调用。
8. **环的顺序**：环上 DP 需要按环的实际顺序处理，找环后要整理出顺序数组。

---

## 8. 练习建议

由易到难：

1. **入门**：Luogu P1453 城市环路 —— 基环树最大独立集模板；
2. **基环树森林**：Luogu P2607 [ZJOI2008] 骑士 —— 多分量 + 自环处理；
3. **断环成树**：Luogu P5022 旅行 —— 枚举断边 + 贪心；
4. **基环树直径**：Luogu P4381 [IOI2008] Island —— 单调队列优化；
5. **内向基环树**：Luogu P2921 [USACO08DEC] Trick or Treat on the Farm —— 拓扑找环；
6. **环上 DP**：Luogu P1541 乌龟棋（不是基环树，但环形 DP 思想类似）；
7. **进阶**：Luogu P3533 [POI2012] RAN-Rendezvous —— 内向基环树 LCA；
8. **综合**：Luogu P3574 [POI2014] FAR-FarmCraft —— 基环树 + 树形 DP。

学习建议：

- 先掌握「找环」的三种写法，建议都能手写；
- 理解「断环成树后为什么要分两类讨论」；
- 基环树的题目通常不难在算法，难在细节（自环、森林、环序），写完务必对拍。
