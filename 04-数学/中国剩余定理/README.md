# 中国剩余定理（Chinese Remainder Theorem, CRT）

> 一句话概括：**中国剩余定理告诉我们，只要各个模数两两互质，一组同余方程就能唯一确定一个模乘积意义下的解；扩展 CRT 则进一步处理模数不互质的情况。**

---

## 目录

1. [问题的提出](#1-问题的提出)
2. [中国剩余定理（模数两两互质）](#2-中国剩余定理模数两两互质)
3. [构造法证明](#3-构造法证明)
4. [扩展中国剩余定理（模数不互质）](#4-扩展中国剩余定理模数不互质)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 问题的提出

### 1.1 孙子算经

中国南北朝时期的《孙子算经》中有一道著名的问题：

> 今有物不知其数，三三数之剩二，五五数之剩三，七七数之剩二，问物几何？

翻译成现代数学语言，就是求 $x$ 满足

$$
\begin{cases}
x \equiv 2 \pmod 3 \\
x \equiv 3 \pmod 5 \\
x \equiv 2 \pmod 7
\end{cases}
$$

答案是 $x \equiv 23 \pmod{105}$。

### 1.2 一般形式

给定 $n$ 个同余方程

$$
\begin{cases}
x \equiv a_1 \pmod{m_1} \\
x \equiv a_2 \pmod{m_2} \\
\quad \vdots \\
x \equiv a_n \pmod{m_n}
\end{cases}
$$

求 $x$ 的最小非负解。

- **普通 CRT**：要求 $m_1, m_2, \ldots, m_n$ **两两互质**；
- **扩展 CRT（exCRT）**：不要求互质，甚至可以处理模数不互质的情形。

---

## 2. 中国剩余定理（模数两两互质）

### 2.1 定理

设 $m_1, m_2, \ldots, m_n$ 两两互质，令

$$
M = \prod_{i=1}^{n} m_i, \qquad M_i = \frac{M}{m_i}
$$

则同余方程组在模 $M$ 意义下有唯一解

$$
x \equiv \sum_{i=1}^{n} a_i M_i \left(M_i^{-1} \bmod m_i\right) \pmod M
$$

其中 $M_i^{-1}$ 表示 $M_i$ 在模 $m_i$ 下的逆元。

### 2.2 关键观察

由于 $m_i$ 两两互质，$\gcd(M_i, m_i) = 1$，所以 $M_i$ 在模 $m_i$ 下的逆元一定存在。

### 2.3 验证

设

$$
x = \sum_{j=1}^{n} a_j M_j \left(M_j^{-1} \bmod m_j\right)
$$

对第 $i$ 个方程验证：

- 当 $j = i$ 时，$M_i \cdot M_i^{-1} \equiv 1 \pmod{m_i}$，贡献 $a_i$；
- 当 $j \ne i$ 时，$m_i \mid M_j$（因为 $M_j = M/m_j$ 含有因子 $m_i$），故 $M_j \equiv 0 \pmod{m_i}$，贡献 $0$。

因此 $x \equiv a_i \pmod{m_i}$，满足每个方程。

### 2.4 唯一性

若 $x, x'$ 都是解，则对每个 $i$ 有 $x \equiv x' \pmod{m_i}$，即 $m_i \mid (x - x')$。由于 $m_i$ 两两互质，$M = \prod m_i \mid (x - x')$，故 $x \equiv x' \pmod M$。

---

## 3. 构造法证明

### 3.1 存在性构造

定理的公式本身就是构造：

1. 计算 $M = \prod m_i$；
2. 对每个 $i$，计算 $M_i = M / m_i$；
3. 用扩展欧几里得求 $t_i = M_i^{-1} \bmod m_i$；
4. 累加 $a_i M_i t_i$，最后对 $M$ 取模。

### 3.2 孙子问题的求解

以 $x \equiv 2 \pmod 3$，$x \equiv 3 \pmod 5$，$x \equiv 2 \pmod 7$ 为例：

$$
M = 3 \times 5 \times 7 = 105
$$

$$
M_1 = 35,\quad M_2 = 21,\quad M_3 = 15
$$

求逆元：

- $35 t_1 \equiv 1 \pmod 3$：$35 \equiv 2$，$2 \times 2 = 4 \equiv 1$，故 $t_1 = 2$；
- $21 t_2 \equiv 1 \pmod 5$：$21 \equiv 1$，故 $t_2 = 1$；
- $15 t_3 \equiv 1 \pmod 7$：$15 \equiv 1$，故 $t_3 = 1$。

于是

$$
x = 2 \times 35 \times 2 + 3 \times 21 \times 1 + 2 \times 15 \times 1 = 140 + 63 + 30 = 233
$$

$$
233 \bmod 105 = 23
$$

答案为 $23$，与史书记载一致。

### 3.3 文字示意

```
        mod 3        mod 5        mod 7
目标:    2            3            2
         ↑            ↑            ↑
M_i:     35           21           15
t_i:     2            1            1
贡献:  2*35*2=140   3*21*1=63   2*15*1=30
        ↓            ↓            ↓
      模3余2       模5余3       模7余2   ← 各自方程被满足
      模5、7余0    模3、7余0    模3、5余0
总计: 233 ≡ 23 (mod 105)
```

---

## 4. 扩展中国剩余定理（模数不互质）

### 4.1 为什么需要扩展

普通 CRT 要求模数两两互质。但实际问题中模数可能不互质，例如

$$
\begin{cases}
x \equiv 1 \pmod 4 \\
x \equiv 3 \pmod 6
\end{cases}
$$

此时不能直接用 CRT（$\gcd(4,6) = 2 \ne 1$）。

### 4.2 两个方程的情形

先考虑两个方程

$$
x \equiv a_1 \pmod{m_1}, \qquad x \equiv a_2 \pmod{m_2}
$$

设 $x = a_1 + m_1 k$，代入第二个方程：

$$
a_1 + m_1 k \equiv a_2 \pmod{m_2}
\Longrightarrow
m_1 k \equiv a_2 - a_1 \pmod{m_2}
$$

这是关于 $k$ 的一次同余方程。设 $g = \gcd(m_1, m_2)$：

- 若 $(a_2 - a_1) \bmod g \ne 0$，**无解**；
- 否则用 exgcd 解出 $k$，得到

$$
x \equiv a_1 + m_1 k \pmod{\operatorname{lcm}(m_1, m_2)}
$$

于是两个方程合并成一个：

$$
x \equiv a' \pmod{m'}, \qquad m' = \operatorname{lcm}(m_1, m_2) = \frac{m_1 m_2}{g}
$$

### 4.3 合并 n 个方程的算法

从第一个方程开始，依次与后面的方程合并：

```
初始: x ≡ a_1 (mod m_1)
合并: x ≡ a_2 (mod m_2) → x ≡ a' (mod lcm(m_1, m_2))
合并: x ≡ a_3 (mod m_3) → x ≡ a'' (mod lcm(...))
...
```

每一步都化简为「两个方程合并」，最终得到唯一的同余式。

### 4.4 推导细节

设当前已合并为 $x \equiv a \pmod m$，下一个方程为 $x \equiv b \pmod n$。设 $x = a + mk$ 代入得 $mk \equiv b - a \pmod n$。设 $g = \gcd(m,n)$，$d = b-a$；若 $d \bmod g \ne 0$ 则无解。否则两边同除 $g$，因 $\gcd(m/g,\, n/g) = 1$，可求逆元：

$$
k \equiv \left(\frac{m}{g}\right)^{-1} \frac{d}{g} \pmod{\frac{n}{g}}
$$

取最小非负的 $k$，新的同余式为 $x \equiv a + mk \pmod{\operatorname{lcm}(m,n)}$。

**注意溢出**：$\operatorname{lcm}(m,n) = mn/g$ 可能极大，若题目只要求对某个大数取模可在计算中取模；若要精确值需用 `__int128` 甚至高精度。

### 4.5 无解判定

两方程合并时，若 $(b - a) \bmod g \ne 0$，整个方程组无解——这是 exCRT 最重要的检查点。**例**：$x \equiv 1 \pmod 4$，$x \equiv 2 \pmod 6$，$g=2$，$b-a=1$，$1 \bmod 2 \ne 0$，无解。

---

## 5. 模板代码

### 5.1 普通 CRT（模数两两互质，C++14）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

// 扩展欧几里得
ll exgcd(ll a, ll b, ll &x, ll &y) {
    if (!b) { x = 1; y = 0; return a; }
    ll g = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return g;
}

// 普通 CRT：m[i] 两两互质，求 x ≡ a[i] (mod m[i])
// 返回最小非负解，模数为所有 m[i] 的乘积
ll crt(int n, ll a[], ll m[]) {
    ll M = 1;
    for (int i = 0; i < n; i++) M *= m[i];
    ll ans = 0;
    for (int i = 0; i < n; i++) {
        ll Mi = M / m[i];
        ll x, y;
        exgcd(Mi, m[i], x, y);            // 求 Mi 在模 m[i] 下的逆元
        x = (x % m[i] + m[i]) % m[i];
        ans = (ans + (__int128)a[i] * Mi % M * x) % M;
    }
    return ans;
}

int main() {
    ll a[] = {2, 3, 2};
    ll m[] = {3, 5, 7};
    printf("%lld\n", crt(3, a, m));       // 输出 23
    return 0;
}
```

### 5.2 扩展 CRT（模数不要求互质，C++14）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll exgcd(ll a, ll b, ll &x, ll &y) {
    if (!b) { x = 1; y = 0; return a; }
    ll g = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return g;
}

// 扩展 CRT：合并 x ≡ a (mod m) 与 x ≡ b (mod n)
// 返回合并后的模数 lcm(m,n)，解存于 a；无解返回 -1
ll merge(ll &a, ll &m, ll b, ll n) {
    ll x, y;
    ll g = exgcd(m, n, x, y);              // g = gcd(m, n)
    ll d = b - a;
    if (d % g != 0) return -1;             // 无解
    ll mod = n / g;
    x = (__int128)x * (d / g) % mod;       // 特解 k
    x = (x % mod + mod) % mod;
    a = a + m * x;                         // 新的余数
    m = m / g * n;                         // 新的模数 lcm(m,n)
    a = (a % m + m) % m;
    return m;
}

int main() {
    int n;
    scanf("%d", &n);
    ll a, m;
    scanf("%lld%lld", &a, &m);             // 第一个方程
    for (int i = 1; i < n; i++) {
        ll b, n2;
        scanf("%lld%lld", &b, &n2);
        if (merge(a, m, b, n2) == -1) { puts("No solution"); return 0; }
    }
    printf("%lld\n", a);
    return 0;
}
```

### 5.3 数据范围与溢出处理

- 当 $m_i \le 10^9$ 时 $M$ 可达 $10^{9n}$，**必须**用高精度或题目指定的模数；
- 当 $M \le 10^{18}$ 时，用 `long long` 加 `__int128` 中间运算即可；
- `merge` 中 `m / g * n` 应先除后乘，避免溢出；
- `a + m * x` 可能溢出，必要时用 `__int128`。

---

## 6. 经典例题

### 例题 1：洛谷 P1495 【模板】中国剩余定理（CRT）

**题意**：给定 $n$ 个方程 $x \equiv a_i \pmod{m_i}$，$m_i$ 两两互质，求最小非负解。

**思路**：直接套用 5.1 的 CRT 模板。

**复杂度**：$O(n \log M)$。

---

### 例题 2：洛谷 P4777 【模板】扩展中国剩余定理（exCRT）

**题意**：给定 $n$ 个方程 $x \equiv a_i \pmod{m_i}$，$m_i$ **不一定互质**，求最小非负解。

**思路**：用 5.2 的合并算法逐个合并，遇到无解直接输出。

**复杂度**：$O(n \log M)$。

**注意**：

- 输入数据可能很大（$m_i \le 10^{12}$），中间乘法要用 `__int128`；
- 必须处理无解的情况；
- `a` 需要先对 $m$ 取模。

---

### 例题 3：洛谷 P3868 [TJOI2009] 猜数字

**题意**：给定 $a_i, b_i$，求最小的非负整数 $n$，使对每个 $i$ 有 $b_i \mid (n - a_i)$，即 $n \equiv a_i \pmod{b_i}$。

**思路**：直接建立同余方程组，用 exCRT 求解。注意 $a_i$ 可能为负，需要先取模。

**复杂度**：$O(n \log M)$。

**核心代码**：

```cpp
for (int i = 0; i < n; i++) a[i] = (a[i] % b[i] + b[i]) % b[i];
ll ans = a[0], mod = b[0];
for (int i = 1; i < n; i++) {
    ll x, y;
    ll g = exgcd(mod, b[i], x, y);
    ll d = a[i] - ans;
    if (d % g) { puts("-1"); return 0; }
    ll t = (__int128)x * (d / g) % (b[i] / g);
    ans += mod * ((t % (b[i] / g) + (b[i] / g)) % (b[i] / g));
    mod = mod / g * b[i];
    ans = (ans % mod + mod) % mod;
}
```

---

### 例题 4：洛谷 P2480 [SDOI2010] 古代猪文

**题意**：求 $g^{\sum_{d \mid n} C_n^d} \bmod 999911659$，$n \le 10^9$。

**思路**：

1. 模数 $999911659$ 为素数，$\varphi = 999911658 = 2 \times 3 \times 4679 \times 35617$；
2. 指数部分需要对 $\varphi$ 取模，但 $\varphi$ 不是素数，不能直接用 Lucas；
3. 分别对 $2, 3, 4679, 35617$ 四个素数模求 $C_n^d$ 的和（Lucas 定理），得到四个同余式；
4. 用 CRT 合并得到指数；
5. 快速幂求答案。

**复杂度**：$O(\sqrt n \cdot \log n)$ 左右。

这题是**费马小定理 + Lucas + CRT**的经典综合题。

---

## 7. 易错点

1. **普通 CRT 忘记检查互质**：模数不互质时必须用 exCRT，否则逆元不存在，结果错误。
2. **逆元取模为负**：`x = (x % m[i] + m[i]) % m[i]`，不能直接 `x % m[i]`。
3. **乘法溢出**：$M$ 可能很大，`a[i] * Mi` 会爆 `long long`，必须用 `__int128`。
4. **exCRT 忘记判无解**：$(b-a) \bmod g \ne 0$ 时必须输出无解，不能强行算。
5. **`a` 未取模**：合并过程中 `a` 可能变成负数或超过 `m`，每次都要规范到 $[0, m)$。
6. **lcm 溢出**：`m / g * n` 仍可能溢出，看题目是否有取模要求。
7. **`n = 1` 的边界**：只有一个方程时答案就是 $a_1 \bmod m_1$，不要漏特判。
8. **模数为 $1$**：$\bmod 1$ 时所有数都是 $0$，逆元无意义，需特判。
9. **答案从 $0$ 开始还是从最小正整数开始**：题目要求不同，注意 $x=0$ 是否合法。

---

## 8. 练习建议

由易到难：

1. **入门**：洛谷 P1495 【模板】中国剩余定理（CRT）—— 熟悉构造公式。
2. **入门**：洛谷 P4777 【模板】扩展中国剩余定理（exCRT）—— 掌握合并算法。
3. **基础**：洛谷 P3868 [TJOI2009] 猜数字 —— 建模为同余方程组。
4. **基础**：洛谷 P2480 [SDOI2010] 古代猪文 —— CRT + Lucas 综合。
5. **进阶**：洛谷 P4609 [FJOI2016] 建筑师 —— 组合数学 + CRT。
6. **进阶**：洛谷 P4774 [NOI2018] 屠龙勇士 —— exCRT 与扩展欧拉定理结合。
7. **提高**：洛谷 P5350 序列 —— 复杂数据结构 + 模运算。

**学习路径建议**：

- 先用「孙子算经」的例子手推一遍 CRT 公式，理解 $M_i$ 与逆元的作用；
- 再学习 exCRT 的两两合并，重点掌握「$m_1 k \equiv a_2 - a_1 \pmod{m_2}$」这一步；
- 最后刷综合题，把 CRT 与 Lucas、费马小定理、快速幂组合起来。

> 一句话总结：**CRT 的本质是「用互相独立的模数拼出一个大模数下的唯一解」**；互质时直接构造，不互质时逐个合并并检查整除条件。
