# 离散化

> 一句话概括：**离散化是把「值很大但个数很少」的原始数据映射成「$1 \sim n$ 的排名」，从而让值域变得可以开数组、可以用下标做算法。**

---

## 目录

1. [为什么需要离散化](#1-为什么需要离散化)
2. [离散化的核心思想](#2-离散化的核心思想)
3. [标准离散化实现](#3-标准离散化实现)
4. [保序与不保序](#4-保序与不保序)
5. [离散化的常见搭配](#5-离散化的常见搭配)
6. [模板代码](#6-模板代码)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 为什么需要离散化

看这道题：给定 $n \le 10^5$ 个整数，每个数 $a_i \le 10^9$，求有多少对 $(i, j)$ 满足 $a_i < a_j$。

如果用「值域数组」统计每个数出现次数，需要开 $10^9$ 大小的数组，约 4 GB，直接 MLE。但注意：**真正出现的值最多只有 $10^5$ 个**。

离散化就是：**把这 $10^5$ 个互不相同的值，按大小重新编号为 $1, 2, \ldots, m$**，之后所有操作都在 $[1, m]$ 上进行。

```
原始值:  1000000000   5   999999999   5   42
排序去重: 5   42   999999999   1000000000
映射后:   2    1        3         4        2
```

**关键性质**：离散化只改变「值的大小刻度和名称」，不改变「值之间的相对大小关系」。因此所有只依赖比较的算法（排序、二分、树状数组）都不受影响。

---

## 2. 离散化的核心思想

### 2.1 三步走

1. **收集**：把所有需要离散化的值放进一个数组 `b`；
2. **排序 + 去重**：`sort(b)` 后 `unique`，得到有序的值域；
3. **映射**：对每个原值 `x`，用 `lower_bound` 找到它在 `b` 中的下标（+1），作为新值。

### 2.2 为什么用 `lower_bound`

因为 `b` 已经有序，`lower_bound(b.begin(), b.end(), x) - b.begin()` 就是 `x` 在 `b` 中的**排名（0 起始）**。这个操作是 $O(\log n)$，总复杂度 $O(n\log n)$。

### 2.3 复杂度

- 收集：$O(n)$
- 排序：$O(n\log n)$
- 每个值二分：$O(n\log n)$

总复杂度 $O(n\log n)$，空间 $O(n)$。

### 2.4 什么时候不需要离散化

- 值域本身就很小时（如 $a_i \le 10^6$），直接开数组；
- 只需要相对顺序、不需要按下标索引时，直接排序即可；
- 用 `map` / `unordered_map` 也能做「值 → 排名」的映射，但常数更大。

---

## 3. 标准离散化实现

### 3.1 基础版本

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int& x : a) cin >> x;

    // 1. 拷贝一份用于排序去重
    vector<int> b = a;
    sort(b.begin(), b.end());
    b.erase(unique(b.begin(), b.end()), b.end());   // b 现在是去重后的值域

    // 2. 把 a 中每个元素映射成它在 b 中的排名（从 1 开始）
    for (int i = 0; i < n; ++i) {
        a[i] = lower_bound(b.begin(), b.end(), a[i]) - b.begin() + 1;
    }

    // 现在 a[i] ∈ [1, m]，其中 m = b.size()
    for (int x : a) cout << x << ' ';
    return 0;
}
```

输入 `1000000000 5 999999999 5 42`，输出 `4 1 3 1 2`。

### 3.2 需要保留原值的版本

有时既要排名，又要原始值（如输出原数）。用结构体保存：

```cpp
struct Node {
    int val, id, rank;
};

vector<Node> a(n);
for (int i = 0; i < n; ++i) {
    cin >> a[i].val;
    a[i].id = i;
}
sort(a.begin(), a.end(), [](const Node& x, const Node& y) {
    return x.val < y.val;
});
for (int i = 0; i < n; ++i) a[i].rank = i + 1;   // 直接按排序后的位置编号
```

> 注意：这种写法**不去重**，相同的值会得到不同的 rank。若需要相同值同排名，要单独处理。

### 3.3 离散化二维坐标

处理平面上的点、矩形时，行坐标与列坐标要**分别离散化**：

```cpp
vector<int> xs, ys;
for (int i = 0; i < n; ++i) {
    xs.push_back(x[i]);
    ys.push_back(y[i]);
}
sort(xs.begin(), xs.end()); xs.erase(unique(xs.begin(), xs.end()), xs.end());
sort(ys.begin(), ys.end()); ys.erase(unique(ys.begin(), ys.end()), ys.end());

for (int i = 0; i < n; ++i) {
    int cx = lower_bound(xs.begin(), xs.end(), x[i]) - xs.begin() + 1;
    int cy = lower_bound(ys.begin(), ys.end(), y[i]) - ys.begin() + 1;
    // (cx, cy) 就是压缩后的坐标
}
```

---

## 4. 保序与不保序

### 4.1 保序离散化

上面的标准做法：**值越小，排名越小**。即

$$a_i < a_j \iff \text{rank}(a_i) < \text{rank}(a_j)$$

这保留了原始的大小关系，适用于几乎所有场景。

### 4.2 不保序离散化

有时只需要「不同的值有不同的编号」，不关心大小顺序。例如统计颜色种类：

```cpp
unordered_map<int, int> id;
int cnt = 0;
for (int x : a) {
    if (!id.count(x)) id[x] = ++cnt;
}
```

这种写法 $O(n)$ 平均，比排序更快，但**不能用于需要比较大小的算法**（如树状数组求逆序对）。

### 4.3 离散化后「相邻」的含义

离散化后 `rank` 相邻**不代表原始值相邻**。例如原值 `1, 100, 1000` 离散化为 `1, 2, 3`，它们之间没有别的数，但原值差距巨大。**需要区间长度时要乘回原值差。**

---

## 5. 离散化的常见搭配

### 5.1 离散化 + 树状数组（求逆序对）

```cpp
// 值域 1..m，用树状数组统计每个值出现过几次
for (int i = n - 1; i >= 0; --i) {
    ans += query(a[i] - 1);   // 右边比它小的个数
    add(a[i], 1);
}
```

### 5.2 离散化 + 线段树（区间覆盖）

坐标范围 $10^9$ 的区间覆盖、矩形面积并，必须离散化坐标后建线段树。

### 5.3 离散化 + 二分

离散化后，`lower_bound` 直接在排名数组上跑，省去处理大值域。

### 5.4 离散化 + 计数数组

```cpp
vector<int> cnt(m + 1, 0);
for (int x : a) ++cnt[x];      // 现在 x 是排名，数组开得下
```

---

## 6. 模板代码

### 6.1 通用离散化函数

```cpp
#include <bits/stdc++.h>
using namespace std;

// 返回离散化后的数组，值域为 [1, m]
vector<int> discrete(const vector<int>& a) {
    vector<int> b = a;
    sort(b.begin(), b.end());
    b.erase(unique(b.begin(), b.end()), b.end());

    vector<int> res(a.size());
    for (size_t i = 0; i < a.size(); ++i)
        res[i] = lower_bound(b.begin(), b.end(), a[i]) - b.begin() + 1;
    return res;
}

int main() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int& x : a) cin >> x;

    vector<int> c = discrete(a);
    for (int x : c) cout << x << ' ';
    return 0;
}
```

### 6.2 离散化 + 树状数组求逆序对

```cpp
#include <bits/stdc++.h>
using namespace std;

int n, m;
vector<int> c;

void add(int i, int v) {
    for (; i <= m; i += i & -i) c[i] += v;
}

int query(int i) {
    int s = 0;
    for (; i > 0; i -= i & -i) s += c[i];
    return s;
}

int main() {
    cin >> n;
    vector<int> a(n), b(n);
    for (int i = 0; i < n; ++i) { cin >> a[i]; b[i] = a[i]; }

    sort(b.begin(), b.end());
    b.erase(unique(b.begin(), b.end()), b.end());
    m = b.size();

    for (int i = 0; i < n; ++i)
        a[i] = lower_bound(b.begin(), b.end(), a[i]) - b.begin() + 1;

    c.assign(m + 1, 0);
    long long ans = 0;
    for (int i = n - 1; i >= 0; --i) {
        ans += query(a[i] - 1);
        add(a[i], 1);
    }
    cout << ans << '\n';
    return 0;
}
```

### 6.3 二维坐标离散化

```cpp
int n;
cin >> n;
vector<int> x(n), y(n), xs, ys;
for (int i = 0; i < n; ++i) {
    cin >> x[i] >> y[i];
    xs.push_back(x[i]);
    ys.push_back(y[i]);
}
sort(xs.begin(), xs.end()); xs.erase(unique(xs.begin(), xs.end()), xs.end());
sort(ys.begin(), ys.end()); ys.erase(unique(ys.begin(), ys.end()), ys.end());

vector<vector<int>> g(xs.size(), vector<int>(ys.size(), 0));
for (int i = 0; i < n; ++i) {
    int cx = lower_bound(xs.begin(), xs.end(), x[i]) - xs.begin();
    int cy = lower_bound(ys.begin(), ys.end(), y[i]) - ys.begin();
    g[cx][cy] = 1;
}
```

---

## 7. 经典例题

### 例题 1：离散化模板

**题意**：给定 $n$（$n \le 10^5$）个整数（$|a_i| \le 10^9$），输出每个数离散化后的排名（相同数排名相同，从 1 开始）。

**思路**：标准三步：拷贝、排序去重、`lower_bound`。

**复杂度**：$O(n\log n)$。

```cpp
int main() {
    int n;
    cin >> n;
    vector<int> a(n), b;
    for (int& x : a) { cin >> x; b.push_back(x); }

    sort(b.begin(), b.end());
    b.erase(unique(b.begin(), b.end()), b.end());

    for (int x : a)
        cout << lower_bound(b.begin(), b.end(), x) - b.begin() + 1 << ' ';
    return 0;
}
```

### 例题 2：区间求和（大值域）

**题意**：有 $n$ 个操作，每次在数轴位置 $p$ 加上 $v$（$|p| \le 10^9$），最后询问区间 $[L, R]$ 的和。

**思路**：把所有出现过的位置（包括查询端点）离散化，然后用树状数组或前缀和维护。

**复杂度**：$O(n\log n)$。

```cpp
struct Op { int type; long long p, v; };

int main() {
    int n;
    cin >> n;
    vector<Op> ops(n);
    vector<long long> xs;
    for (auto& o : ops) {
        cin >> o.type >> o.p;
        if (o.type == 1) { cin >> o.v; xs.push_back(o.p); }
        else xs.push_back(o.p);
    }
    sort(xs.begin(), xs.end());
    xs.erase(unique(xs.begin(), xs.end()), xs.end());

    vector<long long> bit(xs.size() + 1, 0);
    auto add = [&](int i, long long v) {
        for (; i <= (int)xs.size(); i += i & -i) bit[i] += v;
    };
    auto sum = [&](int i) {
        long long s = 0;
        for (; i > 0; i -= i & -i) s += bit[i];
        return s;
    };

    for (auto& o : ops) {
        int pos = lower_bound(xs.begin(), xs.end(), o.p) - xs.begin() + 1;
        if (o.type == 1) add(pos, o.v);
        else cout << sum(pos) << '\n';
    }
    return 0;
}
```

### 例题 3：矩形覆盖面积

**题意**：给定 $n$ 个与坐标轴平行的矩形，求它们的并集面积。坐标绝对值可达 $10^9$。

**思路**：扫描线 + 离散化。把矩形的左右边界和上下边界全部离散化，按 $x$ 扫描，用线段树维护 $y$ 方向的覆盖长度。

**复杂度**：$O(n\log n)$。

（完整实现见「计算几何 / 扫描线」一章。）

### 例题 4：离散化后求第 k 小

**题意**：给定 $n$ 个整数，求其中第 $k$ 小的**不同**值。

**思路**：离散化后排名就是不同值的序号，第 $k$ 小即排名为 $k$ 的值（若 $k >$ 不同值个数则不存在）。

**复杂度**：$O(n\log n)$。

```cpp
int main() {
    int n, k;
    cin >> n >> k;
    vector<int> a(n), b;
    for (int& x : a) { cin >> x; b.push_back(x); }
    sort(b.begin(), b.end());
    b.erase(unique(b.begin(), b.end()), b.end());

    if (k < 1 || k > (int)b.size()) { cout << -1 << '\n'; }
    else cout << b[k - 1] << '\n';
    return 0;
}
```

---

## 8. 易错点

1. **忘记去重**：用 `b.size()` 当值域却保留了重复值，会导致下标错乱。
2. **`lower_bound` 用错容器**：要在去重后的 `b` 上二分，不是在原数组上。
3. **下标从 0 还是 1 开始**：树状数组要求从 1 开始，别忘了 `+1`。
4. **对负数离散化**：`lower_bound` 对负数同样正确，但映射后一律是非负排名。
5. **相同值必须得到相同排名**：用 `lower_bound` 才能保证，用 `sort` 后直接编号会得到不同 rank。
6. **离散化后区间长度变了**：需要乘回原值差，不能直接用排名差。
7. **多次离散化混用**：每个值域要单独建映射表，不要共用一个 `b`。
8. **离散化前忘了排序**：`lower_bound` 要求有序。
9. **值域个数为 1 时**：`m = 1`，树状数组大小要 `m + 1`。
10. **离散化 + 前缀和**：前缀和的下标是排名，不是原值，查询区间要先用 `lower_bound` 找边界。

---

## 9. 练习建议

### 9.1 学习路线（由易到难）

1. **手写离散化**：排序、去重、二分三步。
2. **离散化 + 计数**：统计每个值出现次数。
3. **离散化 + 树状数组**：求逆序对。
4. **二维离散化**：平面坐标压缩。
5. **离散化 + 线段树**：扫描线、矩形面积。
6. **离散化 + 二分**：大值域上的查找。

### 9.2 刷题清单

| 难度 | 题目 | 训练点 |
|---|---|---|
| 入门 | 洛谷 P1059 明明的随机数 | 排序 + 去重（离散化雏形） |
| 普及− | 洛谷 P1908 逆序对 | 离散化 + 树状数组 |
| 普及− | 洛谷 P1955 程序自动分析 | 离散化 + 并查集 |
| 普及/提高− | 洛谷 P1886 滑动窗口 | 单调队列（配合离散化） |
| 普及/提高− | 洛谷 P5490 扫描线 | 离散化 + 线段树 |
| 普及/提高− | 洛谷 P1966 火柴排队 | 离散化 + 逆序对 |

### 9.3 学习检查清单

- [ ] 能说清离散化的三个步骤
- [ ] 能写出 `sort + unique + lower_bound` 模板
- [ ] 知道离散化保留了什么、改变了什么
- [ ] 能处理相同值映射到同一排名
- [ ] 能对二维坐标分别离散化
- [ ] 知道离散化后下标从 1 开始对树状数组的意义

> **核心思想**：离散化是**「把值域压进下标空间」**的桥梁。它让 $10^9$ 的值域变成 $10^5$ 的数组，是树状数组、线段树、扫描线能处理大坐标数据的前提。
