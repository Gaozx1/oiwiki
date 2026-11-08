# 网络流

> 一句话概括：网络流研究「在有容量的有向网络上最多能输送多少流量」，最大流等于最小割（最大流最小割定理），几乎所有「分配/匹配/选取」类问题都能建模成它。

网络流是图论中建模能力最强的工具：最大流、最小割、上下界流、费用流、最大权闭合子图……本节从流网络的定义出发，严谨推出增广路与残量网络的概念，给出 Dinic 模板，并用若干经典建模展示它的用法。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 流网络与可行流
   - 1.2 残量网络与增广路
   - 1.3 最大流最小割定理
   - 1.4 Dinic 算法
   - 1.5 常见建模
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 流网络与可行流

**流网络** $G=(V,E)$ 是一个有向图，每条边 $(u,v)$ 有**容量** $c(u,v)\ge 0$。规定两个特殊点：源点 $S$、汇点 $T$。

**流（flow）** 是定义在边上的函数 $f(u,v)$，满足：

1. **容量限制**：$0\le f(u,v)\le c(u,v)$；
2. **流量守恒**：除 $S,T$ 外，每个点流入量 = 流出量：

$$
\sum_{(u,v)\in E}f(u,v)=\sum_{(v,w)\in E}f(v,w).
$$

流的总量定义为源点的净流出（等于汇点的净流入）：

$$
|f|=\sum_{(S,v)\in E}f(S,v)-\sum_{(v,S)\in E}f(v,S).
$$

**最大流**：使 $|f|$ 最大的可行流。

```
      2/3      1/1
 S --------- A ------ T
  \          |       /
   \ 1/2     | 2/2  /
    \        B  ---/
     \_______/ \____/

每条边标注 流量/容量
```

### 1.2 残量网络与增广路

**残量（residual capacity）**：$c_f(u,v)=c(u,v)-f(u,v)$，即这条边还能再压多少流。

**反向边**：当 $f(u,v)>0$ 时，可以「撤销」这条边上的流量，因此在残量网络中增加反向边 $(v,u)$，容量为 $f(u,v)$。

**残量网络** $G_f$：由所有 $c_f(u,v)>0$ 的边构成。

**增广路**：残量网络 $G_f$ 中从 $S$ 到 $T$ 的一条路径。沿增广路把流增加 $\delta=\min_{(u,v)\in P}c_f(u,v)$，得到更大的流。

**核心结论（增广路定理）**：可行流 $f$ 是最大流 $\iff$ 残量网络 $G_f$ 中不存在 $S\to T$ 的路径。

**为什么必须建反向边？** 因为贪心地沿任意增广路增广可能做出「错误决定」，反向边给了算法「反悔」的机会。经典反例：

```
S → A (容量 1), S → B (容量 1), A → T (容量 1), B → T (容量 1), A → B (容量 1)

若先增广 S→A→B→T（占用了 A→B），则 S→B 和 A→T 无法同时满足
但有了反向边 B→A，可以撤销 A→B 上的流，改走 S→B→T，得到完整流 2
```

### 1.3 最大流最小割定理

**割（cut）**：把点集分成 $S\in X$、$T\in \bar X$ 两部分，割的**容量**为所有从 $X$ 指向 $\bar X$ 的边容量之和：

$$
c(X,\bar X)=\sum_{u\in X,v\notin X}c(u,v).
$$

**最小割**：容量最小的割。

**定理（Max-Flow Min-Cut）**：

$$
\max_f |f|=\min_{(X,\bar X)}c(X,\bar X).
$$

即最大流的值等于最小割的容量。

**证明思路**：

- 对任意流 $f$ 和割 $(X,\bar X)$，有 $|f|\le c(X,\bar X)$（流必须跨过割，且不能超过容量）；
- 取最大流 $f^*$，令 $X$ 为 $G_{f^*}$ 中从 $S$ 可达的点集，则 $T\notin X$（否则存在增广路），且 $X$ 到 $\bar X$ 的所有边都满流、反向边流量为 0，故 $|f^*|=c(X,\bar X)$。

