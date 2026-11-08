# 树链剖分（Heavy-Light Decomposition）

> 一句话概括：**树链剖分**把树上的一条路径拆成 $O(\log n)$ 条连续的重链区间，从而把「树上路径修改/查询」转化为「序列上的区间操作」，交给线段树等数据结构处理。

---

## 目录

1. [从问题出发](#1-从问题出发)
2. [核心概念：重儿子与重链](#2-核心概念重儿子与重链)
3. [两次 DFS 与数组含义](#3-两次-dfs-与数组含义)
4. [路径操作](#4-路径操作)
5. [子树操作](#5-子树操作)
6. [完整模板代码](#6-完整模板代码)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 从问题出发

维护一棵 $n$ 个点的树，支持：

- 路径加：把 $u\to v$ 路径上所有点权加 $k$；
- 路径求和：求 $u\to v$ 路径上点权之和；
- 子树加 / 子树求和。

如果对每条路径单独 DFS 修改，单次 $O(n)$，无法接受。

**核心矛盾**：树是二维结构，而线段树只能处理一维区间。

**树链剖分的思路**：给每个点重新编号，使得「树上的一条路径」能被拆成少量（$O(\log n)$）的**连续编号区间**。这样每个区间用线段树处理即可。

---

## 2. 核心概念：重儿子与重链

### 2.1 定义

对每个节点 $u$，记 $\operatorname{size}(u)$ 为其子树大小：

- **重儿子（heavy son）**：$u$ 的所有儿子中 $\operatorname{size}$ 最大的那个（若有多个任取一个）；
- **轻儿子（light son）**：除重儿子外的其他儿子；
- **重边**：$u$ 与其重儿子之间的边；
- **轻边**：其余边；
- **重链**：由连续的重边连接而成的极大路径。每个点恰好属于一条重链，链的顶端称为**链头**（`top`）。

### 2.2 文字示意图

考虑下面这棵树（括号中为子树大小）：

```
                    1 (size=9)
                  / | \
                 /  |  \
         (size=5)2  3(size=2)  4(size=1)
               / \    |
       (size=3)5  6   7 (size=1)
             / \
            8   9        (8、9 的 size 均为 1)
```

逐点选择重儿子：

```
节点 1 的儿子：2(size=5), 3(size=2), 4(size=1) → 重儿子 2
节点 2 的儿子：5(size=3), 6(size=1)            → 重儿子 5
节点 3 的儿子：7(size=1)                       → 重儿子 7
节点 5 的儿子：8(size=1), 9(size=1)            → 重儿子 8（并列任取）
```

于是重链为：`1 - 2 - 5 - 8`、`6`、`3 - 7`、`4`、`9`，共 5 条重链。

若按「先重儿子」的顺序做 DFS，得到的 `dfn` 可能是：

| 节点 | 1 | 2 | 5 | 8 | 9 | 6 | 3 | 7 | 4 |
|---|---|---|---|---|---|---|---|---|---|
| `dfn` | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
| `top` | 1 | 1 | 1 | 1 | 9 | 2 | 3 | 3 | 4 |

可以看到：重链 `1-2-5-8` 的 `dfn` 为 `1,2,3,4`，是连续的；重链 `3-7` 的 `dfn` 为 `7,8`，也是连续的；节点 `9` 作为轻儿子单独成链（`top[9]=9`）。

### 2.3 关键性质

**性质 1**：从任意节点到根的路径上，轻边数量不超过 $\log_2 n$。

**证明**：设 $v$ 是 $u$ 的轻儿子，则 $\operatorname{size}(v) \le \operatorname{size}(u)/2$。否则 $v$ 会成为重儿子。每经过一条轻边，子树大小至少减半，故轻边数 $O(\log n)$。

**性质 2**：任意路径 $u\to v$ 被拆成 $O(\log n)$ 条重链。

**证明**：路径 $u\to v$ 可拆为 $u\to \operatorname{lca}$ 与 $v\to \operatorname{lca}$。每段向上走时，每穿过一条轻边就换一条重链，而轻边最多 $O(\log n)$ 条，故重链数量为 $O(\log n)$。

---

## 3. 两次 DFS 与数组含义

### 3.1 第一次 DFS：求父子关系与子树大小

- `fa[u]`：父节点；
- `dep[u]`：深度；
- `size[u]`：子树大小；
- `son[u]`：重儿子编号（没有则为 0）。

### 3.2 第二次 DFS：给重链编号

按「先走重儿子」的顺序 DFS，给每个点分配 `dfn`（DFS 序），同时记录 `top[u]`（所在重链的链头）。

**关键保证**：同一条重链上的点，`dfn` 是连续的；且每个点的**子树**在 DFS 序中也是连续的一段 $[\operatorname{dfn}(u),\ \operatorname{dfn}(u)+\operatorname{size}(u)-1]$。

### 3.3 数组汇总

| 数组 | 含义 |
|---|---|
| `fa[u]` | 父节点 |
| `dep[u]` | 深度（根的深度为 1） |
| `size[u]` | 子树大小 |
| `son[u]` | 重儿子 |
| `top[u]` | 所在重链的链头 |
| `dfn[u]` | DFS 序（用于线段树编号） |
| `rnk[i]` | DFS 序为 $i$ 的节点编号（`rnk[dfn[u]] = u`） |

---

## 4. 路径操作

### 4.1 核心框架

```
while top[u] != top[v]:
    若 dep[top[u]] < dep[top[v]]: 交换 u, v
    对区间 [dfn[top[u]], dfn[u]] 做操作
    u = fa[top[u]]
最后：对区间 [min(dfn[u],dfn[v]), max(dfn[u],dfn[v])] 做操作
```

### 4.2 为什么正确

- 每次把「链头更深」的那条重链整个处理掉；
- 因为同一条重链上的 `dfn` 连续，处理一条链只需一次区间操作；
- 处理完后跳到链头的父节点，相当于跨过一条轻边；
- 轻边最多 $O(\log n)$ 条，所以循环最多 $O(\log n)$ 次；
- 最后两点在同一条重链上，`dfn` 较小者（深度较浅）就是 LCA。

### 4.3 手算示意

沿用 2.2 节的树与编号：重链为 `1-2-5-8`（`dfn` 1, 2, 3, 4）、`6`（`dfn` 6）、`3-7`（`dfn` 7, 8）、`4`（`dfn` 9）、`9`（`dfn` 5）。

查询路径 $8\to9$（真实路径为 $8\to5\to9$）：

```
初态：u = 8, v = 9
top[8] = 1, top[9] = 9；dep[top[8]] = dep[1] = 1，dep[top[9]] = dep[9] = 4
链头 1 更浅 -> 交换：u = 9, v = 8
处理区间 [dfn[top[9]], dfn[9]] = [dfn[9], dfn[9]] = [5, 5]     （即节点 9）
u = fa[top[9]] = fa[9] = 5
现在 u = 5, v = 8：top[5] = top[8] = 1，同一条链，退出循环
处理区间 [min(dfn[5], dfn[8]), max(dfn[5], dfn[8])] = [3, 4]  （即节点 5, 8）
```

两段区间 `[5,5]` 与 `[3,4]` 恰好覆盖节点 $9,5,8$，不重不漏，正是路径 $8\to5\to9$ 上的所有点。

> 关键点：每次循环处理的区间是 `[dfn[top[u]], dfn[u]]`，处理完把 `u` 跳到 `fa[top[u]]`，**旧链上的点不会再被访问**，所以各段区间互不重叠。

---

## 5. 子树操作

因为第二次 DFS 是「先重儿子」，且 DFS 序天然保证子树连续，所以：

$$
\text{子树 } u \text{ 对应区间 } [\operatorname{dfn}(u),\ \operatorname{dfn}(u)+\operatorname{size}(u)-1]
$$

子树加、子树求和都是一次线段树区间操作，$O(\log n)$。

> 注意：子树操作与重链无关，任何 DFS 序都满足子树连续；但树链剖分的 DFS 序**同时**满足「重链连续」与「子树连续」，一举两得。

---

## 6. 完整模板代码

以「路径加 + 路径求和 + 子树加 + 子树求和」为例，点权初始值给定。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

const int MAXN = 100010;

int n, m, root;
ll a[MAXN];                   // 初始点权
vector<int> g[MAXN];

int fa[MAXN], dep[MAXN], sz[MAXN], son[MAXN];
int top[MAXN], dfn[MAXN], rnk[MAXN], timer;

// ---------- 第一次 DFS ----------
void dfs1(int u, int p) {
    fa[u] = p;
    dep[u] = dep[p] + 1;
    sz[u] = 1;
    son[u] = 0;
    int maxSize = 0;
    for (int v : g[u]) {
        if (v == p) continue;
        dfs1(v, u);
        sz[u] += sz[v];
        if (sz[v] > maxSize) { maxSize = sz[v]; son[u] = v; }
    }
}

// ---------- 第二次 DFS ----------
void dfs2(int u, int tp) {
    top[u] = tp;
    dfn[u] = ++timer;
    rnk[timer] = u;
    if (!son[u]) return;              // 叶子
    dfs2(son[u], tp);                 // 先走重儿子，保证同链连续
    for (int v : g[u]) {
        if (v == fa[u] || v == son[u]) continue;
        dfs2(v, v);                   // 轻儿子自己开一条新链
    }
}

// ---------- 线段树 ----------
ll sum[MAXN << 2], lazy[MAXN << 2];

void pushUp(int p) { sum[p] = sum[p << 1] + sum[p << 1 | 1]; }

void apply(int p, int l, int r, ll v) {
    sum[p] += v * (r - l + 1);
    lazy[p] += v;
}

void pushDown(int p, int l, int r) {
    if (lazy[p]) {
        int mid = (l + r) >> 1;
        apply(p << 1, l, mid, lazy[p]);
        apply(p << 1 | 1, mid + 1, r, lazy[p]);
        lazy[p] = 0;
    }
}

void build(int p, int l, int r) {
    if (l == r) { sum[p] = a[rnk[l]]; return; }
    int mid = (l + r) >> 1;
    build(p << 1, l, mid);
    build(p << 1 | 1, mid + 1, r);
    pushUp(p);
}

void update(int p, int l, int r, int ql, int qr, ll v) {
    if (ql <= l && r <= qr) { apply(p, l, r, v); return; }
    pushDown(p, l, r);
    int mid = (l + r) >> 1;
    if (ql <= mid) update(p << 1, l, mid, ql, qr, v);
    if (qr > mid) update(p << 1 | 1, mid + 1, r, ql, qr, v);
    pushUp(p);
}

ll query(int p, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) return sum[p];
    pushDown(p, l, r);
    int mid = (l + r) >> 1;
    ll res = 0;
    if (ql <= mid) res += query(p << 1, l, mid, ql, qr);
    if (qr > mid) res += query(p << 1 | 1, mid + 1, r, ql, qr);
    return res;
}

// ---------- 路径操作 ----------
void pathUpdate(int u, int v, ll w) {
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        update(1, 1, n, dfn[top[u]], dfn[u], w);
        u = fa[top[u]];
    }
    if (dep[u] > dep[v]) swap(u, v);
    update(1, 1, n, dfn[u], dfn[v], w);
}

ll pathQuery(int u, int v) {
    ll res = 0;
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        res += query(1, 1, n, dfn[top[u]], dfn[u]);
        u = fa[top[u]];
    }
    if (dep[u] > dep[v]) swap(u, v);
    res += query(1, 1, n, dfn[u], dfn[v]);
    return res;
}

// ---------- 子树操作 ----------
void subtreeUpdate(int u, ll w) { update(1, 1, n, dfn[u], dfn[u] + sz[u] - 1, w); }
ll subtreeQuery(int u) { return query(1, 1, n, dfn[u], dfn[u] + sz[u] - 1); }

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m >> root;
    for (int i = 1; i <= n; ++i) cin >> a[i];
    for (int i = 1; i < n; ++i) {
        int u, v; cin >> u >> v;
        g[u].push_back(v); g[v].push_back(u);
    }
    dfs1(root, 0);
    dfs2(root, root);
    build(1, 1, n);
    while (m--) {
        int op, x, y; ll z;
        cin >> op;
        if (op == 1) { cin >> x >> y >> z; pathUpdate(x, y, z); }
        else if (op == 2) { cin >> x >> y; cout << pathQuery(x, y) << '\n'; }
        else if (op == 3) { cin >> x >> z; subtreeUpdate(x, z); }
        else { cin >> x; cout << subtreeQuery(x) << '\n'; }
    }
    return 0;
}
```

### 6.1 复杂度

- 两次 DFS：$O(n)$；
- 线段树建树：$O(n)$；
- 每次路径操作：$O(\log^2 n)$（$O(\log n)$ 段，每段 $O(\log n)$ 线段树）；
- 每次子树操作：$O(\log n)$。

---

## 7. 经典例题

### 例题 1：【模板】树链剖分（Luogu P3384）

**题目大意**：给定一棵 $n$ 个点的树，点权初始值给定，支持四种操作：路径加、路径求和、子树加、子树求和，结果对 $p$ 取模。

**思路**：直接套第 6 节模板，线段树中所有加法与求和都取模。

**复杂度**：$O((n+m)\log^2 n)$。

---

### 例题 2：树上路径第 $k$ 小（Luogu P4175 [CTSC2008] 网络管理）

**题目大意**：给定一棵 $n$ 个点的树，每个点有一个权值。支持单点修改、查询路径 $u\to v$ 上第 $k$ 小的权值。

**思路**：树链剖分 + 可持久化线段树 / 树状数组套主席树：

1. 树剖把路径拆成 $O(\log n)$ 段连续区间；
2. 对每个区间用可持久化线段树求第 $k$ 小；
3. 多段合并时用「整体二分」或「可持久化线段树的多根同时查询」。

**复杂度**：$O(n\log^2 n + m\log^3 n)$（因实现方式而异）。

---

### 例题 3：软件包管理器（Luogu P2146 [NOI2015]）

**题目大意**：有 $n$ 个软件包，依赖关系构成一棵树（安装一个软件包必须先安装其所有祖先）。支持两种操作：

- `install x`：安装 $x$（若未安装），返回本次新安装的软件包数量；
- `uninstall x`：卸载 $x$ 及其子树中所有已安装的软件包，返回卸载数量。

**思路**：把「已安装」记为 1，「未安装」记为 0：

- `install x`：查询根到 $x$ 路径上 0 的个数，然后把路径全部置 1；
- `uninstall x`：查询子树 $x$ 中 1 的个数，然后把子树全部置 0。

树链剖分 + 线段树维护区间和与区间赋值懒标记。

**复杂度**：$O((n+m)\log^2 n)$。

```cpp
// install: 路径查询 + 路径赋值（模板中 dep[root]=1，故路径 root..x 共有 dep[x] 个点）
int install(int x) {
    int before = pathQuery(root, x);       // 路径上已有 1 的个数
    pathAssign(root, x, 1);
    return dep[x] - before;                // 路径总点数 - 已安装数 = 新安装数
}

// uninstall: 子树查询 + 子树赋值
int uninstall(int x) {
    int cnt = subtreeQuery(x);
    subtreeAssign(x, 0);
    return cnt;
}
```

---

### 例题 4：旅行（Luogu P3313 [SDOI2014]）

**题目大意**：一棵 $n$ 个点的树，每个点有颜色 $c_i$ 和权值 $w_i$。支持：

- 修改某点的颜色或权值；
- 查询路径 $u\to v$ 上颜色与 $u$ 相同的点的权值和 / 最大值。

**思路**：对每种颜色开一棵动态开点线段树（以 `dfn` 为下标），树剖把路径拆成 $O(\log n)$ 段后，在对应颜色的线段树上查询。修改颜色时从旧颜色树中删除、插入到新颜色树。

**复杂度**：$O((n+m)\log^2 n)$，空间 $O(n\log n)$。

---

## 8. 易错点

1. **`dfs1` 求 size 的顺序**：先递归儿子再累加 `sz[u]`，不要先加后递归。
2. **重儿子选择**：`sz[v] > maxSize` 而不是 `>=`（用 `>=` 会导致链划分退化，虽然正确性不受影响但复杂度可能变差，建议用 `>`）。
3. **`dfs2` 先走重儿子**：这是保证同一条重链 `dfn` 连续的关键，顺序错了整个算法就失效。
4. **轻儿子开新链**：`dfs2(v, v)`，不是 `dfs2(v, tp)`。
5. **路径操作最后的区间**：同一条链上时是 `[dfn[u], dfn[v]]`（需保证 `dep[u] <= dep[v]`），不要漏掉 LCA 本身。
6. **`dfn` 与 `rnk` 的对应**：`rnk[dfn[u]] = u`，建树时用 `a[rnk[l]]`。
7. **取模问题**：若题目要求取模，所有加法和乘法都要取模；`lazy` 也要取模。
8. **递归爆栈**：$n=10^5$ 时 DFS 递归可能爆栈，可以：
   - 开大栈（Windows 下 `-Wl,--stack=268435456`）；
   - 手写栈模拟；
   - 使用 BFS 求 size 和重儿子。
9. **多组数据**：清空所有数组与邻接表。

---

## 9. 练习建议

由易到难：

1. **模板**：Luogu P3384 【模板】树链剖分 —— 必须熟练到能默写；
2. **LCA 应用**：Luogu P3379 【模板】最近公共祖先 —— 用树剖求 LCA，体会 `top` 数组的作用；
3. **路径赋值**：Luogu P2146 [NOI2015] 软件包管理器 —— 区间赋值 + 路径查询；
4. **边权转点权**：Luogu P3038 [USACO11DEC] Grass Planting —— 把边权挂到儿子点上；
5. **动态开点**：Luogu P3313 [SDOI2014] 旅行 —— 每种颜色一棵线段树；
6. **第 k 小**：Luogu P4175 [CTSC2008] 网络管理 —— 树剖 + 主席树；
7. **进阶**：Luogu P3979 遥远的国度 —— 换根操作 + 树剖；
8. **综合**：Luogu P1505 [国家集训队] 旅游 —— 边权树剖 + 区间取反。

学习建议：

- 先画图理解「重链连续」这一核心性质，再写代码；
- 建议先写「路径求和」版本，跑通后再加懒标记、子树操作；
- 写完后用暴力程序对拍，特别注意边界：路径只有一个点、子树是叶子、根节点操作等。
