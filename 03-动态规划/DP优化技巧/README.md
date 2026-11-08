# DP 优化技巧

> 一句话概括：DP 优化的本质是"**让状态更少、让转移更快**"——从状态定义、枚举顺序、数据结构、代数变形四个方向入手，把朴素做法里冗余的计算砍掉。

## 目录

1. [概念与原理](#一概念与原理)：复杂度来源、滚动数组、记忆化、前缀和、决策单调性、数据结构、倍增
2. [模板代码](#二模板代码)：01 背包滚动、数位 DP 记忆化、前缀和优化、分治优化
3. [经典例题](#三经典例题)：最大子段和、编辑距离、石子合并、最长上升子序列
4. [易错点](#四易错点)
5. [练习建议](#五练习建议)

---

## 一、概念与原理

### 1.1 DP 复杂度的两个来源

DP 复杂度通常写作 $O(\text{状态数}\times\text{单次转移代价})$，所以优化只有两条路：**减少状态数**（合并等价状态、滚动数组、压维、离散化、记忆化）与**减少转移代价**（前缀和、单调队列、斜率优化、数据结构、决策单调性、矩阵快速幂）。

```
                      ┌─ 状态太多 ─┬─ 滚动数组（只依赖前一层）
                      │            ├─ 记忆化搜索（有效状态稀疏）
   DP 太慢 ───────────┤            └─ 压维 / 离散化 / 等价类合并
                      │
                      └─ 转移太慢 ─┬─ 转移量可前缀和 ──► 前缀和/差分
                                   ├─ 窗口最值 ────────► 单调队列
                                   ├─ 线性乘积项 ──────► 斜率优化
                                   ├─ 区间最值/计数 ───► 线段树/树状数组
                                   ├─ 决策单调 ────────► 分治/二分栈
                                   ├─ 固定系数递推 ────► 矩阵快速幂
                                   └─ 重复子问题 ──────► 倍增预处理
```

### 1.2 滚动数组与压维

若 $f[i][\cdot]$ 只依赖 $f[i-1][\cdot]$（或最近常数层），就不必保存所有层。**关键：注意枚举方向。** 以 01 背包

$$
f[j]=\max\bigl(f[j],\ f[j-w_i]+v_i\bigr)
$$

为例：$j$ **从大到小**枚举时 $f[j-w_i]$ 还是上一件物品的值，每件物品最多选一次；$j$ **从小到大**枚举时 $f[j-w_i]$ 可能已被本件物品更新，等价于完全背包。同一个转移式，枚举方向不同，含义完全不同。

```
01 背包（j 递减）:  f[5] ← f[3](旧)  每件物品只用一次
完全背包（j 递增）: f[5] ← f[3](新)  同一件物品可重复用
  递减 │←───────────────┘ 从右往左，读到旧层
  递增 │└───────────────→ 从左往右，读到新层
```

### 1.3 记忆化搜索与有效状态

有些 DP 状态空间看似很大，但真正被访问的状态很少，这时用**记忆化搜索**（DFS + 缓存）只计算可达状态，往往比递推快得多：递归函数进入时先查缓存，未计算则按转移递归子状态，返回前把结果写入缓存。数位 DP 是典型例子：状态为"当前位、是否贴紧上界、是否有前导零"，递推要枚举所有组合，记忆化只走实际出现的分支。注意记忆化要求状态能作为数组下标或哈希键。

### 1.4 前缀和 / 差分优化

若转移是"对一段区间求和/求最值"且端点随 $i$ 单调，可用前缀和把 $O(n)$ 枚举降到 $O(1)$。例如 $f[i]=\sum_{j=L_i}^{i-1}f[j]$，令 $S[i]=\sum_{k\le i}f[k]$，则 $f[i]=S[i-1]-S[L_i-1]$。

二维前缀和：

$$
S[i][j]=S[i-1][j]+S[i][j-1]-S[i-1][j-1]+a[i][j],
$$

子矩阵和 $=S[x_2][y_2]-S[x_1-1][y_2]-S[x_2][y_1-1]+S[x_1-1][y_1-1]$。

### 1.5 决策单调性

设 $f[i]=\min_{j<i}\{f[j]+w(j,i)\}$，若最优决策点 $\mathrm{opt}(i)$ 随 $i$ **单调不降**，称该 DP 具有**决策单调性**。常见充分条件是 $w$ 满足**四边形不等式**：

$$
w(a,c)+w(b,d)\le w(a,d)+w(b,c)\qquad(a\le b\le c\le d).
$$

此时可用两种方法：

**（1）分治优化**（要求转移是"分层"的，即由上一层的 $f$ 推下一层的 $g$）：

```cpp
void solve(int l, int r, int optL, int optR) {
    if (l > r) return;
    int mid = (l + r) >> 1, best = -1;
    long long bestVal = INF;
    for (int j = optL; j <= min(mid - 1, optR); ++j) {
        long long v = f[j] + w(j, mid);
        if (v < bestVal) { bestVal = v; best = j; }
    }
    g[mid] = bestVal;                    // 写入下一层
    solve(l, mid - 1, optL, best);       // 左半边的决策点不超过 best
    solve(mid + 1, r, best, optR);       // 右半边的决策点不小于 best
}
```

**（2）二分栈**：适用于同层转移，每个决策"称霸"一段连续的 $i$（如 j=1 管 $[1,3]$、j=2 管 $[4,7]$），用栈维护这些区间，插入新决策时二分找交点，能抢就弹出栈尾。

### 1.6 数据结构辅助转移

若转移是"在若干点中查询满足某条件的最值"，把点插入数据结构即可：

| 转移形式 | 数据结构 |
|---|---|
| $f[i]=\min_{j<i,a_j<a_i}\{f[j]\}$ | 树状数组（以 $a$ 为下标维护前缀最值） |
| $f[i]=\max_{j\in[l,r]}\{f[j]\}$ | 线段树 / ST 表 |
| $f[i]=\min_j\{f[j]+|x_i-x_j|\}$ | 拆绝对值 + 树状数组 |
| $f[i]=\max_j\{f[j]+k_j x_i\}$ | 李超线段树 / 凸包 |

例：树状数组优化 LIS 计数，把 $a_i$ 离散化后以值域为下标，`query(x)` 求前缀最大值、`update(x,v)` 更新，均摊 $O(\log n)$。

### 1.7 倍增与预处理

对"重复询问同一个 DP 子结构"的问题，可预处理倍增表。例如**倍增求 LCA**：$up[k][u]$ 表示 $u$ 向上跳 $2^k$ 步的祖先，$up[k][u]=up[k-1][up[k-1][u]]$，预处理 $O(n\log n)$、每次询问 $O(\log n)$。若转移是"从 $i$ 跳 $2^k$ 步后的最优值"，也可类似预处理 $g[k][i]$，把链上跳跃从 $O(n)$ 降到 $O(\log n)$。

### 1.8 其他手段速览

滚动数组把空间从 $O(nk)$ 降到 $O(k)$（背包、LCS）；状态压缩把指数状态编码成整数（状压 DP）；离散化压缩值域（坐标相关 DP）；提前终止与剪枝减少无效枚举（搜索型 DP）。这些手段常与前几节组合使用。

---

## 二、模板代码

### 2.1 滚动数组：01 背包

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    scanf("%d%d", &n, &m);              // 物品数、背包容量
    vector<int> f(m + 1, 0);            // f[j]: 容量 j 的最大价值
    for (int i = 1; i <= n; ++i) {
        int w, v;
        scanf("%d%d", &w, &v);          // 重量、价值
        for (int j = m; j >= w; --j)    // 必须倒序：每件物品只用一次
            f[j] = max(f[j], f[j - w] + v);
    }
    printf("%d\n", f[m]);
    return 0;
}
```

### 2.2 记忆化搜索：数位 DP 框架

求 $[1,n]$ 中不含数字 $4$ 的数的个数。

```cpp
#include <bits/stdc++.h>
using namespace std;

int digit[20], len;
int memo[20][2];        // [位置][是否贴紧上界]
bool vis[20][2];

int dfs(int pos, bool tight) {
    if (pos == len) return 1;               // 填完，得到一个合法数（含全 0）
    if (vis[pos][tight]) return memo[pos][tight];
    vis[pos][tight] = true;

    int up = tight ? digit[pos] : 9, res = 0;
    for (int d = 0; d <= up; ++d) {
        if (d == 4) continue;               // 不能含 4
        res += dfs(pos + 1, tight && (d == up));
    }
    return memo[pos][tight] = res;
}

int solve(int n) {
    if (n <= 0) return 0;
    len = 0;
    while (n) { digit[len++] = n % 10; n /= 10; }
    reverse(digit, digit + len);            // 高位在前
    memset(vis, 0, sizeof(vis));
    return dfs(0, true) - 1;                // 减去全 0 这个数
}

int main() {
    int n;
    scanf("%d", &n);
    printf("%d\n", solve(n));
    return 0;
}
```

### 2.3 前缀和优化：区间和转移

$f[i]=\sum_{j=\max(0,i-k)}^{i-1}f[j]$，$f[0]=1$，求 $f[n]\bmod 10^9+7$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1000005;
const long long MOD = 1000000007LL;
long long f[MAXN], S[MAXN];   // S[i] = f[0] + ... + f[i]

int main() {
    int n, k;
    scanf("%d%d", &n, &k);
    f[0] = 1;
    S[0] = 1;
    for (int i = 1; i <= n; ++i) {
        int l = max(0, i - k);
        f[i] = S[i - 1] - (l > 0 ? S[l - 1] : 0);   // f[i] = S[i-1] - S[l-1]
        f[i] = (f[i] % MOD + MOD) % MOD;
        S[i] = (S[i - 1] + f[i]) % MOD;
    }
    printf("%lld\n", f[n]);
    return 0;
}
```

### 2.4 决策单调性：分治优化

把 $n$ 个数分成 $K$ 段，每段费用为段内元素和的平方，求最小总费用：

$$
f[t][i]=\min_{j<i}\Bigl\{f[t-1][j]+(S_i-S_j)^2\Bigr\}.
$$

费用满足四边形不等式，决策点单调，可用分治优化。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXN = 3005;
const ll INF = (1LL << 60);

int n, K;
ll a[MAXN], S[MAXN], f[MAXN][MAXN];

inline ll cost(int j, int i) {          // 把 (j, i] 作为一段的费用
    ll s = S[i] - S[j];
    return s * s;
}

void solve(int t, int l, int r, int optL, int optR) {
    if (l > r) return;
    int mid = (l + r) >> 1, best = -1;
    ll bestVal = INF;
    int upper = min(mid - 1, optR);
    for (int j = optL; j <= upper; ++j) {
        ll v = f[t - 1][j] + cost(j, mid);
        if (v < bestVal) { bestVal = v; best = j; }
    }
    f[t][mid] = bestVal;
    solve(t, l, mid - 1, optL, best);
    solve(t, mid + 1, r, best, optR);
}

int main() {
    scanf("%d%d", &n, &K);
    for (int i = 1; i <= n; ++i) {
        scanf("%lld", &a[i]);
        S[i] = S[i - 1] + a[i];
    }
    for (int i = 1; i <= n; ++i) f[1][i] = cost(0, i);
    for (int t = 2; t <= K; ++t) solve(t, 1, n, 0, n);
    printf("%lld\n", f[K][n]);
    return 0;
}
```

---

## 三、经典例题

### 例题 1：最大子段和

**题意**：长度为 $n$ 的整数序列，求一段连续非空子段的最大和。$n\le 2\times 10^5$。

**思路**：设 $f[i]$ 为"以 $i$ 结尾的最大子段和"，则 $f[i]=\max(f[i-1],0)+a_i$，答案 $\max_i f[i]$。只依赖前一维，可只用一个变量滚动，空间 $O(1)$。

**复杂度**：$O(n)$ 时间，$O(1)$ 空间。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    scanf("%d", &n);
    long long cur = 0, ans = LLONG_MIN;
    for (int i = 0; i < n; ++i) {
        long long x;
        scanf("%lld", &x);
        cur = max(cur, 0LL) + x;      // 滚动变量代替 f[i]
        ans = max(ans, cur);
    }
    printf("%lld\n", ans);
    return 0;
}
```

### 例题 2：编辑距离

**题意**：给定字符串 $A,B$，每次可插入、删除或替换一个字符，求把 $A$ 变成 $B$ 的最少操作次数。$|A|,|B|\le 5000$。

**思路**：设 $f[i][j]$ 为 $A$ 前 $i$ 个字符变成 $B$ 前 $j$ 个字符的最少操作数：

$$
f[i][j]=
\begin{cases}
f[i-1][j-1], & A_i=B_j,\\
1+\min\{f[i-1][j],\ f[i][j-1],\ f[i-1][j-1]\}, & A_i\ne B_j.
\end{cases}
$$

边界 $f[i][0]=i$、$f[0][j]=j$。$f[i][\cdot]$ 只依赖 $f[i-1][\cdot]$，可滚动成两行。

**复杂度**：$O(|A||B|)$ 时间。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    char A[5005], B[5005];
    scanf("%s%s", A + 1, B + 1);
    int n = strlen(A + 1), m = strlen(B + 1);
    vector<int> pre(m + 1), cur(m + 1);

    for (int j = 0; j <= m; ++j) pre[j] = j;   // f[0][j]
    for (int i = 1; i <= n; ++i) {
        cur[0] = i;                             // f[i][0]
        for (int j = 1; j <= m; ++j) {
            if (A[i] == B[j]) cur[j] = pre[j - 1];
            else cur[j] = 1 + min({pre[j], cur[j - 1], pre[j - 1]});
        }
        pre.swap(cur);                          // 滚动
    }
    printf("%d\n", pre[m]);
    return 0;
}
```

### 例题 3：石子合并（四边形不等式）

**题意**：$n$ 堆石子排成一排，每次合并相邻两堆，代价为两堆石子数之和，求合并成一堆的最小总代价。$n\le 3000$。

**思路**：设 $f[i][j]$ 为合并区间 $[i,j]$ 的最小代价，$S$ 为前缀和：

$$
f[i][j]=\min_{i\le k<j}\bigl\{f[i][k]+f[k+1][j]\bigr\}+S_j-S_{i-1}.
$$

朴素枚举 $k$ 是 $O(n^3)$。代价满足四边形不等式，最优决策点满足

$$
\mathrm{opt}[i][j-1]\le \mathrm{opt}[i][j]\le \mathrm{opt}[i+1][j],
$$

枚举范围从 $[i,j)$ 缩到 $[\mathrm{opt}[i][j-1],\mathrm{opt}[i+1][j]]$，总复杂度 $O(n^2)$。

**复杂度**：$O(n^2)$ 时间与空间。环形版本把数组复制一倍接在后面，答案取 $\min_{1\le i\le n}f[i][i+n-1]$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 3005;
const long long INF = (1LL << 60);
int n;
long long a[MAXN], S[MAXN], f[MAXN][MAXN];
int opt[MAXN][MAXN];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) {
        scanf("%lld", &a[i]);
        S[i] = S[i - 1] + a[i];
    }
    for (int i = 1; i <= n; ++i) { f[i][i] = 0; opt[i][i] = i; }

    for (int len = 2; len <= n; ++len)
        for (int i = 1; i + len - 1 <= n; ++i) {
            int j = i + len - 1;
            f[i][j] = INF;
            int L = opt[i][j - 1], R = opt[i + 1][j];
            for (int k = L; k <= R && k < j; ++k) {   // 利用决策单调性缩范围
                long long v = f[i][k] + f[k + 1][j];
                if (v < f[i][j]) { f[i][j] = v; opt[i][j] = k; }
            }
            f[i][j] += S[j] - S[i - 1];
        }
    printf("%lld\n", f[1][n]);
    return 0;
}
```

### 例题 4：最长上升子序列

**题意**：长度为 $n$ 的序列，求最长严格上升子序列长度。$n\le 10^5$。

**思路**：朴素 DP 是 $f[i]=1+\max_{j<i,a_j<a_i}f[j]$，$O(n^2)$。优化有两种：树状数组按值域维护前缀最大值（$O(n\log n)$）；或贪心 + 二分——维护 $d[l]$ 表示长度为 $l$ 的上升子序列的最小结尾，对每个 $a_i$ 二分替换第一个 $\ge a_i$ 的位置。

**复杂度**：$O(n\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    scanf("%d", &n);
    vector<int> d;                      // d[l]: 长度 l+1 的 LIS 的最小结尾
    for (int i = 0; i < n; ++i) {
        int x;
        scanf("%d", &x);
        // 严格上升用 lower_bound；非严格上升用 upper_bound
        auto it = lower_bound(d.begin(), d.end(), x);
        if (it == d.end()) d.push_back(x);
        else *it = x;
    }
    printf("%d\n", (int)d.size());
    return 0;
}
```

---

## 四、易错点

1. **滚动数组枚举方向反了。** 01 背包必须倒序枚举容量，完全背包必须正序；写错会导致语义颠倒。
2. **压维后忘记保存旧值。** 若依赖 $f[i-1][j-1]$ 与 $f[i][j-1]$，压成一维时要额外变量保存 $f[i-1][j-1]$。
3. **记忆化忘记标记访问。** 只写 `memo[s]` 而不用 `vis` 区分"未计算"和"结果为 0"，会重复搜索甚至死循环。
4. **决策单调性未经证明就使用。** 四边形不等式是充分条件，不是所有区间 DP 都满足；用前先验证或对拍。
5. **分治优化的区间边界写错。** 左半边的决策点 $\le$ best，右半边 $\ge$ best，差一位就可能漏掉最优解。
6. **前缀和数组越界。** $S[0]=0$，查询 $[l,r]$ 用 $S[r]-S[l-1]$，当 $l=0$ 时不能用 $S[-1]$。
7. **树状数组最值初始值错误。** 求最大初始化为 $-\infty$，求最小初始化为 $+\infty$，初始化为 0 会在负数数据上出错。
8. **优化后没有对拍。** 任何优化都可能引入边界错误，务必用 $O(n^2)$ 或 $O(n^3)$ 的暴力在小数据上对拍。分治优化常被误认为 $O(n\log n)$，实际是 $O(n\log^2 n)$，复杂度要按实际枚举量分析。

## 五、练习建议

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1115 最大子段和 | 滚动变量 |
| 入门 | 洛谷 P1216 数字三角形 | 滚动数组 |
| 基础 | 洛谷 P2758 编辑距离 | 二维 DP + 滚动 |
| 基础 | 洛谷 P1439 最长公共子序列 | LIS 转化 + 二分 |
| 基础 | 洛谷 P1020 导弹拦截 | LIS + 贪心二分 |
| 进阶 | 洛谷 P1880 石子合并 | 区间 DP + 四边形不等式 |
| 进阶 | 洛谷 P2602 数字计数 | 数位 DP 记忆化 |
| 进阶 | 洛谷 P3287 [SCOI2014] 方伯伯的玉米田 | 树状数组优化 DP |
| 挑战 | 洛谷 P1912 [NOI2009] 诗人小G | 决策单调性 + 二分栈 |
| 挑战 | 洛谷 P3515 [POI2011] Lightning Conductor | 决策单调性 + 分治 |

**刷题方法**：先写**朴素 DP**并确认正确，再考虑优化；优化的第一步是写出复杂度瓶颈——是状态多还是转移慢，对症下药；每次优化后用暴力对拍至少 100 组 $n\le 10$ 的随机数据；记住常见"套路对应关系"：窗口最值→单调队列，乘积项→斜率优化，区间和→前缀和，带条件最值→树状数组/线段树，固定系数递推→矩阵快速幂；多总结"为什么可以优化"，通常是因为决策之间存在支配关系或子问题被重复计算，理解了本质才能举一反三。
