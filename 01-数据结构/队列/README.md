# 队列（Queue）

> 一句话概括：**队列是一种「先进先出（FIFO）」的线性结构**，只能在队尾插入、在队头删除，所有基本操作都是 $O(1)$。

---

## 目录

1. [概念与原理](#1-概念与原理)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点](#4-易错点)
5. [练习建议](#5-练习建议)

---

## 1. 概念与原理

### 1.1 什么是队列

队列（Queue）是一种**受限的线性表**：元素只能从**队尾（rear / tail）**进入，只能从**队头（front / head）**离开。

- `push(x)`：把 `x` 放到队尾；
- `pop()`：删除队头元素；
- `front()` / `back()`：读取队头 / 队尾元素；
- `empty()` / `size()`：判空与查询长度。

先进入的元素排在最前面，所以**最先进入的元素最先离开**，即「先进先出」（First In First Out，FIFO）。

### 1.2 文字示意图

队列就像食堂打饭的队伍：后来的人只能排在队尾，先来的人先打到饭离开。

```
队头（出队）                                      队尾（入队）
   ↓                                                ↓
+-----+-----+-----+-----+-----+
| 10  | 20  | 30  | 40  | 50  |
+-----+-----+-----+-----+-----+
```

操作过程：

```
操作            队列内容（队头 → 队尾）      返回值
push(10)        [10]
push(20)        [10, 20]
push(30)        [10, 20, 30]
front()         [10, 20, 30]                 10
pop()           [20, 30]                     10
push(40)        [20, 30, 40]
front()         [20, 30, 40]                 20
back()          [20, 30, 40]                 40
```

### 1.3 为什么需要队列

队列的核心语义是「**按到达顺序公平地处理**」，因此非常适合：

| 场景 | 为什么适合队列 |
|---|---|
| 广度优先搜索（BFS） | 按距离从近到远逐层扩展，先发现的先扩展 |
| 任务调度 / 打印队列 | 先提交的任务先执行，保证公平 |
| 滑动窗口 | 窗口右移时左边元素自然「过期出队」 |
| 缓冲区 | 生产者-消费者模型，先进先出 |
| 拓扑排序（Kahn 算法） | 入度为 0 的节点按顺序入队处理 |
| SPFA 最短路 | 用队列存「待松弛」的节点 |

### 1.4 顺序队列的「假溢出」

如果简单地用数组 `q[0..n-1]` 加两个指针 `head`、`tail`，每次 `pop` 只把 `head` 后移、不回收空间，那么经过若干次操作后 `tail` 会到达数组末尾，而前面其实还有大量空位：

```
初始:   [ 1 | 2 | 3 |   |   |   ]
              head=0, tail=3

pop 三次后:
        [   |   |   |   |   |   ]
                          head=3, tail=6  ← 越界，但前面全空
```

这就是**假溢出**。解决办法有两种：**循环队列**（把数组看成环，`tail` 到末尾后回到 0）或直接用 **STL `std::queue`**（底层 `deque` 自动管理内存）。

### 1.5 循环队列

将长度为 $n$ 的数组视为环：下标 $i$ 的下一格是 $(i+1) \bmod n$。

```
容量 n = 6 的循环队列（从队头到队尾依次是 20 30 40 50）：

        0     1     2     3     4     5
      +-----+-----+-----+-----+-----+-----+
      | 40  | 50  |     |     | 20  | 30  |
      +-----+-----+-----+-----+-----+-----+
                      ↑           ↑
                    tail=2      head=4
```

队列内容按「从 head 出发绕一圈」读取：`q[4]=20, q[5]=30, q[0]=40, q[1]=50`，到 `tail=2` 结束；`q[2]、q[3]` 是空位。

为了区分「队空」和「队满」，常用两种约定：

- **浪费一个格子**：`head == tail` 表示空，`(tail + 1) % n == head` 表示满，容量实际为 $n-1$；
- **额外记录 size**：用变量 `cnt` 记录元素个数，`cnt == 0` 为空，`cnt == n` 为满。

模板中采用**记录 size** 的写法，更直观且不浪费空间。若用「浪费一格」的写法，元素个数也可以由两个指针直接算出：

$$
\text{cnt} = (\text{tail} - \text{head} + n) \bmod n
$$

### 1.6 双端队列（deque）

双端队列允许在**两端**插入和删除：`push_front(x)` / `push_back(x)`、`pop_front()` / `pop_back()`、`front()` / `back()`。C++ 的 `std::deque` 就是双端队列，它也是 `std::stack` 与 `std::queue` 的默认底层容器。**单调队列**（见后续章节）必须用双端队列实现。

---

## 2. 模板代码

### 2.1 循环队列（数组实现）

```cpp
#include <bits/stdc++.h>
using namespace std;

class CircularQueue {
    vector<int> data;   // 环形数组
    int head, tail;     // 队头下标；队尾的下一格
    int cnt, cap;       // 当前元素个数；容量
public:
    explicit CircularQueue(int capacity)
        : data(capacity), head(0), tail(0), cnt(0), cap(capacity) {}

    bool empty() const { return cnt == 0; }
    bool full() const { return cnt == cap; }
    int size() const { return cnt; }

    bool push(int x) {                    // 入队
        if (full()) return false;
        data[tail] = x;
        tail = (tail + 1) % cap;          // 取模实现「绕回」
        ++cnt;
        return true;
    }
    bool pop() {                          // 出队
        if (empty()) return false;
        head = (head + 1) % cap;
        --cnt;
        return true;
    }
    int front() const { return data[head]; }
    int back() const { return data[(tail - 1 + cap) % cap]; }   // 防负数
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int q;
    cin >> q;
    CircularQueue que(q + 1);
    while (q--) {
        string op;
        cin >> op;
        if (op == "push") {
            int x;
            cin >> x;
            if (!que.push(x)) cout << "full" << '\n';
        } else if (op == "pop") {
            if (!que.pop()) cout << "empty" << '\n';
        } else if (op == "front") {
            if (que.empty()) cout << "empty" << '\n';
            else cout << que.front() << '\n';
        } else if (op == "back") {
            if (que.empty()) cout << "empty" << '\n';
            else cout << que.back() << '\n';
        } else {
            cout << que.size() << '\n';
        }
    }
    return 0;
}
```

输入 `6 / push 1 / push 2 / push 3 / front / pop / back`，输出 `1 / 3`。

### 2.2 STL `std::queue` 与 `std::deque` 速查

```cpp
#include <queue>
queue<int> q;
q.push(3);            // 入队（队尾）
q.front();  q.back(); // 队头 / 队尾元素
q.pop();              // 弹出队头（不返回值）
q.empty();  q.size();

#include <deque>
deque<int> dq;
dq.push_back(1);  dq.push_front(2);   // 尾部 / 头部插入
dq.pop_back();    dq.pop_front();     // 尾部 / 头部删除
dq.front();       dq.back();
dq[0];                                // 支持随机访问（常数比 vector 略大）
```

---

## 3. 经典例题

### 3.1 二叉树的层序遍历（BFS）

**题意**：给定二叉树，按层从上到下、每层从左到右输出节点值。

**思路**：BFS 标准模板。队列中存放「当前层待访问的节点」；每轮先记录队列大小 `sz`，表示这一层节点数，然后连续弹出 `sz` 个节点并把儿子入队，从而按层分组输出。

**复杂度**：每个节点进出队各一次，时间 $O(n)$，空间 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int val;
    Node *left, *right;
    explicit Node(int v) : val(v), left(nullptr), right(nullptr) {}
};

vector<vector<int>> levelOrder(Node* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<Node*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();                 // 当前层的节点数
        vector<int> level;
        for (int i = 0; i < sz; ++i) {
            Node* u = q.front();
            q.pop();
            level.push_back(u->val);
            if (u->left) q.push(u->left);
            if (u->right) q.push(u->right);
        }
        res.push_back(level);
    }
    return res;
}

