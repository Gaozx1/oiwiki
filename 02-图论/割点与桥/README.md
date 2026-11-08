# 割点与桥

> 一句话概括：**割点**是删掉后会让连通块增多的点，**桥**是删掉后会让连通块增多的边；Tarjan 用一次 DFS 同时求出两者，并且是点双/边双连通分量的基础。

割点与桥刻画了图的「脆弱环节」：网络中的关键路由器、交通网中的咽喉要道，本质上就是割点或桥。本节从连通度概念出发，推导 `low` 数组在无向图上的新含义，并扩展到点双连通分量（v-BCC）与边双连通分量（e-BCC）。

## 目录

1. [概念与原理](#1-概念与原理)
   - 1.1 割点与桥的定义
   - 1.2 无向图 Tarjan 的 `dfn` 与 `low`
   - 1.3 割点判定
   - 1.4 桥判定
   - 1.5 点双与边双
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 割点与桥的定义

给定**连通无向图** $G=(V,E)$：

- **割点（Cut Vertex / Articulation Point）**：若删除点 $u$ 及其所有关联边后，图的连通块数量增加，则 $u$ 是割点。
- **桥（Bridge）**：若删除边 $e$ 后，图的连通块数量增加，则 $e$ 是桥。

```
        1 --- 2 --- 3
        |           |
        4 --- 5 --- 6

割点：2、5（删掉任意一个，图分成两半）
桥：1-2、2-3、4-5、5-6  （删掉任意一条，图不连通）
```

注意：

- 桥的两个端点**不一定**都是割点（考虑两个点一条边的图：两个端点都是割点吗？删掉一个点后只剩一个点，连通块数不变，所以都不是割点，但这条边是桥）。
- 割点也可能连着非桥边。
- 只有**无向图**才讨论割点与桥。

### 1.2 无向图 Tarjan 的 `dfn` 与 `low`

与 SCC 一节类似，仍然是一趟 DFS，维护：

| 变量 | 含义 |
|---|---|
| `dfn[u]` | $u$ 第一次被访问的时间戳 |
| `low[u]` | 从 $u$ 的子树出发，不经过 $u$ 的父边，能到达的最小 `dfn` |

**与有向图的关键区别**：

- 无向图的边是双向的，DFS 时会遇到「父节点」，必须**跳过父边**，否则会误以为找到了回边。
- 因为可能有**重边**，不能简单地「跳过所有指向父节点的边」，而应「跳过**这一条**父边」（用边的编号判断）。
- 无向图没有横叉边（DFS 树中不存在指向已访问兄弟子树的边），所以更新规则简化为两类：树边和回边。

```
DFS 树（无向图）:
      1
     / \
    2   3
   / \
  4   5
  |
  6 --- 2   ← 6 指向祖先 2 的回边

low[6] = dfn[2] = 2
low[4] = 2, low[2] = 1? 若 2 无其它回边则 low[2]=dfn[2]
```

### 1.3 割点判定

设 $u$ 是当前点，$v$ 是 $u$ 的一个子节点（DFS 树上的儿子）。分两种情况：

**情况 1：$u$ 是根节点**。若 $u$ 有**两个及以上**的 DFS 子树，则 $u$ 是割点。因为不同子树之间没有边相连（否则 DFS 会走通），删掉 $u$ 后它们就断开了。

```
      1
     / \
    2   3      ← 1 是根，有两棵子树，是割点
```

**情况 2：$u$ 不是根节点**。若存在一个子节点 $v$ 满足

$$
low[v]\ge dfn[u],
$$

则 $u$ 是割点。含义是：$v$ 的子树「爬不回」$u$ 的祖先，只能到达 $u$，所以删掉 $u$ 后 $v$ 的子树就与上面断开了。

**注意等号**：`low[v] == dfn[u]` 时，$v$ 的子树能回到 $u$ 本身，但删掉 $u$ 后仍然断开，所以 $\ge$ 是必须的。

### 1.4 桥判定

对树边 $(u,v)$（$v$ 是 $u$ 的儿子），若

$$
low[v] > dfn[u],
$$

则边 $(u,v)$ 是桥。

**为什么是严格大于？** 若 `low[v] == dfn[u]`，说明 $v$ 的子树能通过回边回到 $u$，删掉 $(u,v)$ 后仍能从回边绕回去，所以不是桥。

**对比记忆**：

| 判定 | 条件 |
|---|---|
| 割点（非根） | $low[v]\ge dfn[u]$ |
| 桥 | $low[v]>dfn[u]$ |

### 1.5 点双与边双

**边双连通分量（e-BCC / 边双）**：不含桥的极大连通子图。等价说法：任意两点间存在**至少两条边不相交**的路径。

**求法**：先求出所有桥，把桥删掉，剩下的每个连通块就是一个边双。或者用栈在 Tarjan 中维护边，遇到 `low[v] > dfn[u]` 时弹栈。

**缩点**：把每个边双缩成一个点，得到的是一棵**树**（因为所有环都被缩掉了，剩下只有桥）。

**点双连通分量（v-BCC / 点双）**：不含割点的极大连通子图。等价说法：任意两点间存在**至少两条点不相交**的路径（除端点外）。

**求法**：在 Tarjan 中用栈维护点，当 `low[v] >= dfn[u]` 时弹栈到 $v$（注意 $u$ 不弹），弹出的点集加上 $u$ 构成一个点双。

**重要性质**：

- 一条边恰好属于一个点双；一个点可以属于多个点双（割点）。
- 点双缩点后得到的是一棵**圆方树**（原点为圆点，点双为方点）。

| 概念 | 删什么不连通 | 缩点后的结构 |
|---|---|---|
| 割点 | 删点 | 圆方树 |
| 桥 | 删边 | 树 |
| 边双 | 不含桥 | 树 |
| 点双 | 不含割点 | 圆方树 |

---

## 2. 模板代码

### 2.1 求割点（链式前向星）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
const int MAXM = 200005;

struct Edge { int to, nxt; } e[MAXM * 2];
int head[MAXN], tot = 1;            // 从 1 开始，i^1 为反向边
int dfn[MAXN], low[MAXN], timer = 0;
bool isCut[MAXN];
int n, m;

void addEdge(int u, int v) {
    e[++tot] = {v, head[u]};
    head[u] = tot;
}

void tarjanCut(int u, int faEdge) {   // faEdge: 进入 u 的边的编号
    dfn[u] = low[u] = ++timer;
    int child = 0;                    // DFS 树上的子树个数
    for (int i = head[u]; i; i = e[i].nxt) {
        if (i == (faEdge ^ 1)) continue;   // 跳过同一条边的反向边（父边）
        int v = e[i].to;
        if (!dfn[v]) {
            ++child;
            tarjanCut(v, i);
            low[u] = min(low[u], low[v]);
            // 非根节点：存在 low[v] >= dfn[u] 的子节点
            if (faEdge != 0 && low[v] >= dfn[u]) isCut[u] = true;
        } else {
            low[u] = min(low[u], dfn[v]);  // 回边
        }
    }
    // 根节点：子树个数 >= 2
    if (faEdge == 0 && child >= 2) isCut[u] = true;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= m; ++i) {
        int u, v; scanf("%d%d", &u, &v);
        addEdge(u, v);
        addEdge(v, u);
    }
    for (int i = 1; i <= n; ++i)
        if (!dfn[i]) tarjanCut(i, 0);
    int cnt = 0;
    for (int i = 1; i <= n; ++i) if (isCut[i]) ++cnt;
    printf("%d\n", cnt);
    for (int i = 1; i <= n; ++i) if (isCut[i]) printf("%d ", i);
    puts("");
    return 0;
}
```

### 2.2 求桥

```cpp
int dfn[MAXN], low[MAXN], timer = 0;
bool isBridge[MAXM * 2];             // 按边编号标记桥（边表大小与链式前向星一致）
vector<pair<int,int>> bridges;

