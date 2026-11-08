# 分块（Sqrt Decomposition / Block Decomposition）

> 一句话概括：**分块把序列切成 $\sqrt{n}$ 个长度为 $\sqrt{n}$ 的块，整块打标记、零散元素暴力处理**，用「$\sqrt{n}$ 的暴力」换取代码简单与极强的可扩展性，是「线段树写不动」时的万能替代方案。

---

## 目录

1. [什么是分块](#1-什么是分块)
2. [核心思想与结构](#2-核心思想与结构)
3. [区间修改与区间查询](#3-区间修改与区间查询)
4. [完整模板代码](#4-完整模板代码)
5. [进阶：分块的其他形态](#5-进阶分块的其他形态)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [复杂度与常数分析](#8-复杂度与常数分析)
9. [练习建议](#9-练习建议)

---

## 1. 什么是分块

### 1.1 从「线段树写不动」说起

线段树很强，但它要求信息**可合并**，而且懒标记的叠加顺序、`pushDown` 的时机很容易写错。遇到下面这些情况，线段树会很难受：

- 要维护的信息很怪异（如「区间内大于 $x$ 的数有多少个」）；
- 修改操作不是简单的加减（如「区间开根号」「区间取模」）；
- 需要支持「区间内某值的出现次数」这类无法 $O(1)$ 合并的信息。

**分块**用最朴素的办法解决这些问题：把序列切成若干块，**整块的信息预处理好，零散部分直接暴力**。

### 1.2 一个直观的例子

假设 $n = 10$，块长 $B = 3$：

```
下标:  1  2  3 | 4  5  6 | 7  8  9 | 10
数据:  a1 a2 a3| a4 a5 a6| a7 a8 a9| a10
块号:   1  1  1 |  2  2  2 |  3  3  3 |  4
```

- 块 1 覆盖 $[1,3]$，块 2 覆盖 $[4,6]$，块 3 覆盖 $[7,9]$，块 4 只有 $[10,10]$；
- 查询 $[2,8]$ 时：$2,3$ 是零散元素（暴力），$[4,6]$、$[7,9]$ 中的 $[7,8]$ 是部分块，需要小心处理；
- 查询 $[4,9]$ 时：恰好是完整的块 2 和块 3，直接用块内预处理答案。

### 1.3 核心权衡

| 做法 | 单次操作复杂度 | 代码难度 | 适用性 |
|---|---|---|---|
| 纯暴力 | $O(n)$ | 极简单 | 小数据 |
| 线段树 | $O(\log n)$ | 中等偏难 | 信息可合并 |
| **分块** | $O(\sqrt{n})$ | **简单** | **几乎万能** |

分块的精髓：**用 $\sqrt{n}$ 的复杂度，换取「想怎么暴力就怎么暴力」的自由**。当 $n, m \le 10^5$ 时，$m\sqrt{n} \approx 3 \times 10^7$，卡一卡常就能过。

---

## 2. 核心思想与结构

### 2.1 块长怎么选

设块长为 $B$，块数为 $\lceil n/B \rceil$。一次区间操作：

- 零散元素最多 $2B$ 个，暴力 $O(B)$；
- 完整块最多 $n/B$ 个，每块 $O(1)$ 处理。

总复杂度 $O(B + n/B)$，由均值不等式，当 $B = \sqrt{n}$ 时取最小值 $O(\sqrt{n})$。

> 实践技巧：块长常取 $\sqrt{n}$ 或 $\sqrt{n \log n}$；如果修改操作重、查询轻，可以适当调大 $B$。

### 2.2 需要维护的信息

```cpp
const int MAXN = 100005;
const int MAXB = 405;          // 块数上限，约 sqrt(MAXN)

int n, m, B, cnt;              // cnt = 块数
int a[MAXN];                   // 原数组（1 起始）
int L[MAXB], R[MAXB];          // 每块的左右端点
int bel[MAXN];                 // 每个下标所属的块号
long long sum[MAXB];           // 每块的元素和
long long tag[MAXB];           // 每块的加法懒标记
```

### 2.3 预处理

```cpp
void build() {
    B = max(1, (int)sqrt(n));      // 块长
    cnt = (n + B - 1) / B;         // 块数
    for (int i = 1; i <= cnt; ++i) {
        L[i] = (i - 1) * B + 1;
        R[i] = min(n, i * B);
    }
    for (int i = 1; i <= n; ++i) {
        bel[i] = (i - 1) / B + 1;
        sum[bel[i]] += a[i];
    }
}
```

---

## 3. 区间修改与区间查询

### 3.1 区间加

以「区间每个数加 $v$」为例，处理区间 $[l, r]$：

1. 若 $l$ 与 $r$ 在**同一块**：直接暴力修改 $a[l..r]$，同步更新 `sum`；
2. 否则：
   - 暴力修改 $l$ 到 $l$ 所在块的右端点；
   - 暴力修改 $r$ 所在块的左端点到 $r$；
   - 中间完整的块：`tag[块] += v`，`sum[块] += v * 块长度`。

```
区间 [2, 8]，B = 3：

块1 [1,3]   块2 [4,6]   块3 [7,9]
   ↑零散        ↑整块        ↑零散
 [2,3]        [4,6]        [7,8]
 暴力改        打标记        暴力改
```

### 3.2 区间求和

同理：同一块暴力累加，否则零散部分暴力、完整块直接累加 `sum[块]`。注意零散元素的真实值是 $a[i] + tag[bel[i]]$。

### 3.3 核心代码

```cpp
void rangeAdd(int l, int r, long long v) {
    if (bel[l] == bel[r]) {                       // 同一块：纯暴力
        for (int i = l; i <= r; ++i) {
            a[i] += v;
            sum[bel[i]] += v;
        }
        return;
    }
    // 左端零散
    for (int i = l; i <= R[bel[l]]; ++i) {
        a[i] += v;
        sum[bel[i]] += v;
    }
    // 右端零散
    for (int i = L[bel[r]]; i <= r; ++i) {
        a[i] += v;
        sum[bel[i]] += v;
    }
    // 中间整块
    for (int b = bel[l] + 1; b <= bel[r] - 1; ++b) {
        tag[b] += v;
        sum[b] += v * (R[b] - L[b] + 1);
    }
}

long long rangeSum(int l, int r) {
    long long res = 0;
    if (bel[l] == bel[r]) {
        for (int i = l; i <= r; ++i) res += a[i] + tag[bel[i]];
        return res;
    }
    for (int i = l; i <= R[bel[l]]; ++i) res += a[i] + tag[bel[i]];
    for (int i = L[bel[r]]; i <= r; ++i) res += a[i] + tag[bel[i]];
    for (int b = bel[l] + 1; b <= bel[r] - 1; ++b) res += sum[b];
    return res;
}
```

---

## 4. 完整模板代码

一份可直接编译的分块模板（区间加 + 区间求和 + 区间最大值）：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
const int MAXB = 405;

int n, m, B, cnt;
long long a[MAXN];
int L[MAXB], R[MAXB], bel[MAXN];
long long sum[MAXB], mx[MAXB], tag[MAXB];

void build() {
    B = max(1, (int)sqrt(n));          // 块长取 sqrt(n)
    cnt = (n + B - 1) / B;             // 块数
    for (int i = 1; i <= cnt; ++i) {
        L[i] = (i - 1) * B + 1;
        R[i] = min(n, i * B);
        sum[i] = 0;
        mx[i] = LLONG_MIN / 4;         // 清空块信息
    }
    for (int i = 1; i <= n; ++i) {
        bel[i] = (i - 1) / B + 1;
    }
    for (int i = 1; i <= n; ++i) {
        int b = bel[i];
        sum[b] += a[i];
        mx[b] = max(mx[b], a[i]);
    }
}

// 重算第 b 块的信息（块内暴力改完后调用）
void rebuild(int b) {
    long long s = 0, m = LLONG_MIN / 4;
    for (int i = L[b]; i <= R[b]; ++i) {
        s += a[i];
        m = max(m, a[i]);
    }
    int len = R[b] - L[b] + 1;
    sum[b] = s + tag[b] * len;      // sum 是「真实值」之和，要带上块标记
    mx[b]  = m + tag[b];            // 同理
}

void rangeAdd(int l, int r, long long v) {
    if (bel[l] == bel[r]) {                     // 同一块：暴力
        for (int i = l; i <= r; ++i) a[i] += v;
        rebuild(bel[l]);
        return;
    }
    for (int i = l; i <= R[bel[l]]; ++i) a[i] += v;   // 左零散
    rebuild(bel[l]);
    for (int i = L[bel[r]]; i <= r; ++i) a[i] += v;   // 右零散
    rebuild(bel[r]);
    for (int b = bel[l] + 1; b <= bel[r] - 1; ++b) {  // 中间整块
        tag[b] += v;
        sum[b] += v * (R[b] - L[b] + 1);
        mx[b]  += v;
    }
}

long long querySum(int l, int r) {
    long long res = 0;
    if (bel[l] == bel[r]) {
        for (int i = l; i <= r; ++i) res += a[i] + tag[bel[i]];
        return res;
    }
    for (int i = l; i <= R[bel[l]]; ++i) res += a[i] + tag[bel[i]];
    for (int i = L[bel[r]]; i <= r; ++i) res += a[i] + tag[bel[i]];
    for (int b = bel[l] + 1; b <= bel[r] - 1; ++b) res += sum[b];
    return res;
}

long long queryMax(int l, int r) {
    long long res = LLONG_MIN / 4;
    if (bel[l] == bel[r]) {
        for (int i = l; i <= r; ++i) res = max(res, a[i] + tag[bel[i]]);
        return res;
    }
    for (int i = l; i <= R[bel[l]]; ++i) res = max(res, a[i] + tag[bel[i]]);
    for (int i = L[bel[r]]; i <= r; ++i) res = max(res, a[i] + tag[bel[i]]);
    for (int b = bel[l] + 1; b <= bel[r] - 1; ++b) res = max(res, mx[b]);
    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) cin >> a[i];
    build();
    while (m--) {
        int op, l, r;
        cin >> op >> l >> r;
        if (op == 1) {
            long long v; cin >> v;
            rangeAdd(l, r, v);
        } else if (op == 2) {
            cout << querySum(l, r) << '\n';
        } else {
            cout << queryMax(l, r) << '\n';
        }
    }
    return 0;
}
```

### 4.1 关于 `rebuild` 与 `tag`

上面的 `a[i]` 存的是「**除掉块标记后的原始值**」，而 `tag[b]` 是整块共有的增量；`sum[b]`、`mx[b]` 存的则是**真实值**（含标记）的统计量。因此：

- 修改零散元素时直接改 `a[i]`，然后 `rebuild(b)` 重算块内信息；
- `rebuild` 里用 $a[i]$ 的统计结果再加上 `tag[b]` 的影响（和乘长度、最大值加标记）；
- 查询零散元素时用 `a[i] + tag[bel[i]]`，查询整块直接用 `sum[b]` 或 `mx[b]`。

**只要「`a` 不含标记、`sum`/`mx` 含标记」这个口径保持统一，就不会错**。另一种写法是把 `tag` 直接下放清空，代码更直观但常数更大。

---

## 5. 进阶：分块的其他形态

### 5.1 块内排序：区间加 + 区间第 k 小

每个块内维护一个**排好序的数组** `sorted[b]`：

- 修改：整块打 `tag`，零散块暴力改后**重新排序**（$O(B \log B)$）；
- 查询「区间内 $< x$ 的个数」：整块内二分 `lower_bound`（$O(\log B)$），零散块暴力。

单次操作 $O(\sqrt{n}\log n)$。典型题：区间加、区间查询排名。

### 5.2 值域分块

不只对下标分块，也可以对**值域**分块。例如「区间内出现次数」类问题：统计每个块内每个值的出现次数，修改时 $O(1)$ 更新，查询时整块直接查表、零散块暴力。

### 5.3 莫队与根号分治

- **莫队**：把询问按「左端点所在块」排序，本质是分块（详见[莫队算法](../莫队算法/README.md)）。
- **根号分治**：按「出现次数」或「度数」是否超过 $\sqrt{n}$ 分成两类。出现次数 $> \sqrt{n}$ 的数最多只有 $\sqrt{n}$ 个，可以预处理；$\le \sqrt{n}$ 的直接暴力。这是分块思想在**阈值**上的推广。
- **树上分块**：把树按深度或子树大小分块，用于树上莫队、树上路径查询等问题。

---

## 6. 经典例题

### 6.1 例题一：区间加 + 区间求和（模板题）

**题意**：给定长度为 $n$ 的数列，$m$ 次操作：区间加 $v$，或询问区间和。$n, m \le 10^5$。

**思路**：直接套用第 4 节模板，块内维护 `sum`，整块打 `tag`。每次操作先处理左右零散块并 `rebuild`，再给中间整块统一加标记。

**复杂度**：单次 $O(\sqrt{n})$，总 $O(m\sqrt{n})$，约 $3 \times 10^7$ 次运算，可过。

### 6.2 例题二：区间开根号

**题意**：维护序列，支持区间开根号（下取整）与区间求和。$n, m \le 10^5$，$a_i \le 10^9$。

**思路**：这是线段树懒标记写不动的典型。关键观察：一个数最多开 $O(\log\log a_i)$ 次根号就变成 1，之后开根号不再变化。因此：

- 每个块维护 `sum` 和「块内是否全部为 1」的标记 `allOne`；
- 修改时：若整块 `allOne`，直接跳过；否则暴力对每个元素开根号并重算；
- 每个元素最多被有效修改 $O(\log\log V)$ 次，总复杂度 $O(n\log\log V + m\sqrt{n})$。

**复杂度**：均摊 $O((n + m)\sqrt{n})$ 以内。

```cpp
void sqrtBlock(int b) {
    if (allOne[b]) return;                  // 全 1 直接跳过
    bool flag = true;
    sum[b] = 0;
    for (int i = L[b]; i <= R[b]; ++i) {
        a[i] = (long long)sqrt((long double)a[i]);
        sum[b] += a[i];
        if (a[i] > 1) flag = false;
    }
    allOne[b] = flag;
}
```

### 6.3 例题三：区间加 + 区间第 $k$ 小

**题意**：维护序列，支持区间加、询问区间内第 $k$ 小的数。

**思路**：块内排序 + 二分答案。

- 修改：整块打 `tag`；零散块暴力改后重新排序；
- 查询：二分答案 $x$，统计区间内 $\le x$ 的个数（整块 `upper_bound`、零散块暴力），若个数 $\ge k$ 则答案可行。

**复杂度**：修改 $O(\sqrt{n}\log n)$，查询 $O(\sqrt{n}\log n \log V)$。

### 6.4 例题四：维护区间众数

**题意**：静态序列，多次询问区间众数（出现次数最多的数）。

**思路**：预处理 `f[i][j]` 表示第 $i$ 块到第 $j$ 块的众数，块内再用 `vector` 存每个值出现的位置。询问时先取整块众数，再暴力检查零散元素能否更新答案（用 `upper_bound` 统计出现次数）。

**复杂度**：预处理 $O(n\sqrt{n})$，单次询问 $O(\sqrt{n}\log n)$。

---

## 7. 易错点

1. **块长选取**：应写成 `max(1, (int)sqrt(n))`，不要写死 `B = 320`，也要防止 $n$ 很小时除零。
2. **块数上界**：`MAXB` 开到 $\sqrt{MAXN} + 5$ 即可；开成 `MAXN` 虽不出错但浪费内存。
3. **边界块重复处理**：当 `bel[l] == bel[r]` 时必须**单独处理并 return**，否则左右零散会重复修改中间元素。
4. **`tag` 与 `a` 的口径**：`a[i]` 不含块标记，查询零散元素时必须加 `tag[bel[i]]`，否则少算整块增量。
5. **`rebuild` 忘记调用**：暴力修改零散块后必须重算 `sum` / `mx`，否则整块查询结果错误。
6. **`rebuild` 口径不一致**：`sum`/`mx` 存的是含标记的真实值，`rebuild` 必须补上 `tag[b]` 的贡献。
7. **`long long` 溢出**：`v * 块长度` 可能超过 `int`，要开 `long long`。
8. **块内排序后原数组变了**：排序要用副本 `sorted[]`，不能直接排 `a[]`，否则下标与值对不上。
9. **`sqrt` 精度**：对 `long long` 开根号建议用 `sqrtl` 或 `(long long)sqrt((long double)x)` 后微调。
10. **卡常**：用快读、减少函数调用，或把 `bel` 数组换成 `i / B + 1` 直接算。

---

## 8. 复杂度与常数分析

### 8.1 为什么是 $O(\sqrt{n})$

设块长 $B$，一次区间操作要处理左右两个零散块 $O(B)$ 和中间整块 $O(n/B)$，总时间

$$
T(B) = c_1 B + c_2 \frac{n}{B} \ge 2\sqrt{c_1 c_2 n}
$$

由 AM-GM 不等式，等号在 $B = \sqrt{c_2 n / c_1}$ 处取得，实践中取 $B \approx \sqrt{n}$ 即可。

### 8.2 常见规模下的表现

$n=10^4$ 时块长约 100、单次约 $2\times10^2$；$n=10^5$ 时块长约 316、单次约 $6\times10^2$；$n=10^6$ 时块长约 1000、单次约 $2\times10^3$。$n = m = 10^5$ 时总操作约 $6 \times 10^7$，在 1 秒时限内需要一定常数优化。

### 8.3 分块 vs 线段树

| 维度 | 分块 | 线段树 |
|---|---|---|
| 单次复杂度 | $O(\sqrt{n})$ | $O(\log n)$ |
| 代码量 / 调试难度 | 短 / 低 | 长 / 高 |
| 可扩展性 | 极强 | 受限于信息可合并 |
| 常数 | 较小 | 较大（递归） |

**结论**：能线段树就线段树；线段树写不动时，果断上分块。

---

## 9. 练习建议

### 9.1 由易到难

| 难度 | 题目类型 | 考察点 |
|---|---|---|
| ★ | 区间加 + 单点查询 | 分块入门 |
| ★★ | 区间加 + 区间求和 | 模板 |
| ★★ | 区间开根号 | 均摊分析 |
| ★★★ | 区间加 + 区间第 k 小 | 块内排序 + 二分 |
| ★★★ | 区间众数 | 预处理 + 零散暴力 |
| ★★★★ | 值域分块 / 根号分治 | 分块思想的迁移 |
| ★★★★★ | 树上分块 / 树套块 | 进阶 |

### 9.2 学习检查清单

- [ ] 能说清块长为什么取 $\sqrt{n}$，能默写 `build` / `rangeAdd` / `rangeSum` / `rebuild`
- [ ] 能解释「同一块」为什么要单独处理，能说清 `a[i]` 与 `tag[b]` 的关系
- [ ] 会用「块内排序 + 二分」解决区间排名问题，理解根号分治的思想

### 9.3 推荐练习顺序

1. 洛谷 P2801 教主的魔法（区间加 + 区间查询排名）
2. 洛谷 P4145 上帝造题的七分钟 2（区间开根号 + 区间求和）
3. 洛谷 P4168 [Violet] 蒲公英（区间众数）
4. 洛谷 P5355 [Ynoi2017] 由乃的玉米田（根号分治）
5. 洛谷 P5048 [Ynoi2019 模拟赛] Yuno loves sqrt technology III（区间众数加强版）

---

> **核心思想**：分块用「整块预处理 + 零散暴力」的方式，把单次操作压到 $O(\sqrt{n})$。它牺牲了理论复杂度，换来了**极低的实现门槛与极强的可扩展性**——当线段树写不出来时，分块永远是你的保底方案。
