# 最短路

> 一句话概括：给定一张带权图和起点 $s$，「最短路」求的是 $s$ 到其它所有点的最小路径长度；不同数据范围适用不同算法，选错算法比写错代码更致命。

最短路是图论中出现频率最高的模型之一：导航、网络路由、差分约束、费用流、分层图 DP 都在用它。本节按「边权性质 + 数据范围」梳理四大主流算法（Dijkstra、Bellman-Ford、SPFA、Floyd）以及 0-1 BFS、Johnson 等补充技巧，并给出可直接编译的模板。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 问题的定义与性质
   - 1.2 Dijkstra 算法
   - 1.3 Bellman-Ford 与 SPFA
   - 1.4 Floyd 算法
   - 1.5 其它技巧
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 问题的定义与性质

给定带权有向图 $G=(V,E)$，边 $(u,v)$ 的权值为 $w(u,v)$，起点 $s$。定义

$$
d(s,v)=\min_{\pi:s\to v}\sum_{e\in\pi}w(e),
$$

即所有从 $s$ 到 $v$ 的路径中权值和的最小值。若不存在路径则 $d(s,v)=+\infty$。

**最短路的两个基本性质**（后面所有算法都依赖它们）：

- **最优子结构**：最短路的任意子路径也是最短路。若 $s\to u\to v$ 是 $s$ 到 $v$ 的最短路，则 $s\to u$ 必是 $s$ 到 $u$ 的最短路。
- **三角不等式（松弛条件）**：对所有边 $(u,v)$，恒有

$$
d(s,v)\le d(s,u)+w(u,v).
$$

算法中的「松弛（relax）」操作就是不断用右边去更新左边，直到所有边都满足该不等式。

按边权分类：

| 边权情况 | 可用算法 |
|---|---|
| 全部为正权 | Dijkstra（$O(m\log n)$）、Floyd（小图） |
| 全部非负，含 0 | Dijkstra 仍然正确；0-1 BFS 可处理 0/1 权 |
| 存在负权边，无负环 | Bellman-Ford、SPFA |
| 存在负环 | 最短路无意义（可无限绕圈），需负环判定 |
| 任意两点最短路，$n\le 400$ | Floyd |

### 1.2 Dijkstra 算法

**核心思想**：贪心。维护集合 $S$ 表示「最短路已确定的点」，每次从 $V\setminus S$ 中取出当前 $d$ 值最小的点 $u$ 加入 $S$，然后用 $u$ 的所有出边松弛邻居。

**为什么正确？** 因为边权非负，所以「当前 $d$ 最小的点」的最短路不可能再被其它点更新得更小：任何经过未确定点的路径，其长度至少是当前 $d(u)$ 加上一个非负数。

**为什么不能有负权边？** 反例：$s\to a$ 权 2，$s\to b$ 权 3，$b\to a$ 权 $-2$。Dijkstra 会先确定 $a$ 的最短路为 2，但实际上 $s\to b\to a=1$ 更短。

**流程示意**：

```
初始: d[s]=0, 其余 INF, S=∅
① 取出 d 最小的 s，加入 S，松弛 s 的出边
② 取出剩余点中 d 最小的 u，加入 S，松弛 u 的出边
③ 重复直到所有点都在 S 中
```

**复杂度**：用二叉堆（`priority_queue`）维护，每个点可能被多次入堆，总入堆次数 $O(m)$，每次 $O(\log n)$，故

$$
T=O(m\log n),
$$

空间 $O(n+m)$。

### 1.3 Bellman-Ford 与 SPFA

#### Bellman-Ford

**核心思想**：暴力松弛。由于最短路最多包含 $n-1$ 条边，所以对所有边做 $n-1$ 轮松弛，第 $k$ 轮结束后 $d(v)$ 至多是「最多用 $k$ 条边」的最短路。

```
for i = 1 .. n-1:
    for each edge (u,v,w):
        d[v] = min(d[v], d[u] + w)
```

