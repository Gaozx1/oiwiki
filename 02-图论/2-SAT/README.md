# 2-SAT（2-可满足性问题）

> 一句话概括：**2-SAT** 解决「一堆二元约束」的可满足性问题——每个变量只有真假两种取值，每个约束形如「$x$ 为真 或 $y$ 为假」，通过**建图 + 强连通分量**判定是否有解并构造方案。

---

## 目录

1. [问题模型](#1-问题模型)
2. [核心转化：蕴含边](#2-核心转化蕴含边)
3. [求解原理：强连通分量](#3-求解原理强连通分量)
4. [方案构造](#4-方案构造)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 问题模型

### 1.1 SAT 与 2-SAT

**SAT（布尔可满足性问题）**：给定若干个布尔变量 $x_1,\dots,x_n$ 和一组子句，每个子句是若干文字（变量或变量的否定）的析取，问是否存在一组赋值使所有子句为真。

- **2-SAT**：每个子句**恰好包含两个文字**；
- **3-SAT**：每个子句包含三个文字，是 NP 完全问题；
- 因此 2-SAT 是 SAT 中「恰好可多项式求解」的甜点区。

### 1.2 常见的二元约束

2-SAT 能处理的约束非常丰富，常见形式包括：

| 约束 | 含义 | 子句形式 |
|---|---|---|
| $x \lor y$ | $x$、$y$ 至少一个为真 | $(x\lor y)$ |
| $\lnot x \lor y$ | $x$ 为真则 $y$ 为真 | $x\Rightarrow y$ |
| $x = y$ | 两者相同 | $(x\lor\lnot y)\land(\lnot x\lor y)$ |
| $x \ne y$ | 两者不同 | $(x\lor y)\land(\lnot x\lor\lnot y)$ |
| $\lnot(x\land y)$ | 不能同时为真 | $\lnot x\lor\lnot y$ |
| $x$（单变量） | $x$ 必须为真 | $(x\lor x)$ |

### 1.3 一个直观例子

三位同学 $A,B,C$ 中，要求：

- $A$ 和 $B$ 至少去一个；
- $B$ 和 $C$ 不能都去；
- 若 $A$ 去则 $C$ 必须去。

设变量 $x_A,x_B,x_C$ 表示是否去，则有：

$$
(x_A\lor x_B)\ \land\ (\lnot x_B\lor\lnot x_C)\ \land\ (x_A\Rightarrow x_C)
$$

可以验证 $x_A=1,x_B=0,x_C=1$ 是一组解。

---

## 2. 核心转化：蕴含边

### 2.1 关键观察

子句 $(a\lor b)$ 等价于两个蕴含式：

$$
\lnot a \Rightarrow b \qquad \text{且} \qquad \lnot b \Rightarrow a
$$

**证明**：$(a\lor b)$ 为真 $\iff$ 当 $a$ 为假时 $b$ 必为真，且当 $b$ 为假时 $a$ 必为真。

这两条蕴含式是同一个约束的两种「等价视角」，缺一不可。

### 2.2 建图方式

为每个变量 $x_i$ 建两个点：

- $i$：表示 $x_i$ 为真；
- $i+n$：表示 $x_i$ 为假（即 $\lnot x_i$）。

> 约定：节点 $i$ 与节点 $i+n$ 互为「反节点」，记 $\operatorname{neg}(i)=i+n$，$\operatorname{neg}(i+n)=i$。

对每个子句 $(a\lor b)$（$a,b$ 是文字，可能是 $x_i$ 或 $\lnot x_i$），连两条有向边：

- $\lnot a \to b$
- $\lnot b \to a$

例：子句 $(x_1\lor \lnot x_2)$：
- $\lnot x_1 \to \lnot x_2$，即 $1+n \to 2+n$；
- $x_2 \to x_1$，即 $2 \to 1$。

### 2.3 文字示意图

以 $(x_1\lor x_2)$、$(\lnot x_1\lor x_2)$、$(x_1\lor\lnot x_2)$ 为例：

```
变量 x1: 真=1, 假=1'
变量 x2: 真=2, 假=2'

(x1 ∨ x2)   : 1' -> 2,   2' -> 1
(¬x1 ∨ x2)  : 1  -> 2,   2' -> 1'
(x1 ∨ ¬x2)  : 1' -> 2',  2  -> 1
```

如果图中出现「$x_1$ 能推出 $\lnot x_1$，且 $\lnot x_1$ 能推出 $x_1$」，即二者在同一个强连通分量中，则矛盾——无论 $x_1$ 取真还是假都会推出反面。

---

## 3. 求解原理：强连通分量

### 3.1 判定定理

**定理**：2-SAT 有解，当且仅当对每个变量 $i$，节点 $i$ 与 $i+n$ **不在同一个强连通分量（SCC）中**。

**证明（必要性）**：若 $i$ 与 $i+n$ 在同一 SCC 中，则 $x_i\Rightarrow\lnot x_i$ 且 $\lnot x_i\Rightarrow x_i$。无论 $x_i$ 取何值，沿着蕴含链都会推出矛盾。

**证明（充分性）**：若没有变量与其反节点同 SCC，则可以构造一组解（见第 4 节）。

### 3.2 为什么用 Tarjan 求 SCC

- 建图后节点数 $2n$，边数 $2m$；
- 用 Tarjan 或 Kosaraju 求 SCC，$O(n+m)$；
- Tarjan 一次 DFS 即可，常数小。

### 3.3 Tarjan 回顾

```cpp
int dfn[MAXN], low[MAXN], scc[MAXN], sccCnt, timer;
int stk[MAXN], top;
bool inStk[MAXN];

void tarjan(int u) {
    dfn[u] = low[u] = ++timer;
    stk[++top] = u;
    inStk[u] = true;
    for (int v : g[u]) {
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (inStk[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (low[u] == dfn[u]) {
        ++sccCnt;
        int v;
        do {
            v = stk[top--];
            inStk[v] = false;
            scc[v] = sccCnt;
        } while (v != u);
    }
}
```

---

## 4. 方案构造

### 4.1 利用 SCC 编号

Tarjan 求出的 SCC 编号是**逆拓扑序**（编号越大越靠前）。对每个变量 $i$：

$$
x_i = \begin{cases}
\text{真} & \text{若 } scc[i] < scc[i+n] \\
\text{假} & \text{若 } scc[i] > scc[i+n]
\end{cases}
$$

**直觉**：若 $x_i$ 能推出 $\lnot x_i$，则 $\lnot x_i$ 所在的 SCC 在拓扑序上更靠后，取「后出现的那个」为真值即可避免矛盾。

### 4.2 为什么这样构造正确

设变量 $i$ 有 $scc[i] < scc[i+n]$，取 $x_i$ 为真。假设这导致某条蕴含链推出矛盾，则存在路径 $i\to i+n$，即二者在同一 SCC，与假设矛盾。因此该赋值合法。

### 4.3 文字示意

```
SCC 编号（Tarjan 出栈顺序）：
  scc[1] = 3, scc[1'] = 1
  scc[2] = 2, scc[2'] = 4

则：
  x1: scc[1]=3 > scc[1']=1  →  x1 = 假
  x2: scc[2]=2 < scc[2']=4  →  x2 = 真
```

---

## 5. 模板代码

### 5.1 完整可编译模板

输入格式：

```
n m
a b
...
```

其中每个 `a b` 表示一个形如 $(x_a \lor x_b)$ 的子句，负数表示否定。输出 `YES` + 一组解，或 `NO`。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 2000010;   // 2 * 1e6 + 10

vector<int> g[MAXN];

int dfn[MAXN], low[MAXN], scc[MAXN];
int stk[MAXN], top;
bool inStk[MAXN];
int timer = 0, sccCnt = 0;

void tarjan(int u) {
    dfn[u] = low[u] = ++timer;
    stk[++top] = u;
    inStk[u] = true;
    for (int v : g[u]) {
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (inStk[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (low[u] == dfn[u]) {
        ++sccCnt;
        int v;
        do {
            v = stk[top--];
            inStk[v] = false;
            scc[v] = sccCnt;
        } while (v != u);
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    auto id = [n](int x) -> int {          // 把文字映射到节点编号
        return x > 0 ? x : -x + n;
    };
    for (int i = 1; i <= m; ++i) {
        int a, b;
        cin >> a >> b;
        int u = id(a), v = id(b);
        // (a ∨ b) 等价于 ¬a -> b 且 ¬b -> a
        g[u <= n ? u + n : u - n].push_back(v);
        g[v <= n ? v + n : v - n].push_back(u);
    }
    for (int i = 1; i <= 2 * n; ++i) {
        if (!dfn[i]) tarjan(i);
    }
    for (int i = 1; i <= n; ++i) {
        if (scc[i] == scc[i + n]) {
            cout << "NO\n";
            return 0;
        }
    }
    cout << "YES\n";
    for (int i = 1; i <= n; ++i) {
        cout << (scc[i] < scc[i + n] ? 1 : 0) << ' ';
    }
    cout << '\n';
    return 0;
}
```

### 5.2 常用建图封装

```cpp
int N;                      // 变量个数
int cnt;                    // 实际节点数（= 2N）
vector<int> g[MAXN];

// 节点编号约定：
//   变量 i (1-based) 为真: i
//   变量 i 为假: i + N
int neg(int u) { return u <= N ? u + N : u - N; }

// 添加子句 (a ∨ b)，a、b 为节点编号
void addClause(int a, int b) {
    g[neg(a)].push_back(b);
    g[neg(b)].push_back(a);
}

// 强制 x 为真
void forceTrue(int x) { addClause(x, x); }

// 强制 x 为假
void forceFalse(int x) { addClause(neg(x), neg(x)); }

// x => y（即 ¬x ∨ y）
void addImplication(int x, int y) { addClause(neg(x), y); }

// x 与 y 至少一个为真
void atLeastOne(int x, int y) { addClause(x, y); }

// x 与 y 不能同时为真
void notBoth(int x, int y) { addClause(neg(x), neg(y)); }

// x 与 y 恰好一个为真
void exactlyOne(int x, int y) {
    addClause(x, y);            // 至少一个
    addClause(neg(x), neg(y));  // 不能同时
}
```

### 5.3 复杂度

- 建图：$O(n+m)$；
- Tarjan：$O(n+m)$；
- 总复杂度 $O(n+m)$，非常高效。

---

## 6. 经典例题

### 例题 1：【模板】2-SAT 问题（Luogu P4782）

**题目大意**：有 $n$ 个布尔变量，$m$ 个形如「$x_a$ 为 $v_a$ 或 $x_b$ 为 $v_b$」的条件，判断是否有解并输出一组方案。

**思路**：直接套用第 5 节模板。注意本题的输入格式是「变量下标 + 取值」，需要自己转换文字。

**复杂度**：$O(n+m)$。

```cpp
// 输入：a va b vb，表示 x_a = va 或 x_b = vb
// 文字：若 va=1 则为 x_a，否则为 ¬x_a
int litA = va ? a : a + n;
int litB = vb ? b : b + n;
addClause(litA, litB);
```

---

### 例题 2：和平委员会（Luogu P5782 [POI2001]）

**题目大意**：$n$ 个党派，每个党派有 $2$ 名代表（共 $2n$ 人），要选出 $n$ 人组成委员会，每个党派恰好选 $1$ 人。另有 $m$ 对代表互相厌恶，不能同时入选。求一组方案。

**思路**：设变量 $x_i$ 表示「第 $i$ 个党派选第一个人」，则 $\lnot x_i$ 表示「选第二个人」。由于每个党派必须且只能选一人，而 $x_i$ 与 $\lnot x_i$ 必有一个为真，这个条件**天然满足**，不需要额外建边。

剩下只需处理「$a$ 与 $b$ 不能同时入选」：把每个代表映射成一个文字——若代表是党派 $p$ 的 1 号，文字为 $x_p$；否则为 $\lnot x_p$。于是厌恶关系 $(a,b)$ 就是 $\text{notBoth}(\text{lit}(a), \text{lit}(b))$。

**复杂度**：$O(n+m)$。

---

### 例题 3：地图（Luogu P3209 [HNOI2010] 平面图判定）

**题目大意**：给定一个带哈密顿回路的图，判断它是否为平面图。

**思路**：把哈密顿回路画成一个圆，剩下的每条边是圆上的一条弦。两条弦「相交」当且仅当它们的端点在圆上交错。每条弦可以画在圆内或圆外，设变量 $x_i$ 表示第 $i$ 条弦画在圆内。

- 对每一对相交的弦 $i,j$，它们**不能同侧**，即要求 $x_i\ne x_j$，也就是「恰好一个为真」，对应子句 $(x_i\lor x_j)\land(\lnot x_i\lor\lnot x_j)$（若同侧则两条弦相交，平面性被破坏）。

**复杂度**：$O(n+m^2)$（枚举弦对），$n\le 10^4$、$m\le 10^4$ 时需注意常数，实际可过。

---

### 例题 4：游戏（Luogu P3825 [NOI2017] 游戏）

**题目大意**：有 $n$ 个场地，每个场地适合若干种赛车（A/B/C）。有 $m$ 个限制「若场地 $i$ 使用赛车 $x$，则场地 $j$ 必须使用赛车 $y$」。求一组方案。其中最多有 $d$ 个场地「三种车都适合」，$d\le 8$。

**思路**：这是 2-SAT 的经典难题：

1. 对每个「三种都适合」的场地，枚举它**禁用**哪种车（3 种选择），共 $3^d$ 种情况；
2. 每种情况下，每个场地只剩两种可选车，用 2-SAT 建模：
   - 变量 $x_i$ 表示场地 $i$ 选第一种车；
   - 限制「$i$ 用 $x$ 则 $j$ 用 $y$」转化为蕴含边；
   - 注意「$i$ 不能使用 $x$」时该限制自动满足，需要特殊处理；
3. 任一情况有解即输出。

**复杂度**：$O(3^d\cdot(n+m))$，$d\le 8$ 时可接受。

---

## 7. 易错点

1. **文字到节点的映射写错**：约定清楚「$i$ 表示真、$i+n$ 表示假」，并用 `neg()` 统一处理，避免手写 `i+n` 时出错。
2. **蕴含边方向反了**：$(a\lor b)$ 对应 $\lnot a\to b$ 与 $\lnot b\to a$，两个方向都要连。
3. **漏掉「两个方向」**：只连一条边是常见错误，会导致判定过于宽松。
4. **SCC 编号与方案**：Tarjan 的 SCC 编号是逆拓扑序，取 `scc[i] < scc[i+n]` 为真；用 Kosaraju 时编号方向相反，需要调整。
5. **节点数开小**：$2n$ 个节点，数组开 $2n+10$。
6. **单变量约束**：$x$ 必须为真写成 $(x\lor x)$，即连 $\lnot x\to x$。
7. **多组数据清空**：`g`、`dfn`、`low`、`scc`、`stk` 全部清空。
8. **递归爆栈**：$n$ 很大时 Tarjan 递归可能爆栈，可改用迭代版或 Kosaraju。
9. **变量下标从 0 开始**：统一转成 1-based，避免 `i+n` 与 0 冲突。

---

## 8. 练习建议

由易到难：

1. **模板**：Luogu P4782 【模板】2-SAT 问题 —— 背熟建图与 Tarjan；
2. **简单建模**：Luogu P5782 [POI2001] 和平委员会 —— 练习「文字映射」；
3. **几何建模**：Luogu P3209 [HNOI2010] 平面图判定 —— 体会「相交=不同侧」的建模；
4. **枚举 + 2-SAT**：Luogu P3825 [NOI2017] 游戏 —— 经典难题；
5. **前后缀优化建图**：Luogu P6378 [PA2010] Riddle —— 2-SAT 建图优化；
6. **更多练习**：Codeforces 776D The Door Problem、Luogu P4171 [JSOI2010] 满汉全席。

学习建议：

- 先把「$i$ 真 / $i+n$ 假」的约定写死在纸上，所有建图都基于它；
- 每写一个约束，先写出对应的子句，再转成两条蕴含边，不要跳步；
- 2-SAT 的难点全在建模，建议整理一份「常见约束 → 子句」的对照表。
