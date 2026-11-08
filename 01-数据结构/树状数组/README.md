# 树状数组（Binary Indexed Tree, BIT）

> 一句话概括：**树状数组用「二进制拆分」把前缀和拆成 $O(\log n)$ 段**，在 $O(\log n)$ 内完成单点修改与区间查询，代码极短、常数极小。

---

## 目录

1. [概念与原理](#1-概念与原理)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点](#4-易错点)
5. [练习建议](#5-练习建议)

---

## 1. 概念与原理

### 1.1 从「前缀和」的两难说起

维护数组 $a_1 \dots a_n$，要支持「单点修改」和「区间求和」。两种朴素做法各有致命缺陷：

| 做法 | 单点修改 | 前缀和查询 |
|---|---|---|
| 普通数组 | $O(1)$ | $O(n)$ |
| 前缀和数组 | $O(n)$（后面全要改） | $O(1)$ |

树状数组把两者折中到 $O(\log n)$，同时代码量远小于线段树。

### 1.2 核心思想：每个结点管理一段区间

核心是 **lowbit**：

$$
\operatorname{lowbit}(x) = x \ \& \ (-x)
$$

它取出 $x$ 二进制中**最低位的 1** 及其后面的 0。例如 `1→1, 2→2, 3→1, 4→4, 5→1, 6→2, 7→1, 8→8`。

树状数组的 `tree[i]` 管理的是**以 $i$ 结尾、长度为 $\operatorname{lowbit}(i)$ 的区间**：

$$
tree[i] = \sum_{k = i - \operatorname{lowbit}(i) + 1}^{i} a_k
$$

也就是：`tree[i]` 的管辖范围是 $(i - \text{lowbit}(i),\ i]$，长度为 $\text{lowbit}(i)$。

### 1.3 文字示意图

以 $n = 8$ 为例，各结点的管辖区间：

```
下标 i:      1     2     3     4     5     6     7     8
lowbit:      1     2     1     4     1     2     1     8
tree[i]管:  [1,1] [1,2] [3,3] [1,4] [5,5] [5,6] [7,7] [1,8]
```

画成树形（`i` 的父结点是 `i + lowbit(i)`，父结点管辖范围包含子结点）：

```
                       tree[8] = a[1..8]
                     /       |        \
          tree[4]          tree[6]        tree[7]
         = a[1..4]        = a[5..6]      = a[7..7]
         /      \        /
    tree[2]    tree[3] tree[5]
   = a[1..2]   = a[3]  = a[5..5]
     /
  tree[1]
  = a[1..1]
```

**`tree[i]` 负责的区间，等于从 $i$ 往前数 $\text{lowbit}(i)$ 个元素。**

### 1.4 两个基本操作

**（1）单点修改 `add(i, x)`：向上跳**

修改 $a_i$ 会影响所有管辖范围包含 $i$ 的结点，从 $i$ 开始每次加上 `lowbit(i)`：$i \leftarrow i + \operatorname{lowbit}(i)$。

例如修改 $a_3$：$3 \to 4 \to 8 \to 16$，更新 `tree[3], tree[4], tree[8]`（三者都包含 $a_3$）。

**（2）前缀和查询 `query(i)`：向下跳**

要求 $\sum_{k=1}^{i} a_k$，每次累加 `tree[i]` 后令 $i \leftarrow i - \operatorname{lowbit}(i)$，直到 $i = 0$。例如：

```
S7 = tree[7] + tree[6] + tree[4]
   = a[7] + (a[5]+a[6]) + (a[1]+a[2]+a[3]+a[4])
   = a[1..7]  ✓
```

二进制视角：$7=111_2$，`lowbit(7)=1` 得 $110_2=6$；`lowbit(6)=2` 得 $100_2=4$；`lowbit(4)=4` 得 $0$。每次恰好去掉一个二进制 1，所以最多 $O(\log n)$ 轮。

### 1.5 区间查询与复杂度

前缀和查询得到 $S_i$ 后，$\sum_{k=l}^{r} a_k = S_r - S_{l-1}$。

| 操作 | 复杂度 | 说明 |
|---|---|---|
| 单点修改 | $O(\log n)$ | 向上跳，最多 $\log n$ 个结点 |
| 前缀和 / 区间和查询 | $O(\log n)$ | 向下跳 / 两次前缀和 |
| 建树 | $O(n)$ | 递推 `tree[i]` 贡献给父亲 |
| 空间 | $O(n)$ | 一个数组 |

| 对比项 | 树状数组 | 线段树 |
|---|---|---|
| 代码量 / 常数 | 极短 / 很小 | 较长 / 较大 |
| 支持操作 | 可减信息：和、异或、计数 | 最值、区间赋值、懒标记 |
| 区间改 + 区间查 | 差分 + 两个树状数组 | 懒标记 |
| 动态开点 / 可持久化 | 困难 | 支持 |

**关键判断**：信息满足「区间可减」（和、异或）就优先树状数组；要求区间最值（不可减）只能线段树或 ST 表。

---

## 2. 模板代码

### 2.1 最简版：单点修改 + 区间查询

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 500005;
int n;
long long tree[MAXN];      // tree[i] 管理 (i - lowbit(i), i] 的区间和

inline int lowbit(int x) { return x & -x; }

void add(int i, long long x) {                    // a[i] += x
    for (; i <= n; i += lowbit(i)) tree[i] += x;  // 向上跳，更新所有祖先
}

long long query(int i) {                          // a[1] + ... + a[i]
    long long s = 0;
    for (; i > 0; i -= lowbit(i)) s += tree[i];   // 向下跳，累加管辖块
    return s;
}

long long rangeSum(int l, int r) { return query(r) - query(l - 1); }

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int m;
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) { long long x; cin >> x; add(i, x); }
    while (m--) {
        int op, x, y;
        cin >> op >> x >> y;
        if (op == 1) add(x, y);                 // a[x] += y
        else cout << rangeSum(x, y) << '\n';    // 区间和
    }
    return 0;
}
```

输入 `5 4 / 1 2 3 4 5 / 2 1 3 / 1 2 10 / 2 1 3 / 2 2 2`，输出 `6 / 16 / 12`。

### 2.2 $O(n)$ 建树

上面的 `main` 用 $n$ 次 `add` 建树是 $O(n \log n)$，可以优化到 $O(n)$：先把 `a[i]` 放进 `tree[i]`，再把自己的整段贡献推给父亲 `i + lowbit(i)`。

```cpp
void build(const vector<long long>& a) {   // a 下标从 1 开始
    for (int i = 1; i <= n; ++i) {
        tree[i] += a[i];
        int j = i + lowbit(i);
        if (j <= n) tree[j] += tree[i];
    }
}
```

### 2.3 区间修改 + 单点查询（差分树状数组）

维护差分数组 $d_i = a_i - a_{i-1}$：区间 $[l, r]$ 加 $x$ 就 `add(l, x); add(r+1, -x);`；查询 $a_i$ 就是 `query(i)`。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 500005;
int n;
long long tree[MAXN];
inline int lowbit(int x) { return x & -x; }

void add(int i, long long x) { for (; i <= n; i += lowbit(i)) tree[i] += x; }
long long query(int i) {
    long long s = 0;
    for (; i > 0; i -= lowbit(i)) s += tree[i];
    return s;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int m;
    cin >> n >> m;
    long long last = 0;
    for (int i = 1; i <= n; ++i) {
        long long x;
        cin >> x;
        add(i, x - last);          // 建差分数组
        last = x;
    }
    while (m--) {
        int op;
        cin >> op;
        if (op == 1) {
            int l, r;
            long long x;
            cin >> l >> r >> x;
            add(l, x);
            add(r + 1, -x);        // r+1 可能为 n+1，循环自然跳过
        } else {
            int i;
            cin >> i;
            cout << query(i) << '\n';
        }
    }
    return 0;
}
```

