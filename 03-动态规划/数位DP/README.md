# 数位 DP：按位统计满足条件的数

> 一句话概括：**数位 DP 把整数拆成从高位到低位的数字串，逐位确定并记录「是否贴着上界」等约束**，用于统计区间 $[L,R]$ 中满足某种数位条件的数的个数或贡献。

---

## 目录

1. [概念与原理](#1-概念与原理)
   - [1.1 从「枚举数」到「枚举数位」](#11-从枚举数到枚举数位)
   - [1.2 数位 DP 的四个关键量](#12-数位-dp-的四个关键量)
   - [1.3 记忆化搜索模板](#13-记忆化搜索模板)
   - [1.4 模型一：windy 数（相邻数位差）](#14-模型一windy-数相邻数位差)
   - [1.5 模型二：不要 62（禁止子串）](#15-模型二不要-62禁止子串)
   - [1.6 模型三：统计每个数字出现次数](#16-模型三统计每个数字出现次数)
   - [1.7 模型四：二进制数位 DP](#17-模型四二进制数位-dp)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 从「枚举数」到「枚举数位」

如果题目要求统计 $[1,N]$ 中满足某条件的数的个数，而 $N\le 10^{18}$，直接枚举每个数是不可能的。但**数的十进制表示只有约 19 位**，每一位只有 $0\sim 9$ 十种选择。于是我们可以**逐位构造数字**，用 DP 统计方案数。

核心思想是**前缀计数**：设 $f(N)$ 表示 $[1,N]$ 中满足条件的数的个数，则答案

$$
\text{ans}=f(R)-f(L-1)
$$

这把「区间」问题变成了「前缀」问题，是数位 DP 最常用的转化。

```
统计 [1, N] 中满足条件的数：

N = 3 1 4 5
    ↑ ↑ ↑ ↑
    逐位确定。每一位的取值分两种情况：
    ① 之前的高位已经严格小于 N 的高位 → 这一位可以取 0..9
    ② 之前的高位与 N 完全相同 → 这一位只能取 0..N[i]

把所有「贴着上界」和「已经自由」的分支相加，就是不重不漏的计数
```

### 1.2 数位 DP 的四个关键量

设计数位 DP 状态时，通常需要记录以下四类信息：

| 关键量 | 含义 | 是否写进状态 |
|---|---|---|
| `pos` | 当前处理到第几位（从高到低） | 是 |
| `limit` | 前面是否与上界完全相同（贴着上界） | 通常作为搜索参数，不记忆化 |
| `lead` | 前面是否全是前导零 | 视题目需要 |
| `last` 等 | 上一位数字、数位和、是否出现过某模式 | 是 |

**为什么 `limit` 一般不写进记忆化数组？** 因为 `limit=true` 的状态在整个搜索中只有一条链（每一位都取上界），数量极少，记忆化它没有收益，反而会让数组多一维。

**前导零（lead）**：数字 $0$ 与「前导零」不同。例如统计「相邻两位之差至少为 2」时，前导零不能参与比较，否则 `00012` 会被误判。前导零标志用来区分「还没开始写有效数字」和「写了数字 0」。

### 1.3 记忆化搜索模板

数位 DP 最推荐**记忆化搜索**写法，思路直白、边界清晰：

```
dfs(pos, ...其他约束..., limit, lead):
    if pos == 位数:  return 1（或按题意返回贡献）
    if not limit and not lead and 已记忆: return memo[...]
    up = limit ? 当前位上界 : 9
    res = 0
    for d in 0..up:
        if 需要 lead 且 d == 0:
            res += dfs(pos+1, ..., limit && d == up, true)
        else:
            if 约束不满足(d, ...): continue
            res += dfs(pos+1, ..., limit && d == up, false)
    if not limit and not lead: memo[...] = res
    return res
```

**记忆化成立的前提**：当 `limit=false` 且 `lead=false` 时，后面的位不再受原数上界影响，答案只与「当前位置 + 其他约束」有关，可以复用。

### 1.4 模型一：windy 数（相邻数位差）

**问题**：不含前导零且相邻两位数字之差至少为 $2$ 的正整数称为 windy 数，求 $[A,B]$ 中 windy 数的个数。

**状态设计**：`dfs(pos, last, limit, lead)`：

- `pos`：当前位；
- `last`：上一位的数字（`lead=false` 时才有意义）；
- `limit`：是否贴着上界；
- `lead`：是否还在前导零阶段。

**转移**：枚举当前位 $d$，若 `lead` 且 $d=0$，继续递归且 `lead` 保持 true；否则要求 `abs(d - last) >= 2`，然后递归。

**边界**：`pos` 走完所有位时返回 1（说明构造出了一个合法数）。

**答案**：`f(B) - f(A-1)`。

### 1.5 模型二：不要 62（禁止子串）

**问题**：统计 $[L,R]$ 中不含数字 $4$、且不含连续子串 `62` 的数的个数。

**状态设计**：`dfs(pos, last, limit, lead)`，其中 `last` 记录上一位是否为 $6$。

**转移约束**：

- 当前位不能是 $4$；
- 若上一位是 $6$，当前位不能是 $2$。

**边界**：走完所有位返回 1。

这类「禁止某些子串/数字」的题是数位 DP 最基础的形态，约束直接写在枚举 $d$ 的循环里。

### 1.6 模型三：统计每个数字出现次数

**问题**：给定 $N$，统计 $1\sim N$ 中每个数字 $0\sim 9$ 分别出现了多少次。

**思路一（推荐）**：对每个数字 $k$ 单独做一次数位 DP，`dfs(pos, cnt, limit, lead)` 返回「在剩余位任意填写时，数字 $k$ 还能出现多少次」；把当前位选 $d$ 产生的贡献（`d == k` 时加 1）累加。

**思路二**：直接枚举每一位的贡献——固定某一位为 $k$，统计高位与低位的组合数。这种方法更快，但分类讨论多，容易漏掉前导零和上界情况。

无论哪种思路，答案都满足 $f(R)-f(L-1)$。

### 1.7 模型四：二进制数位 DP

数位 DP 不只用于十进制。若题目按**二进制**给条件（例如「二进制中 1 的个数不超过 $k$」），只需把进制换成 2，`up` 从 9 变成 1，其余框架完全一样。

```cpp
#include <bits/stdc++.h>
using namespace std;

long long N;
int K;
long long memo[65][65];                    // memo[pos][cnt]

// 统计 [0, N] 中二进制恰好有 K 个 1 的数的个数
long long dfs(int pos, int cnt, bool limit) {
    if (pos < 0) return cnt == K;
    if (!limit && memo[pos][cnt] != -1) return memo[pos][cnt];
    int up = limit ? (int)((N >> pos) & 1) : 1;
    long long res = 0;
    for (int d = 0; d <= up; ++d)
        res += dfs(pos - 1, cnt + (d == 1), limit && d == up);
    if (!limit) memo[pos][cnt] = res;
    return res;
}

int main() {
    cin >> N >> K;
    memset(memo, -1, sizeof(memo));
    int hi = 0;
    while ((1LL << (hi + 1)) <= N) ++hi;   // 最高位下标
    cout << dfs(hi, 0, true) << '\n';
    return 0;
}
```

**注意**：二进制版本里没有前导零问题（前导零就是 0，不额外占位），状态更简单。

---

## 2. 模板代码

### 2.1 通用记忆化搜索框架（以 windy 数为例）

```cpp
#include <bits/stdc++.h>
using namespace std;

int a[20];                                  // 拆位后的数字，从高到低
long long memo[20][10];                     // memo[pos][last]
bool vis[20][10];

// pos: 当前位；last: 上一位数字；limit: 是否贴上限；lead: 是否前导零
long long dfs(int pos, int last, bool limit, bool lead) {
    if (pos == 0) return lead ? 0 : 1;      // 构造完成；全前导零表示数字 0，不计入
    if (!limit && !lead && vis[pos][last]) return memo[pos][last];

    int up = limit ? a[pos] : 9;
    long long res = 0;
    for (int d = 0; d <= up; ++d) {
        if (lead && d == 0) {
            res += dfs(pos - 1, 0, limit && d == up, true);   // 仍是前导零
        } else {
            if (!lead && abs(d - last) < 2) continue;          // windy 数约束
            res += dfs(pos - 1, d, limit && d == up, false);
        }
    }
    if (!limit && !lead) { vis[pos][last] = true; memo[pos][last] = res; }
    return res;
}

long long solve(long long x) {              // 统计 [1, x] 的答案
    if (x <= 0) return 0;
    int len = 0;
    while (x) { a[++len] = x % 10; x /= 10; }   // 注意：低位存在 a[1]
    memset(vis, 0, sizeof(vis));
    return dfs(len, 0, true, true);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    long long A, B;
    cin >> A >> B;
    cout << solve(B) - solve(A - 1) << '\n';
    return 0;
}
```

> 说明：上面把数字**倒着**存进 `a[1..len]`（`a[1]` 是个位），于是 `dfs(len, ...)` 从最高位开始处理，`pos` 递减。这是最常见的写法，注意下标别搞反。

### 2.2 不要 62 的模板

```cpp
#include <bits/stdc++.h>
using namespace std;

int a[20];
long long memo[20][2];
bool vis[20][2];

// lastSix: 上一位是否是 6
long long dfs(int pos, bool lastSix, bool limit, bool lead) {
    if (pos == 0) return lead ? 0 : 1;      // 排除全前导零的数字 0
    if (!limit && !lead && vis[pos][lastSix]) return memo[pos][lastSix];

    int up = limit ? a[pos] : 9;
    long long res = 0;
    for (int d = 0; d <= up; ++d) {
        if (d == 4) continue;                        // 禁止 4
        if (lastSix && d == 2) continue;             // 禁止 62
        if (lead && d == 0)
            res += dfs(pos - 1, false, limit && d == up, true);
        else
            res += dfs(pos - 1, d == 6, limit && d == up, false);
    }
    if (!limit && !lead) { vis[pos][lastSix] = true; memo[pos][lastSix] = res; }
    return res;
}

long long solve(long long x) {
    if (x <= 0) return 0;
    int len = 0;
    while (x) { a[++len] = x % 10; x /= 10; }
    memset(vis, 0, sizeof(vis));
    return dfs(len, false, true, true);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    long long L, R;
    cin >> L >> R;
    cout << solve(R) - solve(L - 1) << '\n';
    return 0;
}
```

### 2.3 统计数字出现次数（逐数字版本）

```cpp
#include <bits/stdc++.h>
using namespace std;

int a[20];
long long memo[20][20];                     // memo[pos][cnt]
bool vis[20][20];
int target;                                 // 当前统计的数字

long long dfs(int pos, int cnt, bool limit, bool lead) {
    if (pos == 0) return cnt;               // 返回贡献而不是方案数
    if (!limit && !lead && vis[pos][cnt]) return memo[pos][cnt];

    int up = limit ? a[pos] : 9;
    long long res = 0;
    for (int d = 0; d <= up; ++d) {
        if (lead && d == 0) {
            res += dfs(pos - 1, cnt, limit && d == up, true);
        } else {
            res += dfs(pos - 1, cnt + (d == target), limit && d == up, false);
        }
    }
    if (!limit && !lead) { vis[pos][cnt] = true; memo[pos][cnt] = res; }
    return res;
}

long long solve(long long x, int k) {
    if (x <= 0) return 0;
    target = k;
    int len = 0;
    while (x) { a[++len] = x % 10; x /= 10; }
    memset(vis, 0, sizeof(vis));
    return dfs(len, 0, true, true);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    long long L, R;
    cin >> L >> R;
    for (int k = 0; k <= 9; ++k)
        cout << solve(R, k) - solve(L - 1, k) << (k == 9 ? '\n' : ' ');
    return 0;
}
```

---

## 3. 经典例题

### 例题 1：windy 数（洛谷 P2657 [SCOI2009]）

**题意**：不含前导零且相邻两位数字之差至少为 $2$ 的正整数叫 windy 数，求 $[A,B]$ 中 windy 数的个数，$1\le A\le B\le 2\times 10^9$。

**思路**：记忆化搜索，`dfs(pos, last, limit, lead)`，详见 2.1 节模板。

**复杂度**：状态数约 $19\times 10$，时间 $O(\log N\cdot 10\cdot 10)$，可视为常数。

### 例题 2：不要 62（HDU 2089）

**题意**：统计 $[L,R]$ 中不含数字 $4$、且不含子串 `62` 的数的个数，$1\le L\le R\le 10^9$。

**思路**：记忆化搜索，`lastSix` 记录上一位是否为 6，详见 2.2 节模板。

**复杂度**：时间 $O(\log N\cdot 2\cdot 10)$。

### 例题 3：数字计数（洛谷 P2602 [ZJOI2010]）

**题意**：给定 $L,R$（$1\le L\le R\le 10^{12}$），统计 $[L,R]$ 中每个数字 $0\sim 9$ 分别出现了多少次。

**思路**：对每个数字 $k$ 单独跑一次记忆化搜索，`dfs(pos, cnt, ...)` 返回「剩余位任意填时数字 $k$ 的总出现次数」。注意前导零不能被计入。

**复杂度**：时间 $O(10\cdot\log R\cdot\log R\cdot 10)$，空间 $O(\log R\cdot\log R)$。

> 完整代码见 [2.3 节模板](#23-统计数字出现次数逐数字版本)，只需把区间改为 `solve(R,k) - solve(L-1,k)` 即可。

### 例题 4：手机号码（洛谷 P4124 [CQOI2016]）

**题意**：统计 $[L,R]$ 中满足「至少出现三个连续相同数字」且「不同时出现 4 和 8」的 11 位手机号码个数。

**思路**：`dfs(pos, last, len, has4, has8, ok, limit, lead)`，其中 `len` 是当前连续相同数字的长度，`ok` 表示是否已经出现过三个连续相同数字。最终要求 `ok && !(has4 && has8)`。

**复杂度**：状态数约 $11\times 10\times 3\times 2\times 2\times 2$，非常小。

```cpp
#include <bits/stdc++.h>
using namespace std;

long long memo[15][11][2][2][2][2];
bool vis[15][11][2][2][2][2];
int a[15];

long long dfs(int pos, int last, int run, bool has4, bool has8, bool ok, bool limit, bool lead) {
    if (pos == 0) return ok && !(has4 && has8) && !lead;   // 排除前导零补出的短数
    if (!limit && !lead && vis[pos][last][has4][has8][ok][run >= 2])
        return memo[pos][last][has4][has8][ok][run >= 2];

    int up = limit ? a[pos] : 9;
    long long res = 0;
    for (int d = 0; d <= up; ++d) {
        bool nlead = lead && d == 0;
        int nrun = (nlead || d != last) ? 1 : run + 1;
        if (nlead) nrun = 0;
        res += dfs(pos - 1, d, nrun,
                   has4 || (!nlead && d == 4),
                   has8 || (!nlead && d == 8),
                   ok || (!nlead && nrun >= 3),
                   limit && d == up, nlead);
    }
    if (!limit && !lead && !vis[pos][last][has4][has8][ok][run >= 2]) {
        vis[pos][last][has4][has8][ok][run >= 2] = true;
        memo[pos][last][has4][has8][ok][run >= 2] = res;
    }
    return res;
}

long long solve(long long x) {
    if (x < 10000000000LL) return 0;        // 不足 11 位
    int len = 0;
    while (x) { a[++len] = x % 10; x /= 10; }
    memset(vis, 0, sizeof(vis));
    return dfs(len, 10, 0, false, false, false, true, true);
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    long long L, R;
    cin >> L >> R;
    cout << solve(R) - solve(L - 1) << '\n';
    return 0;
}
```

---

## 4. 易错点与练习建议

### 4.1 常见易错点

1. **区间转前缀时忘减一**：`f(R) - f(L-1)`，不是 `f(R) - f(L)`。
2. **前导零处理错误**：`lead` 为真时不能把当前位的 0 计入统计，也不能用它和 `last` 比较。
3. **`limit` 写进记忆化数组**：会导致状态数暴增且难以复用；正确做法是只在 `!limit && !lead` 时记忆化。
4. **拆位方向搞反**：`a[1]` 是个位、`a[len]` 是最高位时，`dfs` 必须从 `len` 开始。两种存法都行，但要全文统一。
5. **多组数据不清空 memo**：每次 `solve` 前都要清空 `vis`/`memo`，或让状态中带上「当前询问的编号」。
6. **$L=0$ 或 $L=1$ 的边界**：`solve(0)` 要返回 0，否则会算出负数或访问越界。
7. **贡献型 DP 的返回值**：统计出现次数时，`dfs` 返回的是「贡献」而不是「方案数」，边界写 `return cnt`。
8. **`long long` 与取模**：答案可能很大，若题目要取模，注意在每次加法处取模。

### 4.2 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P2657 windy 数 | 相邻数位约束、前导零 |
| 入门 | 洛谷 P2602 数字计数 | 贡献型数位 DP |
| 普及 | 洛谷 P4124 手机号码 | 多约束状态设计 |
| 普及 | 洛谷 P4999 烦人的数学作业 | 数位和取模 |
| 提高 | 洛谷 P3311 数数 | 数位 DP + AC 自动机 |
| 提高 | 洛谷 P3286 方伯伯的商场之旅 | 数位 DP + 贪心 |
| 提高 | 洛谷 P2518 计数 | 数位 DP + 组合数学 |

**训练方法**：

1. 先把「模板框架」抄三遍，直到能默写出 `dfs(pos, ..., limit, lead)` 的骨架；
2. 每道题先确定要记录哪些约束（上一位、数位和、是否出现过某模式），再决定 memo 的维度；
3. 用暴力枚举 $[1,10^6]$ 对拍，重点检查前导零、$L=1$、$R=10^k$ 等边界；
4. 注意数位 DP 的复杂度几乎都是「状态数 × 10」，状态设计对了就不会超时。
