# 链表（Linked List）

> 一句话概括：**链表用「指针 + 结点」把数据串成一条链**，插入、删除只要 $O(1)$ 改指针，但随机访问需要 $O(n)$ 逐个走。

---

## 目录

1. [概念与原理](#1-概念与原理)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点](#4-易错点)
5. [练习建议](#5-练习建议)

---

## 1. 概念与原理

### 1.1 为什么需要链表

`vector` / 数组的优点是**随机访问 $O(1)$**，缺点是**中间插入、删除要搬移 $O(n)$ 个元素**，且扩容时要整体复制。链表反过来：用一组**离散存储**的结点，结点之间用指针连起来。

- 插入 / 删除：先找到位置（$O(n)$），改指针即可（$O(1)$）；
- 随机访问：必须从头结点开始逐个数，$O(n)$；
- 不需要连续空间，扩容只需新建结点。

因此链表适合「**频繁在已知位置插入删除、很少随机访问**」的场景。

### 1.2 结点与链的结构

单链表的每个结点包含**数据域 `data`** 和**指针域 `next`**（指向下一个结点）。一条单链表：

```
head
 ↓
+---+---+    +---+---+    +---+---+    +---+---+
| 1 | ●-+-->| 2 | ●-+-->| 3 | ●-+-->| 4 | / |
+---+---+    +---+---+    +---+---+    +---+---+
                                        tail（next = nullptr）
```

最后一个结点的 `next` 指向 `nullptr`，表示链表结束。

### 1.3 单链表、双链表、循环链表

- **单链表**：每个结点只有一个 `next` 指针，只能单向遍历，不能 $O(1)$ 找到前驱。
- **双链表**：每个结点有 `prev` 和 `next` 两个指针：

```
      +-------+-------+-------+
      | prev  | data  | next  |
      +-------+-------+-------+
         ↖       ↑        ↘
    指向前一个   数据    指向后一个
```

双链表可以双向遍历，已知某结点时能在 $O(1)$ 内删除它（因为能拿到前驱）。代价是每个结点多一个指针，插入删除要维护两条链。

- **循环链表**：尾结点的 `next` 指回头结点，形成环：

```
      +---+---+    +---+---+    +---+---+
  +-->| 1 | ●-+-->| 2 | ●-+-->| 3 | ●-+--+
  |   +---+---+    +---+---+    +---+---+  |
  +----------------------------------------+
```

循环链表适合「围成一圈」的问题（如约瑟夫环），遍历时用「是否回到起点」做终止条件。

### 1.4 带哨兵结点（dummy head）

直接在头结点上操作时，插入/删除第一个结点是**特殊情况**，代码里到处要写 `if (head == nullptr)`。加一个**哨兵结点**（虚拟头结点）可以统一处理：

```
dummy -> [1] -> [2] -> [3] -> nullptr
```

哨兵不存有效数据，真正的头结点是 `dummy->next`。这样「在头部插入」和「在中间插入」写法完全一样，大幅减少边界判断，是竞赛中最推荐的技巧。

### 1.5 数组模拟链表

竞赛中常遇到「结点编号是 $1 \sim n$、需要多次插入删除」的题。动态 `new` 速度慢且容易内存泄漏，更常用**数组模拟**：用 `val[MAXN]` 存值、`nxt[MAXN]` 存后继下标，`0` 作为空指针。插入删除只改 `nxt[]`，且支持 $O(1)$ 访问任意编号的结点——「邻接表」「链式前向星」也是同一套思想。

### 1.6 复杂度对比

随机访问数组 $O(1)$、链表 $O(n)$；头部插入删除链表 $O(1)$、数组 $O(n)$；删除已知结点单链表需 $O(n)$ 找前驱，双链表 $O(1)$。数组内存连续、缓存友好；链表结点离散，每个结点额外付出 1～2 个指针的空间。

---

## 2. 模板代码

### 2.1 单链表（指针版，带哨兵）

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int val;
    Node* next;
    explicit Node(int v = 0, Node* n = nullptr) : val(v), next(n) {}
};

class LinkedList {
    Node* dummy;          // 哨兵，dummy->next 才是真正的头
    int len = 0;
public:
    LinkedList() : dummy(new Node()) {}
    ~LinkedList() {                       // 释放所有结点
        for (Node* p = dummy; p;) { Node* q = p->next; delete p; p = q; }
    }
    int size() const { return len; }

    // 在第 index 个位置（0 起始）前插入，index == len 表示尾插
    void insert(int index, int value) {
        if (index < 0 || index > len) return;
        Node* prev = dummy;
        for (int i = 0; i < index; ++i) prev = prev->next;
        prev->next = new Node(value, prev->next);   // 接在前驱之后
        ++len;
    }
    // 删除第 index 个结点
    bool erase(int index) {
        if (index < 0 || index >= len) return false;
        Node* prev = dummy;
        for (int i = 0; i < index; ++i) prev = prev->next;
        Node* victim = prev->next;
        prev->next = victim->next;        // 跨过被删结点
        delete victim;
        --len;
        return true;
    }
    // 查找第一个值为 value 的结点下标，找不到返回 -1
    int find(int value) const {
        int idx = 0;
        for (Node* p = dummy->next; p; p = p->next, ++idx)
            if (p->val == value) return idx;
        return -1;
    }
    void print() const {
        for (Node* p = dummy->next; p; p = p->next) cout << p->val << ' ';
        cout << '\n';
    }
};

int main() {
    LinkedList list;
    list.insert(0, 1);  list.insert(1, 3);  list.insert(1, 2);
    list.print();                        // 1 2 3
    cout << list.find(2) << '\n';        // 1
    list.erase(0);  list.print();        // 2 3
    return 0;
}
```

### 2.2 数组模拟单链表（竞赛常用）

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
int val[MAXN], nxt[MAXN];   // 0 表示空指针
int tot = 0;                // 已分配结点数，编号从 1 开始

int newNode(int v) { val[++tot] = v; nxt[tot] = 0; return tot; }

int main() {
    int head = 0;
    // 头插法：依次插入 1, 2, 3，链表变成 3 -> 2 -> 1
    for (int i = 1; i <= 3; ++i) {
        int id = newNode(i);
        nxt[id] = head;  head = id;
    }
    // 尾插法：维护尾指针
    int tail = 0;  head = 0;
    for (int i = 1; i <= 3; ++i) {
        int id = newNode(i);
        if (head == 0) head = tail = id;
        else { nxt[tail] = id; tail = id; }
    }
    for (int p = head; p; p = nxt[p]) cout << val[p] << ' ';
    cout << '\n';                   // 1 2 3

    // 在 head 后插入 100，再删除这个新结点
    int id = newNode(100);
    nxt[id] = nxt[head];  nxt[head] = id;
    nxt[head] = nxt[nxt[head]];

    for (int p = head; p; p = nxt[p]) cout << val[p] << ' ';
    cout << '\n';                   // 1 3
    return 0;
}
```

### 2.3 双链表（数组模拟）

用 `0` 和 `1` 作为虚拟的头、尾哨兵，真实结点编号从 `2` 开始，这样插入删除不必特判边界。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 100005;
int val[MAXN], pre[MAXN], nxt[MAXN];
int tot = 1;                      // 0、1 被哨兵占用

// 在结点 p 之后插入值为 v 的新结点
void insertAfter(int p, int v) {
    int id = ++tot;
    val[id] = v;
    pre[id] = p;                  // 新结点前驱是 p
    nxt[id] = nxt[p];             // 新结点接管 p 原来的后继
    pre[nxt[p]] = id;             // 原后继的前驱改成新结点
    nxt[p] = id;                  // p 的后继改成新结点
}

// 删除真实结点 p（两侧都有邻居）
void erase(int p) {
    nxt[pre[p]] = nxt[p];         // 前驱跨过 p
    pre[nxt[p]] = pre[p];         // 后继跨过 p
}

int main() {
    nxt[0] = 1;  pre[1] = 0;      // 初始：0 <-> 1（空表）
    insertAfter(0, 10);           // 0 <-> 10 <-> 1
    insertAfter(0, 20);           // 0 <-> 20 <-> 10 <-> 1
    for (int p = nxt[0]; p != 1; p = nxt[p]) cout << val[p] << ' ';
    cout << '\n';                 // 20 10
    erase(2);                     // 删除结点 2（值 10）
    for (int p = nxt[0]; p != 1; p = nxt[p]) cout << val[p] << ' ';
    cout << '\n';                 // 20
    return 0;
}
```

---

## 3. 经典例题
### 3.1 反转链表

**题意**：给定单链表头结点，把它反转，返回新的头结点。

**思路**：用三个指针 `prev`、`cur`、`next` 迭代：记录 `next = cur->next`；把 `cur->next` 指向 `prev`；再令 `prev = cur`，`cur = next`。

**复杂度**：时间 $O(n)$，空间 $O(1)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node { int val; Node* next; explicit Node(int v) : val(v), next(nullptr) {} };

Node* reverseList(Node* head) {
    Node* prev = nullptr;
    for (Node* cur = head; cur;) {
        Node* nxt = cur->next;   // 先保存后继，否则改指针后丢失
        cur->next = prev;        // 掉头
        prev = cur;
        cur = nxt;
    }
    return prev;                 // prev 指向原尾结点，即新头
}

int main() {
    Node* head = new Node(1);
    head->next = new Node(2);
    head->next->next = new Node(3);
    head->next->next->next = new Node(4);
    for (Node* p = reverseList(head); p; p = p->next) cout << p->val << ' ';
    cout << '\n';                // 4 3 2 1
    return 0;
}
```

### 3.2 合并两个有序链表

**题意**：给定两个升序单链表，把它们合并成一个新的升序链表。

**思路**：用哨兵结点简化边界，双指针依次比较两个链表当前结点，把较小的接到结果尾部，最后接上剩余部分。

**复杂度**：时间 $O(n+m)$，空间 $O(1)$（只改指针，不新建结点）。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node { int val; Node* next; explicit Node(int v) : val(v), next(nullptr) {} };

Node* mergeTwoLists(Node* a, Node* b) {
    Node dummy(0);                       // 哨兵
    Node* tail = &dummy;
    while (a && b) {
        if (a->val <= b->val) { tail->next = a; a = a->next; }
        else                  { tail->next = b; b = b->next; }
        tail = tail->next;
    }
    tail->next = a ? a : b;              // 接上未走完的那条链
    return dummy.next;
}

int main() {
    Node* a = new Node(1);  a->next = new Node(4);
    Node* b = new Node(2);  b->next = new Node(3);
    for (Node* p = mergeTwoLists(a, b); p; p = p->next) cout << p->val << ' ';
    cout << '\n';                        // 1 2 3 4
    return 0;
}
```

### 3.3 删除链表的倒数第 N 个结点

**题意**：给定链表，删除倒数第 $n$ 个结点，要求一次遍历完成。

**思路**：快慢指针。先让快指针走 $n+1$ 步（从哨兵出发），然后快慢同时前进；快指针到 `nullptr` 时，慢指针恰好指向倒数第 $n$ 个结点的**前驱**，改指针删掉即可。

**复杂度**：时间 $O(L)$，空间 $O(1)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node { int val; Node* next; explicit Node(int v) : val(v), next(nullptr) {} };

Node* removeNthFromEnd(Node* head, int n) {
    Node dummy(0);
    dummy.next = head;
    Node* fast = &dummy;
    Node* slow = &dummy;
    for (int i = 0; i <= n; ++i) fast = fast->next;   // fast 先走 n+1 步
    while (fast) { fast = fast->next; slow = slow->next; }
    slow->next = slow->next->next;                    // 删除 slow 的后继
    return dummy.next;
}

int main() {
    Node* head = new Node(1);
    head->next = new Node(2);
    head->next->next = new Node(3);
    head->next->next->next = new Node(4);
    head->next->next->next->next = new Node(5);
    for (Node* p = removeNthFromEnd(head, 2); p; p = p->next) cout << p->val << ' ';
    cout << '\n';                                     // 1 2 3 5
    return 0;
}
```

### 3.4 判断链表是否有环（快慢指针）

**题意**：判断单链表中是否存在环，若存在返回入环的第一个结点，否则返回 `nullptr`。

**思路**：Floyd 判圈。慢指针每次走 1 步，快指针每次走 2 步，相遇则有环。相遇后让一个指针回到头结点，两个指针都改走 1 步，再次相遇处就是入环点。

**正确性直觉**：设头到环入口距离 $a$，入口到相遇点距离 $b$，环长 $L$。相遇时慢指针走了 $a+b$，快指针走了 $a+b+kL$，而快指针路程是慢指针两倍：

$$
2(a+b) = a + b + kL \quad\Longrightarrow\quad a = (k-1)L + (L-b)
$$

也就是说，从相遇点再走 $a$ 步正好回到入口。

**复杂度**：时间 $O(n)$，空间 $O(1)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node { int val; Node* next; explicit Node(int v) : val(v), next(nullptr) {} };

Node* detectCycle(Node* head) {
    Node* slow = head;
    Node* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {                 // 相遇，说明有环
            Node* p = head;
            while (p != slow) { p = p->next; slow = slow->next; }
            return p;                       // 再次相遇即入口
        }
    }
    return nullptr;
}