**推论**：最大流算法结束时，若在残量网络上从 $S$ 做一次 BFS，能到达的点集 $X$ 与不能到达的点集 $\bar X$ 就给出了一个**最小割方案**。

### 1.4 Dinic 算法

**核心思想**：BFS 分层 + DFS 多路增广。

1. **BFS 分层**：在残量网络上 BFS，求出每个点到 $S$ 的距离 `dep[v]`。若 $T$ 不可达，算法结束。
2. **DFS 增广**：沿 `dep[v] == dep[u] + 1` 的边（分层图）做 DFS，一次 DFS 可以找到多条增广路（多路增广），并记录当前弧 `cur[u]` 避免重复扫描。
3. 重复 BFS 直到 $T$ 不可达。

**当前弧优化**：`cur[u]` 记录点 $u$ 下一次应该从哪条边开始尝试，避免每条边在同一分层图中被反复扫描。

**复杂度**：

- 一般图：$O(n^2m)$；
- 单位容量图（如二分图匹配）：$O(m\sqrt n)$；
- 实际运行远快于理论界。

### 1.5 常见建模

| 问题 | 建模方式 |
|---|---|
| 二分图最大匹配 | $S\to$ 左部（容量 1），左部 $\to$ 右部（容量 1），右部 $\to T$（容量 1） |
| 二分图多重匹配 | 左部到右部边容量为 1，$S\to$ 左部容量为 $k$ |
| 最小点覆盖 | 求最小割，割集对应点覆盖 |
| 最大权闭合子图 | 正权点连 $S$，负权点连 $T$，答案 = 正权和 − 最小割 |
| 有源汇上下界可行流 | 加边 $T\to S$ 容量 $+\infty$，转化无源汇 |
| 最大密度子图 | 二分答案 + 最小割 |
| 方格取数 | 黑白染色 + 最小割 |

**建模的关键**：先想清楚「什么是流量」「什么容量对应题目中的限制」，再用最小割回答「最少舍弃多少」。

---

## 2. 模板代码

### 2.1 Dinic 最大流（链式前向星 + 当前弧）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 10005;
const int MAXM = 200005;
const long long INF = 1e18;

struct Edge {
    int to, nxt;
    long long cap;
} e[MAXM * 2];

int head[MAXN], tot = 1;       // 从 1 开始，i ^ 1 为反向边
int dep[MAXN], cur[MAXN];
int n, m, S, T;

void addEdge(int u, int v, long long c) {
    e[++tot] = {v, head[u], c};
    head[u] = tot;
    e[++tot] = {u, head[v], 0};   // 反向边容量为 0
    head[v] = tot;
}

// BFS 分层：求残量网络上各点到 S 的距离
bool bfs() {
    memset(dep, -1, sizeof(dep));
    queue<int> q;
    dep[S] = 0;
    q.push(S);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (dep[v] == -1 && e[i].cap > 0) {
                dep[v] = dep[u] + 1;
                q.push(v);
            }
        }
    }
    return dep[T] != -1;
}

// DFS 多路增广：只走分层图（dep[v] == dep[u] + 1）
long long dfs(int u, long long limit) {
    if (u == T) return limit;
    long long flow = 0;
    for (int &i = cur[u]; i && flow < limit; i = e[i].nxt) {
        int v = e[i].to;
        if (e[i].cap > 0 && dep[v] == dep[u] + 1) {
            long long f = dfs(v, min(e[i].cap, limit - flow));
            if (!f) { dep[v] = -1; continue; }   // 剪枝：v 无法增广
            e[i].cap -= f;
            e[i ^ 1].cap += f;                    // 反向边增加
            flow += f;
        }
    }
    return flow;
}

long long dinic() {
    long long maxFlow = 0;
    while (bfs()) {
        memcpy(cur, head, sizeof(head));          // 当前弧初始化
        long long f;
        while ((f = dfs(S, INF)) > 0) maxFlow += f;
    }
    return maxFlow;
}

