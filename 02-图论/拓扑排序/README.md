# 拓扑排序

> 一句话概括：对有向无环图（DAG）的所有顶点排出一个线性序列，使**每条边都从序列前面的点指向后面的点**，这个序列就叫拓扑序。

拓扑排序是处理「依赖关系」的通用工具：课程安排、任务调度、表达式求值、DAG 上 DP、强连通分量缩点后的处理，背后都是它。本节讲清 DAG 与拓扑序的等价关系，给出 Kahn（入度法）与 DFS 两种实现，并归纳 DAG 上 DP 的常见套路。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 偏序与拓扑序
   - 1.2 拓扑序存在性与 DAG
   - 1.3 Kahn 算法（入度法）
   - 1.4 DFS 求拓扑序
   - 1.5 DAG 上 DP
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 偏序与拓扑序

**偏序关系**是满足自反性、反对称性、传递性的二元关系，例如「课程 A 必须在课程 B 之前修完」。偏序关系允许两个元素**不可比较**（谁先谁后都行），这正是它区别于全序的地方。

**拓扑序**是对偏序集的一个**全序扩展**：保留原有偏序关系的相对顺序，同时给不可比较的元素任意指定顺序。

形式化地，给定有向图 $G=(V,E)$，若排列 $p_1,p_2,\dots,p_n$ 满足：对任意边 $\langle u,v\rangle\in E$，都有 $u$ 在序列中排在 $v$ 之前，则称该排列是一个拓扑序。

```
依赖关系示例：        一个合法拓扑序：

  A → B                 A, C, D, B, E, F
  A → C                 （A 必须先于 B、C；B 先于 E……）
  B → E
  C → D
  D → E
  E → F
```

注意拓扑序**通常不唯一**：例如 `A, C, B, D, E, F` 也合法。

### 1.2 拓扑序存在性与 DAG

**定理**：有向图存在拓扑序 $\iff$ 图中**没有有向环**（即图是 DAG，Directed Acyclic Graph）。

- 必要性：若有环 $v_1\to v_2\to\cdots\to v_k\to v_1$，拓扑序会要求 $v_1$ 在 $v_2$ 前、$v_2$ 在 $v_3$ 前……$v_k$ 在 $v_1$ 前，矛盾。
- 充分性：DAG 中一定存在入度为 0 的点（否则沿入边一直往前走到无穷，必然重复经过某点形成环）。删掉它继续归纳即可。

**关键推论**：若拓扑排序结束后仍有顶点未输出，则图中存在环，且这些未输出的点分别属于某些环或由环可达。

**DAG 的有用性质**：

- 所有点可以按拓扑序分成若干层，边只从前往后。
- 在 DAG 上可以按拓扑序做 DP（无后效性）。
- 任意 DAG 至少有一个入度为 0 的点和一个出度为 0 的点。

### 1.3 Kahn 算法（入度法）

**思路**：维护每个点的入度。把所有入度为 0 的点加入队列；每次取出一个点 $u$ 加入答案序列，并「删除」它的所有出边（即把邻居的入度减 1）；若某个邻居入度变为 0 就入队。重复直到队列为空。

```
初始图:  1 → 2, 1 → 3, 2 → 4, 3 → 4
入度:    in[1]=0  in[2]=1  in[3]=1  in[4]=2

q={1}
① 弹出 1，输出 1；删边 1→2、1→3，in[2]=0, in[3]=0 → q={2,3}
② 弹出 2，输出 2；删边 2→4，in[4]=1
③ 弹出 3，输出 3；删边 3→4，in[4]=0 → q={4}
④ 弹出 4，输出 4；结束
答案: 1 2 3 4
```

**流程伪代码**：

```
for each v: if in[v] == 0: q.push(v)
while q not empty:
    u = q.front(); q.pop()
    ans.push_back(u)
    for each edge (u,v):
        if (--in[v] == 0) q.push(v)
if ans.size() < n: 图中有环
```

**复杂度**：每个点入队一次、出队一次，每条边被处理一次，

$$
T=O(n+m),\qquad S=O(n+m).
$$

**字典序最小拓扑序**：把队列换成小根堆（`priority_queue<int, vector<int>, greater<int>>`），每次取编号最小的入度 0 点，即得字典序最小的拓扑序。

