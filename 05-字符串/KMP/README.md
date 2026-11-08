# KMP 算法（Knuth-Morris-Pratt）

> KMP 用 $O(n+m)$ 时间完成字符串匹配，核心是 **next（失配）数组**：当匹配失败时，利用已匹配的前缀信息，让模式串「滑」到正确位置，而不是从头再来。

---

## 目录

1. [暴力匹配的问题](#1-暴力匹配的问题)
2. [前缀函数（next 数组）](#2-前缀函数next-数组)
3. [next 数组的求法](#3-next-数组的求法)
4. [KMP 匹配过程](#4-kmp-匹配过程)
5. [模板代码](#5-模板代码)
6. [KMP 的扩展应用](#6-kmp-的扩展应用)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)

---

## 1. 暴力匹配的问题

```
文本串 S: a b a b a b c
模式串 T: a b a b c
              ↑ 失配
```

暴力做法在失配后把 $T$ 右移一位，重新从头比较，最坏 $O(nm)$。

**问题**：失配时，我们已经知道 $S$ 前面匹配了 `abab`，这段信息被浪费了。

**KMP 的洞察**：`abab` 的最长「相等前后缀」是 `ab`，所以可以直接把 $T$ 移到 `ab` 对齐的位置继续比较。

---

## 2. 前缀函数（next 数组）

### 2.1 定义

$\pi[i]$（或 $nxt[i]$）表示：**子串 $T[0..i]$ 的最长相等真前缀与真后缀的长度**。

以 `T = "ababaca"` 为例：

| $i$ | 子串 | 最长相等前后缀 | $\pi[i]$ |
|---|---|---|---|
| 0 | `a` | 无 | 0 |
| 1 | `ab` | 无 | 0 |
| 2 | `aba` | `a` | 1 |
| 3 | `abab` | `ab` | 2 |
| 4 | `ababa` | `aba` | 3 |
| 5 | `ababac` | 无 | 0 |
| 6 | `ababaca` | `a` | 1 |

### 2.2 直观理解

`abab` 的前缀 `ab` 与后缀 `ab` 相等，长度为 2。这意味着：如果匹配到 `abab` 后面失配，前面这 4 个字符的「有效信息」是前 2 个字符已经对上了，可以直接从第 3 个字符继续比。

---

## 3. next 数组的求法

**递推思想**：

- `nxt[0] = 0`（或 `-1`，看写法）；
- 对于 $i > 0$：令 $j = nxt[i-1]$，
  - 若 $T[i] == T[j]$，则 $nxt[i] = j + 1$；
  - 否则令 $j = nxt[j-1]$，继续尝试，直到 $j = 0$。

```cpp
// 0-indexed，nxt[i] = 最长相等前后缀长度
vector<int> buildNext(const string& p) {
    int m = p.size();
    vector<int> nxt(m, 0);
    for (int i = 1, j = 0; i < m; ++i) {
        while (j > 0 && p[i] != p[j]) j = nxt[j-1];
        if (p[i] == p[j]) ++j;
        nxt[i] = j;
    }
    return nxt;
}
```

### 手动模拟 `ababaca`

```
i=1: j=0, p[1]=b != p[0]=a → nxt[1]=0
i=2: j=0, p[2]=a == p[0]=a → j=1, nxt[2]=1
i=3: j=1, p[3]=b == p[1]=b → j=2, nxt[3]=2
i=4: j=2, p[4]=a == p[2]=a → j=3, nxt[4]=3
i=5: j=3, p[5]=c != p[3]=b → j=nxt[2]=1
     p[5]=c != p[1]=b → j=nxt[0]=0
     p[5]=c != p[0]=a → nxt[5]=0
i=6: j=0, p[6]=a == p[0]=a → j=1, nxt[6]=1
```

结果：`[0,0,1,2,3,0,1]`，与上表一致。

---

## 4. KMP 匹配过程

```
S: a b a b a b c
T: a b a b c
```

1. 前 4 个字符匹配成功，第 5 个失配（`S[4]=a` vs `T[4]=c`）；
2. `j = 4`，令 `j = nxt[3] = 2`；
3. 从 `S[4]` 与 `T[2]` 继续比较：`a == a`，`j=3`；
4. `S[5]=b == T[3]=b`，`j=4`；`S[6]=c == T[4]=c`，匹配成功！

**关键**：文本串指针 $i$ 从不回退，只前进。

---

## 5. 模板代码

### 5.1 0-indexed 版

```cpp
#include <bits/stdc++.h>
using namespace std;

// 返回所有匹配起点（0-indexed）
vector<int> kmp(const string& s, const string& p) {
    int n = s.size(), m = p.size();
    if (m == 0) return {};
    vector<int> nxt(m, 0);
    for (int i = 1, j = 0; i < m; ++i) {
        while (j > 0 && p[i] != p[j]) j = nxt[j-1];
        if (p[i] == p[j]) ++j;
        nxt[i] = j;
    }
    vector<int> res;
    for (int i = 0, j = 0; i < n; ++i) {
        while (j > 0 && s[i] != p[j]) j = nxt[j-1];
        if (s[i] == p[j]) ++j;
        if (j == m) {
            res.push_back(i - m + 1);
            j = nxt[j-1];
        }
    }
    return res;
}

int main() {
    string s, p;
    cin >> s >> p;
    auto pos = kmp(s, p);
    for (int x : pos) printf("%d\n", x + 1);   // 输出 1-indexed
    return 0;
}
```

### 5.2 1-indexed 版（经典写法）

```cpp
int nxt[MAXN];

void getNext(const string& p) {
    int m = p.size();
    nxt[1] = 0;
    for (int i = 2, j = 0; i <= m; ++i) {
        while (j && p[i] != p[j+1]) j = nxt[j];
        if (p[i] == p[j+1]) ++j;
        nxt[i] = j;
    }
}

void match(const string& s, const string& p) {
    int n = s.size(), m = p.size();
    getNext(p);
    for (int i = 1, j = 0; i <= n; ++i) {
        while (j && s[i] != p[j+1]) j = nxt[j];
        if (s[i] == p[j+1]) ++j;
        if (j == m) {
            printf("%d\n", i - m + 1);
            j = nxt[j];
        }
    }
}
```

---

## 6. KMP 的扩展应用

### 6.1 求字符串的最小循环节

若 $n \bmod (n - nxt[n]) == 0$，则最小循环节长度为 $n - nxt[n]$，循环次数为 $n / (n - nxt[n])$。

### 6.2 求所有 border（相等前后缀）

```
i = n
while (i > 0) { print(i); i = nxt[i]; }
```

### 6.3 统计每个前缀出现次数

KMP 自动机 + 拓扑/逆序累加：

```cpp
for (int i = n; i >= 1; --i) cnt[nxt[i]] += cnt[i];
```

### 6.4 AC 自动机

KMP 在多模式串上的推广（见 [AC自动机](../AC自动机/README.md)）。

---

## 7. 经典例题

### 7.1 【模板】KMP 字符串匹配

**题意**：$|S|, |P| \le 10^6$，输出 $P$ 在 $S$ 中所有出现位置。

**复杂度**：$O(n + m)$。

### 7.2 最小循环节

**题意**：给字符串 $S$，求最小循环节长度和循环次数。

**做法**：KMP 求 $nxt$，判断 $n \bmod (n - nxt[n])$。

### 7.3 字符串周期

**题意**：求所有周期。

**做法**：沿 $nxt$ 链向上跳。

### 7.4 前缀出现次数

**题意**：统计每个前缀在原串中出现次数。

**做法**：逆序累加 `cnt[nxt[i]] += cnt[i]`。

### 7.5 最长公共前后缀

直接输出 `nxt[n]`。

---

## 8. 易错点

1. **下标从 0 还是 1**：两种写法不能混用；
2. **`nxt[0]` 的值**：0-indexed 写成 0 或 -1 都可以，但逻辑要一致；
3. **失配后 `j = nxt[j-1]`**：0-indexed 是 `j-1`，1-indexed 是 `j`；
4. **匹配成功后的处理**：`j = nxt[j-1]`（0-indexed）以继续找重叠匹配；
5. **空串**：特判；
6. **`while` 条件顺序**：先判 `j > 0` 再访问 `p[j]`；
7. **多组数据**：`nxt` 数组要重置。