int main() {
    // 构造的树：
    //          1
    //        /   \
    //       2     3
    //      /     / \
    //     4     5   6
    Node* root = new Node(1);
    root->left = new Node(2);
    root->right = new Node(3);
    root->left->left = new Node(4);
    root->right->left = new Node(5);
    root->right->right = new Node(6);

    for (auto& level : levelOrder(root)) {
        for (int x : level) cout << x << ' ';
        cout << '\n';
    }
    return 0;
}
```

### 3.2 用队列实现栈

**题意**：只使用两个队列实现一个后进先出（LIFO）的栈，支持 `push`、`pop`、`top`、`empty`。

**思路**：主队列 `q1`、辅助队列 `q2`。`push` 直接压 `q1`；`pop` 时把 `q1` 中除最后一个外的元素都移到 `q2`，剩下的唯一元素就是栈顶，弹出后再把 `q2` 倒回 `q1`。

**复杂度**：`push` 为 $O(1)$，`pop` / `top` 为 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

class MyStack {
    queue<int> q1, q2;
public:
    void push(int x) { q1.push(x); }
    int pop() {
        while (q1.size() > 1) {            // 只留最后一个（即栈顶）
            q2.push(q1.front());
            q1.pop();
        }
        int topVal = q1.front();
        q1.pop();
        swap(q1, q2);                       // 其余元素倒回 q1
        return topVal;
    }
    int top() { int v = pop(); q1.push(v); return v; }
    bool empty() const { return q1.empty(); }
};

int main() {
    MyStack st;
    st.push(1); st.push(2); st.push(3);
    cout << st.top() << '\n';   // 3
    cout << st.pop() << '\n';   // 3
    cout << st.top() << '\n';   // 2
    return 0;
}
```

### 3.3 约瑟夫问题（队列模拟）

**题意**：$n$ 个人围成一圈，从第 1 个人开始报数，报到 $m$ 的人出圈，然后从下一个人重新报数，求最后剩下的人的编号。

**思路**：队列模拟。每轮取出队头：报数没到 $m$ 就放回队尾；到了 $m$ 就删除并重置计数器。