**负环判定**：第 $n$ 轮若还有边能被松弛，说明存在负环（可以再绕一圈让路径更短）。

**复杂度**：$O(nm)$，适用于 $n,m$ 较小的图。

#### SPFA（队列优化的 Bellman-Ford）

**核心思想**：只有被松弛过的点才可能引起邻居的更新，因此用队列存「刚被更新的点」，只对这些点做松弛。

```
d[s]=0, q.push(s), inq[s]=true
while q not empty:
    u = q.front(); q.pop(); inq[u]=false
    for each edge (u,v,w):
        if d[u]+w < d[v]:
            d[v] = d[u]+w
            if not inq[v]: q.push(v), inq[v]=true
```

**复杂度**：随机图上表现接近 $O(m)$，但**最坏是 $O(nm)$**，出题人常构造网格图/菊花图卡 SPFA（即所谓「SPFA 已死」）。正式比赛中除非确定数据随机，否则优先 Dijkstra。

**负环判定**：记录每个点的**入队次数**，若某点入队超过 $n$ 次则存在负环；或记录最短路经过的边数 `cnt[v] = cnt[u] + 1`，若 `cnt[v] >= n` 则有负环。

### 1.4 Floyd 算法

**核心思想**：区间 DP。令 $f_{k}(i,j)$ 表示「只允许经过编号 $\le k$ 的点作为中转」时 $i$ 到 $j$ 的最短路，则

$$
f_k(i,j)=\min\bigl(f_{k-1}(i,j),\ f_{k-1}(i,k)+f_{k-1}(k,j)\bigr).
$$

把第一维滚动掉，就得到经典的三重循环：

```cpp
for (int k = 1; k <= n; ++k)
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= n; ++j)
            f[i][j] = min(f[i][j], f[i][k] + f[k][j]);
```

**注意 $k$ 必须在最外层**：$k$ 表示「允许使用的中转点集合」逐步扩大，写在最内层会破坏 DP 的阶段含义。

**复杂度**：$O(n^3)$，空间 $O(n^2)$。可以顺便处理负权边（但不能有负环），也常用于求**传递闭包**（把 `min` 换成 `or`）。

### 1.5 其它技巧

#### 0-1 BFS（双端队列 BFS）

边权只有 0 和 1 时，用 `deque`：权为 0 的边把邻居压到队头，权为 1 的压到队尾。这样队列始终保持「按距离单调」，复杂度 $O(n+m)$，比 Dijkstra 少一个 $\log$。

#### 多源最短路

建立超级源点 $S$，向所有源点连一条权为 0 的边，再跑一次单源最短路即可。

#### 分层图

当有 $k$ 次「免费通行」机会时，把状态扩展为 $(u,k)$，每用一次机会就从第 $k$ 层走到第 $k+1$ 层，边权为 0。整张图有 $n(k+1)$ 个点，跑 Dijkstra 即可。

#### Johnson 全源最短路

有负权边又要求全源最短路时：先用 SPFA/Bellman-Ford 求一个势能 $h(v)$（从超级源点到各点的最短路），然后把每条边权改为 $w'(u,v)=w(u,v)+h(u)-h(v)\ge 0$，再对每个点跑 Dijkstra。总复杂度 $O(nm\log n)$。

---

## 2. 模板代码

### 2.1 Dijkstra（堆优化，邻接表）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
const long long INF = 1e18;

struct Edge { int to; long long w; };
vector<Edge> g[MAXN];
long long dist[MAXN];
bool done[MAXN];

void dijkstra(int s, int n) {
    fill(dist, dist + n + 1, INF);
    memset(done, 0, sizeof(bool) * (n + 1));
    // 小根堆：pair 默认按 first 升序，second 存点编号
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[s] = 0;
    pq.push({0, s});
    while (!pq.empty()) {
        long long d = pq.top().first;
        int u = pq.top().second;
        pq.pop();
        if (done[u]) continue;      // 懒惰删除：同一个点可能多次入堆
        done[u] = true;
        for (auto &e : g[u]) {
            if (dist[u] + e.w < dist[e.to]) {
                dist[e.to] = dist[u] + e.w;
                pq.push({dist[e.to], e.to});
            }
        }
    }
}

