# 后缀自动机（Suffix Automaton, SAM）

> 后缀自动机是识别一个字符串所有子串的**最小 DFA**。它能在 $O(n)$ 时间、$O(n)$ 空间内表示所有子串，是字符串题的大杀器。

---

## 目录

1. [基本概念](#1-基本概念)
2. [SAM 的结构](#2-sam-的结构)
3. [构造算法](#3-构造算法)
4. [模板代码](#4-模板代码)
5. [常见应用](#5-常见应用)
6. [经典例题](#6-经典例题)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 基本概念

- **子串**：字符串中连续的一段；
- **endpos**：子串在字符串中所有结束位置的集合；
- **等价类**：endpos 相同的子串归为一类；
- **SAM 的状态**：每个状态对应一个 endpos 等价类。

**关键性质**：

- 每个状态对应若干个长度连续的子串（长度区间 $(\text{len}(link), \text{len}]$）；
- 状态数 $\le 2n - 1$，转移数 $\le 3n - 4$。

---

## 2. SAM 的结构

每个状态包含：

| 字段 | 含义 |
|---|---|
| `len` | 该状态表示的最长子串长度 |
| `link` | 后缀链接（指向最长的、endpos 更大的等价类） |
| `next[c]` | 字符 $c$ 的转移 |

**初始状态**：`len = 0`，`link = -1`（或 0）。

**根到某状态的路径**：对应一个子串。

---

## 3. 构造算法

**增量构造**：每次在末尾加入一个字符。

```
cur = 新状态，len = last.len + 1
p = last
while p != -1 and p 没有 c 转移:
    p.next[c] = cur
    p = p.link
if p == -1:
    cur.link = root
else:
    q = p.next[c]
    if p.len + 1 == q.len:
        cur.link = q
    else:
        clone = q 的副本，len = p.len + 1
        q.link = cur.link = clone
        修正 p 及其祖先的 c 转移指向 clone
last = cur
```

**复杂度**：$O(n)$（每个字符均摊 $O(1)$）。

---

## 4. 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2000005;

struct State {
    int len, link;
    int next[26];
} st[N];

int sz, last;

void init() {
    st[0].len = 0;
    st[0].link = -1;
    memset(st[0].next, -1, sizeof st[0].next);
    sz = 1;
    last = 0;
}

void extend(int c) {
    int cur = sz++;
    st[cur].len = st[last].len + 1;
    memset(st[cur].next, -1, sizeof st[cur].next);

    int p = last;
    while (p != -1 && st[p].next[c] == -1) {
        st[p].next[c] = cur;
        p = st[p].link;
    }
    if (p == -1) {
        st[cur].link = 0;
    } else {
        int q = st[p].next[c];
        if (st[p].len + 1 == st[q].len) {
            st[cur].link = q;
        } else {
            int clone = sz++;
            st[clone] = st[q];
            st[clone].len = st[p].len + 1;
            while (p != -1 && st[p].next[c] == q) {
                st[p].next[c] = clone;
                p = st[p].link;
            }
            st[q].link = st[cur].link = clone;
        }
    }
    last = cur;
}

int main() {
    init();
    string s;
    cin >> s;
    for (char c : s) extend(c - 'a');
    // ...
    return 0;
}
```

> 注意：`memset(st[cur].next, -1, ...)` 每个字符 26 个 int，$2 \times 10^6$ 状态会 MLE。大字符串要用 `map` 或稀疏存储。

---

## 5. 常见应用

### 5.1 判断子串是否存在

从根沿转移走，走不通则不存在。

### 5.2 本质不同子串数量

$$
\sum_{v \ne root} (\text{len}(v) - \text{len}(link(v)))
$$

### 5.3 每个子串出现次数

在 parent 树上（link 树）做子树和，初始 `cnt[cur] = 1`（除 clone）。

### 5.4 最长公共子串（多个串）

对一个串建 SAM，其他串在上面跑匹配。

### 5.5 字典序第 $k$ 小子串

在 DAG 上按拓扑序 DP 求路径数，再逐位确定。

### 5.6 最短未出现子串

SAM 上 BFS。

---

## 6. 经典例题

### 6.1 【模板】后缀自动机

**题意**：判断子串是否存在。

**做法**：沿转移走。

### 6.2 本质不同子串

**题意**：求字符串的不同子串数量。

**公式**：$\sum (\text{len} - \text{len}(link))$。

### 6.3 出现次数

**题意**：统计每个子串出现次数。

**做法**：parent 树子树和。

### 6.4 多个串的最长公共子串

**做法**：SAM + 匹配。

### 6.5 字典序第 $k$ 小子串

**做法**：SAM + DP。

---

## 7. 易错点与练习

### 易错点

1. **状态数**：开 $2n$；
2. **`clone` 的 `cnt`**：clone 初始 `cnt = 0`，不计入出现次数；
3. **`link` 的更新**：clone 的 link 要指向 q 的 link；
4. **多组数据**：重新 `init`；
5. **字符集**：用 `map` 或离散化；
6. **parent 树**：`link` 构成一棵树，按 `len` 排序后倒序统计。

### 练习建议

1. SAM 模板；
2. 判断子串；
3. 本质不同子串；
4. 出现次数（parent 树）；
5. 多串公共子串。
