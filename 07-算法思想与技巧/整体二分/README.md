# 整体二分（Parallel Binary Search）

> 整体二分 = 把「多个询问各自二分」合并成「所有询问一起二分」。适用于**答案可二分**且**判定函数可以用统一的数据结构维护**的离线问题。

---

## 目录

1. [从单次二分说起](#1-从单次二分说起)
2. [整体二分的核心思想](#2-整体二分的核心思想)
3. [算法流程](#3-算法流程)
4. [模板代码](#4-模板代码)
5. [经典例题](#5-经典例题)
6. [复杂度分析](#6-复杂度分析)
7. [易错点](#7-易错点)
8. [练习建议](#8-练习建议)

---

## 1. 从单次二分说起

考虑这样一个问题：

> 给定一个序列 $a_1..a_n$ 和 $q$ 个询问，每次问「区间 $[l,r]$ 中第 $k$ 小的数是多少」。

**单次二分的做法**：二分答案 $mid$，统计区间内 $\le mid$ 的数的个数，若 $\ge k$ 则答案 $\le mid$，否则 $> mid$。

单次二分复杂度 $O(n \log V)$，$q$ 次就是 $O(qn\log V)$，太慢。

但注意：**每次二分的判定都是「统计区间内 $\le mid$ 的个数」**，可以用树状数组做。于是我们能对所有询问同时二分——这就是整体二分。

---

## 2. 整体二分的核心思想

整体二分把操作序列分成两类：

- **修改操作**：往数据结构里加入一个值（例如序列中的某个数）；
- **询问操作**：查询当前数据结构中满足条件的元素个数。

然后对答案值域 $[L, R]$ 二分：

1. 取 $mid = (L+R)/2$；
2. 把所有「值 $\le mid$」的修改操作执行掉（加进树状数组）；
3. 对每个询问，计算它当前区间内的贡献 $c$：
   - 若 $c \ge k$：答案在 $[L, mid]$，分到左半边；
   - 否则：$k \leftarrow k - c$，答案在 $[mid+1, R]$，分到右半边；
4. **撤销**刚才的修改（把树状数组清空）；
5. 递归处理左右两半。

**关键点**：每层递归中每个操作只被处理一次，总共 $O(\log V)$ 层。

---

## 3. 算法流程

```
solve(L, R, ops):
    if L == R:
        所有询问的答案都是 L
        return
    mid = (L + R) / 2
    把 ops 分成 leftOps, rightOps
    for op in ops:
        if op 是修改且 op.val <= mid:
            在数据结构上执行 op
            leftOps.push(op)
        else if op 是修改:
            rightOps.push(op)
        else if op 是询问:
            c = 查询 op 的区间贡献
            if c >= op.k:
                leftOps.push(op)
            else:
                op.k -= c
                rightOps.push(op)
    撤销所有本层执行的修改
    solve(L, mid, leftOps)
    solve(mid+1, R, rightOps)
```

---

## 4. 模板代码

### 4.1 带修区间第 k 小（整体二分）

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const int N = 200005;
const int INF = 0x3f3f3f3f;

int n, m;
int a[N], ans[N];
ll bit[N];

void add(int i, int v) { for (; i <= n; i += i & -i) bit[i] += v; }
ll sum(int i) { ll s = 0; for (; i > 0; i -= i & -i) s += bit[i]; return s; }
ll query(int l, int r) { return sum(r) - sum(l - 1); }

struct Op {
    int type;        // 1 = 修改, 2 = 询问
    int l, r, k;     // 询问: 区间 [l,r] 第 k 小
    int val;         // 修改: 插入的值
    int id;          // 询问编号
    int delta;       // 修改: +1 或 -1（用于带修改）
};

Op ops[N], tmpL[N], tmpR[N];

void solve(int L, int R, int ql, int qr) {
    if (ql > qr) return;
    if (L == R) {
        for (int i = ql; i <= qr; ++i)
            if (ops[i].type == 2) ans[ops[i].id] = L;
        return;
    }
    int mid = (L + R) >> 1;
    int cntL = 0, cntR = 0;
    for (int i = ql; i <= qr; ++i) {
        Op& o = ops[i];
        if (o.type == 1) {
            if (o.val <= mid) { add(o.l, o.delta); tmpL[cntL++] = o; }
            else tmpR[cntR++] = o;
        } else {
            ll c = query(o.l, o.r);
            if (c >= o.k) tmpL[cntL++] = o;
            else { o.k -= c; tmpR[cntR++] = o; }
        }
    }
    // 撤销
    for (int i = 0; i < cntL; ++i)
        if (tmpL[i].type == 1) add(tmpL[i].l, -tmpL[i].delta);
    // 回填
    for (int i = 0; i < cntL; ++i) ops[ql + i] = tmpL[i];
    for (int i = 0; i < cntR; ++i) ops[ql + cntL + i] = tmpR[i];
    solve(L, mid, ql, ql + cntL - 1);
    solve(mid + 1, R, ql + cntL, qr);
}

int main() {
    scanf("%d%d", &n, &m);
    int tot = 0;
    for (int i = 1; i <= n; ++i) {
        scanf("%d", &a[i]);
        ops[++tot] = {1, i, 0, 0, a[i], 0, 1};
    }
    for (int i = 1; i <= m; ++i) {
        int l, r, k;
        scanf("%d%d%d", &l, &r, &k);
        ops[++tot] = {2, l, r, k, 0, i, 0};
    }
    solve(1, INF, 1, tot);
    for (int i = 1; i <= m; ++i) printf("%d\n", ans[i]);
    return 0;
}
```

### 4.2 带修改版本

在 `main` 里把修改操作也当作 `type=1, delta=±1` 的操作插入即可：

```cpp
// 修改 a[pos] 为 newVal：
ops[++tot] = {1, pos, 0, 0, a[pos], 0, -1};   // 删除旧值
ops[++tot] = {1, pos, 0, 0, newVal, 0, 1};    // 加入新值
a[pos] = newVal;
```

---

## 5. 经典例题

### 5.1 【模板】静态区间第 k 小

**题意**：$n \le 2 \times 10^5$，$q \le 2 \times 10^5$，每次询问区间第 $k$ 小。

**思路**：把每个位置视为「权值为 $a_i$ 的修改操作」，询问二分判定。

**复杂度**：$O((n+q)\log n \log V)$。

### 5.2 带修改区间第 k 小

**题意**：在静态版本基础上支持单点修改。

**思路**：修改拆成「删除旧值 + 插入新值」两个操作，仍用整体二分。

**复杂度**：$O((n+q)\log n \log V)$。

### 5.3 第 k 小路径（树上）

**题意**：树上每次问路径第 $k$ 小。

**思路**：树上路径计数用树状数组维护到根的路径，整体二分套树剖。

### 5.4 区间第 k 大 / 中位数

把「第 $k$ 小」改成「第 $k$ 大」只需将 $k$ 反序；中位数是 $k = (r-l)/2+1$ 的特例。

---

## 6. 复杂度分析

- 递归深度：$O(\log V)$，其中 $V$ 是值域；
- 每层处理所有操作共 $O(n + q)$ 次，每次操作 $O(\log n)$（树状数组）；
- **总复杂度**：$O((n + q) \log V \log n)$。

对比单次二分 $O((n+q)\log V \log n)$ 其实一样？关键差别在 **每层每个操作只处理一次**，而单次二分每个询问都要重新扫一遍序列。整体二分避免了重复扫描。

---

## 7. 易错点

1. **忘记撤销**：每层结束时必须把树状数组清空，否则会影响其他分支；
2. **修改操作也要分左右**：值 $\le mid$ 的修改进左半边，其余进右半边；
3. **$k$ 要减掉左半边的贡献**：分到右半边时 `k -= c`；
4. **值域要离散化**：值域 $10^9$ 时先离散化，减少 $\log V$；
5. **询问和修改的顺序**：必须按输入顺序处理，保证修改在询问之前生效；
6. **递归边界**：`L == R` 时把所有询问的答案记录为 `L`。

---

## 8. 练习建议

1. **先练单次二分**：区间第 k 小用「二分 + 主席树」写一遍；
2. **静态整体二分**：套模板；
3. **带修改整体二分**：把修改拆成插入/删除；
4. **树上整体二分**：结合树剖或 DFS 序；
5. **对比主席树**：体会离线 vs 在线的取舍。

### 与主席树的对比

| 方法 | 是否在线 | 空间 | 代码难度 | 适用 |
|---|---|---|---|---|
| 主席树 | 在线 | $O(n \log n)$ | 中 | 静态区间第 k 小 |
| 整体二分 | 离线 | $O(n)$ | 中 | 带修、树上、复杂判定 |
| 树套树 | 在线 | $O(n \log^2 n)$ | 高 | 动态第 k 小 |
