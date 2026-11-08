# 单调队列（Monotonic Queue）

> 一句话概括：**单调队列是一个内部元素保持单调的双端队列**，用 $O(n)$ 的时间求出每个长度为 $k$ 的滑动窗口中的最大值或最小值。

---

## 目录

1. [概念与原理](#1-概念与原理)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点](#4-易错点)
5. [练习建议](#5-练习建议)

---

## 1. 概念与原理

### 1.1 滑动窗口最大值问题

给定长度为 $n$ 的数组 $a$ 和窗口大小 $k$，窗口从左向右每次滑动一格，求每个窗口内的最大值。

朴素做法每个窗口扫一遍，复杂度 $O(nk)$。当 $n=10^6, k=10^5$ 时不可接受。

### 1.2 核心观察：淘汰「又老又小」的元素

对窗口最大值，考虑窗口中两个下标 $i < j$：

- 若 $a_i \le a_j$，那么 $a_i$ 不但比 $a_j$ 小，还会**比 $a_j$ 更早离开窗口**。因此只要 $a_j$ 还在窗口中，$a_i$ 就永远不可能是最大值——可以放心丢弃；
- 反之，若 $a_i > a_j$，则 $a_j$ 在 $a_i$ 离开之前还有机会成为最大值，需要保留。

结论：**窗口中真正有用的元素构成一个「下标递增、值递减」的子序列**。用双端队列维护它，就得到单调队列。

### 1.3 单调队列的维护规则

以求窗口最大值为例，队列中存**下标**，对应的值从队头到队尾**递减**：

1. **入队前弹队尾**：当 `a[dq.back()] <= a[i]` 时弹出队尾（新元素更大更年轻，旧元素永远没用了）；
2. **入队**：把 `i` 压入队尾；
3. **弹队头**：若 `dq.front() <= i - k`，说明队头已滑出窗口，弹出；
4. **取答案**：此时 `a[dq.front()]` 就是当前窗口最大值。

每个元素最多进出队各一次，总复杂度 $O(n)$。

### 1.4 文字示意图

数组 `a = [1, 3, -1, -3, 5, 3, 6, 7]`，$k = 3$，求滑动窗口最大值：

```
i=0, a=1:   入 0                    队列: [0]        窗口 [1]          最大 1（未满）
i=1, a=3:   弹 0（1<=3），入 1      队列: [1]        窗口 [1,3]        最大 3（未满）
i=2, a=-1:  入 2                    队列: [1,2]      窗口 [1,3,-1]     最大 3
i=3, a=-3:  入 3                    队列: [1,2,3]    窗口 [3,-1,-3]    最大 3
i=4, a=5:   弹 3,2,1（都<=5），入 4  队列: [4]        窗口 [-1,-3,5]    最大 5
i=5, a=3:   入 5                    队列: [4,5]      窗口 [-3,5,3]     最大 5
i=6, a=6:   弹 5,4，入 6            队列: [6]        窗口 [5,3,6]      最大 6
i=7, a=7:   弹 6，入 7              队列: [7]        窗口 [3,6,7]      最大 7
```

对应的柱状图（$i=4$ 时元素 $5$ 把前面的 $3,-1,-3$ 全「压死」了）：

```
值
 7 |                        █
 6 |                    █   █
 5 |                █   █   █
 3 |     █   █   █   █   █   █
 1 | █   █   █   █   █   █   █
-1 |     █   █   █   █   █   █
-3 |         █   █   █   █   █
   +----------------------------
     0 1 2 3 4 5 6 7
```

### 1.5 单调队列 vs 单调栈 vs 优先队列

| 结构 | 解决的问题 | 复杂度 |
|---|---|---|
| 单调栈 | 每个元素左/右第一个更大/更小 | $O(n)$ |
| 单调队列 | 定长滑动窗口的最值 | $O(n)$ |
| 优先队列（堆） | 动态集合最值，可任意插入 | $O(\log n)$ / 次，需惰性删除 |
| 线段树 / ST 表 | 任意区间最值 | $O(\log n)$ / $O(1)$ |

**为什么单调队列比堆好**：滑动窗口是先进先出的，老元素按顺序过期；堆无法高效删除任意元素，只能打标记惰性删除，且每次带 $\log$。单调队列天然匹配 FIFO 的过期顺序，做到均摊 $O(1)$。

### 1.6 单调队列的两种常见题型

1. **定长窗口最值**：窗口长度固定为 $k$，如滑动窗口最大值；
2. **带约束的最优化 DP**：转移形如

$$
f_i = \min_{i-m \le j < i}\{f_j\} + a_i
$$

决策集合是滑动窗口且每个决策只用一次，可用单调队列把 $O(nm)$ 优化到 $O(n)$。

---

## 2. 模板代码

### 2.1 滑动窗口最大值（手写双端队列）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for (int& x : a) cin >> x;

    vector<int> ans;
    deque<int> dq;                    // 存下标，对应值从队头到队尾递减
    for (int i = 0; i < n; ++i) {
        while (!dq.empty() && a[dq.back()] <= a[i]) dq.pop_back();  // 1. 弹队尾
        dq.push_back(i);                                            // 2. 入队
        if (dq.front() <= i - k) dq.pop_front();                    // 3. 弹过期队头
        if (i >= k - 1) ans.push_back(a[dq.front()]);               // 4. 记录答案
    }
    for (int i = 0; i < (int)ans.size(); ++i)
        cout << ans[i] << " \n"[i + 1 == (int)ans.size()];
    return 0;
}
```

输入：

```
8 3
1 3 -1 -3 5 3 6 7
```

输出：

```
3 3 5 5 6 7
```

### 2.2 滑动窗口最小值

只需把「弹队尾」的比较符号反过来：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for (int& x : a) cin >> x;

    deque<int> dq;                    // 存下标，对应值从队头到队尾递增
    vector<int> ans;
    for (int i = 0; i < n; ++i) {
        while (!dq.empty() && a[dq.back()] >= a[i]) dq.pop_back();  // 弹掉更大的
        dq.push_back(i);
        if (dq.front() <= i - k) dq.pop_front();
        if (i >= k - 1) ans.push_back(a[dq.front()]);
    }
    for (int i = 0; i < (int)ans.size(); ++i)
        cout << ans[i] << " \n"[i + 1 == (int)ans.size()];
    return 0;
}
```

### 2.3 数组模拟双端队列（常数更小）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1000005;
int q[MAXN];                  // 存下标
int head = 0, tail = 0;       // 队列区间 [head, tail)

int main() {
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    for (int i = 0; i < n; ++i) {
        while (tail > head && a[q[tail - 1]] <= a[i]) --tail;   // 弹队尾
        q[tail++] = i;                                          // 入队
        while (tail > head && q[head] <= i - k) ++head;         // 弹过期队头
        if (i >= k - 1) cout << a[q[head]] << ' ';
    }
    cout << '\n';
    return 0;
}
```

### 2.4 单调队列优化 DP 通用骨架

```cpp
#include <bits/stdc++.h>
using namespace std;

// 转移：dp[i] = min(dp[j]) + a[i]，其中 i-m <= j < i
vector<long long> solve(const vector<long long>& a, int m) {
    int n = (int)a.size() - 1;          // a 下标从 1 开始
    vector<long long> dp(n + 1, 0);
    deque<int> dq;
    dq.push_back(0);                    // dp[0] = 0 作为初始决策
    for (int i = 1; i <= n; ++i) {
        while (!dq.empty() && dq.front() < i - m) dq.pop_front();  // 1. 移除过期决策
        dp[i] = dp[dq.front()] + a[i];                             // 2. 用队头转移
        while (!dq.empty() && dp[dq.back()] >= dp[i]) dq.pop_back();// 3. 弹掉不如 i 的
        dq.push_back(i);
    }
    return dp;
}

int main() {
    vector<long long> a = {0, 3, 1, 4, 1, 5, 9};   // 下标 0 占位
    auto dp = solve(a, 2);
    for (int i = 1; i <= 6; ++i) cout << dp[i] << " \n"[i == 6];
    return 0;
}
```

---

## 3. 经典例题

### 3.1 滑动窗口的最大值与最小值

**题意**：给定长度为 $n$ 的数组和窗口长度 $k$，输出每个窗口中的最小值和最大值。

**思路**：维护两个单调队列，一个递增（最小值）、一个递减（最大值），同时扫描。

**复杂度**：时间 $O(n)$，空间 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for (int& x : a) cin >> x;

    deque<int> mn, mx;
    auto work = [&](deque<int>& dq, int i, bool isMin) {
        while (!dq.empty() && (isMin ? a[dq.back()] >= a[i] : a[dq.back()] <= a[i]))
            dq.pop_back();
        dq.push_back(i);
        if (dq.front() <= i - k) dq.pop_front();
    };
    for (int i = 0; i < n; ++i) { work(mn, i, true);  if (i >= k - 1) cout << a[mn.front()] << ' '; }
    cout << '\n';
    for (int i = 0; i < n; ++i) { work(mx, i, false); if (i >= k - 1) cout << a[mx.front()] << ' '; }
    cout << '\n';
    return 0;
}
```

### 3.2 最大子序和（长度不超过 m）

**题意**：给定长度为 $n$ 的整数序列，找长度不超过 $m$ 的连续子段，使其和最大。

**思路**：设前缀和 $S_i$，区间 $(j, i]$ 的和为 $S_i - S_j$。固定 $i$ 需找 $j \in [i-m, i-1]$ 中最小的 $S_j$——滑动窗口最小值，用单调队列维护：

$$
\text{ans} = \max_{1 \le i \le n}\left(S_i - \min_{i-m \le j < i} S_j\right)
$$

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    vector<long long> s(n + 1, 0);
    for (int i = 1; i <= n; ++i) { long long x; cin >> x; s[i] = s[i - 1] + x; }

    deque<int> dq;                     // 存前缀和下标，对应 S 值递增
    dq.push_back(0);
    long long ans = LLONG_MIN;
    for (int i = 1; i <= n; ++i) {
        while (!dq.empty() && dq.front() < i - m) dq.pop_front();
        ans = max(ans, s[i] - s[dq.front()]);
        while (!dq.empty() && s[dq.back()] >= s[i]) dq.pop_back();
        dq.push_back(i);
    }
    cout << ans << '\n';
    return 0;
}
```

### 3.3 旅行问题（环上单调队列）

**题意**：环形公路有 $n$ 个加油站，第 $i$ 站可加油 $p_i$ 升，到下一站消耗 $d_i$ 升，油箱容量无限且初始为空，问从哪些站出发能绕一圈。

**思路**：断环成链（长度 $2n$），令 $w_i = p_i - d_i$，前缀和 $S$。从 $i$ 出发能绕一圈等价于 $[i, i+n-1]$ 内 $S$ 的最小值不小于 $S_{i-1}$，用单调队列求长度 $n$ 的窗口最小值。

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n;
    cin >> n;
    vector<long long> p(n + 1), d(n + 1), s(2 * n + 1, 0);
    for (int i = 1; i <= n; ++i) cin >> p[i] >> d[i];
    for (int i = 1; i <= 2 * n; ++i) {
        int id = (i - 1) % n + 1;
        s[i] = s[i - 1] + p[id] - d[id];
    }
    deque<int> dq;
    vector<bool> ok(n + 1, false);
    for (int i = 1; i <= 2 * n; ++i) {
        while (!dq.empty() && dq.front() < i - n + 1) dq.pop_front();
        while (!dq.empty() && s[dq.back()] >= s[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= n && s[dq.front()] >= s[i - n]) ok[(i - n) % n + 1] = true;
    }
    for (int i = 1; i <= n; ++i) cout << (ok[i] ? "TAK" : "NIE") << '\n';
    return 0;
}
```

