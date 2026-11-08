# 可持久化数据结构（Persistent Data Structure）

> 一句话概括：**可持久化数据结构在修改时保留旧版本，只新建被改动的那条路径上的节点、其余全部共享**，常用「主席树」（可持久化权值线段树）实现区间第 $k$ 小、可持久化数组等经典问题，单次操作时间与空间都是 $O(\log n)$。

---

## 目录

1. [什么是可持久化](#1-什么是可持久化)
2. [核心思想：路径复制](#2-核心思想路径复制)
3. [可持久化数组](#3-可持久化数组)
4. [主席树：可持久化权值线段树](#4-主席树可持久化权值线段树)
5. [完整模板：区间第 k 小](#5-完整模板区间第-k-小)
6. [进阶应用](#6-进阶应用)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [复杂度与空间估算](#9-复杂度与空间估算)
10. [练习建议](#10-练习建议)

---

## 1. 什么是可持久化

### 1.1 需求场景

很多问题会这样问：

- 每次修改后，询问某个历史版本的某个值；
- 给定区间 $[l,r]$，询问「只考虑第 $l$ 到第 $r$ 个元素」时的答案；
- 维护一个序列，每次操作基于上一个版本，可以回退到任意版本。

普通数据结构只有「当前状态」，一旦修改，旧状态就再也没有了。

**可持久化（persistent）** 就是让数据结构在修改时**保留历史版本**。

### 1.2 朴素做法及其问题

最朴素的做法：每次修改都把整个数据结构复制一份。时间 $O(n)$ 每次、$m$ 次操作 $O(nm)$，空间同样爆炸。

### 1.3 关键观察

一次单点修改，只会影响**从根到该位置的一条链**（$O(\log n)$ 个节点），其余节点**完全没变**。于是：**新建这条链上的节点，其余节点与旧版本共享**。这就是可持久化的全部秘密。

---

## 2. 核心思想：路径复制

### 2.1 图示

考虑一棵线段树，修改位置 3（下标 3 对应的叶子）：

```
旧版本 root0                新版本 root1
     [1,4]                      [1,4]'
    /     \                    /      \
 [1,2]   [3,4]              [1,2]    [3,4]'
         /   \                        /    \
       [3]   [4]                   [3]'   [4]
        |                            |
        旧值                        新值
```

- 只有 `[1,4]`、`[3,4]`、`[3]` 三个节点被复制；
- `[1,2]`、`[4]` 两个节点被两个版本**共享**；
- 修改一次的代价 $O(\log n)$。

### 2.2 代码框架

```cpp
int update(int oldRoot, int l, int r, int pos, int val) {
    int p = ++tot;                    // 新建节点
    t[p] = t[oldRoot];                // 先完整复制旧节点
    if (l == r) {
        t[p].sum = val;               // 叶子：直接改
        return p;
    }
    int mid = (l + r) >> 1;
    if (pos <= mid) t[p].ls = update(t[oldRoot].ls, l, mid, pos, val);
    else            t[p].rs = update(t[oldRoot].rs, mid + 1, r, pos, val);
    pushUp(p);
    return p;
}
```

**关键点**：`t[p] = t[oldRoot]` 先复制，保证未改动的儿子指向旧节点；只递归进入被修改的那一边，另一边保持共享；返回新节点编号，父节点用 `t[p].ls = ...` 接住。

---

## 3. 可持久化数组

### 3.1 问题

维护一个数组，支持：

- 单点修改，产生新版本；
- 查询某个历史版本某个位置的值；
- 版本号构成一棵版本树（每次基于某个旧版本修改）。

### 3.2 做法

用**可持久化线段树**维护数组：每个版本是一棵线段树，叶子存数组元素。

```cpp
const int MAXN = 1000005;
const int MAXM = 20000005;      // 节点池：n 个初始节点 + m * log n 个新节点

struct Node {
    int ls, rs;
    long long val;
} t[MAXM];

int root[MAXN];                 // root[v]：第 v 个版本的根
int tot = 0;

int build(int l, int r) {
    int p = ++tot;
    if (l == r) { t[p].val = a[l]; return p; }
    int mid = (l + r) >> 1;
    t[p].ls = build(l, mid);
    t[p].rs = build(mid + 1, r);
    return p;
}

int update(int old, int l, int r, int pos, long long val) {
    int p = ++tot;
    t[p] = t[old];
    if (l == r) { t[p].val = val; return p; }
    int mid = (l + r) >> 1;
    if (pos <= mid) t[p].ls = update(t[old].ls, l, mid, pos, val);
    else            t[p].rs = update(t[old].rs, mid + 1, r, pos, val);
    return p;
}

long long query(int p, int l, int r, int pos) {
    if (l == r) return t[p].val;
    int mid = (l + r) >> 1;
    if (pos <= mid) return query(t[p].ls, l, mid, pos);
    else            return query(t[p].rs, mid + 1, r, pos);
}
```

### 3.3 版本管理

```cpp
root[0] = build(1, n);
for (int i = 1; i <= m; ++i) {
    int v, op, pos;
    cin >> v >> op >> pos;      // 基于版本 v
    if (op == 1) {
        long long val; cin >> val;
        root[i] = update(root[v], 1, n, pos, val);
    } else {
        cout << query(root[v], 1, n, pos) << '\n';
        root[i] = root[v];      // 查询也产生一个版本（直接共享）
    }
}
```

版本号构成一棵树：每个版本可以从任意旧版本派生，所以「回退到版本 $v$」只是把根换成 `root[v]`，$O(1)$ 完成。若只关心「当前版本」，也可以只保存每次操作后的根。

---

## 4. 主席树：可持久化权值线段树

### 4.1 问题：静态区间第 $k$ 小

给定长度为 $n$ 的序列 $a$，$m$ 次询问 $[l,r]$ 中第 $k$ 小的数。

### 4.2 核心思路

**权值线段树**：下标是「值域」，节点维护「该值域内数字的个数」。

对每个前缀 $[1,i]$ 建一棵权值线段树，记为 `root[i]`。则区间 $[l,r]$ 对应的「计数」就是：

$$
\text{cnt}_{[l,r]}(v) = \text{cnt}_{[1,r]}(v) - \text{cnt}_{[1,l-1]}(v)
$$

即**两棵线段树相减**。

查询时在值域上二分：

- 令左半区间的计数 $c = \text{sz}(左_{r}) - \text{sz}(左_{l-1})$；
- 若 $c \ge k$，答案在左半，递归左儿子；
- 否则答案在右半，$k \leftarrow k - c$，递归右儿子。

```
值域 [1,5]，查询 [2,4] 第 2 小：

root[4] 计数： 1:1  2:1  3:2  4:1  5:1
root[1] 计数： 1:1  2:0  3:0  4:0  5:0
相减       ： 1:0  2:1  3:2  4:1  5:1
左半 [1,3] 共 3 个 >= 2 → 进入左半
左半的左半 [1,2] 共 1 个 < 2 → 进入 [3,3]，k = 2 - 1 = 1
答案 3
```

### 4.3 建树方式

从 `root[i-1]` 出发，在位置 $a_i$ 上「加 1」，得到 `root[i]`。这就是路径复制的可持久化修改，单次 $O(\log V)$。

```cpp
int update(int old, int l, int r, int pos) {
    int p = ++tot;
    t[p] = t[old];
    ++t[p].sz;                        // 该值域内数字个数 +1
    if (l == r) return p;
    int mid = (l + r) >> 1;
    if (pos <= mid) t[p].ls = update(t[old].ls, l, mid, pos);
    else            t[p].rs = update(t[old].rs, mid + 1, r, pos);
    return p;
}
```

### 4.4 查询

```cpp
int query(int u, int v, int l, int r, int k) {   // v 是右版本，u 是左版本
    if (l == r) return l;
    int mid = (l + r) >> 1;
    int leftCnt = t[t[v].ls].sz - t[t[u].ls].sz;
    if (k <= leftCnt) return query(t[u].ls, t[v].ls, l, mid, k);
    else              return query(t[u].rs, t[v].rs, mid + 1, r, k - leftCnt);
}
```

---

## 5. 完整模板：区间第 k 小

可直接编译的静态区间第 $k$ 小模板（值域离散化）：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 200005;
const int MAXM = MAXN * 20;        // 每个版本新增 log 个节点

int n, m;
int a[MAXN];
vector<int> vals;                  // 离散化后的值域

struct Node {
    int ls, rs, sz;
} t[MAXM];

int root[MAXN];
int tot = 0;

int update(int old, int l, int r, int pos) {
    int p = ++tot;
    t[p] = t[old];                 // 复制旧节点
    ++t[p].sz;                     // 计数 +1
    if (l == r) return p;
    int mid = (l + r) >> 1;
    if (pos <= mid) t[p].ls = update(t[old].ls, l, mid, pos);
    else            t[p].rs = update(t[old].rs, mid + 1, r, pos);
    return p;
}

int query(int u, int v, int l, int r, int k) {
    if (l == r) return l;
    int mid = (l + r) >> 1;
    int leftCnt = t[t[v].ls].sz - t[t[u].ls].sz;
    if (k <= leftCnt) return query(t[u].ls, t[v].ls, l, mid, k);
    else              return query(t[u].rs, t[v].rs, mid + 1, r, k - leftCnt);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> m;
    for (int i = 1; i <= n; ++i) { cin >> a[i]; vals.push_back(a[i]); }

    sort(vals.begin(), vals.end());
    vals.erase(unique(vals.begin(), vals.end()), vals.end());
    int V = (int)vals.size();

    for (int i = 1; i <= n; ++i) {
        int pos = (int)(lower_bound(vals.begin(), vals.end(), a[i]) - vals.begin()) + 1;
        root[i] = update(root[i - 1], 1, V, pos);
    }

    while (m--) {
        int l, r, k;
        cin >> l >> r >> k;
        int pos = query(root[l - 1], root[r], 1, V, k);   // 注意 u = l-1, v = r
        cout << vals[pos - 1] << '\n';
    }
    return 0;
}
```

### 5.1 为什么是 $O(\log n)$

- 建版本：从 `root[i-1]` 复制一条链，$O(\log V)$；
- 查询：在值域上二分，$O(\log V)$；
- 总时间 $O((n+m)\log V)$，空间 $O(n\log V)$。

---

## 6. 进阶应用

### 6.1 可持久化并查集

用可持久化数组维护每个点的 `fa` 和 `size`，路径压缩会破坏可持久化（修改太多节点），因此只能用**按秩合并**（每次只改两个位置）。单次操作 $O(\log n \cdot \log n)$。

### 6.2 可持久化 Trie

维护前缀版本的 01-Trie，查询区间异或最大值：用 `root[r]` 与 `root[l-1]` 的节点计数差判断某分支是否有元素。

```cpp
int query(int u, int v, int x) {        // [l,r] 内与 x 异或的最大值
    int res = 0;
    for (int b = 30; b >= 0; --b) {
        int bit = (x >> b) & 1;
        int want = bit ^ 1;
        int cntWant = t[t[v].ch[want]].sz - t[t[u].ch[want]].sz;
        if (cntWant > 0) {
            res |= (1 << b);
            u = t[u].ch[want];
            v = t[v].ch[want];
        } else {
            u = t[u].ch[bit];
            v = t[v].ch[bit];
        }
    }
    return res;
}
```

### 6.3 可持久化平衡树与线段树合并

- **可持久化平衡树**：用无旋 Treap 的 `split` / `merge` 做路径复制，每次只新建 $O(\log n)$ 个节点，用于「区间复制」「历史版本」类问题。
- **可持久化线段树合并**：在树上每个点建权值线段树，自底向上合并；若需保留每个子树的历史版本，可在合并时复制节点。

---

## 7. 经典例题

### 7.1 例题一：静态区间第 $k$ 小（模板题）

**题意**：给定 $n$ 个数，$m$ 次询问 $[l,r]$ 中第 $k$ 小的数。$n,m \le 2 \times 10^5$。

**思路**：主席树，见第 5 节。

**复杂度**：$O((n+m)\log n)$，空间 $O(n\log n)$。

### 7.2 例题二：可持久化数组

**题意**：维护数组，支持单点修改产生新版本、查询历史版本某个位置的值。操作数 $m \le 10^6$。

**思路**：可持久化线段树，见第 3 节。

**复杂度**：$O((n+m)\log n)$ 时间，$O((n+m)\log n)$ 空间。

> 注意：$m = 10^6$ 时节点数约 $2 \times 10^7$，必须开大数组并注意内存（可用 `int` 而非 `long long`）。

### 7.3 例题三：区间异或最大值

**题意**：给定序列，每次询问 $[l,r]$ 内选一个数 $a_i$ 使 $a_i \oplus x$ 最大。

**思路**：可持久化 01-Trie。`root[i]` 是前 $i$ 个数的 Trie 版本，查询时用 `root[r]` 与 `root[l-1]` 的计数差判断分支是否存在。

**复杂度**：$O((n+m)\log V)$。

### 7.4 例题四：区间内不同数的个数

**题意**：多次询问 $[l,r]$ 中有多少个不同的数。**思路**：对每个位置 $i$ 记录上一个相同数的位置 `pre[i]`，问题转化为「$[l,r]$ 中有多少个 $i$ 满足 `pre[i] < l`」。用主席树按 $i$ 建版本、按 `pre[i]` 为下标维护计数，查询 `root[r]` 中 `pre < l` 的个数。**复杂度**：$O((n+m)\log n)$。

---

## 8. 易错点

1. **节点池开小**：主席树节点数约为 $n \log n$，$n = 2\times10^5$ 时约 $4 \times 10^6$，要开 `MAXN * 20` 以上。
2. **忘记复制旧节点**：`t[p] = t[old]` 不能省，否则未修改的儿子会丢失。
3. **查询版本传反**：区间 $[l,r]$ 对应 `root[r]` 减 `root[l-1]`，注意是 $l-1$ 不是 $l$。
4. **整数溢出**：节点数、答案可能很大，用 `long long` 或注意 `int` 上限。
5. **离散化后忘记映射回来**：主席树查询返回的是离散化下标，要 `vals[pos-1]` 还原。
6. **重复元素**：权值线段树按「值域」建，重复元素计数即可，不需要去重（但离散化要去重）。
7. **多组数据不重置**：`tot` 要清零，`root` 数组要重置。
8. **内存超限**：`struct Node { int ls, rs, sz; }` 是 12 字节，`4e6` 个约 48 MB；若同时维护 `sum` 等会更大。
9. **递归深度**：主席树深度 $O(\log V)$，一般安全，但值域 $10^9$ 要动态开点而不是直接建满。
10. **可持久化并查集不能用路径压缩**：压缩会修改大量节点，破坏 $O(\log n)$ 的新增节点数。

---

## 9. 复杂度与空间估算

### 9.1 时间

| 操作 | 普通线段树 | 可持久化线段树 |
|---|---|---|
| 建树 / 单点修改 | $O(n)$ / $O(\log n)$ | $O(n\log V)$（$n$ 个版本）/ $O(\log n)$ 新建节点 |
| 区间查询 / 版本回退 | $O(\log n)$ | $O(\log n)$ / $O(1)$（换根） |

### 9.2 空间

节点数估算约为 $n\lceil\log_2 V\rceil + n$。$n = 2 \times 10^5$、$V = 2 \times 10^5$（$\log V \approx 18$）时约 $4 \times 10^6$ 个节点，每个节点 3 个 `int`（12 字节）约 48 MB。

**优化技巧**：用 `int` 而不是 `long long` 存 `sz`；用数组而不是 `struct` 指针；若只需要查第 $k$ 小，可以不存 `sum`，只存 `sz`。

### 9.3 主席树 vs 分块 / 树套树

| 方案 | 建树 | 查询 | 空间 | 代码 |
|---|---|---|---|---|
| 主席树 | $O(n\log n)$ | $O(\log n)$ | $O(n\log n)$ | 中 |
| 分块 + 二分 | $O(n\sqrt{n})$ | $O(\sqrt{n}\log n\log V)$ | $O(n)$ | 短 |
| 树套树 | $O(n\log^2 n)$ | $O(\log^2 n)$ | $O(n\log n)$ | 长 |
| 整体二分 | $O((n+m)\log n)$ | 离线 | $O(n)$ | 中 |

---

## 10. 练习建议

### 10.1 由易到难

| 难度 | 题目类型 | 考察点 |
|---|---|---|
| ★★ | 可持久化数组 | 路径复制入门 |
| ★★ | 静态区间第 $k$ 小 | 主席树模板 |
| ★★★ | 区间不同数个数 | 主席树 + 转化 |
| ★★★ | 区间异或最大值 | 可持久化 01-Trie |
| ★★★★ | 可持久化并查集 | 按秩合并 |
| ★★★★ | 可持久化平衡树 | 无旋 Treap |
| ★★★★★ | 树上主席树 / 路径第 k 小 | 树上前缀和 |

### 10.2 学习检查清单

- [ ] 能画出「路径复制」的示意图，说清为什么一次修改只新建 $O(\log n)$ 个节点
- [ ] 能默写可持久化线段树的 `update`，解释主席树查询时「两树相减」的含义
- [ ] 能估算主席树的节点数与内存，知道可持久化并查集为什么不能路径压缩

### 10.3 推荐练习顺序

1. 洛谷 P3919【模板】可持久化数组
2. 洛谷 P3834【模板】可持久化线段树 2（区间第 k 小）
3. 洛谷 P1972 [SDOI2009] HH 的项链（区间不同数）
4. 洛谷 P4735 最大异或和（可持久化 01-Trie）
5. 洛谷 P3402【模板】可持久化并查集
6. 洛谷 P2633 Count on a tree（树上主席树）

---

> **核心思想**：可持久化用「路径复制 + 节点共享」把每次修改的额外空间压到 $O(\log n)$。主席树是它在权值线段树上的直接应用，把「区间第 $k$ 小」变成「两个版本的差」，是离线静态区间问题的经典解法。
