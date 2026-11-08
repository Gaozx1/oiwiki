# 分治（Divide and Conquer）

> 分治 = **把大问题拆成若干个规模更小、结构相同的子问题**，分别求解后合并。它是归并排序、快速排序、快速幂、CDQ 分治的思想源头。

---

## 目录

1. [核心思想](#1-核心思想)
2. [分治三步骤](#2-分治三步骤)
3. [典型例子](#3-典型例子)
4. [复杂度分析](#4-复杂度分析)
5. [经典例题](#5-经典例题)
6. [分治与其他技巧的结合](#6-分治与其他技巧的结合)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 核心思想

**分治（Divide and Conquer）** 的三个步骤：

1. **Divide（分）**：把原问题划分成若干个规模较小的子问题；
2. **Conquer（治）**：递归求解子问题；
3. **Combine（合）**：把子问题的解合并成原问题的解。

**关键条件**：

- 子问题**结构相同**、规模更小（递归终止）；
- 子问题**相互独立**（不重叠，否则用 DP）；
- 合并的代价可以接受。

---

## 2. 分治三步骤

以**归并排序**为例：

```
Divide:  把数组从中间切成两半
Conquer: 递归排序左右两半
Combine: 合并两个有序数组
```

```cpp
void mergeSort(int a[], int l, int r) {
    if (l >= r) return;
    int mid = (l + r) >> 1;
    mergeSort(a, l, mid);
    mergeSort(a, mid + 1, r);
    // 合并
    int i = l, j = mid + 1, k = 0;
    while (i <= mid && j <= r) {
        if (a[i] <= a[j]) tmp[k++] = a[i++];
        else tmp[k++] = a[j++];
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) tmp[k++] = a[j++];
    for (int t = 0; t < k; ++t) a[l+t] = tmp[t];
}
```

---

## 3. 典型例子

### 3.1 快速幂（分治思想）

$$
a^n = \begin{cases} (a^{n/2})^2 & n \text{ 偶} \\ a \cdot (a^{(n-1)/2})^2 & n \text{ 奇} \end{cases}
$$

### 3.2 快速排序

- Divide：选基准，划分成「小于」和「大于」两部分；
- Conquer：递归排序两部分；
- Combine：无需合并（原地）。

### 3.3 二分查找

每次把搜索范围减半，是「退化版」的分治（只有一个子问题）。

### 3.4 大整数乘法（Karatsuba）

把 $n$ 位整数分成两半，用 3 次乘法代替 4 次，复杂度 $O(n^{1.585})$。

### 3.5 平面最近点对

按 $x$ 排序后分治，合并时只需检查中线附近 $O(\delta)$ 范围内的点。

```cpp
double closestPair(vector<Point>& p, int l, int r) {
    if (r - l <= 3) return brute(p, l, r);
    int mid = (l + r) >> 1;
    double d = min(closestPair(p, l, mid), closestPair(p, mid + 1, r));
    // 合并：检查 x 在中线 ±d 内的点
    vector<Point> strip;
    for (int i = l; i <= r; ++i)
        if (fabs(p[i].x - p[mid].x) < d) strip.push_back(p[i]);
    sort(strip.begin(), strip.end(), [](auto& a, auto& b){ return a.y < b.y; });
    for (int i = 0; i < (int)strip.size(); ++i)
        for (int j = i + 1; j < (int)strip.size() && strip[j].y - strip[i].y < d; ++j)
            d = min(d, dist(strip[i], strip[j]));
    return d;
}
```

---

## 4. 复杂度分析

分治算法通常满足递推式：

$$
T(n) = aT(n/b) + f(n)
$$

**主定理（Master Theorem）**：

- 若 $f(n) = O(n^{\log_b a - \epsilon})$，则 $T(n) = \Theta(n^{\log_b a})$；
- 若 $f(n) = \Theta(n^{\log_b a})$，则 $T(n) = \Theta(n^{\log_b a} \log n)$；
- 若 $f(n) = \Omega(n^{\log_b a + \epsilon})$，则 $T(n) = \Theta(f(n))$。

**常见例子**：

| 算法 | 递推式 | 复杂度 |
|---|---|---|
| 归并排序 | $T(n) = 2T(n/2) + O(n)$ | $O(n \log n)$ |
| 二分查找 | $T(n) = T(n/2) + O(1)$ | $O(\log n)$ |
| 快速幂 | $T(n) = T(n/2) + O(1)$ | $O(\log n)$ |
| Karatsuba | $T(n) = 3T(n/2) + O(n)$ | $O(n^{1.585})$ |
| 快速排序（平均） | $T(n) = 2T(n/2) + O(n)$ | $O(n \log n)$ |

---

## 5. 经典例题

### 5.1 归并排序求逆序对

**题意**：求数组逆序对数量。

**做法**：归并时统计「右边元素小于左边元素」的次数。

```cpp
long long mergeCount(int a[], int l, int r) {
    if (l >= r) return 0;
    int mid = (l + r) >> 1;
    long long cnt = mergeCount(a, l, mid) + mergeCount(a, mid+1, r);
    int i = l, j = mid + 1, k = 0;
    while (i <= mid && j <= r) {
        if (a[i] <= a[j]) tmp[k++] = a[i++];
        else { cnt += mid - i + 1; tmp[k++] = a[j++]; }
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) tmp[k++] = a[j++];
    for (int t = 0; t < k; ++t) a[l+t] = tmp[t];
    return cnt;
}
```

**复杂度**：$O(n \log n)$。

### 5.2 平面最近点对

**题意**：$n$ 个点，求最近距离。

**做法**：按 $x$ 分治 + 合并时检查 $\pm d$ 条带，$O(n \log n)$。

### 5.3 快速幂

**题意**：求 $a^b \bmod p$。

**做法**：分治或二进制。

### 5.4 大整数乘法

**做法**：Karatsuba。

### 5.5 棋盘覆盖

**题意**：$2^k \times 2^k$ 棋盘有一个特殊格，用 L 型骨牌覆盖。

**做法**：分治成 4 个 $2^{k-1}$ 子棋盘，特殊格所在子棋盘递归，其余三个放一块骨牌变成新的特殊格。

---

## 6. 分治与其他技巧的结合

| 结合 | 名称 | 用途 |
|---|---|---|
| 分治 + 排序 | 归并排序 / CDQ | 逆序对、三维偏序 |
| 分治 + 二分 | 整体二分 | 离线询问 |
| 分治 + 树 | 点分治 | 树上路径统计 |
| 分治 + 数据结构 | 线段树分治 | 动态删除问题 |
| 分治 + 几何 | 最近点对 | 计算几何 |

### 点分治简介

处理树上「路径满足条件」的问题：每次找到树的重心，统计经过重心的路径，然后递归子树。复杂度 $O(n \log n)$。

---

## 7. 易错点与练习

### 易错点

1. **递归边界**：`l >= r` 时返回，否则死递归；
2. **`mid` 的计算**：`(l + r) >> 1` 可能溢出（$l+r$ 超 int），用 `l + (r - l) / 2`；
3. **合并时的临时数组**：要清空或正确覆盖；
4. **子问题重叠**：如果子问题有重叠，应该用 DP 而不是分治；
5. **递归深度**：深度 $O(\log n)$ 一般安全，但链状分治可能 $O(n)$。

### 练习建议

1. 手写归并排序、快速排序；
2. 用归并求逆序对；
3. 实现快速幂；
4. 平面最近点对（分治版）；
5. 学习 CDQ 分治（三维偏序）。
