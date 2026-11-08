# 矩阵优化 DP

> 一句话概括：当 DP 的转移是**线性递推**（新状态是旧状态的线性组合）且系数**不随时间改变**时，把一次转移写成矩阵乘法，用**矩阵快速幂**把 $O(n)$ 递推加速到 $O(k^3\log n)$。

## 目录

1. [概念与原理](#一概念与原理)：线性递推、友矩阵、矩阵快速幂、常数项扩展、广义矩阵乘法
2. [模板代码](#二模板代码)：矩阵类与快速幂
3. [经典例题](#三经典例题)：斐波那契、爬楼梯、广义斐波那契、图上路径计数
4. [易错点](#四易错点)
5. [练习建议](#五练习建议)

---

## 一、概念与原理

### 1.1 线性递推与矩阵

斐波那契数列 $f_1=f_2=1,\ f_n=f_{n-1}+f_{n-2}$，朴素递推 $O(n)$，$n=10^{18}$ 时不可接受。观察 $f_n$ 只由前两项决定，把"前两项"打包成向量

$$
\mathbf{v}_n=\begin{pmatrix}f_n\\ f_{n-1}\end{pmatrix},
\qquad
\mathbf{v}_n=\begin{pmatrix}1&1\\ 1&0\end{pmatrix}
\begin{pmatrix}f_{n-1}\\ f_{n-2}\end{pmatrix}
=M\,\mathbf{v}_{n-1}.
$$

于是 $\mathbf{v}_n=M^{n-2}\mathbf{v}_2$。矩阵乘法满足结合律，可用**快速幂**在 $O(k^3\log n)$ 内求出 $M^{n-2}$。

### 1.2 常系数线性递推的矩阵构造

设 $f_n=a_1f_{n-1}+a_2f_{n-2}+\cdots+a_kf_{n-k}$，构造 $k$ 阶转移矩阵：

$$
\begin{pmatrix}
f_n\\ f_{n-1}\\ \vdots\\ f_{n-k+1}
\end{pmatrix}
=
\begin{pmatrix}
a_1&a_2&\cdots&a_k\\
1&0&\cdots&0\\
0&1&\cdots&0\\
\vdots&&\ddots&\vdots\\
0&\cdots&1&0
\end{pmatrix}
\begin{pmatrix}
f_{n-1}\\ f_{n-2}\\ \vdots\\ f_{n-k}
\end{pmatrix}.
$$

第一行填递推系数，下面是**移位**的单位矩阵，每一行把上一行的旧值往下挪一格。这个矩阵叫**友矩阵（companion matrix）**。以 $f_n=2f_{n-1}+3f_{n-2}+4f_{n-3}$ 为例：

$$
\begin{pmatrix}f_n\\f_{n-1}\\f_{n-2}\end{pmatrix}
=\begin{pmatrix}2&3&4\\1&0&0\\0&1&0\end{pmatrix}
\begin{pmatrix}f_{n-1}\\f_{n-2}\\f_{n-3}\end{pmatrix}.
$$

```
   ┌───────────────────────┐
   │  a1  a2  a3  ...  ak  │  ← 第一行：递推系数
   │   1   0   0  ...   0  │  ← 移位
   │   0   1   0  ...   0  │
   │   ...                 │
   │   0   0  ...  1    0  │
   └───────────────────────┘
             M
```

### 1.3 矩阵快速幂

按指数二进制拆分：

$$
M^n=\begin{cases}(M^{n/2})^2,&n\text{ 偶},\\[2pt](M^{\lfloor n/2\rfloor})^2\cdot M,&n\text{ 奇}.\end{cases}
$$

迭代实现：

```cpp
Matrix power(Matrix base, long long exp) {
    Matrix res = identity(base.n);       // 单位矩阵
    while (exp > 0) {
        if (exp & 1) res = res * base;
        base = base * base;
        exp >>= 1;
    }
    return res;
}
```

复杂度 $O(k^3\log n)$。

### 1.4 带常数项与带参数的转移

递推带常数项时，**增加一个恒为 1 的分量**：

$$
f_n=f_{n-1}+f_{n-2}+c
\quad\Longrightarrow\quad
\begin{pmatrix}f_n\\f_{n-1}\\1\end{pmatrix}
=\begin{pmatrix}1&1&c\\1&0&0\\0&0&1\end{pmatrix}
\begin{pmatrix}f_{n-1}\\f_{n-2}\\1\end{pmatrix}.
$$

常见扩展分量：

| 需要 | 增加的状态 |
|---|---|
| 常数 $c$ | 常数 $1$，矩阵对应列填 $c$ |
| 线性项 $n$ | 状态 $n$，用 $n+1=n+1$ 递推 |
| 组合数 $n^k$ | 二项式展开，增加 $n^0,\dots,n^k$ 各分量 |
| 前缀和 | 增加 $S_n=\sum f_i$ 分量 |

### 1.5 状态向量的设计原则

1. **写标量递推**：$f_n$ 依赖哪些量；
2. **列全必须记住的量**：历史项、常数、参数 $n$、前缀和等；
3. **向量长度即矩阵阶数** $k$；
4. **逐行填写转移**：第 $r$ 行表示"新向量第 $r$ 分量 = 旧向量的线性组合"；
5. **确定初始向量**，乘 $M^{n-\text{start}}$。

```
 标量递推 ──► 状态向量 v_n ──► 转移矩阵 M ──► v_n = M^{n-start} v_start
```

### 1.6 广义矩阵乘法

把矩阵乘法中的"乘"与"加"换成满足**半环**公理的运算，可处理最短路、最大权路径等：

| 问题 | "加法" | "乘法" | 单位元 |
|---|---|---|---|
| 计数/递推 | $+$ | $\times$ | 0 / 1 |
| 最短路（min-plus） | $\min$ | $+$ | $+\infty$ / 0 |
| 最长路（max-plus） | $\max$ | $+$ | $-\infty$ / 0 |
| 布尔可达性 | $\lor$ | $\land$ | false / true |

min-plus 版本：

```cpp
// C[i][j] = min_k (A[i][k] + B[k][j])
for (int i = 0; i < n; ++i)
    for (int j = 0; j < n; ++j) {
        C[i][j] = INF;
        for (int k = 0; k < n; ++k)
            C[i][j] = min(C[i][j], A[i][k] + B[k][j]);
    }
```

用它求 $A^k$ 得到"恰好走 $k$ 条边"的最短路。

---

## 二、模板代码

下面是一个通用小型矩阵类，支持取模、乘法、快速幂，适合 $k\le 100$。`main` 中以斐波那契数列演示用法。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXK = 105;

struct Matrix {
    int n;                       // 阶数
    ll a[MAXK][MAXK];
    static ll MOD;               // 模数，按题目设置

    Matrix(int n = 0, bool identity = false) : n(n) {
        memset(a, 0, sizeof(a));
        if (identity)
            for (int i = 0; i < n; ++i) a[i][i] = 1;
    }

    Matrix operator*(const Matrix &o) const {
        Matrix c(n);
        for (int i = 0; i < n; ++i)
            for (int k = 0; k < n; ++k) {
                if (a[i][k] == 0) continue;        // 稀疏优化
                for (int j = 0; j < n; ++j)
                    c.a[i][j] = (c.a[i][j] + a[i][k] * o.a[k][j]) % MOD;
            }
        return c;
    }
};

ll Matrix::MOD = 1000000007LL;

Matrix mpow(Matrix base, long long exp) {
    Matrix res(base.n, true);    // 单位矩阵
    while (exp > 0) {
        if (exp & 1) res = res * base;
        base = base * base;
        exp >>= 1;
    }
    return res;
}

int main() {
    long long n;
    scanf("%lld", &n);
    if (n <= 2) { puts("1"); return 0; }

    Matrix M(2);
    M.a[0][0] = 1; M.a[0][1] = 1;
    M.a[1][0] = 1; M.a[1][1] = 0;

    Matrix P = mpow(M, n - 2);                 // [f_n, f_{n-1}] = M^(n-2) [1, 1]
    printf("%lld\n", (P.a[0][0] + P.a[0][1]) % Matrix::MOD);
    return 0;
}
```

`MOD` 是静态成员，需在全局定义；多组数据模数不同时改成参数传递。

---

## 三、经典例题

### 例题 1：斐波那契数列第 n 项

**题意**：求 $f_n\bmod 10^9+7$，$f_1=f_2=1$，$f_n=f_{n-1}+f_{n-2}$，$1\le n\le 10^{18}$。

**思路**：$M=\begin{pmatrix}1&1\\1&0\end{pmatrix}$，$\begin{pmatrix}f_n\\f_{n-1}\end{pmatrix}=M^{n-2}\begin{pmatrix}1\\1\end{pmatrix}$。

**复杂度**：$O(\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1000000007LL;

struct Mat {
    long long a[2][2];
    Mat(bool id = false) {
        memset(a, 0, sizeof(a));
        if (id) a[0][0] = a[1][1] = 1;
    }
};

Mat mul(const Mat &x, const Mat &y) {
    Mat z;
    for (int i = 0; i < 2; ++i)
        for (int k = 0; k < 2; ++k)
            for (int j = 0; j < 2; ++j)
                z.a[i][j] = (z.a[i][j] + x.a[i][k] * y.a[k][j]) % MOD;
    return z;
}

Mat qpow(Mat b, long long e) {
    Mat r(true);
    while (e) {
        if (e & 1) r = mul(r, b);
        b = mul(b, b);
        e >>= 1;
    }
    return r;
}

int main() {
    long long n;
    scanf("%lld", &n);
    if (n <= 2) { puts("1"); return 0; }

    Mat m;
    m.a[0][0] = 1; m.a[0][1] = 1;
    m.a[1][0] = 1; m.a[1][1] = 0;

    Mat p = qpow(m, n - 2);
    printf("%lld\n", (p.a[0][0] + p.a[0][1]) % MOD);
    return 0;
}
```

### 例题 2：爬楼梯的加强版

**题意**：$n$ 级台阶，每次上 $1$ 或 $2$ 级，求方案数模 $10^9+7$，$n\le 10^{18}$。

**思路**：$f_i=f_{i-1}+f_{i-2}$，$f_0=f_1=1$，与斐波那契只差平移一位，仍用矩阵快速幂。

**复杂度**：$O(\log n)$。代码同例题 1，仅把幂次改为 $n-1$。

### 例题 3：矩阵加速（广义斐波那契）

**题意**：给定 $a,b,c$ 与 $n$，$f_1=f_2=1$，$f_i=af_{i-1}+bf_{i-2}+c$，求 $f_n\bmod m$。$n\le 2\times 10^9$。

**思路**：常数项 $c$ 需扩展一个恒为 1 的分量：

$$
\begin{pmatrix}f_i\\f_{i-1}\\1\end{pmatrix}
=\begin{pmatrix}a&b&c\\1&0&0\\0&0&1\end{pmatrix}
\begin{pmatrix}f_{i-1}\\f_{i-2}\\1\end{pmatrix}.
$$

答案取 $M^{n-2}$ 作用后的第一分量。

**复杂度**：$O(3^3\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
int MOD;

struct Mat {
    int n;
    ll a[4][4];
    Mat(int n = 0, bool id = false) : n(n) {
        memset(a, 0, sizeof(a));
        if (id) for (int i = 0; i < n; ++i) a[i][i] = 1;
    }
};

Mat mul(const Mat &x, const Mat &y) {
    Mat z(x.n);
    for (int i = 0; i < x.n; ++i)
        for (int k = 0; k < x.n; ++k) {
            if (!x.a[i][k]) continue;
            for (int j = 0; j < x.n; ++j)
                z.a[i][j] = (z.a[i][j] + x.a[i][k] * y.a[k][j]) % MOD;
        }
    return z;
}

Mat qpow(Mat b, long long e) {
    Mat r(b.n, true);
    while (e) {
        if (e & 1) r = mul(r, b);
        b = mul(b, b);
        e >>= 1;
    }
    return r;
}

int main() {
    ll a, b, c, n;
    scanf("%lld%lld%lld%lld%d", &a, &b, &c, &n, &MOD);
    a %= MOD; b %= MOD; c %= MOD;
    if (n <= 2) { puts("1"); return 0; }

    Mat M(3);
    M.a[0][0] = a; M.a[0][1] = b; M.a[0][2] = c;
    M.a[1][0] = 1; M.a[1][1] = 0; M.a[1][2] = 0;
    M.a[2][0] = 0; M.a[2][1] = 0; M.a[2][2] = 1;

    Mat P = qpow(M, n - 2);
    ll ans = (P.a[0][0] + P.a[0][1] + P.a[0][2]) % MOD;   // 初值 [1,1,1]
    printf("%lld\n", ans);
    return 0;
}
```

### 例题 4：图上路径计数

**题意**：$n$ 个点的有向图（$n\le 100$），求从 $1$ 到 $n$ 恰好经过 $k$ 条边的路径数模 $10^9+7$，$k\le 10^{18}$。

**思路**：设邻接矩阵 $A$（$A_{ij}=1$ 表示有一条 $i\to j$ 的边），则 $A^k$ 的 $(i,j)$ 元恰为"从 $i$ 到 $j$ 恰好走 $k$ 条边"的路径数——每乘一次就是枚举一个中间点。

**复杂度**：$O(n^3\log k)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXN = 105;
const ll MOD = 1000000007LL;

struct Mat {
    int n;
    ll a[MAXN][MAXN];
    Mat(int n = 0, bool id = false) : n(n) {
        memset(a, 0, sizeof(a));
        if (id) for (int i = 0; i < n; ++i) a[i][i] = 1;
    }
};

Mat mul(const Mat &x, const Mat &y) {
    Mat z(x.n);
    for (int i = 0; i < x.n; ++i)
        for (int k = 0; k < x.n; ++k) {
            if (!x.a[i][k]) continue;
            for (int j = 0; j < x.n; ++j)
                z.a[i][j] = (z.a[i][j] + x.a[i][k] * y.a[k][j]) % MOD;
        }
    return z;
}

Mat qpow(Mat b, long long e) {
    Mat r(b.n, true);
    while (e) {
        if (e & 1) r = mul(r, b);
        b = mul(b, b);
        e >>= 1;
    }
    return r;
}

int main() {
    int n, m;
    ll k;
    scanf("%d%d%lld", &n, &m, &k);
    Mat A(n);
    for (int i = 0; i < m; ++i) {
        int u, v;
        scanf("%d%d", &u, &v);
        --u; --v;
        A.a[u][v] = (A.a[u][v] + 1) % MOD;   // 处理重边
    }
    Mat R = qpow(A, k);
    printf("%lld\n", R.a[0][n - 1]);
    return 0;
}
```

> 求"至多 $k$ 条边"时可把答案矩阵改为 $\sum_{i=0}^{k}A^i$，用分块矩阵 $\begin{pmatrix}A&I\\0&I\end{pmatrix}$ 求等比矩阵和。

---

## 四、易错点

1. **初始向量与幂次错位。** $\mathbf{v}_n=M^{n-\text{start}}\mathbf{v}_{\text{start}}$；斐波那契用 $M^{n-2}$，爬楼梯用 $M^{n-1}$，差一位就全错。
2. **忘记单位矩阵。** 快速幂的结果初始化为单位矩阵，不能是零矩阵。
3. **矩阵乘法顺序。** 不满足交换律，快速幂里必须统一左乘。
4. **取模遗漏或溢出。** $10^9$ 级元素相乘可达 $10^{18}$，要开 `long long` 且每步取模。
5. **常数项没扩展分量。** 出现 $+c$、$+n$、$+n^2$ 时必须把对应量加入状态向量。
6. **向量长度与矩阵阶数不一致。** 先列清状态再定阶数，避免少一行或少一列。
7. **模数为 1。** 所有答案都是 0；非质数不影响矩阵运算，但不要用逆元。
8. **广义矩阵乘法的单位元写错。** min-plus 对角线 $0$、其余 $+\infty$；max-plus 对角线 $0$、其余 $-\infty$。

## 五、练习建议

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1962 斐波那契数列 | 矩阵快速幂模板 |
| 入门 | 洛谷 P1255 数楼梯 | 高精度递推（对比矩阵） |
| 基础 | 洛谷 P1939 【模板】矩阵加速（数列） | 三项递推矩阵构造 |
| 基础 | 洛谷 P1306 斐波那契公约数 | 矩阵快速幂 + 数论 |
| 进阶 | 洛谷 P3758 [TJOI2017] 可乐 | 图上路径计数 + 自环 |
| 进阶 | 洛谷 P4159 [SCOI2009] 迷路 | 边权拆点 + 矩阵快速幂 |
| 进阶 | 洛谷 P3216 [HNOI2011] 数学作业 | 分段矩阵快速幂 |
| 挑战 | 洛谷 P4000 斐波那契数列 | 循环节 + 矩阵快速幂 |
| 挑战 | 洛谷 P4569 [BJWC2011] 禁忌 | AC 自动机 + 矩阵快速幂 |

**刷题方法**：先写标量递推，再列状态向量，最后逐行填矩阵，不要跳步；阶数小（$k\le 5$）时手算验证一次转移；用 $n\le 20$ 的暴力递推对拍验证矩阵构造；注意题目给的模数不一定是 $10^9+7$；广义矩阵乘法适合"恰好走 $k$ 条边的最短路"，注意初始化 $+\infty$。
