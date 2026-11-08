# 二分图

> 一句话概括：能把点集分成两个内部无边的小组的图叫**二分图**；它能用染色判定，能求最大匹配（匈牙利算法），并且在 $n$ 个点、$m$ 条边的图上用 $O(nm)$ 或 $O(m\sqrt{n})$ 解决大量「配对」问题。

二分图是图论里少数既有漂亮理论（König 定理、Hall 定理）又有极强实战价值（匹配模型）的知识点。凡是题目中出现「两种对象互相配对」「每人只能选一个」「冲突分配」，大概率是二分图匹配。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 二分图的定义与判定
   - 1.2 最大匹配与增广路
   - 1.3 匈牙利算法
   - 1.4 König 定理与最小点覆盖
   - 1.5 多重匹配与网络流建模
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 二分图的定义与判定

**定义**：若图 $G=(V,E)$ 的点集可以分成两个不相交的集合 $X,Y$，使得**每条边的两个端点分别属于 $X$ 和 $Y$**（即集合内部没有边），则称 $G$ 为**二分图（Bipartite Graph）**，$X,Y$ 称为两部。

```
二分图:
    X 部: 1  2  3
          |\/|\/|
    Y 部: 4  5  6
（每条边都从 X 连到 Y）

非二分图（三角形）:
      1
     / \
    2---3        ← 奇环，无法二分
```

**判定定理**：图是二分图 $\iff$ 图中**不存在奇环**（长度为奇数的环）。

**判定方法：染色（BFS/DFS）**

- 任选一点染成 1 号颜色；
- 对每条边 $(u,v)$，若 $v$ 未染色则染成与 $u$ 相反的颜色；若已染色且与 $u$ 同色，则存在奇环，不是二分图。
- 需要对每个连通块都做一次染色。

复杂度 $O(n+m)$。

**小技巧**：二分图判定可以用**并查集**（扩展域）实现——把每个点拆成「同类」和「异类」两个并查集。但染色法更简单，优先使用。

### 1.2 最大匹配与增广路

**匹配（Matching）**：一组没有公共端点的边。**最大匹配**：边数最多的匹配。**完美匹配**：所有点都被匹配（要求 $|X|=|Y|$ 且存在）。

**增广路（Augmenting Path）**：一条起点和终点都是**未匹配点**、且边**交替**为「非匹配边—匹配边—非匹配边—…—非匹配边」的路径。

**核心定理（Berge 定理）**：匹配 $M$ 是最大匹配 $\iff$ 不存在增广路。

> 为什么？若存在增广路，把路径上「非匹配边」和「匹配边」互换，匹配数增加 1；反之若匹配不是最大，两个匹配的对称差中必有一条增广路。

**增广路示意**：

```
未匹配点 a ---(非匹配)--- b ===(匹配)=== c ---(非匹配)--- d 未匹配点
沿这条路径翻转匹配关系：
a -- b   变成   a    b -- c   变成   b    c -- d
匹配边数 1 → 2，多了 1 条
```

### 1.3 匈牙利算法

**思路**：对左部每个点依次尝试找增广路。用 DFS：

```
try_kuhn(u):
    for v in adj[u]:
        if vis[v] continue          // 本轮已尝试过
        vis[v] = true
        if matchR[v] == -1 or try_kuhn(matchR[v]):
            matchR[v] = u
            return true
    return false
```

**关键点**：`vis[v]` 是「本轮 DFS 中右部点 $v$ 是否已被访问」，每次为新的左部点找匹配前要清空。`matchR[v]` 记录右部点 $v$ 当前匹配的左部点。

**复杂度**：左部 $n$ 个点，每个点做一次 DFS，每次 DFS 最坏访问 $m$ 条边，

$$
T=O(nm).
$$

对于 $n,m\le 10^3$ 的题完全够用。$n,m\le 10^5$ 时需要用 Hopcroft-Karp（$O(m\sqrt{n})$）或网络流。

**匈牙利算法的正确性**：每次尝试为左部点 $u$ 找增广路，若找到则匹配数 +1；若找不到，由 Berge 定理当前匹配已是最大。注意算法依赖于「已匹配的左部点可以被重新调整」，DFS 里的递归 `try_kuhn(matchR[v])` 就是让原来的主人让位。

### 1.4 König 定理与最小点覆盖

**点覆盖**：一组点，使每条边至少有一个端点被选中。**最小点覆盖**：点数最少的点覆盖。

**König 定理**：在二分图中，

$$
\text{最大匹配数} = \text{最小点覆盖数}.
$$

这条定理把「选最少的点覆盖所有边」转化为「求最大匹配」，是二分图最常用的转化之一。

**最大独立集**：一组点，两两之间没有边。在二分图中有

$$
\text{最大独立集} = n - \text{最小点覆盖} = n - \text{最大匹配}.
$$

**最小路径覆盖**（DAG 上）：把每个点拆成「出点」和「入点」，原边 $u\to v$ 连 $u_{out}\to v_{in}$，则

$$
\text{最小路径覆盖数} = n - \text{最大匹配}.
$$

