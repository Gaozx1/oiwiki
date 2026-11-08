# 回文自动机（Palindromic Automaton, PAM）

> 回文自动机（PAM，也叫回文树）能在 $O(n)$ 时间内求出所有**本质不同的回文子串**及其出现次数。结构比 SAM 简单得多。

---

## 目录

1. [结构](#1-结构)
2. [构造算法](#2-构造算法)
3. [模板代码](#3-模板代码)
4. [常见应用](#4-常见应用)
5. [经典例题](#5-经典例题)
6. [易错点与练习](#6-易错点与练习)

---

## 1. 结构

PAM 的每个节点表示一个**本质不同的回文串**，有两个根：

- **奇根**（节点 1）：长度为 $-1$，方便处理单个字符；
- **偶根**（节点 0）：长度为 $0$。

每个节点包含：

| 字段 | 含义 |
|---|---|
| `len` | 回文串长度 |
| `fail` | 失配指针（最长的回文后缀） |
| `next[c]` | 两侧加字符 $c$ 后的回文串 |

**性质**：

- 节点数 $\le n + 2$；
- 每个节点对应唯一的回文串。

---

## 2. 构造算法

增量构造，每次在末尾加入字符 $s[i]$：

1. 从 `last` 开始沿 `fail` 往上跳，找到第一个满足 $s[i - len - 1] == s[i]$ 的节点；
2. 若该节点已有字符 $c$ 的转移，直接走过去；
3. 否则新建节点：
   - `len = 上一个节点.len + 2`；
   - 求 `fail`：从 `fail[上一个]` 开始跳，找满足条件的节点，指向它的 $c$ 转移；
4. 更新 `last`。

---

## 3. 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1000005;

struct Node {
    int next[26];
    int fail, len;
    int cnt;      // 该回文串出现次数
} tr[N];

int sz, last;
char s[N];

void init() {
    sz = 1; last = 0;
    tr[0].len = 0; tr[0].fail = 1;
    tr[1].len = -1; tr[1].fail = 1;
    memset(tr[0].next, 0, sizeof tr[0].next);
    memset(tr[1].next, 0, sizeof tr[1].next);
}

int getFail(int u, int pos) {
    while (pos - tr[u].len - 1 < 0 || s[pos - tr[u].len - 1] != s[pos])
        u = tr[u].fail;
    return u;
}

void insert(int pos) {   // pos 从 1 开始
    int c = s[pos] - 'a';
    int u = getFail(last, pos);
    if (!tr[u].next[c]) {
        int v = ++sz;
        tr[v].len = tr[u].len + 2;
        tr[v].cnt = 0;
        memset(tr[v].next, 0, sizeof tr[v].next);

        if (tr[v].len == 1) {
            tr[v].fail = 0;
        } else {
            int f = getFail(tr[u].fail, pos);
            tr[v].fail = tr[f].next[c];
        }
        tr[u].next[c] = v;
    }
    last = tr[u].next[c];
    tr[last].cnt++;
}

int main() {
    scanf("%s", s + 1);
    int n = strlen(s + 1);
    init();
    for (int i = 1; i <= n; ++i) insert(i);

    // 按 len 从大到小，把 cnt 累加到 fail
    // 需要先按 len 计数排序
    vector<int> ord(sz + 1);
    for (int i = 1; i <= sz; ++i) ord[i] = i;
    sort(ord.begin() + 1, ord.end(), [](int a, int b) { return tr[a].len > tr[b].len; });
    for (int i = 1; i <= sz; ++i)
        tr[tr[ord[i]].fail].cnt += tr[ord[i]].cnt;

    // 统计本质不同回文串数量 = sz - 1（去掉奇根）
    printf("%d\n", sz - 1);
    return 0;
}
```

---

## 4. 常见应用

### 4.1 本质不同回文子串数量

节点数 $- 1$（去掉奇根，或 $-2$ 去掉两个根）。

### 4.2 每个回文串出现次数

构造后按 `len` 从大到小，把 `cnt` 累加到 `fail` 上。

### 4.3 最长回文子串

$\max(\text{len})$。

### 4.4 回文子串总数

所有节点的 `cnt` 之和。

### 4.5 以每个位置结尾的回文串数量

`last` 的 fail 链长度。

```cpp
// 维护 fail 树的深度
tr[v].depth = tr[tr[v].fail].depth + 1;
// 每次插入后输出 tr[last].depth
```

### 4.6 双回文串

PAM + 前后缀。

---

## 5. 经典例题

### 5.1 【模板】回文自动机

**题意**：求本质不同回文子串数量。

**做法**：PAM 节点数 $- 1$。

### 5.2 回文子串出现次数

**题意**：统计每个回文串出现次数。

**做法**：PAM + 按 `len` 排序累加。

### 5.3 最长回文子串

**题意**：求最长回文子串。

**做法**：PAM 最大 `len`。

### 5.4 以每个位置结尾的回文数

**做法**：维护 `depth`。

### 5.5 回文串计数

**题意**：统计所有回文子串个数。

**做法**：PAM 后 `sum(cnt)`。

---

## 6. 易错点与练习

### 易错点

1. **两个根**：奇根 `len = -1`，偶根 `len = 0`；
2. **`getFail` 的边界**：`pos - len - 1 < 0` 要特判；
3. **`cnt` 累加顺序**：按 `len` 从大到小；
4. **多组数据**：清空所有节点；
5. **字符集**：用 `map` 或固定 26；
6. **`fail` 初始化**：两个根的 fail 都指向奇根。

### 练习建议

1. PAM 模板；
2. 本质不同回文串；
3. 出现次数；
4. 以每个位置结尾的回文数；
5. 对比 Manacher。
