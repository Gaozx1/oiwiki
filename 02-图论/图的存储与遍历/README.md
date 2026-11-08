# 图的存储与遍历

> 一句话概括：图由「点」和「边」组成，**存储方式决定算法的时空复杂度**，而 DFS 与 BFS 是打开所有图论算法的两把钥匙。

本节要解决的问题是：怎样把一张图放进计算机，以及怎样系统地访问到每一个点和每一条边。看似基础，但几乎所有图论算法——最短路、最小生成树、Tarjan、网络流——都建立在「邻接表 + DFS/BFS」这套基础设施之上。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 图的基本概念
   - 1.2 四种存储方式
   - 1.3 DFS 与 BFS 的思想
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 图的基本概念

**图（Graph）** 由顶点集合 $V$ 和边集合 $E$ 组成，记作 $G=(V,E)$。$n=|V|$ 称为点数，$m=|E|$ 称为边数。

按边的性质可以分为：

| 类型 | 说明 | 典型例子 |
|---|---|---|
| 无向图 | 边 $(u,v)$ 没有方向，$u$ 到 $v$、$v$ 到 $u$ 相通 | 公路、朋友关系 |
| 有向图 | 边 $\langle u,v\rangle$ 有方向，只能从 $u$ 走到 $v$ | 单向街道、依赖关系 |
| 带权图 | 每条边附带一个权值 $w$ | 距离、花费、时间 |
| 简单图 | 无自环、无重边 | 绝大多数算法题 |
| 完全图 | 任意两点之间都有边，$m=\frac{n(n-1)}{2}$ | 稠密场景 |

两个常用术语：

- **度（degree）**：与一个点相连的边数；有向图中分为**入度**和**出度**。
- **路径（path）**：由边连接起来的点序列；**简单路径**要求点不重复。

一个重要的计数结论（握手定理）：无向图中所有点的度数之和等于边数的两倍，

$$
\sum_{v\in V}\deg(v)=2m.
$$

因此**奇度点一定有偶数个**，这是欧拉路径存在性判定的基础。

### 1.2 四种存储方式

#### （1）直接存边（边表）

用一个结构体数组把所有边原样存下来：`struct Edge { int u, v, w; }; Edge e[MAXM];`。

- 优点：实现最简单，便于按「边」为单位处理，如 Kruskal 算法。
- 缺点：无法快速回答「与 $u$ 相邻的点有哪些」，每次都要扫全部边，$O(m)$。

#### （2）邻接矩阵

用二维数组 `g[u][v]` 表示 $u$ 到 $v$ 是否有边（或边权）：`int g[MAXN][MAXN];`，无边用 `INF` 表示，无向图要同时写 `g[u][v]` 与 `g[v][u]`。这样得到的矩阵沿对角线对称。

- 优点：$O(1)$ 判断两点是否相邻，Floyd 算法必须用它。
- 缺点：空间 $O(n^2)$。$n=10^5$ 时无论怎么开都会爆炸，只适合 $n\le 5000$ 左右的稠密图。

#### （3）邻接表

对每个点 $u$，只存「所有与 $u$ 相连的边」。这是 OI 中最主流的存图方式。

用 `vector` 实现：

```cpp
vector<int> g[MAXN];              // 不带权
vector<pair<int,int>> gw[MAXN];   // (to, w) 带权
g[u].push_back(v);
gw[u].push_back({v, w});
```

```
点 1: → 2 → 4
点 2: → 1 → 3
点 3: → 2
点 4: → 1
```

- 空间 $O(n+m)$，遍历一个点的所有出边是 $O(\deg(u))$，总遍历 $O(n+m)$。
- 缺点：`vector` 动态扩容有常数开销，且不方便快速找反向边（网络流需要）。

#### （4）链式前向星

用「数组模拟链表」存邻接表，每个点挂一条边链表：

```cpp
struct Edge { int to, w, nxt; } e[MAXM];
int head[MAXN], tot = 0;
void addEdge(int u, int v, int w) {
    e[++tot] = {v, w, head[u]};
    head[u] = tot;
}
```

```
head[1] → 边3(v=4) → 边1(v=2) → 0
head[2] → 边4(v=3) → 边2(v=1) → 0
```

