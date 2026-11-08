# 平衡树（Balanced Binary Search Tree）

> 一句话概括：**平衡树通过旋转或分裂合并，让二叉搜索树的高度始终维持在 $O(\log n)$**，从而支持插入、删除、查询排名、前驱后继、区间翻转等操作，是「有序集合 + 动态排名」问题的核心工具。

---

## 目录

1. [从二叉搜索树说起](#1-从二叉搜索树说起)
2. [平衡树家族与选择](#2-平衡树家族与选择)
3. [FHQ Treap：思想与核心操作](#3-fhq-treap思想与核心操作)
4. [FHQ Treap 完整模板](#4-fhq-treap-完整模板)
5. [Splay：另一种主流平衡树](#5-splay另一种主流平衡树)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [复杂度对比](#8-复杂度对比)
9. [练习建议](#9-练习建议)

---

## 1. 从二叉搜索树说起

### 1.1 二叉搜索树（BST）

一棵二叉树是 BST，当且仅当对任意节点 $x$：

- 左子树中所有值 $< x$；
- 右子树中所有值 $> x$。

于是**中序遍历**得到严格递增序列。查找 $v$ 时从根出发：$v$ 小于当前值走左，大于走右，等于命中。

### 1.2 BST 的致命弱点

按顺序插入 $1,2,3,4,5$：

```
插入 1:  1
插入 2:  1
           \
            2
插入 3:  1
           \
            2
             \
              3
```

树退化成一条链，高度 $O(n)$，单次操作退化为 $O(n)$。**平衡树就是用来解决这个问题的**。

### 1.3 平衡的定义

常见定义：对任意节点，左右子树高度差不超过 1（AVL 树的严格平衡）；或「期望高度 $O(\log n)$」（Treap 的随机平衡）。

只要能保证高度 $O(\log n)$，所有基于「树高」的操作（查找、插入、删除、排名）都是 $O(\log n)$。

---

## 2. 平衡树家族与选择

| 名称 | 平衡方式 | 优缺点 | 适用场景 |
|---|---|---|---|
| AVL 树 | 严格高度平衡 + 四种旋转 | 查询快，删除调整复杂 | 查询远多于修改 |
| 红黑树 | 染色 + 旋转 | 常数小，实现复杂 | STL `map/set` 底层 |
| Treap | 随机优先级 + 旋转 | 好写，依赖随机 | 教学、一般平衡树 |
| **FHQ Treap** | 随机优先级 + **分裂/合并** | **最易写、支持可持久化**，常数略大 | 竞赛首选 |
| Splay | 每次访问后伸展到根 | 均摊优、支持区间操作，常数大 | LCT、区间翻转 |
| 替罪羊树 | 失衡时暴力重构 | 无旋转，均摊分析复杂 | 特殊场景 |

> **建议**：入门先学 **FHQ Treap**（代码短、无旋转、支持区间翻转与可持久化），再学 **Splay**（LCT 的前置知识）。

---

## 3. FHQ Treap：思想与核心操作

### 3.1 同时满足两个性质

每个节点有两个值：

- **权值 `val`**：满足 BST 性质（左小右大）；
- **随机优先级 `pri`**：满足小根堆性质（父节点 `pri` 小于儿子）。

因为 `pri` 是随机的，树形等价于「按随机顺序插入的 BST」，**期望高度 $O(\log n)$**。

```
            val=5, pri=3
           /            \
     val=3,pri=7      val=8,pri=9
      /      \          /     \
  val=1    val=4    val=6   val=10
  pri=11   pri=13   pri=15  pri=17
```

（`val` 满足 BST，`pri` 满足小根堆）

### 3.2 两个基本操作：分裂与合并

**分裂 `split(p, key)`**：把以 `p` 为根的树拆成两棵，`x` 中所有 `val <= key`，`y` 中所有 `val > key`。

```
原树：
        5
       / \
      3   8
     / \ / \
    1  4 6 10

split(根, 5) 之后：
   x（val <= 5）            y（val > 5）
        5                      8
       /                      / \
      3                      6  10
     / \
    1   4
```

（`5` 的右子树 `8` 被拆走：`6` 留在 `y`、`10` 也留在 `y`，所以 `x` 中 `5` 没有右儿子。）

实现思路：比较 `val[p]` 与 `key`：

- 若 `val[p] <= key`，则 `p` 及其左子树全部属于 `x`，递归分裂 `p` 的右子树；
- 否则 `p` 及其右子树全部属于 `y`，递归分裂 `p` 的左子树。

**合并 `merge(x, y)`**：要求 `x` 中所有值 $<$ `y` 中所有值，按优先级合并成一棵：

- 若 `pri[x] < pri[y]`，`x` 作根，递归合并 `x` 的右子树与 `y`；
- 否则 `y` 作根，递归合并 `x` 与 `y` 的左子树。

这两个函数加起来不到 20 行，却是整个 FHQ Treap 的全部基础，完整实现见第 4 节。

---

## 4. FHQ Treap 完整模板

下面这份模板支持：插入、删除、查询排名、查询第 $k$ 小、前驱、后继。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100005;

struct Node {
    int ls, rs, val, pri, sz;
} t[N];

int tot = 0;
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());

int newNode(int v) {
    ++tot;
    t[tot].ls = t[tot].rs = 0;
    t[tot].val = v;
    t[tot].pri = (int)rng();
    t[tot].sz = 1;
    return tot;
}

void pushUp(int p) {
    t[p].sz = t[t[p].ls].sz + t[t[p].rs].sz + 1;
}

void split(int p, int key, int& x, int& y) {
    if (!p) { x = y = 0; return; }
    if (t[p].val <= key) {
        x = p;
        split(t[p].rs, key, t[p].rs, y);
    } else {
        y = p;
        split(t[p].ls, key, x, t[p].ls);
    }
    pushUp(p);
}

int merge(int x, int y) {
    if (!x || !y) return x + y;
    if (t[x].pri < t[y].pri) {
        t[x].rs = merge(t[x].rs, y);
        pushUp(x);
        return x;
    } else {
        t[y].ls = merge(x, t[y].ls);
        pushUp(y);
        return y;
    }
}

// 插入 v
void insert(int& root, int v) {
    int x, y;
    split(root, v, x, y);
    root = merge(merge(x, newNode(v)), y);
}

// 删除一个 v（若不存在则无操作）
void erase(int& root, int v) {
    int x, y, z;
    split(root, v, x, z);          // x: <= v, z: > v
    split(x, v - 1, x, y);         // y: == v
    if (y) y = merge(t[y].ls, t[y].rs);   // 丢掉 y 的根
    root = merge(merge(x, y), z);
}

// 查询 v 的排名（比 v 小的个数 + 1）
int getRank(int& root, int v) {
    int x, y;
    split(root, v - 1, x, y);      // x: < v
    int ans = t[x].sz + 1;
    root = merge(x, y);            // 恢复原树（注意 root 必须传引用）
    return ans;
}

// 查询第 k 小
int getKth(int root, int k) {
    int p = root;
    while (p) {
        int leftSize = t[t[p].ls].sz;
        if (k <= leftSize) p = t[p].ls;
        else if (k == leftSize + 1) return t[p].val;
        else { k -= leftSize + 1; p = t[p].rs; }
    }
    return -1;                     // 不存在
}

// 前驱：小于 v 的最大值
int getPre(int& root, int v) {
    int x, y;
    split(root, v - 1, x, y);
    int p = x, ans = -1;
    while (p) { ans = t[p].val; p = t[p].rs; }   // 一直往右走
    root = merge(x, y);
    return ans;
}

// 后继：大于 v 的最小值
int getSuf(int& root, int v) {
    int x, y;
    split(root, v, x, y);
    int p = y, ans = -1;
    while (p) { ans = t[p].val; p = t[p].ls; }   // 一直往左走
    root = merge(x, y);
    return ans;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, root = 0;
    cin >> n;
    while (n--) {
        int op, x;
        cin >> op >> x;
        if (op == 1) insert(root, x);
        else if (op == 2) erase(root, x);
        else if (op == 3) cout << getRank(root, x) << '\n';
        else if (op == 4) cout << getKth(root, x) << '\n';
        else if (op == 5) cout << getPre(root, x) << '\n';
        else cout << getSuf(root, x) << '\n';
    }
    return 0;
}
```

### 4.1 复杂度说明

`split` 与 `merge` 都沿树高递归，期望 $O(\log n)$；插入、删除各需两次 split + 两次 merge，排名 / 第 k 小 / 前驱 / 后继也是期望 $O(\log n)$，空间 $O(n)$。

> 期望复杂度来自随机优先级；若 `rand()` 质量差或数据针对固定种子，可能退化。建议用 `mt19937` 随机种子。

---

## 5. Splay：另一种主流平衡树

### 5.1 核心思想

Splay 树**不维护严格平衡**，而是在每次访问节点 $x$ 后，通过一系列旋转把 $x$ 转到根节点，称为 **splay** 操作。均摊复杂度 $O(\log n)$。

重要性质：**splay 后，整棵树的形态与访问历史有关**，常用于 LCT（动态树）。

### 5.2 三种旋转

设 $x$ 的父节点为 $y$，祖父为 $z$：

1. **单旋（Zig）**：$y$ 是根，直接旋转 $x$ 到根；
2. **一字型（Zig-Zig）**：$x$、$y$ 同侧，**先旋 $y$ 再旋 $x$**；
3. **之字型（Zig-Zag）**：$x$、$y$ 异侧，**旋两次 $x$**。

### 5.3 旋转代码

```cpp
// 判断 x 是否为其父亲的右儿子
bool isRight(int x) { return t[t[x].fa].rs == x; }

void rotate(int x) {
    int y = t[x].fa, z = t[y].fa;
    int k = isRight(x);
    // 把 x 的对应儿子过继给 y
    if (k) {
        t[y].rs = t[x].ls;
        if (t[x].ls) t[t[x].ls].fa = y;
        t[x].ls = y;
    } else {
        t[y].ls = t[x].rs;
        if (t[x].rs) t[t[x].rs].fa = y;
        t[x].rs = y;
    }
    t[y].fa = x;
    t[x].fa = z;
    if (z) {
        if (t[z].ls == y) t[z].ls = x;
        else if (t[z].rs == y) t[z].rs = x;
    }
    pushUp(y);
    pushUp(x);
}

void splay(int x, int goal = 0) {
    while (t[x].fa != goal) {
        int y = t[x].fa, z = t[y].fa;
        if (z != goal) {
            if (isRight(x) == isRight(y)) rotate(y);   // Zig-Zig
            else rotate(x);                            // Zig-Zag
        }
        rotate(x);
    }
    if (!goal) root = x;
}
```

### 5.4 Splay 的典型应用

**文艺平衡树**：把第 $l-1$ 个节点 splay 到根、第 $r+1$ 个 splay 到根的右儿子，则根的右儿子的左子树就是区间 $[l,r]$，打上翻转标记即可。此外还用于 **LCT（Link-Cut Tree）** 与序列的插入、删除、区间操作。

---

## 6. 经典例题

### 6.1 例题一：普通平衡树（模板题）

**题意**：维护一个可重集合，支持：插入 $x$、删除一个 $x$、查询 $x$ 的排名、查询第 $k$ 小、求 $x$ 的前驱与后继。操作数 $n \le 10^5$。

**思路**：直接套用第 4 节 FHQ Treap。

**复杂度**：每次操作期望 $O(\log n)$，总 $O(n \log n)$。

### 6.2 例题二：区间翻转（文艺平衡树）

**题意**：给定长度为 $n$ 的序列，$m$ 次操作，每次翻转区间 $[l,r]$，最后输出整个序列。$n,m \le 10^5$。

**思路**：用**无旋 Treap 按排名分裂**（不是按值），每个节点维护 `rev` 翻转标记。`split(root, k, x, y)` 表示把前 $k$ 个元素分给 $x$；翻转 $[l,r]$ 时分裂出 $[1,l-1]$、$[l,r]$、$[r+1,n]$ 三段，给中间那段的根打 `rev ^= 1`；`pushDown` 时交换左右儿子并下传标记；最后中序遍历输出。

**复杂度**：$O((n+m)\log n)$。

```cpp
void pushDown(int p) {
    if (!t[p].rev) return;
    swap(t[p].ls, t[p].rs);
    t[t[p].ls].rev ^= 1;
    t[t[p].rs].rev ^= 1;
    t[p].rev = 0;
}

void split(int p, int k, int& x, int& y) {   // 按大小分裂
    if (!p) { x = y = 0; return; }
    pushDown(p);
    if (t[t[p].ls].sz < k) {
        x = p;
        split(t[p].rs, k - t[t[p].ls].sz - 1, t[p].rs, y);
    } else {
        y = p;
        split(t[p].ls, k, x, t[p].ls);
    }
    pushUp(p);
}
```

### 6.3 例题三：维护区间最大子段和（Splay 版）

**题意**：维护序列，支持单点修改、区间插入、区间删除、区间求和、区间最大子段和。**思路**：Splay 每个节点维护 `sum`、`pre`、`suf`、`best`，合并方式与线段树相同（见[线段树](../线段树/README.md)）。提取区间 $[l,r]$ 时把 $l-1$ splay 到根、$r+1$ splay 到根的右儿子，答案就在根的右儿子的左子树。**复杂度**：均摊 $O(\log n)$。

### 6.4 例题四：动态逆序对

**题意**：给定排列，每次删除一个数，求每次删除前的逆序对数量。

**思路**：用平衡树维护当前序列，删除时统计「左边比它大的个数 + 右边比它小的个数」。用 FHQ Treap 按值分裂后查询子树大小即可。也可用 CDQ 分治离线解决。

**复杂度**：$O(n \log n)$。

---

## 7. 易错点

1. **随机优先级重复**：`rand()` 范围小可能重复，建议用 `mt19937` 或 `rand() << 15 | rand()`。
2. **`split` 引用传参**：必须写成 `split(t[p].rs, key, t[p].rs, y)`，不能先存临时变量，否则树结构断裂。
3. **`merge` 前提**：必须保证 $x$ 中所有值小于 $y$ 中所有值，否则结果不是 BST。
4. **删除不存在的值**：`erase` 前应确认存在，或像模板那样判断 `if (y)`。
5. **多组数据不清空**：`tot` 要重置为 0，节点数组无需清空（新节点会覆盖）。
6. **`pushUp` 顺序**：先递归后 `pushUp`，且 `pushUp` 要在修改儿子之后。
7. **Splay 的 `pushDown`**：若维护翻转标记，`rotate` 和 `splay` 前必须从根到该节点一路 `pushDown`。
8. **Splay 的 `goal` 参数**：`splay(x, 0)` 表示转到根；`splay(x, y)` 表示转到 $y$ 的儿子。
9. **递归深度与溢出**：FHQ Treap 期望深度 $O(\log n)$，极端数据可能较深；值域运算注意 `long long`。

---

## 8. 复杂度对比

| 操作 | FHQ Treap | Splay | AVL | 红黑树 |
|---|---|---|---|---|
| 查找 | $O(\log n)$ 期望 | $O(\log n)$ 均摊 | $O(\log n)$ | $O(\log n)$ |
| 插入 | $O(\log n)$ 期望 | $O(\log n)$ 均摊 | $O(\log n)$ | $O(\log n)$ |
| 删除 | $O(\log n)$ 期望 | $O(\log n)$ 均摊 | $O(\log n)$ | $O(\log n)$ |
| 排名 / 第 k 小 | $O(\log n)$ | $O(\log n)$ | 需额外维护 sz | 需额外维护 sz |
| 区间翻转 | ✅ 支持 | ✅ 支持 | ❌ 困难 | ❌ 困难 |
| 可持久化 | ✅ 容易 | ❌ 困难 | ❌ | ❌ |
| 代码长度 | 短 | 长 | 中 | 长 |

---

## 9. 练习建议

### 9.1 由易到难

| 难度 | 题目类型 | 考察点 |
|---|---|---|
| ★ | 普通平衡树模板 | FHQ Treap 基本操作 |
| ★★ | 文艺平衡树 | 按排名分裂 + 翻转标记 |
| ★★ | 动态中位数 | 平衡树 / 对顶堆 |
| ★★★ | 维护数列（最大子段和） | Splay + 信息合并 |
| ★★★ | 动态逆序对 | 平衡树 + 统计 |
| ★★★★ | 可持久化平衡树 | 无旋 Treap 的路径复制 |
| ★★★★★ | LCT | Splay 进阶 |

### 9.2 学习检查清单

- [ ] 能画出 Treap 的结构并解释「为什么期望平衡」，能默写 `split` 与 `merge`
- [ ] 能说清 `split` 中引用传参的作用，能手写按排名分裂（用于区间操作）
- [ ] 能说出 Splay 的三种旋转，理解「区间翻转」为什么用平衡树而不是线段树

### 9.3 推荐练习顺序

1. 洛谷 P3369【模板】普通平衡树
2. 洛谷 P3391【模板】文艺平衡树
3. 洛谷 P1486 [NOI2004] 郁闷的出纳员
4. 洛谷 P2042 [NOI2005] 维护数列
5. 洛谷 P3157 [CQOI2011] 动态逆序对
6. 洛谷 P3835【模板】可持久化平衡树

---

> **核心思想**：平衡树用随机优先级（Treap）或旋转（Splay）把 BST 的高度控制在 $O(\log n)$。FHQ Treap 的「分裂 + 合并」把插入删除化归为两个基本操作，代码短、易扩展，是竞赛中平衡树的首选。
