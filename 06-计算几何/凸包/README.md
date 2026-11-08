# 凸包（Convex Hull）

> 凸包 = 能包住所有点的**最小凸多边形**。它是计算几何最基础也最常考的结构。

---

## 目录

1. [定义与直观理解](#1-定义与直观理解)
2. [Graham 扫描法](#2-graham-扫描法)
3. [Andrew 算法（推荐）](#3-andrew-算法推荐)
4. [模板代码](#4-模板代码)
5. [凸包的应用](#5-凸包的应用)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)

---

## 1. 定义与直观理解

给定平面上的点集 $S$，**凸包**是包含 $S$ 的所有凸集中最小的一个。

想象把钉子钉在板上，用橡皮筋从外面一箍——橡皮筋形成的形状就是凸包。

**性质**：

- 凸包上的点是 $S$ 的子集；
- 凸包内部的所有点都不是顶点；
- 凸包顶点按**逆时针**顺序排列。

```
      ×                ×
    ×   ×            ×
   ×  ×     →     ×       ×
     ×               ×
        ×              ×
```

---

## 2. Graham 扫描法

**思路**：

1. 找到 $y$ 最小（$y$ 相同取 $x$ 最小）的点作为基准点 $P_0$；
2. 以 $P_0$ 为原点，对其余点按极角逆时针排序；
3. 用一个栈维护凸包：每次加入新点时，如果栈顶的两个点与新点构成**右转**（叉积 $\le 0$），就弹出栈顶；
4. 最终栈内即为凸包。

**复杂度**：$O(n \log n)$（排序主导）。

---

## 3. Andrew 算法（推荐）

**优点**：不需要极角排序，直接用坐标排序，更好写、无精度问题。

**思路**：

1. 所有点按 $(x, y)$ 排序（去重）；
2. **从左到右**扫描，维护下凸壳：若栈顶两点与新点不构成左转，就弹栈；
3. **从右到左**扫描，维护上凸壳；
4. 合并两段得到完整凸包。

```cpp
// 返回逆时针排列的凸包顶点
vector<Point> convexHull(vector<Point> p) {
    sort(p.begin(), p.end());
    p.erase(unique(p.begin(), p.end()), p.end());
    int n = p.size();
    if (n <= 1) return p;
    vector<Point> h(2 * n);
    int k = 0;
    for (int i = 0; i < n; ++i) {          // 下凸壳
        while (k >= 2 && cross(h[k-1] - h[k-2], p[i] - h[k-2]) <= EPS) --k;
        h[k++] = p[i];
    }
    for (int i = n - 2, t = k + 1; i >= 0; --i) {   // 上凸壳
        while (k >= t && cross(h[k-1] - h[k-2], p[i] - h[k-2]) <= EPS) --k;
        h[k++] = p[i];
    }
    h.resize(k - 1);   // 最后一点与起点重复
    return h;
}
```

**注意**：`<= EPS` 会**去掉共线点**；如果需要保留共线点（如求凸包周长时无所谓，但某些题需要），改成 `< -EPS`。

---

## 4. 模板代码

完整可编译版本：

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
    bool operator==(const Point& b) const { return fabs(x - b.x) < EPS && fabs(y - b.y) < EPS; }
};

double cross(const Point& a, const Point& b) { return a.x * b.y - a.y * b.x; }
double dist(const Point& a, const Point& b) {
    return sqrt((a.x-b.x)*(a.x-b.x) + (a.y-b.y)*(a.y-b.y));
}

vector<Point> convexHull(vector<Point> p) {
    sort(p.begin(), p.end());
    p.erase(unique(p.begin(), p.end()), p.end());
    int n = p.size();
    if (n <= 1) return p;
    vector<Point> h(2 * n);
    int k = 0;
    for (int i = 0; i < n; ++i) {
        while (k >= 2 && cross(h[k-1] - h[k-2], p[i] - h[k-2]) <= EPS) --k;
        h[k++] = p[i];
    }
    for (int i = n - 2, t = k + 1; i >= 0; --i) {
        while (k >= t && cross(h[k-1] - h[k-2], p[i] - h[k-2]) <= EPS) --k;
        h[k++] = p[i];
    }
    h.resize(k - 1);
    return h;
}

// 凸包周长
double perimeter(const vector<Point>& h) {
    double s = 0;
    int m = h.size();
    for (int i = 0; i < m; ++i) s += dist(h[i], h[(i+1) % m]);
    return s;
}

// 凸包面积（鞋带公式）
double area(const vector<Point>& h) {
    double s = 0;
    int m = h.size();
    for (int i = 0; i < m; ++i) s += cross(h[i], h[(i+1) % m]);
    return fabs(s) / 2;
}
```

---

## 5. 凸包的应用

### 5.1 凸包周长

「围墙问题」：给 $n$ 棵树，求围住所有树的最短围墙长度（通常加一个圆角半径 $L$，答案是凸包周长 + $2\pi L$）。

### 5.2 凸包面积

直接鞋带公式。

### 5.3 判断点是否在凸包内

- **二分法**：$O(\log n)$，利用凸包的扇形结构；
- **叉积法**：$O(n)$，逐个判断点是否在所有边的左侧。

```cpp
// O(n) 判断点 p 是否在逆时针凸包内（含边界）
bool inConvex(const vector<Point>& h, const Point& p) {
    int m = h.size();
    for (int i = 0; i < m; ++i)
        if (cross(h[(i+1)%m] - h[i], p - h[i]) < -EPS) return false;
    return true;
}
```

### 5.4 凸包 + 旋转卡壳

求凸包直径、最小矩形覆盖等，见[旋转卡壳](../旋转卡壳/README.md)。

### 5.5 动态凸包

支持插入点的凸包，用 `set` 维护上下凸壳，$O(\log n)$ 每次。

---

## 6. 经典例题

### 6.1 【模板】凸包周长

**题意**：$n$ 个点，求凸包周长。

**做法**：Andrew 算法 + 遍历求距离和。

**复杂度**：$O(n \log n)$。

### 6.2 围墙问题（Wall）

**题意**：求包围所有点的最短围墙，围墙离最近的点至少 $L$。

**答案**：凸包周长 + $2\pi L$（因为转角处的圆弧总角度为 $2\pi$）。

### 6.3 最大三角形

**题意**：在点集中选三个点，使三角形面积最大。

**做法**：凸包 + 旋转卡壳（$O(n^2)$）或三分（$O(n \log n)$）。

### 6.4 点到凸包的最近距离

**做法**：二分找到对应扇形，再算点到边距离。

---

## 7. 易错点

1. **共线点处理**：`<= EPS` 去共线点，`< -EPS` 保留；题目要求不同；
2. **全部点共线**：凸包退化成线段，要特判；
3. **只有一个点 / 两个点**：`n <= 2` 直接返回；
4. **逆时针 / 顺时针**：Andrew 算法得到逆时针；Graham 扫描也是逆时针（若极角排序正确）；
5. **EPS 太大**：坐标 $10^9$ 时 `1e-9` 相对误差不够，要用相对误差或整数运算；
6. **重复点**：先 `sort + unique`；
7. **浮点叉积精度**：大坐标时用 `long double` 或 `__int128`（整数坐标）。
