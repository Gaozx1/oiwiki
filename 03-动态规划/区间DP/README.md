# 区间 DP：把大区间拆成小区间

> 一句话概括：**区间 DP 以「区间 $[l,r]$」为状态，通过枚举分割点或端点合并，把大区间的最优解由小区间的最优解推出**，典型代表是石子合并与括号序列。

---

## 目录

1. [概念与原理](#1-概念与原理)
   - [1.1 什么样的问题适合区间 DP](#11-什么样的问题适合区间-dp)
   - [1.2 状态设计与枚举顺序](#12-状态设计与枚举顺序)
   - [1.3 模型一：石子合并（分割点型）](#13-模型一石子合并分割点型)
   - [1.4 模型二：括号匹配（端点配对型）](#14-模型二括号匹配端点配对型)
   - [1.5 模型三：回文串删除（端点决策型）](#15-模型三回文串删除端点决策型)
   - [1.6 环形区间 DP](#16-环形区间-dp)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点与练习建议](#4-易错点与练习建议)

---

## 1. 概念与原理

### 1.1 什么样的问题适合区间 DP

区间 DP 适合满足下面特征的问题：

1. **答案定义在一段连续的区间上**，例如「把第 $l$ 到第 $r$ 堆石子合并成一堆的最小代价」；
2. **大区间的答案由若干小区间的答案合并而来**，而且合并方式可以枚举（分割点、端点配对等）；
3. **合并操作具有「最后一步」结构**：无论中间怎么合并，最后一步一定是把区间分成左右两部分再拼起来。

```
              [1 ............ r]
                  /        \
       [l .... k]            [k+1 .... r]
           /   \                /   \
      ......   ......      ......   ......
                     (递归地分成更小的区间)
```

因为状态是「区间」，一共有 $O(n^2)$ 个，每次转移若枚举分割点 $k$ 则是 $O(n)$，总复杂度 $O(n^3)$，这是区间 DP 最典型的时间复杂度。

### 1.2 状态设计与枚举顺序

**状态定义**：$dp_{l,r}$ 表示区间 $[l,r]$ 上的答案。

**枚举顺序**：必须**先枚举区间长度 $len$，再枚举左端点 $l$**，因为长区间的转移依赖短区间。如果按 $l$ 从 1 到 $n$、$r$ 从 $l$ 到 $n$ 的顺序枚举，会用到还没算完的状态。

```
长度 1: [1,1] [2,2] [3,3] ...
长度 2: [1,2] [2,3] [3,4] ...
长度 3: [1,3] [2,4] [3,5] ...
   ↑
每个状态只依赖长度更小的区间，所以按长度从小到大枚举一定正确
```

标准三段循环（完整可编译骨架）：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;                         // 示例长度
    vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));

    for (int len = 2; len <= n; ++len)                 // 外层：区间长度
        for (int l = 1; l + len - 1 <= n; ++l) {       // 中层：左端点
            int r = l + len - 1;                       // 右端点
            for (int k = l; k < r; ++k) {              // 内层：枚举分割点
                dp[l][r] = max(dp[l][r], dp[l][k] + dp[k + 1][r]);
            }
        }

    cout << dp[1][n] << '\n';
    return 0;
}
```

### 1.3 模型一：石子合并（分割点型）

**问题**：$n$ 堆石子排成一排，第 $i$ 堆有 $a_i$ 个。每次可以合并相邻两堆，代价为两堆石子数之和，求合并成一堆的最小总代价。

**状态设计**：$dp_{l,r}$ 表示把 $[l,r]$ 合并成一堆的最小代价。

**转移方程**：

$$
dp_{l,r}=\min_{l\le k<r}\left(dp_{l,k}+dp_{k+1,r}+\sum_{i=l}^{r}a_i\right)
$$

**解释**：最后一次合并一定是把 $[l,k]$ 合并成的一堆与 $[k+1,r]$ 合并成的一堆合并，代价是区间总和。区间总和用前缀和 $O(1)$ 求出。

```
以 [1,2,3,4] 为例，最后一次合并可能是：
  [1] | [2,3,4]     代价 = dp[1,1] + dp[2,4] + 10
  [1,2] | [3,4]     代价 = dp[1,2] + dp[3,4] + 10
  [1,2,3] | [4]     代价 = dp[1,3] + dp[4,4] + 10
取三者最小值，其中每个 dp 子区间又递归地取最优
```

**边界**：$dp_{i,i}=0$（一堆不需要合并）。

**复杂度**：状态 $O(n^2)$，转移 $O(n)$，总 $O(n^3)$。用四边形不等式 + 决策单调性可优化到 $O(n^2)$。

### 1.4 模型二：括号匹配（端点配对型）

**问题**：给定一个括号序列，求最长的合法括号子序列长度（可以不连续）。

**状态设计**：$dp_{l,r}$ 表示区间 $[l,r]$ 内最长合法括号子序列长度。

**转移分两类**：

1. **端点不配对**：$dp_{l,r}=\max(dp_{l+1,r},\ dp_{l,r-1})$；
2. **端点配对**：若 $s_l$ 与 $s_r$ 是一对合法括号，则 $dp_{l,r}=\max(dp_{l,r},\ dp_{l+1,r-1}+2)$。

此外还要枚举中间断点：

$$
dp_{l,r}=\max_{l\le k<r}\left(dp_{l,k}+dp_{k+1,r}\right)
$$

**注意**：如果只写前两类转移，会漏掉「$[l,r]$ 由两段合法括号拼接」的情况。

```
( ) ( )
  ↑   ↑
可以整体配对（首尾配对），也可以从中间切开：
  [ ( ) ] + [ ( ) ]   ← 必须枚举断点才能发现
```

### 1.5 模型三：回文串删除（端点决策型）

**问题**：给定字符串 $s$，每次可以删除一个回文子串，求删空整个串的最少操作次数。

**状态设计**：$dp_{l,r}$ 表示删空 $[l,r]$ 的最少操作次数。

**转移方程**：

$$
dp_{l,r}=
\begin{cases}
dp_{l+1,r-1}, & s_l=s_r \text{ 且内部可一起删}\\
\min\left(dp_{l+1,r},\ dp_{l,r-1}\right)+1, & \text{其他}
\end{cases}
$$

更通用的写法是枚举与 $l$ 相同字符的位置 $k$，让 $s_l$ 和 $s_k$ 在同一次操作中被删除：

$$
dp_{l,r}=\min_{l\le k\le r,\ s_k=s_l}\left(dp_{l+1,k-1}+dp_{k+1,r}\right)
$$

这类「端点字符相同就一起处理」的思想在回文类区间 DP 中非常常见。

### 1.6 环形区间 DP

如果石子是**环形**排列，需要把环断开成链。常用技巧是**倍长数组**：令 $a_{i+n}=a_i$，在长度 $2n$ 的数组上做区间 DP，然后取所有长度为 $n$ 的区间的答案最优值。

```
环形:  1 - 2 - 3 - 4 - 1
倍长:  1 2 3 4 | 1 2 3 4
       取长度为 n=4 的窗口：
       [1 2 3 4] [2 3 4 1] [3 4 1 2] [4 1 2 3]
       分别计算，取最优
```

复杂度从 $O(n^3)$ 变为 $O((2n)^3)=8O(n^3)$，常数变大但可以接受。

---

## 2. 模板代码

### 2.1 石子合并（链式）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 305;
const int INF = 0x3f3f3f3f;

int n;
int a[MAXN], sum[MAXN];
int dp[MAXN][MAXN];

int main() {
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        sum[i] = sum[i - 1] + a[i];        // 前缀和，O(1) 求区间和
    }

    memset(dp, 0x3f, sizeof(dp));          // 求最小值，初始化为 INF
    for (int i = 1; i <= n; ++i) dp[i][i] = 0;   // 边界：单堆代价 0

    for (int len = 2; len <= n; ++len) {           // 先枚举长度
        for (int l = 1; l + len - 1 <= n; ++l) {   // 再枚举左端点
            int r = l + len - 1;
            for (int k = l; k < r; ++k) {          // 枚举分割点
                dp[l][r] = min(dp[l][r],
                    dp[l][k] + dp[k + 1][r] + sum[r] - sum[l - 1]);
            }
        }
    }
    cout << dp[1][n] << '\n';
    return 0;
}
```

### 2.2 石子合并（环形，输出最小与最大）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 405;
const int INF = 0x3f3f3f3f;

int n, a[MAXN], sum[MAXN];
int dpMin[MAXN][MAXN], dpMax[MAXN][MAXN];

int main() {
    cin >> n;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i];
        a[i + n] = a[i];                   // 倍长成环
    }
    for (int i = 1; i <= 2 * n; ++i) sum[i] = sum[i - 1] + a[i];

    memset(dpMin, 0x3f, sizeof(dpMin));
    memset(dpMax, 0xcf, sizeof(dpMax));      // 最大值的负无穷
    for (int i = 1; i <= 2 * n; ++i) dpMin[i][i] = dpMax[i][i] = 0;

    for (int len = 2; len <= n; ++len) {
        for (int l = 1; l + len - 1 <= 2 * n; ++l) {
            int r = l + len - 1;
            for (int k = l; k < r; ++k) {
                int s = sum[r] - sum[l - 1];
                dpMin[l][r] = min(dpMin[l][r], dpMin[l][k] + dpMin[k + 1][r] + s);
                dpMax[l][r] = max(dpMax[l][r], dpMax[l][k] + dpMax[k + 1][r] + s);
            }
        }
    }

    int ansMin = INF, ansMax = -INF;
    for (int l = 1; l <= n; ++l) {         // 枚举长度为 n 的窗口
        ansMin = min(ansMin, dpMin[l][l + n - 1]);
        ansMax = max(ansMax, dpMax[l][l + n - 1]);
    }
    cout << ansMin << '\n' << ansMax << '\n';
    return 0;
}
```

### 2.3 最长回文子序列

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    cin >> s;
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; ++i) dp[i][i] = 1;      // 单字符长度 1

    for (int len = 2; len <= n; ++len)
        for (int l = 0; l + len - 1 < n; ++l) {
            int r = l + len - 1;
            if (s[l] == s[r]) dp[l][r] = dp[l + 1][r - 1] + 2;   // 两端配对
            else dp[l][r] = max(dp[l + 1][r], dp[l][r - 1]);     // 舍弃一端
        }
    cout << dp[0][n - 1] << '\n';
    return 0;
}
```

---

## 3. 经典例题

### 例题 1：石子合并（洛谷 P1775）

**题意**：$n$ 堆石子排成一排，每次合并相邻两堆，代价为两堆之和，求合并成一堆的最小总代价。$1\le n\le 300$。

**思路**：区间 DP 模板题，$dp_{l,r}=\min_{k}(dp_{l,k}+dp_{k+1,r})+sum(l,r)$。

**复杂度**：时间 $O(n^3)$，空间 $O(n^2)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    cin >> n;
    vector<int> sum(n + 1, 0);
    for (int i = 1; i <= n; ++i) {
        int x;
        cin >> x;
        sum[i] = sum[i - 1] + x;
    }

    vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));
    for (int len = 2; len <= n; ++len)
        for (int l = 1; l + len - 1 <= n; ++l) {
            int r = l + len - 1;
            dp[l][r] = INT_MAX;
            for (int k = l; k < r; ++k)
                dp[l][r] = min(dp[l][r], dp[l][k] + dp[k + 1][r] + sum[r] - sum[l - 1]);
        }
    cout << dp[1][n] << '\n';
    return 0;
}
```

### 例题 2：能量项链（洛谷 P1063）

**题意**：环形排列的 $n$ 颗珠子，每颗有两个属性，相邻两颗合并释放能量，求最大总能量。

**思路**：环形区间 DP，倍长数组后枚举长度 $n$ 的窗口；转移时把区间视为「头尾相接」，注意下标的含义。

**复杂度**：时间 $O(n^3)$，空间 $O(n^2)$。

### 例题 3：最长回文子序列（LeetCode 516 的 OI 版本）

**题意**：给定字符串 $s$（$|s|\le 1000$），求最长回文子序列长度。

**思路**：$dp_{l,r}$，若 $s_l=s_r$ 则 $dp_{l,r}=dp_{l+1,r-1}+2$，否则 $dp_{l,r}=\max(dp_{l+1,r},dp_{l,r-1})$。

**复杂度**：时间 $O(n^2)$，空间 $O(n^2)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    cin >> s;
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; ++i) dp[i][i] = 1;

    for (int len = 2; len <= n; ++len)
        for (int l = 0; l + len - 1 < n; ++l) {
            int r = l + len - 1;
            if (s[l] == s[r])
                dp[l][r] = dp[l + 1][r - 1] + 2;
            else
                dp[l][r] = max(dp[l + 1][r], dp[l][r - 1]);
        }
    cout << dp[0][n - 1] << '\n';
    return 0;
}
```

