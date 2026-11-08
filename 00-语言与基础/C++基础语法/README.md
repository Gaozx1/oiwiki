# C++ 基础语法

> 一句话概括：**C++ 是 OI 的通用语言，变量、控制流、函数、结构体、指针与引用这五块地基打牢了，后面的算法才有落脚点。**

---

## 目录

1. [为什么是 C++](#1-为什么是-c)
2. [第一个程序](#2-第一个程序)
3. [变量与数据类型](#3-变量与数据类型)
4. [运算符与表达式](#4-运算符与表达式)
5. [控制流](#5-控制流)
6. [数组与字符串](#6-数组与字符串)
7. [函数](#7-函数)
8. [结构体与运算符重载](#8-结构体与运算符重载)
9. [指针与引用](#9-指针与引用)
10. [模板代码](#10-模板代码)
11. [经典例题](#11-经典例题)
12. [易错点](#12-易错点)
13. [练习建议](#13-练习建议)

---

## 1. 为什么是 C++

OI 比赛对**运行速度**和**内存占用**有严格限制。C++ 编译成机器码、无虚拟机开销，标准库（`vector`、`sort`、`map`）开箱即用，还支持模板与运算符重载。Python 常数大、Java 启动慢，在 $10^6$ 级数据下容易超时。因此本库统一使用 **C++14**。比赛中我们只关心两件事：**能否编译通过**、**跑得够不够快**。

---

## 2. 第一个程序

```cpp
#include <bits/stdc++.h>   // GCC 万能头，包含几乎所有标准库
using namespace std;       // 把 std 里的名字引入全局作用域

int main() {
    int a, b;
    cin >> a >> b;          // 从标准输入读入两个整数
    cout << a + b << '\n';  // 输出，'\n' 比 endl 快
    return 0;               // 返回 0 表示正常结束
}
```

编译命令（GCC）：

```bash
g++ -std=c++14 -O2 -Wall -o a a.cpp
```

`-std=c++14` 指定标准；`-O2` 开启二级优化，比赛必备；`-Wall` 打开警告，能提前发现大量低级错误。`main` 的返回值传给操作系统，`0` 表示成功，比赛中固定写 `return 0;`。

---

## 3. 变量与数据类型

### 3.1 基本类型

| 类型 | 典型字节 | 取值范围 | 常见用途 |
|---|---|---|---|
| `bool` | 1 | `true` / `false` | 布尔判断 |
| `char` | 1 | $-128 \sim 127$ | 字符（本质是小整数） |
| `short` | 2 | $-32768 \sim 32767$ | 省内存 |
| `int` | 4 | 约 $-2.1\times10^9 \sim 2.1\times10^9$ | 最常用 |
| `long long` | 8 | 约 $-9.2\times10^{18} \sim 9.2\times10^{18}$ | 大整数 |
| `double` | 8 | $\pm1.7\times10^{308}$，约 15 位有效数字 | 浮点运算 |

**经验法则**：答案可能超过 $2\times10^9$ 就用 `long long`。两个 `int` 相乘前先把一个变成 `long long`：

```cpp
int a = 100000, b = 100000;
long long c = 1LL * a * b;   // 正确：1LL 让整个表达式按 long long 计算
long long d = a * b;         // 错误：先按 int 溢出，结果已错
```

### 3.2 声明、初始化与常量

```cpp
int a = 1;      // 拷贝初始化
int b(2);       // 直接初始化
int c{3};       // 列表初始化（C++11），防止窄化转换
int d;          // 未初始化！局部变量的值不确定

const int MAXN = 100005;        // 编译期常量，方便统一改规模
constexpr int N = 1000;         // C++11：强制编译期求值
int x = static_cast<int>(3.7);  // 推荐写法，结果为 3（向零取整）
```

> **全局变量默认初始化为 0，局部变量不会。** 这是 OI 中最常见的 bug 来源之一。

---

## 4. 运算符与表达式

### 4.1 算术运算符

| 运算符 | 含义 | 注意 |
|---|---|---|
| `+ - *` | 加减乘 | 注意溢出 |
| `/` | 除法 | 两个整数相除**向零取整** |
| `%` | 取模 | 结果符号与被除数一致，`-7 % 3 == -1` |
| `++ --` | 自增自减 | `a++` 先用后加，`++a` 先加后用 |

### 4.2 关系与逻辑运算符

```cpp
if (a > 0 && b > 0) { /* ... */ }    // && 短路：左边为假就不算右边
if (a == 0 || b == 0) { /* ... */ }  // || 短路：左边为真就不算右边
```

短路求值常用来避免越界：`if (i < n && a[i] == target)` 中若 `i >= n`，`a[i]` 不会被访问。

### 4.3 位运算预览与优先级

位运算优先级**低于**关系运算符，要写 `(a & 1) == 0` 而不是 `a & 1 == 0`；左移 `1 << k` 只在 $0 \le k < 31$ 时安全，更大要用 `1LL << k`。**优先级记不住就加括号。**

---

## 5. 控制流

### 5.1 `if` / `else if` / `else`

```cpp
if (score >= 90) {
    cout << "A\n";
} else if (score >= 60) {
    cout << "B\n";
} else {
    cout << "C\n";
}
```

`else` 与最近的未匹配 `if` 配对（**悬空 else**），因此即使只有一条语句也建议写大括号。

### 5.2 `switch`

```cpp
switch (op) {
    case '+': ans = a + b; break;
    case '-': ans = a - b; break;
    default:  ans = 0;     break;   // 每个 case 后别忘 break，否则会「贯穿」
}
```

### 5.3 `for` 循环

```cpp
for (int i = 0; i < n; ++i) sum += a[i];   // 标准三段式

for (const auto& x : v) cout << x << ' ';  // 范围 for（C++11），只读遍历
```

**循环不变式**：写 `for` 时要能说出「进入循环体时，`i` 表示什么、`sum` 表示什么」。这是调试循环的第一把钥匙。

### 5.4 `while`、`do-while`、`break` 与 `continue`

```cpp
int i = 0;
while (i < n) { ++i; }              // 先判断后执行

int x;
do { cin >> x; } while (x != 0);    // 至少执行一次

for (int i = 0; i < n; ++i) {
    if (a[i] < 0) continue;         // 跳过本次循环剩余部分
    if (a[i] > 100) break;          // 跳出当前一层循环
    sum += a[i];
}
```

### 5.5 循环的复杂度直觉

单层 `for` 循环体执行 $n$ 次；两层各 $n$ 次就是 $n^2$ 次。**写循环前先估算次数**：$n = 10^5$ 时 $n^2 = 10^{10}$ 必然超时。

---

## 6. 数组与字符串

### 6.1 一维数组

```cpp
int a[100005];              // 全局数组，自动清零，开在静态区
int b[100005] = {0};        // 局部数组，显式清零
int c[] = {1, 2, 3, 4, 5};  // 长度自动推导为 5
```

下标从 `0` 到 `n-1`。**越界访问不会报错**，但会读写到别的内存，产生难以定位的随机错误。二维数组按**行优先**存储，`g[i][j]` 的地址是 `base + (i * 列数 + j) * sizeof(int)`，因此遍历时外层走行、内层走列对缓存更友好。

### 6.2 `char` 数组与 `std::string`

```cpp
char s[1005];
scanf("%s", s);            // 读入一个不含空格的字符串
int len = strlen(s);       // O(n) 求长度，别在循环条件里反复调用

string t;                  // 推荐：自动管理内存
cin >> t;
t += "abc";
cout << (int)t.size() << '\n';
cout << t.substr(1, 3) << '\n';   // 从下标 1 开始取 3 个字符
```

> `string::size()` 返回 `size_t`（无符号），与 `int` 比较时可能出错，建议写成 `(int)t.size()`。

---

## 7. 函数

### 7.1 定义、传值与传引用

```cpp
int add(int x, int y) { return x + y; }                       // 传值

void badSwap(int a, int b) { int t = a; a = b; b = t; }       // 改的是副本，无效
void goodSwap(int& a, int& b) { int t = a; a = b; b = t; }    // 传引用，有效
```

大对象（`vector`、`string`）传参用 `const T&` 避免拷贝：

```cpp
void print(const vector<int>& v) {
    for (int x : v) cout << x << ' ';
}
```

### 7.2 递归

```cpp
long long fact(int n) {
    if (n <= 1) return 1;          // 递归边界
    return 1LL * n * fact(n - 1);  // 递归式
}
```

递归三要素：**边界条件、递归式、返回值含义**。缺边界会无限递归导致栈溢出。小函数可加 `inline` 减少调用开销；虽然 `#define MAX(a,b) ((a)>(b)?(a):(b))` 也能用，但宏容易产生副作用（如 `MAX(i++, j++)` 会自增两次），现代 C++ 更推荐 `inline` 函数与 `constexpr`。

---

## 8. 结构体与运算符重载

```cpp
struct Point {
    int x, y;
    Point() : x(0), y(0) {}                  // 默认构造函数
    Point(int _x, int _y) : x(_x), y(_y) {}  // 带参构造函数
    int norm2() const { return x * x + y * y; }
};

struct Node {
    int val, id;
    bool operator<(const Node& o) const {
        if (val != o.val) return val < o.val;   // 先按 val 升序
        return id < o.id;                        // val 相同按 id 升序
    }
};
```

重载 `<` 之后，`std::sort`、`std::set`、`std::priority_queue` 都能直接使用。

> **严格弱序**：比较函数必须满足「不可自反、传递、反对称」。写 `a.val <= b.val` 会让 `std::sort` 越界崩溃。

---

## 9. 指针与引用

```cpp
int x = 10;
int* p = &x;         // p 保存 x 的地址
cout << *p << '\n';  // 解引用，输出 10
*p = 20;             // 通过指针修改 x

int a[5] = {1, 2, 3, 4, 5};
int* q = a;                // 数组名退化为首元素地址
cout << *(q + 2) << '\n';  // 3，等价于 a[2]

int& r = x;   // r 是 x 的别名
r = 100;      // x 也变成 100
```

引用必须初始化且不能重新绑定；函数参数用引用可以「原地修改」，且没有指针的空值风险。OI 中很少手动 `new`，除非写链表、Trie 等动态结构；空指针统一用 `nullptr`（C++11），不要用 `NULL`。

---

## 10. 模板代码

下面这份骨架可直接编译，覆盖大多数题目的输入输出需求：

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MAXN = 100005;
const int INF = 0x3f3f3f3f;

int n, m;
int a[MAXN];

void readArray() {
    for (int i = 1; i <= n; ++i) cin >> a[i];
}

ll rangeSum(int l, int r) {      // 区间 [l, r] 求和（1 ≤ l ≤ r ≤ n）
    ll s = 0;
    for (int i = l; i <= r; ++i) s += a[i];
    return s;
}

bool isPrime(int x) {            // 试除法判素，O(√n)
    if (x < 2) return false;
    for (int i = 2; 1LL * i * i <= x; ++i)
        if (x % i == 0) return false;
    return true;
}

int main() {
    ios::sync_with_stdio(false);   // 关闭 C 与 C++ 流同步，加速 cin/cout
    cin.tie(nullptr);              // 解除 cin 与 cout 的绑定

    cin >> n;
    readArray();

    int l, r;
    cin >> l >> r;
    cout << rangeSum(l, r) << '\n';

    int cnt = 0;
    for (int i = 1; i <= n; ++i)
        if (isPrime(a[i])) ++cnt;
    cout << cnt << '\n';
    return 0;
}
```

---

## 11. 经典例题

### 例题 1：A+B Problem

**题意**：输入两个整数 $a, b$（$|a|, |b| \le 10^9$），输出 $a + b$。

**思路**：直接读入相加。$10^9 + 10^9 = 2\times10^9$ 仍在 `int` 范围内，但保险起见用 `long long`。

**复杂度**：$O(1)$。

```cpp
int main() {
    long long a, b;
    cin >> a >> b;
    cout << a + b << '\n';
    return 0;
}
```

### 例题 2：斐波那契数列

**题意**：求第 $n$ 项模 $10^9+7$，$n \le 10^6$。

**思路**：递推式

$$f_i = (f_{i-1} + f_{i-2}) \bmod (10^9+7)$$

用数组保存每一项，注意每一步都取模防止溢出。

**复杂度**：时间 $O(n)$，空间 $O(n)$（可滚动数组优化到 $O(1)$）。

```cpp
const int MOD = 1000000007;
long long f[1000005];

int main() {
    int n;
    cin >> n;
    f[0] = 0; f[1] = 1;
    for (int i = 2; i <= n; ++i)
        f[i] = (f[i - 1] + f[i - 2]) % MOD;
    cout << f[n] << '\n';
    return 0;
}
```

### 例题 3：最大子段和

**题意**：给定长度为 $n$ 的整数序列（可能含负数），求连续子段的最大和。

**思路**：Kadane 算法。设 $dp_i$ 表示以 $i$ 结尾的最大子段和，则

$$dp_i = \max(a_i,\ dp_{i-1} + a_i)$$

答案为 $\max_i dp_i$。直观理解：前面的贡献若是负数，不如从当前元素重新开始。

**复杂度**：时间 $O(n)$，空间 $O(1)$。

```cpp
long long maxSubarray(const vector<int>& a) {
    long long ans = LLONG_MIN, cur = 0;
    for (int x : a) {
        cur = max((long long)x, cur + x);
        ans = max(ans, cur);
    }
    return ans;
}
```

---

## 12. 易错点

1. **局部变量未初始化**：`int sum; sum += a[i];` 是未定义行为，养成 `int sum = 0;` 的习惯。
2. **整数溢出**：`int` 上限约 $2.1\times10^9$，乘法前用 `1LL *` 提升类型。
3. **整数除法截断**：`1 / 2 == 0`，要写 `1.0 / 2`；负数除法向零取整。
4. **数组越界**：`a[n]` 已越界。开数组时多开 5~10 个位置（`MAXN + 5`）。
5. **`=` 与 `==` 混用**：`if (x = 0)` 合法但永远为假，编译加 `-Wall` 可提醒。
6. **`for` 边界错误**：`i <= n` 还是 `i < n`，取决于下标从 1 还是 0 开始。
7. **`string::size()` 是无符号数**：`s.size() - 1` 在 `s` 为空时会回绕成巨大值。
8. **`scanf`/`printf` 与 `cin`/`cout` 混用**：关闭同步后混用会乱序，二选一。
9. **递归没有终止条件**：导致栈溢出（RE）。
10. **多组数据不清空**：`vector`、`vis`、计数器要在每组数据开头重置。
11. **`long long` 用 `%lld` 输出**：`printf("%d", 1LL * x)` 会输出错误的值。
12. **`sort` 比较函数写 `<=`**：违反严格弱序，可能段错误。

---

## 13. 练习建议

### 13.1 学习路线（由易到难）

1. **输入输出**：A+B、求平均数、判断奇偶。
2. **分支**：成绩分级、判断三角形、闰年判断。
3. **循环**：求和、求阶乘、九九乘法表、素数判断。
4. **数组**：数组逆序、最大最小值、统计出现次数。
5. **字符串**：回文判断、字符统计、单词计数。
6. **函数**：把上面的题目改写成函数，体会参数传递。
7. **结构体**：学生成绩排序、日期计算。
8. **指针与引用**：手写 `swap`、链表插入删除。
9. **综合**：模拟题、简单贪心、简单递推。

### 13.2 学习检查清单

- [ ] 能独立写出完整程序骨架
- [ ] 能说清 `int` 与 `long long` 的范围差异
- [ ] 能正确使用 `if` / `for` / `while` 并处理边界
- [ ] 能定义结构体并重载 `<`
- [ ] 理解传值与传引用的区别
- [ ] 知道局部变量不会自动清零
- [ ] 能使用 `-Wall` 编译并看懂常见警告

> **核心思想**：C++ 基础语法是「把人类思路翻译成机器步骤」的工具。语法本身不难，难的是**边界条件**与**类型范围**——这两点决定你是 0 分还是 100 分。
