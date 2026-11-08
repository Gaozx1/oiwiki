# 欧几里得算法（Euclidean Algorithm）

> 一句话概括：**欧几里得算法用「辗转相除」在 $O(\log \min(a,b))$ 时间内求出最大公约数，其扩展形式（exgcd）还能同时解出裴榈方程 $ax+by=\gcd(a,b)$ 的一组整数解，是数论中最重要的算法之一。**

---

## 目录

1. [最大公约数与辗转相除](#1-最大公约数与辗转相除)
2. [欧几里得算法的正确性证明](#2-欧几里得算法的正确性证明)
3. [复杂度分析](#3-复杂度分析)
4. [扩展欧几里得算法](#4-扩展欧几里得算法)
5. [裴蜀定理](#5-裴蜀定理)
6. [模板代码](#6-模板代码)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 最大公约数与辗转相除

### 1.1 定义

设 $a, b$ 为不全为零的整数，$a, b$ 的**最大公约数** $\gcd(a, b)$ 是能同时整除 $a$ 和 $b$ 的最大正整数。

特别地：$\gcd(a, 0) = |a|$，$\gcd(0, 0)$ 无定义（代码中通常返回 $0$）。

### 1.2 核心引理

**引理**：对任意整数 $a \ge b > 0$，

$$
\gcd(a, b) = \gcd(b, a \bmod b)
$$

**证明**：设 $a = qb + r$，其中 $r = a \bmod b$，$0 \le r < b$。需要证明 $a, b$ 的公约数集合与 $b, r$ 的公约数集合相同。

- 设 $d \mid a$ 且 $d \mid b$。则 $r = a - qb$，由 $d \mid a$、$d \mid qb$ 得 $d \mid r$。故 $d$ 也是 $b, r$ 的公约数。
- 设 $d \mid b$ 且 $d \mid r$。则 $a = qb + r$，由 $d \mid qb$、$d \mid r$ 得 $d \mid a$。故 $d$ 也是 $a, b$ 的公约数。

两个集合相同，最大元素自然也相同，即 $\gcd(a,b) = \gcd(b,r)$。$\square$

### 1.3 算法流程

不断用「大数模小数」，直到余数为 $0$：

```
gcd(48, 18)
= gcd(18, 48 mod 18 = 12)
= gcd(12, 18 mod 12 = 6)
= gcd(6, 12 mod 6 = 0)
= 6
```

写成递推式：

$$
\gcd(a, b) =
\begin{cases}
a, & b = 0 \\
\gcd(b, a \bmod b), & b \ne 0
\end{cases}
$$

### 1.4 手算示意

以 $\gcd(1071, 462)$ 为例：

| 步骤 | $a$ | $b$ | $a \bmod b$ |
|---|---|---|---|
| 1 | 1071 | 462 | 147 |
| 2 | 462 | 147 | 21 |
| 3 | 147 | 21 | 0 |
| 4 | 21 | 0 | — |

所以 $\gcd(1071, 462) = 21$。

---

## 2. 欧几里得算法的正确性证明

### 2.1 终止性

每轮 $b$ 严格减小：$a \bmod b < b$，且 $b$ 为非负整数，故至多 $b$ 轮后 $b = 0$，算法终止（实际上远少于 $b$ 轮）。

### 2.2 正确性（归纳证明）

**命题**：对任意非负整数 $a, b$（不全为 $0$），算法返回 $\gcd(a,b)$。

**证明**（对 $b$ 归纳）：

- **基础情形**：$b = 0$ 时返回 $a$，而 $\gcd(a, 0) = |a|$（约定 $a \ge 0$），成立。
- **归纳假设**：设对所有 $b' < b$ 的输入算法正确。
- **归纳步骤**：当 $b > 0$ 时，算法递归为 $\gcd(b, a \bmod b)$。由于 $a \bmod b < b$，由归纳假设，递归返回 $\gcd(b, a \bmod b)$。由 1.2 的引理，$\gcd(b, a\bmod b) = \gcd(a,b)$，故算法正确。$\square$

---

## 3. 复杂度分析

### 3.1 最坏情况：斐波那契数列

欧几里得算法的最坏输入是相邻的斐波那契数。设 $F_1 = F_2 = 1$，$F_n = F_{n-1} + F_{n-2}$。

**Lamé 定理**：若 $a > b \ge 1$ 且 $b < F_{n}$，则算法至多进行 $n - 2$ 次除法。

**证明思路**：每次取模后余数至少比前一个余数减少一个「斐波那契层级」。设过程中余数序列为 $r_0 > r_1 > \cdots > r_k = 0$，可证 $r_i \ge F_{k-i+1}$，故 $b \ge F_{k+1}$，即 $k = O(\log b)$。$\square$

更常用的估计：由 $F_n \approx \frac{1}{\sqrt 5}\varphi^n$（$\varphi = \frac{1+\sqrt 5}{2}$），除法次数约为

$$
\log_{\varphi} \min(a,b) \approx 1.44 \log_2 \min(a,b)
$$

即 $O(\log \min(a,b))$。

### 3.2 常数与实际表现

- 对 $10^{18}$ 以内的数，除法次数最多约 $87$ 次；
- 取模运算很快，实际运行几乎瞬时；
- 因此欧几里得算法可以放心地在循环内大量调用。

### 3.3 更相减损术与 Stein 算法

中国古代的**更相减损术**用减法代替取模：$\gcd(a,b) = \gcd(a-b, b)$（$a \ge b$），正确性同样由「公约数集合不变」保证；但最坏情况（如 $a=10^{18}, b=1$）需要 $10^{18}$ 次减法。

**Stein 算法**（二进制 GCD）只用减法和移位：$a,b$ 都偶时 $\gcd(a,b)=2\gcd(a/2,b/2)$；一偶一奇时约去因子 $2$；都奇时 $\gcd(a,b)=\gcd(|a-b|,\min(a,b))$。复杂度同样 $O(\log\max(a,b))$。

---

## 4. 扩展欧几里得算法

### 4.1 问题

给定 $a, b$，求一组整数解 $(x, y)$ 使得

$$
ax + by = \gcd(a, b)
$$

### 4.2 推导

设递归调用 `exgcd(b, a % b)` 已经求出 $(x', y')$ 满足

$$
b x' + (a \bmod b)\, y' = \gcd(a, b)
$$

把 $a \bmod b = a - \lfloor a/b \rfloor b$ 代入：

$$
b x' + \left(a - \left\lfloor \frac{a}{b} \right\rfloor b\right) y' = \gcd(a,b)
$$

整理得

$$
a y' + b\left(x' - \left\lfloor \frac{a}{b} \right\rfloor y'\right) = \gcd(a,b)
$$

与 $ax + by = \gcd(a,b)$ 对比，得到

$$
x = y', \qquad y = x' - \left\lfloor \frac{a}{b} \right\rfloor y'
$$

### 4.3 边界条件

当 $b = 0$ 时，$\gcd(a, 0) = a$，取 $x = 1, y = 0$ 即可满足 $a \cdot 1 + 0 \cdot 0 = a$。

### 4.4 递归代码

```cpp
// 求 ax + by = gcd(a,b) 的一组解，返回 gcd(a,b)
long long exgcd(long long a, long long b, long long &x, long long &y) {
    if (b == 0) {
        x = 1; y = 0;
        return a;
    }
    long long g = exgcd(b, a % b, y, x);   // 注意 x, y 传参顺序互换
    y -= a / b * x;
    return g;
}
```

**关键细节**：递归时 `y, x` 是交换传入的——这样返回后 `x` 已经是 $x'$，`y` 是 $y'$，只需再执行 `y -= a / b * x` 即可。

### 4.5 通解

若 $(x_0, y_0)$ 是一组特解，则所有整数解为

$$
x = x_0 + \frac{b}{g} t, \qquad y = y_0 - \frac{a}{g} t \qquad (t \in \mathbb{Z})
$$

其中 $g = \gcd(a,b)$。这用于求最小正整数解等问题。

---

## 5. 裴蜀定理

### 5.1 定理内容

**裴蜀定理（Bézout's identity）**：对任意整数 $a, b$（不全为 $0$），存在整数 $x, y$ 使

$$
ax + by = \gcd(a, b)
$$

更一般地，$ax + by$ 能表示的最小正整数恰为 $\gcd(a,b)$。

### 5.2 证明

设 $S = \{ax + by \mid x, y \in \mathbb{Z}\}$，取 $S$ 中最小正元素 $d = ax_0 + by_0$。

**第一步：证明 $d \mid a$ 且 $d \mid b$。**

用带余除法写 $a = qd + r$（$0 \le r < d$），则

$$
r = a - qd = a - q(ax_0 + by_0) = a(1 - qx_0) + b(-qy_0) \in S
$$

若 $r > 0$，则 $r$ 是 $S$ 中比 $d$ 更小的正元素，矛盾。故 $r = 0$，即 $d \mid a$。同理 $d \mid b$。

**第二步：证明 $d$ 是最大的。**

设 $c \mid a$ 且 $c \mid b$，则 $c \mid (ax_0 + by_0) = d$，故 $c \le d$。

综上 $d = \gcd(a,b)$。$\square$

### 5.3 推论

1. $ax + by = c$ 有整数解 $\iff$ $\gcd(a,b) \mid c$。
2. $\gcd(a,b) = 1$ $\iff$ 存在 $x, y$ 使 $ax + by = 1$。
3. 若 $a \mid bc$ 且 $\gcd(a,b) = 1$，则 $a \mid c$（Euclid 引理）。

---

## 6. 模板代码

### 6.1 完整可编译示例（C++14）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll gcd_iter(ll a, ll b) {                // 迭代版 gcd，常数小
    while (b) { ll t = a % b; a = b; b = t; }
    return a;
}

// 扩展欧几里得：返回 gcd(a,b)，并求出 ax + by = gcd 的一组解
ll exgcd(ll a, ll b, ll &x, ll &y) {
    if (b == 0) { x = 1; y = 0; return a; }
    ll g = exgcd(b, a % b, y, x);        // 交换 x, y 传入
    y -= a / b * x;
    return g;
}

int main() {
    ll a = 1071, b = 462, x, y;
    printf("gcd(%lld, %lld) = %lld\n", a, b, gcd_iter(a, b));
    ll g = exgcd(a, b, x, y);
    printf("exgcd: %lld * %lld + %lld * %lld = %lld\n", a, x, b, y, a * x + b * y);
    return 0;
}
```

### 6.2 求解不定方程 $ax + by = c$

```cpp
bool solve_linear(ll a, ll b, ll c, ll &x, ll &y) {
    ll g = exgcd(a, b, x, y);
    if (c % g != 0) return false;          // 无解
    ll k = c / g;
    x *= k; y *= k;                        // 一组特解
    // 把 x 调整到最小正整数
    ll dx = b / g;
    x = (x % dx + dx) % dx;
    if (x == 0) x += dx;
    y = (c - a * x) / b;
    return true;
}
```

**注意**：`dx = b / g` 可能是负数，代码假设 $b > 0$；若 $b < 0$，应先取绝对值。

### 6.3 用 exgcd 求逆元

$ax \equiv 1 \pmod m$ 等价于 $ax + my = 1$，用 exgcd 解出 $x$ 后取模即可：

```cpp
ll inv_mod(ll a, ll m) {
    ll x, y;
    exgcd(a, m, x, y);
    return (x % m + m) % m;
}
```

这与「同余与逆元」一节的 `inv_exgcd` 完全一致。

---

## 7. 经典例题

### 例题 1：洛谷 P1082 同余方程

**题意**：求 $ax \equiv 1 \pmod b$ 的最小正整数解（保证有解）。

**思路**：即求 $a$ 在模 $b$ 下的逆元，用 exgcd 解 $ax + by = 1$。

**复杂度**：$O(\log b)$。

**核心代码**：

```cpp
ll x, y;
exgcd(a, b, x, y);
x = (x % b + b) % b;
if (x == 0) x += b;
printf("%lld\n", x);
```

---

### 例题 2：洛谷 P4549 【模板】裴蜀定理

**题意**：给定 $n$ 个整数 $a_1, \ldots, a_n$，求 $S = \sum a_i x_i$（$x_i$ 为整数）的最小正取值。

**思路**：推广的裴蜀定理：$\sum a_i x_i$ 的所有可能取值恰为 $\gcd(|a_1|, \ldots, |a_n|)$ 的倍数，故最小正值为所有 $|a_i|$ 的 gcd。

**复杂度**：$O(n \log A)$。

**核心代码**：

```cpp
ll g = 0;
for (int i = 1; i <= n; i++) {
    ll x; scanf("%lld", &x);
    g = gcd_iter(g, abs(x));
}
printf("%lld\n", g);
```

---

### 例题 3：洛谷 P5656 【模板】二元一次不定方程

**题意**：给定 $a, b, c$，求 $ax + by = c$ 的正整数解个数，以及 $x$ 最小、$y$ 最小的解。

**思路**：先用 exgcd 求出 $ax + by = \gcd(a,b)$ 的一组解，再乘 $c/g$ 得特解，然后利用通解

$$
x = x_0 + \frac{b}{g}t, \qquad y = y_0 - \frac{a}{g}t
$$

推导正整数解区间，求个数与极值。

**复杂度**：$O(\log \max(a,b))$。

**要点**：

- 先判断 $c \bmod g \ne 0$ 则无解；
- 用通解把 $x$ 调到最小正数，再算对应 $y$；
- 注意 $a, b, c$ 可能为负，需先处理符号。

---

### 例题 4：洛谷 P1516 青蛙的约会

**题意**：两只青蛙在长为 $L$ 的环上，初始位置 $x, y$，每次分别跳 $m, n$ 米，求最少跳几次相遇。

**思路**：设跳 $t$ 次后相遇，则

$$
x + mt \equiv y + nt \pmod L
\Longrightarrow
(m - n)t \equiv y - x \pmod L
$$

这是标准的一次同余方程，用 exgcd 求解。

**复杂度**：$O(\log L)$。

**核心代码**：

```cpp
ll a = m - n, b = L, c = y - x;
ll g = exgcd(a, b, x0, y0);
if (c % g != 0) { puts("Impossible"); return 0; }
ll mod = b / g;
x0 = (__int128)x0 * (c / g) % mod;
x0 = (x0 % mod + mod) % mod;
printf("%lld\n", x0);
```

---

## 8. 易错点

1. **`a % b` 的符号**：C++ 中 `a % b` 的符号与被除数相同。若 $a < 0$，`a % b` 可能为负，应先用 `abs` 或调整成非负。
2. **`gcd(0, 0)` 无定义**：代码会返回 $0$，但数学上无意义，需特判。
3. **exgcd 的 `x, y` 传参顺序**：递归调用必须写成 `exgcd(b, a % b, y, x)`（交换），否则结果错误。
4. **exgcd 求逆元未取模**：得到的 $x$ 可能为负，必须 `(x % m + m) % m`。
5. **`a / b * x` 溢出**：`a / b` 可能很大，乘以 `x` 会溢出 `long long`，建议用 `__int128`。
6. **通解中 `b/g` 的符号**：若 $b < 0$，`b/g` 为负，调整最小正整数解时要先取绝对值。
7. **不定方程忘记乘 `c/g`**：exgcd 解出的是 $= \gcd$ 的解，要乘 $c/g$ 才对应 $= c$。
8. **递归深度**：虽然 $O(\log)$，但极端数据（如 $a=1, b=10^{18}$）时深度约 $60$，安全。
9. **`__gcd` 与 `std::gcd`**：`__gcd` 是 GCC 扩展，C++14 无 `std::gcd`（C++17 才有），跨平台建议手写。

---

## 9. 练习建议

由易到难：

1. **入门**：洛谷 P1177 快速排序 —— 熟悉基本 gcd 调用与排序。
2. **入门**：洛谷 P1082 同余方程 —— exgcd 求逆元。
3. **基础**：洛谷 P4549 【模板】裴蜀定理 —— 多数的 gcd 推广。
4. **基础**：洛谷 P5656 【模板】二元一次不定方程 —— exgcd 通解。
5. **进阶**：洛谷 P1516 青蛙的约会 —— 同余方程建模。
6. **进阶**：洛谷 P2421 [NOI2002] 荒岛野人 —— 枚举 + exgcd 判定。
7. **提高**：洛谷 P5170 【模板】类欧几里得算法 —— 欧几里得思想的高级推广。
8. **提高**：洛谷 P3535 [POI2012] TOU-Tour de Byteotia —— 复杂建模。

**学习路径建议**：

- 先能手推 $\gcd(1071, 462)$ 的每一步，并解释为什么模数会严格变小；
- 再理解 exgcd 的递推式 $x = y', y = x' - \lfloor a/b\rfloor y'$ 的推导；
- 最后刷不定方程与同余方程的建模题，把「实际问题 → $ax+by=c$」这一步练熟。

> 一句话总结：**欧几里得算法的本质是「公约数集合在取模下不变」**；扩展形式则利用这一递归结构，反向构造出裴蜀等式的解。
