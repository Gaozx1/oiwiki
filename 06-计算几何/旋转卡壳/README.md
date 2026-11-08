# 旋转卡壳（Rotating Calipers）

> 旋转卡壳 = 用一对「卡住凸包的平行线」绕凸包旋转，同时维护最优解。名字很吓人，代码其实只有二十行。

---

## 目录

1. [核心思想](#1-核心思想)
2. [凸包直径](#2-凸包直径)
3. [最小矩形覆盖](#3-最小矩形覆盖)
4. [模板代码](#4-模板代码)
5. [经典例题](#5-经典例题)
6. [复杂度与易错点](#6-复杂度与易错点)

---

## 1. 核心思想

**对踵点对（antipodal pairs）**：凸包上存在一对平行支撑线，分别经过点 $P$ 和 $Q$，则称 $(P, Q)$ 为对踵点对。

**关键性质**：当绕凸包逆时针转动时，对踵点的位置也**单调**移动（只会向前，不会后退）。所以可以用**双指针**在 $O(n)$ 时间内枚举所有对踵点对。

```
     ╱───────────────╲
    ╱   ●         ●   ╲     两条平行线卡住凸包
   │    ↑         ↑    │
    ╲   P         Q   ╱
     ╲───────────────╱
```

---

## 2. 凸包直径

**直径** = 凸包上距离最远的点对距离。

**做法**：

1. 求出凸包（逆时针）；
2. 用双指针 $j$ 维护「距离边 $i \to i+1$ 最远的点」；
3. 旋转过程中，若 $\text{dist}(h_j, h_i) > \text{dist}(h_j, h_{i+1})$ 类似地比较面积，则 $j$ 前进；
4. 更新答案。

**判定条件**：比较三角形面积

$$
S(i, i+1, j) \quad \text{vs} \quad S(i, i+1, j+1)
$$

即 `cross(h[i+1]-h[i], h[j]-h[i])` 与下一个比较，若后者更大则 $j$ 前进。

---

## 3. 最小矩形覆盖

**题意**：求能覆盖凸包的最小面积矩形。

**做法**：枚举矩形的一条边与凸包的某条边重合（可以证明最优矩形必有一条边与凸包边平行）。对每条边，用旋转卡壳求出：

- 最远点（决定高）；
- 最左点、最右点（决定宽）。

复杂度 $O(n)$。

---

## 4. 模板代码

### 4.1 凸包直径

```cpp
#include <bits/stdc++.h>
using namespace std;

const double EPS = 1e-9;

struct Point {
    double x, y;
    Point() {}
    Point(double x, double y) : x(x), y(y) {}
    Point operator-(const Point& b) const { return Point(x - b.x, y - b.y); }
    bool operator<(const Point& b) const { return x != b.x ? x < b.x : y < b.y; }
    bool operator==(const Point& b) const { return fabs(x-b.x) < EPS && fabs(y-b.y) < EPS; }
};

double cross(const Point& a, const Point& b) { return a.x * b.y - a.y * b.x; }
double cross(const Point& o, const Point& a, const Point& b) { return cross(a-o, b-o); }
double dist2(const Point& a, const Point& b) {
    return (a.x-b.x)*(a.x-b.x) + (a.y-b.y)*(a.y-b.y);
}

vector<Point> convexHull(vector<Point> p) {
    sort(p.begin(), p.end());
    p.erase(unique(p.begin(), p.end()), p.end());
    int n = p.size();
    if (n <= 1) return p;
    vector<Point> h(2 * n);
    int k = 0;
    for (int i = 0; i < n; ++i) {
        while (k >= 2 && cross(h[k-1]-h[k-2], p[i]-h[k-2]) <= EPS) --k;
        h[k++] = p[i];
    }
    for (int i = n-2, t = k+1; i >= 0; --i) {
        while (k >= t && cross(h[k-1]-h[k-2], p[i]-h[k-2]) <= EPS) --k;
        h[k++] = p[i];
    }
    h.resize(k-1);
    return h;
}

// 返回凸包直径的平方
double diameter2(const vector<Point>& h) {
    int m = h.size();
    if (m <= 1) return 0;
    if (m == 2) return dist2(h[0], h[1]);

    double ans = 0;
    int j = 2;
    for (int i = 0; i < m; ++i) {
        int ni = (i + 1) % m;
        while (cross(h[ni]-h[i], h[(j+1)%m]-h[i]) >
               cross(h[ni]-h[i], h[j]-h[i]) + EPS) {
            j = (j + 1) % m;
        }
        ans = max(ans, max(dist2(h[i], h[j]), dist2(h[ni], h[j])));
    }
    return ans;
}
```

### 4.2 最小矩形覆盖（面积）

```cpp
double minRectangleArea(const vector<Point>& h) {
    int m = h.size();
    if (m <= 2) return 0;
    double ans = 1e100;
    int j = 1, k = 1, l = 1;
    for (int i = 0; i < m; ++i) {
        int ni = (i + 1) % m;
        Point dir = h[ni] - h[i];
        auto proj = [&](const Point& p) { return dot(dir, p - h[i]); };
        auto height = [&](const Point& p) { return fabs(cross(dir, p - h[i])); };

        while (height(h[(j+1)%m]) > height(h[j])) j = (j + 1) % m;
        while (proj(h[(k+1)%m]) > proj(h[k])) k = (k + 1) % m;
        while (proj(h[(l+1)%m]) < proj(h[l])) l = (l + 1) % m;

        double w = proj(h[k]) - proj(h[l]);
        double hh = height(h[j]) / sqrt(dot(dir, dir));
        ans = min(ans, w * hh);
    }
    return ans;
}
```

> 实现上需要 `dot` 函数（点积）。三个指针 $j, k, l$ 分别维护最远点、最右点、最左点，随 $i$ 单调移动。

---

## 5. 经典例题

### 5.1 【模板】凸包直径

**题意**：$n$ 个点，求最远点对距离。

**做法**：凸包 + 旋转卡壳，$O(n \log n)$（排序主导）。

**注意**：答案要输出平方或开方，看题目要求。

### 5.2 最小矩形覆盖

**题意**：求覆盖所有点的最小面积矩形。

**做法**：凸包 + 旋转卡壳，$O(n \log n)$。

### 5.3 最大三角形

**题意**：在凸包上选三点使面积最大。

**做法**：枚举一条底边，双指针找最高点，$O(n^2)$；或用三分优化。

### 5.4 凸包宽度

**题意**：凸包的最小宽度（两条平行支撑线的最小距离）。

**做法**：旋转卡壳找最小高度。

---

## 6. 复杂度与易错点

### 复杂度

| 操作 | 复杂度 |
|---|---|
| 求凸包 | $O(n \log n)$ |
| 旋转卡壳 | $O(n)$ |
| 总 | $O(n \log n)$ |

### 易错点

1. **双指针的单调性**：必须保证 $\text{cross}$ 比较方向正确，否则指针会「倒转」；
2. **模运算循环**：$j = (j+1) \% m$ 最多绕一圈，要防止无限循环；
3. **重复点/共线点**：先处理凸包；
4. **`while` 条件要用 `+EPS`**：纯浮点比较容易卡住；
5. **答案类型**：距离平方用 `double` 或 `long long`，避免 `sqrt` 丢精度；
6. **$m \le 2$ 特判**：不能进入循环。