int main() {
    scanf("%d%d%d%d", &n, &m, &S, &T);
    for (int i = 1; i <= m; ++i) {
        int u, v;
        long long c;
        scanf("%d%d%lld", &u, &v, &c);
        addEdge(u, v, c);
    }
    printf("%lld\n", dinic());
    return 0;
}
```

### 2.2 最小割方案输出

```cpp
// 最大流结束后，在残量网络上从 S 做 BFS，能到达的点属于 S 侧
vector<int> minCut(int n) {
    vector<int> side(n + 1, 0);     // side[v]=1 表示从 S 在残量网络上可达
    queue<int> q;
    q.push(S);
    side[S] = 1;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (e[i].cap > 0 && !side[v]) {
                side[v] = 1;
                q.push(v);
            }
        }
    }
    return side;
}
// 割边：所有满足 side[u] == 1 && side[v] == 0 的原边 (u,v)
```

### 2.3 无源汇上下界可行流

```cpp
// 每条边 (u,v) 有下界 low、上界 high
// 建图：addEdge(u, v, high - low)，并记录 d[u] -= low, d[v] += low
// 建立超级源 SS 和超级汇 TT
// 对每个点：若 d[i] > 0 则 addEdge(SS, i, d[i])
//           若 d[i] < 0 则 addEdge(i, TT, -d[i])
// 跑 Dinic，若 SS 的所有出边都满流则有可行流
bool feasibleFlow(int n) {
    long long total = 0;
    for (int i = 1; i <= n; ++i) {
        if (d[i] > 0) { addEdge(SS, i, d[i]); total += d[i]; }
        else if (d[i] < 0) addEdge(i, TT, -d[i]);
    }
    return dinic(SS, TT) == total;
}
```

### 2.4 有源汇上下界最大流

先加边 `addEdge(T, S, INF)` 把有源汇问题转化为无源汇可行流；判定可行后，再在残量网络上从 $S$ 到 $T$ 跑一次 Dinic。**答案 = 可行流流量 + 第二次增广的流量**。

---

## 3. 经典例题

### 例题 1：最大流模板（洛谷 P3376）

**题意**：给定 $n$ 个点 $m$ 条边的有向图，求 $S$ 到 $T$ 的最大流（数据范围以题面为准，容量总和较大）。

**思路**：直接套 Dinic 模板。注意容量总和可能很大，用 `long long`。

```cpp
// 完整代码见 2.1 节
printf("%lld\n", dinic());
```

**复杂度**：$O(n^2m)$ 最坏，实际运行很快。

### 例题 2：方格取数（洛谷 P2774）

**题意**：一个 $n\times m$ 的网格，每个格子有正整数。从中取数，要求任意两个取出的格子不能相邻（四连通），求最大取数和。

**思路**：黑白染色后是二分图，问题转化为**最大权独立集**。对二分图有：

$$
\text{最大权独立集} = \text{总权值} - \text{最小点权覆盖}.
$$

把点权覆盖转成最小割：$S\to$ 黑点（容量为点权），白点 $\to T$（容量为点权），相邻黑白点连容量 $+\infty$ 的边。最小割即最小点权覆盖，答案 = 总权 − 最小割。

```cpp
// 建图：
for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= m; ++j) {
        int id = (i - 1) * m + j;
        if ((i + j) % 2 == 0) {
            addEdge(S, id, a[i][j]);
            for (int d = 0; d < 4; ++d) {
                int ni = i + dx[d], nj = j + dy[d];
                if (ni >= 1 && ni <= n && nj >= 1 && nj <= m)
                    addEdge(id, (ni - 1) * m + nj, INF);
            }
        } else addEdge(id, T, a[i][j]);
    }
