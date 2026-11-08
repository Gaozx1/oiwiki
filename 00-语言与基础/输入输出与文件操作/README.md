# 输入输出与文件操作

> 一句话概括：**OI 程序的第一件事是读数据、最后一件事是写答案，掌握 `cin/cout`、`scanf/printf` 与 `freopen` 三种武器，才能既快又稳地完成这道工序。**

---

## 目录

1. [为什么输入输出值得单独学](#1-为什么输入输出值得单独学)
2. [C 风格：scanf 与 printf](#2-c-风格scanf-与-printf)
3. [C++ 风格：cin 与 cout](#3-c-风格cin-与-cout)
4. [两种风格的速度与取舍](#4-两种风格的速度与取舍)
5. [字符与字符串的读入](#5-字符与字符串的读入)
6. [文件操作：freopen 与 fstream](#6-文件操作freopen-与-fstream)
7. [格式化输出技巧](#7-格式化输出技巧)
8. [模板代码](#8-模板代码)
9. [经典例题](#9-经典例题)
10. [易错点](#10-易错点)
11. [练习建议](#11-练习建议)

---

## 1. 为什么输入输出值得单独学

一道题的数据规模可以到 $10^6$ 个整数。如果读写本身慢了 0.5 秒，算法再优秀也会 TLE。更常见的情况是**读错格式**：

- `scanf("%d")` 读 `long long` → 只写入了低 4 字节，结果随机；
- `cin >> s` 读含空格的字符串 → 只读到第一个空格；
- 多组数据忘写 `while (scanf(...) != EOF)` → 只处理第一组。

这些错误不会编译报错，只会让你在 WA 与 RE 之间反复横跳。所以输入输出必须**当成一个知识点系统学**。

---

## 2. C 风格：scanf 与 printf

### 2.1 常用格式符

| 类型 | 读入 | 输出 |
|---|---|---|
| `int` | `%d` | `%d` |
| `long long` | `%lld` | `%lld` |
| `unsigned` | `%u` | `%u` |
| `double` | `%lf` | `%f`（注意区别！） |
| `char` | `%c` | `%c` |
| `char[]` | `%s` | `%s` |
| `float` | `%f` | `%f` |

> **易错**：`double` 读入用 `%lf`，输出用 `%f`；`long long` 必须用 `%lld`。写错不会报错，只会得到垃圾值。

### 2.2 基本用法

```cpp
int a, b;
scanf("%d%d", &a, &b);         // 读入两个整数，注意取地址符 &
printf("%d\n", a + b);
long long x;
scanf("%lld", &x);
printf("%lld\n", x);
double d;
scanf("%lf", &d);
printf("%.3f\n", d);           // 保留 3 位小数
```

`scanf` 的返回值是**成功读入的变量个数**，可用来判断是否读到文件末尾：

```cpp
while (scanf("%d", &n) != EOF) {
    // 处理一组数据
}
while (~scanf("%d", &n)) { }     // 更简洁：~EOF == ~(-1) == 0，读不到时循环结束
```

### 2.3 printf 的格式控制

```cpp
printf("%5d\n", 42);        // 右对齐，宽度 5：`   42`
printf("%-5d|\n", 42);      // 左对齐：`42   |`
printf("%05d\n", 42);       // 补前导零：`00042`
printf("%.2f\n", 3.14159);  // 保留 2 位小数：`3.14`
printf("%x\n", 255);        // 十六进制小写：`ff`
printf("%o\n", 8);          // 八进制：`10`
printf("%c\n", 'A');        // 字符：`A`
printf("%%\n");             // 输出一个百分号
```

宽度可以用 `*` 动态指定：`printf("%*d\n", 5, 42);` 等价于 `%5d`。

---

## 3. C++ 风格：cin 与 cout

### 3.1 基本用法与加速

```cpp
int a, b;
cin >> a >> b;                // 自动识别类型，不需要格式符与取地址符
cout << a + b << '\n';

ios::sync_with_stdio(false);   // 关闭 C 与 C++ 流同步
cin.tie(nullptr);              // 解除 cin 与 cout 的绑定
```

`cin` 的优势是**类型安全**：读 `long long` 不会像 `%d` 那样截断；`cout` 不用记格式符。代价是默认速度比 `scanf/printf` 慢，而上面两行加速通常能带来 **3~5 倍**提速。注意：关闭同步后**不能再用 `scanf`/`printf`**，否则输入输出顺序可能错乱。

### 3.2 读取失败的判断

```cpp
int x;
while (cin >> x) {             // 读到 EOF 时表达式为 false
    // 处理 x
}
```

`cin >> x` 返回流对象，在布尔上下文中表示「是否读取成功」。

---

## 4. 两种风格的速度与取舍

| 维度 | `scanf/printf` | `cin/cout`（未加速） | `cin/cout`（加速后） |
|---|---|---|---|
| 速度 | 快 | 慢 | 很快 |
| 类型安全 | 依赖格式符 | 强 | 强 |
| 写法简洁度 | 一般 | 高 | 高 |
| 输出格式控制 | 强大（`%.3f`、补零） | 需 `iomanip` | 需 `iomanip` |

**实战建议**：数据量 $< 10^5$ 随便用哪个顺手；数据量 $> 10^6$ 用加速后的 `cin/cout` 或 `scanf/printf`；需要 `%.3f` 这类格式化输出用 `printf` 最方便；读 `long long` 用 `cin` 更不容易出错。

---

## 5. 字符与字符串的读入

### 5.1 `scanf("%c")` 的换行符陷阱

```cpp
int n;
scanf("%d", &n);
char c;
scanf("%c", &c);      // 读到的可能是上一行遗留的 '\n'！
```

解决方法：在 `%c` 前加一个空格，跳过空白字符：

```cpp
scanf(" %c", &c);     // 注意 %c 前面有个空格
```

或者用 `cin >> c`，它默认跳过空白。

### 5.2 读一整行

```cpp
char line[1005];
fgets(line, sizeof(line), stdin);   // C 风格，包含行末换行符
cin.getline(line, 1005);            // C++ 风格
string s;
getline(cin, s);                    // 推荐：读整行到 string
```

`char[]` 要预留足够空间，且 `scanf("%s")` **不会检查越界**（可以用 `%1000s` 限宽）。字符判断函数 `isalpha`、`isdigit`、`isspace`、`tolower` 等参数是 `int`，传入 `char` 时最好先转 `unsigned char`，否则负数行为未定义。

---

## 6. 文件操作：freopen 与 fstream

### 6.1 freopen（比赛最常用）

```cpp
#include <cstdio>
int main() {
    freopen("input.txt", "r", stdin);    // 把标准输入重定向到文件
    freopen("output.txt", "w", stdout);  // 把标准输出重定向到文件
    int a, b;
    scanf("%d%d", &a, &b);
    printf("%d\n", a + b);
    return 0;
}
```

参数含义：`"r"` 只读、`"w"` 只写（覆盖）、`"a"` 追加。重定向后，`cin`/`cout` 也会跟着走文件，因为底层都是标准流。

顺便估算一下内存：读入 $10^6$ 个 `int` 需要

$$10^6 \times 4\ \text{字节} \approx 4\ \text{MB}$$

**注意**：很多 OJ 要求文件名与题目规定一致（如 `in.txt`/`out.txt` 或 `xxx.in`/`xxx.out`），提交前务必核对。使用 `freopen` 后，本地调试时不要再从键盘输入。

### 6.2 fstream（更灵活的 C++ 方式）

```cpp
#include <fstream>
using namespace std;
int main() {
    ifstream fin("input.txt");
    ofstream fout("output.txt");
    int a, b;
    fin >> a >> b;
    fout << a + b << '\n';
    return 0;
}
```

`fstream` 不污染标准流，适合需要同时读写多个文件的场景。比赛一般用 `freopen` 更省事。

### 6.3 文件打开失败检查

```cpp
ifstream fin("data.txt");
if (!fin) {
    cerr << "无法打开文件\n";   // cerr 输出到标准错误，不会被重定向
    return 1;
}
```
---

## 7. 格式化输出技巧

```cpp
printf("%.6f\n", ans);                              // 保留 6 位小数
cout << fixed << setprecision(6) << ans << '\n';    // 需要 <iomanip>
printf("%02d:%02d\n", h, m);                        // 09:05
printf("%8.3f\n", x);                               // 宽度 8，保留 3 位小数
```

浮点数比较不要用 `==`，用 `fabs(a - b) < 1e-8`。另外 `cout << endl` 会**强制刷新缓冲区**，大量输出时用 `'\n'` 代替 `endl`：

```cpp
for (int i = 0; i < n; ++i) cout << a[i] << '\n';   // 快
// for (int i = 0; i < n; ++i) cout << a[i] << endl; // 慢，可能 TLE
```

---

## 8. 模板代码

### 8.1 快速读入（手写快读）

数据量极大（$10^7$ 级别）时，可以手写基于 `getchar` 的读入：

```cpp
#include <bits/stdc++.h>
using namespace std;
inline int readInt() {
    int x = 0, f = 1;
    char c = getchar();
    while (c < '0' || c > '9') {          // 跳过非数字字符
        if (c == '-') f = -1;
        c = getchar();
    }
    while (c >= '0' && c <= '9') {        // 逐位累加
        x = x * 10 + (c - '0');
        c = getchar();
    }
    return x * f;
}
int main() {
    int n = readInt();
    long long sum = 0;
    for (int i = 0; i < n; ++i) sum += readInt();
    printf("%lld\n", sum);
    return 0;
}
```

### 8.2 标准输入输出骨架

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n;
    cin >> n;
    vector<long long> a(n);
    for (auto& x : a) cin >> x;
    long long sum = 0;
    for (long long x : a) sum += x;
    cout << sum << '\n';
    return 0;
}
```

### 8.3 多组数据骨架

```cpp
int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n;
        scanf("%d", &n);
        // 处理每组数据
    }
    return 0;
}
```

若题目不给 $T$、读到 EOF 为止：

```cpp
int n;
while (scanf("%d", &n) == 1) { /* 处理 */ }
```

---

## 9. 经典例题

### 例题 1：A+B（多组数据）

**题意**：不断读入两个整数 $a, b$，输出 $a+b$，直到文件结束。

**思路**：用 `while (cin >> a >> b)` 循环读取，读到 EOF 自动结束。

**复杂度**：每组 $O(1)$。

```cpp
int main() {
    long long a, b;
    while (cin >> a >> b) {
        cout << a + b << '\n';
    }
    return 0;
}
```

### 例题 2：保留小数

**题意**：给定三个浮点数，输出它们的平均值，保留 3 位小数。

**思路**：累加后除以 3，用 `printf("%.3f")` 或 `fixed << setprecision(3)`。

**复杂度**：$O(1)$。

```cpp
int main() {
    double a, b, c;
    scanf("%lf%lf%lf", &a, &b, &c);
    printf("%.3f\n", (a + b + c) / 3.0);
    return 0;
}
```

### 例题 3：统计单词数

**题意**：读入一行英文句子（含空格），统计其中单词的个数。单词之间由若干空格分隔。

**思路**：用 `getline` 读整行，然后逐字符扫描：遇到非空格且前一个是空格（或开头）时，单词数加一。

**复杂度**：$O(n)$，$n$ 为句子长度。

```cpp
int main() {
    string s;
    getline(cin, s);
    int cnt = 0;
    bool inWord = false;
    for (char c : s) {
        if (c != ' ') {
            if (!inWord) { ++cnt; inWord = true; }
        } else {
            inWord = false;
        }
    }
    cout << cnt << '\n';
    return 0;
}
```

### 例题 4：文件读写

**题意**：从 `input.txt` 读入 $n$ 个整数，求和后写入 `output.txt`。

**思路**：用 `freopen` 重定向标准输入输出，其余代码与普通题完全一致。

**复杂度**：$O(n)$。

```cpp
int main() {
    freopen("input.txt", "r", stdin);
    freopen("output.txt", "w", stdout);
    int n;
    scanf("%d", &n);
    long long sum = 0;
    for (int i = 0; i < n; ++i) {
        int x;
        scanf("%d", &x);
        sum += x;
    }
    printf("%lld\n", sum);
    return 0;
}
```

---

## 10. 易错点

1. **`scanf` 忘写 `&`**：`scanf("%d", a)` 会崩溃或写坏内存。
2. **`long long` 用 `%d`**：只写入低 4 字节，结果错误。必须 `%lld`。
3. **`double` 读 `%f`**：读入必须 `%lf`，输出才是 `%f`。
4. **`%c` 读到换行符**：在 `%c` 前加空格，或用 `cin >> c`。
5. **`gets` 已废弃**：C++14 起不能用，改用 `fgets` 或 `getline`。
6. **`cin`/`cout` 与 `scanf`/`printf` 混用**：关闭同步后混用会乱序。
7. **`endl` 太慢**：大量输出时用 `'\n'`。
8. **`freopen` 文件名写错**：本地能跑、提交 0 分。核对题目要求。
9. **多组数据不清空**：全局数组、`vector`、计数器每组重置。
10. **`while (scanf("%d", &n))` 死循环**：`scanf` 失败返回 0 或 EOF，`0` 在布尔上下文中是假，但若返回 EOF（-1）仍为真——应写 `!= EOF` 或 `== 1`。
11. **输出格式多空格/少换行**：OJ 通常严格比对，末尾空格也可能判错。
12. **`printf("%.3f")` 四舍五入**：浮点误差可能导致 `2.675` 输出 `2.67`，需要时加 `eps`。

---

## 11. 练习建议

### 11.1 学习路线（由易到难）

1. **单组输入输出**：A+B、求平均数。
2. **多组数据**：读到 EOF、`while (T--)`。
3. **字符与字符串**：统计字符、单词计数、整行读入。
4. **格式化输出**：保留小数、补零、对齐。
5. **文件操作**：`freopen` 读写本地文件。
6. **性能优化**：`ios::sync_with_stdio(false)`、手写快读。

### 11.2 刷题清单

| 难度 | 题目 | 训练点 |
|---|---|---|
| 入门 | 洛谷 P1001 A+B Problem | 基本输入输出 |
| 入门 | 洛谷 P1000 超级玛丽游戏 | 多行输出 |
| 普及− | 洛谷 P1035 级数求和 | 循环 + 浮点输出 |
| 普及− | 洛谷 P1308 统计单词数 | 整行读入 + 字符串 |
| 普及/提高− | 洛谷 P1553 数字反转 | 字符串处理 |
| 普及/提高− | 洛谷 P1090 合并果子 | 大规模读写效率 |

### 11.3 学习检查清单

- [ ] 能默写 `scanf`/`printf` 的常用格式符
- [ ] 知道 `%lf` 与 `%f` 的区别
- [ ] 会写 `while (cin >> x)` 与 `while (scanf(...) != EOF)`
- [ ] 会用 `getline` 读整行并处理换行符残留
- [ ] 会写 `freopen` 的文件重定向
- [ ] 知道 `endl` 与 `'\n'` 的区别
- [ ] 能写出手写快读模板

> **核心思想**：输入输出是程序的「咽喉」，读错一个字符、写慢一个数量级，都可能让正确的算法拿不到分。**先保证读对，再追求读快。**
