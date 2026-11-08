# 卡常技巧（Constant Optimization）

> 卡常 = 在算法复杂度已经正确的前提下，通过减少常数让程序跑得更快。**复杂度决定能不能过，常数决定过几个点。**

---

## 目录

1. [先判断是不是复杂度问题](#1-先判断是不是复杂度问题)
2. [输入输出优化](#2-输入输出优化)
3. [循环优化](#3-循环优化)
4. [数据结构层面的卡常](#4-数据结构层面的卡常)
5. [内存与缓存优化](#5-内存与缓存优化)
6. [编译参数](#6-编译参数)
7. [常见「假卡常」陷阱](#7-常见假卡常陷阱)
8. [卡常检查清单](#8-卡常检查清单)

---

## 1. 先判断是不是复杂度问题

卡常之前先问：**我的复杂度是不是已经最优？**

| 现象 | 判断 |
|---|---|
| $n=10^5$，$O(n^2)$ | 换算法，不是卡常 |
| 复杂度正确但常数大 3~10 倍 | 卡常有效 |
| 只有个别点 TLE | 大概率是常数或边界 |
| 大数据全 TLE | 先检查复杂度与死循环 |

> 卡常的上限通常是 **2~5 倍**。如果差了 100 倍，别浪费时间。

---

## 2. 输入输出优化

### 2.1 关闭同步 + 解绑

```cpp
ios::sync_with_stdio(false);
cin.tie(nullptr);
```

比 `scanf` 还快的常见写法，但**不能**再混用 `printf`/`scanf`。

### 2.2 手写快读

```cpp
template <class T>
inline void read(T& x) {
    x = 0;
    int f = 1;
    char c = getchar();
    while (c < '0' || c > '9') {
        if (c == '-') f = -1;
        c = getchar();
    }
    while (c >= '0' && c <= '9') {
        x = x * 10 + (c - '0');
        c = getchar();
    }
    x *= f;
}
```

更快的 `getchar_unlocked`（Linux 专用）：

```cpp
#define gc() getchar_unlocked()
```

### 2.3 手写快写

```cpp
inline void write(int x) {
    if (x < 0) { putchar('-'); x = -x; }
    if (x > 9) write(x / 10);
    putchar(x % 10 + '0');
}
```

### 2.4 缓冲区输出

大量输出时，用 `fwrite` 维护自己的缓冲区：

```cpp
char buf[1 << 20];
int len = 0;

inline void flush() { fwrite(buf, 1, len, stdout); len = 0; }
inline void putc_fast(char c) {
    if (len == (1 << 20)) flush();
    buf[len++] = c;
}
```

---

## 3. 循环优化

### 3.1 减少函数调用

`inline` 小函数、把 `min`/`max` 手写展开：

```cpp
#define Min(a, b) ((a) < (b) ? (a) : (b))
```

（现代编译器对 `std::min` 一般也能内联，别过度迷信宏。）

### 3.2 循环展开

```cpp
for (int i = 0; i + 3 < n; i += 4) {
    s += a[i]; s += a[i+1]; s += a[i+2]; s += a[i+3];
}
for (; i < n; ++i) s += a[i];
```

### 3.3 提前终止

```cpp
if (ans == 0) break;   // 已最优
if (cnt > k) return;   // 剪枝
```

### 3.4 用位运算代替乘除

```cpp
x * 2   →   x << 1
x / 2   →   x >> 1
x % 2   →   x & 1
```

**仅对非负整数等价**，负数右移是实现定义行为。

### 3.5 循环顺序

多维数组按行优先访问（C++ 行主序），`a[i][j]` 循环时 `j` 在内层，缓存友好。

---

## 4. 数据结构层面的卡常

### 4.1 线段树

- 用**迭代式线段树**（zkw 线段树）代替递归版；
- 区间加法 + 区间求和用**懒标记**而非暴力下传；
- 查询时用 `while` 循环而不是递归；
- 结构体数组比 `vector` 快。

### 4.2 并查集

路径压缩 + 按秩合并，**必须两个都写**：

```cpp
int find(int x) {
    return fa[x] == x ? x : fa[x] = find(fa[x]);
}
```

### 4.3 树状数组

能用树状数组就不要用线段树。`lowbit` 用 `i & -i`。

### 4.4 堆

`priority_queue` 已经很快；手写堆能省一点常数但通常没必要。

### 4.5 哈希表

`unordered_map` 容易被卡，改用**手写哈希表**或 `gp_hash_table`：

```cpp
struct Hash {
    static uint64_t splitmix64(uint64_t x) {
        x += 0x9e3779b97f4a7c15ULL;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
        x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
        return x ^ (x >> 31);
    }
    size_t operator()(uint64_t x) const {
        static const uint64_t FIXED_RANDOM =
            chrono::steady_clock::now().time_since_epoch().count();
        return splitmix64(x + FIXED_RANDOM);
    }
};
unordered_map<long long, int, Hash> mp;
```

---

## 5. 内存与缓存优化

### 5.1 数组连续存储

用 `vector<int> g[N]` 或**链式前向星**存图，比 `vector<vector<int>>` 更省内存、更快。

```cpp
int head[N], to[M], nxt[M], ecnt = 0;

void addEdge(int u, int v) {
    to[++ecnt] = v;
    nxt[ecnt] = head[u];
    head[u] = ecnt;
}
```

### 5.2 结构体对齐

把 `int` 放一起、`long long` 放一起，减少 padding 浪费。

### 5.3 避免动态分配

`new`/`delete` 很慢，比赛里尽量用静态数组或 `vector` 预留 `reserve`。

---

## 6. 编译参数

OI 比赛通常固定 `-O2 -std=c++14 -static`。本地可尝试：

```bash
g++ -O2 -std=c++14 -march=native -funroll-loops
```

但**提交时不能改编译参数**（规则禁止 `#pragma` 改编译选项）。

可以在代码里加：

```cpp
#pragma GCC optimize("O2")
#pragma GCC optimize("unroll-loops")
```

> ⚠️ 注意：多数正式比赛**禁止**使用 `#pragma` 修改编译参数，提交前务必确认规则。

---

## 7. 常见「假卡常」陷阱

1. **迷信 `#pragma`**：规则禁止，且现代评测机不吃这套；
2. **把 `int` 全改成 `long long`**：不会更快，反而更慢；
3. **过度循环展开**：代码膨胀导致 cache miss；
4. **用 `register`**：C++17 起已废弃，编译器会忽略；
5. **频繁 `push_back` 不 `reserve`**：容器扩容是主要开销之一；
6. **`endl` 刷新缓冲区**：大量输出时用 `'\n'`。

---

## 8. 卡常检查清单

- [ ] 复杂度是否已最优？
- [ ] 是否关闭了 `cin/cout` 同步？
- [ ] 是否用了 `'\n'` 而不是 `endl`？
- [ ] 大数组是否开在全局（避免栈溢出）？
- [ ] 是否避免了无谓的 `long long` 运算？
- [ ] 数据结构是否用了最轻量的实现？
- [ ] 是否有可以提前 `break`/`return` 的分支？
- [ ] 是否有不必要的函数调用在热循环里？
- [ ] 图的存储是否用了链式前向星？
- [ ] 是否验证过「去掉卡常也能过小数据」？

---

## 附：常用快读快写完整模板

```cpp
#include <bits/stdc++.h>
using namespace std;

static char buf[1 << 23], *p1 = buf, *p2 = buf;
#define gc() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 23, stdin), p1 == p2) ? EOF : *p1++)

template <class T>
inline void read(T& x) {
    x = 0;
    int f = 1;
    char c = gc();
    while (c < '0' || c > '9') { if (c == '-') f = -1; c = gc(); }
    while (c >= '0' && c <= '9') { x = x * 10 + (c - '0'); c = gc(); }
    x *= f;
}

template <class T>
inline void write(T x) {
    if (x < 0) { putchar('-'); x = -x; }
    static char stk[40];
    int top = 0;
    do { stk[top++] = char(x % 10 + '0'); x /= 10; } while (x);
    while (top) putchar(stk[--top]);
}

int main() {
    int n;
    read(n);
    write(n);
    putchar('\n');
    return 0;
}
```

---

## 附：常见错误与练习清单

### 常见错误

1. **忽略边界情形**：空输入、最小规模、极大值、全相同元素。
2. **想当然地套模板**：没有先确认问题的前提条件是否满足。
3. **调试输出忘记删除**：提交前检索 `cerr`、`printf("debug` 等。
4. **变量作用域混乱**：局部变量遮蔽全局变量，多组数据不清空。
5. **时间估算错误**：只看复杂度，忽略常数与评测机性能。

### 练习建议

- 先用最小规模数据手算一遍，确认思路无误；
- 写完后用对拍脚本自测（见 [对拍](../对拍/README.md)）；
- 整理一份自己的错题本，定期回看；
- 把本文的方法固化成checklist，每次比赛前过一遍。