### 1.4 DFS 求拓扑序

**思路**：对图做 DFS，记录每个点的**完成时间**（回溯时的时间戳）。拓扑序 = 按完成时间**从大到小**排列。

直觉：若存在边 $u\to v$，则 DFS 中要么 $v$ 在 $u$ 的子树里（$v$ 先完成），要么 $v$ 早已访问完（也先完成）。因此 $u$ 的完成时间总是晚于 $v$，按完成时间逆序排列即满足 $u$ 在 $v$ 前。

```
DFS 从 1 开始：
  dfs(1) → dfs(2) → dfs(4) 完成顺序: 4, 2
  dfs(3) → 3 已完成
  1 完成最后
完成时间逆序: 1, 3, 2, 4   ← 一个合法拓扑序
```

**环检测**：DFS 中用三色标记（0=未访问，1=在栈中，2=已完成）。若访问到「在栈中」的点，说明存在环。

**适用场景**：需要同时求其它 DFS 信息（如强连通分量、割点）时更方便；但递归深度大时有爆栈风险。

### 1.5 DAG 上 DP

拓扑序的最大用途是给 DP 提供**无后效性的计算顺序**。

**一般步骤**：

1. 建图并求出拓扑序；
2. 按拓扑序依次处理每个点，用它的 DP 值更新所有后继；
3. 答案为所有终点的最优值。

**经典模型**：

| 模型 | 状态设计 | 转移 |
|---|---|---|
| 最长路 | $f[v]$：以 $v$ 结尾的最长路 | $f[v]=\max(f[v],f[u]+w)$ |
| 路径计数 | $f[v]$：从 $s$ 到 $v$ 的路径数 | $f[v]=\sum f[u]$ |
| 关键路径 | $f[v]$ 最早开始，$g[v]$ 最晚开始 | 正反两遍拓扑 |
| 树上/图上依赖背包 | 依赖关系建图，按拓扑序做背包 | 见背包 DP |

---

## 2. 模板代码

### 2.1 Kahn 算法（邻接表 + 队列）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
vector<int> g[MAXN];
int indeg[MAXN];
int n, m;

// 返回 true 表示是 DAG，拓扑序存入 ans；false 表示有环
bool topoSort(vector<int> &ans) {
    queue<int> q;
    for (int i = 1; i <= n; ++i)
        if (indeg[i] == 0) q.push(i);
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        ans.push_back(u);
        for (int v : g[u]) {
            if (--indeg[v] == 0) q.push(v);   // 入度减到 0 才能入队
        }
    }
    return (int)ans.size() == n;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; ++i) {
        int u, v; scanf("%d%d", &u, &v);
        g[u].push_back(v);
        ++indeg[v];
    }
    vector<int> ans;
    if (!topoSort(ans)) {
        puts("有环");
    } else {
        for (int x : ans) printf("%d ", x);
        puts("");
    }
    return 0;
}
```

### 2.2 字典序最小拓扑序

```cpp
bool topoSortMin(vector<int> &ans) {
    priority_queue<int, vector<int>, greater<int>> pq;   // 小根堆
    for (int i = 1; i <= n; ++i)
        if (indeg[i] == 0) pq.push(i);
    while (!pq.empty()) {
        int u = pq.top(); pq.pop();
        ans.push_back(u);
        for (int v : g[u])
            if (--indeg[v] == 0) pq.push(v);
    }
    return (int)ans.size() == n;
}
```

### 2.3 DFS 版拓扑排序（三色标记 + 环检测）

```cpp
const int WHITE = 0, GRAY = 1, BLACK = 2;
int color[MAXN];
vector<int> order;          // 完成顺序
bool hasCycle = false;

void dfs(int u) {
    color[u] = GRAY;        // 进入：标记为「在栈中」
    for (int v : g[u]) {
        if (color[v] == GRAY) { hasCycle = true; return; }   // 回边 => 有环
        if (color[v] == WHITE) dfs(v);
    }
    color[u] = BLACK;       // 离开：标记为「已完成」
    order.push_back(u);     // 完成时间递增
}

// 主函数中：
// for (int i = 1; i <= n; ++i) if (!color[i]) dfs(i);
// if (!hasCycle) reverse(order.begin(), order.end());  // 逆序即为拓扑序
```

### 2.4 DAG 上最长路

```cpp
// 按拓扑序 DP；dist 初始为 -INF，源点设为 0
long long dist[MAXN];

