# 启发式合并（Heuristic Merging / DSU on Tree）

> 启发式合并 = **每次合并时把小的结构并入大的结构**。看似朴素，却能证明总复杂度为 $O(n \log n)$。它是「合并」类问题的万能钥匙。

---

## 目录

1. [核心思想](#1-核心思想)
2. [复杂度证明](#2-复杂度证明)
3. [实现方式一：set/map 启发式合并](#3-实现方式一setmap-启发式合并)
4. [实现方式二：DSU on Tree（树上启发式合并）](#4-实现方式二dsu-on-tree树上启发式合并)
5. [经典例题](#5-经典例题)
6. [易错点](#6-易错点)
7. [练习建议](#7-练习建议)

---

## 1. 核心思想

假设我们要合并两棵（或两堆）数据结构 $A$ 和 $B$：

- 若 $|A| < |B|$，把 $A$ 的每个元素**插入** $B$；
- 否则把 $B$ 的每个元素插入 $A$。

**为什么快？** 每个元素每次被「搬动」时，它所在的集合大小至少翻倍。所以一个元素最多被搬动 $O(\log n)$ 次。

---

## 2. 复杂度证明

设元素总数为 $n$。

- 单次插入代价 $O(1)$（或 $O(\log n)$ 取决于容器）；
- 每个元素被搬动次数 $\le \log_2 n$（集合大小每次至少翻倍）；
- 总复杂度 $O(n \log n)$（或 $O(n \log^2 n)$）。

> 关键：**永远让小的向大的合并**。反过来会退化成 $O(n^2)$。

---

## 3. 实现方式一：set/map 启发式合并

### 3.1 合并两个 set

```cpp
set<int> s[N];

void mergeSet(int u, int v) {   // 把 v 并入 u
    if (s[u].size() < s[v].size()) s[u].swap(s[v]);   // 保证 u 是大的
    for (int x : s[v]) s[u].insert(x);
    s[v].clear();
}
```

### 3.2 维护每个集合的最大值

```cpp
map<int, int> cnt;   // 值 -> 出现次数
int ans = 0;

void addVal(int x) {
    if (++cnt[x] == 1) ans = max(ans, x);   // 只记最大
}
```

---

## 4. 实现方式二：DSU on Tree（树上启发式合并）

### 4.1 问题模型

给定一棵树，每个节点有一个颜色，询问每个子树内「出现次数最多的颜色」或「不同颜色数」。

### 4.2 算法流程

对每个节点 $u$：

1. 先递归处理**轻儿子**，处理完**清空**它们的贡献；
2. 递归处理**重儿子**，**保留**它的贡献；
3. 把轻儿子子树的贡献暴力加回来；
4. 加入 $u$ 自己；
5. 此时得到 $u$ 子树的答案。

**复杂度**：每个节点被加入 $O(\log n)$ 次（因为它只会在祖先的轻边处被重新加入），总 $O(n \log n)$。

### 4.3 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100005;
int n;
int col[N], sz[N], son[N], cnt[N], ans[N];
int maxCnt = 0;
long long sumMax = 0;
vector<int> g[N];

void dfs1(int u, int p) {
    sz[u] = 1;
    son[u] = 0;
    for (int v : g[u]) {
        if (v == p) continue;
        dfs1(v, u);
        sz[u] += sz[v];
        if (sz[v] > sz[son[u]]) son[u] = v;
    }
}

void add(int u, int p, int delta) {
    int c = col[u];
    cnt[c] += delta;
    if (cnt[c] > maxCnt) { maxCnt = cnt[c]; sumMax = c; }
    else if (cnt[c] == maxCnt) { sumMax += c; }
    for (int v : g[u]) if (v != p && v != son[u]) add(v, u, delta);
}

void dfs2(int u, int p, bool keep) {
    for (int v : g[u])
        if (v != p && v != son[u]) dfs2(v, u, false);

    if (son[u]) dfs2(son[u], u, true);

    // 加入轻儿子
    for (int v : g[u])
        if (v != p && v != son[u]) add(v, u, 1);

    // 加入自己
    cnt[col[u]]++;
    if (cnt[col[u]] > maxCnt) { maxCnt = cnt[col[u]]; sumMax = col[u]; }
    else if (cnt[col[u]] == maxCnt) { sumMax += col[u]; }

    ans[u] = sumMax;

    if (!keep) {
        add(u, p, -1);
        // 注意：add 会遍历整棵子树（不包括重儿子），这里需要完整清空
        maxCnt = 0;
        sumMax = 0;
    }
}
```

> ⚠️ 上面的 `add(u, p, -1)` 逻辑需要小心：清空时要把 $u$ 的整棵子树（含重儿子）都减掉。更稳妥的写法是单独写一个 `clear(u, p)`，或者用 `add` 时传 `p = 0` 遍历全子树。

### 4.4 更清晰的清空版本

```cpp
void clearSubtree(int u, int p) {
    cnt[col[u]]--;
    for (int v : g[u]) if (v != p) clearSubtree(v, u);
}
```

---

## 5. 经典例题

### 5.1 树上子树颜色众数

**题意**：$n \le 10^5$，每个点一个颜色，问每个子树中出现次数最多的颜色之和。

**思路**：DSU on Tree 模板题。

**复杂度**：$O(n \log n)$。

### 5.2 集合合并求第 k 大

**题意**：初始 $n$ 个集合，$m$ 次操作合并两个集合并询问某个集合第 $k$ 大。

**思路**：用 `set` 启发式合并，合并时保持大集合。

### 5.3 并查集 + 启发式合并

并查集其实也是一种启发式合并：**按秩合并**保证树高 $O(\log n)$。

```cpp
int find(int x) { return fa[x] == x ? x : fa[x] = find(fa[x]); }
void merge(int x, int y) {
    x = find(x); y = find(y);
    if (x == y) return;
    if (rnk[x] < rnk[y]) swap(x, y);
    fa[y] = x;
    if (rnk[x] == rnk[y]) rnk[x]++;
}
```

### 5.4 线段树合并

两个动态开点线段树合并，也是「小的并到大的」思想：

```cpp
int merge(int a, int b, int l, int r) {
    if (!a || !b) return a + b;
    if (l == r) { tr[a].sum += tr[b].sum; return a; }
    int mid = (l + r) >> 1;
    tr[a].ls = merge(tr[a].ls, tr[b].ls, l, mid);
    tr[a].rs = merge(tr[a].rs, tr[b].rs, mid+1, r);
    pushup(a);
    return a;
}
```

---

## 6. 易错点

1. **必须保证「小并大」**：否则复杂度退化为 $O(n^2)$；
2. **DSU on Tree 中重儿子判断**：`son[u]` 要在 `dfs1` 中正确计算；
3. **清空时机**：轻儿子先处理并清空，重儿子最后处理并保留；
4. **`keep` 参数含义**：`true` 表示保留当前子树的贡献给父亲用；
5. **清空时要清干净**：包括 $u$ 自身与整棵子树；
6. **多组数据初始化**：`cnt`、`maxCnt`、`sumMax` 要重置。

---

## 7. 练习建议

1. **先写并查集按秩合并**，理解「小并大」；
2. **用 `set` 实现集合合并**，练习 `swap` 技巧；
3. **DSU on Tree 模板题**：子树众数；
4. **线段树合并**：主席树进阶；
5. **对比普通暴力**：体会 $O(n \log n)$ 与 $O(n^2)$ 的差距。
