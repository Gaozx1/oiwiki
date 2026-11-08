# 字典树（Trie）

> 一句话概括：**Trie 把字符串按字符逐层挂在树上，用「公共前缀共享路径」的方式存储与检索字符串**，插入和查询都是 $O(L)$（$L$ 为串长），是前缀检索、字符串匹配、01-Trie 异或问题的核心结构。

---

## 目录

1. [什么是 Trie](#1-什么是-trie)
2. [结构原理](#2-结构原理)
3. [插入与查询](#3-插入与查询)
4. [完整模板代码](#4-完整模板代码)
5. [进阶一：01-Trie](#5-进阶一01-trie)
6. [进阶二：Trie 的其他变体](#6-进阶二trie-的其他变体)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [复杂度与空间分析](#9-复杂度与空间分析)
10. [练习建议](#10-练习建议)

---

## 1. 什么是 Trie

### 1.1 问题的提出

维护一个字符串集合，支持：

- 插入一个字符串；
- 查询某个字符串是否出现过；
- 查询有多少个字符串以某个串为前缀；
- 求多个串的最长公共前缀。

用 `std::set<string>`，比较两个字符串是 $O(L)$，而且**前缀查询需要遍历所有串**。我们需要一种按「字符」组织的数据结构——Trie。

### 1.2 直观图示

插入 `app`、`apple`、`apart`、`bat`、`batch`：

```
        (root)
        /    \
       a      b
       |      |
       p      a
      / \     |
     p   r    t
     |   |    |
     l   t    c
     |        |
     e        h
```

- 从根到某节点的路径拼起来就是一个前缀；
- `app`、`apple` 共享路径 `a→p→p`；
- 每个节点代表一个**前缀**，不一定是完整的单词。

### 1.3 关键性质

1. **公共前缀共享同一条路径**，节省空间；
2. 查询一个串只需沿着树走 $L$ 步，与集合大小无关；
3. 节点需要标记「这里是否是一个单词的结尾」（`isEnd`），以及「经过这个前缀的串有多少个」（`cnt`）。

---

## 2. 结构原理

### 2.1 节点定义

```cpp
const int MAXN = 100005;      // 节点数上限 = 所有串长之和 + 1
const int SIGMA = 26;         // 字符集大小（小写字母）

int ch[MAXN][SIGMA];          // ch[u][c]：节点 u 沿字符 c 走到的儿子编号，0 表示不存在
int cnt[MAXN];                // 经过节点 u 的字符串数量（前缀计数）
bool isEnd[MAXN];             // 节点 u 是否是一个字符串的结尾
int tot = 0;                  // 已用节点数，0 号是根
```

### 2.2 空间估算

节点数 = 所有插入串的长度之和 $+ 1$；每个节点存 $26$ 个 `int`（$104$ 字节），$10^5$ 个字符约 $10$ MB。若字符集是全体 ASCII（$128$），内存会爆炸，此时应改用 `map` 或哈希表存儿子。

### 2.3 字符映射

```cpp
int id(char c) { return c - 'a'; }   // 小写字母
```

注意字符集不同时映射要改：大小写敏感用 `c - 'A'`，数字用 `c - '0'`。

---

## 3. 插入与查询

### 3.1 插入

从根出发，对每个字符：

- 若对应的儿子不存在，新建节点；
- 走到该儿子，沿途 `cnt++`；
- 最后一个节点 `isEnd = true`。

```cpp
void insert(const string& s) {
    int u = 0;                        // 从根开始
    for (char c : s) {
        int v = c - 'a';
        if (!ch[u][v]) ch[u][v] = ++tot;   // 没有就新建
        u = ch[u][v];
        ++cnt[u];                     // 经过该前缀的串数 +1
    }
    isEnd[u] = true;
}
```

### 3.2 查询单词是否存在

```cpp
bool find(const string& s) {
    int u = 0;
    for (char c : s) {
        int v = c - 'a';
        if (!ch[u][v]) return false;  // 路径断了，不存在
        u = ch[u][v];
    }
    return isEnd[u];                  // 必须是某个单词的结尾
}
```

### 3.3 查询前缀出现次数

```cpp
int countPrefix(const string& s) {
    int u = 0;
    for (char c : s) {
        int v = c - 'a';
        if (!ch[u][v]) return 0;
        u = ch[u][v];
    }
    return cnt[u];                    // 经过该节点的串数
}
```

---

## 4. 完整模板代码

支持插入、查询单词、查询前缀次数、删除单词：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;      // 节点总数上限
const int SIGMA = 26;

int ch[MAXN][SIGMA];
int cnt[MAXN];                // 经过该节点的字符串数
int wordEnd[MAXN];            // 以该节点结尾的字符串数（可重集合）
int tot = 0;

void insert(const string& s) {
    int u = 0;
    for (char c : s) {
        int v = c - 'a';
        if (!ch[u][v]) ch[u][v] = ++tot;
        u = ch[u][v];
        ++cnt[u];
    }
    ++wordEnd[u];             // 允许重复插入同一个串
}

bool find(const string& s) {
    int u = 0;
    for (char c : s) {
        int v = c - 'a';
        if (!ch[u][v]) return false;
        u = ch[u][v];
    }
    return wordEnd[u] > 0;
}

int countPrefix(const string& s) {
    int u = 0;
    for (char c : s) {
        int v = c - 'a';
        if (!ch[u][v]) return 0;
        u = ch[u][v];
    }
    return cnt[u];
}

// 删除一个字符串（保证存在）
void erase(const string& s) {
    int u = 0;
    for (char c : s) {
        int v = c - 'a';
        u = ch[u][v];
        --cnt[u];
    }
    --wordEnd[u];
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    for (int i = 0; i < n; ++i) {
        string s;
        cin >> s;
        insert(s);
    }

    int m;
    cin >> m;
    while (m--) {
        int op;
        string s;
        cin >> op >> s;
        if (op == 1) insert(s);
        else if (op == 2) erase(s);
        else if (op == 3) cout << (find(s) ? "YES" : "NO") << '\n';
        else cout << countPrefix(s) << '\n';
    }
    return 0;
}
```

### 4.1 要不要用指针？

可以用 `struct Node { Node* ch[26]; }` 的动态写法，但 `new` 很慢、大量小对象造成内存碎片；竞赛中一律用**数组模拟**（如上），更快更省心。

---

## 5. 进阶一：01-Trie

### 5.1 把整数看成二进制串

把每个非负整数按二进制从高位到低位插入 Trie，得到一棵**二叉 Trie**：

```
插入 5 (101)、3 (011)、6 (110)，从第 2 位到第 0 位：

              root
             /    \
            0      1
            |     / \
            1    0   1
            |    |   |
            1    1   0
           (3)  (5) (6)
```

（括号里是最终落到该叶子的数）

每个节点只有两个儿子（0 / 1），树高为位数 $B$（如 $B = 31$）。

### 5.2 最大异或对

**问题**：给定 $n$ 个数，求 $\max_{i \ne j} (a_i \oplus a_j)$。

**思路**：把所有数插入 01-Trie。对每个 $a_i$，在树上**贪心**地走「与当前位相反」的分支（这样异或结果该位为 1），若不存在才走相同分支。

```cpp
const int MAXB = 31;          // 值域 < 2^31

int ch[MAXN * 32][2];
int tot = 0;

void insertInt(int x) {
    int u = 0;
    for (int b = MAXB; b >= 0; --b) {
        int v = (x >> b) & 1;
        if (!ch[u][v]) ch[u][v] = ++tot;
        u = ch[u][v];
    }
}

int queryMaxXor(int x) {
    int u = 0, res = 0;
    for (int b = MAXB; b >= 0; --b) {
        int v = (x >> b) & 1;
        int want = v ^ 1;                 // 希望走相反位
        if (ch[u][want]) {
            res |= (1 << b);              // 该位异或结果为 1
            u = ch[u][want];
        } else {
            u = ch[u][v];
        }
    }
    return res;
}
```

**复杂度**：$O(n \cdot B)$，$B = 31$ 时约 $3 \times 10^6$ 次操作。

### 5.3 01-Trie 的其他用法

| 问题 | 做法 |
|---|---|
| 最大异或对 | 贪心走相反位 |
| 第 $k$ 大异或 | 每个节点维护子树大小，按位二分 |
| 可持久化 01-Trie | 支持区间异或最大值（见[可持久化数据结构](../可持久化数据结构/README.md)） |
| 异或路径问题 | 树上差分 + 01-Trie |

---

## 6. 进阶二：Trie 的其他变体

### 6.1 两种常见变体

- **AC 自动机**：在 Trie 上建 `fail` 指针，就能做多模式串匹配（详见字符串章节）。
- **压缩 Trie（Patricia Trie / 基数树）**：把「只有一个儿子的链」压缩成一条边，边上存一个字符串，适合稀疏长串，如 IP 路由表。

### 6.2 后缀 Trie 与可持久化 Trie

把所有后缀插入 Trie 得到后缀 Trie，压缩后即后缀树；但它空间 $O(n^2)$，实际多用后缀数组 / 后缀自动机替代。**可持久化 Trie** 则每次插入只新建 $O(B)$ 个节点、其余共享，支持「查询某个历史版本」「区间异或最大值」，详见[可持久化数据结构](../可持久化数据结构/README.md)。

---

## 7. 经典例题

### 7.1 例题一：Trie 字符串统计（模板题）

**题意**：维护字符串集合，支持插入与查询某个串出现次数。$n \le 10^5$，总长度 $\le 10^5$。

**思路**：直接套用第 4 节模板，用 `wordEnd[u]` 计数。

**复杂度**：单次 $O(L)$，总 $O(\sum L)$。

### 7.2 例题二：最长公共前缀

**题意**：给定 $n$ 个字符串，求它们的最长公共前缀。

**思路**：把所有串插入 Trie。从根出发，只要当前节点**只有一个儿子**且**不是任何串的结尾**，就继续往下走；走的步数就是最长公共前缀长度。

```cpp
int longestCommonPrefix(int n) {
    int u = 0, len = 0;
    while (true) {
        int sonCnt = 0, nxt = -1;
        for (int c = 0; c < 26; ++c) {
            if (ch[u][c]) { ++sonCnt; nxt = ch[u][c]; }
        }
        if (sonCnt != 1 || wordEnd[u] > 0) break;   // 分叉或已到单词结尾
        u = nxt;
        ++len;
    }
    return len;
}
```

**复杂度**：$O(\sum L)$。

### 7.3 例题三：最大异或对

**题意**：给定 $n$ 个整数，求 $\max_{i \ne j}(a_i \oplus a_j)$。$n \le 10^5$，$a_i < 2^{31}$。

**思路**：01-Trie 贪心，见 5.2 节。

**复杂度**：$O(n \log V)$。

### 7.4 例题四：单词接龙 / 前缀匹配

**题意**：给定词典，判断某个句子能否被词典中的单词拼接而成。

**思路**：把词典建 Trie，对句子做 DP：`dp[i]` 表示前 $i$ 个字符能否被拼出。转移时从 $i$ 出发在 Trie 上走，每走到一个 `wordEnd` 就更新对应的 `dp[j]`。

**复杂度**：$O(|S| \cdot L_{\max})$。

---

## 8. 易错点
1. **节点数组开小**：节点数 = 所有串长之和 + 1，不是 $n$。多组数据要重置 `tot = 0`。
2. **根节点是 0**：`ch[0][c]` 为 0 表示不存在，所以真实节点从 1 开始编号，避免与「空」混淆。
3. **`isEnd` 与 `cnt` 混用**：存在性看 `wordEnd`，前缀计数看 `cnt`，两者含义不同。
4. **删除不存在的串**：`erase` 前要确认存在，否则 `cnt` 会变负。
5. **字符集映射**：`c - 'a'` 只适用小写字母；出现大写、数字要改，或统一 `c - 'a' + 26`。
6. **01-Trie 的位数**：值域 `< 2^31` 要从第 30 位（或 31 位）开始，不是从第 0 位；位数不够会漏高位。
7. **01-Trie 空间**：节点数 = 插入次数 $\times$ 位数，$10^5 \times 31 = 3.1 \times 10^6$，数组要开够。
8. **`int` 溢出**：`1 << 31` 是未定义行为，用 `1LL << b`。
9. **字符串含空格**：`cin >> s` 遇空格停止，要用 `getline`。
10. **多组数据重置**：只重置 `tot` 不够，`ch` 数组里旧节点的儿子指针仍存在，建议 `memset(ch, 0, sizeof(ch))`。

---

## 9. 复杂度与空间分析

### 9.1 时间

插入、查询、前缀计数都是 $O(L)$；建整棵 Trie 是 $O(\sum L)$；01-Trie 的插入与查询是 $O(B)$（$B$ 为位数）。

### 9.2 空间

| 实现 | 空间 | 说明 |
|---|---|---|
| 数组 `ch[MAXN][26]` | $26 \times 4 \times MAXN$ 字节 | $10^5$ 节点约 10 MB |
| `map<char,int>` 存儿子 | $O(节点数 \log \sigma)$ | 字符集大时用 |
| 哈希表存儿子 | 期望 $O(节点数)$ | 字符集极大时用 |
| 01-Trie | $2 \times 4 \times (n \times B)$ 字节 | $10^5 \times 31$ 约 25 MB |

### 9.3 与哈希表的对比

| 维度 | Trie | 哈希表 |
|---|---|---|
| 查询单词 | $O(L)$ | 期望 $O(L)$ |
| 前缀查询 | ✅ $O(L)$ | ❌ 需遍历所有串 |
| 有序遍历 | ✅ 按字典序 | ❌ 无序 |
| 空间 | 较大（$26$ 倍） | 较小 |
| 抗冲突 | 天然无冲突 | 需处理哈希冲突 |

**结论**：需要前缀操作就用 Trie；只需精确匹配且内存紧张就用哈希。

---

## 10. 练习建议

### 10.1 由易到难

| 难度 | 题目类型 | 考察点 |
|---|---|---|
| ★ | Trie 字符串统计 | 模板 |
| ★★ | 最长公共前缀 | 树上遍历 |
| ★★ | 单词替换 / 前缀匹配 | Trie + 贪心 |
| ★★★ | 最大异或对 | 01-Trie |
| ★★★ | 单词接龙 DP | Trie + DP |
| ★★★★ | 可持久化 01-Trie | 区间异或 |
| ★★★★ | AC 自动机 | Trie + fail 指针 |
| ★★★★★ | 后缀树 / 广义 SAM | 进阶 |

### 10.2 学习检查清单

- [ ] 能画出插入若干字符串后的 Trie 结构，能默写 `insert` / `find` / `countPrefix`
- [ ] 能说清 `wordEnd` 与 `cnt` 的区别，能写出 01-Trie 的最大异或对
- [ ] 能估算 Trie 的空间占用，知道 Trie 与哈希表的取舍

### 10.3 推荐练习顺序

1. 洛谷 P2580 于是他错误的点名开始了（Trie 模板）
2. 洛谷 P2922 [USACO08DEC] Secret Message G（前缀计数）
3. 洛谷 P4551 最长异或路径（01-Trie）
4. 洛谷 P4735 最大异或和（可持久化 01-Trie）
5. 洛谷 P3966 [TJOI2013] 单词（Trie + fail 树）
6. 洛谷 P2444 [POI2000] 病毒（AC 自动机）

---

> **核心思想**：Trie 用「字符逐层分叉」把字符串集合组织成树，让前缀共享路径、查询只与串长有关。它是字符串前缀问题的标准答案，也是 01-Trie、AC 自动机、可持久化 Trie 的共同基础。
