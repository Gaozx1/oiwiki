# 线段树（Segment Tree）

> 一句话概括：**线段树把区间递归二分建树，用 $O(\log n)$ 个节点拼出任意区间**，从而在 $O(\log n)$ 内完成区间修改与区间查询，是处理「区间信息可合并」问题的通用利器。

---

## 目录

1. [什么是线段树](#1-什么是线段树)
2. [结构原理](#2-结构原理)
3. [建树与区间查询](#3-建树与区间查询)
4. [区间修改与懒标记](#4-区间修改与懒标记)
5. [完整模板代码](#5-完整模板代码)
6. [进阶：动态开点与线段树合并](#6-进阶动态开点与线段树合并)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 什么是线段树

### 1.1 问题引入

给定长度为 $n$ 的数组 $a$，需要支持：

- **单点修改**：`a[i] += x`
- **区间查询**：求 `a[l..r]` 的和、最大值、最小值……

朴素做法单次查询 $O(n)$；前缀和能把查询降到 $O(1)$，但**任何一次修改都会让所有后缀失效**，修改退化为 $O(n)$。我们需要让修改和查询都在 $O(\log n)$ 内完成——这就是线段树。

### 1.2 核心思想：分治

把区间 $[1,n]$ 不断二分，得到一棵二叉树：

```
                     [1,8]
                 /          \
             [1,4]           [5,8]
            /     \         /     \
         [1,2]   [3,4]   [5,6]   [7,8]
         /  \    /  \    /  \    /  \
       [1] [2] [3] [4] [5] [6] [7] [8]
```

- 每个节点代表一个区间，**父区间 = 左儿子 ∪ 右儿子**，叶子是单个元素；
- **任意区间 $[l,r]$ 都能拆成 $O(\log n)$ 个树节点区间的不交并**；
- 每个节点维护自己区间的答案，合并左右儿子即可得到父亲答案。

### 1.3 信息可合并性

线段树维护的信息必须**可合并**：知道左右半区的答案就能快速算出整段答案。

| 维护信息 | 合并方式 | 是否适合 |
|---|---|---|
| 区间和 | `sum = sumL + sumR` | ✅ |
| 区间最值 | `mx = max(mxL, mxR)` | ✅ |
| 区间 gcd | `g = gcd(gL, gR)` | ✅ |
| 区间不同数个数 | 无简单合并 | ❌（改用莫队/主席树） |

---

## 2. 结构原理

### 2.1 节点编号

根编号 1，左儿子 `2i`，右儿子 `2i+1`（与堆一致）。区间 $[1,n]$ 需要开 `4*n` 的数组（实际最大下标约 $4n$）。

### 2.2 节点存什么

```cpp
struct Node {
    long long sum;   // 区间和
    long long mx;    // 区间最大值
    long long tag;   // 懒标记：待下传给儿子的加法量
};
Node tr[N << 2];
```

### 2.3 关键性质

设当前节点区间 $[l,r]$，$mid=\lfloor(l+r)/2\rfloor$：左儿子管 $[l,mid]$，右儿子管 $[mid+1,r]$。递归深度 $O(\log n)$，从根走到叶子约 $\log_2 n$ 步。

---

## 3. 建树与区间查询

### 3.1 建树

```cpp
void pushUp(int p) {                 // 用儿子更新父亲
    tr[p].sum = tr[p << 1].sum + tr[p << 1 | 1].sum;
    tr[p].mx  = max(tr[p << 1].mx, tr[p << 1 | 1].mx);
}

void build(int p, int l, int r) {
    if (l == r) {                    // 叶子：直接赋值
        tr[p].sum = tr[p].mx = a[l];
        return;
    }
    int mid = (l + r) >> 1;
    build(p << 1, l, mid);           // 建左子树
    build(p << 1 | 1, mid + 1, r);   // 建右子树
    pushUp(p);                       // 回溯时合并信息
}
```

共 $O(n)$ 个节点，每节点常数时间，建树 $O(n)$。

### 3.2 区间查询的三种关系

查询区间 $[ql,qr]$ 与当前节点区间 $[l,r]$ 的关系：

1. **完全无关**：返回单位元（求和为 0，最值为 $-\infty$）；
2. **完全包含**：$ql \le l$ 且 $r \le qr$，直接返回当前节点答案；
3. **部分相交**：递归左右儿子，把两边答案合并。

```
查询 [3,6]，根 [1,8]：
           [1,8] 部分
          /      \
      [1,4] 部分   [5,8] 部分
      /   \        /   \
   [1,2] 无   [3,4] 全含  [5,6] 全含  [7,8] 无
              ↑           ↑
           直接返回      直接返回
```

### 3.3 代码

```cpp
long long querySum(int p, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) return tr[p].sum;   // 完全包含
    int mid = (l + r) >> 1;
    long long res = 0;
    if (ql <= mid) res += querySum(p << 1, l, mid, ql, qr);
    if (qr > mid)  res += querySum(p << 1 | 1, mid + 1, r, ql, qr);
    return res;
}
```

用「若相交才递归」代替显式处理无关分支，写法更简洁。查询把目标区间拆成 $O(\log n)$ 个完整节点，每层最多访问 4 个节点，复杂度 $O(\log n)$。

---

## 4. 区间修改与懒标记

### 4.1 为什么需要懒标记

若区间修改也递归到每个叶子，单次会退化到 $O(n)$。**懒标记思想**：若当前节点区间被**完全覆盖**，只更新当前节点答案并打标记，表示「这个区间每个元素都还欠一次操作」，**暂不往下传**；等以后真正访问儿子时再下传。

> 类比：老师在班级群里发一次通知（打标记），不逐个通知学生（不下传）；等学生来问问题时才转告（下传）。

### 4.2 下传与修改

```cpp
void apply(int p, long long v, int len) {  // 对节点 p 施加「区间加 v」
    tr[p].sum += v * len;                  // 区间和增加 v * 长度
    tr[p].mx  += v;                        // 最值增加 v
    tr[p].tag += v;                        // 标记可叠加
}

void pushDown(int p, int l, int r) {
    if (tr[p].tag == 0) return;
    int mid = (l + r) >> 1;
    apply(p << 1, tr[p].tag, mid - l + 1);      // 传给左儿子
    apply(p << 1 | 1, tr[p].tag, r - mid);      // 传给右儿子
    tr[p].tag = 0;                              // 清除当前标记
}

void add(int p, int l, int r, int ql, int qr, long long v) {
    if (ql <= l && r <= qr) {      // 完全包含：打标记后返回
        apply(p, v, r - l + 1);
        return;
    }
    pushDown(p, l, r);             // 进入儿子前必须下传
    int mid = (l + r) >> 1;
    if (ql <= mid) add(p << 1, l, mid, ql, qr, v);
    if (qr > mid)  add(p << 1 | 1, mid + 1, r, ql, qr, v);
    pushUp(p);                     // 回溯更新
}
```

### 4.3 带标记的查询

查询遇到有标记的节点，同样要先 `pushDown`，否则儿子里的信息是旧的。只需在普通查询的「部分相交」分支前加一行 `pushDown(p, l, r);` 即可，完整写法见第 5 节 `querySum`。

---

## 5. 完整模板代码

支持「区间加 + 区间赋值 + 区间求和 + 区间最大值」，可直接编译：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100005;
const long long NONE = LLONG_MIN / 4;   // 表示「无赋值标记」

struct Node {
    long long sum, mx;
    long long addTag;     // 加法懒标记
    long long setTag;     // 赋值懒标记，NONE 表示无
};

Node tr[N << 2];
long long a[N];
int n, m;

void pushUp(int p) {
    tr[p].sum = tr[p << 1].sum + tr[p << 1 | 1].sum;
    tr[p].mx  = max(tr[p << 1].mx, tr[p << 1 | 1].mx);
}

void build(int p, int l, int r) {
    tr[p].addTag = 0;
    tr[p].setTag = NONE;
    if (l == r) { tr[p].sum = tr[p].mx = a[l]; return; }
    int mid = (l + r) >> 1;
    build(p << 1, l, mid);
    build(p << 1 | 1, mid + 1, r);
    pushUp(p);
}

void applySet(int p, long long v, int len) {   // 区间赋值
    tr[p].sum = v * len;
    tr[p].mx  = v;
    tr[p].setTag = v;
    tr[p].addTag = 0;                          // 赋值覆盖之前的加法
}

void applyAdd(int p, long long v, int len) {   // 区间加
    tr[p].sum += v * len;
    tr[p].mx  += v;
    if (tr[p].setTag != NONE) tr[p].setTag += v;
    else                      tr[p].addTag += v;
}

void pushDown(int p, int l, int r) {
    int mid = (l + r) >> 1;
    int lenL = mid - l + 1, lenR = r - mid;
    if (tr[p].setTag != NONE) {                // 先处理赋值
        applySet(p << 1, tr[p].setTag, lenL);
        applySet(p << 1 | 1, tr[p].setTag, lenR);
        tr[p].setTag = NONE;
    }
    if (tr[p].addTag != 0) {                   // 再处理加法
        applyAdd(p << 1, tr[p].addTag, lenL);
        applyAdd(p << 1 | 1, tr[p].addTag, lenR);
        tr[p].addTag = 0;
    }
}

void rangeSet(int p, int l, int r, int ql, int qr, long long v) {
    if (ql <= l && r <= qr) { applySet(p, v, r - l + 1); return; }
    pushDown(p, l, r);
    int mid = (l + r) >> 1;
    if (ql <= mid) rangeSet(p << 1, l, mid, ql, qr, v);
    if (qr > mid)  rangeSet(p << 1 | 1, mid + 1, r, ql, qr, v);
    pushUp(p);
}

void rangeAdd(int p, int l, int r, int ql, int qr, long long v) {
    if (ql <= l && r <= qr) { applyAdd(p, v, r - l + 1); return; }
    pushDown(p, l, r);
    int mid = (l + r) >> 1;
    if (ql <= mid) rangeAdd(p << 1, l, mid, ql, qr, v);
    if (qr > mid)  rangeAdd(p << 1 | 1, mid + 1, r, ql, qr, v);
    pushUp(p);
}

long long querySum(int p, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) return tr[p].sum;
    pushDown(p, l, r);
    int mid = (l + r) >> 1;
    long long res = 0;
    if (ql <= mid) res += querySum(p << 1, l, mid, ql, qr);
    if (qr > mid)  res += querySum(p << 1 | 1, mid + 1, r, ql, qr);
    return res;
}

long long queryMax(int p, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) return tr[p].mx;
    pushDown(p, l, r);
    int mid = (l + r) >> 1;
    long long res = NONE;
    if (ql <= mid) res = max(res, queryMax(p << 1, l, mid, ql, qr));
    if (qr > mid)  res = max(res, queryMax(p << 1 | 1, mid + 1, r, ql, qr));
    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) cin >> a[i];
    build(1, 1, n);
    while (m--) {
        int op, l, r;
        cin >> op >> l >> r;
        if (op == 1) { long long v; cin >> v; rangeAdd(1, 1, n, l, r, v); }
        else if (op == 2) { long long v; cin >> v; rangeSet(1, 1, n, l, r, v); }
        else if (op == 3) cout << querySum(1, 1, n, l, r) << '\n';
        else cout << queryMax(1, 1, n, l, r) << '\n';
    }
    return 0;
}
```

> **标记叠加顺序**：同一节点既有赋值又有加法标记时，必须**先赋值后加法**。代码里 `applyAdd` 遇到赋值标记就把赋值标记加 $v$，从而保证语义正确。

---

## 6. 进阶：动态开点与线段树合并

### 6.1 动态开点

值域很大（如 $10^9$）而实际点很少时，`4n` 开不下，可**用到哪个节点才创建哪个节点**。每个节点额外记录 `ls, rs` 两个儿子编号，`update(int& p, int l, int r, int pos, long long v)` 中若 `p == 0` 就新建节点，递归到叶子后回溯更新 `sum`。单次操作新建 $O(\log V)$ 个节点，总空间 $O(m \log V)$。

```cpp
int update(int& p, int l, int r, int pos, long long v) {
    if (!p) p = newNode();                       // 用到才创建
    if (l == r) { t[p].sum += v; return p; }
    int mid = (l + r) >> 1;
    if (pos <= mid) t[p].ls = update(t[p].ls, l, mid, pos, v);
    else            t[p].rs = update(t[p].rs, mid + 1, r, pos, v);
    t[p].sum = t[t[p].ls].sum + t[t[p].rs].sum;
    return p;
}
```

### 6.2 线段树合并

把两棵动态开点线段树对应位置相加：设 `merge(a, b, l, r)` 返回合并后的根，若 `a` 或 `b` 为空则直接返回非空的那一个；否则递归合并左右儿子并把 `sum` 相加，返回 `a`。总复杂度为两树重叠节点数，配合树上启发式合并可做到 $O(n \log n)$，常用于「每个点一棵权值线段树、自底向上合并」的题目。

---

## 7. 经典例题

### 7.1 区间加、区间求和（模板题）

**题意**：长度 $n$ 的序列，$m$ 次操作：区间每个数加 $v$，或询问区间和。$n,m \le 10^5$。

**思路**：套用第 5 节模板，维护 `sum` 与 `addTag`，修改时把区间和增加 $v \times (r-l+1)$。

**复杂度**：每次 $O(\log n)$，总 $O((n+m)\log n)$。

### 7.2 区间最大子段和

**题意**：维护序列，支持单点修改与查询 $[l,r]$ 的最大子段和（连续且非空）。

**思路**：每个节点维护四个量——`sum` 总和、`pre` 最大前缀和、`suf` 最大后缀和、`best` 最大子段和。合并左 $L$ 右 $R$：

$$
\begin{aligned}
sum &= L.sum + R.sum\\
pre &= \max(L.pre,\ L.sum + R.pre)\\
suf &= \max(R.suf,\ R.sum + L.suf)\\
best &= \max(L.best,\ R.best,\ L.suf + R.pre)
\end{aligned}
$$

**复杂度**：每次 $O(\log n)$。

```cpp
struct Info { long long sum, pre, suf, best; };

Info mergeInfo(const Info& L, const Info& R) {
    Info res;
    res.sum  = L.sum + R.sum;
    res.pre  = max(L.pre, L.sum + R.pre);
    res.suf  = max(R.suf, R.sum + L.suf);
    res.best = max({L.best, R.best, L.suf + R.pre});
    return res;
}
```

> 这是「信息可合并」的典型：看似复杂的最大子段和，补足前缀/后缀信息就能合并。

### 7.3 扫描线求矩形面积并

**题意**：给出 $n$ 个与坐标轴平行的矩形，求覆盖总面积。$n \le 10^5$。

**思路**：把每个矩形拆成两条竖直边（左界 +1、右界 -1），按 $x$ 排序后从左往右扫描。线段树维护 $y$ 轴被覆盖的长度：节点存 `cover`（被完整覆盖次数）与 `len`（实际覆盖长度）。若 `cover > 0` 则 `len` 为区间长度，否则为左右儿子 `len` 之和。每次用「当前 $x$ − 上一条边 $x$」乘当前覆盖长度累加。

**复杂度**：$O(n \log n)$。

```cpp
void update(int p, int l, int r, int ql, int qr, int v) {
    if (ql <= l && r <= qr) {
        tr[p].cover += v;
    } else {
        int mid = (l + r) >> 1;
        if (ql <= mid) update(p << 1, l, mid, ql, qr, v);
        if (qr > mid)  update(p << 1 | 1, mid + 1, r, ql, qr, v);
    }
    if (tr[p].cover > 0) tr[p].len = ys[r + 1] - ys[l];
    else if (l == r)      tr[p].len = 0;
    else                  tr[p].len = tr[p << 1].len + tr[p << 1 | 1].len;
}
```

---

## 8. 易错点

1. **数组开小**：静态线段树要开 `4 * n`，不是 `2 * n`；动态开点按操作数乘 $\log$ 估算。
2. **忘记 `pushDown`**：进入儿子前必须下传；**查询时同样需要**，否则读到旧值。
3. **标记顺序**：赋值与加法共存时，必须先赋值后加法。
4. **忘记 `pushUp`**：修改完儿子返回前必须更新父亲。
5. **边界写错**：左儿子 $[l,mid]$、右儿子 $[mid+1,r]$，判断用 `ql <= mid` 与 `qr > mid`。
6. **单位元选错**：求和用 0，最值用极小值，注意不要用 `INT_MIN` 导致溢出。
7. **`long long` 溢出**：区间和、面积等会超过 `int`。
8. **多组数据不清空**：`tr` 与标记都要重置，建议在 `build` 中统一初始化。
9. **`apply` 长度算错**：区间长度是 $r-l+1$，不是 $mid-l+1$；`p<<1|1` 的优先级低于 `+`，必要时加括号。

---

## 9. 练习建议

### 9.1 由易到难

| 难度 | 题目类型 | 考察点 |
|---|---|---|
| ★ | 单点修改 + 区间查询 | 基本框架 |
| ★★ | 区间加 + 区间求和 | 懒标记入门 |
| ★★ | 区间最大子段和 | 信息合并 |
| ★★★ | 扫描线求面积并 | 线段树 + 离散化 |
| ★★★ | 区间乘 + 区间加 | 双标记顺序 |
| ★★★★ | 动态开点 + 线段树合并 | 空间优化 |
| ★★★★ | 线段树维护矩阵 / 复杂信息 | 抽象建模 |
| ★★★★★ | 线段树分治、李超线段树 | 进阶技巧 |

### 9.2 学习检查清单

- [ ] 能默写 `build` / `query` / `update` / `pushDown` / `pushUp`，并说清查询与修改为什么是 $O(\log n)$
- [ ] 能说清懒标记「先赋值后加法」的原因，能手写区间最大子段和的合并函数；理解动态开点与普通线段树的空间差异

### 9.3 推荐练习顺序

1. 洛谷 P3372【模板】线段树 1（区间加 + 区间求和）
2. 洛谷 P3373【模板】线段树 2（区间乘 + 区间加）
3. 洛谷 P4513 小白逛公园（最大子段和）
4. 洛谷 P5490【模板】扫描线（矩形面积并）
5. 洛谷 P4556 雨天的尾巴（线段树合并）

---

> **核心思想**：线段树把区间递归二分，用 $O(\log n)$ 个节点表示任意区间，再配合懒标记把区间修改压到 $O(\log n)$。掌握它，就掌握了所有「区间可合并信息」问题的通用解法。