### 2.4 区间修改 + 区间查询（两个树状数组）

由差分 $d_i$ 推导：

$$
\sum_{i=1}^{x} a_i = \sum_{j=1}^{x} d_j (x - j + 1) = (x+1)\sum_{j=1}^{x} d_j - \sum_{j=1}^{x} j \cdot d_j
$$

于是维护 `t1` 存 $d_i$、`t2` 存 $i \cdot d_i$；前缀和 $S_x = (x+1)\cdot\text{t1.query}(x) - \text{t2.query}(x)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 500005;
int n;
long long t1[MAXN], t2[MAXN];
inline int lowbit(int x) { return x & -x; }

void add(long long t[], int i, long long x) {
    for (; i <= n; i += lowbit(i)) t[i] += x;
}
long long query(long long t[], int i) {
    long long s = 0;
    for (; i > 0; i -= lowbit(i)) s += t[i];
    return s;
}
void rangeAdd(int l, int r, long long x) {
    add(t1, l, x);          add(t1, r + 1, -x);
    add(t2, l, l * x);      add(t2, r + 1, -(r + 1) * x);
}
long long prefixSum(int x) { return (x + 1) * query(t1, x) - query(t2, x); }
long long rangeSum(int l, int r) { return prefixSum(r) - prefixSum(l - 1); }

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int m;
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) { long long x; cin >> x; rangeAdd(i, i, x); }
    while (m--) {
        int op, l, r;
        cin >> op >> l >> r;
        if (op == 1) { long long x; cin >> x; rangeAdd(l, r, x); }
        else cout << rangeSum(l, r) << '\n';
    }
    return 0;
}
```

---

## 3. 经典例题

### 3.1 树状数组模板（单点修改 + 区间查询）

**题意**：给定数列，支持「单点加」和「区间求和」。**思路**：见 2.1。**复杂度**：每次操作 $O(\log n)$，总 $O((n+m)\log n)$。

### 3.2 逆序对（权值树状数组）

**题意**：给定长度为 $n$ 的序列，求满足 $i < j$ 且 $a_i > a_j$ 的二元组数量。

**思路**：从左到右扫描，树状数组维护「每个值已出现几次」。处理到 $a_i$ 时，已出现的比它大的元素个数为 $(i-1) - \text{query}(a_i)$，累加即可。值域大时先**离散化**。

**复杂度**：$O(n \log n)$，空间 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n;
vector<int> tree;
inline int lowbit(int x) { return x & -x; }
void add(int i, int x) { for (; i <= n; i += lowbit(i)) tree[i] += x; }
int query(int i) {
    int s = 0;
    for (; i > 0; i -= lowbit(i)) s += tree[i];
    return s;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n;
    vector<int> a(n + 1), vals;
    vals.reserve(n);
    for (int i = 1; i <= n; ++i) { cin >> a[i]; vals.push_back(a[i]); }

    sort(vals.begin(), vals.end());                  // 离散化到 1..n
    vals.erase(unique(vals.begin(), vals.end()), vals.end());
    for (int i = 1; i <= n; ++i)
        a[i] = lower_bound(vals.begin(), vals.end(), a[i]) - vals.begin() + 1;

    tree.assign(n + 1, 0);
    long long ans = 0;
    for (int i = 1; i <= n; ++i) {
        ans += (i - 1) - query(a[i]);   // 已插入元素中比 a[i] 大的个数
        add(a[i], 1);
    }
    cout << ans << '\n';
    return 0;
}
```