**复杂度**：$O(nm)$（模拟），空间 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;
    queue<int> q;
    for (int i = 1; i <= n; ++i) q.push(i);
    int cnt = 0;
    while (q.size() > 1) {
        int x = q.front();
        q.pop();
        if (++cnt == m) cnt = 0;   // 这个人出圈，不回到队尾
        else q.push(x);            // 报数未到，回到队尾
    }
    cout << q.front() << '\n';
    return 0;
}
```

> 本题有 $O(n)$ 的递推解法 $f_i = (f_{i-1} + m) \bmod i$，队列模拟适合理解过程。

### 3.4 最短路径（BFS 迷宫）

**题意**：$n \times m$ 的网格中有障碍物，求从起点到终点的最少步数（四方向移动）。

**思路**：边权全为 1 的最短路就是 BFS。用队列逐层扩展，`dist` 数组记录步数，第一次到达终点即为最短。

**复杂度**：$O(nm)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;
    vector<string> g(n);
    for (auto& s : g) cin >> s;
    pair<int,int> st{-1,-1}, ed{-1,-1};
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < m; ++j) {
            if (g[i][j] == 'S') st = {i, j};
            if (g[i][j] == 'E') ed = {i, j};
        }

    vector<vector<int>> dist(n, vector<int>(m, -1));
    queue<pair<int,int>> q;
    dist[st.first][st.second] = 0;
    q.push(st);
    const int dx[4] = {-1, 1, 0, 0}, dy[4] = {0, 0, -1, 1};
    while (!q.empty()) {
        int x = q.front().first;
        int y = q.front().second;
        q.pop();
        for (int d = 0; d < 4; ++d) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            if (g[nx][ny] == '#' || dist[nx][ny] != -1) continue;
            dist[nx][ny] = dist[x][y] + 1;
            q.push({nx, ny});
        }
    }
    cout << dist[ed.first][ed.second] << '\n';
    return 0;
}
```

输入：

```
5 5
S...#
.#.#.
.#...
...#.
...#E
```

输出：

```
8
```

---

## 4. 易错点

1. **空队访问**：`front()`、`back()`、`pop()` 前必须判 `empty()`，否则是未定义行为。
2. **`std::queue::pop()` 不返回值**：先 `front()` 取值，再 `pop()` 删除。
3. **循环队列的下标取模**：`tail = (tail + 1) % cap`，忘记取模会越界；`back()` 要用 `(tail - 1 + cap) % cap` 防止负数。
4. **队空与队满判定混淆**：若采用「浪费一格」方案，空是 `head == tail`，满是 `(tail + 1) % n == head`，两者不能都写成相等。
5. **多组数据忘记清空**：`queue` 没有 `clear()`，用 `q = queue<int>();` 或重新声明。
6. **BFS 中忘记标记访问**：应在**入队时**标记 `vis`，而不是出队时，否则同一节点可能被重复入队导致超时。
7. **BFS 层级处理**：需要按层统计时必须在每轮开始时保存 `q.size()`，不能直接 `while (!q.empty())` 一次性处理完。
8. **`deque` 随机访问较慢**：`deque` 支持 `operator[]`，但常数比 `vector` 大，不要用它替代 `vector` 做大量随机访问。

---

## 5. 练习建议

### 5.1 由易到难的路线

1. **基础模板**：手写循环队列，实现 `push / pop / front / back / size`。
2. **STL 熟练**：用 `queue` 完成 BFS 层序遍历、迷宫最短路（步数最少）。
3. **队列模拟**：约瑟夫问题、卡片游戏、银行排队模拟。
4. **双端队列**：用 `deque` 完成回文判断、滑动窗口的朴素版本。
5. **队列 + BFS**：多源 BFS（如「腐烂的橘子」）、0-1 BFS（用双端队列）。
6. **队列 + 图论**：拓扑排序（Kahn）、SPFA 最短路。
7. **进阶**：单调队列（见「单调队列」一章）、优先队列（见「堆」一章）。

### 5.2 推荐题目

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1996 约瑟夫问题 | 队列模拟 |
| 入门 | 洛谷 P1540 机器翻译 | 队列 + 查重 |
| 基础 | LeetCode 102 二叉树的层序遍历 | BFS |
| 基础 | LeetCode 225 用队列实现栈 | 队列设计 |
| 基础 | 洛谷 P1443 马的遍历 | BFS 最短路 |
| 提高 | 洛谷 P1332 血色先锋队 | 多源 BFS |
| 提高 | 洛谷 P1886 滑动窗口 | 单调队列 |

### 5.3 学习检查清单

- [ ] 能说清「先进先出」的含义并画出 push/pop 过程
- [ ] 能默写循环队列并解释「假溢出」
- [ ] 能默写 BFS 模板，并正确处理「按层」与「标记访问」
- [ ] 知道 `std::queue::pop()` 不返回值的坑
- [ ] 能区分 `queue` 与 `deque` 的适用场景
- [ ] 能用队列实现栈、用栈实现队列

> **核心思想**：队列用「一端进、另一端出」保证了处理顺序与到达顺序一致，是 BFS、任务调度、缓冲区等「按序处理」问题的标准工具；循环队列则用取模把数组首尾相接，彻底解决假溢出。