long long dagLongestPath(int s, const vector<int> &topo) {
    fill(dist, dist + n + 1, LLONG_MIN / 2);
    dist[s] = 0;
    for (int u : topo) {
        if (dist[u] == LLONG_MIN / 2) continue;   // 源点不可达
        for (auto &e : gw[u]) {                   // gw[u]: (to, w)
            dist[e.to] = max(dist[e.to], dist[u] + e.w);
        }
    }
    long long best = 0;
    for (int i = 1; i <= n; ++i) best = max(best, dist[i]);
    return best;
}
```

### 2.5 关键路径（AOE 网）

```cpp
// f[v]：最早开始时间；g[v]：最晚开始时间；d[u][v] 为活动时长
// 正向拓扑求 f，反向拓扑求 g；f[u] == g[u] 的点为关键节点
void criticalPath(int n, const vector<int> &topo) {
    fill(f, f + n + 1, 0);
    for (int u : topo)
        for (auto &e : gw[u])
            f[e.to] = max(f[e.to], f[u] + e.w);
    long long T = *max_element(f + 1, f + n + 1);
    fill(g, g + n + 1, T);
    for (int i = n - 1; i >= 0; --i) {
        int u = topo[i];
        for (auto &e : gw[u])
            g[u] = min(g[u], g[e.to] - e.w);
    }
}
```

---

## 3. 经典例题

### 例题 1：杂务（洛谷 P1113）

**题意**：有 $n$ 项杂务，第 $i$ 项耗时 $t_i$，并给出若干前置杂务。求完成所有杂务的最短总时间（可以并行做互不依赖的杂务）。

**思路**：把依赖关系建成 DAG，$f[v]$ 表示「完成 $v$ 及其所有前置」的最早时间，则

$$
f[v]=t_v+\max_{u\to v}f[u].
$$

按拓扑序转移即可，答案为 $\max_v f[v]$。

```cpp
int t[MAXN];
long long f[MAXN];
long long ans = 0;

bool topoDP() {
    queue<int> q;
    for (int i = 1; i <= n; ++i)
        if (indeg[i] == 0) { q.push(i); f[i] = t[i]; }
    int cnt = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop(); ++cnt;
        ans = max(ans, f[u]);
        for (int v : g[u]) {
            f[v] = max(f[v], f[u] + t[v]);   // 所有前置都处理完前 f[v] 只会变大
            if (--indeg[v] == 0) q.push(v);
        }
    }
    return cnt == n;
}
```

**复杂度**：$O(n+m)$。

### 例题 2：最大食物链计数（洛谷 P4017）

**题意**：给定 $n$ 个点 $m$ 条边的 DAG 食物链（生产者入度为 0，顶级消费者出度为 0），求食物链总数，对 80112002 取模。

**思路**：设 $f[v]$ 为从任意生产者到 $v$ 的路径数。初始所有入度为 0 的点 $f=1$，按拓扑序转移 $f[v]=\sum_{u\to v}f[u]$（取模），答案为所有出度为 0 的点的 $f$ 之和。

```cpp
const int MOD = 80112002;
long long f[MAXN];
int outdeg[MAXN];

long long countFoodChains() {
    queue<int> q;
    for (int i = 1; i <= n; ++i)
        if (indeg[i] == 0) { q.push(i); f[i] = 1; }
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : g[u]) {
            f[v] = (f[v] + f[u]) % MOD;
            if (--indeg[v] == 0) q.push(v);
        }
    }
    long long res = 0;
    for (int i = 1; i <= n; ++i)
        if (outdeg[i] == 0) res = (res + f[i]) % MOD;
    return res;
}
```

**复杂度**：$O(n+m)$。

### 例题 3：课程表（LeetCode 207）

**题意**：共有 $n$ 门课，给定先修关系数组 `prerequisites`，判断是否可能修完所有课程。

**思路**：这是纯粹的「判断 DAG」。直接跑 Kahn，若拓扑序长度为 $n$ 则可行，否则存在环。

```cpp
bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> g(numCourses);
    vector<int> indeg(numCourses, 0);
    for (auto &p : prerequisites) {   // p[1] -> p[0]
        g[p[1]].push_back(p[0]);
        ++indeg[p[0]];
    }
    queue<int> q;
    for (int i = 0; i < numCourses; ++i)
        if (indeg[i] == 0) q.push(i);
    int cnt = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop(); ++cnt;
        for (int v : g[u])
            if (--indeg[v] == 0) q.push(v);
    }
    return cnt == numCourses;
}
```

**复杂度**：$O(n+m)$。

### 例题 4：最长路（洛谷 P1807）

**题意**：给定 $n$ 个点 $m$ 条边的 DAG，边权可正可负，求 1 到 $n$ 的最长路；若不可达输出 $-1$。$n\le 1500$，$m\le 5\times10^4$。

**思路**：DAG 上最长路按拓扑序 DP。$f[1]=0$，其余为 $-\infty$，转移 $f[v]=\max(f[v],f[u]+w)$。注意**边权可为负**，不能用「不可达就跳过」之外的初始化错误。

```cpp
long long f[MAXN];