### 3.3 第 k 小（树状数组上二分）

**题意**：维护可重集合，支持插入、删除、查询第 $k$ 小。

**思路**：权值树状数组记录每个值出现次数。查询第 $k$ 小时从高位到低位倍增：若累加后的前缀计数仍小于 $k$ 就跳过该段，最后位置 $+1$ 即答案。前提是**前缀计数单调不减**，只能用于计数类信息。

```cpp
int kth(int k) {                  // 第 k 小，k 从 1 开始
    int pos = 0, step = 1;
    while ((step << 1) <= MAXV) step <<= 1;   // 最大的 2 的幂
    for (; step; step >>= 1) {
        int nxt = pos + step;
        if (nxt <= MAXV && tree[nxt] < k) { pos = nxt; k -= tree[nxt]; }
    }
    return pos + 1;
}
```

### 3.4 二维树状数组（单点修改 + 子矩阵求和）

**题意**：$n \times m$ 矩阵，支持单点加、查询左上角 $(1,1)$ 到 $(x,y)$ 的子矩阵和（也是 2.4 中「区间改 + 区间查」的二维版本）。

**思路**：两个维度同时向上跳 / 向下跳，容斥求任意子矩阵：

$$
S(x_1,y_1,x_2,y_2) = Q(x_2,y_2) - Q(x_1-1,y_2) - Q(x_2,y_1-1) + Q(x_1-1,y_1-1)
$$

