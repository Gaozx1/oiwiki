# 素数筛法（Sieve of Primes）

> 一句话概括：**素数筛法通过「用每个素数去筛掉它的倍数」的方式，在 $O(n)$ 或 $O(n\log\log n)$ 时间内求出 $1$ 到 $n$ 中所有素数**，是数论中最基础、最常用的预处理手段。

---

## 目录

1. [素数与合数](#1-素数与合数)
2. [朴素筛与埃氏筛](#2-朴素筛与埃氏筛)
3. [线性筛（欧拉筛）](#3-线性筛欧拉筛)
4. [线性筛的副产品](#4-线性筛的副产品)
5. [模板代码](#5-模板代码)
6. [区间筛与大数判素数](#6-区间筛与大数判素数)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 素数与合数

**定义**：若正整数 $p > 1$ 且它的正因数只有 $1$ 和 $p$，则称 $p$ 为**素数**（质数）；否则称为**合数**。$1$ 既不是素数也不是合数。

### 1.1 素数的基本性质

1. **唯一分解定理**：任意大于 $1$ 的整数都能唯一写成
   $$
   n = p_1^{e_1} p_2^{e_2} \cdots p_k^{e_k}
   $$
   其中 $p_i$ 为互不相同的素数，$e_i \ge 1$。

2. **素数有无穷多个**（欧几里得证明）：若素数只有有限个 $p_1, \ldots, p_k$，令 $N = p_1p_2\cdots p_k + 1$，则 $N$ 被任一 $p_i$ 除都余 $1$，故 $N$ 要么本身是素数，要么含有不在列表中的素因子，矛盾。

3. **分布稀疏**：由素数定理，不超过 $n$ 的素数个数 $\pi(n) \sim \dfrac{n}{\ln n}$。$n = 10^8$ 时约有 $5.76 \times 10^6$ 个素数。

### 1.2 试除法判素数

判断单个数 $n$ 是否为素数，只需枚举 $i \in [2, \lfloor\sqrt n\rfloor]$ 检查 $i \mid n$。原因是：若 $n = ab$ 且 $a \le b$，则 $a \le \sqrt n$。

```cpp
bool isPrime(long long n) {
    if (n < 2) return false;
    for (long long i = 2; i * i <= n; i++)
        if (n % i == 0) return false;
    return true;
}
```

单次判断 $O(\sqrt n)$。但若要判断 $1 \sim n$ 中所有数，逐个试除是 $O(n\sqrt n)$，太慢，于是有了「筛法」。

---

## 2. 朴素筛与埃氏筛

### 2.1 朴素筛

最直接的想法：从小到大枚举 $i$，若 $i$ 未被标记，则 $i$ 是素数，把 $2i, 3i, 4i, \ldots$ 全部标记为合数。

```cpp
bool vis[N];              // vis[i] = true 表示 i 被筛掉（合数）
void naive_sieve(int n) {
    for (int i = 2; i <= n; i++) {
        if (vis[i]) continue;          // 已被更小的素数筛掉
        for (int j = 2 * i; j <= n; j += i)
            vis[j] = true;             // 标记 i 的倍数
    }
}
```

总标记次数为 $\sum_{p \le n} n/p \approx n\ln\ln n = O(n\log\log n)$。问题在于：**每个合数会被它的每一个素因子重复筛一次**，例如 $12$ 会被 $2$ 和 $3$ 各筛一次。

### 2.2 埃氏筛（Eratosthenes Sieve）

关键优化：枚举素因子 $p$ 时，**只需从 $p^2$ 开始筛**。因为 $2p, 3p, \ldots, (p-1)p$ 都含有比 $p$ 小的素因子，早已被筛掉。

```cpp
bool vis[N];
void eratosthenes(int n) {
    for (int i = 2; i <= n; i++) {
        if (vis[i]) continue;
        for (long long j = 1LL * i * i; j <= n; j += i)   // 从 i*i 开始
            vis[j] = true;
    }
}
```

复杂度 $O(n \log\log n)$，常数极小，$n = 10^8$ 时通常 1 秒内可完成。

### 2.3 埃氏筛的正确性证明

**命题**：算法结束时 `vis[x] = false` 当且仅当 $x$ 是素数。

**证明**：

- 若 $x$ 是合数，设 $x = ab$，$2 \le a \le b < x$，则 $a \le \sqrt x$。$a$ 必有素因子 $p \le a \le \sqrt x$。外层循环枚举到 $p$ 时（$p$ 是素数，`vis[p]` 为 `false`，必然进入内层），内层从 $p^2$ 起以步长 $p$ 筛，而 $x = p\cdot(x/p)$ 且 $x/p \ge p$（因为 $p \le \sqrt x$），故 $x$ 被筛掉。
- 若 $x$ 是素数，它不会被任何 $p < x$ 的倍数覆盖（因为 $p \nmid x$），故保持 `false`。$\square$

### 2.4 只筛奇数的优化

偶数（除 $2$ 外）都是合数，可只存奇数下标，时空均约减半：下标 $i$ 表示数字 $2i+1$，标记时从 $p^2$ 起以 $2p$ 为步长即可。

---

## 3. 线性筛（欧拉筛）

### 3.1 核心思想

埃氏筛中一个合数仍可能被多个素数筛多次。线性筛的目标是：**让每个合数只被它的最小素因子筛掉一次**，从而严格 $O(n)$。

做法：维护已找到的素数表 `primes`，对每个 $i$，用 $i$ 乘上 `primes[j]` 去筛，并且当 `primes[j]` 整除 $i$ 时立刻 `break`。

### 3.2 代码

```cpp
const int N = 1e7 + 5;
bool vis[N];              // vis[i] = true 表示 i 是合数
int primes[N / 10], cnt;  // 素数表

void linear_sieve(int n) {
    for (int i = 2; i <= n; i++) {
        if (!vis[i]) primes[++cnt] = i;       // i 未被筛掉，是素数
        for (int j = 1; j <= cnt && 1LL * i * primes[j] <= n; j++) {
            vis[i * primes[j]] = true;         // 用 i × p_j 筛
            if (i % primes[j] == 0) break;     // p_j 是 i 的最小素因子，停止
        }
    }
}
```

### 3.3 线性复杂度的证明

**关键不变式**：每个合数 $x$ 恰好在 $i = x / \text{spf}(x)$、$p_j = \text{spf}(x)$ 时被筛掉，其中 $\text{spf}$ 表示最小素因子。

**证明**：

1. **一定会被筛到**：设 $x$ 的最小素因子为 $p$，令 $i = x/p$。因为 $p$ 不大于 $x$ 的任何素因子，所以 $i$ 的所有素因子都 $\ge p$。当外层循环到 $i$ 时，素数表中小于 $p$ 的素数都不能整除 $i$，循环不会提前 `break`，会一直枚举到 $p_j = p$，从而筛掉 $x = i \cdot p$。
2. **只被筛一次**：要筛掉 $x = i\cdot p_j$，必须 $p_j \le \text{spf}(i)$（否则在 $p_j = \text{spf}(i)$ 处已 `break`）。给定 $x$，满足该条件且乘积为 $x$ 的分解只有 $p_j = \text{spf}(x)$ 一种——若 $p_j > \text{spf}(x)$，则 $i = x/p_j$ 含有更小的素因子 $\text{spf}(x) < p_j$，与 $p_j \le \text{spf}(i)$ 矛盾。

因此每个合数被标记恰好一次，总标记次数不超过 $n$，复杂度 $O(n)$。$\square$

### 3.4 两种筛法的对比

| 指标 | 埃氏筛 | 线性筛 |
|---|---|---|
| 时间复杂度 | $O(n\log\log n)$ | $O(n)$ |
| 常数 | 小 | 略大（取模、数组访问多） |
| 额外功能 | 无 | 可顺带求积性函数 |

$n \le 10^7$ 时两者都能过；若只求素数且 $n$ 很大，埃氏筛常数更优；若要求积性函数，必须用线性筛。

---

## 4. 线性筛的副产品

线性筛最强大的地方在于：每个合数 $x$ 都通过 $x = i \cdot p$ 被唯一分解（$p = \text{spf}(x)$），因此可以顺势递推各种**积性函数**。

### 4.1 欧拉函数 $\varphi(n)$

$\varphi(n)$ 表示 $1 \sim n$ 中与 $n$ 互质的数的个数。关键性质：

- 若 $p$ 为素数，$\varphi(p) = p - 1$；
- 若 $p \mid n$，$\varphi(np) = p \cdot \varphi(n)$；
- 若 $p \nmid n$，$\varphi(np) = (p-1)\varphi(n)$。

```cpp
int phi[N];
void sieve_phi(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!vis[i]) { primes[++cnt] = i; phi[i] = i - 1; }
        for (int j = 1; j <= cnt && 1LL * i * primes[j] <= n; j++) {
            int p = primes[j];
            vis[i * p] = true;
            if (i % p == 0) { phi[i * p] = phi[i] * p; break; }
            else            { phi[i * p] = phi[i] * (p - 1); }
        }
    }
}
```

### 4.2 约数个数 $d(n)$

设 $n = \prod p_i^{e_i}$，则 $d(n) = \prod (e_i + 1)$。线性筛时额外记录「最小素因子的指数」`cnt[i]`：

- 若 $p \mid i$：$d(ip) = d(i) / (\text{cnt}[i]+1) \cdot (\text{cnt}[i]+2)$，`cnt[ip] = cnt[i]+1`；
- 若 $p \nmid i$：$d(ip) = 2\,d(i)$，`cnt[ip] = 1`。

### 4.3 莫比乌斯函数 $\mu(n)$

$$
\mu(n) =
\begin{cases}
1, & n = 1 \\
(-1)^k, & n = p_1p_2\cdots p_k \text{（无平方因子）} \\
0, & \text{存在 } p^2 \mid n
\end{cases}
$$

线性筛递推：若 $p \mid i$ 则 $\mu(ip) = 0$，否则 $\mu(ip) = -\mu(i)$。

---

## 5. 模板代码

### 5.1 完整线性筛（C++14，可直接编译）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 10000005;
bool vis[MAXN];              // true 表示合数
int primes[MAXN / 10];       // 素数表，约 n/ln n 个
int cnt = 0;

// 线性筛：求出 2..n 的所有素数
void linear_sieve(int n) {
    for (int i = 2; i <= n; i++) {
        if (!vis[i]) primes[++cnt] = i;      // 没被筛掉，i 是素数
        for (int j = 1; j <= cnt; j++) {
            long long v = 1LL * i * primes[j];
            if (v > n) break;
            vis[v] = true;                    // 用 i × p_j 标记合数
            if (i % primes[j] == 0) break;    // 保证每个合数只被最小素因子筛一次
        }
    }
}

int main() {
    int n, q;
    scanf("%d%d", &n, &q);
    linear_sieve(n);
    while (q--) {
        int k;
        scanf("%d", &k);
        printf("%d\n", primes[k]);            // 第 k 小的素数
    }
    return 0;
}
```

### 5.2 线性筛求欧拉函数（完整版）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e7 + 5;
bool vis[MAXN];
int primes[MAXN / 10], cnt;
int phi[MAXN];

void sieve_phi(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!vis[i]) { primes[++cnt] = i; phi[i] = i - 1; }
        for (int j = 1; j <= cnt; j++) {
            long long v = 1LL * i * primes[j];
            if (v > n) break;
            vis[v] = true;
            if (i % primes[j] == 0) {
                phi[v] = phi[i] * primes[j];        // p | i
                break;
            } else {
                phi[v] = phi[i] * (primes[j] - 1);  // p ∤ i
            }
        }
    }
}

int main() {
    int n;
    scanf("%d", &n);
    sieve_phi(n);
    long long sum = 0;
    for (int i = 1; i <= n; i++) sum += phi[i];
    printf("%lld\n", sum);
    return 0;
}
```

---

## 6. 区间筛与大数判素数

### 6.1 区间筛

求 $[L, R]$ 内的所有素数（$R$ 可达 $10^{12}$，但 $R - L \le 10^6$）。

**思路**：先筛出 $\sqrt R$ 以内的素数，再用这些素数去筛 $[L, R]$：对每个素数 $p$，从 $\max(p^2, \lceil L/p\rceil p)$ 开始，以步长 $p$ 标记合数。

```cpp
// 求 [L, R] 内的素数，要求 R <= 1e12, R - L <= 1e6
vector<long long> segment_sieve(long long L, long long R) {
    vector<bool> isComp(R - L + 1, false);
    if (L == 1) isComp[0] = true;                     // 1 不是素数
    for (long long p = 2; p * p <= R; p++) {
        bool isP = true;                              // 试除判断 p 是否为素数
        for (long long d = 2; d * d <= p; d++)
            if (p % d == 0) { isP = false; break; }
        if (!isP) continue;
        long long start = max(p * p, ((L + p - 1) / p) * p);
        for (long long x = start; x <= R; x += p)
            isComp[x - L] = true;
    }
    vector<long long> res;
    for (long long i = L; i <= R; i++)
        if (!isComp[i - L]) res.push_back(i);
    return res;
}
```

上面的试除只是为了代码独立可读；实际应先预处理 $\sqrt R \le 10^6$ 的素数表再枚举，效率更高。

### 6.2 Miller–Rabin 素性测试

当 $n$ 达到 $10^{18}$ 时无法筛，需要概率性算法 Miller–Rabin。它基于费马小定理的逆否命题与二次探测定理：

- 若 $n$ 为奇素数，写 $n - 1 = 2^s d$（$d$ 为奇数），则对任意 $a$ 有
  $$
  a^d \equiv 1 \pmod n \quad \text{或} \quad \exists\, 0 \le r < s,\ a^{2^r d} \equiv -1 \pmod n
  $$
- 取 $a \in \{2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37\}$ 即可确定性覆盖 $n < 3.3 \times 10^{24}$ 的所有情况。

单次测试 $O(\log^3 n)$（快速幂），常数小，是竞赛中判大素数的标准工具。若 $n < 2^{32}$，只需测试 $a \in \{2, 3, 5, 7\}$。

---

## 7. 经典例题

### 例题 1：洛谷 P3383 【模板】线性筛素数

**题意**：给定 $n, q$，求第 $k$ 小的素数（$n \le 10^8$，$q \le 10^6$）。

**思路**：线性筛预处理出所有素数，$O(1)$ 回答每次询问。

**复杂度**：预处理 $O(n)$，询问 $O(1)$，总 $O(n + q)$。代码见 5.1。

---

### 例题 2：洛谷 P1835 素数密度

**题意**：给定 $L, R$（$1 \le L \le R \le 2^{31}-1$，$R - L \le 10^6$），求区间内素数个数。

**思路**：区间筛。先筛出 $\sqrt R \le 46341$ 内的素数，再用它们标记 $[L, R]$ 中的合数。

**复杂度**：$O(\sqrt R \log\log R + (R-L)\log\log R)$。

**核心代码**：

```cpp
vector<int> small_primes;         // sqrt(R) 以内的素数
void build_small(int lim) {
    vector<bool> comp(lim + 1, false);
    for (int i = 2; i <= lim; i++) {
        if (!comp[i]) {
            small_primes.push_back(i);
            for (long long j = 1LL * i * i; j <= lim; j += i) comp[j] = true;
        }
    }
}

int count_primes(long long L, long long R) {
    vector<bool> isComp(R - L + 1, false);
    if (L == 1) isComp[0] = true;
    for (int p : small_primes) {
        long long start = max(1LL * p * p, ((L + p - 1) / p) * p);
        for (long long x = start; x <= R; x += p)
            isComp[x - L] = true;
    }
    int cnt = 0;
    for (long long i = L; i <= R; i++) if (!isComp[i - L]) cnt++;
    return cnt;
}
```

---

### 例题 3：洛谷 P2158 [SDOI2008] 仪仗队

**题意**：一个 $n \times n$ 的方阵，从 $(0,0)$ 能直接看到多少个点（$1 \le x, y < n$，视线不被其他点遮挡）。

**思路**：能看到的点 $(x, y)$ 满足 $\gcd(x, y) = 1$。由对称性，答案为

$$
2\sum_{i=1}^{n-1} \varphi(i) + 1
$$

其中 $+1$ 对应对角线上的点 $(1,1)$ 这一类边界情形，具体按题目约定。

**复杂度**：线性筛求 $\varphi$，$O(n)$。

---

### 例题 4：线性筛应用——求 $\sum_{i=1}^{n} d(i)$

**题意**：求 $\sum_{i=1}^{n} d(i)$，$n \le 10^7$。

**思路一**：线性筛预处理 $d(i)$ 后求和，$O(n)$。
**思路二**：$\sum_{i=1}^n d(i) = \sum_{d=1}^n \lfloor n/d \rfloor$，用整除分块做到 $O(\sqrt n)$。

后者更优，但前者是线性筛求积性函数的典型练习。

---

## 8. 易错点

1. **数组开小**：素数表大小约为 $n / \ln n$，$10^7$ 时约 $664579$ 个，保险起见开 `MAXN / 10`。
2. **`i * i` 溢出**：`int` 的 `i * i` 在 $i > 46340$ 时溢出，务必写成 `1LL * i * i`。
3. **线性筛的 `break` 位置**：必须先标记 `vis[i * p]`，再判断 `i % p == 0` 后 `break`；顺序颠倒会漏筛。
4. **区间筛下标偏移**：用 `vis[i - L]` 存结果时 $L$ 可能很大，注意 `long long` 类型与整除上取整写法。
5. **$1$ 的处理**：$1$ 不是素数，区间筛若 $L = 1$ 需手动标记 `isComp[0] = true`。
6. **多组数据清空开销**：不要每组都 `memset` 整个 `MAXN`，只清 $[0, n]$ 或使用时间戳技巧。
7. **积性函数递推写反**：$p \mid i$ 与 $p \nmid i$ 两种情况要分清，写反答案全错。
8. **埃氏筛从 $2i$ 开始**：虽正确但慢一倍，应从 $i^2$ 开始。

---

## 9. 练习建议

由易到难：

1. **入门**：洛谷 P3383 【模板】线性筛素数 —— 熟悉线性筛。
2. **入门**：洛谷 P5736 【深基7.例2】质数筛 —— 埃氏筛基础。
3. **基础**：洛谷 P1835 素数密度 —— 区间筛。
4. **基础**：洛谷 P2158 [SDOI2008] 仪仗队 —— 欧拉函数 + 线性筛。
5. **进阶**：洛谷 P3327 [SDOI2015] 约数个数和 —— 线性筛 + 莫比乌斯反演。
6. **进阶**：洛谷 P4450 双亲数 —— 线性筛 + 数论分块。
7. **提高**：洛谷 P4718 【模板】Pollard-Rho —— 大数分解，需 Miller–Rabin。

**学习路径建议**：

- 先手写埃氏筛并证明正确性，理解「为什么从 $i^2$ 开始」；
- 再啃线性筛的 `break` 条件，用 $12, 18, 20$ 手动模拟筛的过程；
- 最后掌握线性筛求积性函数的套路，这是数论题的高频考点。

> 一句话总结：**筛法的本质是「用已知的素数去排除未知的合数」**；线性筛的精华在于「每个合数只被最小素因子筛一次」。