int main() {
    int n, m, s;
    scanf("%d%d%d", &n, &m, &s);
    for (int i = 1; i <= m; ++i) {
        int u, v, w;
        scanf("%d%d%d", &u, &v, &w);
        g[u].push_back({v, w});
    }
    dijkstra(s, n);
    for (int i = 1; i <= n; ++i) {
        if (dist[i] == INF) printf("INF ");
        else printf("%lld ", dist[i]);
    }
    puts("");
    return 0;
}
```

### 2.2 Bellman-Ford + 负环判定

```cpp
struct BFEdge { int u, v; long long w; };
vector<BFEdge> edges;
long long dist[MAXN];

bool bellmanFord(int n, int s) {
    fill(dist, dist + n + 1, INF);
    dist[s] = 0;
    for (int i = 1; i <= n - 1; ++i) {
        bool changed = false;
        for (auto &e : edges) {
            if (dist[e.u] == INF) continue;
            if (dist[e.u] + e.w < dist[e.v]) {
                dist[e.v] = dist[e.u] + e.w;
                changed = true;
            }
        }
        if (!changed) break;        // 提前退出：已经收敛
    }
    // 第 n 轮仍能松弛 => 存在负环
    for (auto &e : edges) {
        if (dist[e.u] != INF && dist[e.u] + e.w < dist[e.v]) return false;
    }
    return true;
}
```

### 2.3 SPFA + 负环判定（BFS 版）

```cpp
int cnt[MAXN];              // cnt[v] = 最短路经过的边数
bool inq[MAXN];
queue<int> q;

bool spfa(int n, int s) {
    fill(dist, dist + n + 1, INF);
    memset(cnt, 0, sizeof(int) * (n + 1));
    memset(inq, 0, sizeof(bool) * (n + 1));
    dist[s] = 0; q.push(s); inq[s] = true;
    while (!q.empty()) {
        int u = q.front(); q.pop(); inq[u] = false;
        for (auto &e : g[u]) {
            if (dist[u] + e.w < dist[e.to]) {
                dist[e.to] = dist[u] + e.w;
                cnt[e.to] = cnt[u] + 1;
                if (cnt[e.to] >= n) return false;   // 负环
                if (!inq[e.to]) { q.push(e.to); inq[e.to] = true; }
            }
        }
    }
    return true;
}
```

### 2.4 Floyd（含路径还原）

```cpp
const int MAXN2 = 405;
const long long INF2 = 1e18;
long long f[MAXN2][MAXN2];
int nxt[MAXN2][MAXN2];          // nxt[i][j]：i 到 j 最短路上 i 的下一个点

void floyd(int n) {
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= n; ++j)
            nxt[i][j] = (f[i][j] < INF2) ? j : 0;
    for (int k = 1; k <= n; ++k)
        for (int i = 1; i <= n; ++i) {
            if (f[i][k] == INF2) continue;       // 剪枝：不可达
            for (int j = 1; j <= n; ++j) {
                if (f[k][j] == INF2) continue;
                if (f[i][k] + f[k][j] < f[i][j]) {
                    f[i][j] = f[i][k] + f[k][j];
                    nxt[i][j] = nxt[i][k];
                }
            }
        }
}

// 输出 i -> j 的路径
void printPath(int i, int j) {
    if (!nxt[i][j]) { puts("No path"); return; }
    printf("%d", i);
    while (i != j) { i = nxt[i][j]; printf(" -> %d", i); }
    puts("");
}
```

### 2.5 0-1 BFS

边权只有 0 和 1 时，把邻接表存成 `vector<pair<int,int>> g01[]`，用双端队列：权 0 的边把邻居压到队头，权 1 的压到队尾，队列始终按距离单调。

```cpp
vector<pair<int,int>> g01[MAXN];     // (to, w)，w ∈ {0,1}

