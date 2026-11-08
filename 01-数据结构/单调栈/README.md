# 单调栈（Monotonic Stack）

> 一句话概括：**单调栈是一种栈内元素始终保持单调的栈**，用来在线性时间内求出每个元素「左边/右边第一个比它大/小的元素」。

---

## 目录

1. [概念与原理](#1-概念与原理)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点](#4-易错点)
5. [练习建议](#5-练习建议)

---

## 1. 概念与原理

### 1.1 从暴力到单调栈

考虑这样一个问题：给定数组 $a_1, a_2, \dots, a_n$，对每个位置 $i$，求**右边第一个严格大于 $a_i$ 的位置**（不存在则为 0）。

暴力做法：对每个 $i$ 向右扫描，最坏 $O(n^2)$。

**关键观察（从右往左扫描）**：假设当前处理到 $i$，而某个 $j > i$ 满足 $a_j \le a_i$。那么对任何 $k < i$：

- 如果 $a_j > a_k$，那么 $a_i \ge a_j > a_k$ 且 $i$ 比 $j$ 更靠左，所以 $k$ 的「右边第一个更大元素」会是 $i$ 而不是 $j$；
- 如果 $a_j \le a_k$，那 $a_j$ 本来就不是答案。

也就是说，**只要 $i$ 存在，$j$ 就永远不可能成为任何左侧位置的答案**，可以安全丢弃。而 $i$ 本身还可能被更左边的位置需要，所以要保留。

于是从右往左扫描时，用一个栈保存「还有可能成为答案」的下标，并保证栈中对应值从栈底到栈顶**严格递减**：遇到比栈顶更大的当前值，就把那些「又矮又靠左」的栈顶弹掉，因为它们已被当前元素彻底取代。每个元素最多进栈一次、出栈一次，总复杂度 $O(n)$。

### 1.2 单调栈的定义

单调栈就是**栈内元素从栈底到栈顶保持单调**的栈：

- **单调递增栈**（从栈底到栈顶递增）：用于求「左边/右边第一个更小的元素」；
- **单调递减栈**（从栈底到栈顶递减）：用于求「左边/右边第一个更大的元素」。

维护方式（以求右边第一个更大元素为例，从右往左扫描）：

1. 当前元素为 $a_i$；
2. 当栈非空且 `a[st.top()] <= a[i]` 时，不断弹出栈顶（这些元素都比 $a_i$ 小，不可能再成为左边元素的答案）；
3. 此时栈顶就是右边第一个严格大于 $a_i$ 的元素，记录答案；
4. 把 $i$ 压入栈。

### 1.3 文字示意图

数组 `a = [3, 1, 4, 2, 5]`，求每个元素右边第一个更大的元素。从右往左扫描：

```
i=4, a[4]=5:  栈空 → 答案 0，压入 4
              栈（底→顶）: [4(5)]

i=3, a[3]=2:  栈顶 5 > 2，答案 = 4（值 5）
              压入 3
              栈: [4(5), 3(2)]

i=2, a[2]=4:  栈顶 2 <= 4，弹出 3；栈顶 5 > 4，答案 = 4（值 5）
              压入 2
              栈: [4(5), 2(4)]

i=1, a[1]=1:  栈顶 4 > 1，答案 = 2（值 4）
              压入 1
              栈: [4(5), 2(4), 1(1)]

i=0, a[0]=3:  栈顶 1 <= 3，弹出 1；栈顶 4 > 3，答案 = 2（值 4）
              压入 0
              栈: [4(5), 2(4), 0(3)]
```

结果：`[4, 2, 4, 4, 0]`（下标从 0 开始，0 表示不存在）。

对应的「柱子」直觉：

```
值
5 |                 █
4 |         █       █
3 | █       █       █
2 | █   █   █   █   █
1 | █   █   █   █   █
  +-------------------
    0 1 2 3 4   下标

a[0]=3 右边第一个比它高的是 a[2]=4
a[1]=1 右边第一个比它高的是 a[2]=4
a[2]=4 右边第一个比它高的是 a[4]=5
a[3]=2 右边第一个比它高的是 a[4]=5
```

### 1.4 单调栈能解决哪些问题

| 问题类型 | 使用哪种单调栈 |
|---|---|
| 右/左边第一个更大元素（NGE） | 单调递减栈 |
| 右/左边第一个更小元素（NSE） | 单调递增栈 |
| 柱状图中最大的矩形 | 单调递增栈（找左右第一个更矮） |
| 接雨水 | 单调递减栈 或 双指针 |
| 最大矩形（二进制矩阵） | 逐行转柱状图 + 单调栈 |
| 去掉 k 位数字使结果最小 | 单调递增栈（贪心） |
| 股票价格跨度 | 单调递减栈存 `(price, count)` |
| 贡献法统计子数组最值 | 单调栈求每个元素的「管辖区间」 |

### 1.5 重要细节：相等元素怎么处理

求「左边第一个严格小于」和「左边第一个小于等于」时，弹栈条件里的 `<` 和 `<=` 不同，会影响**相等元素归属于谁**。

- 若希望相等元素把区间「让」给左边（避免重复计数），左侧用 `<`、右侧用 `<=`；
- 做「所有子数组最小值之和」时，必须规定每个子数组的最小值归属于**最左边**或**最右边**的那个相等元素，否则会重复统计。

这是单调栈最容易出错的地方，写题时一定要想清楚「等于号放哪边」。

---

## 2. 模板代码

### 2.1 右边第一个更大元素（从右往左）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> a(n + 1), ans(n + 1, 0);
    for (int i = 1; i <= n; ++i) cin >> a[i];

    stack<int> st;                       // 存下标，栈内对应值递减
    for (int i = n; i >= 1; --i) {
        // 栈内比 a[i] 小的都不可能是答案，弹出
        while (!st.empty() && a[st.top()] <= a[i]) st.pop();
        ans[i] = st.empty() ? 0 : st.top();   // 栈顶就是右边第一个更大元素
        st.push(i);
    }

    for (int i = 1; i <= n; ++i) cout << ans[i] << " \n"[i == n];
    return 0;
}
```

输入：

```
5
3 1 4 2 5
```

输出（1 起始下标，0 表示不存在）：

```
3 3 5 5 0
```

### 2.2 左边第一个更小元素（从左往右）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> a(n + 1), ans(n + 1, 0);
    for (int i = 1; i <= n; ++i) cin >> a[i];

    stack<int> st;                       // 栈内对应值递增
    for (int i = 1; i <= n; ++i) {
        while (!st.empty() && a[st.top()] >= a[i]) st.pop();  // 更小的才留
        ans[i] = st.empty() ? 0 : st.top();
        st.push(i);
    }

    for (int i = 1; i <= n; ++i) cout << ans[i] << " \n"[i == n];
    return 0;
}
```

### 2.3 通用单调栈封装

```cpp
#include <bits/stdc++.h>
using namespace std;

// 返回每个位置左边第一个满足 cmp 的元素下标（0 表示不存在）
// cmp(a[j], a[i]) == true 表示 j 可以成为 i 的答案
template <class T, class Cmp>
vector<int> previousGreater(const vector<T>& a, Cmp cmp) {
    int n = (int)a.size() - 1;           // a 下标从 1 开始
    vector<int> res(n + 1, 0);
    vector<int> st;                      // 手动数组栈，比 std::stack 更快
    st.reserve(n);
    for (int i = 1; i <= n; ++i) {
        while (!st.empty() && !cmp(a[st.back()], a[i])) st.pop_back();
        res[i] = st.empty() ? 0 : st.back();
        st.push_back(i);
    }
    return res;
}

int main() {
    vector<int> a = {0, 3, 1, 4, 2, 5};   // 下标 0 占位
    // 左边第一个严格更大：cmp(x, y) 表示 x > y
    auto res = previousGreater(a, [](int x, int y) { return x > y; });
    for (int i = 1; i <= 5; ++i) cout << res[i] << " \n"[i == 5];
    return 0;
}
```

---

## 3. 经典例题

### 3.1 柱状图中最大的矩形

**题意**：给定 $n$ 个非负整数表示柱子的高度，每个柱子宽度为 1，求能勾勒出的最大矩形面积。

**思路**：枚举「矩形的高度由哪根柱子决定」。对于柱子 $i$，以 $a_i$ 为高的矩形能向左右扩展到**第一个比它矮的柱子**为止。设左边界为 $L_i$、右边界为 $R_i$，则面积为

$$
S_i = a_i \times (R_i - L_i - 1)
$$

用单调递增栈分别求出每个位置的左、右第一个更矮位置，再取最大值。

**复杂度**：时间 $O(n)$，空间 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<long long> a(n + 2, 0);        // 首尾加 0，省去边界特判
    for (int i = 1; i <= n; ++i) cin >> a[i];

    vector<int> L(n + 2), R(n + 2);
    vector<int> st;

    // 左边第一个严格更矮
    st.clear();
    for (int i = 1; i <= n + 1; ++i) {
        while (!st.empty() && a[st.back()] >= a[i]) st.pop_back();
        L[i] = st.empty() ? 0 : st.back();
        st.push_back(i);
    }

    // 右边第一个严格更矮
    st.clear();
    for (int i = n; i >= 0; --i) {
        while (!st.empty() && a[st.back()] >= a[i]) st.pop_back();
        R[i] = st.empty() ? n + 1 : st.back();
        st.push_back(i);
    }

    long long ans = 0;
    for (int i = 1; i <= n; ++i)
        ans = max(ans, a[i] * (long long)(R[i] - L[i] - 1));

    cout << ans << '\n';
    return 0;
}
```

> 也可以只扫一遍，在弹栈时结算面积，代码更短。

### 3.2 接雨水

**题意**：给定 $n$ 个非负整数表示每个宽度为 1 的柱子的高度图，求下雨后能接多少雨水。

**思路（单调栈）**：维护一个单调递减栈（存下标）。当遇到比栈顶高的柱子时，说明形成了一个「凹槽」：

- 弹出栈顶 `mid`，它是凹槽底部；
- 若栈仍非空，左边界为新的栈顶 `left`，右边界为当前 `i`；
- 水的高度为 `min(a[left], a[i]) - a[mid]`，宽度为 `i - left - 1`。

**复杂度**：时间 $O(n)$，空间 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<long long> h(n);
    for (auto& x : h) cin >> x;

    stack<int> st;          // 下标栈，对应高度递减
    long long ans = 0;
    for (int i = 0; i < n; ++i) {
        while (!st.empty() && h[st.top()] < h[i]) {
            int mid = st.top();
            st.pop();
            if (st.empty()) break;               // 左边没有挡板
            int left = st.top();
            long long height = min(h[left], h[i]) - h[mid];
            long long width = i - left - 1;
            ans += height * width;
        }
        st.push(i);
    }
    cout << ans << '\n';
    return 0;
}
```