- 空间连续、常数小，是卡常和网络流（成对存边 `i ^ 1` 取反边）的首选。
- 缺点是书写略繁琐，并且**加边顺序与遍历顺序相反**（头插法）。

四种存储方式的选择建议：$n\le 5000$ 的稠密图或 Floyd 用邻接矩阵；一般图论题用 `vector` 邻接表；网络流、需要反向边或卡常用链式前向星；以边为处理单位（Kruskal）用边表。

### 1.3 DFS 与 BFS 的思想

#### 深度优先搜索（DFS）

**策略**：沿一条路一直走到底，走不通再回溯，换一条路继续。

```
       1
      / \
     2   3
    / \   \
   4   5   6

DFS 访问顺序（从 1 出发）：1 → 2 → 4 → 5 → 3 → 6
```

DFS 的本质是**递归 + 标记已访问**。伪代码：

```
dfs(u):
    vis[u] = true
    访问 u
    for v in neighbors(u):
        if not vis[v]:
            dfs(v)
```

DFS 的优点是代码短、能天然维护「进入/离开」时间戳，因而是 Tarjan 系列算法（强连通分量、割点、桥、点双）的载体。缺点是在极深的图上递归会爆栈。

#### 广度优先搜索（BFS）

**策略**：从起点出发，先访问所有距离为 1 的点，再访问距离为 2 的点……像水波一样层层扩散。

```
       1
      / \
     2   3
    / \   \
   4   5   6

BFS 访问顺序（从 1 出发）：1 → 2 → 3 → 4 → 5 → 6
```

BFS 用**队列**实现：

```
bfs(s):
    vis[s] = true; q.push(s)
    while q not empty:
        u = q.front(); q.pop()
        访问 u
        for v in neighbors(u):
            if not vis[v]:
                vis[v] = true; q.push(v)
```

关键性质：**在无权图（或边权全为 1）中，BFS 第一次访问到 $u$ 时的层数就是 $s$ 到 $u$ 的最短距离**。这也是后面「最短路」一章中 0-1 BFS、多源 BFS 的出发点。

#### 复杂度分析

设使用邻接表，则每个点入队/出队一次，每条边被检查两次（无向图）或一次（有向图）：

$$
T_{\text{DFS}}=T_{\text{BFS}}=O(n+m),\qquad S=O(n).
$$

使用邻接矩阵时每次找邻居要扫一整行，复杂度退化为 $O(n^2)$。

#### 连通性

- **无向图**：从任意点出发做一次 DFS/BFS，能访问到的点集是一个**连通块**。对每个未访问的点再启动一次，即可统计连通块个数。
- **有向图**：分为**弱连通**（忽略方向后连通）和**强连通**（任意两点互相可达）。强连通分量将在专门章节用 Tarjan 求解。

---

## 2. 模板代码

下面给出一个完整可编译的程序：读入 $n$ 个点、$m$ 条边的有向图，输出从 $s$ 出发的 DFS 序和 BFS 序。采用链式前向星存图。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
const int MAXM = 200005;

struct Edge {
    int to, w, nxt;
} e[MAXM];

int head[MAXN], tot = 0;
int n, m, s;
bool vis[MAXN];

// 链式前向星加边：头插法，新边挂在链表头部
void addEdge(int u, int v, int w = 1) {
    e[++tot] = {v, w, head[u]};
    head[u] = tot;
}

// 递归 DFS：沿一条路走到底，回溯后换路
void dfs(int u) {
    vis[u] = true;
    printf("%d ", u);
    for (int i = head[u]; i; i = e[i].nxt) {
        int v = e[i].to;
        if (!vis[v]) dfs(v);
    }
}

// BFS：队列实现，按层扩散
void bfs(int start) {
    memset(vis, 0, sizeof(vis));
    queue<int> q;
    q.push(start);
    vis[start] = true;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        printf("%d ", u);
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (!vis[v]) {
                vis[v] = true;   // 入队时就标记，避免同一点重复入队
                q.push(v);
            }
        }
    }
}

