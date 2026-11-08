# 线性代数与高斯消元（Linear Algebra & Gaussian Elimination）

> 高斯消元 = 用初等行变换把矩阵化成**行阶梯形**，从而解线性方程组、求秩、求行列式、求逆矩阵。

---

## 目录

1. [矩阵基础](#1-矩阵基础)
2. [线性方程组](#2-线性方程组)
3. [高斯消元法](#3-高斯消元法)
4. [模板代码](#4-模板代码)
5. [应用](#5-应用)
6. [经典例题](#6-经典例题)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 矩阵基础

### 1.1 定义

$n \times m$ 的矩阵是 $n$ 行 $m$ 列的数表：

$$
A = \begin{pmatrix}
a_{11} & a_{12} & \cdots & a_{1m} \\
a_{21} & a_{22} & \cdots & a_{2m} \\
\vdots & \vdots & \ddots & \vdots \\
a_{n1} & a_{n2} & \cdots & a_{nm}
\end{pmatrix}
$$

### 1.2 常用运算

| 运算 | 说明 |
|---|---|
| 加法 | 对应位置相加 |
| 数乘 | 每个元素乘 $k$ |
| 乘法 | $C_{ij} = \sum_k A_{ik}B_{kj}$ |
| 转置 | $A^T_{ij} = A_{ji}$ |
| 行列式 | $\det(A)$ |

### 1.3 矩阵乘法

```cpp
const int N = 105;
struct Matrix {
    long long a[N][N];
    int n, m;
    Matrix(int n = 0, int m = 0) : n(n), m(m) { memset(a, 0, sizeof a); }
};

Matrix operator*(const Matrix& A, const Matrix& B) {
    Matrix C(A.n, B.m);
    for (int i = 0; i < A.n; ++i)
        for (int k = 0; k < A.m; ++k) {
            if (!A.a[i][k]) continue;        // 稀疏优化
            for (int j = 0; j < B.m; ++j)
                C.a[i][j] = (C.a[i][j] + A.a[i][k] * B.a[k][j]) % MOD;
        }
    return C;
}
```

**复杂度**：$O(n^3)$。

### 1.4 矩阵快速幂

```cpp
Matrix qpow(Matrix a, long long b) {
    Matrix res(a.n, a.n);
    for (int i = 0; i < a.n; ++i) res.a[i][i] = 1;   // 单位矩阵
    while (b) {
        if (b & 1) res = res * a;
        a = a * a;
        b >>= 1;
    }
    return res;
}
```

---

## 2. 线性方程组

### 2.1 形式

$$
\begin{cases}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n = b_1 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n = b_2 \\
\vdots \\
a_{m1}x_1 + a_{m2}x_2 + \cdots + a_{mn}x_n = b_m
\end{cases}
$$

**矩阵形式**：$A\vec x = \vec b$，其中 $A$ 是系数矩阵，$\vec b$ 是常数向量。

**增广矩阵**：$[A \mid \vec b]$。

### 2.2 解的情况

- **唯一解**：$\text{rank}(A) = \text{rank}([A|b]) = n$；
- **无穷多解**：$\text{rank}(A) = \text{rank}([A|b]) < n$；
- **无解**：$\text{rank}(A) < \text{rank}([A|b])$。

---

## 3. 高斯消元法

### 3.1 步骤

1. 从第 1 列开始，找到当前列绝对值最大的行（**列主元**，减小误差），换到当前行；
2. 用该行把下面所有行的这一列消成 0；
3. 处理下一列；
4. 最后得到上三角矩阵，回代求解。

### 3.2 图示

```
原矩阵:          消元后:
[1 2 3 | 6]      [1 2 3 | 6]
[2 5 3 | 5]  →   [0 1 -3 | -7]
[1 0 8 | 8]      [0 0 1 | 1]
```

回代：$x_3 = 1$，$x_2 = -7 + 3 \cdot 1 = -4$，$x_1 = 6 - 2(-4) - 3(1) = 9$。

---

## 4. 模板代码

### 4.1 实数高斯消元

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 505;
const double EPS = 1e-9;

int n;
double a[N][N];   // 增广矩阵

// 返回: 0 = 唯一解, 1 = 无穷多解, 2 = 无解
int gauss() {
    int row = 1;
    for (int col = 1; col <= n && row <= n; ++col) {
        // 找主元
        int pivot = row;
        for (int i = row; i <= n; ++i)
            if (fabs(a[i][col]) > fabs(a[pivot][col])) pivot = i;

        if (fabs(a[pivot][col]) < EPS) continue;   // 该列全 0

        swap(a[row], a[pivot]);

        // 消元
        for (int i = row + 1; i <= n; ++i) {
            double f = a[i][col] / a[row][col];
            for (int j = col; j <= n + 1; ++j)
                a[i][j] -= f * a[row][j];
        }
        ++row;
    }

    // 判断
    if (row <= n) {
        for (int i = row; i <= n; ++i)
            if (fabs(a[i][n+1]) > EPS) return 2;   // 无解
        return 1;                                   // 无穷多解
    }

    // 回代
    for (int i = n; i >= 1; --i) {
        for (int j = i + 1; j <= n; ++j)
            a[i][n+1] -= a[i][j] * a[j][n+1];
        a[i][n+1] /= a[i][i];
    }
    return 0;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= n + 1; ++j)
            scanf("%lf", &a[i][j]);
    int res = gauss();
    if (res == 0) {
        for (int i = 1; i <= n; ++i)
            printf("x%d = %.4f\n", i, a[i][n+1]);
    } else if (res == 1) puts("Infinite solutions");
    else puts("No solution");
    return 0;
}
```

### 4.2 模意义下高斯消元

```cpp
const int MOD = 1000000007;

long long a[N][N];

long long qpow(long long a, long long b) { ... }

int gaussMod(int n) {
    int row = 1;
    for (int col = 1; col <= n && row <= n; ++col) {
        int pivot = -1;
        for (int i = row; i <= n; ++i)
            if (a[i][col]) { pivot = i; break; }
        if (pivot == -1) continue;

        swap(a[row], a[pivot]);
        long long inv = qpow(a[row][col], MOD - 2);

        for (int i = row + 1; i <= n; ++i) {
            long long f = a[i][col] * inv % MOD;
            for (int j = col; j <= n + 1; ++j)
                a[i][j] = (a[i][j] - f * a[row][j] % MOD + MOD) % MOD;
        }
        ++row;
    }
    // 回代同上
}
```

---

## 5. 应用

### 5.1 解线性方程组

如上。

### 5.2 求矩阵的秩

消元后非零行数即为秩。

### 5.3 求行列式

消元后对角线元素之积（注意交换行要变号）。

```cpp
double det(int n) {
    double res = 1;
    for (int i = 1; i <= n; ++i) {
        int pivot = i;
        for (int j = i + 1; j <= n; ++j)
            if (fabs(a[j][i]) > fabs(a[pivot][i])) pivot = j;
        if (fabs(a[pivot][i]) < EPS) return 0;
        if (pivot != i) { swap(a[i], a[pivot]); res = -res; }
        res *= a[i][i];
        for (int j = i + 1; j <= n; ++j) {
            double f = a[j][i] / a[i][i];
            for (int k = i; k <= n; ++k) a[j][k] -= f * a[i][k];
        }
    }
    return res;
}
```

### 5.4 求逆矩阵

把 $[A \mid I]$ 消成 $[I \mid A^{-1}]$。

### 5.5 期望 DP + 高斯消元

状态转移有环时，列方程用高斯消元。

### 5.6 线性基

求异或空间的最大值/第 $k$ 大，可视为高斯消元（异或版）。

---

## 6. 经典例题

### 6.1 【模板】解线性方程组

**题意**：$n$ 元一次方程组，判断解的情况。

**做法**：高斯消元。

### 6.2 行列式

**题意**：求 $n \times n$ 矩阵的行列式。

**做法**：高斯消元变上三角，对角线乘积。

### 6.3 矩阵求逆

**题意**：求 $A^{-1}$。

**做法**：$[A|I] \to [I|A^{-1}]$。

### 6.4 开关问题

**题意**：$n$ 个开关，每个开关影响若干灯，求方案数。

**做法**：异或方程组 + 高斯消元，自由元个数决定 $2^k$。

### 6.5 期望 + 高斯消元

**题意**：随机游走，求期望步数。

**做法**：列方程 + 高斯消元。

---

## 7. 易错点与练习

### 易错点

1. **不选列主元**：误差会急剧放大；
2. **EPS 大小**：`1e-9` 一般够，题目数据大时用 `1e-8`；
3. **判断无解/多解**：消元后看剩余行的常数项；
4. **模意义下**：主元可能为 0，要判断；逆元用快速幂；
5. **交换行**：行列式要变号；
6. **精度**：实数消元用 `double` 或 `long double`。

### 练习建议

1. 实数高斯消元模板；
2. 模意义下高斯消元；
3. 行列式；
4. 矩阵求逆；
5. 开关问题（异或方程组）；
6. 期望 DP + 高斯消元。