### 3.3 每日温度

**题意**：给定气温数组 `T`，对每天求「还要等多少天才会出现更高的温度」，如果之后都不会升高则输出 0。

**思路**：标准的「右边第一个更大」问题，从右往左单调栈或从左往右在弹栈时结算。

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    vector<int> T(n);
    for (int& x : T) cin >> x;

    vector<int> ans(n, 0);
    stack<int> st;
    for (int i = n - 1; i >= 0; --i) {
        while (!st.empty() && T[st.top()] <= T[i]) st.pop();
        ans[i] = st.empty() ? 0 : st.top() - i;
        st.push(i);
    }

    for (int i = 0; i < n; ++i) cout << ans[i] << " \n"[i == n - 1];
    return 0;
}
```

### 3.4 去掉 K 位数字使剩下的数最小

**题意**：给定一个以字符串表示的非负整数 `num` 和整数 $k$，移除 $k$ 个数字使剩下的数字最小。

**思路**：贪心 + 单调递增栈。从左到右扫描，若当前数字比栈顶小，说明「保留栈顶不如保留当前更优」，就弹出栈顶（消耗一次删除机会）。扫描完后若还有删除次数，从尾部删。最后去掉前导零。

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

string removeKdigits(string num, int k) {
    string st;                       // 用 string 当栈，方便最后取结果
    for (char c : num) {
        while (!st.empty() && k > 0 && st.back() > c) {
            st.pop_back();
            --k;
        }
        st.push_back(c);
    }
    while (k-- > 0 && !st.empty()) st.pop_back();   // 剩余次数从尾部删

    int pos = 0;
    while (pos < (int)st.size() && st[pos] == '0') ++pos;   // 去前导零
    st = st.substr(pos);
    return st.empty() ? "0" : st;
}

int main() {
    cout << removeKdigits("1432219", 3) << '\n';   // 1219
    cout << removeKdigits("10200", 1) << '\n';     // 200
    cout << removeKdigits("10", 2) << '\n';        // 0
    return 0;
}
```