int main() {
    scanf("%d%d%d", &n, &m, &s);
    for (int i = 1; i <= m; ++i) {
        int u, v;
        scanf("%d%d", &u, &v);
        addEdge(u, v);
    }
    dfs(s);
    puts("");
    bfs(s);
    puts("");
    return 0;
}
```

### 2.1 非递归 DFS（防止爆栈）

当图是一条长链（$n=10^6$）时，递归 DFS 会栈溢出。可以用显式栈模拟：

```cpp
void dfsIterative(int start) {
    stack<int> st;
    st.push(start);
    while (!st.empty()) {
        int u = st.top();
        st.pop();
        if (vis[u]) continue;      // 可能被重复压栈，出栈时判重
        vis[u] = true;
        printf("%d ", u);
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (!vis[v]) st.push(v);
        }
    }
}
```

注意这种写法得到的 DFS 序与递归版**不一定相同**（因为兄弟节点会逆序处理），但访问到的点集与连通性判断完全等价。

### 2.2 连通块统计

枚举每个未访问的点，每启动一次 DFS 就吃掉一整个连通块，计数加一：主函数中写 `for (int i = 1; i <= n; ++i) if (!vis[i]) { ++cnt; dfs(i); }` 即可。

### 2.3 网格图上的遍历

很多题面不是直接给图，而是给一个 $n\times m$ 的网格，四连通或八连通。把每个格子看作点，相邻格子之间连边即可，用方向数组 `dx[4]={1,-1,0,0}`、`dy[4]={0,0,1,-1}` 简化代码；BFS 时把 `(x,y)` 打包成 `pair` 入队，出队时先判断越界再访问格子。

---

## 3. 经典例题

### 例题 1：图的遍历（洛谷 P5318）

**题意**：给定 $n$ 个点、$m$ 条边的有向图，从 1 号点出发，分别输出 DFS 序和 BFS 序；要求 DFS 时优先访问编号小的点，BFS 时同样优先编号小的点。

**思路**：题目要求「编号小的优先」，因此不能直接用头插法的链式前向星，需要先把每个点的邻居按编号升序排序再遍历。用 `vector` 存邻接表并 `sort` 即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 100005;
vector<int> g[MAXN];
bool vis[MAXN];

void dfs(int u) {
    vis[u] = true;
    printf("%d ", u);
    for (int v : g[u])
        if (!vis[v]) dfs(v);
}

void bfs(int s) {
    memset(vis, 0, sizeof(vis));
    queue<int> q;
    q.push(s); vis[s] = true;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        printf("%d ", u);
        for (int v : g[u])
            if (!vis[v]) { vis[v] = true; q.push(v); }
    }
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; ++i) {
        int u, v; scanf("%d%d", &u, &v);
        g[u].push_back(v);
    }
    for (int i = 1; i <= n; ++i) sort(g[i].begin(), g[i].end());
    dfs(1); puts("");
    bfs(1); puts("");
    return 0;
}
```

**复杂度**：排序 $O(m\log m)$，遍历 $O(n+m)$，总 $O(m\log m+n)$。

### 例题 2：01 迷宫（洛谷 P1141）

**题意**：有一个 $n\times n$ 的 01 矩阵，每次询问从 $(x,y)$ 出发，在相邻且数字不同的格子之间移动，最多能到达多少个格子。询问 $m$ 次，$n\le 1000$，$m\le 10^5$。

