# 向量与叉积（Vector & Cross Product）

> 计算几何的地基：一切「方向」「面积」「左右关系」都靠叉积判断。

---

## 目录

1. [向量的表示](#1-向量的表示)
2. [点积](#2-点积)
3. [叉积](#3-叉积)
4. [叉积的三大用途](#4-叉积的三大用途)
5. [极角排序](#5-极角排序)
6. [模板代码](#6-模板代码)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)

---

## 1. 向量的表示

```cpp
struct Point {
    double x, y;
    Point() {}
    Point(double x, double y) : x(x), y(y) {}
    Point operator+(const Point& b) const { return Point(x + b.x, y + b.y); }
    Point operator-(const Point& b) const { return Point(x - b.x, y - b.y); }
    Point operator*(double k) const { return Point(x * k, y * k); }
    Point operator/(double k) const { return Point(x / k, y / k); }
    bool operator<(const Point& b) const { return x != b.x ? x < b.x : y < b.y; }
    bool operator==(const Point& b) const { return fabs(x - b.x) < EPS && fabs(y - b.y) < EPS; }
};
typedef Point Vector;
```

**点与向量在代码里是同一个类型**：点是位置，向量是位移，运算形式相同。

### 常用运算

| 运算 | 公式 | 含义 |
|---|---|---|
| 加减 | $(x_1 \pm x_2, y_1 \pm y_2)$ | 平移 |
| 数乘 | $(kx, ky)$ | 缩放 |
| 模长 | $\sqrt{x^2+y^2}$ | 长度 |
| 单位化 | $\frac{\vec v}{|\vec v|}$ | 方向 |

---

## 2. 点积

$$
\vec a \cdot \vec b = x_1 x_2 + y_1 y_2 = |\vec a||\vec b|\cos\theta
$$

**用途**：

- **判断夹角**：$\vec a \cdot \vec b > 0$ 锐角，$=0$ 垂直，$< 0$ 钝角；
- **求投影长度**：$\dfrac{\vec a \cdot \vec b}{|\vec b|}$；
- **求夹角余弦**：$\cos\theta = \dfrac{\vec a\cdot\vec b}{|\vec a||\vec b|}$。

```cpp
double dot(const Point& a, const Point& b) { return a.x * b.x + a.y * b.y; }
```

---

## 3. 叉积

$$
\vec a \times \vec b = x_1 y_2 - x_2 y_1 = |\vec a||\vec b|\sin\theta
$$

**几何意义**：以 $\vec a, \vec b$ 为邻边的**平行四边形的有向面积**。三角形面积就是它的一半。

```cpp
double cross(const Point& a, const Point& b) { return a.x * b.y - a.y * b.x; }
double cross(const Point& o, const Point& a, const Point& b) { return cross(a - o, b - o); }
```

### 符号判断

设 $\vec a = \overrightarrow{OA}$，$\vec b = \overrightarrow{OB}$：

- $\vec a \times \vec b > 0$：$\vec b$ 在 $\vec a$ **逆时针**方向（左侧）；
- $= 0$：共线（同向或反向）；
- $< 0$：$\vec b$ 在 $\vec a$ **顺时针**方向（右侧）。

```
        b
       /
      /
-----o------→ a      cross(a,b) > 0
```

---

## 4. 叉积的三大用途

### 4.1 判断三点方向（左转 / 右转 / 共线）

```cpp
int orient(const Point& a, const Point& b, const Point& c) {
    double v = cross(b - a, c - a);
    if (fabs(v) < EPS) return 0;   // 共线
    return v > 0 ? 1 : -1;         // 1 = 逆时针（左转），-1 = 顺时针（右转）
}
```

### 4.2 判断线段是否相交

两条线段 $AB$ 与 $CD$ 相交，当且仅当：

- $C, D$ 分居直线 $AB$ 两侧（或共线且在范围内），**且**
- $A, B$ 分居直线 $CD$ 两侧。

```cpp
bool onSegment(const Point& a, const Point& b, const Point& p) {
    return fabs(cross(b - a, p - a)) < EPS
        && min(a.x, b.x) - EPS <= p.x && p.x <= max(a.x, b.x) + EPS
        && min(a.y, b.y) - EPS <= p.y && p.y <= max(a.y, b.y) + EPS;
}

bool segmentIntersect(const Point& a, const Point& b, const Point& c, const Point& d) {
    double c1 = cross(b - a, c - a), c2 = cross(b - a, d - a);
    double c3 = cross(d - c, a - c), c4 = cross(d - c, b - c);
    if (((c1 > EPS && c2 < -EPS) || (c1 < -EPS && c2 > EPS)) &&
        ((c3 > EPS && c4 < -EPS) || (c3 < -EPS && c4 > EPS))) return true;
    if (onSegment(a, b, c) || onSegment(a, b, d) ||
        onSegment(c, d, a) || onSegment(c, d, b)) return true;
    return false;
}
```

### 4.3 求多边形面积

见 [多边形面积](../多边形面积/README.md)：鞋带公式本质就是叉积求和。

---

## 5. 极角排序

按向量与 $x$ 轴正方向的夹角排序（$[0, 2\pi)$），是凸包等算法的预处理。

```cpp
bool cmpByAngle(const Point& a, const Point& b) {
    int ha = (a.y > 0 || (a.y == 0 && a.x > 0)) ? 0 : 1;   // 上下半平面
    int hb = (b.y > 0 || (b.y == 0 && b.x > 0)) ? 0 : 1;
    if (ha != hb) return ha < hb;
    return cross(a, b) > 0;                                 // 同半平面按叉积
}
```

**另一种写法**（更常用）：用 `atan2`，简单但有精度问题且慢：

```cpp
bool cmpByAtan2(const Point& a, const Point& b) {
    return atan2(a.y, a.x) < atan2(b.y, b.x);
}
```

> 比赛推荐**叉积版**，避免精度与性能问题。

---

## 6. 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const double EPS = 1e-9;
const double PI = acos(-1.0);

struct Point {
    double x, y;
    Point() {}
    Point(double x, double y) : x(x), y(y) {}
    Point operator+(const Point& b) const { return Point(x + b.x, y + b.y); }
    Point operator-(const Point& b) const { return Point(x - b.x, y - b.y); }
    Point operator*(double k) const { return Point(x * k, y * k); }
    bool operator<(const Point& b) const { return x != b.x ? x < b.x : y < b.y; }
    bool operator==(const Point& b) const { return fabs(x - b.x) < EPS && fabs(y - b.y) < EPS; }
};

double dot(const Point& a, const Point& b) { return a.x * b.x + a.y * b.y; }
double cross(const Point& a, const Point& b) { return a.x * b.y - a.y * b.x; }
double cross(const Point& o, const Point& a, const Point& b) { return cross(a - o, b - o); }
double len(const Point& a) { return sqrt(dot(a, a)); }
double dist(const Point& a, const Point& b) { return len(a - b); }
double angle(const Point& a, const Point& b) { return acos(dot(a, b) / len(a) / len(b)); }
Point rotate(const Point& a, double rad) {
    return Point(a.x * cos(rad) - a.y * sin(rad), a.x * sin(rad) + a.y * cos(rad));
}
```

---

## 7. 经典例题

### 7.1 判断点在直线的哪一侧

**题意**：给直线 $AB$ 和点 $P$，判断 $P$ 在有向直线 $A \to B$ 的左侧、右侧还是线上。

**做法**：`cross(B-A, P-A)` 的符号。

### 7.2 凸多边形面积

**题意**：按顺序给定简单多边形顶点，求面积。

**做法**：叉积求和（见「多边形面积」）。

### 7.3 判断两线段相交

**题意**：给定两条线段，判断是否相交（含端点）。

**做法**：叉积判跨立 + 共线特判。

### 7.4 求点到直线距离

$$
d = \frac{|\overrightarrow{AP} \times \overrightarrow{AB}|}{|\overrightarrow{AB}|}
$$

```cpp
double pointToLine(const Point& a, const Point& b, const Point& p) {
    return fabs(cross(b - a, p - a)) / dist(a, b);
}
```

### 7.5 求点到线段的距离

如果垂足在段内，就是点到直线距离；否则取到两端点距离的最小值。

```cpp
double pointToSegment(const Point& a, const Point& b, const Point& p) {
    if (dot(b - a, p - a) < -EPS) return dist(a, p);
    if (dot(a - b, p - b) < -EPS) return dist(b, p);
    return fabs(cross(b - a, p - a)) / dist(a, b);
}
```

---

## 8. 易错点

1. **EPS 取多少**：一般 `1e-9`；坐标很大时用 `1e-8`；整数坐标可用 `0`。
2. **浮点比较**：永远不要 `a == b`，用 `fabs(a-b) < EPS`。
3. **叉积溢出**：坐标是 `long long` 时，叉积可能超 `long long`，要转 `__int128` 或用 `double`。
4. **逆时针 / 顺时针搞反**：画个图确认符号。
5. **共线情况漏判**：线段相交要特别处理端点重合、共线重叠。
6. **`atan2` 排序的精度**：大坐标下可能出错，优先叉积排序。
7. **极角排序半平面判断**：`y > 0` 还是 `y >= 0` 要与 `x` 一起判断，否则 $180°$ 方向会错。
