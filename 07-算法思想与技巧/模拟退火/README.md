# 模拟退火（Simulated Annealing）

> 模拟退火是一种**随机化搜索**算法：以一定概率接受「更差」的解，从而跳出局部最优。名字来自金属退火——高温时原子剧烈运动，缓慢降温后趋于稳定。

---

## 目录

1. [爬山算法与它的困境](#1-爬山算法与它的困境)
2. [模拟退火的原理](#2-模拟退火的原理)
3. [算法流程](#3-算法流程)
4. [参数设置](#4-参数设置)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [优化技巧](#7-优化技巧)
8. [易错点与练习](#8-易错点与练习)

---

## 1. 爬山算法与它的困境

**爬山算法**：每次随机扰动当前解，如果更优就接受，否则拒绝。

问题：很容易卡在**局部最优**。

```
目标函数图像：
      ╱╲
     ╱  ╲      ╱╲
    ╱    ╲    ╱  ╲
   ╱      ╲__╱    ╲
  ─────────────────→
      局部最优  全局最优
```

从左边出发，爬山会停在第一个峰，永远到不了更高的峰。

---

## 2. 模拟退火的原理

**核心思想**：以概率接受更差的解，概率随「温度」降低而减小。

设当前解 $s$，新解 $s'$，目标函数差 $\Delta = f(s') - f(s)$（求最大值）：

- 若 $\Delta \ge 0$：直接接受；
- 若 $\Delta < 0$：以概率 $e^{\Delta / T}$ 接受，其中 $T$ 是当前温度。

**温度 $T$ 的作用**：

- $T$ 大：$e^{\Delta/T}$ 接近 1，几乎无脑接受，**全局探索**；
- $T$ 小：$e^{\Delta/T}$ 接近 0，只接受更优解，**局部收敛**。

随着迭代进行，$T$ 从 $T_0$ 缓慢降到 $T_{end}$，算法从「乱逛」变为「精修」。

---

## 3. 算法流程

```
初始化：随机一个解 s，设定初始温度 T = T0，降温系数 delta
while T > Tend:
    for i in 1..iterations:
        s' = 扰动(s)              // 随机邻域解
        Δ = f(s') - f(s)
        if Δ > 0 or rand() < exp(Δ / T):
            s = s'
            更新全局最优
    T *= delta                    // 降温
输出全局最优
```

---

## 4. 参数设置

| 参数 | 典型值 | 说明 |
|---|---|---|
| $T_0$ | $10^4 \sim 10^6$ | 初始温度，与目标函数量级相关 |
| $T_{end}$ | $10^{-4} \sim 10^{-2}$ | 终止温度 |
| $\delta$ | $0.99 \sim 0.999$ | 降温系数，越接近 1 越慢 |
| 每温度迭代次数 | $100 \sim 1000$ | 保证充分探索 |

**调参经验**：

- 时间限制紧：$\delta = 0.99$，迭代次数少；
- 精度要求高：$\delta = 0.999$，迭代次数多；
- 最好**跑多次取最优**（比赛里可用 `while (clock() < 0.9 * CLOCKS_PER_SEC)`）。

---

## 5. 模板代码

### 5.1 通用框架

```cpp
#include <bits/stdc++.h>
using namespace std;

mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
double rand01() { return uniform_real_distribution<double>(0, 1)(rng); }
double randRange(double l, double r) { return uniform_real_distribution<double>(l, r)(rng); }

const double T0 = 1e5, Tend = 1e-4, DELTA = 0.999;
const int ITER = 300;

double ans = 1e18;   // 全局最优
// 解的结构自己定义，例如 vector<int> cur, best;

double calc() { /* 计算目标函数 */ }

void sa() {
    double T = T0;
    while (T > Tend) {
        for (int i = 0; i < ITER; ++i) {
            // 1. 生成邻域解
            // 2. 计算 Δ
            // 3. 按概率接受
        }
        T *= DELTA;
    }
}
```

### 5.2 一维函数极值（演示）

```cpp
#include <bits/stdc++.h>
using namespace std;

mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
double rand01() { return uniform_real_distribution<double>(0, 1)(rng); }
double randRange(double l, double r) { return uniform_real_distribution<double>(l, r)(rng); }

double f(double x) {
    return -x * x + 3 * x - 2;   // 最大值在 x = 1.5
}

int main() {
    double cur = randRange(-10, 10);
    double best = cur;
    double T = 100;
    while (T > 1e-6) {
        double nxt = cur + randRange(-1, 1) * T;
        double d = f(nxt) - f(cur);
        if (d > 0 || rand01() < exp(d / T)) cur = nxt;
        if (f(cur) > f(best)) best = cur;
        T *= 0.999;
    }
    printf("%.6f\n", best);
    return 0;
}
```

### 5.3 数独求解（应用）

```cpp
// 每个空格随机填 1~9，冲突数作为代价
// 每次随机选两个格子交换，冲突数减少则接受
```

---

## 6. 经典例题

### 6.1 最小圆覆盖（近似）

**题意**：给 $n$ 个点求覆盖所有点的最小圆。

**退火解法**：随机圆心，扰动半径与圆心，代价为「最远点距离 - 半径」。**精确解用随机增量法**，退火只作为理解示例。

### 6.2 旅行商问题（TSP）

**题意**：$n$ 个城市，求最短哈密顿回路。

**退火解法**：

- 解：一个排列；
- 邻域：交换两个城市；
- 目标：路径总长度。

```cpp
int n, p[N];
double dist(int a, int b);

double calc() {
    double s = 0;
    for (int i = 0; i < n; ++i)
        s += dist(p[i], p[(i+1) % n]);
    return s;
}

void saTSP() {
    double T = 1e6;
    while (T > 1e-4) {
        int i = rng() % n, j = rng() % n;
        if (i == j) continue;
        swap(p[i], p[j]);
        double d = calc() - curCost;
        if (d < 0 || rand01() < exp(-d / T)) {   // 求最小
            curCost += d;
            bestCost = min(bestCost, curCost);
        } else {
            swap(p[i], p[j]);   // 撤销
        }
        T *= 0.999;
    }
}
```

### 6.3 模拟退火 + 网络流（经典组合）

「最小生成树 + 度数限制」类问题，用退火调整度数限制。

---

## 7. 优化技巧

1. **多次退火**：跑 5~10 次取最优，加 `clock()` 控制；
2. **精调邻域**：邻域太小容易卡，太大会乱跳；
3. **记录历史最优**：不要只看当前解；
4. **温度自适应**：根据接受率调整；
5. **固定随机种子**：调试时用固定种子复现；
6. **使用 `clock()`**：避免超时。

```cpp
while ((double)clock() / CLOCKS_PER_SEC < 0.95) {
    sa();
}
```

---

## 8. 易错点与练习

### 易错点

1. **温度太高**：迟迟不收敛；
2. **温度太低**：迅速卡局部最优；
3. **迭代次数太少**：没探索够；
4. **没有记录全局最优**：最终输出的是局部最优；
5. **随机种子固定**：每次结果一样，失去随机性（调试时可以）；
6. **概率公式方向**：求最大值用 `exp(Δ/T)`，求最小值用 `exp(-Δ/T)`；
7. **超时**：退火是「看运气」的算法，必须用时钟控制。

### 练习建议

1. 先用一维函数练手；
2. TSP 小规模（$n \le 20$）；
3. 最小圆覆盖（对比精确解）；
4. 带约束的优化问题（如「最多选 $k$ 个」）；
5. 多跑几次，体会随机性。

> **注意**：正式比赛中模拟退火只能作为**部分分**或**兜底**手段。能写出确定算法就写确定算法。
