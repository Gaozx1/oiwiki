# 容斥原理（Inclusion-Exclusion Principle）

> 容斥原理 = **先全部算上，再减去重复的，再加回多减的……** 它是解决「至少 / 至多 / 恰好」计数问题的核心工具。

---

## 目录

1. [两个集合](#1-两个集合)
2. [三个集合](#2-三个集合)
3. [一般形式](#3-一般形式)
4. [子集反演](#4-子集反演)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 两个集合

$$
|A \cup B| = |A| + |B| - |A \cap B|
$$

**直观**：$A$ 和 $B$ 重叠的部分被算了两次，减一次。

---

## 2. 三个集合

$$
|A \cup B \cup C| = |A| + |B| + |C| - |A\cap B| - |A\cap C| - |B\cap C| + |A\cap B\cap C|
$$

**规律**：奇数个集合的交加，偶数个集合的交减。

---

## 3. 一般形式

$$
\left|\bigcup_{i=1}^{n} A_i\right| = \sum_{\emptyset \ne S \subseteq \{1..n\}} (-1)^{|S|-1} \left|\bigcap_{i \in S} A_i\right|
$$

**另一种常见形式**（求补集）：

$$
|\overline{A_1} \cap \overline{A_2} \cap \cdots \cap \overline{A_n}|
= |U| - \sum |A_i| + \sum |A_i \cap A_j| - \cdots
$$

### 3.1 常用技巧

- **求「至少一个」**：用 $\cup$；
- **求「一个都不」**：用补集；
- **求「恰好 $k$ 个」**：通常转化为「至少 $k$ 个」的容斥：

$$
\text{恰好 } k = \sum_{i=k}^{n} (-1)^{i-k} \binom{i}{k} \cdot (\text{至少 } i)
$$

---

## 4. 子集反演

**公式**：

$$
f(S) = \sum_{T \subseteq S} g(T)
\iff
g(S) = \sum_{T \subseteq S} (-1)^{|S|-|T|} f(T)
$$

**用途**：把「子集和」转成「恰好」。

**例子**：求恰好选 $k$ 个的方案数，可以先求「至少选 $k$ 个」再反演。

---

## 5. 模板代码

### 5.1 枚举子集

```cpp
for (int S = 1; S < (1 << n); ++S) {
    int bits = __builtin_popcount(S);
    if (bits % 2 == 1) ans += calc(S);   // 奇数个交加
    else ans -= calc(S);                  // 偶数个交减
}
```

### 5.2 枚举子集的子集

```cpp
for (int S = 0; S < (1 << n); ++S)
    for (int T = S; T; T = (T - 1) & S)
        // T 是 S 的非空子集
```

**复杂度**：$O(3^n)$。

### 5.3 容斥求「至少」

```cpp
long long solve() {
    long long ans = 0;
    for (int S = 1; S < (1 << n); ++S) {
        long long cnt = countIntersection(S);
        if (__builtin_popcount(S) & 1) ans += cnt;
        else ans -= cnt;
    }
    return ans;
}
```

---

## 6. 经典例题

### 6.1 错排问题

**题意**：$n$ 个元素的全排列中，没有元素在原位的排列数。

**容斥**：

$$
D_n = n! \sum_{i=0}^{n} \frac{(-1)^i}{i!}
$$

递推：$D_n = (n-1)(D_{n-1} + D_{n-2})$。

### 6.2 求 $[1, n]$ 中与 $m$ 互质的数

**题意**：$m$ 的质因子为 $p_1..p_k$，求 $[1,n]$ 中不被任何 $p_i$ 整除的数的个数。

**容斥**：

$$
\text{ans} = n - \sum \lfloor n/p_i \rfloor + \sum \lfloor n/(p_ip_j) \rfloor - \cdots
$$

```cpp
long long countCoprime(long long n, vector<int>& primes) {
    long long ans = n;
    int k = primes.size();
    for (int S = 1; S < (1 << k); ++S) {
        long long prod = 1;
        int bits = 0;
        for (int i = 0; i < k; ++i)
            if (S >> i & 1) { prod *= primes[i]; ++bits; }
        if (bits & 1) ans -= n / prod;
        else ans += n / prod;
    }
    return ans;
}
```

### 6.3 至少选一个

**题意**：$n$ 个物品，求至少选一个的方案数。

**答案**：$2^n - 1$。

### 6.4 恰好 $k$ 个

**题意**：$n$ 个条件，求恰好满足 $k$ 个的方案数。

**做法**：先算「至少 $k$ 个」，再容斥。

### 6.5 多重集排列

**题意**：有重复元素的排列数。

**公式**：

$$
\frac{n!}{c_1! c_2! \cdots c_k!}
$$

### 6.6 硬币问题

**题意**：用面值 $a_1..a_k$ 的硬币凑 $n$，每类个数不限，求方案数。

**做法**：生成函数或容斥。

---

## 7. 易错点与练习

### 易错点

1. **符号**：奇数个集合的交是加，偶数是减；
2. **空集**：一般从 $S = 1$ 开始，空集不加；
3. **溢出**：乘积可能很大，用 `long long` 或 `__int128`；
4. **重复计算**：容斥的「至少」定义要清晰；
5. **质因子分解**：先对 $m$ 分解质因数，得到 $p_1..p_k$；
6. **$n$ 很大**：用 `n / prod` 而不是枚举。

### 练习建议

1. 两集合容斥（模板）；
2. 错排问题；
3. 与 $m$ 互质的数；
4. 恰好 $k$ 个的容斥；
5. 子集反演（$\text{AND}$ / $\text{OR}$ 卷积）。
