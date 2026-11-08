# 字符串哈希（String Hashing）

> 字符串哈希 = 把字符串映射成一个整数，从而把「比较字符串」变成「比较整数」。它是 OI 中最实用的技巧之一。

---

## 目录

1. [基本原理](#1-基本原理)
2. [滚动哈希](#2-滚动哈希)
3. [子串哈希](#3-子串哈希)
4. [双哈希与防卡](#4-双哈希与防卡)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)

---

## 1. 基本原理

把字符串看作一个 **base 进制数**：

$$
H(s) = \sum_{i=1}^{n} s_i \cdot base^{n-i} \pmod p
$$

例如 `"abc"` 在 base=131 下：

$$
H = 97 \times 131^2 + 98 \times 131 + 99
$$

**性质**：不同的字符串大概率得到不同的哈希值（有碰撞概率）。

**常用参数**：

| 参数 | 常用值 |
|---|---|
| base | 131、13331、19260817 |
| mod | $10^9+7$、$10^9+9$、$2^{64}$（自然溢出） |

> 自然溢出就是 `unsigned long long` 自动取模 $2^{64}$，写起来最方便但**容易被卡**。

---

## 2. 滚动哈希

**目标**：$O(n)$ 预处理出所有前缀的哈希，$O(1)$ 查询任意子串哈希。

定义前缀哈希：

$$
h[i] = h[i-1] \cdot base + s_i \pmod p
$$

同时预处理 $base$ 的幂：

$$
pw[i] = pw[i-1] \cdot base \pmod p
\]

那么子串 $s[l..r]$ 的哈希为：

$$
H(l,r) = h[r] - h[l-1] \cdot pw[r-l+1] \pmod p
$$

**为什么？** 把 $h[r]$ 中属于前缀 $[1, l-1]$ 的部分「减掉」。

---

## 3. 子串哈希

```cpp
h[r] - h[l-1] * pw[r-l+1]
```

**直观图示**：

```
s:  [1 ... l-1][l ... r]
h[r] = h[l-1] * base^(r-l+1) + hash(l..r)
所以 hash(l..r) = h[r] - h[l-1] * base^(r-l+1)
```

---

## 4. 双哈希与防卡

单哈希有碰撞风险。解决方法：

1. **双哈希**：用两组 $(base_1, p_1)$ 和 $(base_2, p_2)$，两个哈希都相等才认为相等；
2. **随机 base**：用 `chrono` 生成随机 base，防止出题人构造碰撞；
3. **自然溢出 + 大质数模**：组合使用。

### 双哈希模板

```cpp
struct Hash {
    static const int P1 = 1000000007, P2 = 1000000009;
    static const int B1 = 131, B2 = 13331;
    vector<long long> h1, h2, p1, p2;

    Hash(const string& s) {
        int n = s.size();
        h1.assign(n + 1, 0); h2.assign(n + 1, 0);
        p1.assign(n + 1, 1); p2.assign(n + 1, 1);
        for (int i = 1; i <= n; ++i) {
            h1[i] = (h1[i-1] * B1 + s[i-1]) % P1;
            h2[i] = (h2[i-1] * B2 + s[i-1]) % P2;
            p1[i] = p1[i-1] * B1 % P1;
            p2[i] = p2[i-1] * B2 % P2;
        }
    }

    pair<long long, long long> get(int l, int r) {   // 1-indexed, 闭区间
        long long a = ((h1[r] - h1[l-1] * p1[r-l+1]) % P1 + P1) % P1;
        long long b = ((h2[r] - h2[l-1] * p2[r-l+1]) % P2 + P2) % P2;
        return {a, b};
    }
};
```

---

## 5. 模板代码

### 5.1 自然溢出（最快，比赛常用）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef unsigned long long ull;

const ull BASE = 131;

struct StrHash {
    vector<ull> h, pw;
    int n;

    StrHash(const string& s) {
        n = s.size();
        h.assign(n + 1, 0);
        pw.assign(n + 1, 1);
        for (int i = 1; i <= n; ++i) {
            h[i] = h[i-1] * BASE + (unsigned char)s[i-1];
            pw[i] = pw[i-1] * BASE;
        }
    }

    ull get(int l, int r) {    // 1-indexed
        return h[r] - h[l-1] * pw[r-l+1];
    }
};

int main() {
    string a = "hello world", b = "world";
    StrHash ha(a), hb(b);
    int n = a.size(), m = b.size();
    int cnt = 0;
    for (int i = 1; i + m - 1 <= n; ++i)
        if (ha.get(i, i + m - 1) == hb.get(1, m)) ++cnt;
    printf("%d\n", cnt);
    return 0;
}
```

### 5.2 随机 base 防卡

```cpp
mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
const ull BASE = rng() % 100000 + 131;   // 随机 base
```

---

## 6. 经典例题

### 6.1 字符串匹配

**题意**：判断模式串 $T$ 在文本串 $S$ 中出现几次。

**做法**：预处理 $S$ 的哈希，枚举起点比较，$O(n)$。

### 6.2 最长回文子串（哈希 + 二分）

**题意**：求最长回文子串。

**做法**：预处理正反哈希，枚举中心二分回文半径，$O(n \log n)$。

```cpp
// 判断 s[l..r] 是否回文：正哈希 == 反哈希
bool isPalindrome(int l, int r) {
    return h.get(l, r) == rh.get(n - r + 1, n - l + 1);
}
```

### 6.3 最长公共子串（哈希 + 二分）

**题意**：两个字符串的最长公共子串。

**做法**：二分长度，把 $A$ 的所有该长度子串哈希丢进 `set`，再查 $B$。

### 6.4 判断循环同构

**题意**：判断 $B$ 是否是 $A$ 的循环移位。

**做法**：把 $A$ 复制一份 $AA$，判断 $B$ 是否为 $AA$ 的子串。

### 6.5 字符串去重

用哈希 + `sort` + `unique` 对字符串数组去重。

---

## 7. 易错点

1. **自然溢出会被卡**：出题人可以构造 $2^{64}$ 的碰撞；重大比赛用双哈希；
2. **base 太小**：容易碰撞，建议 $\ge 131$；
3. **base 是偶数**：会让冲突概率上升，选奇数；
4. **`unsigned char` 转换**：`char` 可能是负数，转 `unsigned char` 保证正数；
5. **子串哈希下标**：`h[r] - h[l-1]*pw[r-l+1]`，注意是 `l-1`；
6. **取模减法**：可能出现负数，要 `(x % p + p) % p`；
7. **`ull` 自然溢出**：不要手动取模，直接让溢出。

### 卡哈希的常见手法

- 构造两个不同字符串但哈希相同（很难但不难自动生成）；
- 出题人用固定 base 生成反例；
- 解决办法：随机 base + 双哈希。
