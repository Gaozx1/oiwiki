# 莫比乌斯反演（Möbius Inversion）

> 莫比乌斯反演 = 用 $\mu$ 函数把「约数和」转成「原函数」。它能把形如 $\sum_{d|n}$ 的式子化简，是数论计数的核心工具。

---

## 目录

1. [莫比乌斯函数](#1-莫比乌斯函数)
2. [莫比乌斯函数的性质](#2-莫比乌斯函数的性质)
3. [莫比乌斯反演](#3-莫比乌斯反演)
4. [线性筛求 μ](#4-线性筛求-μ)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 莫比乌斯函数

$$
\mu(n) = \begin{cases}
1 & n = 1 \\
(-1)^k & n = p_1 p_2 \cdots p_k \text{（互不相同质因子）}\\
0 & n \text{ 含有平方因子}
\end{cases}
$$

**例子**：

| $n$ | $\mu(n)$ | 说明 |
|---|---|---|
| 1 | 1 | 定义 |
| 2 | -1 | 一个质因子 |
| 4 | 0 | $2^2$ |
| 6 | 1 | $2 \times 3$ |
| 12 | 0 | $2^2 \times 3$ |

---

## 2. 莫比乌斯函数的性质

### 2.1 关键恒等式

$$
\sum_{d \mid n} \mu(d) = [n = 1]
$$

（$[P]$ 表示 $P$ 为真时取 1，否则取 0）

### 2.2 与欧拉函数的关系

$$
\sum_{d \mid n} \mu(d) \cdot \frac{n}{d} = \varphi(n)
$$

### 2.3 狄利克雷卷积

$(\mu * 1)(n) = \sum_{d|n}\mu(d) = [n=1] = \varepsilon(n)$

---

## 3. 莫比乌斯反演

### 3.1 形式一（约数形式）

若

$$
F(n) = \sum_{d \mid n} f(d)
$$

则

$$
f(n) = \sum_{d \mid n} \mu(d) F\left(\frac{n}{d}\right)
$$

### 3.2 形式二（倍数形式）

若

$$
F(n) = \sum_{n \mid d} f(d)
$$

则

$$
f(n) = \sum_{n \mid d} \mu\left(\frac{d}{n}\right) F(d)
$$

### 3.3 常用转化

$$
\sum_{i=1}^{n}\sum_{j=1}^{m} [\gcd(i,j) = 1]
= \sum_{d=1}^{\min(n,m)} \mu(d) \left\lfloor \frac{n}{d} \right\rfloor \left\lfloor \frac{m}{d} \right\rfloor
$$

这个式子是**最常用的套路**：把「互质」转成「枚举约数 $d$ + 容斥系数 $\mu(d)$」。

---

## 4. 线性筛求 μ

```cpp
const int N = 10000005;
int primes[N], cnt = 0;
bool isComp[N];
int mu[N];

void sieveMu(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; ++i) {
        if (!isComp[i]) { primes[++cnt] = i; mu[i] = -1; }
        for (int j = 1; j <= cnt && (long long)i * primes[j] <= n; ++j) {
            int p = primes[j];
            isComp[i * p] = true;
            if (i % p == 0) { mu[i * p] = 0; break; }
            else mu[i * p] = -mu[i];
        }
    }
}
```

**复杂度**：$O(n)$。

---

## 5. 模板代码

### 5.1 求 $\sum_{i=1}^{n}\sum_{j=1}^{m} [\gcd=1]$

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const int N = 10000005;
int primes[N], cnt = 0, mu[N], sumMu[N];
bool isComp[N];

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; ++i) {
        if (!isComp[i]) { primes[++cnt] = i; mu[i] = -1; }
        for (int j = 1; j <= cnt && (ll)i * primes[j] <= n; ++j) {
            int p = primes[j];
            isComp[i * p] = true;
            if (i % p == 0) { mu[i * p] = 0; break; }
            mu[i * p] = -mu[i];
        }
    }
    for (int i = 1; i <= n; ++i) sumMu[i] = sumMu[i-1] + mu[i];
}

// 数论分块求 F(n, m)
ll solve(int n, int m) {
    if (n > m) swap(n, m);
    ll ans = 0;
    for (int l = 1, r; l <= n; l = r + 1) {
        r = min(n / (n / l), m / (m / l));
        ans += (ll)(sumMu[r] - sumMu[l-1]) * (n / l) * (m / l);
    }
    return ans;
}
```

### 5.2 数论分块

```cpp
for (int l = 1, r; l <= n; l = r + 1) {
    r = n / (n / l);   // 值相同的区间右端点
    // 处理 [l, r]
}
```

**复杂度**：$O(\sqrt n)$。

---

## 6. 经典例题

### 6.1 【模板】$\gcd$ 为 1 的数对

**题意**：$n, m \le 10^7$，求 $\sum\sum [\gcd(i,j)=1]$。

**做法**：莫比乌斯反演 + 数论分块。

### 6.2 $\gcd$ 为 $k$ 的数对

**题意**：求 $\gcd(i,j) = k$ 的对数。

**做法**：令 $i' = i/k$，$j' = j/k$，转化为 $\gcd(i',j')=1$。

### 6.3 最大公约数之和

**题意**：求 $\sum_{i,j} \gcd(i,j)$。

**做法**：$\gcd = \sum_{d | \gcd} \varphi(d)$（欧拉反演）或莫比乌斯反演。

### 6.4 完全平方数

**题意**：统计满足某种约数条件的数。

**做法**：$\mu$ 的容斥含义。

### 6.5 杜教筛

**题意**：求 $\sum_{i=1}^{n} \mu(i)$，$n \le 10^{10}$。

**做法**：杜教筛（莫比乌斯反演 + 记忆化递归）。

---

## 7. 易错点与练习

### 易错点

1. **$\mu(1) = 1$**：不要漏掉；
2. **线性筛的 `break`**：`i % p == 0` 时 `mu[i*p] = 0` 并 break；
3. **前缀和数组**：数论分块要用 `sumMu`；
4. **溢出**：$\lfloor n/d \rfloor \lfloor m/d \rfloor$ 可能超 `int`；
5. **$n, m$ 大小**：先 `swap` 让 $n \le m$；
6. **多组询问**：预处理前缀和，每组 $O(\sqrt n)$。

### 练习建议

1. 线性筛求 $\mu$；
2. $\gcd=1$ 数对；
3. $\gcd=k$ 数对；
4. 最大公约数之和；
5. 杜教筛（进阶）。
