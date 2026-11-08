# 多项式与 FFT（Polynomial & Fast Fourier Transform）

> FFT 用 $O(n \log n)$ 完成多项式乘法。核心是把「系数表示」转成「点值表示」，在点值下乘法是 $O(n)$，再用逆变换转回来。

---

## 目录

1. [多项式表示](#1-多项式表示)
2. [为什么需要 FFT](#2-为什么需要-fft)
3. [单位根](#3-单位根)
4. [FFT 流程](#4-fft-流程)
5. [模板代码](#5-模板代码)
6. [NTT（数论变换）](#6-ntt数论变换)
7. [经典例题](#7-经典例题)
8. [易错点与练习](#8-易错点与练习)

---

## 1. 多项式表示

多项式 $A(x) = a_0 + a_1x + \cdots + a_{n-1}x^{n-1}$ 有两种表示：

**系数表示**：$\{a_0, a_1, \ldots, a_{n-1}\}$

**点值表示**：$\{(x_0, A(x_0)), (x_1, A(x_1)), \ldots\}$

**关键性质**：

- 两个多项式的**点值**可以直接相乘：$C(x_i) = A(x_i) \cdot B(x_i)$；
- $n$ 个点可以唯一确定一个 $n-1$ 次多项式（拉格朗日插值）。

---

## 2. 为什么需要 FFT

普通多项式乘法是 $O(n^2)$：

$$
c_k = \sum_{i+j=k} a_i b_j
$$

FFT 的思路：

1. 选 $n$ 个特殊点（单位根），求 $A, B$ 在这些点的值（**DFT**）；
2. 点值相乘；
3. 用逆 DFT 转回系数。

暴力求值 $O(n^2)$，但选单位根可以让求值变成 $O(n \log n)$（分治）。

---

## 3. 单位根

复数域上，$n$ 次单位根为：

$$
\omega_n = e^{2\pi i / n} = \cos\frac{2\pi}{n} + i\sin\frac{2\pi}{n}
$$

**性质**：

1. $\omega_n^n = 1$；
2. $\omega_n^{k} = \omega_n^{k \bmod n}$；
3. $\omega_n^{2k} = \omega_{n/2}^{k}$（折半引理）；
4. $\omega_n^{k + n/2} = -\omega_n^k$（对称性）。

**折半引理是分治的基础**。

---

## 4. FFT 流程

### 4.1 分治（Cooley-Tukey）

把 $A(x)$ 按奇偶次项拆开：

$$
A(x) = A_0(x^2) + x A_1(x^2)
$$

其中 $A_0$ 是偶次项，$A_1$ 是奇次项。

代入 $\omega_n^k$：

$$
A(\omega_n^k) = A_0(\omega_{n/2}^k) + \omega_n^k A_1(\omega_{n/2}^k)
$$

对 $k$ 和 $k + n/2$ 分别计算，递归规模减半。

### 4.2 蝴蝶操作

```
u = A0(ω^k)
v = ω^k * A1(ω^k)
A(ω^k)       = u + v
A(ω^(k+n/2)) = u - v
```

### 4.3 逆变换

用 $\omega_n^{-1}$ 做同样的变换，最后除以 $n$。

---

## 5. 模板代码

### 5.1 迭代版 FFT

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1 << 21;
const double PI = acos(-1.0);

struct Complex {
    double x, y;
    Complex(double x = 0, double y = 0) : x(x), y(y) {}
    Complex operator+(const Complex& b) const { return Complex(x + b.x, y + b.y); }
    Complex operator-(const Complex& b) const { return Complex(x - b.x, y - b.y); }
    Complex operator*(const Complex& b) const {
        return Complex(x * b.x - y * b.y, x * b.y + y * b.x);
    }
};

int rev[N];

void fft(Complex* a, int n, int inv) {
    // 位逆序置换
    for (int i = 0; i < n; ++i)
        if (i < rev[i]) swap(a[i], a[rev[i]]);

    for (int len = 2; len <= n; len <<= 1) {
        double ang = 2 * PI / len * inv;
        Complex wn(cos(ang), sin(ang));
        for (int i = 0; i < n; i += len) {
            Complex w(1, 0);
            for (int j = 0; j < len / 2; ++j) {
                Complex u = a[i+j], v = w * a[i+j+len/2];
                a[i+j] = u + v;
                a[i+j+len/2] = u - v;
                w = w * wn;
            }
        }
    }
    if (inv == -1)
        for (int i = 0; i < n; ++i) a[i].x /= n;
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);
    int len = 1, bit = 0;
    while (len <= n + m) { len <<= 1; ++bit; }
    for (int i = 0; i < len; ++i)
        rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (bit - 1));

    vector<Complex> a(len), b(len);
    for (int i = 0; i <= n; ++i) scanf("%lf", &a[i].x);
    for (int i = 0; i <= m; ++i) scanf("%lf", &b[i].x);

    fft(a.data(), len, 1);
    fft(b.data(), len, 1);
    for (int i = 0; i < len; ++i) a[i] = a[i] * b[i];
    fft(a.data(), len, -1);

    for (int i = 0; i <= n + m; ++i)
        printf("%d ", (int)(a[i].x + 0.5));
    return 0;
}
```

### 5.2 三次变两次优化

把 $B$ 塞进 $A$ 的虚部，一次 FFT 即可。

---

## 6. NTT（数论变换）

FFT 用复数有精度问题。**NTT** 用模意义下的单位根，全是整数运算，无精度误差。

**需要的模数**：$p = c \cdot 2^k + 1$，且有原根 $g$。

**常用**：

| 模数 | 原根 |
|---|---|
| $998244353 = 119 \times 2^{23} + 1$ | 3 |
| $1004535809 = 479 \times 2^{21} + 1$ | 3 |
| $469762049 = 7 \times 2^{26} + 1$ | 3 |

```cpp
const int MOD = 998244353, G = 3;

int qpow(long long a, long long b) {
    long long r = 1;
    while (b) { if (b & 1) r = r * a % MOD; a = a * a % MOD; b >>= 1; }
    return r;
}

void ntt(int* a, int n, int type) {
    for (int i = 0; i < n; ++i)
        if (i < rev[i]) swap(a[i], a[rev[i]]);
    for (int len = 2; len <= n; len <<= 1) {
        int wn = qpow(G, (MOD - 1) / len);
        if (type == -1) wn = qpow(wn, MOD - 2);
        for (int i = 0; i < n; i += len) {
            long long w = 1;
            for (int j = 0; j < len / 2; ++j) {
                int u = a[i+j], v = w * a[i+j+len/2] % MOD;
                a[i+j] = (u + v) % MOD;
                a[i+j+len/2] = (u - v + MOD) % MOD;
                w = w * wn % MOD;
            }
        }
    }
    if (type == -1) {
        int invn = qpow(n, MOD - 2);
        for (int i = 0; i < n; ++i) a[i] = (long long)a[i] * invn % MOD;
    }
}
```

---

## 7. 经典例题

### 7.1 【模板】多项式乘法

**题意**：求两个多项式的乘积。

**做法**：FFT / NTT。

**复杂度**：$O(n \log n)$。

### 7.2 高精度乘法

**题意**：两个大整数相乘。

**做法**：把数字看作多项式系数，FFT 后进位。

### 7.3 卷积

**题意**：求 $c_k = \sum_{i+j=k} a_i b_j$。

**做法**：FFT。

### 7.4 生成函数

**题意**：用多项式乘法求方案数。

**做法**：构造生成函数，FFT 加速。

### 7.5 分治 FFT

**题意**：$f_i = \sum_{j<i} f_j g_{i-j}$，求 $f$。

**做法**：CDQ 分治 + FFT。

---

## 8. 易错点与练习

### 易错点

1. **长度要补到 2 的幂**：且 $\ge n + m$；
2. **位逆序置换**：`rev` 数组的计算；
3. **逆变换要除以 $n$**；
4. **精度**：复数 FFT 的舍入误差，答案取 `(int)(x + 0.5)`；
5. **NTT 模数**：必须是 $c \cdot 2^k + 1$ 形式；
6. **原根**：确认模数的原根；
7. **数组大小**：$10^6$ 级别的多项式要开到 $2^{21}$。

### 练习建议

1. FFT 模板（多项式乘法）；
2. NTT 模板；
3. 高精度乘法；
4. 生成函数 + FFT；
5. 分治 FFT（CDQ）。