### 3.4 修剪草坪（DP + 单调队列）

**题意**：$n$ 头奶牛排成一排，第 $i$ 头效率 $E_i$，不能连续选超过 $k$ 头，求最大总效率。

**思路**：设 $f_i$ 为前 $i$ 头牛的最大效率。不选第 $i$ 头时 $f_i = f_{i-1}$；若最后一段连续选 $j+1 \sim i$（$i-j \le k$），则

$$
f_i = \max_{i-k \le j < i}\{f_j + S_i - S_j\} = S_i + \max_{i-k \le j < i}\{f_j - S_j\}
$$

用单调队列维护 $g_j = f_j - S_j$ 的窗口最大值，总复杂度 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, k;
    cin >> n >> k;
    vector<long long> s(n + 1, 0), f(n + 1, 0);
    for (int i = 1; i <= n; ++i) { long long e; cin >> e; s[i] = s[i - 1] + e; }

    deque<int> dq;                     // 存 j，对应 g[j] = f[j] - s[j] 递减
    dq.push_back(0);
    for (int i = 1; i <= n; ++i) {
        while (!dq.empty() && dq.front() < i - k) dq.pop_front();
        f[i] = s[i] + f[dq.front()] - s[dq.front()];
        f[i] = max(f[i], f[i - 1]);    // 不选第 i 头
        long long g = f[i] - s[i];
        while (!dq.empty() && f[dq.back()] - s[dq.back()] <= g) dq.pop_back();
        dq.push_back(i);
    }
    cout << f[n] << '\n';
    return 0;
}
```

---

## 4. 易错点

1. **队列里存值而不是存下标**：判断「是否滑出窗口」必须依赖下标，队列应存下标，通过 `a[dq.front()]` 取值。
2. **弹队头与弹队尾的顺序**：一般先弹队尾（维护单调性）、再入队、最后弹队头（判断过期）；顺序写错可能把当前元素误删。
3. **窗口未满就输出**：只有 `i >= k - 1` 时窗口才满，提前输出会得到错误答案。
4. **相等元素处理**：`<=` 与 `<` 会影响队列长度和答案下标，但不影响最值；若题目需要具体位置，必须明确取最左还是最右。
5. **求最小值时忘记反向**：最小值维护**递增**队列、弹队尾用 `>=`；最大值维护**递减**队列、用 `<=`。
6. **`deque::pop_front` 前未判空**：先判 `!dq.empty()` 再操作。
7. **DP 优化中忘记把初始决策入队**：如 `dp[0] = 0` 必须提前 `push_back(0)`。
8. **过期判断差一**：窗口范围为 `[i-k+1, i]` 时，`dq.front() < i - k + 1` 才过期；写成 `<=` 会多删一个。
9. **`long long` 溢出**：前缀和、DP 值可能超 `int`。

---

## 5. 练习建议

### 5.1 由易到难的路线

1. **模板题**：滑动窗口最大值、最小值（洛谷 P1886）。
2. **前缀和 + 单调队列**：最大子序和（长度限制）。
3. **环上问题**：旅行问题、环形最大子段和。
4. **DP 优化**：修剪草坪、选择数字、多重背包单调队列优化。
5. **二维扩展**：最大子矩阵（枚举行 + 一维滑动窗口）。
6. **综合应用**：股票交易（含冷冻期）、跳跃游戏 VI、绝对差不超过限制的最长连续子数组。

### 5.2 推荐题目

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1886 滑动窗口 | 模板 |
| 基础 | 洛谷 P1440 求 m 区间内的最小值 | 模板 |
| 基础 | LeetCode 239 滑动窗口最大值 | 模板 |
| 提高 | 洛谷 P1714 切蛋糕 | 前缀和 + 单调队列 |
| 提高 | 洛谷 P2629 好消息，坏消息 | 环上单调队列 |
| 提高 | 洛谷 P2034 选择数字 | DP + 单调队列 |
| 提高 | 洛谷 P2216 理想的正方形 | 二维滑动窗口 |

### 5.3 学习检查清单

- [ ] 能说清为什么「又老又小」的元素可以直接淘汰
- [ ] 能徒手模拟单调队列的完整过程
- [ ] 能默写滑动窗口最大值和最小值的模板
- [ ] 理解「先弹尾、再入队、后弹头」的顺序及原因
- [ ] 能把 `dp[i] = min dp[j] + a[i]` 这类转移用单调队列优化
- [ ] 知道单调队列与优先队列、ST 表的适用场景差异

> **核心思想**：单调队列用「双端队列 + 及时淘汰」把滑动窗口最值做到均摊 $O(1)$——只要决策集合是滑动窗口且满足单调淘汰性，就能把 $O(nk)$ 或 $O(nm)$ 优化成 $O(n)$。