void tarjanBridge(int u, int faEdge) {
    dfn[u] = low[u] = ++timer;
    for (int i = head[u]; i; i = e[i].nxt) {
        if (i == (faEdge ^ 1)) continue;
        int v = e[i].to;
        if (!dfn[v]) {
            tarjanBridge(v, i);
            low[u] = min(low[u], low[v]);
            if (low[v] > dfn[u]) {             // 严格大于才是桥
                isBridge[i] = isBridge[i ^ 1] = true;
                bridges.push_back({u, v});
            }
        } else {
            low[u] = min(low[u], dfn[v]);
        }
    }
}
```

### 2.3 边双缩点

先求桥（2.2 节），再忽略所有桥做一次 DFS：每个连通块就是一个边双，编号记入 `belong[]`。缩点后把每条非桥边的两个端点所属边双相连，得到一棵树。

```cpp
int belong[MAXN], bccCnt = 0;
bool isBridge[MAXN * 2];        // 桥标记，按边编号存储

void dfsBCC(int u, int c) {
    belong[u] = c;
    for (int i = head[u]; i; i = e[i].nxt) {
        int v = e[i].to;
        if (isBridge[i] || isBridge[i ^ 1]) continue;   // 跳过桥
        if (!belong[v]) dfsBCC(v, c);
    }
}