---

## 4. 易错点

1. **栈里存下标还是存值**：求距离、求区间时必须存**下标**；只求值可以存值。混用会导致答案错误。
2. **单调方向搞反**：求「第一个更大」用递减栈（弹掉 `<=` 当前值的），求「第一个更小」用递增栈（弹掉 `>=` 当前值的）。
3. **相等元素的处理**：`<` 与 `<=` 的差别会改变「相等元素归属于谁」，做贡献法统计时尤其关键，必须让每个子数组只被统计一次。
4. **遍历方向**：从右往左求「右边第一个」，从左往右求「左边第一个」；也可以从左往右用「弹栈时结算」的写法求右边第一个。
5. **栈空时的答案**：栈空表示不存在答案，通常记为 0 或 $n+1$，要按题目输出格式处理。
6. **用 `std::stack` 无法随机访问**：需要 `st[0]`、`st[1]` 等操作时用 `vector` 模拟栈。
7. **忘记 `long long`**：柱状图面积最大可达 $10^5 \times 10^9 = 10^{14}$，必须开 `long long`。
8. **只弹一次 vs 循环弹**：必须用 `while` 而不是 `if`，否则栈的单调性会被破坏。

---

## 5. 练习建议

### 5.1 由易到难的路线

1. **模板题**：右边第一个更大元素、左边第一个更小元素，先用暴力再优化到 $O(n)$。
2. **距离类**：每日温度、下一个更大元素 II（循环数组，复制一遍或取模）。
3. **面积类**：柱状图中最大的矩形 → 最大矩形（矩阵逐行转柱状图）。
4. **接水类**：接雨水（单调栈 / 双指针 / 前后缀最大值三种解法对比）。
5. **贪心类**：去掉 K 位数字、移掉 K 位数字使结果最小、构造最小序列。
6. **贡献法**：所有子数组最小值之和、最大值之和、子数组最小乘积。
7. **进阶**：单调栈 + 线段树 / 笛卡尔树、最大子矩阵、股票价格跨度。