void zeroOneBFS(int s, int n) {
    fill(dist, dist + n + 1, INF);
    deque<int> dq;
    dist[s] = 0;
    dq.push_front(s);
    while (!dq.empty()) {
        int u = dq.front(); dq.pop_front();
        for (auto &e : g01[u]) {
            if (dist[u] + e.second < dist[e.first]) {
                dist[e.first] = dist[u] + e.second;
                if (e.second == 0) dq.push_front(e.first);
                else dq.push_back(e.first);
            }
        }
    }
}
```

---

## 3. 经典例题

### 例题 1：单源最短路径（洛谷 P4779，Dijkstra 模板）

**题意**：给定 $n$ 个点、$m$ 条边的有向图，边权非负，求 $s$ 到所有点的最短路。$n\le 10^5$，$m\le 2\times 10^5$，边权 $\le 10^9$。

**思路**：标准的堆优化 Dijkstra。注意边权总和可能超过 `int`，距离数组必须开 `long long`。直接调用 2.1 节的 `dijkstra(s, n)`，再依次输出 `dist[1..n]`（不可达输出题目要求的标记）。

**复杂度**：$O(m\log n)$，空间 $O(n+m)$。

### 例题 2：负环（洛谷 P3385）

**题意**：给定 $n$ 个点、$m$ 条边的有向图，判断是否存在一个从起点可达的负环。$n\le 2000$，$m\le 3000$。

**思路**：Bellman-Ford 或 SPFA 判负环。注意题目通常要求「从 1 号点可达」的负环：只需把起点设为 1，并且只对可达点做松弛。若要求图中任意负环，则把所有点初始入队、`dist` 全 0。调用 2.3 节的 `spfa(n, 1)`，返回 `false` 即存在负环。

**复杂度**：$O(nm)$ 最坏。

### 例题 3：路径统计（洛谷 P1608）

**题意**：给定 $n$ 个点 $m$ 条边的有向图，求 1 到 $n$ 的最短路条数。若有重边需要去重（相同 $(u,v)$ 只保留最短的一条）。

**思路**：在 Dijkstra 松弛时维护 `cnt[v]`：如果找到更短的路径，`cnt[v] = cnt[u]`；如果长度相等，`cnt[v] += cnt[u]`。重边用 `map<pair<int,int>,int>` 或排序去重。

```cpp
long long dist[MAXN], cnt[MAXN];
bool done[MAXN];

void dijkstraCount(int s, int n) {
    fill(dist, dist + n + 1, INF);
    fill(cnt, cnt + n + 1, 0);
    memset(done, 0, sizeof(bool) * (n + 1));
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[s] = 0; cnt[s] = 1;
    pq.push({0, s});
    while (!pq.empty()) {
        long long d = pq.top().first;
        int u = pq.top().second;
        pq.pop();
        if (done[u]) continue;
        done[u] = true;
        for (auto &e : g[u]) {
            if (dist[u] + e.w < dist[e.to]) {
                dist[e.to] = dist[u] + e.w;
                cnt[e.to] = cnt[u];
                pq.push({dist[e.to], e.to});
            } else if (dist[u] + e.w == dist[e.to]) {
                cnt[e.to] += cnt[u];
            }
        }
    }
}
```

**复杂度**：$O(m\log n)$。

### 例题 4：分层图最短路（洛谷 P4568 飞行路线）

**题意**：$n$ 个城市 $m$ 条航线，有 $k$ 次免费乘坐机会，求从 $s$ 到 $t$ 的最小花费。$n\le 10^4$，$m\le 5\times10^4$，$k\le 10$。

**思路**：把每个点拆成 $k+1$ 层，第 $j$ 层表示「已用 $j$ 次免费机会」。原边 $(u,v,w)$ 在每一层连权为 $w$ 的边，同时从第 $j$ 层的 $u$ 向第 $j+1$ 层的 $v$ 连权为 0 的边。答案取第 $k$ 层的 $t$（其实每层的 $t$ 都可以，因为可以「不用完」机会）。

```cpp
int id(int u, int k) { return u + k * n; }   // 分层编号

