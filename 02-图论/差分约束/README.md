# 差分约束（Difference Constraints）

> 一句话概括：**差分约束**把一组形如 $x_j - x_i \le c$ 的不等式转化为有向图上的最短路/最长路问题，用 SPFA 或 Bellman-Ford 判断可行性并求出可行解。

---

## 目录

1. [问题模型](#1-问题模型)
2. [核心原理：不等式 → 最短路](#2-核心原理不等式--最短路)
3. [最大值与最小值的区别](#3-最大值与最小值的区别)
4. [可行性判定与负环](#4-可行性判定与负环)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 问题模型

### 1.1 定义

给定 $n$ 个变量 $x_1,\dots,x_n$ 和 $m$ 个约束，每个约束形如：

$$
x_j - x_i \le c_k
$$

其中 $c_k$ 是常数（可正可负）。求一组满足所有约束的解，或判断无解。

### 1.2 变形后的常见形式

| 原约束 | 等价形式 |
|---|---|
| $x_j - x_i \le c$ | $x_j \le x_i + c$ |
| $x_j - x_i \ge c$ | $x_i \le x_j - c$ |
| $x_j - x_i = c$ | $x_j \le x_i + c$ 且 $x_i \le x_j - c$ |
| $x_j < x_i + c$（整数） | $x_j \le x_i + c - 1$ |
| $x_j > x_i + c$（整数） | $x_i \le x_j - c - 1$ |

> 差分约束要求约束形式统一为「**两个变量之差**与常数比较」。若出现 $x_i + x_j \le c$ 这类和式，则不是差分约束，需要用其他方法。

### 1.3 一个直观例子

某班级有三位同学的身高 $x_1,x_2,x_3$（整数），已知：

- $x_1 - x_2 \le 3$（1 号比 2 号最多高 3）；
- $x_2 - x_3 \le -2$（2 号比 3 号至少矮 2）；
- $x_1 - x_3 \ge 5$（1 号比 3 号至少高 5）。

问是否存在满足条件的整数身高差。

**答案是不存在**。因为 $x_1-x_3=(x_1-x_2)+(x_2-x_3)\le 3+(-2)=1$，与 $x_1-x_3\ge 5$ 矛盾。这个「不等式相加导出矛盾」的想法，正是后面「负环判定」的来源。

---

## 2. 核心原理：不等式 → 最短路

### 2.1 关键引理

**引理（最短路的三角不等式）**：设 $d(v)$ 为源点 $s$ 到 $v$ 的最短距离，则对任意边 $(u,v,w)$ 有 $d(v) \le d(u) + w$，移项即 $d(v)-d(u)\le w$。

### 2.2 建图规则

对每个约束 $x_j - x_i \le c$，连一条有向边 $i \to j$，边权为 $c$：

```
约束：x_j - x_i <= c      建图：i --(c)--> j      含义：dis[j] <= dis[i] + c
```

于是「满足所有约束的一组解」就等价于「图中以某点为源点的最短路数组 $dis$」。

### 2.3 超级源点

为了让所有点都有距离值，通常建一个**超级源点** $S$，向每个点 $i$ 连一条边权为 $0$ 的边：

$$
S\to i,\quad w=0
$$

这样从 $S$ 出发求最短路，得到的 $dis[i]$ 就是一组合法解（若存在）。

> 为什么要加超级源点？因为图可能不连通，从单一起点无法到达所有点；超级源点保证一次 SPFA 覆盖全图，也方便检测全图的负环。

### 2.4 文字示意

**可行例子**。约束：

$$
\begin{cases}
x_2 - x_1 \le 3\\
x_3 - x_2 \le -2\\
x_3 - x_1 \le -1
\end{cases}
$$

建图（另加超级源点 $S$ 向 $1,2,3$ 连权 $0$ 的边）：

```
S --0--> 1 --3--> 2 --(-2)--> 3
          \                   ^
           \-----(-1)---------+
```

从 $S$ 出发的最短路：

$$
dis[1]=0,\quad dis[2]=3,\quad dis[3]=\min(3-2,\ 0-1)=-1
$$

取 $(x_1,x_2,x_3)=(0,3,-1)$ 逐一验证：

- $x_2-x_1=3\le 3$ ✓
- $x_3-x_2=-4\le -2$ ✓
- $x_3-x_1=-1\le -1$ ✓

确实是一组可行解。

**无解例子**。把上例第三条改成 $x_1-x_3\le -2$：

$$
\begin{cases}
x_2 - x_1 \le 3\\
x_3 - x_2 \le -2\\
x_1 - x_3 \le -2
\end{cases}
$$

三条不等式相加得 $0 \le 3-2-2=-1$，矛盾。对应到图上，环 $1\to2\to3\to1$ 的边权和为 $3+(-2)+(-2)=-1<0$，是负环。

> 结论：**差分约束无解 $\iff$ 对应图中存在负环**。这也是可以用 SPFA 判负环来判定可行性的原因。

---

## 3. 最大值与最小值的区别

### 3.1 求最大值：最短路

若要求 $\max x_n$（在满足约束的前提下），则把所有约束化为 $x_j \le x_i + c$ 的形式，建图求**最短路**，$dis[n]$ 就是最大值。原因是「最短路」给出的正是所有约束共同允许的**上界**。

### 3.2 求最小值：最长路

若要求 $\min x_n$，则把所有约束化为 $x_j \ge x_i + c$ 的形式（即 $x_i - x_j \le -c$），建图求**最长路**，$dis[n]$ 就是最小值。原因是「最长路」给出的正是所有约束共同要求的**下界**。

### 3.3 记忆口诀与最长路实现

| 目标 | 约束形式 | 求解 |
|---|---|---|
| 最大值 | $x_j \le x_i + c$（$\le$） | 最短路 |
| 最小值 | $x_j \ge x_i + c$（$\ge$） | 最长路 |

> 两种写法本质相同：把 $\ge$ 两边乘 $-1$ 就变成 $\le$。

最长路可以直接把 SPFA 的松弛条件反过来写（`dis[v] < dis[u] + w` 则松弛），此时检测的是**正环**而不是负环。完整的 `spfaLongest` 实现见 5.2 节。

---

## 4. 可行性判定与负环

### 4.1 有解的条件

**定理**：差分约束系统有解 $\iff$ 对应图中不存在负环（最短路版本）。

- 若存在负环，绕环一圈不等式会推出 $x < x$，矛盾；
- 若不存在负环，最短路数组 $dis$ 就是一组合法解。

### 4.2 SPFA 判负环

SPFA 中，若某个点入队次数超过 $n$（或路径边数超过 $n-1$），则存在负环。

```cpp
// cnt[v] 表示到 v 的最短路经过的边数
if (dis[u] + w < dis[v]) {
    dis[v] = dis[u] + w;
    cnt[v] = cnt[u] + 1;
    if (cnt[v] >= n) return false;   // 负环
    if (!inq[v]) { q.push(v); inq[v] = true; }
}
```

### 4.3 Bellman-Ford 判负环

```cpp
bool bellmanFord(int s, int n, int m) {
    fill(dis, dis + n + 1, INF);
    dis[s] = 0;
    for (int i = 1; i <= n; ++i) {
        bool updated = false;
        for (int j = 1; j <= m; ++j) {
            int u = eu[j], v = ev[j], w = ew[j];
            if (dis[u] != INF && dis[v] > dis[u] + w) {
                dis[v] = dis[u] + w;
                updated = true;
                if (i == n) return false;   // 第 n 轮还能松弛 => 负环
            }
        }
        if (!updated) break;
    }
    return true;
}
```

> 为什么是第 $n$ 轮？含 $n$ 个点的图，任何简单路径最多 $n-1$ 条边。若第 $n$ 轮仍能松弛，说明最短路经过了 $\ge n$ 条边，必存在负环。

---

## 5. 模板代码

### 5.1 最短路版本（求最大值）

输入格式：

```
n m
u v w     （表示 x_v - x_u <= w）
```

输出 `YES` + 一组解，或 `NO`（无解）。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 5010;
const int MAXM = 10010;
const int INF = 0x3f3f3f3f;

struct Edge { int to, w, nxt; } e[MAXM << 1];

int head[MAXN], ecnt = 0;
int dis[MAXN], cnt[MAXN];
bool inq[MAXN];

void addEdge(int u, int v, int w) {
    e[++ecnt] = {v, w, head[u]};
    head[u] = ecnt;
}

// 返回 false 表示有负环（无解）
bool spfa(int s, int n) {
    fill(dis, dis + n + 1, INF);
    fill(cnt, cnt + n + 1, 0);
    fill(inq, inq + n + 1, false);
    queue<int> q;
    dis[s] = 0; q.push(s); inq[s] = true;
    while (!q.empty()) {
        int u = q.front(); q.pop(); inq[u] = false;
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (dis[u] + e[i].w < dis[v]) {
                dis[v] = dis[u] + e[i].w;
                cnt[v] = cnt[u] + 1;
                if (cnt[v] >= n) return false;
                if (!inq[v]) { inq[v] = true; q.push(v); }
            }
        }
    }
    return true;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= m; ++i) {
        int u, v, w; cin >> u >> v >> w;
        addEdge(u, v, w);          // x_v - x_u <= w
    }
    int S = n + 1;                 // 超级源点
    for (int i = 1; i <= n; ++i) addEdge(S, i, 0);

    if (!spfa(S, n + 1)) {
        cout << "NO\n";
    } else {
        cout << "YES\n";
        for (int i = 1; i <= n; ++i) cout << dis[i] << ' ';
        cout << '\n';
    }
    return 0;
}
```

> 注意：`cnt[v] >= n` 中的 $n$ 要传「实际点数」（含超级源点为 $n+1$），否则负环判定会偏松或偏紧。

### 5.2 最长路版本（求最小值）

```cpp
// 约束统一为 x_v - x_u >= w  =>  x_v >= x_u + w，建边 u -> v 权 w，求最长路
bool spfaLongest(int s, int n) {
    fill(dis, dis + n + 1, -INF);
    fill(cnt, cnt + n + 1, 0);
    fill(inq, inq + n + 1, false);
    queue<int> q;
    dis[s] = 0; q.push(s); inq[s] = true;
    while (!q.empty()) {
        int u = q.front(); q.pop(); inq[u] = false;
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (dis[u] != -INF && dis[u] + e[i].w > dis[v]) {
                dis[v] = dis[u] + e[i].w;
                cnt[v] = cnt[u] + 1;
                if (cnt[v] >= n) return false;   // 正环，无解
                if (!inq[v]) { inq[v] = true; q.push(v); }
            }
        }
    }
    return true;
}
```

### 5.3 常用约束转换表

```cpp
// 约束：x_v - x_u <= w
void leq(int u, int v, int w) { addEdge(u, v, w); }

// 约束：x_v - x_u >= w  <=>  x_u - x_v <= -w
void geq(int u, int v, int w) { addEdge(v, u, -w); }

// 约束：x_v - x_u == w
void eq(int u, int v, int w) { leq(u, v, w); geq(u, v, w); }

// 约束：x_v - x_u < w（整数）  <=>  x_v - x_u <= w - 1
void lt(int u, int v, int w) { addEdge(u, v, w - 1); }

// 约束：x_v - x_u > w（整数）  <=>  x_u - x_v <= -w - 1
void gt(int u, int v, int w) { addEdge(v, u, -w - 1); }
```

### 5.4 复杂度

- SPFA：平均 $O(n+m)$，最坏 $O(nm)$；
- Bellman-Ford：$O(nm)$；
- 若图中无负权边，可用 Dijkstra 求最短路，$O((n+m)\log n)$；但差分约束一般有负权，需先处理。

---

## 6. 经典例题

### 例题 1：【模板】差分约束（Luogu P5960）

**题目大意**：给定 $n$ 个变量和 $m$ 个约束 $x_v - x_u \le w$，求一组解或判断无解。

**思路**：直接套用第 5.1 节模板，建超级源点，SPFA 判负环。

**复杂度**：$O(nm)$ 最坏。

---

### 例题 2：小 K 的农场（Luogu P1993）

**题目大意**：农场里有 $n$ 个作物，$m$ 条信息，每条信息是以下三种之一：

1. $x_a - x_b \ge c$（a 的收成比 b 至少多 c）；
2. $x_a - x_b \le c$（a 的收成比 b 至多 c）；
3. $x_a = x_b$。

判断是否存在满足所有信息的方案。

**思路**：把所有约束统一为 $x_j - x_i \le c$：

- 条件 1：$x_b - x_a \le -c$；
- 条件 2：$x_a - x_b \le c$；
- 条件 3：$x_a - x_b \le 0$ 且 $x_b - x_a \le 0$。

建图后 SPFA 判负环即可。

**复杂度**：$O(nm)$。

```cpp
// 条件 1: x_a - x_b >= c  =>  x_b - x_a <= -c  =>  addEdge(a, b, -c)
// 条件 2: x_a - x_b <= c                        =>  addEdge(b, a, c)
// 条件 3: x_a = x_b                             =>  addEdge(a, b, 0); addEdge(b, a, 0);
```

---

### 例题 3：排队布局（Luogu P4878 [USACO05DEC] Layout）

**题目大意**：$n$ 头奶牛排队（按编号顺序），有 $m_1$ 对奶牛互相喜欢（距离不超过 $d$），$m_2$ 对奶牛互相讨厌（距离至少为 $d$）。求 1 号奶牛与 $n$ 号奶牛之间的最大距离；若无法满足所有约束输出 $-1$；若距离可以无限大输出 $-2$。

**思路**：

1. 约束：
   - 喜欢：$x_j - x_i \le d$；
   - 讨厌：$x_j - x_i \ge d$，即 $x_i - x_j \le -d$；
   - 编号顺序：$x_{i+1} - x_i \ge 0$（即 $x_i - x_{i+1} \le 0$）。
2. 建图求最短路：
   - 从超级源点跑 SPFA，若有负环输出 $-1$；
   - 从 1 号点跑最短路，若 $n$ 不可达输出 $-2$；
   - 否则输出 $dis[n]$。

**复杂度**：$O(nm)$。

---

### 例题 4：天平（Luogu P2474 [SCOI2008]）

**题目大意**：有 $n$ 个砝码，重量满足若干关系（$A$ 比 $B$ 重、轻或相等）。问有多少对砝码 $(i,j)$ 满足「方案 1：$i+j$ 一定大于 $k$」「方案 2：$i+j$ 一定小于 $k$」等。

**思路**：用 Floyd 求所有点对之间的**最短路**（上界）与**最长路**（下界）：

- $d_1[i][j]$：$x_j - x_i$ 的最大值（最短路）；
- $d_2[i][j]$：$x_j - x_i$ 的最小值（最长路）。

然后枚举每对砝码，根据上下界判断 $x_i+x_j$ 与 $x_k$ 的关系。复杂度 $O(n^3)$。

---

## 7. 易错点

1. **不等式方向搞反**：$x_j - x_i \le c$ 对应边 $i\to j$ 权 $c$；写反会得到完全错误的结果。
2. **最大值用最长路**：求最大值要用最短路（所有约束化为 $\le$），求最小值用最长路（化为 $\ge$），不要混淆。
3. **负环判定阈值**：`cnt[v] >= n` 中的 $n$ 是**总点数**（含超级源点），不是变量数。
4. **超级源点边权**：一般设为 0；若题目要求变量非负，可加约束 $x_i - x_0 \ge 0$。
5. **整数严格不等式**：$x_j < x_i + c$ 在整数意义下是 $x_j \le x_i + c - 1$，不要漏掉 $-1$。
6. **SPFA 被卡**：差分约束图可能有特殊构造卡 SPFA，必要时用 SLF/LLL 优化或 Bellman-Ford。
7. **多组数据不清空**：`head`、`ecnt`、`dis`、`cnt`、`inq` 都要重置。
8. **输出解的范围**：SPFA 求出的解可能超出 `int`，视题目范围用 `long long`。
9. **自环**：若出现 $x_i - x_i \le c$ 且 $c<0$，说明无解（负自环）。

---

## 8. 练习建议

由易到难：

1. **模板**：Luogu P5960 【模板】差分约束 —— 熟悉建图与 SPFA；
2. **三种条件**：Luogu P1993 小 K 的农场 —— 练习不等式转换；
3. **最短路 + 判环**：Luogu P4878 [USACO05DEC] Layout —— 体会「不可达」与「负环」两种无解；
4. **上下界**：Luogu P2474 [SCOI2008] 天平 —— Floyd 求上下界；
5. **区间约束**：Luogu P3275 [SCOI2011] 糖果 —— 差分约束 + 最长路；
6. **进阶**：Luogu P4926 [1007] 倍杀测量者 —— 对数变换 + 差分约束 + 二分；
7. **综合**：Luogu P5590 赛车游戏 —— 差分约束构造。

学习建议：

- 牢记「$\le$ 用最短路、$\ge$ 用最长路」，做题前先把所有约束统一方向；
- 每个不等式都写成「$x_j \le x_i + c$」再建边，不要凭感觉；
- 写完模板后用暴力枚举小数据验证，特别是严格不等式的 $-1$ 处理。