### 5.2 推荐题目

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P5788 单调栈模板 | 右边第一个更大 |
| 基础 | LeetCode 739 每日温度 | 距离 |
| 基础 | LeetCode 496 下一个更大元素 I | 哈希 + 单调栈 |
| 提高 | LeetCode 84 柱状图中最大的矩形 | 左右边界 |
| 提高 | LeetCode 42 接雨水 | 单调栈 / 双指针 |
| 提高 | LeetCode 85 最大矩形 | 逐行柱状图 |
| 提高 | LeetCode 402 移掉 K 位数字 | 贪心 + 栈 |
| 进阶 | LeetCode 907 子数组的最小值之和 | 贡献法 |

### 5.3 学习检查清单

- [ ] 能徒手模拟单调栈的入栈、弹栈过程
- [ ] 能说清为什么每个元素最多进出栈一次、总复杂度 $O(n)$
- [ ] 能默写「右边第一个更大」和「左边第一个更小」两个模板
- [ ] 理解 `<=` 与 `<` 对相等元素归属的影响
- [ ] 能独立完成柱状图最大矩形和接雨水
- [ ] 知道什么时候该用 `vector` 代替 `std::stack`

> **核心思想**：单调栈通过「及时弹掉不可能成为答案的元素」把暴力 $O(n^2)$ 降为 $O(n)$——本质是用栈维护一个「候选答案的单调序列」，让每个元素只进出一遍。
