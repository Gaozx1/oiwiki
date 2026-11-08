# 同余与逆元（Congruence and Modular Inverse）

> 一句话概括：**同余是整数世界里的「等式」，逆元则是模意义下的「除法」；掌握同余的运算规则、费马小定理与扩展欧几里得求逆元，才能真正安全地在模意义下做乘除运算。**

---

## 目录

1. [同余的定义与性质](#1-同余的定义与性质)
2. [同余方程与一次同余方程](#2-同余方程与一次同余方程)
3. [费马小定理与欧拉定理](#3-费马小定理与欧拉定理)
4. [逆元的概念](#4-逆元的概念)
5. [求逆元的四种方法](#5-求逆元的四种方法)
6. [模板代码](#6-模板代码)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 同余的定义与性质

### 1.1 定义

设 $m$ 为正整数。若 $m \mid (a - b)$，则称 $a$ 与 $b$ **模 $m$ 同余**，记作

$$
a \equiv b \pmod m
$$

否则记作 $a \not\equiv b \pmod m$。

等价地：$a$ 与 $b$ 除以 $m$ 的余数相同。在 C++ 中就是 `(a % m + m) % m == (b % m + m) % m`（注意负数）。

### 1.2 同余的运算性质

设 $a \equiv b \pmod m$，$c \equiv d \pmod m$，则：

1. **加减**：$a \pm c \equiv b \pm d \pmod m$；
2. **乘法**：$ac \equiv bd \pmod m$；
3. **幂**：$a^k \equiv b^k \pmod m$（$k \ge 0$）；
4. **传递**：若 $a \equiv b \pmod m$ 且 $b \equiv c \pmod m$，则 $a \equiv c \pmod m$。

**证明（乘法）**：由 $a - b = m s$，$c - d = m t$，有

$$
ac - bd = ac - bc + bc - bd = c(a-b) + b(c-d) = m(cs + bt)
$$

故 $m \mid (ac - bd)$，即 $ac \equiv bd \pmod m$。$\square$

### 1.3 除法不成立

**重要**：同余式两边**不能随便除以一个数**。例如

$$
6 \equiv 12 \pmod 6, \quad \text{但 } 2 \not\equiv 4 \pmod 6
$$

即两边同除 $3$ 后不成立。只有当除数与模数**互质**时才可以除：

$$
ac \equiv bc \pmod m \ \text{且}\ \gcd(c, m) = 1 \ \Longrightarrow\ a \equiv b \pmod m
$$

**证明**：$m \mid c(a-b)$，由 $\gcd(c,m)=1$ 及 Euclid 引理 $m \mid (a-b)$。$\square$

这就是「逆元」存在的意义：**把除法变成乘以逆元**。

### 1.4 同余类与完全剩余系

模 $m$ 意义下所有整数被分成 $m$ 个同余类，从每类取一个代表组成**完全剩余系**，常用 $\{0,1,\ldots,m-1\}$。

---

## 2. 同余方程与一次同余方程

### 2.1 同余方程

形如 $ax \equiv b \pmod m$ 的方程称为**一次同余方程**。

**定理**：$ax \equiv b \pmod m$ 有解当且仅当 $\gcd(a, m) \mid b$。

**证明**：

- **必要性**：若存在 $x$ 使 $ax \equiv b$，则 $m \mid (ax - b)$。设 $d = \gcd(a,m)$，则 $d \mid m$ 且 $d \mid a$，故 $d \mid ax$，从而 $d \mid b$。
- **充分性**：设 $d = \gcd(a,m) \mid b$，则方程等价于 $\frac{a}{d}x \equiv \frac{b}{d} \pmod{\frac{m}{d}}$。此时 $\gcd\!\left(\frac{a}{d}, \frac{m}{d}\right) = 1$，故 $\frac{a}{d}$ 在模 $\frac{m}{d}$ 下有逆元，$x \equiv \left(\frac{a}{d}\right)^{-1} \frac{b}{d} \pmod{\frac{m}{d}}$，必有解。$\square$

在有解时，方程在模 $m$ 下恰有 $d = \gcd(a,m)$ 个不同的解，它们相差 $\frac{m}{d}$ 的倍数。

### 2.2 用扩展欧几里得求解

$ax \equiv b \pmod m$ 等价于存在整数 $y$ 使

$$
ax + my = b
$$

先用 exgcd 解 $ax + my = \gcd(a,b)$，若 $\gcd \mid b$ 则两边同乘 $\frac{b}{\gcd}$ 得到一组特解，再对所有解取模。

```cpp
// 解 a*x ≡ b (mod m)，返回最小非负解；无解返回 -1
long long solve_congruence(long long a, long long b, long long m) {
    long long x, y;
    long long g = exgcd(a, m, x, y);
    if (b % g != 0) return -1;              // 无解
    x = (__int128)x * (b / g) % (m / g);    // 特解
    x = (x % (m / g) + (m / g)) % (m / g);  // 最小非负
    return x;
}
```

---

## 3. 费马小定理与欧拉定理

### 3.1 费马小定理

**定理**：若 $p$ 为素数，$a$ 为整数且 $p \nmid a$，则

$$
a^{p-1} \equiv 1 \pmod p
$$

**证明**：考虑 $a, 2a, 3a, \ldots, (p-1)a$ 这 $p-1$ 个数。它们模 $p$ 两两不同余：若 $ia \equiv ja \pmod p$，则 $p \mid a(i-j)$，由 $p \nmid a$ 得 $p \mid (i-j)$，而 $|i-j| < p$，故 $i = j$。又因为没有一个是 $p$ 的倍数，所以它们恰好是 $1, 2, \ldots, p-1$ 的一个排列。于是

$$
\prod_{i=1}^{p-1}(ia) \equiv \prod_{i=1}^{p-1} i \pmod p
\Longrightarrow
a^{p-1}(p-1)! \equiv (p-1)! \pmod p
$$

又 $\gcd((p-1)!, p) = 1$，两边约去得 $a^{p-1} \equiv 1 \pmod p$。$\square$

**推论**：$a^p \equiv a \pmod p$（对任意整数 $a$ 都成立，含 $p \mid a$ 的情形）。

### 3.2 欧拉定理

**定理**：若 $\gcd(a, m) = 1$，则

$$
a^{\varphi(m)} \equiv 1 \pmod m
$$

其中 $\varphi(m)$ 是欧拉函数，表示 $1 \sim m$ 中与 $m$ 互质的数的个数。

**证明**：设 $r_1, r_2, \ldots, r_{\varphi(m)}$ 是模 $m$ 的一个**既约剩余系**（所有与 $m$ 互质的剩余类代表）。由于 $\gcd(a,m)=1$，$ar_1, \ldots, ar_{\varphi(m)}$ 也构成既约剩余系（它们互不相同且都与 $m$ 互质）。于是

$$
\prod (ar_i) \equiv \prod r_i \pmod m
\Longrightarrow
a^{\varphi(m)} \prod r_i \equiv \prod r_i \pmod m
$$

两边约去 $\prod r_i$（与 $m$ 互质，可逆）即得。$\square$

费马小定理是欧拉定理在 $m = p$ 为素数时的特例（此时 $\varphi(p) = p - 1$）。

### 3.3 扩展欧拉定理（降幂）

当 $\gcd(a, m) \ne 1$ 时，欧拉定理不成立，但有以下推广：对 $b \ge \varphi(m)$，

$$
a^b \equiv a^{\,b \bmod \varphi(m) + \varphi(m)} \pmod m
$$

这允许我们在指数极大（如以字符串给出）时先对 $\varphi(m)$ 取模再计算，是处理 $a^b \bmod m$ 类问题的关键。

---

## 4. 逆元的概念

### 4.1 定义

设 $\gcd(a, m) = 1$。若存在整数 $x$ 使

$$
ax \equiv 1 \pmod m
$$

则称 $x$ 为 $a$ 在模 $m$ 下的**乘法逆元**，记作 $a^{-1}$。

### 4.2 存在性与唯一性

**存在性**：$\gcd(a,m)=1$ $\iff$ 存在 $x,y$ 使 $ax + my = 1$（裴蜀定理）$\iff$ $ax \equiv 1 \pmod m$ 有解。**唯一性**：若 $ax_1 \equiv ax_2 \equiv 1 \pmod m$，则 $a(x_1-x_2) \equiv 0$，由 $\gcd(a,m)=1$ 得 $x_1 \equiv x_2$。故逆元在模 $m$ 意义下唯一，通常取 $[0,m)$ 中的那个。

### 4.3 逆元的用途

- **模意义下的除法**：$\dfrac{b}{a} \equiv b \cdot a^{-1} \pmod m$；
- **组合数取模**：$C_n^k = \dfrac{n!}{k!(n-k)!}$，除法必须转成乘逆元；
- **解同余方程**：$ax \equiv b$ 两边乘 $a^{-1}$。

### 4.4 逆元的运算法则

$(a^{-1})^{-1} \equiv a$，$(ab)^{-1} \equiv a^{-1}b^{-1}$，$(a^k)^{-1} \equiv (a^{-1})^k \pmod m$。因此可以预处理阶乘的逆元来快速算组合数。

---

## 5. 求逆元的四种方法

### 5.1 费马小定理（模数为素数）

若 $p$ 为素数且 $p \nmid a$，由 $a^{p-1} \equiv 1$ 得

$$
a^{-1} \equiv a^{p-2} \pmod p
$$

用快速幂计算，复杂度 $O(\log p)$，即 `inv = qpow(a, p - 2, p)`。代码短、常数小，但只适用于素数模数。

### 5.2 扩展欧几里得（任意模数，只要互质）

解 $ax + my = 1$，得到的 $x$ 就是逆元。复杂度 $O(\log m)$，适用于任意模数（只要 $\gcd(a,m)=1$），常数比费马小定理略大。

### 5.3 线性递推求 1~n 所有逆元（模数为素数）

当需要 $1 \sim n$ 所有数在素数模 $p$ 下的逆元时，有递推式：

$$
\text{inv}[i] = \left(p - \left\lfloor \frac{p}{i} \right\rfloor \cdot \text{inv}[p \bmod i] \bmod p\right) \bmod p
$$

**证明**：设 $p = qi + r$（$q = \lfloor p/i \rfloor$，$r = p \bmod i$），则 $qi + r \equiv 0 \pmod p$。两边乘 $i^{-1} r^{-1}$：

$$
q r^{-1} + i^{-1} \equiv 0 \pmod p
\Longrightarrow
i^{-1} \equiv -q \cdot r^{-1} \pmod p
$$

即 $\text{inv}[i] \equiv -\lfloor p/i\rfloor \cdot \text{inv}[p \bmod i] \pmod p$。由于 $r < i$，递推有定义。$\square$

```cpp
// 预处理 1..n 在模 p（素数）下的逆元，O(n)
void init_inv(int n, long long p) {
    inv[1] = 1;
    for (int i = 2; i <= n; i++)
        inv[i] = (p - p / i) * inv[p % i] % p;
}
```

**注意**：这里 `(p - p / i)` 先算，避免负数取模；`p / i` 必须用 `long long` 或保证 `p` 在 `long long` 范围。

### 5.4 阶乘逆元的预处理

计算组合数时常用：先算 $n!$，再用费马小定理求 $(n!)^{-1}$，然后倒推：

$$
\text{invfact}[i-1] = \text{invfact}[i] \cdot i \pmod p
$$

因为 $(i!)^{-1} \cdot i = ((i-1)!)^{-1}$。这样只需要一次快速幂，$O(n)$ 预处理全部阶乘逆元。

```cpp
// 预处理阶乘与阶乘逆元，模 p 为素数
fact[0] = 1;
for (int i = 1; i <= n; i++) fact[i] = fact[i - 1] * i % p;
invfact[n] = qpow(fact[n], p - 2, p);       // 只需一次快速幂
for (int i = n; i >= 1; i--) invfact[i - 1] = invfact[i] * i % p;
// C(n, k) = fact[n] * invfact[k] % p * invfact[n-k] % p
```

---

## 6. 模板代码

### 6.1 完整可编译示例（C++14）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll qpow(ll a, ll b, ll p) {              // 快速幂
    ll ans = 1 % p;
    a %= p;
    while (b) {
        if (b & 1) ans = ans * a % p;
        a = a * a % p;
        b >>= 1;
    }
    return ans;
}

ll exgcd(ll a, ll b, ll &x, ll &y) {     // ax + by = gcd(a,b)
    if (!b) { x = 1; y = 0; return a; }
    ll g = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return g;
}

ll inv_fermat(ll a, ll p) { return qpow(a, p - 2, p); }   // p 为素数

ll inv_exgcd(ll a, ll m) {               // 任意模数，需 gcd(a,m)=1
    ll x, y;
    exgcd(a, m, x, y);
    return (x % m + m) % m;
}

const int MAXN = 1e7 + 5;
ll inv[MAXN];
void init_inv(int n, ll p) {             // 线性递推，p 为素数
    inv[1] = 1;
    for (int i = 2; i <= n; i++)
        inv[i] = (p - p / i) * inv[p % i] % p;
}

int main() {
    ll a = 3, p = 1000000007;
    printf("inv_fermat(3) = %lld\n", inv_fermat(a, p));
    printf("inv_exgcd(3)  = %lld\n", inv_exgcd(a, p));
    init_inv(10, p);
    for (int i = 1; i <= 10; i++) printf("%lld ", inv[i]);
    puts("");
    printf("check: %lld\n", a * inv_fermat(a, p) % p);   // 应为 1
    return 0;
}
```

### 6.2 阶乘逆元与组合数

按 5.4 的流程实现即可：预处理 `fact[0..n]`，用费马小定理求一次 `invfact[n]`，再倒推全部 `invfact`，组合数 $C(n,k) = \text{fact}[n]\cdot\text{invfact}[k]\cdot\text{invfact}[n-k] \bmod p$。**该做法要求 $n < p$**，否则 $n! \equiv 0$ 无逆元，必须改用 Lucas 定理。

---

## 7. 经典例题

### 例题 1：洛谷 P3811 【模板】乘法逆元

**题意**：给定 $n, p$（$p$ 为素数），求 $1 \sim n$ 每个数在模 $p$ 下的逆元。

**思路**：用 5.3 的线性递推，$O(n)$ 求出全部。

**复杂度**：$O(n)$。

**核心代码**：

```cpp
inv[1] = 1;
for (int i = 2; i <= n; i++)
    inv[i] = (p - p / i) * inv[p % i] % p;
for (int i = 1; i <= n; i++) printf("%lld\n", inv[i]);
```

**注意**：输出量很大，用 `printf` 或 `puts`，不要用 `cout` 不加优化。

---

### 例题 2：洛谷 P1082 同余方程

**题意**：求关于 $x$ 的同余方程 $ax \equiv 1 \pmod b$ 的最小正整数解（保证有解）。

**思路**：即求 $a$ 在模 $b$ 下的逆元，用扩展欧几里得。

**复杂度**：$O(\log b)$。

**核心代码**：

```cpp
ll a, b, x, y;
scanf("%lld%lld", &a, &b);
exgcd(a, b, x, y);
x = (x % b + b) % b;
if (x == 0) x += b;         // 最小正整数解
printf("%lld\n", x);
```

---

### 例题 3：洛谷 P2613 【模板】有理数取余

**题意**：给定 $a, b$，求 $\dfrac{a}{b} \bmod 19260817$。$a, b$ 可能长达 $10^4$ 位。

**思路**：先边读入边取模得到 $a, b \bmod p$；若 $b \equiv 0$，输出 `Angry!`；否则答案为 $a \cdot b^{-1} \bmod p$（$p$ 为素数，用费马小定理）。

**复杂度**：$O(|a| + |b| + \log p)$。

**核心代码**：

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll MOD = 19260817;

ll qpow(ll a, ll b, ll p) {
    ll r = 1 % p;
    a %= p;
    while (b) {
        if (b & 1) r = r * a % p;
        a = a * a % p;
        b >>= 1;
    }
    return r;
}

ll read_mod() {                 // 边读边取模
    string s;
    cin >> s;
    ll r = 0;
    for (char c : s) r = (r * 10 + c - '0') % MOD;
    return r;
}

int main() {
    ll a = read_mod(), b = read_mod();
    if (b % MOD == 0) { puts("Angry!"); return 0; }
    printf("%lld\n", a * qpow(b, MOD - 2, MOD) % MOD);
    return 0;
}
```

---

### 例题 4：洛谷 P2480 [SDOI2010] 古代猪文

**题意**：求 $g^{\sum_{d \mid n} C_n^d} \bmod 999911659$，$n \le 10^9$。

**思路**：模数 $999911659$ 是素数，且 $999911658 = 2 \times 3 \times 4679 \times 35617$。指数对 $\varphi(p)$ 取模（费马小定理），但模数不是素数，需要用 CRT 分别对四个素因子算 Lucas 后合并，最后快速幂。

**复杂度**：$O(\sqrt n \log n)$ 级别。该题综合了**费马小定理、Lucas 定理、CRT**。

---

## 8. 易错点

1. **模数非素数时用费马小定理**：$a^{p-2}$ 只在 $p$ 为**素数**且 $\gcd(a,p)=1$ 时才是逆元；模数合数时必须用 exgcd 或欧拉定理。
2. **忘记判断 $\gcd(a,m)=1$**：不互质时逆元不存在，代码会算出错误结果。
3. **负数取模**：`x % m` 可能为负，必须 `(x % m + m) % m`。
4. **线性递推逆元的数组越界**：`inv[p % i]` 中 $p \bmod i < i$，不会越界，但 `p` 很大时 `p / i * inv[...]` 可能溢出 `long long`，要用 `__int128` 或先取模。
5. **`p = 1` 的特判**：模 $1$ 时所有数都是 $0$，逆元无意义。
6. **阶乘逆元倒推方向写反**：必须从 $n$ 往 $1$ 倒推：`invfact[i-1] = invfact[i] * i % p`。
7. **组合数中 $n \ge p$ 时直接套阶乘逆元**：当 $n \ge p$ 时 $n! \equiv 0$，逆元不存在，必须用 Lucas 定理。
8. **快速幂指数写成 `p - 2` 但 `p` 是 `int`**：`p - 2` 没问题，但若 `p` 是 `long long` 的 $10^{18}$ 级别素数，`p-2` 也在范围内，注意幂运算的类型。
9. **同余方程系数未约分**：$ax \equiv b \pmod m$ 中 $a, b$ 都要先对 $m$ 取模。

---

## 9. 练习建议

由易到难：

1. **入门**：洛谷 P3811 【模板】乘法逆元 —— 线性递推求逆元。
2. **入门**：洛谷 P1082 同余方程 —— 扩展欧几里得求逆元。
3. **基础**：洛谷 P2613 【模板】有理数取余 —— 大数读入 + 费马小定理。
4. **基础**：洛谷 P1144 最短路计数（模意义运算）—— 理解模运算在 DP 中的应用。
5. **进阶**：洛谷 P2480 [SDOI2010] 古代猪图 —— 费马小定理 + Lucas + CRT 综合。
6. **进阶**：洛谷 P3726 [AH2017/HNOI2017] 抛硬币 —— 组合数 + 逆元 + 高精度。
7. **提高**：洛谷 P5170 【模板】类欧几里得算法 —— 模意义下的复杂推导。

**学习路径建议**：

- 先彻底理解「同余是等式的推广」，特别是**除法不能随便做**这一点；
- 然后掌握费马小定理与欧拉定理的证明，理解它们为什么能造出逆元；
- 最后刷组合数取模的题，熟练「阶乘 + 阶乘逆元」的套路。

> 一句话总结：**模意义下没有除法，只有乘逆元**。逆元存在的前提是互质，这是所有数论题目的第一检查点。
