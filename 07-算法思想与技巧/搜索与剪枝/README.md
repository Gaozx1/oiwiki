# 搜索与剪枝（Search & Pruning）

> 搜索是「枚举所有可能」的暴力艺术。剪枝则是让暴力变快的灵魂——**该砍的枝必须砍干净**。

---

## 目录

1. [DFS 与 BFS](#1-dfs-与-bfs)
2. [回溯法](#2-回溯法)
3. [剪枝的四种类型](#3-剪枝的四种类型)
4. [迭代加深](#4-迭代加深)
5. [双向搜索](#5-双向搜索)
6. [A* 与 IDA*](#6-a-与-ida)
7. [经典例题](#7-经典例题)
8. [易错点与练习](#8-易错点与练习)

---

## 1. DFS 与 BFS

### 1.1 DFS（深度优先搜索）

一条路走到黑，走不通再回头。

```cpp
void dfs(int u) {
    vis[u] = true;
    for (int v : g[u])
        if (!vis[v]) dfs(v);
}
```

**特点**：实现简单、内存小（栈深度）、适合求「一条可行路径」「所有方案」。

### 1.2 BFS（广度优先搜索）

一层层扩展。

```cpp
void bfs(int s) {
    queue<int> q;
    q.push(s);
    dist[s] = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : g[u]) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }
}
```

**特点**：首次到达即最短（无权图）、适合求「最短步数」。

---

## 2. 回溯法

**核心**：做出选择 → 递归 → 撤销选择。

```cpp
void backtrack(int step) {
    if (step == n) { record(); return; }
    for (int choice : candidates) {
        if (!valid(choice)) continue;
        make(choice);
        backtrack(step + 1);
        undo(choice);
    }
}
```

### 2.1 全排列

```cpp
void permute(int step) {
    if (step == n) { print(); return; }
    for (int i = 0; i < n; ++i) {
        if (used[i]) continue;
        used[i] = true;
        p[step] = a[i];
        permute(step + 1);
        used[i] = false;
    }
}
```

### 2.2 N 皇后

```cpp
bool col[N], d1[N], d2[N];

void nqueen(int row) {
    if (row == n) { ++ans; return; }
    for (int c = 0; c < n; ++c) {
        if (col[c] || d1[row + c] || d2[row - c + n]) continue;
        col[c] = d1[row + c] = d2[row - c + n] = true;
        nqueen(row + 1);
        col[c] = d1[row + c] = d2[row - c + n] = false;
    }
}
```

---

## 3. 剪枝的四种类型

### 3.1 可行性剪枝

**当前状态下，无论怎么选都不可能满足条件** → 立刻返回。

```cpp
if (sum > target) return;                 // 和已超
if (cnt + remain < need) return;          // 剩余不够
```

### 3.2 最优性剪枝

**当前代价已经超过已知最优解** → 返回。

```cpp
if (cost >= best) return;
```

### 3.3 搜索顺序剪枝

**优先尝试更可能成功的方向**，能更快找到解。

```cpp
// 从大到小排序，先选大的
sort(a, a + n, greater<int>());
```

### 3.4 记忆化（重复状态）

**同一个状态搜索多次** → 用数组/哈希表记录结果。

```cpp
map<State, int> memo;
int dfs(State s) {
    if (memo.count(s)) return memo[s];
    // ...
    return memo[s] = res;
}
```

### 3.5 剪枝的经验

- **先写暴力，再剪枝**；
- 用 `clock()` 卡时间，别剪到超时；
- 最优性剪枝一定要**在进入递归之前**判断；
- 多测几组数据看剪枝效果。

---

## 4. 迭代加深

**问题**：DFS 可能陷入无限深的路径；BFS 内存爆炸。

**做法**：限制搜索深度，逐步加深。

```cpp
bool dfs(int u, int depth, int limit) {
    if (depth > limit) return false;
    if (u == target) return true;
    for (int v : g[u])
        if (dfs(v, depth + 1, limit)) return true;
    return false;
}

for (int limit = 1; ; ++limit)
    if (dfs(start, 0, limit)) break;
```

**适用**：答案深度不大，但宽度大。

---

## 5. 双向搜索

**问题**：从起点到终点的状态空间太大。

**做法**：从起点和终点同时搜索，在中间相遇。

**复杂度**：$O(b^{d/2})$ 而不是 $O(b^d)$。

```cpp
// meet in the middle
void dfs1(int step, int sum) { ... map[sum]++ ... }
void dfs2(int step, int sum) { ... ans += map[target - sum] ... }
```

---

## 6. A* 与 IDA*

### 6.1 A*

在 BFS 基础上加**估价函数** $h(x)$（到目标的估计距离），按 $f(x) = g(x) + h(x)$ 排序。

```cpp
struct Node {
    int g, h, id;
    bool operator<(const Node& b) const { return g + h > b.g + b.h; }   // 小根堆
};
```

**要求**：$h$ 必须**不高估**真实距离（可采纳），才能保证最优。

### 6.2 IDA*

迭代加深 + 估价函数剪枝，内存友好。

---

## 7. 经典例题

### 7.1 八皇后

**题意**：$8 \times 8$ 棋盘放 8 个皇后，互不攻击。

**做法**：回溯 + 列/对角线标记。

### 7.2 数独

**题意**：填 $9 \times 9$ 数独。

**做法**：DFS + 行/列/宫约束，选候选数最少的格子优先。

### 7.3 生日蛋糕

**题意**：$n$ 层蛋糕，体积 $V$，求最小表面积。

**做法**：DFS + 可行性/最优性剪枝（上下界估计）。

### 7.4 埃及分数

**题意**：把分数表示成若干个单位分数之和，项数最少。

**做法**：迭代加深 + 剪枝。

### 7.5 骑士巡游

**题意**：$n \times m$ 棋盘上马走遍所有格子。

**做法**：DFS + Warnsdorff 启发（优先走下一步选择少的位置）。

---

## 8. 易错点与练习

### 易错点

1. **忘记撤销标记**：回溯法的经典错误；
2. **剪枝过猛**：把正确解剪掉；
3. **DFS 爆栈**：递归深度过大，改迭代或增大栈；
4. **BFS 判重**：入队时判重而不是出队时判重，避免重复入队；
5. **状态设计**：状态要包含影响后续的所有信息；
6. **估价函数不可采纳**：A* 会出错。

### 练习建议

1. 全排列、组合、子集枚举；
2. N 皇后；
3. 数独；
4. 生日蛋糕（剪枝经典）；
5. 迭代加深（埃及分数）；
6. meet-in-the-middle（折半搜索）。
