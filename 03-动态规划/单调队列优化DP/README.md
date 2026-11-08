# 单调队列优化 DP

> 一句话概括：当转移形如 $f[i]=\min_{j\in[L_i,R_i]}\{f[j]+\mathrm{cost}(j)\}+g(i)$、且决策区间端点随 $i$ **单调右移**时，用双端队列维护候选项的单调性，把每次转移从 $O(n)$ 降到均摊 $O(1)$。

## 目录

1. [概念与原理](#一概念与原理)：滑动窗口最值、适用条件、运作过程、正确性与复杂度
2. [模板代码](#二模板代码)：滑动窗口本体、DP 通用框架、多重背包
3. [经典例题](#三经典例题)：滑动窗口、琪露诺、宝物筛选、Little Bird
4. [易错点](#四易错点)
5. [练习建议](#五练习建议)

---

## 一、概念与原理

### 1.1 从滑动窗口最值说起

给定长度 $n$ 的数组 $a$，对每个 $i$ 求

$$
m_i=\min\{a_j\mid j\in[i-k+1,\,i]\}.
$$

暴力每个窗口扫一遍是 $O(nk)$。观察：若 $j_1<j_2$ 且 $a_{j_1}\ge a_{j_2}$，则 $j_1$ 比 $j_2$ **更早离开**窗口，值又**更大**，因此 $j_1$ 永远不可能成为答案，可以永久删除。删完后剩下"下标递增、值也递增"的序列，这就是**单调队列**。

```
数组 a = [1, 3, -1, -3, 5, 3, 6, 7]，窗口 k = 3

i=1: 窗口 [1]          队列(下标,值): (1,1)
i=2: 窗口 [1,3]        队列: (1,1) (2,3)       ← 3 > 1，保留
i=3: 窗口 [1,3,-1]     队列: (3,-1)            ← -1 把 1、3 都弹掉
i=4: 窗口 [3,-1,-3]    队列: (4,-3)            ← -3 把 -1 弹掉
i=5: 窗口 [-1,-3,5]    队列: (4,-3) (5,5)      ← 1 已过期
i=6: 窗口 [-3,5,3]     队列: (4,-3) (6,3)      ← 5 比 3 大，弹掉
i=7: 窗口 [5,3,6]      队列: (7,6)             ← -3 过期，6 把 3 弹掉

每个时刻队首就是窗口最小值。
```

### 1.2 什么时候可以用单调队列优化

考虑

$$
f[i]=\min_{j\in[L_i,\,R_i]}\Bigl\{f[j]+\mathrm{cost}(j)\Bigr\}+g(i). \tag{1}
$$

括号里的量**只与 $j$ 有关**，与 $i$ 无关。令 $v(j)=f[j]+\mathrm{cost}(j)$，则 $f[i]=g(i)+\min\{v(j)\mid j\in[L_i,R_i]\}$，问题退化为滑动窗口最值。只需满足：

1. **决策区间端点单调不降**：$L_i\le L_{i+1}$，$R_i\le R_{i+1}$；
2. **窗口内的 $v(j)$ 一旦算出就不再变化**（$f[j]$ 已确定）；
3. 求的是窗口**最值**，且"更优"的定义与 $i$ 无关。

> **常见误区**：若转移含同时依赖 $i,j$ 的乘积项（如 $f[j]+a_i b_j$），$v(j)$ 不再只与 $j$ 有关，单调队列失效，需要斜率优化。

### 1.3 单调队列的运作过程

以取最小值为例，队列存**下标**，保证下标递增、对应 $v(j)$ 严格递增。每来一个 $i$：

1. **弹队首过期决策**：若 $q[\text{head}]<L_i$，则 `head++`；
2. **插入新决策**：从队尾不断弹出满足 $v(q[\text{tail}])\ge v(R_i)$ 的下标，再把 $R_i$ 放到队尾——因为 $R_i$ 更新且不劣，旧的那些永远没有机会；
3. 此时队首就是最优决策。

```
v 值:   5      3      4      2
下标:   1      2      3      4

插入 1:  [5]
插入 2:  [3]                    ← 5 ≥ 3，弹出 1
插入 3:  [3, 4]
插入 4:  [2]                    ← 4 ≥ 2、3 ≥ 2，全部弹出
```

### 1.4 正确性与复杂度

**正确性**：只需证明被弹出的元素永远不会成为之后任何窗口的答案。队首过期弹出显然安全；队尾弹出时，设被弹为 $p$、新入为 $q$，有 $p<q$ 且 $v(p)\ge v(q)$。对任意未来窗口 $[L,R]$，若 $p\in[L,R]$，则 $q>p\ge L$ 且 $q$ 更晚过期，必有 $q\in[L,R]$，又 $v(q)\le v(p)$，故 $p$ 不可能是唯一最优。

**复杂度**：每个下标最多入队一次、出队一次，总操作 $O(n)$，均摊 $O(1)$ 每次转移。

---

## 二、模板代码

### 2.1 滑动窗口最值（单调队列本体）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1000005;
int n, k, a[MAXN];
int q[MAXN];          // 队列存下标，不存值

int main() {
    scanf("%d%d", &n, &k);
    for (int i = 1; i <= n; ++i) scanf("%d", &a[i]);

    int head = 1, tail = 0;               // 手写双端队列，区间 [head, tail]
    for (int i = 1; i <= n; ++i) {
        while (head <= tail && q[head] < i - k + 1) ++head;   // 1. 过期
        while (head <= tail && a[q[tail]] >= a[i]) --tail;    // 2. 维护递增
        q[++tail] = i;
        if (i >= k) printf("%d ", a[q[head]]);                // 3. 输出
    }
    puts("");
    return 0;
}
```

### 2.2 单调队列优化 DP 通用框架

一般模型：决策区间 $[i-R,\,i-L]$，求最大值。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1000005;
const long long NEG = -(1LL << 60);

int n, L, R;
long long a[MAXN], f[MAXN];
int q[MAXN];

int main() {
    scanf("%d%d%d", &n, &L, &R);
    for (int i = 1; i <= n; ++i) scanf("%lld", &a[i]);

    for (int i = 1; i <= n; ++i) f[i] = NEG;
    f[0] = 0;

    int head = 1, tail = 0;
    for (int i = 1; i <= n; ++i) {
        if (i - L >= 0) {                                  // 新决策 j = i-L 入队
            while (head <= tail && f[q[tail]] <= f[i - L]) --tail;
            q[++tail] = i - L;
        }
        while (head <= tail && q[head] < i - R) ++head;    // 过期决策出队
        if (head <= tail) f[i] = f[q[head]] + a[i];        // 用队首转移
    }

    long long ans = NEG;
    for (int i = max(0, n - R + 1); i <= n; ++i) ans = max(ans, f[i]);
    printf("%lld\n", ans);
    return 0;
}
```

要点：**先让新决策入队，再弹过期决策，最后用队首转移**。顺序写反会误删新决策或漏解。

### 2.3 多重背包的单调队列优化

$n$ 种物品，第 $i$ 种重量 $v_i$、价值 $w_i$、数量 $c_i$，背包容量 $m$。朴素拆 01 背包是 $O(nm\sum c_i)$，二进制拆分 $O(nm\log c_i)$，单调队列可做到 $O(nm)$。

设 $f[j]$ 为容量 $j$ 的最大价值：

$$
f'[j]=\max_{0\le k\le c_i,\;kv_i\le j}\Bigl\{f[j-kv_i]+kw_i\Bigr\}.
$$

按余数 $r=j\bmod v_i$ 分组，令 $j=r+tv_i$、$s=t-k$：

$$
f'[r+tv_i]=\max_{s\in[t-c_i,\,t]}\Bigl\{f[r+sv_i]-s\,w_i\Bigr\}+t\,w_i.
$$

括号里又是"只与 $s$ 有关"的滑动窗口最大值。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXM = 100005;
int n, m;
long long f[MAXM], g[MAXM];   // f: 上一件物品，g: 当前物品
int q[MAXM];                  // 存 s

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; ++i) {
        int v, w, c;
        scanf("%d%d%d", &v, &w, &c);   // 体积、价值、数量
        if (c > m / v) c = m / v;      // 实际最多只能放这么多件

        for (int r = 0; r < v; ++r) {
            int head = 1, tail = 0;
            for (int t = 0; r + t * v <= m; ++t) {
                while (head <= tail && q[head] < t - c) ++head;   // 过期 s < t-c

                long long cur = f[r + 1LL * t * v] - 1LL * t * w;
                while (head <= tail) {                             // 队尾不优则弹出
                    int s = q[tail];
                    long long old = f[r + 1LL * s * v] - 1LL * s * w;
                    if (old <= cur) --tail;
                    else break;
                }
                q[++tail] = t;

                int best = q[head];
                g[r + 1LL * t * v] = f[r + 1LL * best * v] - 1LL * best * w + 1LL * t * w;
            }
        }
        memcpy(f, g, sizeof(f));   // 滚动到下一件物品
    }
    printf("%lld\n", f[m]);
    return 0;
}
```

`g` 必须全部算完再整体拷回 `f`，否则同一件物品会被重复选取。

---

## 三、经典例题

### 例题 1：滑动窗口

**题意**：给定长为 $n$ 的数组和窗口长度 $k$，依次输出每个长度为 $k$ 的连续子区间的最小值和最大值。

**思路**：模板题。一个队列维护递增求最小值，另一个维护递减求最大值；每个下标最多进出队一次。

**复杂度**：$O(n)$ 时间，$O(n)$ 空间。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, k;
    scanf("%d%d", &n, &k);
    vector<int> a(n + 1), q(n + 1);
    for (int i = 1; i <= n; ++i) scanf("%d", &a[i]);

    int head = 1, tail = 0;                     // 最小值：维护单调递增
    for (int i = 1; i <= n; ++i) {
        while (head <= tail && q[head] < i - k + 1) ++head;
        while (head <= tail && a[q[tail]] >= a[i]) --tail;
        q[++tail] = i;
        if (i >= k) printf("%d ", a[q[head]]);
    }
    puts("");

    head = 1, tail = 0;                         // 最大值：维护单调递减
    for (int i = 1; i <= n; ++i) {
        while (head <= tail && q[head] < i - k + 1) ++head;
        while (head <= tail && a[q[tail]] <= a[i]) --tail;
        q[++tail] = i;
        if (i >= k) printf("%d ", a[q[head]]);
    }
    puts("");
    return 0;
}
```

### 例题 2：琪露诺

**题意**：数轴 $0\sim n$ 共 $n+1$ 个位置，位置 $i$（$1\le i\le n$）有分数 $a_i$。从 $0$ 出发，每次向右跳 $[L,R]$ 步，跳出 $n$ 视为结束，求最大分数和。

**思路**：设 $f[i]$ 为跳到 $i$ 时的最大分数：

$$
f[i]=a_i+\max_{j\in[i-R,\,i-L]}f[j].
$$

标准滑动窗口最大值，按 $i$ 递增计算，单调队列维护 $f$ 递减。

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 200005;
const long long NEG = -(1LL << 60);

int n, L, R, q[MAXN];
long long a[MAXN], f[MAXN];

int main() {
    scanf("%d%d%d", &n, &L, &R);
    for (int i = 1; i <= n; ++i) scanf("%lld", &a[i]);
    for (int i = 1; i <= n; ++i) f[i] = NEG;
    f[0] = 0;

    int head = 1, tail = 0;
    for (int i = 1; i <= n; ++i) {
        if (i - L >= 0) {                        // 新决策 j = i-L 可用
            while (head <= tail && f[q[tail]] <= f[i - L]) --tail;
            q[++tail] = i - L;
        }
        while (head <= tail && q[head] < i - R) ++head;
        if (head <= tail) f[i] = f[q[head]] + a[i];
    }

    long long ans = NEG;
    for (int i = max(0, n - R + 1); i <= n; ++i) ans = max(ans, f[i]);
    printf("%lld\n", ans);
    return 0;
}
```

### 例题 3：宝物筛选

**题意**：$n$ 种宝物，第 $i$ 种重量 $v_i$、价值 $w_i$、数量 $c_i$，背包承重 $m$，求最大总价值。

**思路**：多重背包。按重量余数分组后每组变成滑动窗口最大值，单调队列 $O(m)$ 处理一种物品。

**复杂度**：$O(nm)$ 时间，$O(m)$ 空间。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    scanf("%d%d", &n, &m);       // 宝物种数、背包承重

    vector<long long> f(m + 1, 0), g(m + 1, 0);
    vector<int> q(m + 1);

    for (int i = 1; i <= n; ++i) {
        int v, w, c;
        scanf("%d%d%d", &v, &w, &c);      // 重量、价值、数量
        if (c > m / v) c = m / v;

        for (int r = 0; r < v; ++r) {
            int head = 1, tail = 0;
            for (int t = 0; r + t * v <= m; ++t) {
                while (head <= tail && q[head] < t - c) ++head;

                long long cur = f[r + 1LL * t * v] - 1LL * t * w;
                while (head <= tail) {
                    int s = q[tail];
                    long long old = f[r + 1LL * s * v] - 1LL * s * w;
                    if (old <= cur) --tail;
                    else break;
                }
                q[++tail] = t;

                int best = q[head];
                g[r + 1LL * t * v] = f[r + 1LL * best * v] - 1LL * best * w + 1LL * t * w;
            }
        }
        f.swap(g);
    }
    printf("%lld\n", f[m]);
    return 0;
}
```

### 例题 4：Little Bird

**题意**：$n$ 棵树排成一排，第 $i$ 棵高 $h_i$。从第 $1$ 棵出发，每次最多向右跳 $k$ 棵。跳到高度不低于当前树的树上代价为 $1$，否则为 $0$，求到达第 $n$ 棵的最小代价。

**思路**：

$$
f[j]=\min_{i\in[j-k,\,j-1]}\Bigl\{f[i]+[h_j\ge h_i]\Bigr\}.
$$

窗口内取最小值，但"谁更优"依赖 $h$：$f$ 相同时 $h$ **更高**的决策更优（更容易不收费）。比较函数写成 `f[a] < f[b] || (f[a] == f[b] && h[a] >= h[b])`。

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1000005;
int n, k, T, h[MAXN], f[MAXN], q[MAXN];

bool better(int a, int b) {          // 决策 a 是否不劣于 b
    if (f[a] != f[b]) return f[a] < f[b];
    return h[a] >= h[b];             // f 相同时 h 越大越优
}

int main() {
    scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; ++i) scanf("%d", &h[i]);
        scanf("%d", &k);

        f[1] = 0;
        int head = 1, tail = 0;
        q[++tail] = 1;

        for (int j = 2; j <= n; ++j) {
            while (head <= tail && better(j - 1, q[tail])) --tail;   // 加入新决策
            q[++tail] = j - 1;
            while (head <= tail && q[head] < j - k) ++head;          // 移除过期
            f[j] = f[q[head]] + (h[j] >= h[q[head]] ? 1 : 0);
        }
        printf("%d\n", f[n]);
    }
    return 0;
}
```

> 本题难点是"比较函数不是简单的 $f$ 大小"。写单调队列前**先想清楚支配关系**：若决策 $a$ 在任何未来情况下都不比 $b$ 差，才能弹掉 $b$；想不清楚就用 $O(nk)$ 暴力对拍。

---

## 四、易错点

1. **队列存值而不是下标。** 判断过期必须用下标，存值无法知道它是否还在窗口内。
2. **过期判断差一位。** 窗口是 $[i-k+1,i]$，应写 `q[head] < i-k+1`；建议画图确认边界。
3. **先弹过期还是先插入。** 对"新决策 $i-L$、过期界 $i-R$"的模型，先插入再弹过期更安全，否则可能误删刚插入的决策。
4. **忘记初始化。** $f[0]=0$，其余位置求最大设 $-\infty$、求最小设 $+\infty$，否则不可达状态会污染答案。
5. **多重背包滚动数组写错。** 必须先把当前物品的所有 $g$ 算完，再整体覆盖 $f$；边算边覆盖会导致同一物品被无限次选取。
6. **多重背包数量未截断。** $c_i>m/v_i$ 时应令 $c_i=m/v_i$。
7. **比较函数不满足传递性。** 单调队列要求"更优"关系可传递；自定义比较后一定用暴力对拍验证。
8. **硬套端点不单调的窗口。** 若 $L_i$ 或 $R_i$ 会来回移动，单调队列失效，需换线段树、ST 表等。

## 五、练习建议

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1886 滑动窗口 | 单调队列模板 |
| 入门 | 洛谷 P1440 求 m 区间内的最小值 | 单调队列模板 |
| 基础 | 洛谷 P1725 琪露诺 | 区间最值转移 |
| 基础 | 洛谷 P2034 选择数字 | 单调队列 + 补集转化 |
| 进阶 | 洛谷 P1776 宝物筛选 | 多重背包单调队列优化 |
| 进阶 | 洛谷 P3572 [POI2014] PTA-Little Bird | 自定义比较的单调队列 |
| 进阶 | 洛谷 P3957 跳房子 | 二分答案 + 单调队列 |
| 挑战 | 洛谷 P2569 [SCOI2010] 股票交易 | 单调队列优化复杂转移 |
| 挑战 | 洛谷 P5665 [CSP-S2019] 划分 | 单调队列 + 贪心结论 |

**刷题方法**：先写转移方程，明确"决策区间是什么、$v(j)$ 是什么"再决定用不用单调队列；手动画队列变化确认弹出规则；写完 $O(n)$ 做法后一定用 $O(nk)$ 暴力对拍，尤其是自定义比较函数的题；端点不单调时换思路，不要硬套。
