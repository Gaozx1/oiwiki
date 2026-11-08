# 堆（Heap）入门

> 一句话概括：**堆是一种能快速取出「最值」的完全二叉树**。插入、删除最值都是 $O(\log n)$，取最值是 $O(1)$。

---

## 目录

1. [什么是堆](#1-什么是堆)
2. [堆的存储](#2-堆的存储)
3. [两个核心操作：上浮与下沉](#3-两个核心操作上浮与下沉)
4. [基本操作与复杂度](#4-基本操作与复杂度)
5. [手写堆模板（C++）](#5-手写堆模板c)
6. [STL 中的 priority_queue](#6-stl-中的-priority_queue)
7. [堆排序](#7-堆排序)
8. [经典应用](#8-经典应用)
9. [复杂度汇总](#9-复杂度汇总)
10. [常见易错点](#10-常见易错点)
11. [延伸与练习建议](#11-延伸与练习建议)

---

## 1. 什么是堆

**堆（Heap）** 是一种特殊的**完全二叉树**，它满足「堆性质」：

- **小根堆（min-heap）**：每个节点的值都 **小于等于** 它的子节点 ⇒ **根节点是整个堆的最小值**。
- **大根堆（max-heap）**：每个节点的值都 **大于等于** 它的子节点 ⇒ **根节点是整个堆的最大值**。

例如下面是一棵小根堆：

```
        1
      /   \
     3     2
    / \   /
   7   4 5
```

对应的数组为 `[1, 3, 2, 7, 4, 5]`。

### 堆 ≠ 二叉搜索树

堆**只约束父子关系**，不约束左右兄弟之间的大小。所以：

- 小根堆中，第二小的元素一定是根的两个儿子之一，但**不一定**是左儿子；
- 不能对堆做中序遍历得到有序序列；
- 想查「第 k 小」需要额外手段（见第 8 节）。

### 堆 ≠ 内存里的「堆区」

数据结构中的 heap 和操作系统内存分区中的 heap 是两回事，只是重名。

---

## 2. 堆的存储

完全二叉树有一个很好的性质：**可以用数组连续存储，不需要指针**。

以下标从 **1** 开始为例（最常用）：

| 关系 | 下标 |
|---|---|
| 根节点 | `1` |
| 节点 `i` 的左儿子 | `2i` |
| 节点 `i` 的右儿子 | `2i + 1` |
| 节点 `i` 的父亲 | `i / 2`（整除） |
| 最后一个非叶子节点 | `n / 2` |

数组 `h[1..n]` 表示上面那棵小根堆：

```
下标:  1  2  3  4  5  6
值:    1  3  2  7  4  5
```

> 若使用 0 起始下标：左儿子 `2i+1`、右儿子 `2i+2`、父亲 `(i-1)/2`。**两种写法都行，但一定不要混用**，这是新手最常见的 bug 来源。本文统一采用 **1 起始下标**。

---

## 3. 两个核心操作：上浮与下沉

堆的所有操作都建立在两个原子操作上。

### 3.1 上浮（swim / up）

**场景**：新元素被放到数组末尾，它可能比父亲小（小根堆），需要一路「往上冒」。

**做法**：不断与父亲比较，若比父亲更优（小根堆中更小）就交换，直到到达根或满足堆性质。

```
插入 0：

      1                1                0
    /   \            /   \            /   \
   3     2    →     3     2    →     3     1
  / \   / \        / \   / \        / \   / \
 7   4 5   0      7   4 5   0?     7   4 5   2
                  （0 与 2 交换）   （0 与 1 交换，停止）
```

代码：

```cpp
void swim(int i) {
    while (i > 1 && h[i] < h[i >> 1]) {   // 小根堆：比父亲小就上浮
        swap(h[i], h[i >> 1]);
        i >>= 1;
    }
}
```

### 3.2 下沉（sink / down）

**场景**：堆顶被弹出后，把数组最后一个元素搬到堆顶，它可能比儿子大（小根堆），需要一路「往下沉」。

**做法**：每次在「自己、左儿子、右儿子」中选出**最小**的，如果最小的不是自己就交换，直到成为叶子或满足堆性质。

```
弹出堆顶 1：把末尾的 5 搬到根

      5                2                2
    /   \            /   \            /   \
   3     2    →     3     5    →     3     5
  / \              / \              / \
 7   4            7   4            7   4
（5 与 2 交换）    （5 与 4 交换，停止）
```

代码：

```cpp
void sink(int i) {
    int n = size();
    while (true) {
        int l = i << 1, r = l | 1;
        int best = i;
        if (l <= n && h[l] < h[best]) best = l;
        if (r <= n && h[r] < h[best]) best = r;
        if (best == i) break;
        swap(h[i], h[best]);
        i = best;
    }
}
```

**记忆口诀**：

- 插入 ⇒ **上浮**（从下往上）
- 删除堆顶 ⇒ **下沉**（从上往下）
- 建堆 ⇒ **从最后一个非叶子节点开始，依次下沉**

---

## 4. 基本操作与复杂度

| 操作 | 实现方式 | 时间复杂度 |
|---|---|---|
| 取堆顶 `top` | 直接返回 `h[1]` | $O(1)$ |
| 插入 `push` | 放到末尾 + 上浮 | $O(\log n)$ |
| 弹出堆顶 `pop` | 末尾元素换到根 + 下沉 | $O(\log n)$ |
| 建堆 `build` | 从 `n/2` 到 `1` 依次下沉 | $O(n)$ |
| 判空 / 大小 | 维护计数器 | $O(1)$ |
| 堆排序 | 建堆 + 反复取堆顶 | $O(n \log n)$ |

### 为什么建堆是 $O(n)$ 而不是 $O(n \log n)$？

因为大部分节点都在底层，下沉距离很短。严格计算：

$$
\sum_{h=0}^{\lfloor \log n \rfloor} \frac{n}{2^{h+1}} \cdot O(h) = O\left(n \sum_{h \ge 0} \frac{h}{2^h}\right) = O(n)
$$

直观理解：**越靠上的节点越少，越靠下的节点越多**，而靠下的节点几乎不用下沉。

---

## 5. 手写堆模板（C++）

下面是一个完整、可直接使用的小根堆（支持 `long long`）：

```cpp
#include <bits/stdc++.h>
using namespace std;

struct MinHeap {
    vector<long long> h;              // h[0] 占位不用，真实元素在 h[1..n]

    MinHeap() { h.push_back(0); }

    int size() const { return (int)h.size() - 1; }
    bool empty() const { return size() == 0; }
    long long top() const { return h[1]; }   // 调用前请保证非空

    void swim(int i) {                        // 上浮
        while (i > 1 && h[i] < h[i >> 1]) {
            swap(h[i], h[i >> 1]);
            i >>= 1;
        }
    }

    void sink(int i) {                        // 下沉
        int n = size();
        while (true) {
            int l = i << 1, r = l | 1, best = i;
            if (l <= n && h[l] < h[best]) best = l;
            if (r <= n && h[r] < h[best]) best = r;
            if (best == i) break;
            swap(h[i], h[best]);
            i = best;
        }
    }

    void push(long long x) {                  // 插入
        h.push_back(x);
        swim(size());
    }

    void pop() {                              // 弹出堆顶
        if (empty()) return;
        h[1] = h.back();
        h.pop_back();
        if (!empty()) sink(1);
    }

    void build(const vector<long long>& a) {  // O(n) 建堆
        h.assign(1, 0);
        h.insert(h.end(), a.begin(), a.end());
        for (int i = size() / 2; i >= 1; --i) sink(i);
    }
};
```

### 改成大根堆

把 `swim` 和 `sink` 中的 `<` 全部改成 `>` 即可：

```cpp
void swim(int i) { while (i > 1 && h[i] > h[i >> 1]) { swap(h[i], h[i >> 1]); i >>= 1; } }
// sink 中：if (l <= n && h[l] > h[best]) ...
```

### 通用模板（带比较器）

如果想写一份代码同时支持大小根堆：

```cpp
template <class T, class Cmp = less<T>>
struct Heap {
    vector<T> h{0};          // 注意：h[0] 是占位
    Cmp cmp;

    int size() const { return (int)h.size() - 1; }
    bool empty() const { return size() == 0; }
    T top() const { return h[1]; }

    void swim(int i) {
        while (i > 1 && cmp(h[i], h[i >> 1])) {
            swap(h[i], h[i >> 1]);
            i >>= 1;
        }
    }
    void sink(int i) {
        int n = size();
        while (true) {
            int l = i << 1, r = l | 1, best = i;
            if (l <= n && cmp(h[l], h[best])) best = l;
            if (r <= n && cmp(h[r], h[best])) best = r;
            if (best == i) break;
            swap(h[i], h[best]);
            i = best;
        }
    }
    void push(const T& x) { h.push_back(x); swim(size()); }
    void pop() {
        if (empty()) return;
        h[1] = h.back(); h.pop_back();
        if (!empty()) sink(1);
    }
};

// 小根堆：Heap<int, less<int>>
// 大根堆：Heap<int, greater<int>>
```

### 支持「修改 / 删除任意元素」的堆

要点：额外维护每个元素在数组中的**位置下标** `pos[id]`。修改某元素后，比较它与父亲：比父亲更优就上浮，否则下沉；删除时把它换到末尾再调整，同时更新 `pos`。这样单次操作仍是 $O(\log n)$，常用于 Dijkstra 的手写堆优化或需要动态改键的贪心题。

---

## 6. STL 中的 priority_queue

C++ 标准库已经提供了堆容器适配器 `std::priority_queue`，绝大多数题目直接用即可。

### 6.1 基本用法

```cpp
#include <queue>
using namespace std;

priority_queue<int> pq;                        // 默认：大根堆
priority_queue<int, vector<int>, greater<int>> minpq;  // 小根堆

pq.push(5);
pq.push(3);
pq.push(8);
int x = pq.top();   // 8（最大值）
pq.pop();           // 删除堆顶 8
bool e = pq.empty();
int s = pq.size();
```

### 6.2 自定义类型与比较器

`priority_queue` 的比较器含义**和 `sort` 相反**：
`cmp(a, b) == true` 表示 `a` 的优先级**低于** `b`，即 `a` 排在 `b` 后面、更靠近堆底。所以 `greater<int>` 得到的是**小根堆**。

```cpp
struct Node {
    int dist;
    int id;
};

// 小根堆：dist 小的优先
struct Cmp {
    bool operator()(const Node& a, const Node& b) const {
        return a.dist > b.dist;    // 注意是 > ：dist 大的优先级低
    }
};
priority_queue<Node, vector<Node>, Cmp> pq;
```

对于 `pair`，默认先比较 `first`，再比较 `second`，所以 `priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>>` 是一棵按字典序的小根堆（Dijkstra 常这么写）。

### 6.3 priority_queue 的局限

- **不能遍历**、没有迭代器，只能访问堆顶；
- **不能修改**堆中任意元素，也**不能删除**任意元素；
- **没有 `clear()`**，要清空可以 `pq = priority_queue<T>();` 或循环 `pop`；
- 是大根堆/小根堆要在类型里定死，运行中不能切换。

需要以上能力时，要么手写堆，要么用「懒删除」技巧（见 8.5 节）。

### 6.4 常用技巧：懒删除（可删堆）

需要「删除任意值」但不想手写堆时，维护两个堆：

```cpp
priority_queue<int> add, del;   // 都按大根堆

void erase(int x) { del.push(x); }

int top() {
    while (!del.empty() && del.top() == add.top()) {
        del.pop();
        add.pop();
    }
    return add.top();   // 调用前保证 add 非空
}
```

思想：删除时先「记账」，取堆顶时再把已经被删掉的堆顶弹掉。每次元素最多进出一次，总复杂度仍是 $O(n \log n)$。

---

## 7. 堆排序

堆排序 = **建大根堆 + 反复把堆顶换到末尾**，原地排序，额外空间 $O(1)$，时间复杂度 $O(n \log n)$，**不稳定**。

```cpp
// 0 起始下标版本，结果升序
void siftDown(vector<int>& a, int n, int i) {
    while (true) {
        int l = 2 * i + 1, r = 2 * i + 2, best = i;
        if (l < n && a[l] > a[best]) best = l;
        if (r < n && a[r] > a[best]) best = r;
        if (best == i) break;
        swap(a[i], a[best]);
        i = best;
    }
}

void heapSort(vector<int>& a) {
    int n = (int)a.size();
    for (int i = n / 2 - 1; i >= 0; --i) siftDown(a, n, i);  // 建大根堆 O(n)
    for (int i = n - 1; i > 0; --i) {
        swap(a[0], a[i]);       // 当前最大值放到末尾
        siftDown(a, i, 0);      // 剩余部分重新调整
    }
}
```

过程示意（升序）：

```
建堆:  [9, 7, 5, 1, 3]
i=4:   [3, 7, 5, 1, |9]  → 调整 → [7, 3, 5, 1, |9]
i=3:   [1, 3, 5, |7, 9]  → 调整 → [5, 3, 1, |7, 9]
i=2:   [1, 3, |5, 7, 9]  → 调整 → [3, 1, |5, 7, 9]
i=1:   [1, |3, 5, 7, 9]
结果:  [1, 3, 5, 7, 9]
```

> 实际比赛中，`std::sort` 通常比手写堆排序更快；堆排序真正的价值是「边取最值边处理」的场景。

---

## 8. 经典应用

### 8.1 贪心 + 优先队列：合并果子

**题意**：每次合并两堆果子，代价为两堆之和，求最小总代价。

**思路**：每次取最小的两堆合并，把新堆放回。用小根堆维护，复杂度 $O(n \log n)$。

```cpp
int main() {
    int n;
    cin >> n;
    priority_queue<long long, vector<long long>, greater<long long>> pq;
    for (int i = 0; i < n; ++i) {
        long long x;
        cin >> x;
        pq.push(x);
    }
    long long ans = 0;
    while (pq.size() > 1) {
        long long a = pq.top(); pq.pop();
        long long b = pq.top(); pq.pop();
        ans += a + b;
        pq.push(a + b);
    }
    cout << ans << '\n';
    return 0;
}
```

### 8.2 Dijkstra 最短路

堆优化的 Dijkstra 本质就是「每次取出当前距离最小的未确定节点」：

```cpp
vector<long long> dijkstra(int s, int n, const vector<vector<pair<int,int>>>& g) {
    const long long INF = (long long)4e18;
    vector<long long> dist(n + 1, INF);
    priority_queue<pair<long long,int>,
                   vector<pair<long long,int>>,
                   greater<pair<long long,int>>> pq;
    dist[s] = 0;
    pq.push({0, s});
    while (!pq.empty()) {
        pair<long long,int> cur = pq.top(); pq.pop();
        long long d = cur.first;
        int u = cur.second;
        if (d > dist[u]) continue;          // 懒删除：过期状态直接跳过
        for (size_t i = 0; i < g[u].size(); ++i) {
            int v = g[u][i].first, w = g[u][i].second;
            if (dist[v] > d + w) {
                dist[v] = d + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

### 8.3 对顶堆：动态中位数

维护两个堆：

- `low`：**大根堆**，存较小的一半；
- `high`：**小根堆**，存较大的一半。

保持 `low.size()` 等于 `high.size()` 或比它多 1，则中位数就是 `low.top()`。

```cpp
priority_queue<int> low;                               // 大根堆
priority_queue<int, vector<int>, greater<int>> high;   // 小根堆

void add(int x) {
    if (low.empty() || x <= low.top()) low.push(x);
    else high.push(x);

    if (low.size() > high.size() + 1) {
        high.push(low.top());
        low.pop();
    }
    if (high.size() > low.size()) {
        low.push(high.top());
        high.pop();
    }
}

int median() { return low.top(); }   // 保证 low 非空
```

典型题目：数据流中位数、滑动窗口中位数（配合惰性删除）。

### 8.4 第 k 大 / 第 k 小

- **求第 k 大**：维护一个大小为 `k` 的**小根堆**，堆顶就是当前第 k 大；
- **求第 k 小**：维护一个大小为 `k` 的**大根堆**，堆顶就是当前第 k 小。

```cpp
// 数据流中第 k 大
priority_queue<int, vector<int>, greater<int>> pq;
void add(int x) {
    pq.push(x);
    if ((int)pq.size() > k) pq.pop();
}
int kthLargest() { return pq.top(); }
```

### 8.5 多路归并

把 $k$ 个有序序列合并成一个有序序列：用一个小根堆存「每个序列当前的头元素」，每次取出最小者输出，再把它所在序列的下一个元素入堆。复杂度 $O(N \log k)$，其中 $N$ 是总元素数。

### 8.6 模拟类问题

很多「每次处理当前最优/最紧急」的模拟题（任务调度、事件驱动模拟、带优先级的 BFS 等）都是堆的直接应用。

---

## 9. 复杂度汇总

| 操作 | 二叉堆 | 说明 |
|---|---|---|
| 取最值 | $O(1)$ | 堆顶 |
| 插入 | $O(\log n)$ | 上浮 |
| 删除最值 | $O(\log n)$ | 下沉 |
| 建堆 | $O(n)$ | 自底向上 |
| 修改 / 删除任意元素 | $O(\log n)$ | 需记录位置 |
| 合并两个堆 | $O(n + m)$ | 普通二叉堆较慢，可用可并堆优化到 $O(\log n)$ |
| 堆排序 | $O(n \log n)$ | 原地、不稳定 |
| 空间 | $O(n)$ | 数组存储 |

---

## 10. 常见易错点

1. **下标混用**：手写堆统一用 1 起始（`h[0]` 占位）或统一用 0 起始，千万不要一半一半。
2. **空堆取堆顶**：`top()` / `pop()` 前必须判 `empty()`，否则是未定义行为。
3. **大根堆 / 小根堆方向写反**：记住「小根堆：父亲 ≤ 儿子；`swim` 用 `<`，`sink` 选最小的儿子」。
4. **比较器方向搞反**：`priority_queue` 的 `cmp(a,b) == true` 表示 `a` 优先级更低；想要小根堆就传 `greater`。可以记成「**priority_queue 与 sort 相反**」。
5. **自定义比较器不满足严格弱序**：不要写 `a.x <= b.x`，要用 `<` 或 `>`，否则行为未定义。
6. **`priority_queue` 不能清空**：没有 `clear()`，用 `pq = PQType();` 或逐个 `pop`。
7. **数据溢出**：路径长度、合并代价可能超过 `int`，用时 `long long`（约 $9 \times 10^{18}$）。
8. **建堆从 `n/2` 开始**：`n/2 + 1` 到 `n` 都是叶子，不需要下沉。
9. **堆不稳定**：相同值的元素相对顺序不保证，需要稳定时给元素加编号。
10. **`priority_queue` 存结构体时忘记写比较器**：`pair` 可以默认比较，自定义结构体必须提供 `operator<` 或比较器。

---

## 11. 延伸与练习建议

### 11.1 堆的变种

| 名称 | 特点 | 常见用途 |
|---|---|---|
| 二叉堆 | 本文主角，实现最简单 | 通用 |
| $d$ 叉堆 | 每个节点 $d$ 个儿子，降低树高 | 减少下沉次数（如 4 叉堆） |
| 左偏树 / 斜堆 | **可并堆**，支持 $O(\log n)$ 合并 | 可并堆、贪心合并 |
| 配对堆 | 均摊复杂度优秀 | 图论算法 |
| 斐波那契堆 | 理论最优的均摊复杂度 | 理论分析，实际少用 |

### 11.2 练习题路线（由易到难）

1. **模板题**：手写小根堆，实现插入、删除、取最小值。
2. **合并果子**：贪心 + 小根堆入门。
3. **第 k 小 / 第 k 大**：大小为 k 的堆。
4. **数据流中位数**：对顶堆。
5. **Dijkstra 堆优化**：堆 + 图论。
6. **任务调度 / 区间问题**：堆维护当前最优。
7. **可并堆（左偏树）**：进阶。

### 11.3 学习检查清单

- [ ] 能徒手画出小根堆的插入、删除过程
- [ ] 能默写 `swim` / `sink` / `push` / `pop` / `build`
- [ ] 能解释为什么建堆是 $O(n)$
- [ ] 能熟练使用 `priority_queue` 的大小根堆写法
- [ ] 知道 `priority_queue` 与手写堆各自适用的场景
- [ ] 能独立写出对顶堆求中位数

---

## 附：一页速查

```cpp
// 小根堆
priority_queue<int, vector<int>, greater<int>> minq;
// 大根堆（默认）
priority_queue<int> maxq;

minq.push(x);          // 插入 O(log n)
minq.top();            // 最小值 O(1)
minq.pop();            // 删除最小值 O(log n)
minq.empty();          // 判空
minq.size();           // 元素个数
```

> **核心思想**：堆用「完全二叉树的数组存储 + 上浮/下沉」在 $O(\log n)$ 内维护最值，是所有「动态取最值」问题的基础工具。
