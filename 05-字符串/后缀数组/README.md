# 后缀数组（Suffix Array）

> 后缀数组把字符串的所有后缀按字典序排好，配合 height 数组可以在 $O(1)$/ $O(\log n)$ 内回答大量字符串问题。

---

## 目录

1. [定义](#1-定义)
2. [倍增求法](#2-倍增求法)
3. [height 数组](#3-height-数组)
4. [模板代码](#4-模板代码)
5. [经典应用](#5-经典应用)
6. [经典例题](#6-经典例题)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 定义

- **$sa[i]$**：字典序第 $i$ 小的后缀的起始位置（1-indexed）；
- **$rk[i]$**：后缀 $s[i..n]$ 的排名（$rk[sa[i]] = i$）；
- **$height[i]$**：$sa[i]$ 与 $sa[i-1]$ 的最长公共前缀长度（LCP）。

**例子**：$s = \text{"banana"}$（后缀数组）

| 排名 | 后缀 | 起点 |
|---|---|---|
| 1 | a | 6 |
| 2 | ana | 4 |
| 3 | anana | 2 |
| 4 | banana | 1 |
| 5 | na | 5 |
| 6 | nana | 3 |

所以 $sa = [6, 4, 2, 1, 5, 3]$。

---

## 2. 倍增求法

**核心**：把每个后缀看作「从 $i$ 开始、长度为 $2^k$ 的前缀」，用基数排序比较。

1. 初始 $k=0$，按单个字符排序；
2. 每次把长度翻倍，用「前一半排名 + 后一半排名」作为二元组排序；
3. 直到所有排名互不相同。

**复杂度**：$O(n \log n)$（基数排序）或 $O(n \log^2 n)$（`sort`）。

---

## 3. height 数组

**性质**：

$$
height[rk[i]] \ge height[rk[i-1]] - 1
$$

（后缀 $i$ 与后缀 $i-1$ 的 LCP 至少是前者的 $1$）

**求法**：按 $i$ 从 $1$ 到 $n$，用上面的性质线性递推。

```cpp
for (int i = 1, k = 0; i <= n; ++i) {
    if (rk[i] == 1) { k = 0; continue; }
    if (k) --k;
    int j = sa[rk[i] - 1];
    while (i + k <= n && j + k <= n && s[i + k] == s[j + k]) ++k;
    height[rk[i]] = k;
}
```

**复杂度**：$O(n)$。

### 3.1 任意两个后缀的 LCP

$$
\text{LCP}(i, j) = \min(height[rk[i]+1..rk[j]])
$$

用 RMQ（ST 表）$O(1)$ 查询。

---

## 4. 模板代码

### 4.1 倍增 + 基数排序

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1000005;

int n;
char s[N];
int sa[N], rk[N], oldrk[N << 1], id[N], cnt[N];

void buildSA() {
    int m = 128;   // 字符集大小
    for (int i = 1; i <= n; ++i) ++cnt[rk[i] = s[i]];
    for (int i = 1; i <= m; ++i) cnt[i] += cnt[i-1];
    for (int i = n; i >= 1; --i) sa[cnt[rk[i]]--] = i;

    for (int k = 1; k < n; k <<= 1) {
        int p = 0;
        for (int i = n - k + 1; i <= n; ++i) id[++p] = i;
        for (int i = 1; i <= n; ++i)
            if (sa[i] > k) id[++p] = sa[i] - k;

        memset(cnt, 0, sizeof cnt);
        for (int i = 1; i <= n; ++i) ++cnt[rk[id[i]]];
        for (int i = 1; i <= m; ++i) cnt[i] += cnt[i-1];
        for (int i = n; i >= 1; --i) sa[cnt[rk[id[i]]]--] = id[i];

        memcpy(oldrk, rk, sizeof rk);
        p = 0;
        for (int i = 1; i <= n; ++i) {
            if (oldrk[sa[i]] == oldrk[sa[i-1]] &&
                oldrk[sa[i] + k] == oldrk[sa[i-1] + k]) rk[sa[i]] = p;
            else rk[sa[i]] = ++p;
        }
        if (p == n) break;
        m = p;
    }
}

int height[N];

void buildHeight() {
    for (int i = 1, k = 0; i <= n; ++i) {
        if (rk[i] == 1) { k = 0; continue; }
        if (k) --k;
        int j = sa[rk[i] - 1];
        while (i + k <= n && j + k <= n && s[i + k] == s[j + k]) ++k;
        height[rk[i]] = k;
    }
}
```

### 4.2 简化版（sort + 倍增，$O(n \log^2 n)$）

```cpp
// 更容易写，适合 n <= 1e5
```

---

## 5. 经典应用

### 5.1 最长公共前缀（任意两后缀）

ST 表 + height，$O(1)$。

### 5.2 最长重复子串

$\max(height)$。

### 5.3 本质不同子串数量

$$
\frac{n(n+1)}{2} - \sum_{i=2}^{n} height[i]
$$

### 5.4 最长回文子串

把原串和反串拼接，求 LCP（配合 Manacher 更好）。

### 5.5 多串公共子串

把所有串拼接（用不同分隔符），求 height 数组，用单调栈/二分。

### 5.6 字典序第 $k$ 小子串

利用 height 数组去重。

---

## 6. 经典例题

### 6.1 【模板】后缀数组

**题意**：输出 $sa$ 和 $height$。

**做法**：套模板。

### 6.2 本质不同子串

**题意**：求字符串中不同子串个数。

**公式**：$n(n+1)/2 - \sum height$。

### 6.3 最长重复子串

**题意**：求出现至少两次的最长子串。

**答案**：$\max(height)$。

### 6.4 最长公共子串（两串）

**题意**：两个字符串的最长公共子串。

**做法**：拼接 + 后缀数组 + height，找相邻且属于不同串的最大 height。

### 6.5 多个字符串的最长公共子串

**做法**：拼接 + 后缀数组 + 单调栈/二分。

---

## 7. 易错点与练习

### 易错点

1. **数组大小**：`oldrk` 要开 `2n`，防止 `sa[i] + k` 越界；
2. **字符集大小**：初始 `m` 要覆盖所有字符（ASCII 用 128）；
3. **多组数据**：`cnt`、`rk` 等要清空；
4. **`height` 的定义**：是「前一个」后缀的 LCP，不是与前缀；
5. **下标**：1-indexed 与 0-indexed 混用容易错；
6. **拼接分隔符**：多串拼接时用不同的、不在字符集中的字符。

### 练习建议

1. 后缀数组模板；
2. 后缀数组 + height；
3. 本质不同子串；
4. 最长重复子串；
5. 多串公共子串（进阶）。
