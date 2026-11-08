# 斜率优化 DP

> 一句话概括：当转移形如 $f[i]=\min_j\{f[j]+a_i\cdot b_j+c_i+d_j\}$、同时含 $i$ 与 $j$ 的乘积项时，把每个 $j$ 看成平面上的点，把转移改写成直线截距，用**凸包**在均摊 $O(1)$ 或 $O(\log n)$ 内找到最优决策。

## 目录

1. [概念与原理](#一概念与原理)：为什么需要、化式为直线、下凸壳、队首/队尾弹出、三种情形
2. [模板代码](#二模板代码)：双单调版本、凸壳 + 二分
3. [经典例题](#三经典例题)：任务安排、玩具装箱、土地购买、仓库建设
4. [易错点](#四易错点)
5. [练习建议](#五练习建议)

---

## 一、概念与原理

### 1.1 为什么要斜率优化

单调队列优化要求转移括号内**只与 $j$ 有关**。但一大类转移形如

$$
f[i]=\min_{j<i}\Bigl\{f[j]+(s_i-s_j)^2\Bigr\}, \tag{1}
$$

展开得

$$
f[i]=s_i^2+\min_{j<i}\Bigl\{f[j]+s_j^2-2s_i s_j\Bigr\}.
$$

括号里出现 $-2s_i s_j$——**同时依赖 $i$ 与 $j$ 的乘积项**。$s_i$ 变化时每个 $j$ 的优先级都在变，单调队列"谁永远更优"的结论不再成立。

但若令每个 $j$ 对应平面上的点 $(x_j,y_j)=(s_j,\ f[j]+s_j^2)$，则括号内就是

$$
y_j-2s_i x_j,
$$

即斜率为 $2s_i$ 的直线在 $x=x_j$ 处的取值。问题变成：给定一族点，求斜率固定的直线扫过这些点时**截距的最小值**。

### 1.2 把转移式变成直线

一般形式：

$$
f[i]=\min_{j<i}\Bigl\{f[j]+A_i\cdot B_j+C_i+D_j\Bigr\}, \tag{2}
$$

其中 $A_i$ 随 $i$ 单调、$B_j$ 随 $j$ 单调。令 $x_j=B_j$、$y_j=f[j]+D_j$，则

$$
f[i]-C_i=\min_j\{y_j-A_i x_j\}.
$$

设直线斜率为 $k=A_i$，要求最小截距 $b$ 使 $y=kx+b$ 经过某个点，即 $b=y_j-kx_j$。固定 $k$ 把直线从下往上平移，**第一个碰到的点**决定最小截距；所有可能"第一个被碰到"的点构成这些点的**下凸壳**。

```
   y
   │        ·  (永远不可能最优的点)
   │      ·
   │    ●───────●        ← 下凸壳（斜率递增）
   │   ╱         ╲
   │  ●           ●
   │ ╱             ╲
   └─────────────────── x

   斜率 k 的直线从下方推上来，最先接触下凸壳的某个顶点。
   k 越大，接触点越靠右；k 越小，接触点越靠左。
```

### 1.3 决策点构成下凸壳

三个横坐标递增的点 $A,B,C$，$B$ 在**下凸壳**上当且仅当

$$
\frac{y_B-y_A}{x_B-x_A}<\frac{y_C-y_B}{x_C-x_B},
$$

即折线斜率递增（左转）。否则 $B$ 落在 $AC$ 上方，任何从下往上推的直线都会先碰到 $A$ 或 $C$。用叉积避免浮点：

$$
\vec{AB}\times\vec{AC}=(x_B-x_A)(y_C-y_A)-(y_B-y_A)(x_C-x_A).
$$

`cross(A,B,C) <= 0` 时 $B$ 在上方（或共线），应从凸壳中删去；`> 0` 才保留。

### 1.4 队首弹出：斜率比较

队首两个决策 $p_1,p_2$（$x_{p_1}<x_{p_2}$）。$p_2$ 比 $p_1$ 更优当且仅当

$$
y_{p_2}-k\,x_{p_2}<y_{p_1}-k\,x_{p_1}
\iff k>\frac{y_{p_2}-y_{p_1}}{x_{p_2}-x_{p_1}}.
$$

即当前斜率 $k$ **大于**两点连线斜率时，$p_1$ 可永久弹出（前提是 $k$ 单调递增）。

```
        ● p2
       ╱
      ╱  ← 斜率 slope(p1,p2)
     ● p1

k < slope 时 p1 更优；k > slope 时 p2 更优。
```

### 1.5 队尾弹出：维护凸壳

插入新点 $C$ 时，若队尾两点 $A,B$ 满足 `cross(A,B,C) <= 0`，说明 $B$ 在 $AC$ 上方、不在下凸壳上，弹出 $B$，重复直到条件不满足再入队。

```
插入 C 前:   A ─── B        C 在下方
                ╲___
                    C
             cross(A,B,C) <= 0 ⇒ 弹出 B

插入 C 后:   A ────────── C    （B 被删，凸壳斜率保持递增）
```

### 1.6 三种常见情形

| 情形 | 横坐标 $x_j=B_j$ | 查询斜率 $k=A_i$ | 做法 |
|---|---|---|---|
| 双单调 | 单调 | 单调 | 单调队列，队首队尾都弹，$O(n)$ |
| $x$ 单调、$k$ 不单调 | 单调 | 不单调 | 凸壳 + 队首二分，$O(n\log n)$ |
| $x$ 不单调 | 不单调 | 任意 | Li Chao 树 / CDQ 分治，$O(n\log n)$ 起 |

入门题多属**双单调**：$B_j$ 递增、$A_i$ 递增（递减可整体取负转化）。

---

## 二、模板代码

### 2.1 双单调版本

**任务安排**：$n$ 个任务排队，第 $i$ 个耗时 $t_i$、费用系数 $c_i$。分成若干批（每批连续），一批耗时为该批 $t$ 之和 $T$，费用为 $T\times$（该批及之后所有任务的 $c$ 之和）。设 $T_i=\sum_{k\le i}t_k$、$C_i=\sum_{k\le i}c_k$，$S$ 为启动时间，$f[i]$ 为前 $i$ 个任务的最小费用：

$$
f[i]=\min_{j<i}\bigl\{f[j]+(S+T_i-T_j)(C_n-C_j)\bigr\}
=\min_j\Bigl\{\underbrace{f[j]+T_jC_j-T_jC_n}_{y_j}-\underbrace{(S+T_i)}_{k}\cdot\underbrace{C_j}_{x_j}\Bigr\}+(S+T_i)C_n.
$$

$x_j=C_j$ 递增、$k=S+T_i$ 递增，双单调。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXN = 300005;
int n, S;
ll T[MAXN], C[MAXN], f[MAXN];
int q[MAXN], head, tail;

inline ll X(int j) { return C[j]; }
inline ll Y(int j) { return f[j] + T[j] * (C[j] - C[n]); }   // 注意减去 T_j*C_n

// 叉积 (B-A) x (C-A)
inline ll cross(int a, int b, int c) {
    return (X(b) - X(a)) * (Y(c) - Y(a)) - (Y(b) - Y(a)) * (X(c) - X(a));
}

int main() {
    scanf("%d%d", &n, &S);
    for (int i = 1; i <= n; ++i) {
        ll t, c;
        scanf("%lld%lld", &t, &c);
        T[i] = T[i - 1] + t;
        C[i] = C[i - 1] + c;
    }
    head = 1, tail = 0;
    q[++tail] = 0;                       // 决策 0：前 0 个任务

    for (int i = 1; i <= n; ++i) {
        ll k = S + T[i];
        while (head < tail) {            // 队首：下一个点更优就弹出
            int a = q[head], b = q[head + 1];
            if (Y(b) - k * X(b) <= Y(a) - k * X(a)) ++head;
            else break;
        }
        int j = q[head];
        f[i] = f[j] + (S + T[i] - T[j]) * (C[n] - C[j]);
        while (head < tail && cross(q[tail - 1], q[tail], i) <= 0) --tail;
        q[++tail] = i;                   // 队尾：维护下凸壳
    }
    printf("%lld\n", f[n]);
    return 0;
}
```

### 2.2 凸壳 + 二分（斜率不单调）

结构体与 2.1 相同，只需把队首弹出换成二分查询：

```cpp
// 凸壳边斜率递增：找最后一个满足"下一条边斜率 <= k"的点
int query(ll k) {
    int l = head, r = tail;
    while (l < r) {
        int mid = (l + r) >> 1;
        if (Y(q[mid + 1]) - k * X(q[mid + 1]) <= Y(q[mid]) - k * X(q[mid]))
            l = mid + 1;
        else
            r = mid;
    }
    return q[l];
}
```

每次转移前调用 `int j = query(k_i);`。注意此时**不能弹队首**，凸壳整体保留。

---

## 三、经典例题

### 例题 1：任务安排

**题意**：同 2.1 的模型，$n\le 3\times 10^5$，$1\le S,t_i,c_i\le 2^8$。

**思路**：$k=S+T_i$ 与 $x_j=C_j$ 均单调递增，直接用双单调写法。

**复杂度**：$O(n)$ 时间，$O(n)$ 空间。代码见上面的 [2.1 模板](#21-双单调版本)。

### 例题 2：玩具装箱

**题意**：$n$ 个玩具，第 $i$ 个长度 $c_i$。分成若干段，段 $[j+1,i]$ 的费用为 $\bigl(i-j-1+\sum_{k=j+1}^{i}c_k-L\bigr)^2$，求最小总费用。$n\le 5\times 10^4$。

**思路**：设 $s_i=\sum_{k\le i}c_k$、$g_i=s_i+i$、$h_i=g_i-1-L$，则

$$
f[i]=\min_{j<i}\bigl\{f[j]+(h_i-g_j)^2\bigr\}
=\min_j\Bigl\{\underbrace{f[j]+g_j^2}_{y_j}-2h_i\underbrace{g_j}_{x_j}\Bigr\}+h_i^2.
$$

$x_j=g_j$ 递增、$k=2h_i$ 递增（$c_i\ge 1$），双单调。

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXN = 50005;
int n, L;
ll s[MAXN], g[MAXN], h[MAXN], f[MAXN];
int q[MAXN], head, tail;

inline ll X(int j) { return g[j]; }
inline ll Y(int j) { return f[j] + g[j] * g[j]; }
inline ll cross(int a, int b, int c) {
    return (X(b) - X(a)) * (Y(c) - Y(a)) - (Y(b) - Y(a)) * (X(c) - X(a));
}

int main() {
    scanf("%d%d", &n, &L);
    for (int i = 1; i <= n; ++i) {
        ll c;
        scanf("%lld", &c);
        s[i] = s[i - 1] + c;
        g[i] = s[i] + i;
        h[i] = g[i] - 1 - L;
    }
    head = 1, tail = 0;
    q[++tail] = 0;
    g[0] = 0; h[0] = -1 - L; f[0] = 0;

    for (int i = 1; i <= n; ++i) {
        ll k = 2 * h[i];
        while (head < tail) {
            int a = q[head], b = q[head + 1];
            if (Y(b) - k * X(b) <= Y(a) - k * X(a)) ++head;
            else break;
        }
        int j = q[head];
        f[i] = f[j] + (h[i] - g[j]) * (h[i] - g[j]);
        while (head < tail && cross(q[tail - 1], q[tail], i) <= 0) --tail;
        q[++tail] = i;
    }
    printf("%lld\n", f[n]);
    return 0;
}
```

### 例题 3：土地购买

**题意**：$n$ 块矩形土地，第 $i$ 块长 $x_i$、宽 $y_i$。分组购买，一组费用为组内最大长 $\times$ 最大宽，求购买全部的最小总费用。$n\le 5\times 10^4$。

**思路**：先**去支配**：若 $x_i\le x_j$ 且 $y_i\le y_j$，第 $i$ 块可并入第 $j$ 块。按 $x$ 降序排序后删去被支配者，剩余土地 $x$ 递减、$y$ 递增。可以证明最优方案中同一组的土地在排序后是连续一段，于是每段费用为 $x_{\text{段首}}\cdot y_{\text{段尾}}$：

$$
f[i]=\min_{j<i}\bigl\{f[j]+b_{j+1}.x\cdot b_i.y\bigr\}.
$$

注意 $b_{j+1}.x$ 随 $j$ 递减，为保证横坐标递增，令

$$
X(j)=-b_{j+1}.x,\qquad Y(j)=f[j],\qquad k_i=b_i.y,
$$

则 $f[i]=\min_j\{Y(j)-k_iX(j)\}$，$X$ 与 $k$ 都递增，仍是双单调（维护下凸壳）。

**复杂度**：排序 $O(n\log n)$，DP $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXN = 50005;
struct Land { ll x, y; };

int n, m;
Land a[MAXN], b[MAXN];
ll f[MAXN];
int q[MAXN], head, tail;

inline ll X(int j) { return -b[j + 1].x; }   // 取负后随 j 递增
inline ll Y(int j) { return f[j]; }
inline ll cross(int p, int r, int c) {
    return (X(r) - X(p)) * (Y(c) - Y(p)) - (Y(r) - Y(p)) * (X(c) - X(p));
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) scanf("%lld%lld", &a[i].x, &a[i].y);

    sort(a + 1, a + n + 1, [](const Land &p, const Land &r) {
        return p.x != r.x ? p.x > r.x : p.y > r.y;   // x 降序，x 相同 y 降序
    });

    ll maxY = 0;
    for (int i = 1; i <= n; ++i) {                   // 删去被支配的土地
        if (a[i].y > maxY) {
            b[++m] = a[i];
            maxY = a[i].y;
        }
    }

    head = 1, tail = 0;
    q[++tail] = 0;
    f[0] = 0;

    for (int i = 1; i <= m; ++i) {
        while (head < tail) {                        // 查询：r 不劣于 p 就弹 p
            int p = q[head], r = q[head + 1];
            if (Y(r) - b[i].y * X(r) <= Y(p) - b[i].y * X(p)) ++head;
            else break;
        }
        int j = q[head];
        f[i] = f[j] + b[j + 1].x * b[i].y;

        while (head < tail && cross(q[tail - 1], q[tail], i) <= 0) --tail;
        q[++tail] = i;
    }
    printf("%lld\n", f[m]);
    return 0;
}
```

### 例题 4：仓库建设

**题意**：$n$ 个工厂排成一条线，第 $i$ 个距原点 $x_i$、有 $p_i$ 件产品，在 $i$ 建仓库费用 $c_i$。每个工厂的产品运到右侧最近的仓库，每件每单位距离运费 $1$，求最小总费用。

**思路**：设 $f[i]$ 为前 $i$ 个工厂处理完且在第 $i$ 个建仓库的最小费用，枚举上一个仓库 $j$：

$$
f[i]=c_i+\min_{j<i}\Bigl\{f[j]+\sum_{k=j+1}^{i}p_k(x_i-x_k)\Bigr\}.
$$

用 $S_i=\sum_{k\le i}p_k$、$P_i=\sum_{k\le i}p_kx_k$ 化简：

$$
f[i]=c_i+x_iS_i-P_i+\min_{j<i}\Bigl\{\underbrace{f[j]+P_j}_{y_j}-x_i\underbrace{S_j}_{x_j}\Bigr\}.
$$

双单调。

**复杂度**：$O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXN = 1000005;
int n;
ll x[MAXN], p[MAXN], c[MAXN], S[MAXN], P[MAXN], f[MAXN];
int q[MAXN], head, tail;

inline ll X(int j) { return S[j]; }
inline ll Y(int j) { return f[j] + P[j]; }
inline ll cross(int a, int b, int c) {
    return (X(b) - X(a)) * (Y(c) - Y(a)) - (Y(b) - Y(a)) * (X(c) - X(a));
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) {
        scanf("%lld%lld%lld", &x[i], &p[i], &c[i]);
        S[i] = S[i - 1] + p[i];
        P[i] = P[i - 1] + p[i] * x[i];
    }
    head = 1, tail = 0;
    q[++tail] = 0;
    for (int i = 1; i <= n; ++i) {
        while (head < tail) {
            int a = q[head], b = q[head + 1];
            if (Y(b) - x[i] * X(b) <= Y(a) - x[i] * X(a)) ++head;
            else break;
        }
        int j = q[head];
        f[i] = f[j] + P[j] - P[i] + x[i] * (S[i] - S[j]) + c[i];
        while (head < tail && cross(q[tail - 1], q[tail], i) <= 0) --tail;
        q[++tail] = i;
    }
    printf("%lld\n", f[n]);
    return 0;
}
```

---

## 四、易错点

1. **叉积方向写反。** 求最小值维护**下凸壳**：斜率递增才保留，故 `cross(A,B,C) <= 0` 时弹出 $B$；求最大值维护**上凸壳**，不等号相反。写前先画图，并用暴力对拍验证。
2. **横坐标相同。** $x_j$ 可能相等时斜率无定义，需特判只保留 $y$ 最优的那个点，否则凸壳判断出错。
3. **队首弹出条件写反。** 双单调时统一写成 `Y(b)-k*X(b) <= Y(a)-k*X(a)` 不易错。
4. **$k$ 不单调却仍弹队首。** 弹掉的队首后面可能重新变优，必须改成二分或 Li Chao 树。
5. **整数溢出。** 叉积是两个 $10^9$ 级差的乘积，结果可达 $10^{18}$，用 `long long`；更大时用 `__int128`。
6. **插入时机错。** 新决策必须在算完 $f[i]$ 之后入队，因为 $y_i$ 依赖 $f[i]$。
7. **初始化遗漏。** 队列先放决策 $0$，且 $f[0]$、$X(0)$、$Y(0)$ 要按公式正确初始化。
8. **硬套不满足形式的转移。** 只有能整理成 $y_j-k_i x_j$ 且 $x_j$ 有单调性时适用；否则用 Li Chao 树或 CDQ 分治。

## 五、练习建议

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P2365 任务安排（弱化版） | 斜率优化入门 |
| 入门 | 洛谷 P5785 [SDOI2012] 任务安排 | 斜率不单调 + 二分 |
| 基础 | 洛谷 P3195 [HNOI2008] 玩具装箱 | 经典斜率优化 |
| 基础 | 洛谷 P2120 [ZJOI2007] 仓库建设 | 前缀和 + 斜率优化 |
| 进阶 | 洛谷 P2900 [USACO08MAR] Land Acquisition G | 排序去支配 + 斜率优化 |
| 进阶 | 洛谷 P3628 [APIO2010] 特别行动队 | 二次函数展开 |
| 进阶 | 洛谷 P4072 [SDOI2016] 征途 | 斜率优化 + 方差转化 |
| 挑战 | 洛谷 P4027 [NOI2007] 货币兑换 | 斜率不单调 + 平衡树/CDQ |
| 挑战 | 洛谷 P4655 [CEOI2017] Building Bridges | Li Chao 树 / CDQ 分治 |

**刷题方法**：先推式子，把转移整理成 $f[i]=C_i+\min\{y_j-k_i x_j\}$ 的标准形式，标出 $x_j,y_j,k_i$；判断三个单调性（$x_j$、$k_i$、求 min 还是 max）决定写法；手画凸壳确认两边弹出的不等号；建议先用 `long double` 写斜率比较版对拍，确认思路后再改成叉积整数版；数据大时用 `__int128` 计算叉积，避免溢出导致的随机 WA。
