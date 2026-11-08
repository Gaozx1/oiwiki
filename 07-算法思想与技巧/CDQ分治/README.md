# CDQ 分治（CDQ Divide and Conquer）

> CDQ 分治 = **分治 + 排序**，用于处理**三维偏序**等「多维约束」问题。它把一维用分治消除，一维用排序消除，最后一维用树状数组维护。

---

## 目录

1. [从二维偏序说起](#1-从二维偏序说起)
2. [三维偏序（陌上花开）](#2-三维偏序陌上花开)
3. [CDQ 分治的流程](#3-cdq-分治的流程)
4. [模板代码](#4-模板代码)
5. [CDQ 的其他应用](#5-cdq-的其他应用)
6. [经典例题](#6-经典例题)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 从二维偏序说起

**问题**：给 $n$ 个点 $(a_i, b_i)$，求有多少对 $(i, j)$ 满足 $a_i < a_j$ 且 $b_i < b_j$。

**做法**：

1. 按 $a$ 排序（消除第一维）；
2. 对 $b$ 用树状数组求逆序对。

**复杂度**：$O(n \log n)$。

---

## 2. 三维偏序（陌上花开）

**问题**：给 $n$ 个元素，每个有 $(a_i, b_i, c_i)$，求对每个 $i$，满足 $a_j \le a_i$、$b_j \le b_i$、$c_j \le c_i$ 的 $j$ 的数量。

**思路**：

1. **第一维 $a$**：排序；
2. **第二维 $b$**：CDQ 分治——把区间分成左右两半，左边对右边的贡献用「按 $b$ 排序 + 双指针」计算；
3. **第三维 $c$**：树状数组维护。

**复杂度**：$O(n \log^2 n)$。

---

## 3. CDQ 分治的流程

```
solve(l, r):
    if l == r: return
    mid = (l + r) / 2
    solve(l, mid)                    // 递归左半
    solve(mid + 1, r)                // 递归右半
    处理左半对右半的贡献:
        把左半和右半分别按 b 排序
        双指针：对右半每个元素，把左半 b <= 右半 b 的元素插入树状数组
        查询 c 的前缀和
        清空树状数组
    按 b 归并排序（保持有序，方便上层）
```

**关键点**：

- 分治前按 $a$ 排序，分治时**不再改变 $a$ 的顺序**；
- 每层用归并排序维护 $b$ 有序，避免重复排序；
- 树状数组每次用完要清空。

---

## 4. 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100005;
const int MAXC = 200005;

int n, k;
int ans[N], cnt[N];

struct Node {
    int a, b, c, id, cnt;   // cnt: 相同元素个数（去重后）
    bool operator<(const Node& o) const {
        if (a != o.a) return a < o.a;
        if (b != o.b) return b < o.b;
        return c < o.c;
    }
} q[N], tmp[N];

int bit[MAXC];
void add(int i, int v) { for (; i <= k; i += i & -i) bit[i] += v; }
int sum(int i) { int s = 0; for (; i > 0; i -= i & -i) s += bit[i]; return s; }

void cdq(int l, int r) {
    if (l == r) return;
    int mid = (l + r) >> 1;
    cdq(l, mid);
    cdq(mid + 1, r);

    // 归并：左半按 b 插入树状数组
    int i = l, j = mid + 1, t = l;
    while (j <= r) {
        while (i <= mid && q[i].b <= q[j].b) {
            add(q[i].c, q[i].cnt);
            tmp[t++] = q[i++];
        }
        ans[q[j].id] += sum(q[j].c);
        tmp[t++] = q[j++];
    }
    while (i <= mid) tmp[t++] = q[i++];
    // 清空树状数组
    for (int p = l; p < t; ++p)
        if (q[p].id <= mid) add(q[p].c, -q[p].cnt);
    // 回填
    for (int p = l; p <= r; ++p) q[p] = tmp[p];
}

int main() {
    scanf("%d%d", &n, &k);
    vector<Node> a(n);
    for (int i = 0; i < n; ++i) {
        scanf("%d%d%d", &a[i].a, &a[i].b, &a[i].c);
        a[i].id = i;
    }
    sort(a.begin(), a.end());
    // 去重
    int m = 0;
    for (int i = 0; i < n; ++i) {
        if (i && a[i].a == a[i-1].a && a[i].b == a[i-1].b && a[i].c == a[i-1].c) {
            ++q[m].cnt;
        } else {
            q[++m] = a[i];
            q[m].cnt = 1;
            q[m].id = m;
        }
    }
    cdq(1, m);
    for (int i = 1; i <= m; ++i)
        ans[i] += q[i].cnt - 1;   // 自己不算
    // 统计每个答案出现次数
    for (int i = 1; i <= m; ++i) cnt[ans[i]] += q[i].cnt;
    for (int i = 0; i < n; ++i) printf("%d\n", cnt[i]);
    return 0;
}
```

> 注意：上面是简化版，`id` 与下标的处理需要仔细。完整版建议参考经典模板。

---

## 5. CDQ 的其他应用

### 5.1 三维偏序（模板）

如上。

### 5.2 带修改的区间查询

把修改和查询都看作操作，按时间 CDQ 分治。

### 5.3 动态逆序对

删除操作倒过来变成插入，CDQ 分治。

### 5.4 斜率优化 DP

CDQ 分治 + 凸包维护。

### 5.5 树上 CDQ

点分治的变体。

---

## 6. 经典例题

### 6.1 【模板】陌上花开（三维偏序）

**题意**：$n \le 10^5$，统计三维偏序。

**做法**：CDQ 分治 + 树状数组。

**复杂度**：$O(n \log^2 n)$。

### 6.2 动态逆序对

**题意**：给排列，每次删除一个数，求每次删除后的逆序对数。

**做法**：把删除倒序变成插入，CDQ 分治。

### 6.3 三维 LIS

**题意**：最长三维上升子序列。

**做法**：CDQ 分治 + DP。

### 6.4 偏序问题

**题意**：统计满足多种偏序关系的点对。

**做法**：CDQ 分治。

---

## 7. 易错点与练习

### 易错点

1. **去重**：相同元素要合并，否则会重复计数；
2. **树状数组清空**：每层用完要清空，否则影响上层；
3. **排序稳定性**：CDQ 分治要求左半部分在右半部分之前；
4. **递归顺序**：先 `solve(l, mid)`，再 `solve(mid+1, r)`，再处理跨区间贡献；
5. **`id` 对应**：去重后原编号与答案的对应关系要理清；
6. **边界**：$n=1$ 时直接返回。

### 练习建议

1. 先做二维偏序（排序 + 树状数组）；
2. 陌上花开（三维偏序模板）；
3. 动态逆序对；
4. CDQ + 斜率优化（进阶）。
