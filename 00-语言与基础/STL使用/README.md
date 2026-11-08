# STL 使用

> 一句话概括：**STL 是 C++ 标准库中现成的容器与算法集合，熟练使用能把比赛代码量砍掉一半。**

---

## 目录

1. [为什么用 STL](#1-为什么用-stl)
2. [容器总览与选择](#2-容器总览与选择)
3. [vector：动态数组](#3-vector动态数组)
4. [string：字符串](#4-string字符串)
5. [pair 与 tuple](#5-pair-与-tuple)
6. [stack、queue、deque](#6-stackqueuedeque)
7. [priority_queue：优先队列](#7-priority_queue优先队列)
8. [set 与 multiset](#8-set-与-multiset)
9. [map 与 unordered_map](#9-map-与-unordered_map)
10. [algorithm 常用函数](#10-algorithm-常用函数)
11. [模板代码](#11-模板代码)
12. [经典例题](#12-经典例题)
13. [易错点](#13-易错点)
14. [练习建议](#14-练习建议)

---

## 1. 为什么用 STL

手写一棵平衡树要两百行，`std::set` 一行就能用；手写哈希表要处理冲突，`unordered_map` 直接给你。STL 的价值在于**正确性**（被全世界用了三十年）、**效率**（`std::sort` 是优化过的内省排序）与**可读性**（`v.push_back(x)` 比手写扩容清楚）。

代价是容器有常数开销、内存占用略高，但在绝大多数 OI 题目里完全可以接受。推荐学习顺序：**顺序容器（vector / string / deque）→ 容器适配器（stack / queue / priority_queue）→ 关联容器（set / map）→ 算法库（sort / lower_bound / unique）**，先会用，再理解复杂度与失效规则。

---

## 2. 容器总览与选择

| 容器 | 底层结构 | 随机访问 | 插入删除 | 查找 | 典型用途 |
|---|---|---|---|---|---|
| `vector` | 动态数组 | $O(1)$ | 尾部 $O(1)$，中间 $O(n)$ | $O(n)$ | 最常用的序列 |
| `deque` | 分段数组 | $O(1)$ | 两端 $O(1)$ | $O(n)$ | 双端队列 |
| `stack` | 适配器 | 仅栈顶 | 栈顶 $O(1)$ | — | 后进先出 |
| `queue` | 适配器 | 仅两端 | 两端 $O(1)$ | — | 先进先出 |
| `priority_queue` | 二叉堆 | 仅堆顶 | $O(\log n)$ | — | 动态取最值 |
| `set` / `map` | 红黑树 | — | $O(\log n)$ | $O(\log n)$ | 有序、去重 |
| `unordered_map` | 哈希表 | — | 平均 $O(1)$ | 平均 $O(1)$ | 快速查询 |
| `bitset` | 位数组 | $O(1)$ | — | — | 状态压缩 |

**选择口诀**：能用 `vector` 就用 `vector`（连续内存、缓存友好）；需要有序或去重 → `set` / `map`；只需查存在性 → `unordered_map`；需要每次取最值 → `priority_queue`；需要后进先出 / 先进先出 → `stack` / `queue`。

---

## 3. vector：动态数组

### 3.1 基本操作

```cpp
vector<int> v;                 // 空 vector
vector<int> a(10);             // 10 个元素，值初始化为 0
vector<int> b(10, 5);          // 10 个元素，每个都是 5
vector<int> c = {1, 2, 3};     // 列表初始化（C++11）
v.push_back(7);                // 尾部插入，均摊 O(1)
v.pop_back();                  // 删除尾部，O(1)
v.size();                      // 元素个数（size_t）
v.empty();                     // 是否为空
v.clear();                     // 清空，O(n)
v.resize(20);                  // 调整大小
v.front(); v.back();           // 首尾元素
v[0];                          // 下标访问，O(1)，不检查越界
v.at(0);                       // 带越界检查，越界抛异常
```

### 3.2 容量与扩容

当 `size() == capacity()` 时再插入，`vector` 会申请一块**更大的内存**（通常是 2 倍），把旧元素搬过去再释放旧内存。因此 `push_back` 的**均摊**复杂度是 $O(1)$，但某一次可能触发 $O(n)$ 的扩容。已知元素个数时用 `reserve` 避免反复扩容：

```cpp
vector<int> v;
v.reserve(100000);      // 一次性申请够，之后 push_back 不会扩容
for (int i = 0; i < 100000; ++i) v.push_back(i);
```

### 3.3 遍历与二维 vector

```cpp
for (int i = 0; i < (int)v.size(); ++i) cout << v[i] << ' ';   // 下标
for (const auto& x : v) cout << x << ' ';                       // 范围 for
for (vector<int>::iterator it = v.begin(); it != v.end(); ++it) cout << *it;  // 迭代器
int n = 5, m = 6;
vector<vector<int>> g(n, vector<int>(m, 0));   // n 行 m 列，全 0
g[1][2] = 7;
```

> `vector<bool>` 做了位压缩，`v[0]` 返回的是代理对象而非 `bool&`，不能取地址。需要普通布尔数组时用 `vector<char>` 或 `bitset`。

---

## 4. string：字符串

```cpp
string s = "hello";
s += " world";                 // 拼接，均摊 O(1)
s.size();                      // 长度
s.substr(1, 3);                // 从下标 1 取 3 个字符，O(3)
s.find("lo");                  // 查找子串，找不到返回 string::npos
s.find('l', 3);                // 从下标 3 开始找字符
s.replace(0, 5, "hi");         // 把 [0,5) 替换成 "hi"
s.append("!");                 // 追加
s.c_str();                     // 转成 C 风格字符串
stoi(s); stoll(s); stod(s);    // 字符串转数字
to_string(123);                // 数字转字符串
```

读入整行用 `getline(cin, line)`。若前面用 `cin >>` 读过东西，缓冲区里会留下换行符，`getline` 会立刻读到空行，先 `cin.ignore()` 即可。

---

## 5. pair 与 tuple

```cpp
pair<int, string> p = {1, "a"};
p.first;   // 1
p.second;  // "a"
pair<int,int> a = {1, 5}, b = {2, 0};
bool less = (a < b);   // true：先比 first，再比 second
tuple<int, double, string> t = make_tuple(1, 2.5, "x");
get<0>(t);   // 1
```

`pair` 是 STL 中最常用的「小结构体」，`map` 的元素类型就是 `pair<const Key, Value>`。

---

## 6. stack、queue、deque

```cpp
stack<int> st;
st.push(1);      // 入栈
st.top();        // 栈顶
st.pop();        // 弹出栈顶（不返回值！）
queue<int> q;
q.push(1);       // 入队
q.front();       // 队首
q.back();        // 队尾
q.pop();         // 弹出队首（不返回值！）
deque<int> dq;
dq.push_back(1);
dq.push_front(0);
dq.pop_back();
dq.pop_front();
dq[0];           // 支持随机访问
```

`stack` 用于括号匹配、表达式求值、单调栈；`queue` 用于 BFS、滑动窗口；`deque` 是前两者的默认底层容器，两端插入都是 $O(1)$。

---

## 7. priority_queue：优先队列

```cpp
priority_queue<int> pq;                                       // 大根堆（默认）
priority_queue<int, vector<int>, greater<int>> minq;          // 小根堆
pq.push(3);      // 插入，O(log n)
pq.top();        // 堆顶（最大值），O(1)
pq.pop();        // 删除堆顶，O(log n)
pq.empty(); pq.size();
```

自定义类型需要重载 `<`，注意 **`priority_queue` 的 `<` 表示「优先级更低」**：

```cpp
struct Node {
    int val, id;
    bool operator<(const Node& o) const { return val < o.val; }  // val 大的先出
};
priority_queue<Node> pq;
```

**常见陷阱**：没有 `clear()`（清空写 `pq = priority_queue<int>();`）；大根堆用 `less<T>`、小根堆用 `greater<T>`，**和 `sort` 的直觉相反**。

---

## 8. set 与 multiset

```cpp
set<int> s;                    // 有序、去重
multiset<int> ms;              // 有序、可重复
s.insert(3);                   // 插入，O(log n)
s.erase(3);                    // 删除值为 3 的所有元素，O(log n)
s.erase(s.find(3));            // 只删一个
s.count(3);                    // 出现次数（set 只有 0/1）
s.find(3);                     // 返回迭代器，找不到返回 s.end()
s.lower_bound(3);              // 第一个 ≥ 3 的迭代器
s.upper_bound(3);              // 第一个 > 3 的迭代器
*s.begin();                    // 最小值
*s.rbegin();                   // 最大值
```

典型用途：去重 + 排序一步到位；动态维护有序集合支持前驱后继查询；`multiset` 维护可重集合（如动态中位数的暴力版）。

---

## 9. map 与 unordered_map

```cpp
map<string, int> mp;           // 键有序，O(log n)
mp["apple"] = 3;               // 不存在则插入，存在则赋值
mp.count("apple");             // 是否存在（0/1）
mp.find("apple");              // 返回迭代器
mp.erase("apple");
for (auto& kv : mp) cout << kv.first << ' ' << kv.second << '\n';   // 按键升序
unordered_map<int, int> cnt;
cnt[1]++;                      // 平均 O(1)
```

`map` 的键是 `const`，不能修改，只能删了重插。`unordered_map` 平均 $O(1)$ 但无序，且可能被出题人构造数据卡哈希，比赛慎用（或自己写哈希函数）。

> **`operator[]` 有副作用**：`if (mp["not_exist"] == 0)` 会插入一个值为 0 的新元素！查询请用 `mp.find(k) != mp.end()` 或 `mp.count(k)`。

---

## 10. algorithm 常用函数

```cpp
sort(v.begin(), v.end());                     // 升序，O(n log n)
sort(v.begin(), v.end(), greater<int>());     // 降序
sort(v.begin(), v.end(), [](int a, int b) { return a > b; });  // 自定义
reverse(v.begin(), v.end());                  // 翻转，O(n)
v.erase(unique(v.begin(), v.end()), v.end()); // 排序后去重（unique 只搬移不删除）
lower_bound(v.begin(), v.end(), x);           // 第一个 ≥ x 的位置，O(log n)
upper_bound(v.begin(), v.end(), x);           // 第一个 > x 的位置
binary_search(v.begin(), v.end(), x);         // 是否存在，O(log n)
min_element(v.begin(), v.end());              // 最小值迭代器
max_element(v.begin(), v.end());              // 最大值迭代器
count(v.begin(), v.end(), x);                 // 出现次数，O(n)
accumulate(v.begin(), v.end(), 0LL);          // 求和，注意初值类型
fill(v.begin(), v.end(), 0);                  // 全部赋值为 0
```

`lower_bound` 要求区间**已按 `<` 排好序**，返回迭代器，转下标要减 `begin()`；找不到返回 `end()`，**不能直接解引用**：

```cpp
int pos = lower_bound(a.begin(), a.end(), x) - a.begin();
if (pos < (int)a.size() && a[pos] == x) { /* 找到了 */ }
```

这些操作的复杂度都是必须记住的：

$$\text{sort}:O(n\log n),\qquad \text{lower\_bound}:O(\log n),\qquad \text{unique}:O(n)$$

---

## 11. 模板代码

### 11.1 常用 STL 操作合集

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    vector<int> v = {5, 3, 1, 4, 2};
    sort(v.begin(), v.end());                     // 1 2 3 4 5
    v.erase(unique(v.begin(), v.end()), v.end()); // 去重
    for (int x : v) cout << x << ' ';
    cout << '\n';
    set<int> s(v.begin(), v.end());
    s.insert(0);
    cout << *s.begin() << ' ' << *s.rbegin() << '\n';   // 最小 最大
    map<string, int> mp;
    mp["apple"] = 1;
    mp["banana"] = 2;
    for (const auto& kv : mp) cout << kv.first << ':' << kv.second << ' ';
    cout << '\n';
    priority_queue<int, vector<int>, greater<int>> pq;   // 小根堆
    for (int x : v) pq.push(x);
    while (!pq.empty()) { cout << pq.top() << ' '; pq.pop(); }
    cout << '\n';
    cout << *max_element(v.begin(), v.end()) << '\n';
    cout << accumulate(v.begin(), v.end(), 0LL) << '\n';
    int p = lower_bound(v.begin(), v.end(), 3) - v.begin();
    cout << "lower_bound(3) = " << p << '\n';            // 下标 2
    return 0;
}
```

### 11.2 自定义排序

```cpp
struct Student {
    string name;
    int score;
    bool operator<(const Student& o) const {
        if (score != o.score) return score > o.score;   // 分数降序
        return name < o.name;                            // 同分按名字升序
    }
};
vector<Student> v;
sort(v.begin(), v.end());   // 直接按 operator< 排序
```

---

## 12. 经典例题

### 例题 1：统计出现次数最多的数

**题意**：给定 $n$（$n \le 10^5$）个整数，输出出现次数最多的数和它的出现次数；若有多个，输出值最小的。

**思路**：用 `map<int,int>` 统计，遍历时比较次数；因为 `map` 按键升序，第一次遇到最大次数就是值最小的。

**复杂度**：时间 $O(n \log n)$，空间 $O(n)$。

```cpp
int main() {
    int n;
    cin >> n;
    map<int, int> cnt;
    for (int i = 0; i < n; ++i) { int x; cin >> x; ++cnt[x]; }
    int bestVal = 0, bestCnt = -1;
    for (const auto& kv : cnt)
        if (kv.second > bestCnt) { bestCnt = kv.second; bestVal = kv.first; }
    cout << bestVal << ' ' << bestCnt << '\n';
    return 0;
}
```

### 例题 2：合并区间

**题意**：给定 $n$ 个区间 $[l_i, r_i]$，合并所有重叠区间，按左端点升序输出。

**思路**：先按左端点排序，再线性扫描：若当前区间左端点 $\le$ 上一个合并区间的右端点则合并（右端点取 $\max$），否则另起一段。

**复杂度**：排序 $O(n \log n)$，扫描 $O(n)$。

```cpp
int main() {
    int n;
    cin >> n;
    vector<pair<int,int>> seg(n);
    for (auto& p : seg) cin >> p.first >> p.second;
    sort(seg.begin(), seg.end());
    vector<pair<int,int>> ans;
    for (auto& p : seg) {
        if (ans.empty() || p.first > ans.back().second) ans.push_back(p);
        else ans.back().second = max(ans.back().second, p.second);
    }
    for (auto& p : ans) cout << p.first << ' ' << p.second << '\n';
    return 0;
}
```

### 例题 3：滑动窗口最大值

**题意**：给定长度为 $n$ 的数组和窗口大小 $k$，输出每个窗口的最大值。

**思路**：用 `multiset` 维护窗口内元素，每次插入新元素、删除离开窗口的元素，`*rbegin()` 即最大值。

**复杂度**：时间 $O(n \log k)$，空间 $O(k)$。

```cpp
int main() {
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    multiset<int> win;
    for (int i = 0; i < n; ++i) {
        win.insert(a[i]);
        if (i >= k) win.erase(win.find(a[i - k]));   // 只删一个
        if (i >= k - 1) cout << *win.rbegin() << ' ';
    }
    return 0;
}
```

### 例题 4：第 k 小元素

**题意**：动态插入数字并查询当前第 $k$ 小，共 $q$ 次操作。

**思路**：用 `multiset` 维护所有数字，查询时从头迭代 $k-1$ 步。若 $q$ 很大且 $k$ 固定，可改用对顶堆。

**复杂度**：插入 $O(\log n)$，查询 $O(k)$。

```cpp
int main() {
    int q, k;
    cin >> q >> k;
    multiset<int> s;
    while (q--) {
        int op, x;
        cin >> op;
        if (op == 1) { cin >> x; s.insert(x); }
        else { auto it = s.begin(); advance(it, k - 1); cout << *it << '\n'; }
    }
    return 0;
}
```

---

## 13. 易错点

1. **`size()` 返回无符号数**：`for (int i = 0; i < v.size() - 1; ++i)` 在空容器时死循环，写成 `(int)v.size() - 1`。
2. **`vector` 扩容使迭代器失效**：`push_back` 后原来的 `it` 可能指向已释放内存。
3. **`erase` 后迭代器失效**：循环删除要写 `it = s.erase(it);` 而不是 `s.erase(it); ++it;`。
4. **`map::operator[]` 会插入**：查询用 `find` 或 `count`。
5. **`priority_queue` 的 `pop()` 无返回值**：先用 `top()` 取值再 `pop()`；`stack`/`queue` 同理。
6. **`lower_bound` 要求有序**：乱序数组上使用是未定义行为。
7. **`unique` 不真正删除元素**：必须配合 `erase`。
8. **`multiset::erase(x)` 删除所有等于 x 的元素**：只想删一个用 `erase(find(x))`。
9. **`sort` 比较器必须严格弱序**：写 `<=` 会段错误。
10. **`bitset` 大小必须是编译期常量**：`bitset<n>` 里的 `n` 不能用变量。
11. **`unordered_map` 可能被卡**：比赛慎用，或自己写哈希。
12. **`vector<bool>` 不能取地址**：需要引用时改用 `vector<char>`。

---

## 14. 练习建议

### 14.1 学习路线（由易到难）

1. **`vector` 基本操作**：读入数组、排序、去重、逆序。
2. **`string` 操作**：统计字符、查找子串、字符串反转。
3. **`stack` / `queue`**：括号匹配、BFS 最短路。
4. **`priority_queue`**：合并果子、第 k 大。
5. **`set` / `map`**：统计频率、动态有序集合。
6. **`algorithm`**：`lower_bound`、`sort` 自定义比较器。
7. **综合**：离散化、扫描线、事件排序。

### 14.2 刷题清单

| 难度 | 题目 | 用到的 STL |
|---|---|---|
| 入门 | 洛谷 P1059 明明的随机数 | `set` / `sort + unique` |
| 入门 | 洛谷 P1093 奖学金 | `sort` + 结构体 |
| 普及− | 洛谷 P1090 合并果子 | `priority_queue` |
| 普及− | 洛谷 P1102 A-B 数对 | `map` / `lower_bound` |
| 普及/提高− | 洛谷 P1886 滑动窗口 | `deque` / `multiset` |
| 普及/提高− | 洛谷 P1908 逆序对 | 归并排序 / 树状数组 |

### 14.3 学习检查清单

- [ ] 能说出 `vector`、`set`、`map`、`priority_queue` 的底层结构与复杂度
- [ ] 能正确使用 `lower_bound` / `upper_bound`
- [ ] 能写出 `sort + unique + erase` 去重模板
- [ ] 知道 `map::operator[]` 会插入新元素
- [ ] 知道 `priority_queue` 小根堆的写法
- [ ] 能处理迭代器失效问题
- [ ] 能自己选择「该用哪个容器」

> **核心思想**：STL 不是「偷懒」，而是把已经验证过的正确实现拿来用。**知道每个容器的复杂度与失效规则**，才能在赛场上放心地用它。
