# 交互题（Interactive Problems）

> 交互题 = 你的程序与评测程序「对话」：你输出询问，评测机返回答案。它考察的是**信息获取策略**——你能用多少次询问、怎样询问才能确定答案。

---

## 目录

1. [交互题的输入输出模式](#1-交互题的输入输出模式)
2. [基本协议](#2-基本协议)
3. [强制离线（在线）与本地调试](#3-强制离线在线与本地调试)
4. [常见题型与策略](#4-常见题型与策略)
5. [模板代码](#5-模板代码)
6. [经典例题](#6-经典例题)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 交互题的输入输出模式

普通题：一次性读入全部数据 → 输出答案。

交互题：

```
你: 询问 "? 3 5"        （输出一行）
评测机: 返回 "2"         （读入一行）
你: 询问 "? 1 4"
评测机: 返回 "1"
...
你: 回答 "! 7"          （输出最终答案）
```

**关键区别**：

- 必须**及时 `fflush(stdout)`**（或 `cout << endl`），否则评测机收不到你的输出；
- 询问次数有上限（题目会说明）；
- 询问格式必须严格遵守题目约定。

---

## 2. 基本协议

### 2.1 常见格式

| 操作 | 格式 | 说明 |
|---|---|---|
| 询问 | `? l r` | 询问区间 |
| 询问 | `? x` | 询问单个位置 |
| 回答 | `! ans` | 输出最终答案 |
| 比较 | `? i j` | 返回大小关系 |

**注意**：不同题目的具体格式不同，**必须读题**。

### 2.2 输出刷新

```cpp
printf("? %d %d\n", l, r);
fflush(stdout);

int res;
scanf("%d", &res);
```

用 `cout`：

```cpp
cout << "? " << l << " " << r << endl;   // endl 会自动 flush
cin >> res;
```

> **绝对不要**用 `'\n'` 而不 flush，否则会 TLE（评测机在等你的输出）。

### 2.3 本地调试

交互题**不能直接运行**。需要写一个「交互器」模拟评测机：

```cpp
// interactor.cpp：读取你的输出，返回答案
int main() {
    // ...
}
```

或者把题目给的 `grader` 文件编译进去。

---

## 3. 强制离线（在线）与本地调试

### 3.1 写交互器

```cpp
// grader.cpp
#include <bits/stdc++.h>
using namespace std;

int n = 100, target = 42;
int queries = 0;

int main() {
    // 读入选手的询问
    char op;
    while (cin >> op) {
        if (op == '?') {
            int l, r;
            cin >> l >> r;
            ++queries;
            if (queries > 20) { cout << "-1" << endl; return 0; }
            int cnt = 0;
            for (int i = l; i <= r; ++i) cnt += (i == target);
            cout << cnt << endl;
        } else if (op == '!') {
            int ans;
            cin >> ans;
            if (ans == target) { cout << "OK" << endl; return 0; }
            else { cout << "WA" << endl; return 0; }
        }
    }
    return 0;
}
```

### 3.2 本地测试

```bash
# Windows
g++ -O2 -std=c++14 sol.cpp -o sol.exe
g++ -O2 -std=c++14 grader.cpp -o grader.exe

# 用管道同时运行
# 或者用文件模拟
```

最简单的办法：把交互器的逻辑**内联到你的程序里**，写一个 `query(l, r)` 函数直接返回答案。这样可以用普通方式调试。

```cpp
#ifdef LOCAL
int query(int l, int r) {
    // 本地模拟
    int cnt = 0;
    for (int i = l; i <= r; ++i) cnt += (i == target);
    return cnt;
}
#else
int query(int l, int r) {
    printf("? %d %d\n", l, r);
    fflush(stdout);
    int x;
    scanf("%d", &x);
    return x;
}
#endif
```

---

## 4. 常见题型与策略

### 4.1 猜数 / 二分

**题型**：有一个隐藏的数，每次可以询问「是否 $\ge x$」，求这个数。

**策略**：标准二分，$O(\log n)$ 次。

```cpp
int l = 1, r = n;
while (l < r) {
    int mid = (l + r) >> 1;
    if (query(mid)) r = mid;
    else l = mid + 1;
}
answer(l);
```

### 4.2 猜排列 / 交互排序

**题型**：隐藏一个排列，可以询问两个元素的大小关系。

**策略**：基于比较的排序，$O(n \log n)$ 次比较。

### 4.3 图上的询问

**题型**：隐藏一棵树，可以询问「两点距离」或「路径上的某信息」。

**策略**：利用距离信息确定结构，或用随机化。

### 4.4 区间询问

**题型**：可以询问任意区间的某种统计量，求某个值。

**策略**：前缀和思想、倍增、分治。

### 4.5 构造交互

**题型**：交互方给出数据，你需要构造满足条件的答案。

### 4.6 自适应策略

**关键**：后面的询问可以根据前面的回答动态调整。这是交互题的核心——**信息论**。

---

## 5. 模板代码

### 5.1 通用交互框架

```cpp
#include <bits/stdc++.h>
using namespace std;

int query(int l, int r) {
    printf("? %d %d\n", l, r);
    fflush(stdout);
    int x;
    scanf("%d", &x);
    if (x == -1) exit(0);   // 询问超限或非法
    return x;
}

void answer(int x) {
    printf("! %d\n", x);
    fflush(stdout);
}

int main() {
    int n;
    scanf("%d", &n);
    // 这里根据题目协议得到答案 ans
    int ans = 0;            // 占位，实际由询问推出
    answer(ans);
    return 0;
}
```

### 5.2 二分猜数（完整程序）

```cpp
#include <bits/stdc++.h>
using namespace std;

int query(int x) {
    printf("? %d\n", x);
    fflush(stdout);
    int r;
    scanf("%d", &r);
    return r;
}

void answer(int x) {
    printf("! %d\n", x);
    fflush(stdout);
}

int main() {
    int n;
    scanf("%d", &n);
    int l = 1, r = n;
    while (l < r) {
        int mid = (l + r) >> 1;
        if (query(mid)) r = mid;   // 询问 "答案是否 >= mid"
        else l = mid + 1;
    }
    answer(l);
    return 0;
}
```

### 5.3 询问次数限制

```cpp
int qcnt = 0;
const int LIM = 20;

int query(int l, int r) {
    ++qcnt;
    if (qcnt > LIM) { /* 超限处理 */ }
    printf("? %d %d\n", l, r);
    fflush(stdout);
    int x; scanf("%d", &x);
    return x;
}
```

---

## 6. 经典例题

### 6.1 猜数（二分）

**题意**：隐藏一个 $[1, n]$ 的数，每次问「$\ge x$ 吗」，最多 $\lceil \log_2 n \rceil$ 次。

**做法**：二分。

### 6.2 猜排列

**题意**：隐藏排列，每次可以问两个位置的大小关系。

**做法**：归并/快速排序。

### 6.3 交互式最短路

**题意**：隐藏图，可以问两点距离。

**做法**：利用三角不等式、BFS 思想。

### 6.4 随机化交互

**题意**：隐藏数据结构，允许随机询问。

**做法**：随机采样、概率分析。

### 6.5 构造 + 交互

**题意**：交互方给出部分信息，要求构造完整答案。

**做法**：分析信息量，设计询问顺序。

---

## 7. 易错点

1. **忘记 flush**：最常见的挂法；
2. **询问格式错误**：多/少空格都可能被判错；
3. **询问次数超限**：提前计数；
4. **读入返回值后没判 `-1`**：有些题用 `-1` 表示非法；
5. **交互器与程序不同步**：本地调试时容易死锁；
6. **假设数据固定**：有些交互题是**自适应**的，答案会随询问变化；
7. **用 `endl` 过多导致超时**：大量交互时注意性能。

---

## 8. 练习建议

1. **先读一道交互题题面**，理解协议；
2. **写一个本地交互器**，把流程跑通；
3. **二分猜数**：熟悉 flush；
4. **猜排列**：练习比较排序；
5. **随机化交互**：学习概率分析；
6. **查看 Codeforces 的交互题 tag**：大量练习。

### 推荐的交互题

| 题目 | 考察点 |
|---|---|
| CF 679A Bear and Prime 100 | 质数询问 |
| CF 1407D Discrete Centrifugal Jumps | 交互 + 构造 |
| CF 1520F1/F2 Guess the K-th Zero | 二分询问 |
| CF 1620C BA-String | 交互构造 |
| CF 1672G Cross Xor | 交互 + 思维 |

---

## 附：交互题检查清单

- [ ] 询问格式完全符合题面
- [ ] 每次询问后 `fflush(stdout)`
- [ ] 询问次数不超过限制
- [ ] 最终答案格式正确
- [ ] 本地用交互器测试过
- [ ] 处理了非法返回值（如 `-1`）
- [ ] 多组数据时重置询问计数
