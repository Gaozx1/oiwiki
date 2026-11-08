# 多边形面积（Polygon Area）

> 计算多边形面积最常用的方法：**鞋带公式**（Shoelace Formula）。一行循环就能算，还自带「有向面积」的正负号。

---

## 目录

1. [鞋带公式](#1-鞋带公式)
2. [为什么是叉积](#2-为什么是叉积)
3. [凸多边形与凹多边形](#3-凸多边形与凹多边形)
4. [模板代码](#4-模板代码)
5. [Pick 定理](#5-pick-定理)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)

---

## 1. 鞋带公式

给定平面上的 $n$ 个点 $P_1, P_2, \ldots, P_n$（按多边形顶点顺序，可顺时针可逆时针），面积为：

$$
S = \frac{1}{2}\left|\sum_{i=1}^{n} (x_i y_{i+1} - x_{i+1} y_i)\right|
$$

其中 $P_{n+1} = P_1$。

**记忆方法**：交叉相乘再相减，形似系鞋带：

```
x1  y1
     ╲╱
     ╱╲
x2  y2
```

即：$x_1 y_2 - x_2 y_1 + x_2 y_3 - x_3 y_2 + \cdots$

---

## 2. 为什么是叉积

把多边形拆成以原点为顶点的三角形，每个三角形的有向面积是：

$$
S_{\triangle OP_iP_{i+1}} = \frac{1}{2}(P_i \times P_{i+1})
$$

把所有三角形加起来，外部重叠部分正好抵消，剩下多边形本身的有向面积。

```
       P3
      ╱  ╲
     ╱    ╲
   P1──────P2
     ╲    ╱
      ╲  ╱
       P4
```

**有向面积**：

- 逆时针顶点顺序 ⇒ 正；
- 顺时针 ⇒ 负；
- 取绝对值得到实际面积。

---

## 3. 凸多边形与凹多边形

鞋带公式对**任意简单多边形**（边不自交）都成立，包括凹多边形。

```
   凹多边形：
     ┌──────┐
     │      │
     │  ┌───┘
     │  │
     └──┘
```

凹进去的部分在求和时被自动「减去」，无需特殊处理。

> 注意：对**自交多边形**，公式给出的是「有向面积」，可能不是你想要的结果。

---

## 4. 模板代码

### 4.1 基础版

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Point {
    double x, y;
    Point() {}
    Point(double x, double y) : x(x), y(y) {}
};

double polygonArea(const vector<Point>& p) {
    double s = 0;
    int n = p.size();
    for (int i = 0; i < n; ++i) {
        int j = (i + 1) % n;
        s += p[i].x * p[j].y - p[j].x * p[i].y;
    }
    return fabs(s) / 2;
}
```

### 4.2 叉积版（整数坐标也适用）

```cpp
double cross(const Point& a, const Point& b) { return a.x*b.y - a.y*b.x; }

double polygonArea2(const vector<Point>& p) {
    double s = 0;
    int n = p.size();
    for (int i = 0; i < n; ++i)
        s += cross(p[i], p[(i+1) % n]);
    return fabs(s) / 2;
}
```

### 4.3 整数坐标（避免精度问题）

```cpp
typedef long long ll;

ll polygonArea2(const vector<pair<ll,ll>>& p) {
    ll s = 0;
    int n = p.size();
    for (int i = 0; i < n; ++i) {
        int j = (i + 1) % n;
        s += p[i].first * p[j].second - p[j].first * p[i].second;
    }
    return llabs(s);   // 返回两倍面积，整数
}
```

> **技巧**：题目若要求整数面积，直接返回「两倍面积」，避免浮点误差。

### 4.4 判断顶点顺序

```cpp
// 返回 1 = 逆时针，-1 = 顺时针，0 = 退化
int orientation(const vector<Point>& p) {
    double s = 0;
    int n = p.size();
    for (int i = 0; i < n; ++i)
        s += cross(p[i], p[(i+1) % n]);
    if (fabs(s) < 1e-9) return 0;
    return s > 0 ? 1 : -1;
}
```

---

## 5. Pick 定理

对于**顶点为整点**的简单多边形：

$$
S = I + \frac{B}{2} - 1
$$

其中：

- $I$ = 多边形内部整点个数；
- $B$ = 多边形边界上的整点个数。

**边界整点数的计算**：对每条边 $(x_1,y_1) \to (x_2,y_2)$，其上的整点数为

$$
\gcd(|x_1 - x_2|, |y_1 - y_2|)
$$

（不含起点，含终点，累加即得 $B$）

```cpp
#include <numeric>
ll boundaryPoints(const vector<pair<ll,ll>>& p) {
    ll b = 0;
    int n = p.size();
    for (int i = 0; i < n; ++i) {
        int j = (i + 1) % n;
        b += gcd(abs(p[i].first - p[j].first), abs(p[i].second - p[j].second));
    }
    return b;
}
// 内部点数 I = (2S - B + 2) / 2
```

---

## 6. 经典例题

### 6.1 【模板】多边形面积

**题意**：给 $n$ 个点（按顺序），求多边形面积。

**做法**：鞋带公式。

### 6.2 判断凸多边形

**题意**：判断给定多边形是否凸。

**做法**：所有相邻叉积同号。

```cpp
bool isConvex(const vector<Point>& p) {
    int n = p.size();
    int sign = 0;
    for (int i = 0; i < n; ++i) {
        double c = cross(p[(i+1)%n] - p[i], p[(i+2)%n] - p[(i+1)%n]);
        if (fabs(c) < 1e-9) continue;
        int s = c > 0 ? 1 : -1;
        if (sign == 0) sign = s;
        else if (sign != s) return false;
    }
    return true;
}
```

### 6.3 三角形面积

**题意**：给三点求面积。

**做法**：$S = \frac{1}{2}|(B-A) \times (C-A)|$。

```cpp
double triangleArea(const Point& a, const Point& b, const Point& c) {
    return fabs(cross(b - a, c - a)) / 2;
}
```

### 6.4 求多边形重心

**题意**：求多边形重心（质心）。

**公式**：

$$
C_x = \frac{1}{6S}\sum (x_i + x_{i+1})(x_i y_{i+1} - x_{i+1} y_i)
$$

```cpp
Point centroid(const vector<Point>& p) {
    double cx = 0, cy = 0, s = 0;
    int n = p.size();
    for (int i = 0; i < n; ++i) {
        int j = (i + 1) % n;
        double c = cross(p[i], p[j]);
        s += c;
        cx += (p[i].x + p[j].x) * c;
        cy += (p[i].y + p[j].y) * c;
    }
    s /= 2;
    cx /= 6 * s;
    cy /= 6 * s;
    return Point(cx, cy);
}
```

### 6.5 Pick 定理求整点个数

**题意**：给整点多边形，求内部整点数。

**做法**：面积 + 边界点数，套 Pick 定理。

---

## 7. 易错点

1. **顶点顺序**：必须按多边形顶点顺序给出，乱序会算错；
2. **共线点**：不影响结果，但可能影响凸包判断；
3. **自交多边形**：公式给出有向面积，可能不是期望值；
4. **整数溢出**：坐标 $10^9$ 时，叉积可达 $10^{18}$，用 `__int128` 或 `long long`（注意 $n$ 次累加）；
5. **`fabs` 与 `abs`**：`double` 用 `fabs`，整数用 `abs`/`llabs`；
6. **返回两倍面积**：整数题常用，避免除以 2 的精度问题；
7. **Pick 定理只适用于简单多边形**（不自交）。