long long longestPath(int n, int m) {
    fill(f, f + n + 1, LLONG_MIN / 2);
    f[1] = 0;
    queue<int> q;
    vector<int> in2(indeg, indeg + n + 1);
    for (int i = 1; i <= n; ++i) if (!in2[i]) q.push(i);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        if (f[u] > LLONG_MIN / 4)
            for (auto &e : gw[u])
                f[e.to] = max(f[e.to], f[u] + e.w);
        for (auto &e : gw[u])
            if (--in2[e.to] == 0) q.push(e.to);
    }
    return f[n] < LLONG_MIN / 4 ? -1 : f[n];
}
```

**复杂度**：$O(n+m)$。

---

## 4. 易错点与练习建议

### 易错点

1. **有环图仍输出部分结果**：必须检查 `ans.size() == n`，否则输出的是残缺序列。
2. **入度数组被破坏**：若需要多次拓扑排序（如 Tarjan 缩点后），要么备份 `indeg`，要么每次重新计算。
3. **重边导致入度多加**：重边要按实际边数计入度，Kahn 中也会被减多次，一般无碍；但若用「边去重」的邻接矩阵要注意。
4. **自环**：自环使该点入度永远不为 0，拓扑排序自然检测出环，无需特判，但要知道原因。
5. **字典序最小 ≠ 编号小的优先 DFS**：必须用小根堆维护「当前所有入度为 0 的点」，而不是优先访问小编号邻居的 DFS。
6. **DFS 拓扑序忘记 reverse**：`order` 是**完成顺序**，拓扑序是它的**逆序**。
7. **DFS 三色标记漏判**：只用 `vis` 布尔数组无法区分「在栈中」和「已完成」，检测不出环。
8. **DAG 上 DP 的初始化**：求最长路初值设 $-\infty$，求最短路初值设 $+\infty$；若求方案数，入度为 0 的点应初始化为 1。
9. **拓扑序中「层」的概念**：拓扑序不唯一，不要假设题目给的序列一定和你的算法输出一致；涉及「同一层内可以并行」的问题要额外处理。
10. **递归深度**：DFS 版在链状图会爆栈，$n=10^5$ 以上建议用 Kahn。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 B3644 拓扑排序模板 | Kahn 基础 |
| 入门 | LeetCode 207 课程表 | DAG 判定 |
| 入门 | 洛谷 P1113 杂务 | 拓扑序上 DP |
| 提高 | 洛谷 P4017 最大食物链计数 | 路径计数 DP |
| 提高 | 洛谷 P1807 最长路 | DAG 最长路 |
| 提高 | 洛谷 P1983 车站分级 | 拓扑分层 / 差分约束思想 |
| 提高 | 洛谷 P1137 旅行计划 | 拓扑 + 最长链 |
| 进阶 | 洛谷 P3243 [HNOI2015] 菜肴制作 | 反向建图 + 贪心（字典序最大） |
| 进阶 | 洛谷 P2597 [ZJOI2012] 灾难 | 支配树 / 拓扑 + LCA |

学习建议：拓扑排序本身只有二十行，但要理解「为什么队列为空时仍有剩余点就意味着有环」。把 DAG 上 DP 当成一个固定套路练熟：**先拓扑，再转移，最后统计答案**。很多看起来复杂的题（如依赖背包、关键路径）本质上都是这个套路。
