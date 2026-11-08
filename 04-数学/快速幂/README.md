# 快速幂（Binary Exponentiation）

> 一句话概括：**快速幂利用指数的二进制拆分，把 $a^n$ 的计算从 $O(n)$ 次乘法降到 $O(\log n)$ 次乘法**，是所有「幂运算 + 取模」问题的标准工具，也是矩阵快速幂、光速幂等技巧的基础。

---

## 目录

1. [从朴素幂到快速幂](#1-从朴素幂到快速幂)
2. [二进制拆分原理与正确性证明](#2-二进制拆分原理与正确性证明)
3. [迭代写法与循环不变式](#3-迭代写法与循环不变式)
4. [递归写法](#4-递归写法)
5. [模板代码](#5-模板代码)
6. [矩阵快速幂](#6-矩阵快速幂)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 从朴素幂到快速幂

计算 $a^n$（$n$ 为非负整数）最直接的做法是连乘 $n$ 次：

$$
a^n = \underbrace{a \times a \times \cdots \times a}_{n \text{ 个 } a}
$$

这样做需要 $n-1$ 次乘法，时间复杂度 $O(n)$。当 $n \le 10^7$ 时还能接受，但 OI 中指数经常高达 $10^9$、$10^{18}$，甚至以十进制字符串形式给出（$10^{100000}$），此时 $O(n)$ 必然超时。

**关键观察**：$\log_2 n$ 增长极其缓慢。

| $n$ | $\lceil \log_2 n \rceil$ |
|---|---|
| $10^3$ | $10$ |
| $10^6$ | $20$ |
| $10^9$ | $30$ |
| $10^{18}$ | $60$ |

即使 $n = 10^{18}$，也只需要约 $60$ 次「平方」操作。这就是快速幂的核心思想：**用平方代替乘法**。

### 1.1 直观例子

计算 $a^{13}$，朴素做法要 $12$ 次乘法。注意到 $13 = 8 + 4 + 1 = (1101)_2$，于是

$$
a^{13} = a^8 \cdot a^4 \cdot a^1
$$

而 $a^2 = (a^1)^2$、$a^4 = (a^2)^2$、$a^8 = (a^4)^2$，不断平方即可得到所有 $a^{2^i}$。因此只需平方 $3$ 次、乘法 $2$ 次，共 $5$ 次运算。

---

## 2. 二进制拆分原理与正确性证明

### 2.1 二进制展开

任意非负整数 $n$ 都能唯一写成

$$
n = \sum_{i=0}^{\lfloor \log_2 n \rfloor} b_i 2^i, \qquad b_i \in \{0, 1\}
$$

其中 $b_i$ 是 $n$ 的二进制第 $i$ 位。由幂的性质 $a^{x+y} = a^x \cdot a^y$，有

$$
a^n = a^{\sum_i b_i 2^i} = \prod_{i:\, b_i = 1} a^{2^i}
$$

### 2.2 正确性证明

**命题**：设 $n = \sum_{i=0}^{k} b_i 2^i$，则迭代算法返回 $a^n \bmod p$。

**证明**（对二进制位数归纳）：

- **基础情形**：$n = 0$，二进制全为 $0$，空乘积约定为 $1$，而 $a^0 = 1$，成立。
- **归纳假设**：设算法对 $\lfloor n/2 \rfloor$ 位以内的指数均正确，即已算出 $a^{\lfloor n/2 \rfloor}$。
- **归纳步骤**：注意到
  $$
  a^n = a^{2\lfloor n/2 \rfloor + (n \bmod 2)} = \left(a^{\lfloor n/2 \rfloor}\right)^2 \cdot a^{n \bmod 2}
  $$
  即「先算一半的幂，再平方；若 $n$ 为奇数再乘一个 $a$」。算法每轮把底数平方、指数右移一位，遇到当前最低位为 $1$ 就把底数乘进答案，正是这个式子。由归纳假设，循环结束时答案恰为 $a^n$。$\square$

### 2.3 复杂度分析

每轮循环指数至少减半（右移一位），共 $\lfloor \log_2 n \rfloor + 1$ 轮，每轮做常数次乘法与取模：

$$
T(n) = O(\log n), \qquad S(n) = O(1)
$$

递归写法的栈深度为 $O(\log n)$。

---

## 3. 迭代写法与循环不变式

迭代版本维护三个量：`base` 表示当前的 $a^{2^i}$，`n` 是剩余未处理的指数（不断右移），`ans` 是已累乘的结果。

**循环不变式**：

$$
\text{ans} \times \text{base}^{n} \;=\; a^{N_0} \pmod p
$$

其中 $N_0$ 是初始指数。

- **初始时**：$\text{ans} = 1$，$\text{base} = a$，$n = N_0$，左边 $= a^{N_0}$，成立。
- **每轮**：若 $n$ 的最低位为 $1$，把 `base` 乘入 `ans`，相当于从 $n$ 中「扣掉」一个 $2^i$，不变式保持；随后 `base` 平方、$n$ 右移一位，因为 $(\text{base}^2)^{n/2} = \text{base}^{n}$，不变式仍保持。
- **结束时**：$n = 0$，$\text{ans} \times \text{base}^0 = \text{ans} = a^{N_0}$，得证。

以 $a^{13}$ 为例，$13 = (1101)_2$，过程示意如下：

```
n = 13 (1101)   ans = 1
  bit=1: ans *= a        -> ans = a
  base = a^2, n = 6 (110)
  bit=0: 跳过
  base = a^4, n = 3 (11)
  bit=1: ans *= a^4      -> ans = a^5
  base = a^8, n = 1 (1)
  bit=1: ans *= a^8      -> ans = a^13
  base = a^16, n = 0
结束，ans = a^13
```

---

## 4. 递归写法

递归版本更贴近数学定义，代码更短，但常数略大：

$$
\text{pow}(a, n) =
\begin{cases}
1, & n = 0 \\
\text{pow}(a, \lfloor n/2 \rfloor)^2, & n \text{ 为偶数} \\
\text{pow}(a, \lfloor n/2 \rfloor)^2 \cdot a, & n \text{ 为奇数}
\end{cases}
$$

递归写法在需要「从下往上构造」时更自然，例如某些矩阵快速幂的分治问题。

---

## 5. 模板代码

### 5.1 整数快速幂（带取模，C++14）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

// 计算 (a^b) % p，时间复杂度 O(log b)；要求 b >= 0, p > 0
ll qpow(ll a, ll b, ll p) {
    ll ans = 1 % p;          // p = 1 时答案应为 0，写成 1 % p 更安全
    a %= p;                  // 防止 a 本身过大导致乘法溢出
    while (b > 0) {
        if (b & 1) {         // 当前最低位为 1，把 base 累乘进答案
            ans = ans * a % p;
        }
        a = a * a % p;       // base 自乘：a^(2^i) -> a^(2^(i+1))
        b >>= 1;             // 指数右移一位
    }
    return ans;
}

int main() {
    ll a, b, p;
    scanf("%lld%lld%lld", &a, &b, &p);
    printf("%lld^%lld mod %lld=%lld\n", a, b, p, qpow(a, b, p));
    return 0;
}
```

**注意**：`ans = 1 % p` 而不是 `ans = 1`。当 $p = 1$ 时，任何数模 $1$ 都是 $0$，写 `1` 会输出错误答案（洛谷 P1226 就有 $p=1$ 的数据）。

### 5.2 龟速乘（乘法取模防溢出）

当模数 $p \approx 10^{18}$ 时，两个小于 $p$ 的数相乘会爆 `long long`。可以把乘法写成类似快速幂的「二进制加法」形式，称为**龟速乘**：

```cpp
typedef long long ll;
typedef __int128 i128;

// 计算 (a * b) % p，把乘法转成加法防止溢出
ll slow_mul(ll a, ll b, ll p) {
    ll ans = 0;
    a %= p;
    while (b > 0) {
        if (b & 1) ans = (ans + a) % p;
        a = (a + a) % p;
        b >>= 1;
    }
    return ans;
}
```

实际比赛中更推荐直接用 `__int128`：`(ll)((i128)a * b % p)`，代码更短且更快。

### 5.3 光速幂提要

当底数与模数固定、询问次数极多时，预处理 $a^0..a^{B-1}$ 与 $a^B,a^{2B},\ldots$（$B=\lfloor\sqrt{\max n}\rfloor$），把 $n=qB+r$ 拆成 $a^{qB}\cdot a^r$，即可 $O(1)$ 单次查询，预处理 $O(\sqrt n)$。

---

## 6. 矩阵快速幂

### 6.1 为什么可以套用快速幂

矩阵乘法满足**结合律** $(A \cdot B) \cdot C = A \cdot (B \cdot C)$，因此 $A^n$ 同样可以二进制拆分。区别只在于：初始答案从数字 $1$ 变成**单位矩阵** $I$（对角线为 $1$，其余为 $0$），因为 $I \cdot A = A$；「乘法」变成 $O(k^3)$ 的矩阵乘法（$k$ 为阶数）。总复杂度 $O(k^3 \log n)$。

### 6.2 模板

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int MAXN = 105;
ll MOD;          // 模数
int n;           // 矩阵阶数

struct Matrix {
    ll a[MAXN][MAXN];
    Matrix() { memset(a, 0, sizeof(a)); }
    static Matrix identity(int n) {          // 返回 n 阶单位矩阵
        Matrix I;
        for (int i = 0; i < n; i++) I.a[i][i] = 1;
        return I;
    }
};

// 矩阵乘法 C = A * B（均为 n 阶）
Matrix mul(const Matrix &A, const Matrix &B) {
    Matrix C;
    for (int i = 0; i < n; i++)
        for (int k = 0; k < n; k++) {
            if (A.a[i][k] == 0) continue;    // 稀疏优化
            for (int j = 0; j < n; j++)
                C.a[i][j] = (C.a[i][j] + A.a[i][k] * B.a[k][j]) % MOD;
        }
    return C;
}

// 矩阵快速幂 A^b
Matrix mpow(Matrix A, ll b) {
    Matrix ans = Matrix::identity(n);
    while (b > 0) {
        if (b & 1) ans = mul(ans, A);
        A = mul(A, A);
        b >>= 1;
    }
    return ans;
}

int main() {
    ll b;
    scanf("%d%lld%lld", &n, &b, &MOD);
    Matrix A;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) {
            scanf("%lld", &A.a[i][j]);
            A.a[i][j] %= MOD;
        }
    Matrix R = mpow(A, b);
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) printf("%lld ", R.a[i][j]);
        puts("");
    }
    return 0;
}
```

### 6.3 用矩阵快速幂加速线性递推

以斐波那契数列为例：

$$
\begin{pmatrix} F_{n+1} \\ F_n \end{pmatrix}
=
\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}
\begin{pmatrix} F_n \\ F_{n-1} \end{pmatrix},
\qquad
\begin{pmatrix} F_{n+1} \\ F_n \end{pmatrix}
=
\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^{n-1}
\begin{pmatrix} F_2 \\ F_1 \end{pmatrix}
$$

因此求 $F_n$ 只需 $O(\log n)$ 次矩阵乘法。更一般地，任何**常系数线性递推**都可以写成矩阵形式并用快速幂加速，这是矩阵快速幂最重要的应用场景。

---

## 7. 经典例题

### 例题 1：洛谷 P1226 【模板】快速幂

**题意**：给定 $a, b, p$，求 $a^b \bmod p$，$a, b, p$ 在 `long long` 范围内。

**思路**：直接套用 5.1 的二进制快速幂模板（代码同上，此处略）。

**复杂度**：$O(\log b)$。

**注意**：输出格式必须与题面完全一致（包括空格、换行和 `mod` 字样）。

---

### 例题 2：洛谷 P1962 斐波那契数列

**题意**：求 $F_n \bmod 10^9+7$，其中 $F_1 = F_2 = 1$，$F_{n} = F_{n-1} + F_{n-2}$，$n \le 2 \times 10^9$。

**思路**：构造转移矩阵 $\begin{pmatrix}1&1\\1&0\end{pmatrix}^{n-1}$，答案取结果的左上角元素。

**复杂度**：$O(2^3 \log n) = O(\log n)$。

**参考代码**：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll MOD = 1e9 + 7;

struct Mat {
    ll a[2][2];
    Mat() { memset(a, 0, sizeof(a)); }
};

Mat mul(Mat x, Mat y) {
    Mat z;
    for (int i = 0; i < 2; i++)
        for (int k = 0; k < 2; k++) {
            if (!x.a[i][k]) continue;
            for (int j = 0; j < 2; j++)
                z.a[i][j] = (z.a[i][j] + x.a[i][k] * y.a[k][j]) % MOD;
        }
    return z;
}

int main() {
    ll n;
    scanf("%lld", &n);
    if (n <= 2) { puts("1"); return 0; }
    Mat base, ans;
    base.a[0][0] = base.a[0][1] = base.a[1][0] = 1;  // [[1,1],[1,0]]
    ans.a[0][0] = ans.a[1][1] = 1;                   // 单位矩阵
    ll e = n - 1;
    while (e > 0) {
        if (e & 1) ans = mul(ans, base);
        base = mul(base, base);
        e >>= 1;
    }
    printf("%lld\n", ans.a[0][0]);
    return 0;
}
```

---

### 例题 3：洛谷 P1939 【模板】矩阵加速（数列）

**题意**：$a_1 = a_2 = a_3 = 1$，$a_x = a_{x-1} + a_{x-3}$（$x > 3$），求 $a_n \bmod 10^9+7$，$n \le 2 \times 10^9$。

**思路**：状态取三维 $(a_x, a_{x-1}, a_{x-2})$，转移矩阵为

$$
\begin{pmatrix}
1 & 0 & 1 \\
1 & 0 & 0 \\
0 & 1 & 0
\end{pmatrix}
$$

因为 $a_x = a_{x-1} \cdot 1 + a_{x-2} \cdot 0 + a_{x-3} \cdot 1$。

**复杂度**：$O(3^3 \log n)$。

---

### 例题 4：指数超级大——扩展欧拉定理降幂

**题意**：求 $a^b \bmod m$，其中 $b$ 以十进制字符串给出，长度可达 $10^5$，$m \le 10^9$。

**思路**：先判断 $b$ 是否小于 $\varphi(m)$；若 $b \ge \varphi(m)$，用扩展欧拉定理降幂：

$$
a^b \equiv a^{\,b \bmod \varphi(m) + \varphi(m)} \pmod m
$$

否则直接快速幂。

**复杂度**：$O(|b| + \log m)$。

**核心代码**：

```cpp
// 读入字符串形式的指数 b，降幂后调用 qpow
ll solve(string b, ll a, ll m) {
    ll phi = euler_phi(m);       // 需要先实现欧拉函数
    ll r = 0;
    bool big = false;            // b 是否 >= phi
    for (char c : b) {
        r = r * 10 + (c - '0');
        if (r >= phi) { big = true; r %= phi; }
    }
    if (!big) return qpow(a, r, m);          // b < phi，直接算
    return qpow(a, r + phi, m);              // 降幂
}
```

---

## 8. 易错点

1. **`ans` 初始化为 `1` 而不是 `1 % p`**：当 $p=1$ 时答案应为 $0$，这是 P1226 的经典坑。
2. **`a` 忘记先取模**：若 $a$ 本身可能大于 $p$，`a * a` 可能溢出；应先 `a %= p`。
3. **指数类型溢出**：`int` 装不下 $10^{18}$，`b` 必须用 `long long`。
4. **矩阵快速幂的单位矩阵写错**：单位矩阵是主对角线为 $1$，不是全 $1$ 矩阵。
5. **矩阵乘法顺序**：矩阵乘法不满足交换律；做幂运算时 `mul(ans, base)` 与 `mul(base, ans)` 恰好等价，但在维护向量的场景必须区分左乘右乘。
6. **模数在运算中途改变**：多模数题目要每个模数单独做一遍。
7. **负指数**：快速幂只能处理非负整数指数；负指数需先求逆元，且要求底数与模数互质。
8. **`b >>= 1` 与 `b /= 2`**：对非负整数二者等价，但 `b` 为负数时右移是实现定义行为，不要依赖。
9. **忘记 `% MOD` 导致中间结果溢出**：`long long` 乘两个 $10^9$ 级别的数接近 $10^{18}$，模数更大时必须用 `__int128` 或龟速乘。

---

## 9. 练习建议

由易到难：

1. **入门**：洛谷 P1226 【模板】快速幂 —— 熟悉模板与输出格式。
2. **入门**：洛谷 P1045 麦森数 —— 高精度 + 快速幂思想（位数用对数估算）。
3. **进阶**：洛谷 P1962 斐波那契数列 —— 矩阵快速幂入门。
4. **进阶**：洛谷 P1939 【模板】矩阵加速（数列）—— 三维转移矩阵。
5. **进阶**：洛谷 P3390 【模板】矩阵快速幂 —— 纯矩阵幂练习。
6. **提高**：洛谷 P5091 【模板】扩展欧拉定理 —— 降幂，需要先学欧拉函数。
7. **提高**：洛谷 P3216 [HNOI2011] 数学作业 —— 矩阵快速幂 + 分段构造。
8. **提高**：洛谷 P4157 [SCOI2006] 整数划分 —— 高精度快速幂。

**学习路径建议**：

- 先彻底理解「二进制拆分 + 平方」这条主线，能手写迭代版本并解释每一行；
- 再扩展到矩阵快速幂，重点练习「如何构造转移矩阵」；
- 最后学习扩展欧拉定理与光速幂等变体。

> 记住一句话：**快速幂的本质不是「算幂」，而是「用结合律把重复计算合并」**。任何满足结合律的运算（矩阵乘法、模意义下的乘法、自定义二元运算）都可以套用同样的框架。