int main() {
    Node* head = new Node(1);
    head->next = new Node(2);
    head->next->next = new Node(3);
    head->next->next->next = new Node(4);
    head->next->next->next->next = head->next;   // 4 -> 2，入环点为 2
    Node* entry = detectCycle(head);
    cout << (entry ? entry->val : -1) << '\n';   // 2
    return 0;
}
```

---

## 4. 易错点

1. **修改指针前不保存后继**：反转链表时若先改 `cur->next` 再取 `cur->next`，就会丢失后面的链，必须先保存 `nxt`。
2. **头结点特判遗漏**：在头部插入/删除时容易漏判，推荐统一使用哨兵结点。
3. **删除后未释放内存**：`delete` 被删结点，避免内存泄漏；数组模拟则无此问题。
4. **双链表只改一条边**：插入和删除都要同时维护 `pre` 与 `nxt`，少改一条会导致链断裂。
5. **循环链表死循环**：遍历终止条件写错（如 `while (p)`）会永远转圈，应用「回到起点」或「走 n 步」控制。
6. **快慢指针边界**：`fast->next->next` 前必须检查 `fast` 和 `fast->next` 非空。
7. **`std::list` 没有随机访问**：不能用 `lst[i]`，排序要用成员函数 `lst.sort()` 而不是 `std::sort`。
8. **数组模拟的 0 号下标**：通常把 0 当作空指针，真实结点从 1（双链表从 2）开始，混用会出错。
9. **多组数据未清空 `tot` 和 `head`**：数组模拟时每组数据都要重置 `tot`。

---

## 5. 练习建议

### 5.1 由易到难的路线

1. **基础操作**：用数组模拟单链表，实现头插、尾插、遍历、删除。
2. **经典三题**：反转链表、合并两个有序链表、删除倒数第 N 个结点。
3. **双指针技巧**：判环、找环入口、找中点、相交链表。
4. **双链表**：实现 LRU 缓存（哈希表 + 双链表）。
5. **链表排序**：归并排序链表（自顶向下 / 自底向上）。
6. **与其他结构结合**：链式前向星（图的存储）、邻接表、块状链表。
7. **进阶**：可持久化链表、跳舞链（DLX）。

### 5.2 推荐题目

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1996 约瑟夫问题 | 循环链表 |
| 基础 | LeetCode 206 反转链表 | 三指针 |
| 基础 | LeetCode 21 合并两个有序链表 | 哨兵 |
| 基础 | LeetCode 19 删除倒数第 N 个结点 | 快慢指针 |
| 基础 | LeetCode 141 / 142 环形链表 | Floyd 判圈 |
| 提高 | LeetCode 148 排序链表 | 归并排序 |
| 提高 | LeetCode 146 LRU 缓存 | 哈希 + 双链表 |
| 提高 | 洛谷 P1160 队列安排 | 数组模拟双链表 |

### 5.3 学习检查清单

- [ ] 能画出单链表、双链表、循环链表的结构图
- [ ] 能默写带哨兵的单链表插入与删除
- [ ] 能独立写出反转链表、合并链表、删除倒数第 N 个
- [ ] 理解 Floyd 判圈算法及入环点为什么这样求
- [ ] 能用数组模拟链表并说清 0 号下标的含义
- [ ] 知道 `std::list` 与 `vector` 的适用场景差异

> **核心思想**：链表用「指针串联离散结点」换取 $O(1)$ 的插入删除，代价是失去随机访问；哨兵结点、快慢指针、数组模拟是写好链表题的三大基本功。