// 缩点建树（同一对边双之间可能有多条边，需自行去重）
void buildBridgeTree(int n) {
    for (int u = 1; u <= n; ++u)
        for (int i = head[u]; i; i = e[i].nxt) {
            int v = e[i].to;
            if (belong[u] != belong[v])
                tree[belong[u]].push_back(belong[v]);
        }
}
```

### 2.4 点双连通分量（v-BCC）

```cpp
int dfn[MAXN], low[MAXN], timer = 0;
int stk[MAXN], top = 0;
vector<vector<int>> vbcc;

void tarjanVBCC(int u, int faEdge) {
    dfn[u] = low[u] = ++timer;
    stk[++top] = u;
    for (int i = head[u]; i; i = e[i].nxt) {
        if (i == (faEdge ^ 1)) continue;
        int v = e[i].to;
        if (!dfn[v]) {
            tarjanVBCC(v, i);
            low[u] = min(low[u], low[v]);
            if (low[v] >= dfn[u]) {          // u 是割点或根
                vector<int> comp;
                while (true) {
                    int x = stk[top--];
                    comp.push_back(x);
                    if (x == v) break;       // 弹到 v 为止，u 不弹
                }
                comp.push_back(u);           // u 属于这个点双
                vbcc.push_back(comp);
            }
        } else {
            low[u] = min(low[u], dfn[v]);
        }
    }
}
```

### 2.5 割边（桥）的朴素判定（小图）

$n\le 1000,\ m\le 5000$ 时，可以枚举每条边，临时删掉它后从 1 号点做一次 DFS 统计可达点数：若不到 $n$ 个点，这条边就是桥。复杂度 $O(m(n+m))$，仅用于对拍验证 Tarjan 的正确性。

---

## 3. 经典例题

### 例题 1：割点模板（洛谷 P3388）

**题意**：给定 $n$ 个点 $m$ 条边的无向图，求所有割点，按编号升序输出。$n\le 2\times10^4$，$m\le 10^5$。

**思路**：直接套 2.1 节模板。注意图可能不连通，需要对每个未访问的点启动 Tarjan（`for (int i = 1; i <= n; ++i) if (!dfn[i]) tarjanCut(i, 0);`）。

**复杂度**：$O(n+m)$。

### 例题 2：桥模板（洛谷 P1656 炸铁路）

**题意**：给定 $n$ 个点 $m$ 条边的无向图，输出所有桥（按 $u$ 升序、再按 $v$ 升序）。

**思路**：Tarjan 求桥，存下来后统一把每条桥的端点按「小的在前」规范化，再按 `pair` 排序输出即可。

**复杂度**：$O(n+m)$。

### 例题 3：矿场搭建（洛谷 P3225 / HNOI2012）

**题意**：一个煤矿由若干点相连，发生事故后某个点会坍塌，要求在任何点坍塌后，其它点都能通过逃生通道到达出口。求最少需要多少个出口，以及最少出口数下的方案数。

**思路**：经典的**点双 + 分类讨论**题：

1. 求出所有点双和割点。
2. 对每个点双，统计其中割点个数 $c$：
   - $c=0$：这个点双是独立连通块，需要建 2 个出口（防坍塌），方案数 $\binom{s}{2}$，$s$ 为点双大小；
   - $c=1$：需要建 1 个出口（建在非割点上），方案数 $s-1$；
   - $c\ge2$：不需要出口（可以通过割点跑到别的点双）。
3. 所有点双的答案相乘。

```cpp
long long ans1 = 0, ans2 = 1;
for (auto &comp : vbcc) {
    int cut = 0;
    for (int x : comp) cut += isCut[x];
    if (cut == 0) {
        ans1 += 2;
        ans2 *= (long long)comp.size() * (comp.size() - 1) / 2;
    } else if (cut == 1) {
        ++ans1;
        ans2 *= (long long)(comp.size() - 1);
    }
}
```

**复杂度**：$O(n+m)$。

### 例题 4：删点后的连通性（洛谷 P3469 / BZOJ 1123 BLO）

**题意**：给定 $n$ 个点 $m$ 条边的无向图，对每个点 $i$，求删掉 $i$ 后有多少**有序点对** $(x,y)$（$x\ne y$）不再连通。

**思路**：在 Tarjan 过程中统计。删除 $u$ 后，原图被分成若干个连通块：每个满足 `low[v] >= dfn[u]` 的儿子子树（大小 $s_i$），以及「剩余部分」（大小 $r=n-1-\sum s_i$，包含父节点一侧和无法分裂的子树）。设这些块大小为 $c_1,c_2,\dots,c_k$，则

$$
\text{ans}[u]=\sum_{i} c_i\,(n-1-c_i)+2(n-1).
$$

前一项统计不同块之间的有序点对，后一项统计与 $u$ 自身有关的有序点对（$u$ 被删除后与所有点都不连通）。

```cpp
long long ans[MAXN];
int siz[MAXN];

