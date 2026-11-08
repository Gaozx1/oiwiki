# 强连通分量

> 一句话概括：有向图中「任意两点互相可达」的极大点集叫**强连通分量（SCC）**；Tarjan 用一趟 DFS 在 $O(n+m)$ 内求出所有 SCC，缩点后得到一张 DAG。

强连通分量是解决有向图问题的核心工具：只要把每个 SCC 缩成一个点，就把任意有向图转化成了 DAG，于是拓扑排序、DAG 上 DP 全都能用了。本节重点讲清 Tarjan 算法中 `dfn`/`low` 的含义与更新规则，并进行严格的正确性说明。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 强连通与强连通分量
   - 1.2 Tarjan 算法的核心变量
   - 1.3 更新规则与正确性
   - 1.4 Kosaraju 算法
   - 1.5 缩点
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 强连通与强连通分量

对**有向图** $G=(V,E)$：

- 若 $u$ 能到达 $v$ 且 $v$ 能到达 $u$，称 $u,v$ **强连通**。
- 若一个点集 $C$ 中任意两点都强连通，称 $C$ 强连通。
- 极大的强连通点集称为**强连通分量（Strongly Connected Component, SCC）**。

「极大」意味着再加入任何一个点，这个集合就不再强连通。

```
有向图示例：
    1 → 2 → 3
    ↑   ↓   ↓
    4 ← 5   6

SCC: {1,2,4,5}（1→2→5→4→1 成环）
     {3}        （3 出边到 6，但 6 回不来）
     {6}
```

**重要结论**：把每个 SCC 缩成一个点后，得到的一定是 **DAG**。因为若缩点后仍有环，环上所有点原本就互相可达，应该属于同一个 SCC，与「极大」矛盾。

**SCC 与 DAG 的关系**：任意有向图都可以看作「若干 SCC 组成的 DAG」，SCC 之间按拓扑顺序排列。

### 1.2 Tarjan 算法的核心变量

Tarjan 是**一趟 DFS**，每个点只访问一次。维护两个数组：

| 变量 | 含义 |
|---|---|
| `dfn[u]` | 时间戳：点 $u$ 第一次被访问的顺序编号（从 1 递增） |
| `low[u]` | 从 $u$ 出发，沿 DFS 树往下走，最多经过**一条非树边**（回边或横叉边），能追溯到的最小的 `dfn` |

同时维护一个栈 `st`，存放「当前尚未确定归属的 SCC 的点」。

**判定 SCC 的根**：若 `low[u] == dfn[u]`，则 $u$ 是当前 SCC 的「根」，此时栈中从 $u$ 到栈顶的所有点属于同一个 SCC。

**直观理解**：`low[u]` 记录的是 $u$ 这棵子树「能向上爬到的最早祖先」。若爬不上去（`low[u] == dfn[u]`），说明 $u$ 的子树形成了一个封闭的环，可以独立成一个 SCC。

```
DFS 树：
        1 (dfn=1)
       /
      2 (dfn=2)
     /
    3 (dfn=3)
     \→ 1  ← 这条回边让 low[3]=1，low[2]=1，low[1]=1
  则 {1,2,3} 是一个 SCC，根为 1
```

### 1.3 更新规则与正确性

DFS 过程中，对点 $u$ 的每条出边 $u\to v$：

1. **$v$ 未被访问**（树边）：先 `dfs(v)`，回溯后 `low[u] = min(low[u], low[v])`。
2. **$v$ 已访问且在栈中**（回边或指向同 SCC 的边）：`low[u] = min(low[u], dfn[v])`。
3. **$v$ 已访问且不在栈中**：说明 $v$ 属于已经确定的另一个 SCC，不能更新 $low[u]$。

注意第 2 条是 `dfn[v]` 而不是 `low[v]`：因为 `low[v]` 可能指向 $v$ 子树之外的更早的点，而我们要找的是**能从 $u$ 一步走到的祖先**。这一点是 Tarjan 最容易写错的地方。

**回溯时判定**：`dfs(u)` 结束前，若 `low[u] == dfn[u]`，则不断弹栈直到弹出 $u$，这些点构成一个 SCC。

**正确性要点**：

- 若 `low[u] == dfn[u]`，$u$ 的子树中没有边指向 $u$ 的祖先，所以 $u$ 及其子树部分点无法与祖先互通，必然形成独立的 SCC。
- 若 `low[u] < dfn[u]`，说明存在一条回边指向上面的祖先，$u$ 会与祖先合并到同一个 SCC，暂时不出栈。

