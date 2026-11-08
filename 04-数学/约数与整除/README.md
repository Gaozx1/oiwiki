# 约数与整除（Divisors and Divisibility）

> 一句话概括：**整除是数论的出发点，约数刻画了整数的内部结构；掌握约数个数、约数之和、最大公约数与整除分块，是解决数论计数与求和问题的基本功。**

---

## 目录

1. [整除的定义与基本性质](#1-整除的定义与基本性质)
2. [约数的计数与求和](#2-约数的计数与求和)
3. [最大公约数与最小公倍数](#3-最大公约数与最小公倍数)
4. [整除分块](#4-整除分块)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 整除的定义与基本性质

### 1.1 定义

设 $a, b \in \mathbb{Z}$ 且 $b \neq 0$。若存在整数 $q$ 使得 $a = bq$，则称 **$b$ 整除 $a$**，记作 $b \mid a$；否则记作 $b \nmid a$。此时称 $b$ 是 $a$ 的**约数**（因数），$a$ 是 $b$ 的**倍数**。

在 C++ 中判断 $b \mid a$ 就是 `a % b == 0`。注意 C++ 的 `%` 对负数结果是**向零取整**的余数，例如 `-7 % 3 == -1`；余数为 $0$ 时整除判断依然成立，但涉及取模的其他运算要格外小心。

### 1.2 基本性质

对任意整数 $a, b, c$：

1. **传递性**：若 $a \mid b$ 且 $b \mid c$，则 $a \mid c$。
2. **线性组合**：若 $a \mid b$ 且 $a \mid c$，则对任意整数 $x, y$ 有 $a \mid (bx + cy)$。
3. **互素约去**：若 $a \mid bc$ 且 $\gcd(a, b) = 1$，则 $a \mid c$。
4. **约数范围**：若 $d \mid n$ 且 $d > 0$，则 $1 \le d \le n$（$n > 0$）。
5. **配对性**：若 $d \mid n$，则 $n/d \mid n$；$d$ 与 $n/d$ 成对出现，且恰有一个 $\le \sqrt n$。

**性质 2 的证明**：因 $a \mid b$，存在 $k_1$ 使 $b = ak_1$；因 $a \mid c$，存在 $k_2$ 使 $c = ak_2$。于是 $bx + cy = a(k_1x + k_2y)$，故 $a \mid (bx+cy)$。$\square$

**性质 5 的说明**：若 $d \mid n$，设 $n = de$，则 $e = n/d$ 也整除 $n$。若 $d < \sqrt n$ 则 $e > \sqrt n$，反之亦然，故约数成对分布在 $\sqrt n$ 两侧（$\sqrt n$ 为整数时单独居中）。

### 1.3 试除法枚举约数

利用配对性，只需枚举 $d \le \sqrt n$：

```cpp
vector<long long> divisors(long long n) {
    vector<long long> res;
    for (long long d = 1; d * d <= n; d++) {
        if (n % d) continue;
        res.push_back(d);
        if (d != n / d) res.push_back(n / d);   // 防止完全平方数重复
    }
    sort(res.begin(), res.end());
    return res;
}
```

复杂度 $O(\sqrt n)$。**易错点**：完全平方数时 $d = n/d$，不能重复加入。

---

## 2. 约数的计数与求和

### 2.1 约数个数公式

设 $n = p_1^{e_1} p_2^{e_2} \cdots p_k^{e_k}$，则任意约数 $d$ 都可写成 $d = p_1^{f_1} \cdots p_k^{f_k}$，其中 $0 \le f_i \le e_i$。由乘法原理，

$$
d(n) = \prod_{i=1}^{k} (e_i + 1)
$$

**推导**：每个 $f_i$ 有 $e_i+1$ 种取法，且不同取法对应的约数不同（唯一分解定理保证），故总数为乘积。

### 2.2 约数之和公式

$$
\sigma(n) = \prod_{i=1}^{k} \left(1 + p_i + p_i^2 + \cdots + p_i^{e_i}\right)
= \prod_{i=1}^{k} \frac{p_i^{e_i+1} - 1}{p_i - 1}
$$

**推导**：把连乘积 $\prod_i (1 + p_i + \cdots + p_i^{e_i})$ 展开，每一项恰好对应一个约数 $p_1^{f_1}\cdots p_k^{f_k}$，故展开式正是所有约数之和。$\square$

### 2.3 一个具体例子

以 $n = 360 = 2^3 \cdot 3^2 \cdot 5$ 为例：

$$
d(360) = (3+1)(2+1)(1+1) = 24, \qquad
\sigma(360) = (1+2+4+8)(1+3+9)(1+5) = 15 \cdot 13 \cdot 6 = 1170
$$

### 2.4 求单个数的约数个数与和

分解质因数后套公式即可，复杂度 $O(\sqrt n)$：

```cpp
// 返回 {约数个数, 约数之和}
pair<long long, long long> divisor_info(long long n) {
    long long cnt = 1, sum = 1;
    for (long long p = 2; p * p <= n; p++) {
        if (n % p) continue;
        long long e = 0, pk = 1, s = 1;   // s = 1 + p + ... + p^e
        while (n % p == 0) { n /= p; e++; }
        for (int i = 1; i <= e; i++) { pk *= p; s += pk; }
        cnt *= (e + 1);
        sum *= s;
    }
    if (n > 1) {                          // 剩下一个大素数
        cnt *= 2;
        sum *= (1 + n);
    }
    return {cnt, sum};
}
```

### 2.5 批量筛法与积性

要求 $1 \sim n$ 每个数的约数个数，可用「倍数筛」：

```cpp
// d[i] 表示 i 的约数个数，O(n log n)
vector<int> d(n + 1, 0);
for (int i = 1; i <= n; i++)
    for (int j = i; j <= n; j += i)
        d[j]++;                      // i 是 j 的约数
```

复杂度 $\sum_{i=1}^n n/i = O(n \log n)$。更快的做法是利用积性：若 $\gcd(a,b)=1$，则 $d(ab) = d(a)d(b)$、$\sigma(ab)=\sigma(a)\sigma(b)$，于是可用线性筛在 $O(n)$ 内递推（见「素数筛法」一节的积性函数部分）。

### 2.6 约数个数的上界

$n \le 10^9$ 时 $d(n) \le 1344$，$n \le 10^{18}$ 时 $d(n) \le 103680$。枚举约数可行，但绝不能把约数个数当作常数。

---

## 3. 最大公约数与最小公倍数

### 3.1 定义

$\gcd(a, b)$ 表示 $a, b$ 的**最大公约数**，$\operatorname{lcm}(a, b)$ 表示**最小公倍数**。

### 3.2 关键恒等式

$$
a \cdot b = \gcd(a, b) \cdot \operatorname{lcm}(a, b)
$$

**证明**：设 $a = \prod p_i^{\alpha_i}$，$b = \prod p_i^{\beta_i}$。则

$$
\gcd(a,b) = \prod p_i^{\min(\alpha_i,\beta_i)}, \qquad
\operatorname{lcm}(a,b) = \prod p_i^{\max(\alpha_i,\beta_i)}
$$

而 $\min(\alpha_i,\beta_i) + \max(\alpha_i,\beta_i) = \alpha_i + \beta_i$，逐项相加即得 $a b = \gcd \cdot \operatorname{lcm}$。$\square$

**推论**：$\operatorname{lcm}(a,b) = \dfrac{ab}{\gcd(a,b)}$。计算时应**先除后乘**，避免溢出：

```cpp
long long lcm(long long a, long long b) {
    return a / gcd(a, b) * b;     // 先除后乘，防止 a*b 溢出
}
```

### 3.3 多个数的 gcd 与 lcm

$$
\gcd(a_1, \ldots, a_n) = \gcd(\gcd(a_1,\ldots,a_{n-1}), a_n)
$$

lcm 同理（同样要注意溢出）。

### 3.4 与约数的关系

- $d \mid a$ 且 $d \mid b$ $\iff$ $d \mid \gcd(a,b)$；
- $a \mid m$ 且 $b \mid m$ $\iff$ $\operatorname{lcm}(a,b) \mid m$。

这两条是处理「同时满足」条件时的基本转换。

---

## 4. 整除分块

### 4.1 核心观察

计算 $\sum_{i=1}^{n} \lfloor n/i \rfloor$ 时，朴素做法是 $O(n)$。但 $\lfloor n/i \rfloor$ 的取值只有 $O(\sqrt n)$ 种：$i \le \sqrt n$ 时至多 $\sqrt n$ 种；$i > \sqrt n$ 时 $\lfloor n/i \rfloor < \sqrt n$，也至多 $\sqrt n$ 种。

### 4.2 一块的右端点

若当前块从 $l$ 开始，值为 $v = \lfloor n/l \rfloor$，则这块的右端点为

$$
r = \left\lfloor \frac{n}{\lfloor n/l \rfloor} \right\rfloor
$$

**证明**：设 $v = \lfloor n/l \rfloor$，则 $v \le n/l < v+1$，即 $l \le n/v$ 且 $l > n/(v+1)$。所有满足 $\lfloor n/x \rfloor = v$ 的 $x$ 的最大值即 $\lfloor n/v \rfloor$。$\square$

### 4.3 代码与复杂度

```cpp
// sum_{i=1}^{n} floor(n/i)
long long sum_floor(long long n) {
    long long ans = 0;
    for (long long l = 1, r; l <= n; l = r + 1) {
        r = n / (n / l);                 // 当前块的右端点
        ans += (r - l + 1) * (n / l);    // 块长 × 块的值
    }
    return ans;
}
```

块数约为 $2\sqrt n$，复杂度 $O(\sqrt n)$。

### 4.4 二维整除分块

求 $\sum_{i=1}^{n} \lfloor n/i \rfloor \lfloor m/i \rfloor$ 时，每一块的右端点为

$$
r = \min\left(\left\lfloor\frac{n}{\lfloor n/l\rfloor}\right\rfloor, \left\lfloor\frac{m}{\lfloor m/l\rfloor}\right\rfloor\right)
$$

因为要同时保证两个商都不变。

---

## 5. 模板代码

### 5.1 约数相关操作（C++14，可直接编译）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

// 返回 n 的所有正约数（升序）
vector<ll> get_divisors(ll n) {
    vector<ll> res;
    for (ll d = 1; d * d <= n; d++) {
        if (n % d) continue;
        res.push_back(d);
        if (d != n / d) res.push_back(n / d);   // 完全平方数不重复
    }
    sort(res.begin(), res.end());
    return res;
}

// 返回 {约数个数, 约数之和}
pair<ll, ll> divisor_info(ll n) {
    ll cnt = 1, sum = 1;
    for (ll p = 2; p * p <= n; p++) {
        if (n % p) continue;
        ll e = 0, pk = 1, s = 1;
        while (n % p == 0) { n /= p; e++; }
        for (ll i = 1; i <= e; i++) { pk *= p; s += pk; }
        cnt *= (e + 1);
        sum *= s;
    }
    if (n > 1) { cnt *= 2; sum *= (1 + n); }
    return {cnt, sum};
}

ll gcd_ll(ll a, ll b) {                 // 欧几里得算法
    while (b) { ll t = a % b; a = b; b = t; }
    return a;
}

ll lcm_ll(ll a, ll b) {                 // 先除后乘防溢出
    return a / gcd_ll(a, b) * b;
}

int main() {
    ll n;
    scanf("%lld", &n);
    auto divs = get_divisors(n);
    printf("divisors:");
    for (ll d : divs) printf(" %lld", d);
    printf("\ncount = %zu\n", divs.size());
    auto info = divisor_info(n);
    printf("d(n) = %lld, sigma(n) = %lld\n", info.first, info.second);
    printf("gcd(12,18) = %lld, lcm(12,18) = %lld\n", gcd_ll(12, 18), lcm_ll(12, 18));
    return 0;
}
```

### 5.2 整除分块模板

模板见 4.3 节；若要求 $\sum i\lfloor n/i \rfloor$，只需把块内贡献改为等差数列求和 $(l+r)(r-l+1)/2 \cdot \lfloor n/l \rfloor$。

---

## 6. 经典例题

### 例题 1：洛谷 P2926 [USACO08DEC] Patting Heads S

**题意**：给定 $n$ 个数 $a_1, \ldots, a_n$（$a_i \le 10^6$），对每个 $a_i$ 求有多少个 $a_j$ 是它的约数（$j \ne i$）。

**思路**：统计每个数值出现次数 `cnt[x]`，再用倍数筛：

$$
\text{ans}[x] = \sum_{d \mid x} \text{cnt}[d]
$$

独立地「对每个 $d$ 把 `cnt[d]` 加到它的所有倍数上」即可。

**复杂度**：$O(A \log A)$，$A = 10^6$。

**核心代码**：

```cpp
const int A = 1e6 + 5;
int cnt[A], ans[A];
for (int i = 1; i < A; i++)
    if (cnt[i])
        for (int j = i; j < A; j += i)
            ans[j] += cnt[i];       // i 是 j 的约数
// 对 a[i] 的答案：ans[a[i]] - 1（减去自己）
```

---

### 例题 2：洛谷 P1403 [AHOI2005] 约数研究

**题意**：求 $\sum_{i=1}^{n} d(i)$，$n \le 10^6$。

**思路**：交换求和顺序：

$$
\sum_{i=1}^{n} d(i) = \sum_{i=1}^{n}\sum_{d \mid i} 1 = \sum_{d=1}^{n} \left\lfloor \frac{n}{d} \right\rfloor
$$

右边用整除分块 $O(\sqrt n)$ 计算。

**复杂度**：$O(\sqrt n)$。

---

### 例题 3：洛谷 P2261 [CQOI2007] 余数求和

**题意**：给定 $n, k$，求

$$
\sum_{i=1}^{n} (k \bmod i), \qquad n, k \le 10^9
$$

**思路**：利用 $k \bmod i = k - i\lfloor k/i \rfloor$，得

$$
\sum_{i=1}^{n}(k \bmod i) = nk - \sum_{i=1}^{n} i\left\lfloor\frac{k}{i}\right\rfloor
$$

后面的和用整除分块：每一块内 $\lfloor k/i \rfloor$ 不变，用等差数列求和。

**复杂度**：$O(\sqrt k)$。

**核心代码**：

```cpp
ll solve(ll n, ll k) {
    ll ans = n * k;
    for (ll l = 1, r; l <= n; l = r + 1) {
        if (k / l == 0) break;               // 后面都是 0，提前结束
        r = min(n, k / (k / l));             // 注意 n 上限
        ans -= (l + r) * (r - l + 1) / 2 * (k / l);
    }
    return ans;
}
```

---

### 例题 4：洛谷 P1463 [POI2002] 反素数

**题意**：求不超过 $n$（$n \le 2 \times 10^9$）的最大反素数。反素数指约数个数严格大于所有比它小的正整数的数。

**思路**：反素数的质因数必然是**从 2 开始的连续素数**，且指数**单调不增**。只需 DFS 枚举 $2^{e_1}3^{e_2}5^{e_3}\cdots$（指数递减），在 $n$ 范围内找约数个数最多的数（相同取最小）。

**复杂度**：搜索树很小，远低于 $O(\sqrt n)$ 枚举。

**核心代码**：

```cpp
int primes[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29};
long long n, best = 0, bestCnt = 0;

void dfs(int idx, long long cur, long long cnt, int maxExp) {
    if (cnt > bestCnt || (cnt == bestCnt && cur < best)) {
        bestCnt = cnt; best = cur;
    }
    if (idx == 10) return;
    long long p = primes[idx];
    for (int e = 1; e <= maxExp; e++) {
        if (cur > n / p) break;                // 防溢出
        cur *= p;
        dfs(idx + 1, cur, cnt * (e + 1), e);   // 指数单调不增
    }
}
// 调用：dfs(0, 1, 1, 31)
```

---

## 7. 易错点

1. **完全平方数重复计数**：枚举约数时 `d == n/d` 只能加一次。
2. **`lcm` 直接写 `a * b / gcd` 溢出**：应先 `a / gcd * b`，或使用 `__int128`。
3. **`gcd(0, x)`**：约定 $\gcd(0, x) = |x|$，欧几里得算法天然支持；$\gcd(0,0)$ 无定义。
4. **负数取模**：C++ 的 `%` 结果为负，涉及负数的整除分块要先转正。
5. **`d * d <= n` 溢出**：$n$ 接近 `long long` 上限时应写成 `d <= n / d`。
6. **约数个数上界误判**：$n \le 10^{18}$ 时最大约数个数为 $103680$，别把它当常数。
7. **整除分块右端点漏写 `min(n, ...)`**：在 $\sum i\lfloor k/i\rfloor$ 类问题中 $i$ 上限是 $n$ 而非 $k$。
8. **`k / l == 0` 时除零**：整除分块要先判断 `k / l == 0`，否则 `k / (k / l)` 除零。
9. **整除分块用 `int`**：$n$ 较大时中间乘积溢出，必须用 `long long`。

---

## 8. 练习建议

由易到难：

1. **入门**：洛谷 P1403 [AHOI2005] 约数研究 —— 交换求和顺序 + 整除分块入门。
2. **入门**：洛谷 P2926 [USACO08DEC] Patting Heads S —— 倍数筛思想。
3. **基础**：洛谷 P2261 [CQOI2007] 余数求和 —— 整除分块经典题。
4. **基础**：洛谷 P1463 [POI2002] 反素数 —— DFS + 约数个数。
5. **进阶**：洛谷 P2424 约数和 —— 整除分块求 $\sum \sigma(i)$。
6. **进阶**：洛谷 P2158 [SDOI2008] 仪仗队 —— 欧拉函数与互质。
7. **提高**：洛谷 P3327 [SDOI2015] 约数个数和 —— 莫比乌斯反演 + 整除分块。

**学习路径建议**：

- 先熟练「试除法枚举约数」和「分解质因数」两个基本操作；
- 再理解 $d(n)$、$\sigma(n)$ 公式的推导，做到能自己写出证明；
- 最后主攻整除分块，它是所有「含 $\lfloor n/i \rfloor$ 求和」问题的钥匙。

> 记住：**约数问题的两大武器是「配对性」与「交换求和顺序」**；前者把 $O(n)$ 降到 $O(\sqrt n)$，后者把「枚举约数」变成「枚举倍数」。