### 1.5 多重匹配与网络流建模

二分图匹配是网络流的特例：源点 $S$ 向左部每个点连容量 1 的边，左部向右部连容量 1 的边，右部向汇点 $T$ 连容量 1 的边，最大流即最大匹配。

当有「每个左部点可匹配 $k$ 个」或「带权匹配（最大权匹配）」时，网络流/费用流更灵活（见网络流一章）。

---

## 2. 模板代码

### 2.1 染色判定二分图

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
vector<int> g[MAXN];
int color[MAXN];                 // 0 未染色，1/2 两种颜色

bool isBipartite(int n) {
    for (int s = 1; s <= n; ++s) {
        if (color[s]) continue;
        queue<int> q;
        q.push(s);
        color[s] = 1;
        while (!q.empty()) {
            int u = q.front(); q.pop();
            for (int v : g[u]) {
                if (!color[v]) {
                    color[v] = 3 - color[u];    // 1 <-> 2
                    q.push(v);
                } else if (color[v] == color[u]) {
                    return false;               // 同色相邻 => 奇环
                }
            }
        }
    }
    return true;
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; ++i) {
        int u, v; scanf("%d%d", &u, &v);
        g[u].push_back(v);
        g[v].push_back(u);
    }
    puts(isBipartite(n) ? "Yes" : "No");
    return 0;
}
```

### 2.2 匈牙利算法求最大匹配

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1005;
vector<int> g[MAXN];             // 左部 -> 右部
int matchR[MAXN];                // 右部点匹配的左部点，-1 表示未匹配
bool vis[MAXN];                  // 本轮 DFS 中右部点是否已访问

bool tryKuhn(int u) {
    for (int v : g[u]) {
        if (vis[v]) continue;
        vis[v] = true;
        if (matchR[v] == -1 || tryKuhn(matchR[v])) {
            matchR[v] = u;           // 找到增广路，翻转匹配
            return true;
        }
    }
    return false;
}

int maxMatching(int n) {
    fill(matchR, matchR + MAXN, -1);
    int res = 0;
    for (int u = 1; u <= n; ++u) {   // 对每个左部点尝试增广
        memset(vis, 0, sizeof(vis));
        if (tryKuhn(u)) ++res;
    }
    return res;
}

int main() {
    int n, m, e;
    scanf("%d%d%d", &n, &m, &e);     // n 左部点数，m 右部点数，e 边数
    for (int i = 1; i <= e; ++i) {
        int u, v; scanf("%d%d", &u, &v);
        g[u].push_back(v);
    }
    printf("%d\n", maxMatching(n));
    return 0;
}
```

### 2.3 Hopcroft-Karp（$O(m\sqrt n)$）

```cpp
const int INF = 0x3f3f3f3f;
int matchL[MAXN], matchR[MAXN], dist[MAXN];

bool bfs(int n) {
    queue<int> q;
    for (int u = 1; u <= n; ++u) {
        if (matchL[u] == -1) { dist[u] = 0; q.push(u); }
        else dist[u] = INF;
    }
    bool found = false;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : g[u]) {
            int w = matchR[v];
            if (w == -1) found = true;             // 找到增广路
            else if (dist[w] == INF) {
                dist[w] = dist[u] + 1;
                q.push(w);
            }
        }
    }
    return found;
}

bool dfsHK(int u) {
    for (int v : g[u]) {
        int w = matchR[v];
        if (w == -1 || (dist[w] == dist[u] + 1 && dfsHK(w))) {
            matchL[u] = v;
            matchR[v] = u;
            return true;
        }
    }
    dist[u] = INF;
    return false;
}

int hopcroftKarp(int n) {
    fill(matchL, matchL + MAXN, -1);
    fill(matchR, matchR + MAXN, -1);
    int res = 0;
    while (bfs(n)) {
        for (int u = 1; u <= n; ++u)
            if (matchL[u] == -1 && dfsHK(u)) ++res;
    }
    return res;
}
```

### 2.4 二分图最大独立集（König 定理应用）

```cpp
// 最大独立集 = n - 最大匹配
int maxIndependentSet(int n, int m) {
    int match = maxMatching(n);      // 二分图最大匹配
    return n + m - match;            // 总点数 - 最大匹配
}
```

### 2.5 DAG 最小路径覆盖

```cpp
// 把每个点 i 拆成 i(出) 和 i+n(入)
// 对原边 u -> v，建边 u -> v+n
// 答案 = n - 最大匹配
int minPathCover(int n, const vector<pair<int,int>> &edges) {
    for (auto &e : edges) g[e.first].push_back(e.second + n);
    return n - maxMatching(n);
}
```

---

## 3. 经典例题

### 例题 1：二分图判定（洛谷 P1330 封锁阳光大学）

**题意**：给定 $n$ 个点 $m$ 条边的无向图，在点上放「河蟹」，每个河蟹能封锁与它相邻的所有边，但**相邻两点不能同时放河蟹**。求最少放多少只河蟹能封锁所有边；若不可能输出 `Impossible`。