**复杂度**：每个点入栈/出栈一次，每条边被检查一次，

$$
T=O(n+m),\qquad S=O(n).
$$

### 1.4 Kosaraju 算法

Kosaraju 更直观，但需要跑两遍 DFS：

1. 在原图上做 DFS，记录每个点的**完成时间**；
2. 把原图**反建**（所有边反向）；
3. 按完成时间**从大到小**的顺序，在反图上做 DFS，每次 DFS 访问到的点集就是一个 SCC。

**正确性直觉**：原图中「最晚完成的点」一定属于最上游的 SCC（没有出边能到达其它未访问 SCC）；在反图上从它出发只能到达它自己的 SCC。

**复杂度**：$O(n+m)$，但需要存两倍的边，常数比 Tarjan 大。

### 1.5 缩点

求出所有 SCC 后，把每个 SCC 缩成一个超级点：

- 若原图中存在边 $u\to v$，且 $belong[u]\ne belong[v]$，则在缩点图中加边 $belong[u]\to belong[v]$。
- 缩点图是 DAG，可拓扑排序或直接按 SCC 编号处理（Tarjan 的出栈顺序天然是**逆拓扑序**）。

**常见应用**：

| 问题 | 做法 |
|---|---|
| 有向图最长路 | SCC 内权值可累加，缩点后在 DAG 上 DP |
| 2-SAT | 建蕴含图，同一变量的两个赋值点不能在同一个 SCC |
| 最小点基 | 缩点后统计入度为 0 的 SCC 个数 |
| 强连通分量计数 | Tarjan 出栈计数 |

---

## 2. 模板代码

### 2.1 Tarjan 求 SCC

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
vector<int> g[MAXN];
int dfn[MAXN], low[MAXN], timer = 0;
int stk[MAXN], top = 0;
bool inStack[MAXN];
int belong[MAXN], sccCnt = 0;
vector<int> scc[MAXN];              // 每个 SCC 包含的点

void tarjan(int u) {
    dfn[u] = low[u] = ++timer;
    stk[++top] = u;
    inStack[u] = true;
    for (int v : g[u]) {
        if (!dfn[v]) {              // 树边：v 未访问
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (inStack[v]) {    // 回边/横叉边：v 还在栈中
            low[u] = min(low[u], dfn[v]);   // 注意是 dfn[v]！
        }
        // v 已访问且不在栈中：属于别的 SCC，忽略
    }
    if (low[u] == dfn[u]) {         // u 是 SCC 的根
        ++sccCnt;
        while (true) {
            int v = stk[top--];
            inStack[v] = false;
            belong[v] = sccCnt;
            scc[sccCnt].push_back(v);
            if (v == u) break;
        }
    }
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; ++i) {
        int u, v; scanf("%d%d", &u, &v);
        g[u].push_back(v);
    }
    for (int i = 1; i <= n; ++i)
        if (!dfn[i]) tarjan(i);
    printf("SCC 个数: %d\n", sccCnt);
    for (int c = 1; c <= sccCnt; ++c) {
        printf("SCC %d:", c);
        for (int x : scc[c]) printf(" %d", x);
        puts("");
    }
    return 0;
}
```

### 2.2 缩点建 DAG

Tarjan 求得 `belong[]` 之后，枚举原图每条边，若两端不属于同一个 SCC 就在缩点图中加边，并用 `set` 去重、统计入度。

```cpp
vector<int> dag[MAXN];
int indegDag[MAXN];

void buildDAG(int n, const vector<pair<int,int>> &edges) {
    set<pair<int,int>> S;                  // 同一对 SCC 之间可能有多条边，需要去重
    for (auto &e : edges) {
        int u = belong[e.first], v = belong[e.second];
        if (u != v) S.insert({u, v});
    }
    for (auto &p : S) {
        dag[p.first].push_back(p.second);
        ++indegDag[p.second];
    }
}
```

### 2.3 SCC + DAG 上最长路（点权版）

设 $val[u]$ 为原图点权。先按 2.1 节求出 `belong[]` 与 `sccCnt`，用 `sccVal[c]` 累加每个 SCC 内部的点权，再按 2.2 节建 DAG，最后在 DAG 上按拓扑序做一次最长路 DP。

```cpp
long long val[MAXN];        // 原图点权
long long sccVal[MAXN];     // 缩点后权值
long long dp[MAXN];

