# 半平面交（Half-Plane Intersection）

> 半平面交 = 求若干个半平面的公共部分。它是**线性规划**在计算几何中的实现，也是凸包的对偶问题。

---

## 目录

1. [半平面的表示](#1-半平面的表示)
2. [核心思想：增量法 + 双端队列](#2-核心思想增量法--双端队列)
3. [S&I 算法（排序增量）](#3-si-算法排序增量)
4. [模板代码](#4-模板代码)
5. [经典例题](#5-经典例题)
6. [易错点](#6-易错点)

---

## 1. 半平面的表示

一条有向直线 $A \to B$ 把平面分成左右两侧。约定**半平面 = 直线左侧**（逆时针方向）。

用向量表示：点 $P$ 在半平面内当且仅当

$$
\overrightarrow{AB} \times \overrightarrow{AP} \ge 0
$$

```cpp
struct Line {
    Point p, v;      // 直线上一点 p，方向向量 v
    double ang;      // 极角，用于排序
    Line() {}
    Line(Point p, Point v) : p(p), v(v) { ang = atan2(v.y, v.x); }
    bool operator<(const Line& b) const { return ang < b.ang; }
};

// 点 p 在有向直线 l 的左侧（含边界）
bool onLeft(const Line& l, const Point& p) {
    return cross(l.v, p - l.p) >= -EPS;
}
```

---

## 2. 核心思想：增量法 + 双端队列

**算法流程**（类似「凸包 + 单调队列」）：

1. 按极角排序所有直线；
2. 用双端队列维护当前的交集边界；
3. 每次加入一条新直线时：
   - 若队头两条直线的交点在新直线右侧，弹出队头；
   - 若队尾两条直线的交点在新直线右侧，弹出队尾；
   - 把新直线入队；
4. 最后再用队头直线检查一次队尾；
5. 相邻直线求交，得到多边形。

**复杂度**：$O(n \log n)$（排序主导）。

---

## 3. S&I 算法（排序增量）

### 3.1 直线求交

```cpp
Point lineIntersect(const Line& a, const Line& b) {
    Point u = a.p - b.p;
    double t = cross(b.v, u) / cross(a.v, b.v);
    return a.p + a.v * t;
}
```

### 3.2 完整流程

```cpp
vector<Point> halfPlaneIntersect(vector<Line> lines) {
    sort(lines.begin(), lines.end());   // 按极角
    deque<Line> dq;
    deque<Point> pts;

    for (int i = 0; i < (int)lines.size(); ++i) {
        // 去掉极角相同的直线，保留更靠左的
        if (i > 0 && fabs(lines[i].ang - lines[i-1].ang) < EPS) {
            if (onLeft(lines[i-1], lines[i].p)) continue;   // 新线被完全覆盖
            else { dq.pop_back(); /* 旧线被覆盖 */ }
        }
        // 队尾弹
        while (pts.size() >= 2 && !onLeft(lines[i], pts.back())) {
            dq.pop_back();
            pts.pop_back();
        }
        // 队头弹
        while (pts.size() >= 2 && !onLeft(lines[i], pts.front())) {
            dq.pop_front();
            pts.pop_front();
        }
        dq.push_back(lines[i]);
        if (dq.size() >= 2) {
            pts.push_back(lineIntersect(dq[dq.size()-2], dq.back()));
        }
    }

    // 最后用队头直线检查队尾
    while (pts.size() >= 2 && !onLeft(dq.front(), pts.back())) {
        dq.pop_back();
        pts.pop_back();
    }
    while (pts.size() >= 2 && !onLeft(dq.back(), pts.front())) {
        dq.pop_front();
        pts.pop_front();
    }

    // 用队头、队尾直线求最后一个交点
    if (dq.size() >= 2) {
        pts.push_back(lineIntersect(dq.back(), dq.front()));
    }

    vector<Point> res(pts.begin(), pts.end());
    return res;
}
```

---

## 4. 模板代码

完整可编译版本：

```cpp
#include <bits/stdc++.h>
using namespace std;

const double EPS = 1e-9;
const double PI = acos(-1.0);

struct Point {
    double x, y;
    Point() {}
    Point(double x, double y) : x(x), y(y) {}
    Point operator+(const Point& b) const { return Point(x+b.x, y+b.y); }
    Point operator-(const Point& b) const { return Point(x-b.x, y-b.y); }
    Point operator*(double k) const { return Point(x*k, y*k); }
};

double cross(const Point& a, const Point& b) { return a.x*b.y - a.y*b.x; }

struct Line {
    Point p, v;
    double ang;
    Line() {}
    Line(Point p, Point v) : p(p), v(v) { ang = atan2(v.y, v.x); }
    bool operator<(const Line& b) const { return ang < b.ang; }
};

bool onLeft(const Line& l, const Point& p) {
    return cross(l.v, p - l.p) >= -EPS;
}

Point intersect(const Line& a, const Line& b) {
    double t = cross(b.v, a.p - b.p) / cross(a.v, b.v);
    return a.p + a.v * t;
}

vector<Point> halfPlaneIntersect(vector<Line> L) {
    sort(L.begin(), L.end());
    // 保留极角不同的直线
    vector<Line> ls;
    for (int i = 0; i < (int)L.size(); ++i) {
        if (ls.empty() || fabs(L[i].ang - ls.back().ang) > EPS) {
            ls.push_back(L[i]);
        } else if (onLeft(ls.back(), L[i].p)) {
            ls.back() = L[i];
        }
    }

    deque<Line> dq;
    deque<Point> pts;
    for (const Line& l : ls) {
        while (!pts.empty() && !onLeft(l, pts.back())) {
            dq.pop_back(); pts.pop_back();
        }
        while (!pts.empty() && !onLeft(l, pts.front())) {
            dq.pop_front(); pts.pop_front();
        }
        if (dq.empty()) dq.push_back(l);
        else {
            pts.push_back(intersect(dq.back(), l));
            dq.push_back(l);
        }
    }
    while (!pts.empty() && !onLeft(dq.front(), pts.back())) {
        dq.pop_back(); pts.pop_back();
    }
    while (!pts.empty() && !onLeft(dq.back(), pts.front())) {
        dq.pop_front(); pts.pop_front();
    }
    if (dq.size() >= 2) pts.push_back(intersect(dq.back(), dq.front()));

    return vector<Point>(pts.begin(), pts.end());
}

int main() {
    int n;
    scanf("%d", &n);
    vector<Line> ls;
    for (int i = 0; i < n; ++i) {
        int m;
        scanf("%d", &m);
        vector<Point> poly(m);
        for (int j = 0; j < m; ++j) scanf("%lf%lf", &poly[j].x, &poly[j].y);
        for (int j = 0; j < m; ++j)
            ls.push_back(Line(poly[j], poly[(j+1)%m] - poly[j]));
    }
    auto res = halfPlaneIntersect(ls);
    // 多边形面积
    double area = 0;
    int m = res.size();
    for (int i = 0; i < m; ++i) area += cross(res[i], res[(i+1)%m]);
    printf("%.3f\n", fabs(area) / 2);
    return 0;
}
```

> 注意：上面按「逆时针输入多边形，半平面在直线左侧」的约定。若题目给的是顺时针，要改符号。

---

## 5. 经典例题

### 5.1 【模板】多边形面积交

**题意**：给 $n$ 个多边形，求它们的公共部分面积。

**做法**：把所有边按逆时针转为半平面，直接半平面交。

### 5.2 线性规划（二维）

**题意**：给 $n$ 条约束 $a_i x + b_i y \le c_i$，最大化 $cx + dy$。

**做法**：半平面交求出可行域多边形，最优解在顶点上，枚举顶点。

### 5.3 最优建厂位置

**题意**：在若干约束下求可行点。

**做法**：半平面交。

### 5.4 半平面交 + 凸多边形

**特判**：如果半平面交结果为空，输出 $0$。

---

## 6. 易错点

1. **方向约定**：半平面在直线**左侧**还是右侧，要统一；
2. **极角相同的直线**：只保留更靠左的（位置内层）；
3. **双端队列为空时的特判**：必须保证至少两条直线；
4. **最后要检查首尾**：加入所有直线后，队头直线可能还会弹队尾；
5. **无解情况**：返回空数组；题目要求输出 $0$ 或 `-1`；
6. **精度**：`EPS` 一般取 `1e-9`，大坐标用 `1e-8`；
7. **平行线**：`cross(a.v, b.v)` 接近 0 时不能求交；
8. **多边形方向**：题目给的多边形可能是顺时针，需要反转。
