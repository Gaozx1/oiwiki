# 栈（Stack）

> 一句话概括：**栈是一种「后进先出（LIFO）」的线性结构**，只允许在栈顶插入和删除，所有操作都是 $O(1)$。

---

## 目录

1. [概念与原理](#1-概念与原理)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点](#4-易错点)
5. [练习建议](#5-练习建议)

---

## 1. 概念与原理

### 1.1 什么是栈

栈（Stack）是一种**受限的线性表**：插入和删除只能在同一端进行，这一端称为**栈顶（top）**，另一端称为**栈底（bottom）**。

- `push(x)`：把元素 `x` 压入栈顶；
- `pop()`：弹出栈顶元素；
- `top()`：读取栈顶元素（不弹出）；
- `empty()` / `size()`：判空与查询元素个数。

因为只能在栈顶操作，所以**最后压入的元素最先弹出**，这就是「后进先出」（Last In First Out，LIFO）。

### 1.2 文字示意图

把栈想象成一个底部封死、顶部开口的杯子，只能从杯口放取物品：

```
        栈顶
         ↓
      +-----+
      |  30 |  ← 最后 push 进来的元素，最先 pop
      +-----+
      |  20 |
      +-----+
      |  10 |  ← 最早 push 进来的元素，最后 pop
      +-----+
      |     |
      +-----+
        栈底
```

依次执行 `push(10)`、`push(20)`、`push(30)` 后：

```
操作            栈内元素（底 → 顶）      返回值
push(10)        [10]
push(20)        [10, 20]
push(30)        [10, 20, 30]
top()           [10, 20, 30]            30
pop()           [10, 20]                30
pop()           [10]                    20
top()           [10]                    10
```

### 1.3 为什么需要栈

栈的「后进先出」恰好对应很多天然具有**嵌套**或**回溯**性质的过程：

| 场景 | 为什么适合栈 |
|---|---|
| 函数调用 | 后调用的函数先返回，系统用「调用栈」保存返回地址和局部变量 |
| 括号匹配 | 左括号后遇到的右括号，必须先与最近的左括号配对 |
| 表达式求值 | 运算符的优先级与结合方向天然是「最近优先」 |
| 撤销操作（Undo） | 最后一步操作最先被撤销 |
| 深度优先搜索（DFS） | 用栈模拟递归的「回溯」过程 |
| 单调栈 | 用栈维护「尚未找到答案」的元素集合 |

### 1.4 栈的两种常见实现

**（1）数组模拟（顺序栈）**

用一个数组 `data[]` 加一个栈顶下标 `topIndex` 即可，`topIndex == -1` 表示空栈：

```
data:  [ 10 | 20 | 30 |    |    ]
下标:     0    1    2    3    4
                  ↑
              topIndex = 2
```

- `push(x)`：`data[++topIndex] = x`
- `pop()`：`--topIndex`
- `top()`：`data[topIndex]`

优点是常数极小、缓存友好，是竞赛中最常用的写法。

**（2）STL `std::stack`**

```cpp
#include <stack>
std::stack<int> st;
st.push(1);        // 入栈
st.top();          // 取栈顶
st.pop();          // 弹出栈顶（不返回值！）
st.empty();        // 是否为空
st.size();         // 元素个数
```

`std::stack` 是容器适配器，默认用 `std::deque` 作为底层容器，也可以指定 `std::vector`：

```cpp
std::stack<int, std::vector<int>> st;
```

> **注意**：`std::stack::pop()` 只负责删除，不返回被删除的值。要取栈顶请先 `top()` 再 `pop()`。

### 1.5 合法出栈序列

给定入栈顺序 $1,2,\dots,n$，问哪些出栈序列是合法的。判定方法很简单：**用栈模拟**。依次压入 $1 \sim n$，每压入一个就尽可能按目标序列弹出，若最终能弹空则合法。

一个经典结论：长度为 $n$ 的入栈序列对应的合法出栈序列个数为第 $n$ 个卡特兰数：

$$
C_n = \frac{1}{n+1}\binom{2n}{n}
$$

例如 $n=3$ 时共有 $C_3=5$ 种：`123`、`132`、`213`、`231`、`321`，而 `312` 不可能出现——因为 `3` 出栈时 `1、2` 都在栈中，`2` 一定在 `1` 上面，`1` 不可能先于 `2` 出栈。

---

## 2. 模板代码

下面是一份**完整可编译**的数组栈实现，支持 `push`、`pop`、`top`、`size` 四种操作。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;   // 栈的最大容量

class Stack {
private:
    int data[MAXN];        // 用数组存储栈中元素
    int topIndex;          // 栈顶下标，-1 表示空栈

public:
    Stack() : topIndex(-1) {}

    bool empty() const { return topIndex == -1; }

    int size() const { return topIndex + 1; }

    // 入栈：先移动栈顶指针，再写入元素
    void push(int x) {
        if (topIndex + 1 >= MAXN) {          // 防溢出，实际题目中一般不会触发
            cerr << "Stack overflow!" << '\n';
            return;
        }
        data[++topIndex] = x;
    }

    // 出栈：只需要把栈顶指针下移，旧值无需清空
    void pop() {
        if (empty()) return;                 // 空栈弹出是未定义行为，这里做保护
        --topIndex;
    }

    // 取栈顶：调用前必须保证栈非空
    int top() const { return data[topIndex]; }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int q;
    cin >> q;
    Stack st;
    while (q--) {
        string op;
        cin >> op;
        if (op == "push") {
            int x;
            cin >> x;
            st.push(x);
        } else if (op == "pop") {
            if (!st.empty()) st.pop();
        } else if (op == "top") {
            if (st.empty()) cout << "empty" << '\n';
            else cout << st.top() << '\n';
        } else if (op == "size") {
            cout << st.size() << '\n';
        }
    }
    return 0;
}
```

输入示例：

```
7
push 10
push 20
top
pop
top
size
pop
```

输出：

```
20
10
1
```

### STL 版本速查

```cpp
stack<int> st;
st.push(3);              // 入栈
int x = st.top();        // 取栈顶（不删除）
st.pop();                // 删除栈顶（不返回）
bool b = st.empty();
size_t s = st.size();
while (!st.empty()) st.pop();   // 清空栈的唯一稳妥方式
```

---

## 3. 经典例题

### 3.1 括号匹配

**题意**：给定一个只含 `(`、`)`、`[`、`]`、`{`、`}` 的字符串，判断括号是否合法（每个右括号都能与最近的同类型左括号配对）。

**思路**：扫描字符串，遇到左括号就压栈；遇到右括号时：

- 若栈空，说明没有对应的左括号，非法；
- 否则弹出栈顶，检查类型是否匹配；
- 扫描结束后栈必须为空。

**复杂度**：时间复杂度 $O(n)$，空间复杂度 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

bool check(const string& s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') {
            st.push(c);
        } else {
            if (st.empty()) return false;          // 右括号多出来了
            char t = st.top();
            st.pop();
            if ((c == ')' && t != '(') ||
                (c == ']' && t != '[') ||
                (c == '}' && t != '{')) {
                return false;                      // 括号类型不匹配
            }
        }
    }
    return st.empty();                             // 左括号必须全部配对
}

int main() {
    string s;
    cin >> s;
    cout << (check(s) ? "Yes" : "No") << '\n';
    return 0;
}
```

### 3.2 后缀表达式求值

**题意**：给定一个后缀表达式（逆波兰式），其中操作数为整数，运算符为 `+ - * /`，求表达式的值。

**思路**：后缀表达式的求值是栈最直接的应用：

- 遇到数字：压栈；
- 遇到运算符：弹出两个数 `b`（栈顶，右操作数）和 `a`（次栈顶，左操作数），计算 `a op b`，把结果压回栈；
- 最后栈中剩下的唯一元素就是答案。

**复杂度**：时间复杂度 $O(n)$，空间复杂度 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    stack<long long> st;
    string token;
    while (cin >> token) {
        if (token == "+" || token == "-" || token == "*" || token == "/") {
            long long b = st.top(); st.pop();
            long long a = st.top(); st.pop();
            if (token == "+") st.push(a + b);
            else if (token == "-") st.push(a - b);
            else if (token == "*") st.push(a * b);
            else st.push(a / b);
        } else {
            st.push(stoll(token));   // 操作数直接入栈
        }
    }
    cout << st.top() << '\n';
    return 0;
}
```

> **易错**：减法与除法**不满足交换律**，出栈顺序是先右操作数、后左操作数，计算时必须写成 `a - b`、`a / b`，不能写反。

### 3.3 最小栈

**题意**：设计一个栈，支持 `push`、`pop`、`top` 以及在 $O(1)$ 时间内取得栈中最小元素 `getMin`。

**思路**：维护两个栈：数据栈 `data` 和最小值栈 `mins`。

- `push(x)`：`data` 压入 `x`；`mins` 压入 `min(x, mins.top())`（`mins` 为空时压入 `x`）；
- `pop()`：两个栈同时弹出；
- `getMin()`：返回 `mins.top()`。

本质是「每个位置记录从栈底到该位置的最小值」，用空间换时间。

**复杂度**：所有操作 $O(1)$，空间 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinStack {
    stack<int> data, mins;
public:
    void push(int x) {
        data.push(x);
        mins.push(mins.empty() ? x : min(x, mins.top()));
    }
    void pop() {
        if (data.empty()) return;
        data.pop();
        mins.pop();
    }
    int top() { return data.top(); }
    int getMin() { return mins.top(); }
    bool empty() { return data.empty(); }
};

int main() {
    MinStack st;
    st.push(5); st.push(2); st.push(7);
    cout << st.getMin() << '\n';   // 2
    st.pop();
    cout << st.getMin() << '\n';   // 2
    st.pop();
    cout << st.getMin() << '\n';   // 5
    return 0;
}
```

### 3.4 验证栈序列

**题意**：给定入栈序列 `pushed` 和出栈序列 `popped`（元素互不相同），判断 `popped` 是否可能是 `pushed` 的合法出栈序列。

**思路**：用一个栈模拟。按 `pushed` 顺序压栈，每压入一个元素后，只要栈顶等于 `popped` 的当前待弹出元素，就不断弹出并后移指针。最后判断栈是否为空。

**复杂度**：$O(n)$，每个元素至多入栈、出栈一次。

```cpp
#include <bits/stdc++.h>
using namespace std;

bool validateStackSequences(const vector<int>& pushed, const vector<int>& popped) {
    stack<int> st;
    size_t j = 0;
    for (int x : pushed) {
        st.push(x);
        while (!st.empty() && j < popped.size() && st.top() == popped[j]) {
            st.pop();
            ++j;
        }
    }
    return st.empty();
}

int main() {
    vector<int> a = {1, 2, 3, 4, 5};
    vector<int> b = {4, 5, 3, 2, 1};
    cout << (validateStackSequences(a, b) ? "Yes" : "No") << '\n';  // Yes
    return 0;
}
```

---

## 4. 易错点

1. **空栈访问**：`top()` / `pop()` 前必须判 `empty()`，对空栈操作是未定义行为（数组写法会越界读 `data[-1]`）。
2. **`std::stack::pop()` 没有返回值**：想取值必须先 `top()` 再 `pop()`，不能写 `int x = st.pop();`。
3. **出栈顺序搞反**：后缀表达式求值时，栈顶是**右操作数**，次栈顶才是左操作数，`a - b` 与 `b - a` 结果不同。
4. **括号匹配忘清空检查**：扫描完字符串后必须检查栈是否为空，否则 `"(()"` 会被误判为合法。
5. **数组栈容量开小**：元素个数可能达到 $n$，数组至少要开 $n+1$，不要凭感觉开 `1000`。
6. **多组数据忘记清空栈**：`while (!st.empty()) st.pop();` 或干脆在每组数据里重新声明一个局部 `stack`。
7. **栈与递归的对应关系**：递归深度过大（如 $10^6$ 层）会导致系统栈溢出，此时要改写成手写栈的迭代版本。
8. **`stack` 不能遍历**：STL 栈不提供迭代器，需要遍历时请用 `vector` 或 `deque` 模拟。

---

## 5. 练习建议

### 5.1 由易到难的路线

1. **基础模板**：手写数组栈，实现 `push / pop / top / size`，并完成「验证栈序列」。
2. **括号匹配**：单种括号 → 多种括号 → 带其他字符的表达式（洛谷 P1739）。
3. **表达式求值**：后缀表达式 → 中缀转后缀 → 带括号与优先级的完整中缀求值。
4. **最小栈 / 最大栈**：双栈思想，体会「用空间换时间」。
5. **栈 + 模拟**：用栈处理「删除相邻重复项」「字符串解码」「简化路径」等问题。
6. **栈 + DFS**：用显式栈代替递归完成深度优先搜索，理解系统调用栈。
7. **进阶**：单调栈（见「单调栈」一章）、用栈实现队列、用队列实现栈。

### 5.2 推荐题目

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1739 表达式括号匹配 | 括号匹配 |
| 入门 | 洛谷 P1449 后缀表达式 | 后缀表达式求值 |
| 基础 | LeetCode 155 最小栈 | 双栈 |
| 基础 | LeetCode 946 验证栈序列 | 栈模拟 |
| 基础 | LeetCode 20 有效的括号 | 括号匹配 |
| 提高 | 洛谷 P1175 表达式的转换 | 中缀转后缀 + 求值 |
| 提高 | 洛谷 P1044 栈 | 卡特兰数 / 动态规划 |

### 5.3 学习检查清单

- [ ] 能说清「后进先出」的含义并画出 push/pop 过程
- [ ] 能默写数组栈的四个基本操作
- [ ] 能独立完成括号匹配与后缀表达式求值
- [ ] 理解为什么合法出栈序列数是卡特兰数
- [ ] 知道 `std::stack::pop()` 不返回值的坑
- [ ] 能用栈把递归改写成非递归

> **核心思想**：栈用「只在一端操作」换来了 $O(1)$ 的插入删除，并天然匹配嵌套、回溯、最近优先这类问题——看到「配对」「撤销」「最近」等字眼，就该想到栈。
