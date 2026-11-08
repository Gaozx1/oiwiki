# 分数规划（Fractional Programming / 01 Fractional Programming）

> 分数规划 = 求形如 $\dfrac{\sum a_i x_i}{\sum b_i x_i}$ 的最值问题。经典套路是**二分答案 + 转化为判定问题**。

---

## 目录

1. [问题模型](#1-问题模型)
2. [核心转化：二分答案](#2-核心转化二分答案)
3. [01 分数规划](#3-01-分数规划)
4. [最优比率生成树](#4-最优比率生成树)
5. [最优比率环](#5-最优比率环)
6. [模板代码](#6-模板代码)
7. [经典例题](#7-经典例题)
8. [易错点与练习](#8-易错点与练习)

---

## 1. 问题模型

给定 $n$ 个物品，每个物品有收益 $a_i$ 和代价 $b_i$（$b_i > 0$）。选择集合 $S$，求：

$$
\max \frac{\sum_{i \in S} a_i}{\sum_{i \in S} b_i}
$$

**应用场景**：

- 最大化「单位花费的收益」；
- 最大化「平均成绩」；
- 最优比率生成树、最优比率环。

---

## 2. 核心转化：二分答案

设答案为 $x$。判定「是否存在选择使得 $\dfrac{\sum a_i}{\sum b_i} \ge x$」：

$$
\frac{\sum a_i}{\sum b_i} \ge x
\iff \sum a_i - x \sum b_i \ge 0
\iff \sum (a_i - x b_i) \ge 0
$$

于是问题变成：**每个物品的权值变为 $a_i - x b_i$，求最大权值和是否 $\ge 0$**。

- 若最大权值和 $\ge 0$：答案 $\ge x$，二分右移；
- 否则答案 $< x$，二分左移。

**关键**：判定问题与原问题同构，但去掉了分式，变成了线性问题。

---

## 3. 01 分数规划

每个物品**选或不选**（01 选择）：

```cpp
bool check(double x) {
    double sum = 0;
    for (int i = 1; i <= n; ++i)
        sum += a[i] - x * b[i];    // 全部选上还 < 0 则不可能
    return sum >= 0;
}
```

如果要求「恰好选 $k$ 个」，则把 $a_i - x b_i$ 排序后取前 $k$ 大：

```cpp
bool check(double x, int k) {
    vector<double> v(n);
    for (int i = 0; i < n; ++i) v[i] = a[i] - x * b[i];
    sort(v.rbegin(), v.rend());
    double sum = 0;
    for (int i = 0; i < k; ++i) sum += v[i];
    return sum >= 0;
}
```

---

## 4. 最优比率生成树

**问题**：给定无向图，每条边有收益 $a_e$ 和代价 $b_e$，求一棵生成树使 $\dfrac{\sum a_e}{\sum b_e}$ 最大。

**做法**：二分 $x$，令边权 $w_e = a_e - x b_e$，求**最大生成树**，判断总权是否 $\ge 0$。

```cpp
struct Edge { int u, v; double a, b; };

bool check(double x, int n, vector<Edge>& es) {
    vector<pair<double, pair<int,int>>> e;
    for (auto& ed : es)
        e.push_back({ ed.b * x - ed.a, {ed.u, ed.v} });   // 最小生成树取负
    sort(e.begin(), e.end());
    init(n);
    double total = 0;
    int cnt = 0;
    for (auto& [w, uv] : e) {
        if (merge(uv.first, uv.second)) {
            total += w;
            if (++cnt == n - 1) break;
        }
    }
    return total <= 0;   // 对应最大生成树权值 >= 0
}
```

---

## 5. 最优比率环

**问题**：有向图，边有收益 $a_e$ 和代价 $b_e$，求一个环使 $\dfrac{\sum a_e}{\sum b_e}$ 最大。

**做法**：二分 $x$，边权 $w_e = a_e - x b_e$，判断图中**是否存在正权环**（把权取负则判负环，用 SPFA）。

```cpp
bool spfa(int n) {        // 判负环
    for (int i = 1; i <= n; ++i) { dist[i] = 0; cnt[i] = 0; vis[i] = true; }
    queue<int> q;
    for (int i = 1; i <= n; ++i) q.push(i);
    while (!q.empty()) {
        int u = q.front(); q.pop(); vis[u] = false;
        for (auto& e : g[u]) {
            if (dist[e.v] > dist[u] + e.w) {
                dist[e.v] = dist[u] + e.w;
                cnt[e.v] = cnt[u] + 1;
                if (cnt[e.v] >= n) return true;
                if (!vis[e.v]) { q.push(e.v); vis[e.v] = true; }
            }
        }
    }
    return false;
}
```

---

## 6. 模板代码

完整 01 分数规划：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100005;
int n, k;
double a[N], b[N];

bool check(double x) {
    vector<double> v(n);
    for (int i = 0; i < n; ++i) v[i] = a[i] - x * b[i];
    sort(v.rbegin(), v.rend());
    double sum = 0;
    for (int i = 0; i < k; ++i) sum += v[i];
    return sum >= 0;
}

int main() {
    scanf("%d%d", &n, &k);
    for (int i = 0; i < n; ++i) scanf("%lf", &a[i]);
    for (int i = 0; i < n; ++i) scanf("%lf", &b[i]);

    double lo = 0, hi = 1e9;
    for (int it = 0; it < 100; ++it) {
        double mid = (lo + hi) / 2;
        if (check(mid)) lo = mid;
        else hi = mid;
    }
    printf("%.2f\n", lo);
    return 0;
}
```

---

## 7. 经典例题

### 7.1 【模板】01 分数规划

**题意**：$n$ 个物品，每个有 $a_i, b_i$，选恰好 $k$ 个使 $\sum a / \sum b$ 最大。

**复杂度**：$O(n \log n \log V)$。

### 7.2 最优比率生成树

**题意**：每条边有高度和长度，求生成树使「总高度/总长度」最大。

**做法**：二分 + 最大生成树。

### 7.3 最优比率环

**题意**：求有向图中「总收益/总代价」最大的环。

**做法**：二分 + SPFA 判正环。

### 7.4 最大平均子段

**题意**：长度为 $n$ 的序列，求长度 $\ge L$ 的平均值最大的子段。

**做法**：二分平均值，前缀和判定。

---

## 8. 易错点与练习

### 易错点

1. **二分方向**：注意求最大值还是最小值；
2. **精度**：100 次迭代通常够，若需要更高精度用 200 次；
3. **$b_i$ 可能为 0**：会导致除零，题目一般保证 $> 0$；
4. **生成树边权取负**：想清楚是最小还是最大；
5. **负环判定初始化**：SPFA 判负环时 `dist` 全设为 0，所有点入队；
6. **浮点比较**：不要用 `==`。

### 练习建议

1. 01 分数规划模板题；
2. 最优比率生成树（POJ 2728）；
3. 最优比率环（POJ 3621）；
4. 最大平均子段（二分 + 前缀和）；
5. 分数规划 + DP 混合题。