printf("%lld\n", sum - dinic());
```

**复杂度**：$O(V^2E)$，$V=nm$。

### 例题 3：飞行员配对方案问题（洛谷 P2756）

**题意**：二分图匹配问题，用网络流做。

**思路**：$S\to$ 左部容量 1，左部 $\to$ 右部容量 1，右部 $\to T$ 容量 1，最大流即最大匹配。比匈牙利更通用，可扩展为多重匹配。

**复杂度**：$O(m\sqrt n)$（单位容量图）。

### 例题 4：P1345 [USACO5.4] 奶牛的电信 Telecowmunication

**题意**：给定 $n$ 个点 $m$ 条边的无向图，求最少删掉几个点，使 $S$ 与 $T$ 不连通（$S,T$ 不能删）。

**思路**：**拆点**。把每个点 $u$ 拆成 $u_{in}$ 和 $u_{out}$，中间连容量 1 的边（表示删掉这个点）；原边 $(u,v)$ 连 $u_{out}\to v_{in}$ 和 $v_{out}\to u_{in}$，容量 $+\infty$（表示不能通过删边断开）。$S_{in}$ 为源，$T_{out}$ 为汇，最小割即最小点割集。

```cpp
// 点 u 对应 in=2*u-1, out=2*u
for (int i = 1; i <= n; ++i) {
    if (i == S || i == T) addEdge(2*i-1, 2*i, INF);   // 源汇点不能删
    else addEdge(2*i-1, 2*i, 1);                       // 删掉该点的代价为 1
}
for (auto &pr : edges) {                               // 原无向边，容量无穷
    int u = pr.first, v = pr.second;
    addEdge(2*u, 2*v-1, INF);
    addEdge(2*v, 2*u-1, INF);
}
printf("%lld\n", dinic());
```

**复杂度**：$O(n^2m)$。

---

## 4. 易错点与练习建议

### 易错点

1. **不建反向边**：这是最致命的错误，没有反向边就无法「反悔」，贪心会得到非最大流。
2. **`tot` 初值不是 1**：链式前向星成对加边时，`tot` 从 1 开始才能保证 `i ^ 1` 配对。
3. **`cap` 用 `int` 溢出**：容量累加可达 $10^{18}$，务必 `long long`。
4. **BFS 分层时忘记判 `cap > 0`**：残留容量为 0 的边不能走。
5. **DFS 忘记当前弧**：`for (int &i = cur[u]; ...)` 是当前弧优化的关键，写成 `for (int i = head[u]; ...)` 会 TLE。
6. **DFS 剪枝写错**：`if (!f) { dep[v] = -1; continue; }` 可以避免重复访问死路，但不要误把 `dep[v]` 改成 0。
7. **多组数据不重置 `tot`**：`tot = 1` 和 `head` 数组都要清空。
8. **源汇点编号**：$S,T$ 是题目给的还是自己加的（如超级源点 $n+1$）要分清。
9. **最小割方案**：在**残量网络**上从 $S$ BFS，走 `cap > 0` 的边，能到达的点集才是 $S$ 侧。
10. **无向图建边**：无向边要建两条**有向边**（各自带反向边），即调用两次 `addEdge(u,v,c)` 和 `addEdge(v,u,c)`，注意不是 `addEdge(u,v,c)` 加反向边容量 $c$。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P3376 网络最大流 | Dinic 模板 |
| 入门 | 洛谷 P2756 飞行员配对方案 | 二分图匹配转最大流 |
| 入门 | 洛谷 P1345 奶牛的电信 | 拆点最小割 |
| 提高 | 洛谷 P2774 方格取数 | 黑白染色 + 最小割 |
| 提高 | 洛谷 P1361 小M的作物 | 最小割建模 |
| 提高 | 洛谷 P2762 太空飞行计划问题 | 最大权闭合子图 |
| 提高 | 洛谷 P2764 最小路径覆盖问题 | 拆点 + 最大流 |
| 进阶 | 洛谷 P2766 最长不下降子序列问题 | 分层图 + 最大流 |
| 进阶 | 洛谷 P4174 [NOI2006] 最大获利 | 最大权闭合子图 |

学习建议：先默写 Dinic 模板（BFS + DFS + 当前弧），再理解「最小割 = 最大流」的证明。网络流的难点从来不在代码，而在**建模**：看到「最大收益」「最小代价」「选取限制」，先想想能不能转成最小割。建议从「方格取数」「最大权闭合子图」这两个经典模型入手。