long long solveAfterTarjan(int n) {
    // 1. 每个 SCC 的权值 = 内部点权之和（belong 已由 Tarjan 求出）
    for (int i = 1; i <= n; ++i) sccVal[belong[i]] += val[i];
    // 2. 建 DAG 与入度（调用 2.2 节 buildDAG）
    // 3. 拓扑序 DP
    queue<int> q;
    for (int c = 1; c <= sccCnt; ++c) {
        dp[c] = sccVal[c];                 // 至少包含自己
        if (indegDag[c] == 0) q.push(c);
    }
    long long ans = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        ans = max(ans, dp[u]);
        for (int v : dag[u]) {
            dp[v] = max(dp[v], dp[u] + sccVal[v]);
            if (--indegDag[v] == 0) q.push(v);
        }
    }
    return ans;
}
```

### 2.4 Kosaraju（对比参考）

```cpp
vector<int> g[MAXN], rg[MAXN];
bool vis[MAXN];
vector<int> order;

void dfs1(int u) {
    vis[u] = true;
    for (int v : g[u]) if (!vis[v]) dfs1(v);
    order.push_back(u);              // 完成顺序
}

void dfs2(int u, int c) {
    belong[u] = c;
    for (int v : rg[u]) if (!belong[v]) dfs2(v, c);
}

int kosaraju(int n) {
    for (int i = 1; i <= n; ++i) if (!vis[i]) dfs1(i);
    int c = 0;
    for (int i = n - 1; i >= 0; --i) {       // 逆序遍历完成顺序
        int u = order[i];
        if (!belong[u]) dfs2(u, ++c);
    }
    return c;
}
```

---

## 3. 经典例题

### 例题 1：强连通分量模板（洛谷 P2863 / USACO06JAN The Cow Prom）

**题意**：给定 $n$ 个点 $m$ 条边的有向图，求大小 $>1$ 的 SCC 个数。

**思路**：Tarjan 求出所有 SCC 后统计 `scc[c].size() > 1` 的个数。

```cpp
int ans = 0;
for (int c = 1; c <= sccCnt; ++c)
    if (scc[c].size() > 1) ++ans;
printf("%d\n", ans);
```

**复杂度**：$O(n+m)$。

### 例题 2：受欢迎的牛（洛谷 P2341 / USACO03FALL）

**题意**：$n$ 头牛，$m$ 条「A 认为 B 受欢迎」的有向关系，这种关系具有传递性。求被所有牛都欢迎的牛的数量。

**思路**：缩点后统计每个 SCC 的出度。**若缩点后恰有一个出度为 0 的 SCC，则答案是它的大小；否则答案为 0**。

为什么？若有两个以上出度为 0 的 SCC，它们互相无法到达，不可能有牛被所有人欢迎。若只有一个，由于 DAG 中任意点都能沿出边走到某个出度为 0 的点，且该点唯一，所以它被所有点可达。

```cpp
// Tarjan 求 belong 后：
vector<int> outdeg(sccCnt + 1, 0);
for (int u = 1; u <= n; ++u)
    for (int v : g[u])
        if (belong[u] != belong[v]) ++outdeg[belong[u]];
int zeroCnt = 0, id = -1;
for (int c = 1; c <= sccCnt; ++c)
    if (outdeg[c] == 0) { ++zeroCnt; id = c; }
if (zeroCnt == 1) printf("%d\n", (int)scc[id].size());
else puts("0");
```

**复杂度**：$O(n+m)$。

### 例题 3：间谍网络（洛谷 P1262）

**题意**：$n$ 个间谍，部分可被收买（花费 $w_i$），另有 $m$ 条「A 掌握 B 的证据」的有向关系，被收买的间谍会供出他掌握的所有间谍。求控制所有间谍的最小花费；若不可行输出最小无法控制的间谍编号。

**思路**：先缩点。一个 SCC 内只要收买任意一个能被收买的点，整个 SCC 都能被控制，因此 SCC 的花费 = 内部最小可收买花费。缩点后在 DAG 上，只有**入度为 0 的 SCC** 需要付钱（其它 SCC 会被上游供出）。若某个入度 0 的 SCC 内没有任何可收买的点，则不可行。

```cpp
// 读入后 Tarjan 缩点
const int INF = 0x3f3f3f3f;
vector<int> cost(sccCnt + 1, INF);
for (int i = 1; i <= n; ++i)
    if (w[i] < INF) cost[belong[i]] = min(cost[belong[i]], w[i]);