### 例题 4：涂色（洛谷 P4170 [CQOI2007]）

**题意**：给定一个长度为 $n$ 的目标颜色串（大写字母），初始全为空白。每次可以把一段连续区间涂成同一种颜色（后涂的覆盖先涂的），求得到目标串的最少涂色次数。$1\le n\le 50$。

**思路**：$dp_{l,r}$ 表示涂好 $[l,r]$ 的最少次数。

- 先把 $l$ 单独涂掉：$dp_{l,r}=dp_{l+1,r}+1$；
- 若存在 $k\in(l,r]$ 使 $s_k=s_l$，可以让 $s_l$ 和 $s_k$ 在同一次涂色中完成，中间部分单独处理：$dp_{l,r}=\min(dp_{l,r},\ dp_{l+1,k-1}+dp_{k,r})$。

**复杂度**：时间 $O(n^3)$，空间 $O(n^2)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    cin >> s;
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int i = 0; i < n; ++i) dp[i][i] = 1;   // 单个字符涂一次

    for (int len = 2; len <= n; ++len)
        for (int l = 0; l + len - 1 < n; ++l) {
            int r = l + len - 1;
            dp[l][r] = dp[l + 1][r] + 1;         // 方案一：l 单独涂
            for (int k = l + 1; k <= r; ++k) {   // 方案二：l 与 k 同色，一起涂
                if (s[k] == s[l]) {
                    int mid = (k - 1 >= l + 1) ? dp[l + 1][k - 1] : 0;
                    dp[l][r] = min(dp[l][r], mid + dp[k][r]);
                }
            }
        }
    cout << dp[0][n - 1] << '\n';
    return 0;
}
```

---

## 4. 易错点与练习建议

### 4.1 常见易错点

1. **枚举顺序错误**：必须先枚举区间长度，再枚举左端点。按左右端点自然顺序枚举会用到未计算的状态。
2. **数组越界**：$dp$ 数组要开 $n+1$ 大小，且循环条件写成 `l + len - 1 <= n`，不要把 `<=` 写成 `<`。
3. **忘记边界**：$dp_{i,i}$ 要单独初始化；求最小值初始化为 `INF`，求最大值初始化为 `-INF`。
4. **前缀和区间写错**：区间 $[l,r]$ 的和是 `sum[r] - sum[l-1]`，不要写成 `sum[r] - sum[l]`。
5. **环形忘记倍长**：环形问题必须先复制一遍数组，再在长度 $2n$ 上做 DP。
6. **括号序列漏断点转移**：只写端点配对会漏掉「两段合法序列拼接」的情况。
7. **`memset` 初始化非 0 值**：`memset(dp, 0x3f, sizeof(dp))` 对 `int` 是安全的（每字节 0x3f），但 `memset` 不能给 `long long` 设成任意值，最好用 `fill`。
8. **空间开太大**：$dp_{1000,1000}$ 的 `int` 数组约 4 MB，可以接受；但 $5000\times 5000$ 就会超内存，需要注意空间优化或换算法。

### 4.2 练习建议（由易到难）

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1775 石子合并（弱化版） | 区间 DP 模板 |
| 入门 | LeetCode 516 最长回文子序列 | 端点决策 |
| 普及 | 洛谷 P1063 能量项链 | 环形区间 DP |
| 普及 | 洛谷 P1435 回文子串 | 区间 DP + 端点匹配 |
| 提高 | 洛谷 P1880 石子合并（环形） | 环形拆链 |
| 提高 | 洛谷 P1220 关路灯 | 区间 DP 的扩展状态（位置 + 方向） |
| 提高 | 洛谷 P2473 奖励关 | 区间 + 状压的结合 |

**训练方法**：

1. 手推长度为 1、2、3 的区间 DP 表格，确认每个格子的来源；
2. 每道题先问自己「最后一步的合并方式是什么」，再写转移；
3. 区间 DP 的调试技巧：在循环里输出当前区间和分割点，对照暴力搜索的结果；
4. 遇到 $n\le 500$ 的区间问题，如果 $O(n^3)$ 会超时，考虑四边形不等式优化或换思路。