**思路**：先判定是否为二分图。若是，则每个连通块内可以任选一种颜色放河蟹（两色互换），取两种颜色中点数较少的即可。

```cpp
int cnt[3];
long long ans = 0;

// 对每个连通块 BFS 染色，同时统计两种颜色的点数
// 若冲突输出 Impossible
// 否则 ans += min(cnt[1], cnt[2])
```

**复杂度**：$O(n+m)$。

### 例题 2：P4055 [JSOI2009] 游戏（二分图博弈 / 最大匹配）

**题意**：一个 $n\times m$ 的网格，有些格子是障碍。两人轮流在空格上放棋子，每个格子只能放一次，且棋子必须放在**与上一个棋子相邻**（四连通）的空格上。不能操作者输。求先手必胜的所有起点。

**思路**：网格图天然二分图（按 $(i+j)$ 奇偶染色）。这是一个**二分图博弈**：先手必胜 $\iff$ 起点在所有最大匹配中都被匹配。做法是对网格黑白染色建二分图，求最大匹配，再从每个未匹配点出发沿交替路标记，能到达的点加入「必胜起点」。

**复杂度**：$O(nm\sqrt{nm})$（Hopcroft-Karp）。

### 例题 3：P2756 飞行员配对方案问题（最大匹配 + 方案输出）

**题意**：有 $n$ 个外籍飞行员和 $m$ 个英国飞行员，给出若干对可配合关系，求最多能组成多少对，并输出任一方案。

**思路**：裸的二分图最大匹配。用匈牙利算法，最后遍历右部点输出 `matchR[v]` 与 $v$。

```cpp
int res = maxMatching(n);
printf("%d\n", res);
for (int v = 1; v <= m; ++v)
    if (matchR[v] != -1)
        printf("%d %d\n", matchR[v], v);
```

**复杂度**：$O(nm)$。

### 例题 4：P2764 最小路径覆盖问题

**题意**：给定 DAG，求最少用多少条**点不相交**的路径覆盖所有点，并输出方案。

**思路**：拆点建二分图，$u\to v$ 连 $u_{out}\to v_{in}$，跑最大匹配。答案 $= n -$ 最大匹配。方案通过匹配边连接起来。

```cpp
// 1. 拆点建图
for (auto &e : edges)
    g[e.first].push_back(e.second + n);
int match = maxMatching(n);
printf("%d\n", n - match);
// 2. 用 matchL 数组串出每条路径（matchL[u] 是 u 的后继）
```

**复杂度**：$O(nm)$ 或 $O(m\sqrt n)$。

---

## 4. 易错点与练习建议

### 易错点

1. **忘记清空 `vis` 数组**：匈牙利算法每次为新的左部点找增广路前都要 `memset(vis, 0, sizeof(vis))`，否则会漏找。
2. **左右部点编号混淆**：建图时要清楚哪边是左部，`matchR` 的索引是右部点。
3. **单向边建成双向边**：二分图匹配是**有向**建模（左到右），把边双向建会让 DFS 反复横跳。
4. **孤立点也要算入总点数**：最大独立集 $= n - $ 最大匹配中的 $n$ 是全部点数（包括孤立点）。
5. **只对一个连通块染色**：图可能不连通，要对每个未染色的点都启动 BFS。
6. **奇环判定**：染色冲突即奇环，直接输出非二分图；注意自环也是奇环。
7. **匈牙利复杂度**：$O(nm)$ 在 $n,m\ge 10^5$ 时会 TLE，要用 Hopcroft-Karp 或网络流。
8. **König 定理用错对象**：只对二分图成立，一般图要用带花树。
9. **最小路径覆盖要求点不相交**：若允许点相交，问题是 NP-hard 的；拆点建图仅适用于点不相交版本。
10. **方案输出**：最大匹配方案不唯一，输出任意一组即可，但要注意格式和末尾换行。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1330 封锁阳光大学 | 二分图染色 |
| 入门 | 洛谷 P2756 飞行员配对方案 | 匈牙利 + 方案 |
| 入门 | 洛谷 P3386 二分图最大匹配 | 匈牙利模板 |
| 提高 | 洛谷 P2764 最小路径覆盖问题 | 拆点 + 最大匹配 |
| 提高 | 洛谷 P4055 游戏 | 二分图博弈 |
| 提高 | 洛谷 P1640 [SCOI2010] 连续攻击游戏 | 二分图匹配 + 贪心 |
| 提高 | 洛谷 P1129 [ZJOI2007] 矩阵游戏 | 行列建图 |
| 进阶 | 洛谷 P4251 [SCOI2015] 小凸玩矩阵 | 二分 + 匹配 |
| 进阶 | 洛谷 P2055 [ZJOI2009] 假期的宿舍 | 建图 + 最大匹配 |

学习建议：二分图的判定（染色）和匈牙利算法是必须默写的两块模板。做题时先问三个问题：①图能否分成两部？②要匹配的是什么？③答案和最大匹配怎么换算（König / 独立集 / 路径覆盖）？把这三步想清楚，二分图题就完成一半了。
