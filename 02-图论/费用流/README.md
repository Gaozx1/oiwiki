# 费用流（最小费用最大流）

> 一句话概括：当每条边除了容量之外还带有「单位费用」时，我们希望**在流量最大的前提下让总费用最小**，这就是最小费用最大流（Min-Cost Max-Flow，简称 MCMF / 费用流）。

---

## 目录

1. [问题模型](#1-问题模型)
2. [核心概念与原理](#2-核心概念与原理)
3. [SSP 算法：逐次最短增广](#3-ssp-算法逐次最短增广)
4. [模板代码（SPFA 版）](#4-模板代码spfa-版)
5. [势能 Dijkstra 优化](#5-势能-dijkstra-优化)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 问题模型

给定一个有向网络 $G=(V,E)$，每条边 $(u,v)$ 有两个参数：

- 容量 $c(u,v)$：这条边最多能流过的流量；
- 单位费用 $w(u,v)$：每流过 $1$ 单位流量需要付出的代价。

一个可行流 $f$ 需要满足：

$$
0 \le f(u,v) \le c(u,v), \qquad
\sum_{v} f(v,u) - \sum_{v} f(u,v) = 0 \quad (u \ne s,t)
$$

设总流量为 $|f|=\sum_v f(s,v)$，总费用为：

$$
\operatorname{cost}(f)=\sum_{(u,v)\in E} f(u,v)\cdot w(u,v)
$$

**最小费用最大流**要求：先让 $|f|$ 达到最大流 $F$，再在流量为 $F$ 的所有可行流中使 $\operatorname{cost}(f)$ 最小。

例如下面这个网络（边上标注为「容量, 费用」）：

```
        (2, 1)          (2, 1)
  S ----------> A -------------> T
  |                               ^
  | (1, 3)          (1, 3)        |
  +----------> B -----------------+
```

- 最大流：$S\to A\to T$ 可以送 $2$ 单位，$S\to B\to T$ 可以送 $1$ 单位，共 $3$；
- 若全走最短路，费用为 $2\times(1+1)+1\times(3+3)=10$。

> 与最大流的区别：最大流只关心「能送多少」，费用流在「能送多少」的基础上继续关心「送得贵不贵」。因此费用流问题的答案通常是一对 $(F, C)$：最大流量与对应的最小费用。

---

## 2. 核心概念与原理

### 2.1 残量网络

设当前流为 $f$，定义残量网络 $G_f$：

- 对每条边 $(u,v)$，若 $f(u,v)<c(u,v)$，保留一条**正向边** $u\to v$，容量为 $c(u,v)-f(u,v)$，费用为 $w(u,v)$；
- 若 $f(u,v)>0$，加入一条**反向边** $v\to u$，容量为 $f(u,v)$，费用为 $-w(u,v)$。

反向边的负费用表示「退流」：把一单位流量从 $v$ 退回 $u$，相当于把之前付出的 $w(u,v)$ 收回来。

```
正向边 u --(cap, w)--> v
残量网络中同时存在：
    u --(cap - f, w)--> v      （还能继续流）
    v --(f, -w)------> u       （可以退流）
```

### 2.2 增广路的费用

沿着残量网络中的一条 $s\to t$ 路径增广 $\delta$ 单位流量，总费用变化为：

$$
\Delta \operatorname{cost} = \delta \cdot \sum_{(u,v)\in P} w(u,v)
$$

因此「单位费用最小的增广路」就是残量网络中 $s\to t$ 的**最短路**（以费用为边权）。

### 2.3 最优性定理

**定理**：流量为 $f$ 的可行流是最小费用流，当且仅当残量网络 $G_f$ 中**不存在负费用环**。

- 若存在负环，沿着负环增广一圈，流量不变而费用降低，说明当前流不最优；
- 反之若不存在负环，可以用残量网络上的势函数 / 线性规划对偶证明当前流已最优。

### 2.4 SSP 的核心思想

SSP（Successive Shortest Path，逐次最短增广）算法从零流出发：

1. 在残量网络中找一条从 $s$ 到 $t$ 的**费用最短路**；
2. 沿这条路径尽可能多地增广；
3. 重复直到 $s$ 与 $t$ 不连通。

可以证明：若初始网络不存在负环（零流显然没有负环），则每次沿最短路增广后得到的流，都是其流量下的最小费用流。直观理解：

- 最短路是当前「最便宜的增广方式」；
- 增广后新增的反向边不会破坏最优性，因为若产生负环，可以沿负环退流改进，与「最短路」矛盾。

---

## 3. SSP 算法：逐次最短增广

### 3.1 算法流程

```
1. 建图：每条原边加 (cap, cost)，同时加入 (0, -cost) 的反向边
2. 循环：
   a. 在残量网络中求 s -> t 的最短路（按费用）
   b. 若 t 不可达，结束，当前 flow/cost 即答案
   c. 记最短路费用为 d[t]，沿路径找最小残量 add
   d. 沿路径增广 add，flow += add，cost += add * d[t]
```

### 3.2 复杂度

- 用 SPFA 求最短路：单次 $O(nm)$，总复杂度 $O(F \cdot nm)$，其中 $F$ 为最大流量；
- 用势能 + Dijkstra：单次 $O(m\log n)$，总复杂度 $O(F \cdot m\log n)$；
- 若用 ZKW 费用流 / 原始对偶（Primal-Dual）等实现，可进一步优化常数与理论界。

对于绝大多数 OI 题目，图的点数与边数不大、流量总和有限，SPFA 版 SSP 已经足够。

### 3.3 一个手算例子

还是第 1 节的图，初始最短路为 $S\to A\to T$，费用 $2$，增广 $2$ 单位，花费 $4$。

此时 $S\to A$ 与 $A\to T$ 均满流，残量网络中只能走 $S\to B\to T$，费用 $6$，增广 $1$ 单位，花费 $6$。

总流量 $3$，总费用 $10$，结束。

---

## 4. 模板代码（SPFA 版）

下面这份代码可直接编译运行。输入格式为：

```
n m s t
u1 v1 cap1 cost1
...
um vm capm costm
```

输出为「最大流 最小费用」。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

const int MAXN = 5010;      // 点数上限
const int MAXM = 200010;    // 原边数上限（e 数组长度取 MAXM<<1）
const ll INF = (ll)1e18;

struct Edge {
    int to, nxt;
    ll cap, cost;
} e[MAXM << 1];

int head[MAXN], ecnt = 1;   // ecnt 从 1 开始，反向边编号 = i ^ 1

void addEdge(int u, int v, ll cap, ll cost) {
    e[++ecnt] = {v, head[u], cap, cost};
    head[u] = ecnt;
    e[++ecnt] = {u, head[v], 0, -cost};
    head[v] = ecnt;
}

// SPFA 求残量网络中 s -> t 的费用最短路
bool spfa(int s, int t, int n, vector<ll>& dis, vector<int>& pre) {
    vector<bool> inq(n + 1, false);
    fill(dis.begin(), dis.end(), INF);
    fill(pre.begin(), pre.end(), 0);
    dis[s] = 0;
    queue<int> q;
    q.push(s);
    inq[s] = true;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        inq[u] = false;
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (e[i].cap > 0 && dis[v] > dis[u] + e[i].cost) {
                dis[v] = dis[u] + e[i].cost;
                pre[v] = i;             // 记录到达 v 的边，方便回溯
                if (!inq[v]) {
                    inq[v] = true;
                    q.push(v);
                }
            }
        }
    }
    return dis[t] < INF;
}

pair<ll, ll> minCostMaxFlow(int s, int t, int n) {
    ll flow = 0, cost = 0;
    vector<ll> dis(n + 1);
    vector<int> pre(n + 1);
    while (spfa(s, t, n, dis, pre)) {
        // 1. 找瓶颈
        ll add = INF;
        for (int v = t; v != s; v = e[pre[v] ^ 1].to) {
            add = min(add, e[pre[v]].cap);
        }
        // 2. 沿路径修改残量
        for (int v = t; v != s; v = e[pre[v] ^ 1].to) {
            e[pre[v]].cap -= add;
            e[pre[v] ^ 1].cap += add;
        }
        // 3. 累计答案
        flow += add;
        cost += add * dis[t];
    }
    return make_pair(flow, cost);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m, s, t;
    cin >> n >> m >> s >> t;
    for (int i = 1; i <= m; i++) {
        int u, v;
        ll cap, cost;
        cin >> u >> v >> cap >> cost;
        addEdge(u, v, cap, cost);
    }
    pair<ll, ll> ans = minCostMaxFlow(s, t, n);
    cout << ans.first << ' ' << ans.second << '\n';
    return 0;
}
```

> 注意：`MAXM` 是所有**原边**数量的上限，但 `e` 数组会存放两倍的反向边，所以 `e` 要开到 `MAXM << 1`。例如原图最多 $10^5$ 条边，则 `e` 至少需要 $2\times10^5$ 个元素。

---

## 5. 势能 Dijkstra 优化

SPFA 在特殊构造的图上会被卡，比较稳妥的做法是使用**势能函数（Johnson 技巧）**把边权变为非负，再用 Dijkstra。

设上一轮的最短路数组为 $h[u]$（初始时用 SPFA 求一次），定义**约简费用**：

$$
w'(u,v) = w(u,v) + h[u] - h[v]
$$

根据最短路的三角不等式 $h[v] \le h[u] + w(u,v)$，有 $w'(u,v)\ge 0$，于是可以用 Dijkstra。

```cpp
// 假设 dis 为上一轮的最短路，h 为势能
// 每轮结束后执行：h[v] += dis[v]
void dijkstra(int s, int t, int n, vector<ll>& h, vector<ll>& dis, vector<int>& pre) {
    priority_queue<pair<ll,int>, vector<pair<ll,int>>, greater<pair<ll,int>>> pq;
    fill(dis.begin(), dis.end(), INF);
    fill(pre.begin(), pre.end(), 0);
    dis[s] = 0;
    pq.push(make_pair(0LL, s));
    while (!pq.empty()) {
        pair<ll,int> cur = pq.top();
        pq.pop();
        ll d = cur.first;
        int u = cur.second;
        if (d != dis[u]) continue;
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (e[i].cap <= 0) continue;
            ll nd = d + e[i].cost + h[u] - h[v];   // 约简费用
            if (nd < dis[v]) {
                dis[v] = nd;
                pre[v] = i;
                pq.push(make_pair(nd, v));
            }
        }
    }
}
```

使用要点：

- 第一次必须先用 SPFA（或 Bellman-Ford）求出初始 $h$，因为原图可能有负费用边；
- 每轮增广后令 $h[v] \mathrel{+}= dis[v]$，这样下一轮的约简边权仍然非负；
- 最终的「真实最短路费用」等于 $h[t]$（或 $dis[t] + h_{\text{old}}[t] - h_{\text{old}}[s]$），累计费用时不要直接用约简后的 $dis[t]$。

---

## 6. 经典例题

### 例题 1：【模板】最小费用最大流

**题目大意**：给定 $n$ 个点、$m$ 条有向边，每条边有容量 $c_i$ 和单位费用 $w_i$，求从 $s$ 到 $t$ 的最大流，以及在最大流前提下的最小费用。$1\le n\le 5000$，$1\le m\le 5\times 10^4$。

**思路**：直接套用 SSP 模板。建图时正向边容量 $c_i$、费用 $w_i$，反向边容量 $0$、费用 $-w_i$，循环求费用最短路并增广即可。

**复杂度**：$O(F\cdot nm)$（SPFA 版，$F$ 为最大流）；势能 Dijkstra 版为 $O(F\cdot m\log n)$。

**参考代码**：见第 4 节完整模板，只需按输入建边。

---

### 例题 2：餐巾计划问题（Luogu P1251）

**题目大意**：一家餐厅第 $i$ 天需要 $r_i$ 块干净餐巾。每块新餐巾购买价 $p$；快洗部洗一块需 $m$ 天、费用 $f$；慢洗部需 $n$ 天、费用 $s$。每天用过的餐巾可以送去洗，也可以留到以后送洗。求满足所有需求的最小总花费。

**思路**：把每一天拆成「早上（需要干净餐巾）」和「晚上（产生脏餐巾）」两个点，建最小费用最大流：

- 源点 $S\to$ 第 $i$ 天早上，容量 $\infty$、费用 $p$：购买新餐巾；
- 第 $i$ 天早上 $\to$ 汇点 $T$，容量 $r_i$、费用 $0$：满足当天需求（满流即满足）；
- 源点 $S\to$ 第 $i$ 天晚上，容量 $r_i$、费用 $0$：当天产生的脏餐巾；
- 第 $i$ 天晚上 $\to$ 第 $i+m$ 天早上，容量 $\infty$、费用 $f$：快洗；
- 第 $i$ 天晚上 $\to$ 第 $i+n$ 天早上，容量 $\infty$、费用 $s$：慢洗；
- 第 $i$ 天晚上 $\to$ 第 $i+1$ 天晚上，容量 $\infty$、费用 $0$：把脏餐巾留到以后处理。

跑最小费用最大流，费用即为答案。总点数 $2n+2$，边数 $O(n)$。

**复杂度**：$O(F\cdot nm)$，其中 $F=\sum r_i$；实际远小于上界。

```cpp
// 设 N 为天数（与模板中的点数 n 区分），快洗天数 fastD、慢洗天数 slowD
for (int i = 1; i <= N; ++i) {
    addEdge(S, i, INF, price);                        // 买新餐巾
    addEdge(S, i + N, need[i], 0);                    // 当天产生的脏餐巾
    addEdge(i, T, need[i], 0);                        // 当天需求
    if (i + 1 <= N) addEdge(i + N, i + N + 1, INF, 0);// 脏餐巾留到明天
    if (i + fastD <= N) addEdge(i + N, i + fastD, INF, fastCost);  // 快洗
    if (i + slowD <= N) addEdge(i + N, i + slowD, INF, slowCost);  // 慢洗
}
```

---

### 例题 3：负载平衡问题（Luogu P4016）

**题目大意**：有 $n$ 个仓库围成一圈，第 $i$ 个仓库有 $a_i$ 件货物。每次可以把一件货物从相邻仓库搬运一格，代价为 $1$。求让所有仓库货物数量相同的最小总代价。

**思路**：设平均值 $\bar a=\sum a_i/n$。把每个仓库看成点，相邻仓库之间连两条有向边（容量 $\infty$、费用 $1$）。

- 若 $a_i>\bar a$，源点向 $i$ 连容量 $a_i-\bar a$、费用 $0$ 的边；
- 若 $a_i<\bar a$，$i$ 向汇点连容量 $\bar a-a_i$、费用 $0$ 的边。

跑最小费用最大流，流量为「多余货物总量」，费用即最少搬运次数。注意 $n$ 个仓库首尾相接形成环，相邻边要连 $i\to i+1$ 与 $i+1\to i$ 两个方向。

**复杂度**：$O(F\cdot nm)$，$F\le \sum a_i$。

---

### 例题 4：分配问题（Luogu P4014）

**题目大意**：有 $n$ 个人和 $n$ 件工作，第 $i$ 个人做第 $j$ 件工作的效益为 $c_{ij}$。每人做一件工作，每件工作由一人做，求最大总效益与最小总效益。

**思路**：二分图最小费用最大流建模：

- 源点 $S\to$ 第 $i$ 个人，容量 $1$、费用 $0$；
- 第 $i$ 个人 $\to$ 第 $j$ 件工作，容量 $1$、费用 $c_{ij}$；
- 第 $j$ 件工作 $\to$ 汇点 $T$，容量 $1$、费用 $0$。

跑最小费用最大流得到最小总效益。求最大总效益时把费用取相反数（$-c_{ij}$）再跑一次，答案取相反数即可。流量必为 $n$，所以一定满流。

**复杂度**：$O(n\cdot nm)$，即 $O(n^2m)$ 级别，$n\le 100$ 时完全够用。

---

## 7. 易错点

1. **反向边编号错误**：把 `ecnt` 初始化为 $0$ 却用 `i ^ 1` 取反向边，会错位。正确做法是 `ecnt = 1`，`i` 与 `i ^ 1` 成对。
2. **数组开小**：每条原边会产生一条反向边，边的结构体数组要开 $2m$，不是 $m$。
3. **INF 不够大**：费用累计可能超过 `int`，一律用 `long long`；瓶颈初始值用 `1e18`。
4. **把费用当流量**：`flow += add`，`cost += add * dis[t]`，两者不要混。
5. **最短路用 Dijkstra 但图中有负边**：未初始化势能时直接跑 Dijkstra 会得到错误答案，必须先用 SPFA/Bellman-Ford 求初始势能。
6. **费用流结束条件**：求最大流时在 `dis[t] == INF` 时停止，而不是 `dis[t] > 0` 时停止（后者是「最小费用可行流」的变体）。
7. **负环问题**：若原图存在负费用环，SSP 不能直接使用，需要先消去负环，或题目保证无负环。
8. **重边与自环**：自环在费用流中没有意义，建图前应当去掉。
9. **多组数据不清空**：`head`、`ecnt` 以及边数组都要重新初始化。

---

## 8. 练习建议

由易到难：

1. **入门模板**：Luogu P3381【模板】最小费用最大流 —— 熟悉建图与 SPFA 增广；
2. **二分图建模**：Luogu P4014 分配问题、P4015 运输问题 —— 练习源汇点与容量 1 的建模；
3. **拆点建模**：Luogu P1251 餐巾计划问题 —— 学会把「状态」拆成两层点；
4. **环上平衡**：Luogu P4016 负载平衡问题 —— 体会相邻点连边的技巧；
5. **费用流 + 二分答案**：Luogu P3705 [SDOI2017] 新生舞会 —— 分数规划与费用流结合；
6. **费用流 + 动态加边**：Luogu P2050 [NOI2012] 美食节 —— 体会动态加点优化；
7. **区间覆盖建模**：Luogu P3980 [NOI2008] 志愿者招募 —— 经典「流量平衡」建模，难度较高。

学习建议：

- 先手算几组小数据，理解「残量网络 + 反向边负费用」的含义；
- 写题时先用 SPFA 版保证正确，再尝试势能 Dijkstra 版；
- 费用流题目的难点 90% 在建模，模板本身并不复杂，建议整理一份自己的建图套路笔记。