**复杂度**：每次操作 $O(\log n \log m)$。

```cpp
void add(int x, int y, long long v) {
    for (int i = x; i <= n; i += lowbit(i))
        for (int j = y; j <= m; j += lowbit(j)) tree[i][j] += v;
}
long long query(int x, int y) {
    long long s = 0;
    for (int i = x; i > 0; i -= lowbit(i))
        for (int j = y; j > 0; j -= lowbit(j)) s += tree[i][j];
    return s;
}
```

---

## 4. 易错点

1. **下标从 0 开始**：`lowbit(0) = 0` 会导致死循环。树状数组**必须从 1 开始编号**。
2. **`add` 循环条件写成 `i < n`**：应为 `i <= n`，否则漏更新最后一个位置。
3. **`query` 循环条件写成 `i >= 0`**：应为 `i > 0`，否则 `i = 0` 时死循环。
4. **区间和忘记减 `l - 1`**：`rangeSum(l, r) = query(r) - query(l-1)`。
5. **差分修改忘记 `r + 1`**：区间加要同时 `add(r+1, -x)`；`r+1 = n+1` 时循环自然跳过。
6. **`long long` 溢出**：区间和、逆序对数量都可能超过 `int`。
7. **树状数组不能直接维护最值**：最值不可减，需线段树或 ST 表。
8. **建树时重复加**：`build` 与 `add` 不要混用，否则元素被加两次。
9. **离散化后忘记 `+1`**：`lower_bound` 返回 0 起始下标，树状数组要求 1 起始。

---

## 5. 练习建议

### 5.1 由易到难的路线

1. **模板题**：单点修改 + 区间查询。
2. **差分应用**：区间修改 + 单点查询。
3. **两个树状数组**：区间修改 + 区间查询。
4. **权值树状数组**：逆序对、动态第 $k$ 小。
5. **树状数组上二分**：第 $k$ 小、最大前缀和。
6. **二维树状数组**：子矩阵求和、二维偏序。
7. **进阶**：树套树、CDQ 分治中的树状数组。

### 5.2 推荐题目

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P3374【模板】树状数组 1 | 单点改 + 区间查 |
| 入门 | 洛谷 P3368【模板】树状数组 2 | 差分 |
| 基础 | 洛谷 P1908 逆序对 | 权值树状数组 |
| 基础 | 洛谷 P1774 最接近神的人 | 逆序对 |
| 提高 | 洛谷 P4514 上帝造题的七分钟 | 二维树状数组 |
| 提高 | 洛谷 P3332 / P2617 | 动态第 k 小 |
| 提高 | 洛谷 P1966 火柴排队 | 逆序对 + 贪心 |
| 进阶 | 洛谷 P3157 动态逆序对 | 树状数组 + CDQ |

### 5.3 学习检查清单

- [ ] 能写出 `lowbit(x) = x & -x` 并解释含义
- [ ] 能画出 $n=8$ 时每个 `tree[i]` 的管辖区间
- [ ] 能说清 `add` 为什么向上跳、`query` 为什么向下跳
- [ ] 能默写三个模板：单点改区间查、区间改单点查、区间改区间查
- [ ] 能独立完成逆序对与动态第 $k$ 小
- [ ] 知道树状数组不能维护最值的原因

> **核心思想**：树状数组把下标按二进制拆分，让每个结点管理长度为 `lowbit` 的区间，从而把前缀和拆成 $O(\log n)$ 段——「向上更新、向下查询」八个字概括了它的全部。
