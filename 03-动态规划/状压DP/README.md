# 状压 DP：把「选没选」压进一个整数

> 一句话概括：**状压 DP 用二进制整数表示一个集合的选取状态，把「集合」当作下标来转移**，适合 $n\le 20$ 左右的子集枚举与棋盘覆盖问题。

---

## 目录

1. [概念与原理](#1-概念与原理)
   - [1.1 什么时候需要状态压缩](#11-什么时候需要状态压缩)
   - [1.2 二进制表示集合的常用操作](#12-二进制表示集合的常用操作)
   - [1.3 模型一：旅行商问题（TSP）](#13-模型一旅行商问题tsp)
   - [1.4 模型二：棋盘覆盖（行内状压）](#14-模型二棋盘覆盖行内状压)
   - [1.5 模型三：子集枚举与 SOS DP](#15-模型三子集枚举与-sos-dp)
   - [1.6 优化技巧：预处理合法状态](#16-优化技巧预处理合法状态)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 什么时候需要状态压缩

普通 DP 的状态如果写成「一个集合选了哪些元素」，集合有 $2^n$ 种可能，无法用一两个下标表示。但如果 $n$ 很小（通常 $n\le 20$），我们可以**把集合编码成一个整数**：

- 整数 $S$ 的第 $i$ 个二进制位为 1，表示「元素 $i$ 被选中」；
- $S$ 的取值范围是 $[0, 2^n)$，共 $2^n$ 个状态。

于是状态可以写成 $dp_{S}$ 或 $dp_{i,S}$，这就是**状态压缩 DP（状压 DP）**。

**判断能不能状压的三个信号**：

1. 数据范围小得反常，例如 $n\le 20$、$m\le 12$；
2. 决策之间只有「选/不选」两种关系；
3. 需要记录「哪些东西已经用过」。

**复杂度直觉**：$2^{20}\approx 10^6$，再乘一个 $n$ 也只有 $2\times 10^7$，是可接受的；但 $2^{25}\approx 3.3\times 10^7$ 就已经很危险。

### 1.2 二进制表示集合的常用操作

设 $S$ 是一个状态整数（从 0 开始编号元素）：

| 操作 | 写法 | 含义 |
|---|---|---|
| 判断第 $i$ 位 | `(S >> i) & 1` | 元素 $i$ 是否被选 |
| 把第 $i$ 位置 1 | `S \| (1 << i)` | 选中元素 $i$ |
| 把第 $i$ 位置 0 | `S & ~(1 << i)` | 取消元素 $i$ |
| 最低位的 1 | `S & (-S)` | lowbit |
| 去掉最低位的 1 | `S & (S - 1)` | 常用于枚举子集 |
| 全选 | `(1 << n) - 1` | $n$ 位全 1 |

**枚举 $S$ 的所有子集**（重要技巧）：

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    int S = 0b1010;
    for (int sub = S; sub; sub = (sub - 1) & S) cout << sub << ' ';
    cout << '\n';                    // 输出 10 8 2
    return 0;
}
```

每次 `(sub - 1) & S` 都会得到「比 sub 小的下一个子集」，总枚举量是 $3^n$ 级别（所有 $(S, sub)$ 对）。

```
S = 1010 (二进制)
sub 依次为: 1010 → 1000 → 0010 → 0000（结束）
```

### 1.3 模型一：旅行商问题（TSP）

**问题**：$n$ 个城市，两两之间有距离 $d_{i,j}$，从城市 0 出发，经过每个城市恰好一次后回到 0，求最短总路程。

**状态设计**：$dp_{S,i}$ 表示「已经访问过的城市集合为 $S$，当前停在城市 $i$」时的最短路径长度。

**转移方程**：

$$
dp_{S\cup\{j\},j}=\min\left(dp_{S\cup\{j\},j},\ dp_{S,i}+d_{i,j}\right),\quad j\notin S
$$

**边界**：$dp_{\{0\},0}=0$，其余为 $+\infty$。

**答案**：$\min_{i\ne 0}\left(dp_{\text{all},i}+d_{i,0}\right)$。

```
状态转移示意：

   S = {0,2}, i = 2           加入城市 3
   ┌───────────────┐        ┌──────────────────┐
   │ dp[{0,2}][2]  │  ───→  │ dp[{0,2,3}][3]   │
   └───────────────┘        └──────────────────┘
       已有的路径              多走一段 d[2][3]

  状态总数 2^n × n，每个状态枚举 n 个后继 → O(2^n · n^2)
```

### 1.4 模型二：棋盘覆盖（行内状压）

**问题**：在 $n\times m$ 的棋盘上放若干个「不相邻」的棋子（或铺满 $1\times 2$ 骨牌），求方案数。$n\le 12$，$m$ 可以较大。

**思路**：**按行转移**。把每一行的放置情况压缩成一个 $m$ 位二进制数 $S$，$dp_{i,S}$ 表示前 $i$ 行、第 $i$ 行状态为 $S$ 的方案数。

**合法性检查**：

1. **行内合法**：不能有相邻的 1（放棋子问题），即 `(S & (S << 1)) == 0`；
2. **行间合法**：相邻两行的 1 不能在同一列（否则上下相邻），即 `(S & T) == 0`；
3. **骨牌问题**则要把「竖着放」和「横着放」的占位一起考虑，通常用「当前行被上一行占掉的格子」作为状态。

```
第 i-1 行:  0 1 0 0 1 0   →  S = 010010
第 i   行:  1 0 0 1 0 0   →  T = 100100
检查 S & T = 0 ✔（没有同一列上下都放）

行内检查:  S & (S<<1) = 0 ✔（没有左右相邻）
```

转移：

$$
dp_{i,S}=\sum_{T\ \text{合法},\ S\&T=0} dp_{i-1,T}
$$

**复杂度**：状态 $2^m$，每行枚举状态对 $O(4^m)$；若预处理合法状态集合，通常能降到 $O(n\cdot 3^m)$ 或更低。

### 1.5 模型三：子集枚举与 SOS DP

有些问题要对每个集合 $S$ 求「所有子集 $T\subseteq S$ 的权值和」，即

$$
F(S)=\sum_{T\subseteq S} f(T)
$$

直接枚举子集是 $O(3^n)$。用 **SOS DP（Sum Over Subsets）** 可以做到 $O(n2^n)$：逐位考虑，把「第 $i$ 位为 1 的集合」的答案累加到「第 $i$ 位为 0 的集合」上。

核心代码只有三重循环（完整模板见 2.2 节）：

```
for i in [0, n):
    for S in [0, 2^n):
        if S 的第 i 位为 1:
            F[S] += F[S ^ (1 << i)]
```

**正确性直觉**：外层枚举位，保证每个子集恰好被累加一次。这是状压 DP 中非常高频的优化。

### 1.6 优化技巧：预处理合法状态

很多棋盘题的「行内合法状态」很少（例如 $m=12$ 时只有 377 种），可以先把它们存进数组 `valid[]`，转移时只在这些状态之间枚举，复杂度从 $O(4^m)$ 降到 $O(\text{cnt}^2)$。筛法是枚举 $0\sim 2^m-1$，保留满足 `(S & (S << 1)) == 0` 的 $S$。具体实现见「例题 3：玉米田」的代码。

另外，**滚动数组**几乎总是必要的：$dp$ 的第一维只依赖上一行，可以压成两行。

---

## 2. 模板代码

### 2.1 TSP 的状压 DP

```cpp
#include <bits/stdc++.h>
using namespace std;

const int INF = 0x3f3f3f3f;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<vector<int>> d(n, vector<int>(n));
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j) cin >> d[i][j];

    int full = 1 << n;
    vector<vector<int>> dp(full, vector<int>(n, INF));
    dp[1][0] = 0;                          // 只访问过城市 0，停在 0

    for (int S = 1; S < full; ++S) {
        for (int i = 0; i < n; ++i) {
            if (dp[S][i] == INF) continue;         // 非法状态
            if (!(S >> i & 1)) continue;           // i 必须在集合里
            for (int j = 0; j < n; ++j) {
                if (S >> j & 1) continue;          // j 还没访问
                int nS = S | (1 << j);
                dp[nS][j] = min(dp[nS][j], dp[S][i] + d[i][j]);
            }
        }
    }

    int ans = INF;
    for (int i = 1; i < n; ++i)
        ans = min(ans, dp[full - 1][i] + d[i][0]); // 回到起点
    cout << ans << '\n';
    return 0;
}
```

### 2.2 SOS DP（$O(n2^n)$）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<long long> F(1 << n);
    for (int S = 0; S < (1 << n); ++S) cin >> F[S];   // 初值即 f(S)

    for (int i = 0; i < n; ++i)
        for (int S = 0; S < (1 << n); ++S)
            if (S >> i & 1)
                F[S] += F[S ^ (1 << i)];               // 把缺第 i 位的集合累加进来

    for (int S = 0; S < (1 << n); ++S) cout << F[S] << (S + 1 == (1 << n) ? '\n' : ' ');
    return 0;
}
```

---

## 3. 经典例题

### 例题 1：最短哈密顿回路（TSP 模板）

**题意**：$n$ 个点（$n\le 18$）的完全图，给定边权，求从 0 号点出发经过所有点恰好一次再回到 0 的最小总长度。

**思路**：$dp_{S,i}$ 表示访问集合 $S$、当前在 $i$ 的最短路。转移枚举下一个点 $j\notin S$。

**复杂度**：时间 $O(2^n n^2)$，空间 $O(2^n n)$。

### 例题 2：互不侵犯（洛谷 P1896 [SCOI2005]）

**题意**：在 $n\times n$ 的棋盘上放 $k$ 个国王，使任意两个国王不相邻（上下左右及斜对角）。$1\le n\le 9$，$0\le k\le n^2$。

**思路**：按行状压。$dp_{i,S,c}$ 表示前 $i$ 行、第 $i$ 行状态为 $S$、共放了 $c$ 个国王的方案数。行内合法条件：`S & (S<<1) == 0`；行间合法条件：`S & T == 0` 且 `S & (T<<1) == 0` 且 `(S<<1) & T == 0`（斜对角也不能相邻）。

**复杂度**：时间 $O(n\cdot c\cdot \text{cnt}^2)$，其中 cnt 是合法单行状态数（$n=9$ 时约 89）。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, K;
    cin >> n >> K;
    int full = 1 << n;

    vector<int> valid, cnt1(full, 0);
    for (int S = 0; S < full; ++S) {
        if (S & (S << 1)) continue;        // 行内有相邻国王
        valid.push_back(S);
        cnt1[S] = __builtin_popcount(S);   // 本行国王数
    }

    int V = valid.size();
    // dp[v][c]：当前行取第 v 个合法状态、共放 c 个国王的方案数
    vector<vector<long long>> dp(V, vector<long long>(K + 1, 0));
    for (int v = 0; v < V; ++v)
        if (cnt1[valid[v]] <= K) dp[v][cnt1[valid[v]]] = 1;

    for (int row = 2; row <= n; ++row) {
        vector<vector<long long>> ndp(V, vector<long long>(K + 1, 0));
        for (int v = 0; v < V; ++v) {
            int S = valid[v];
            for (int u = 0; u < V; ++u) {
                int T = valid[u];
                if (S & T) continue;                       // 正上方
                if (S & (T << 1)) continue;                // 左上方
                if ((S << 1) & T) continue;                // 右上方
                for (int c = 0; c + cnt1[S] <= K; ++c)
                    ndp[v][c + cnt1[S]] += dp[u][c];
            }
        }
        dp.swap(ndp);
    }

    long long ans = 0;
    for (int v = 0; v < V; ++v) ans += dp[v][K];
    cout << ans << '\n';
    return 0;
}
```

### 例题 3：玉米田（洛谷 P1879 [USACO06NOV]）

**题意**：$n\times m$ 的玉米田（$n,m\le 12$），有些格子不能种，且相邻格子不能同时种。求种植方案数（不种也算一种）。

**思路**：行内状压 + 行间 `S & T == 0`；用 `ok[i]` 记录第 $i$ 行可种的二进制掩码，`S` 必须是 `ok[i]` 的子集。

**复杂度**：时间 $O(n\cdot \text{cnt}^2)$，`cnt` 为合法状态数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MOD = 100000000;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n, m;
    cin >> n >> m;
    vector<int> ok(n, 0);
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < m; ++j) {
            int x;
            cin >> x;
            if (x) ok[i] |= 1 << j;        // 可种的格子置 1
        }

    int full = 1 << m;
    vector<int> valid;
    for (int S = 0; S < full; ++S)
        if ((S & (S << 1)) == 0) valid.push_back(S);

    vector<long long> dp(full, 0), ndp(full, 0);
    for (int S : valid)
        if ((S & ok[0]) == S) dp[S] = 1;   // 第一行：只能种在可种格子上

    for (int i = 1; i < n; ++i) {
        fill(ndp.begin(), ndp.end(), 0);
        for (int S : valid) {
            if ((S & ok[i]) != S) continue;              // 不能种在贫瘠格子
            for (int T : valid) {
                if (S & T) continue;                     // 上下不能相邻
                ndp[S] = (ndp[S] + dp[T]) % MOD;
            }
        }
        dp.swap(ndp);
    }

    long long ans = 0;
    for (int S : valid) ans = (ans + dp[S]) % MOD;
    cout << ans << '\n';
    return 0;
}
```

### 例题 4：集合划分（子集 DP）

**题意**：给定 $n$ 个数（$n\le 16$），要把它们分成若干组，每组内任意两数之和为质数，求最少分几组。

**思路**：先预处理每个子集是否「内部两两和为质数」，记 `good[S]`。然后 $dp_S$ 表示把集合 $S$ 分完的最少组数：

$$
dp_S=\min_{T\subseteq S,\ \text{good}[T]}\left(dp_{S\setminus T}+1\right)
$$

用子集枚举实现，复杂度 $O(3^n)$。

**复杂度**：时间 $O(3^n)$，空间 $O(2^n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> a(n);
    for (int &x : a) cin >> x;

    int full = 1 << n;
    auto isPrime = [](int x) {
        if (x < 2) return false;
        for (int d = 2; d * d <= x; ++d)
            if (x % d == 0) return false;
        return true;
    };

    vector<char> good(full, 0);
    good[0] = 1;
    for (int S = 1; S < full; ++S) {
        bool ok = true;
        for (int i = 0; i < n && ok; ++i)
            if (S >> i & 1)
                for (int j = i + 1; j < n && ok; ++j)
                    if ((S >> j & 1) && !isPrime(a[i] + a[j])) ok = false;
        good[S] = ok;
    }

    const int INF = 0x3f3f3f3f;
    vector<int> dp(full, INF);
    dp[0] = 0;
    for (int S = 1; S < full; ++S)
        for (int T = S; T; T = (T - 1) & S)       // 枚举 S 的非空子集
            if (good[T]) dp[S] = min(dp[S], dp[S ^ T] + 1);

    cout << dp[full - 1] << '\n';
    return 0;
}
```

---

## 4. 易错点与练习建议

### 4.1 常见易错点

1. **忘记初始化 $dp_{\{0\},0}$**：TSP 中起点状态必须手动置 0，其余置无穷。
2. **转移时没判断 $i\in S$**：$dp_{S,i}$ 只在 $i$ 属于集合 $S$ 时有意义，遍历时要用 `if (!(S >> i & 1)) continue;` 过滤。
3. **枚举顺序错**：TSP 要按 $S$ 从小到大枚举；若从大往小，会用到还没算出的状态。
4. **行间约束漏斜对角**：互不侵犯问题中，除了 `S & T`，还要检查 `S & (T<<1)` 与 `(S<<1) & T`。
5. **位运算优先级**：`S & 1 << i` 实际是 `S & (1 << i)`，但 `S & 1 == 1` 是 `S & (1 == 1)`，一定要加括号。
6. **`1 << n` 溢出**：$n\ge 31$ 时 `int` 左移会溢出（未定义行为），必须用 `1LL << n` 或换 `long long`。
7. **状态数爆内存**：$2^{20}\times 20$ 的 `int` 数组约 80 MB，接近很多题的限制，需要滚动或压缩。
8. **SOS DP 循环顺序反了**：外层必须是「位」，内层才是「集合」；反过来会重复累加。

### 4.2 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1896 互不侵犯 | 按行状压、行间约束 |
| 入门 | 洛谷 P1879 玉米田 | 掩码子集判断 |
| 普及 | 洛谷 P1433 吃奶酪 | TSP 状压 |
| 普及 | 洛谷 P3092 没有零钱 | 状压 + 贪心 |
| 提高 | 洛谷 P2150 寿司晚宴 | 状压 + 质因数分解 |
| 提高 | 洛谷 P3959 宝藏 | 状压 + 枚举层 |
| 提高 | 洛谷 P4363 一双木棋 | 状压 + 博弈 |

**训练方法**：

1. 先在纸上把集合画成二进制串，手动模拟一次 `S → S | (1<<j)` 的转移；
2. 记住三个模板：TSP、按行棋盘、子集枚举；
3. 位运算的括号一定要写全，宁可多加几对；
4. 遇到 $n\le 20$ 但想不出多项式算法时，优先考虑状压。
