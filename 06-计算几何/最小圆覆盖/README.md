# 最小圆覆盖（Minimum Enclosing Circle）

> 求能覆盖平面上所有点的**半径最小的圆**。经典解法是**随机增量法**，期望复杂度 $O(n)$，代码极短。

---

## 目录

1. [问题定义](#1-问题定义)
2. [核心思想：三点确定圆](#2-核心思想三点确定圆)
3. [随机增量算法](#3-随机增量算法)
4. [模板代码](#4-模板代码)
5. [复杂度分析](#5-复杂度分析)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)

---

## 1. 问题定义

给定 $n$ 个点，求一个圆 $(O, r)$，使得所有点都在圆内（含边界），且 $r$ 最小。

**关键性质**：最小覆盖圆一定由**两个点**（直径）或**三个点**（外接圆）唯一确定。

---

## 2. 核心思想：三点确定圆

### 2.1 两点确定圆

以两点连线为直径：

```cpp
Circle circleFrom2(const Point& a, const Point& b) {
    Point o = (a + b) / 2;
    return Circle(o, dist(a, b) / 2);
}
```

### 2.2 三点确定圆（外接圆）

三角形外心公式。设 $A, B, C$：

$$
O = \frac{(|A|^2(B-C) + |B|^2(C-A) + |C|^2(A-B))}{2(A \times B + B \times C + C \times A)}
$$

（向量形式，$|A|^2$ 指 $x_A^2 + y_A^2$）

```cpp
Circle circleFrom3(const Point& a, const Point& b, const Point& c) {
    double A = b.x - a.x, B = b.y - a.y;
    double C = c.x - a.x, D = c.y - a.y;
    double E = A * (a.x + b.x) + B * (a.y + b.y);
    double F = C * (a.x + c.x) + D * (a.y + c.y);
    double G = 2 * (A * (c.y - b.y) - B * (c.x - b.x));
    Point o((D * E - B * F) / G, (A * F - C * E) / G);
    return Circle(o, dist(a, o));
}
```

**另一种写法**（更稳健，用叉积）：

```cpp
Circle circleFrom3(const Point& a, const Point& b, const Point& c) {
    Point p1 = b - a, p2 = c - a;
    double d = 2 * cross(p1, p2);
    double ux = (p1.y * dot(p2, p2) - p2.y * dot(p1, p1)) / d;
    double uy = (p2.x * dot(p1, p1) - p1.x * dot(p2, p2)) / d;
    Point o(a.x + ux, a.y + uy);
    return Circle(o, dist(a, o));
}
```

---

## 3. 随机增量算法

**算法流程**（类似「最小生成圆」）：

1. **随机打乱**所有点（关键！保证期望复杂度）；
2. 初始化圆为点 $p_0$，半径为 $0$；
3. 对每个点 $p_i$：
   - 若 $p_i$ 在当前圆内，跳过；
   - 否则，以 $p_i$ 为圆心、半径 $0$ 重新开始：
     - 对每个 $j < i$：
       - 若 $p_j$ 不在当前圆内，以 $p_i p_j$ 为直径建圆；
       - 对每个 $k < j$：
         - 若 $p_k$ 不在当前圆内，以 $p_i p_j p_k$ 三点建圆；
4. 输出圆。

**为什么正确？** 若 $p_i$ 在圆外，新的最小覆盖圆一定经过 $p_i$；同理每次扩展都确定了一个边界点。

---

## 4. 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const double EPS = 1e-9;

struct Point {
    double x, y;
    Point() {}
    Point(double x, double y) : x(x), y(y) {}
    Point operator+(const Point& b) const { return Point(x+b.x, y+b.y); }
    Point operator-(const Point& b) const { return Point(x-b.x, y-b.y); }
    Point operator/(double k) const { return Point(x/k, y/k); }
};

double dot(const Point& a, const Point& b) { return a.x*b.x + a.y*b.y; }
double cross(const Point& a, const Point& b) { return a.x*b.y - a.y*b.x; }
double dist(const Point& a, const Point& b) {
    return sqrt((a.x-b.x)*(a.x-b.x) + (a.y-b.y)*(a.y-b.y));
}

struct Circle {
    Point o;
    double r;
    Circle() {}
    Circle(Point o, double r) : o(o), r(r) {}
    bool contains(const Point& p) const { return dist(o, p) <= r + EPS; }
};

Circle circleFrom2(const Point& a, const Point& b) {
    Point o = (a + b) / 2;
    return Circle(o, dist(a, b) / 2);
}

Circle circleFrom3(const Point& a, const Point& b, const Point& c) {
    Point p1 = b - a, p2 = c - a;
    double d = 2 * cross(p1, p2);
    double ux = (p1.y * dot(p2, p2) - p2.y * dot(p1, p1)) / d;
    double uy = (p2.x * dot(p1, p1) - p1.x * dot(p2, p2)) / d;
    Point o(a.x + ux, a.y + uy);
    return Circle(o, dist(a, o));
}

// 返回最小覆盖圆
Circle minEnclosingCircle(vector<Point> p) {
    int n = p.size();
    if (n == 0) return Circle(Point(0,0), 0);
    if (n == 1) return Circle(p[0], 0);
    random_shuffle(p.begin(), p.end());
    // 或用: shuffle(p.begin(), p.end(), mt19937(...));

    Circle c(p[0], 0);
    for (int i = 1; i < n; ++i) {
        if (c.contains(p[i])) continue;
        c = Circle(p[i], 0);
        for (int j = 0; j < i; ++j) {
            if (c.contains(p[j])) continue;
            c = circleFrom2(p[i], p[j]);
            for (int k = 0; k < j; ++k) {
                if (c.contains(p[k])) continue;
                c = circleFrom3(p[i], p[j], p[k]);
            }
        }
    }
    return c;
}

int main() {
    int n;
    scanf("%d", &n);
    vector<Point> p(n);
    for (int i = 0; i < n; ++i) scanf("%lf%lf", &p[i].x, &p[i].y);
    Circle c = minEnclosingCircle(p);
    printf("%.10f\n%.10f %.10f\n", c.r, c.o.x, c.o.y);
    return 0;
}
```

---

## 5. 复杂度分析

- 每层循环都是「增量」：只有在点跑到圆外时才进入下一层；
- 随机打乱后，每个点成为「边界点」的期望次数是 $O(1)$；
- **期望总复杂度**：$O(n)$。

> **重要**：必须先**随机打乱**。如果点按特殊顺序给出（如按圆周排列），最坏是 $O(n^3)$。

---

## 6. 经典例题

### 6.1 【模板】最小圆覆盖

**题意**：$n \le 10^5$ 个点，求最小覆盖圆的半径和圆心。

**做法**：随机增量法。

**输出**：通常要求输出半径、圆心坐标，保留 10 位小数。

### 6.2 最小覆盖圆（求半径平方）

某些题只需输出 $r^2$，避免精度问题。

### 6.3 带权最小覆盖圆

每个点有权重，求加权最小圆——**不是**随机增量法能解决的，需要其他方法（如三分）。

### 6.4 圆形区域覆盖

给定圆和点，判断覆盖关系。

---

## 7. 易错点

1. **必须随机打乱**：否则退化成 $O(n^3)$；
2. **`contains` 要加 EPS**：`dist(o,p) <= r + EPS`，否则漏点；
3. **三点共线**：`circleFrom3` 分母 `d = 0`，要避免（共线时用两点圆）；
4. **`n == 1` 或 `n == 2`**：特殊处理；
5. **`random_shuffle` 在 C++17 被移除**：用 `shuffle(p.begin(), p.end(), rng)`；
6. **输出精度**：题目一般要求 $10^{-6}$ 或 $10^{-10}$；
7. **坐标范围大**：用 `double` 足够，但乘法可能溢出，必要时 `long double`。

### C++17 的随机打乱写法

```cpp
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
shuffle(p.begin(), p.end(), rng);
```