void buildLayered(int n, int k) {
    for (auto &e : rawEdges) {
        for (int j = 0; j <= k; ++j) {
            g[id(e.u, j)].push_back({id(e.v, j), e.w});
            if (j < k) g[id(e.u, j)].push_back({id(e.v, j + 1), 0});
        }
    }
}
// 答案 = min over j of dist[id(t, j)]
```

**复杂度**：点数 $n(k+1)$，边数 $m(2k+1)$，Dijkstra 为 $O(mk\log(nk))$。

---

## 4. 易错点与练习建议

### 易错点

1. **负权边用 Dijkstra**：结果错误且可能死循环。看到负权必须换 Bellman-Ford/SPFA 或在必要时用 Johnson。
2. **`INF` 溢出**：`dist[u] + e.w` 若 `dist[u]==INF` 会溢出成负数，进而被误判为更短路。松弛前先判 `dist[u] != INF`，或把 `INF` 设为 `0x3f3f3f3f` 并保证两倍不溢出。
3. **`int` 存不下**：$n=10^5$、边权 $10^9$ 时最短路可达 $10^{14}$，必须 `long long`。
4. **Dijkstra 的懒惰删除写错**：弹出时若 `done[u]` 已为真直接 `continue`；不要忘记这个判断，否则复杂度退化。
5. **Floyd 的 $k$ 写在最内层**：DP 阶段含义被破坏，结果错误。**$k$ 必须在最外层**。
6. **Floyd 初始化**：`f[i][i]=0`，`f[i][j]=INF`（$i\ne j$），有边则赋权值，重边取最小。
7. **SPFA 判负环用入队次数**：应统计「入队次数 $>n$」，或「最短路边数 $\ge n$」。只统计松弛次数会误判。
8. **多源最短路忘记超级源点**：直接对每个源跑一遍 Dijkstra 是 $O(km\log n)$，能建超级源点时应优化为一次。
9. **0-1 BFS 用了普通队列**：必须用 `deque` 且 0 权压队头，否则退化成 Bellman-Ford。
10. **无向图负权边**：一条负权无向边等价于一个长度为 2 的负环，此时最短路无定义，题目不会这么给。
11. **输出格式**：不可达点按题目要求输出 `INF`、`-1` 或不输出，不要统一按 0 处理。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P3371 单源最短路径（弱化版） | Bellman-Ford/SPFA 入门 |
| 入门 | 洛谷 P4779 单源最短路径（标准版） | 堆优化 Dijkstra |
| 入门 | 洛谷 P2910 [USACO08OPEN] Clear And Present Danger | Floyd 模板 |
| 提高 | 洛谷 P3385 负环 | SPFA/Bellman-Ford 判负环 |
| 提高 | 洛谷 P1608 路径统计 | 最短路计数 + 去重边 |
| 提高 | 洛谷 P4568 飞行路线 | 分层图最短路 |
| 提高 | 洛谷 P1462 通往奥格瑞玛的道路 | 二分答案 + 最短路 |
| 进阶 | 洛谷 P1073 最优贸易 | 分层图 / 正反图 |
| 进阶 | 洛谷 P5905 Johnson 全源最短路 | 势能 + Dijkstra |

学习建议：先吃透 Dijkstra 的正确性证明和堆优化写法，再理解 Bellman-Ford 的「轮数 = 边数」思想，最后用 Floyd 解决小图全源问题。每道题先分析「边权是否有负」「是否需要全源」「$n,m$ 多大」，再决定算法——这是最短路题的第一反应。