vector<int> indeg(sccCnt + 1, 0);
for (int u = 1; u <= n; ++u)
    for (int v : g[u])
        if (belong[u] != belong[v]) ++indeg[belong[v]];
long long ans = 0;
int bad = -1;
for (int c = 1; c <= sccCnt; ++c)
    if (indeg[c] == 0) {
        if (cost[c] == INF) {
            // 该 SCC 内取编号最小的点
            int mn = INF;
            for (int x : scc[c]) mn = min(mn, x);
            bad = (bad == -1) ? mn : min(bad, mn);
        } else ans += cost[c];
    }
if (bad != -1) printf("NO\n%d\n", bad);
else printf("YES\n%lld\n", ans);
```

**复杂度**：$O(n+m)$。

### 例题 4：缩点 + DAG 最长路（洛谷 P3387 缩点）

**题意**：给定 $n$ 个点 $m$ 条边的有向图，每个点有点权，求一条路径使经过的点权之和最大（同一点只算一次）。

**思路**：同一 SCC 内的点可以互相到达，因此一条最优路径可以「吃干」整个 SCC 的点权。缩点后 SCC 权值 = 内部点权之和，在 DAG 上求最长路，完整代码即 2.3 节的 `solveAfterTarjan`，答案返回所有 `dp[c]` 的最大值。

**复杂度**：$O(n+m)$。

---

## 4. 易错点与练习建议

### 易错点

1. **`low[u] = min(low[u], dfn[v])` 写成 `low[v]`**：这是 Tarjan 最经典的错误，会在某些图上把不该合并的 SCC 合并。
2. **忘记判断 `inStack[v]`**：已访问但不在栈中的点属于已经确定的 SCC，不能再更新 `low`。
3. **多组数据不清空**：`dfn`、`low`、`timer`、`top`、`sccCnt`、`inStack` 都要清零。
4. **栈用 STL `stack`**：Tarjan 需要访问栈内元素和弹栈到指定元素，用数组模拟栈 `stk[++top]` 更方便。
5. **缩点时忘记去重**：同一对 SCC 之间可能有多条边，入度会重复累加，必须用 `set` 去重或用邻接矩阵判重。
6. **SCC 编号与拓扑序**：Tarjan 的出栈顺序是**逆拓扑序**（先出栈的是下游 SCC），若需要正序可倒序使用；但更稳妥的是重新拓扑排序。
7. **有向图与无向图混用**：SCC 只针对有向图；无向图对应的是「边双连通分量」，求法不同（见割点与桥一章）。
8. **递归爆栈**：$10^5$ 级别的链状图递归 Tarjan 可能爆栈，Windows 下尤其要注意。可以手动开栈或写非递归版本。
9. **2-SAT 中 SCC 编号**：判定条件为 `belong[x] == belong[x+n]` 时无解；赋值方案是「取编号较大的那个」或按逆拓扑序，要统一。
10. **点权和溢出**：$n\times w$ 可能超过 `int`，用 `long long`。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P2863 [USACO06JAN] The Cow Prom S | Tarjan 模板 |
| 入门 | 洛谷 P2341 受欢迎的牛 | 缩点 + 出度统计 |
| 提高 | 洛谷 P3387 缩点 | 缩点 + DAG 最长路 |
| 提高 | 洛谷 P1262 间谍网络 | 缩点 + 入度为 0 的 SCC |
| 提高 | 洛谷 P2746 [USACO5.3] 校园网 | 缩点 + 入度/出度 |
| 提高 | 洛谷 P1073 最优贸易 | 缩点 / 分层图 |
| 进阶 | 洛谷 P4782 2-SAT 问题 | SCC 判定 2-SAT |
| 进阶 | 洛谷 P4819 [中山市选] 杀人游戏 | SCC + 概率 |

学习建议：Tarjan 的模板必须背到能盲写，重点是理解 `dfn`/`low` 的物理意义和三种边的处理。建议先在纸上手动模拟一个小图的完整 DFS 过程，把每个点的 `dfn`、`low`、栈内容都写出来，比看十遍代码都有效。