**思路**：如果对每个询问都做一次 BFS，最坏 $O(n^2m)$ 会超时。注意到**同一个连通块内的所有格子答案相同**，因此先对整个网格做一遍 BFS，边 BFS 边记录当前连通块的大小和包含的格子，把答案直接写到每个格子上，之后每个询问 $O(1)$ 回答。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1005;
char grid[MAXN][MAXN];
int ans[MAXN][MAXN];
int n, m;
const int dx[4] = {1, -1, 0, 0};
const int dy[4] = {0, 0, 1, -1};

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; ++i) scanf("%s", grid[i] + 1);
    int comp = 0;
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= n; ++j) {
            if (ans[i][j]) continue;
            ++comp;
            vector<pair<int,int>> cells;
            queue<pair<int,int>> q;
            q.push({i, j}); ans[i][j] = comp;
            while (!q.empty()) {
                int x = q.front().first, y = q.front().second;
                q.pop();
                cells.push_back({x, y});
                for (int d = 0; d < 4; ++d) {
                    int nx = x + dx[d], ny = y + dy[d];
                    if (nx < 1 || nx > n || ny < 1 || ny > n) continue;
                    if (ans[nx][ny] || grid[nx][ny] == grid[x][y]) continue;
                    ans[nx][ny] = comp;
                    q.push({nx, ny});
                }
            }
            int sz = (int)cells.size();
            for (auto &p : cells) ans[p.first][p.second] = sz;   // 用大小覆盖编号
        }
    while (m--) {
        int x, y; scanf("%d%d", &x, &y);
        printf("%d\n", ans[x][y]);
    }
    return 0;
}
```

**复杂度**：预处理 $O(n^2)$，每次询问 $O(1)$，总 $O(n^2+m)$。

### 例题 3：求连通块数量与大小

**题意**：给定 $n$ 个点 $m$ 条边的无向图，求连通块个数，并输出每个连通块的大小（升序）。

**思路**：枚举每个未访问的点启动 DFS，用一个引用参数累计本次访问到的点数，把所有大小排序输出。时间复杂度 $O(n+m)$，排序额外 $O(n\log n)$。

```cpp
void dfsCount(int u, int &cnt) {
    vis[u] = true;
    ++cnt;
    for (int i = head[u]; i; i = e[i].nxt)
        if (!vis[e[i].to]) dfsCount(e[i].to, cnt);
}
// main 中：for (i=1..n) if (!vis[i]) { int c=0; dfsCount(i,c); compSize.push_back(c); }
```

---

## 4. 易错点与练习建议

### 易错点

1. **忘记标记起点**：BFS 只标记 `push` 进去的点，如果一开始没有 `vis[s]=true`，起点可能被重复入队。
2. **入队时标记还是出队时标记**：BFS 必须在**入队时**标记。出队时再标记会导致同一个点被多个邻居重复压入队列，复杂度退化甚至答案错误。
3. **邻接矩阵开太大**：`int g[100000][100000]` 会直接 CE/MLE。先看数据范围再选存储方式。
4. **重边与自环**：大多数算法能容忍重边，但求最短路时重边只保留最小的一条更优；自环在拓扑排序中会直接导致无解。
5. **递归爆栈**：链状图深度可达 $10^5\sim10^6$，Windows 下默认栈很小，本地测试可能正常而评测 RE。必要时改用非递归 DFS 或手动开栈。
6. **有向图与无向图加边次数**：无向图 `addEdge(u,v)` 和 `addEdge(v,u)` 要各调一次；若需要反向边（网络流），`tot` 要从 1 开始并**成对加边**，这样 `i ^ 1` 才是配对的反向边。
7. **多组数据不清空**：`head`、`vis`、`tot` 都要重置，否则上一组数据会污染下一组。
8. **网格图越界判断顺序**：先判断越界再访问 `grid[nx][ny]`，否则可能数组越界。
9. **DFS 序 ≠ 拓扑序**：DFS 的访问顺序在有向图上没有「前驱先访问」的保证，求拓扑序要用专门的入度法或对 DFS 完成时间逆序。
10. **复杂度算错**：邻接矩阵存图 + DFS 是 $O(n^2)$，不是 $O(n+m)$。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P5318 查找文献 | DFS/BFS 基本遍历与排序 |
| 入门 | 洛谷 P1596 Lake Counting | 网格八连通块 |
| 入门 | 洛谷 P1141 01 迷宫 | 连通块预处理 + 多询问 |
| 提高 | 洛谷 P1330 封锁阳光大学 | 二分图染色（见二分图一章） |
| 提高 | 洛谷 P3916 图的遍历 | 反向建图 + 贪心遍历 |
| 提高 | 洛谷 P2296 寻找道路 | 预处理可达性 + BFS |
| 进阶 | 洛谷 P1073 最优贸易 | 分层图 / 双向遍历 |

学习建议：先把「邻接表 + 递归 DFS + 队列 BFS」练到能盲写，再尝试链式前向星。所有模板都要自己敲一遍并编译运行，不要只看不写。图的存储是中后期所有图论题的地基，慢就是快。