void tarjanBLO(int u, int faEdge) {
    dfn[u] = low[u] = ++timer;
    siz[u] = 1;
    long long sum = 0;                    // 分裂出的子树大小之和
    for (int i = head[u]; i; i = e[i].nxt) {
        if (i == (faEdge ^ 1)) continue;
        int v = e[i].to;
        if (!dfn[v]) {
            tarjanBLO(v, i);
            siz[u] += siz[v];
            low[u] = min(low[u], low[v]);
            if (low[v] >= dfn[u]) {       // v 的子树会被 u 隔开
                ans[u] += (long long)siz[v] * (n - 1 - siz[v]);
                sum += siz[v];
            }
        } else {
            low[u] = min(low[u], dfn[v]);
        }
    }
    long long rest = n - 1 - sum;         // 剩余部分大小
    ans[u] += rest * (n - 1 - rest);      // 与剩余部分有关的点对
    ans[u] += 2LL * (n - 1);              // 与 u 自身有关的有序点对
}
```

**复杂度**：$O(n+m)$。

---

## 4. 易错点与练习建议

### 易错点

1. **割点条件漏掉等号**：写成 `low[v] > dfn[u]` 会把「能回到 $u$ 但回不到祖先」的点漏判。割点用 $\ge$，桥用 $>$。
2. **根节点判定错误**：根节点不能直接用 `low[v] >= dfn[u]`（那会把所有有子树的根都判成割点），必须用「子树个数 $\ge 2$」。
3. **重边导致误判桥**：若只用「跳过父节点」而不是「跳过父边编号」，重边会被当成一条边，导致桥判定错误。链式前向星必须成对加边并用 `i ^ 1`。
4. **孤立点**：孤立点不是割点（删掉后连通块数不变），但很多题目需要特殊处理。
5. **图不连通**：要对每个未访问点启动 Tarjan，`timer` 不必重置。
6. **无向图不能有「横叉边」**：DFS 遇到已访问点一定是回边（祖先），直接 `low[u] = min(low[u], dfn[v])`；但要注意 v 是后代的情况（此时 dfn[v] > dfn[u]，取 min 不影响）。
7. **点双弹栈边界**：点双是「弹到 $v$ 为止，$u$ 不弹」，然后手动把 $u$ 加进去；边双是「弹到 $u$ 为止」。
8. **桥数组的空间**：`MAXM * 2`，因为无向图每条边存两次。
9. **`tot` 初值**：链式前向星若从 1 开始编号，`head` 要初始化为 0，且 `i ^ 1` 配对；若从 0 开始，`i ^ 1` 也能配对但 `head` 要初始化为 -1。
10. **点双数量**：孤立点本身也算一个点双（视题目定义），有时需要单独处理。

### 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P3388 割点模板 | Tarjan 割点 |
| 入门 | 洛谷 P1656 炸铁路 | Tarjan 桥 |
| 提高 | 洛谷 P3225 矿场搭建 | 点双 + 组合计数 |
| 提高 | 洛谷 P3469 BLO | 割点 + 子树大小统计 |
| 提高 | 洛谷 P2860 [USACO06JAN] Redundant Paths | 边双缩点 + 树的性质 |
| 提高 | 洛谷 P8435 点双连通分量 | 点双模板 |
| 进阶 | 洛谷 P2403 [SDOI2010] 所驼门王的宝藏 | 强连通分量 / 割点综合 |
| 进阶 | 洛谷 P4606 [SDOI2018] 战略游戏 | 圆方树 |

学习建议：割点与桥的代码只差一个符号（`>=` vs `>`），但含义截然不同，建议对照记忆。一定要用带重边的小数据测试自己的模板，这是最容易翻车的地方。点双与圆方树是省选级别的重点，先掌握基本求法再学圆方树。
