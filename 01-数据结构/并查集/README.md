# 并查集（Disjoint Set Union, DSU）

> 一句话概括：**并查集维护若干不相交集合，支持「合并两个集合」和「查询元素属于哪个集合」，配合路径压缩与按秩合并，单次操作几乎 $O(1)$。**

---

## 目录

1. [概念与原理](#1-概念与原理)
2. [模板代码](#2-模板代码)
3. [经典例题](#3-经典例题)
4. [易错点](#4-易错点)
5. [练习建议](#5-练习建议)

---

## 1. 概念与原理

### 1.1 并查集要解决什么问题

有一类问题只关心**元素之间的连通/等价关系**，而不关心具体如何连通：

- 动态加入若干条「$x$ 与 $y$ 属于同一类」的关系；
- 随时询问「$x$ 与 $y$ 是否属于同一类」；
- 统计当前一共有多少个不同的类。

例如：城市之间不断修路，问两个城市是否连通；朋友的朋友是朋友，问两人是否同一朋友圈。

如果每次合并都重新遍历所有元素打标记，复杂度会很高。并查集用**森林（每棵树代表一个集合）**表示这些关系，把「合并集合」变成「挂一棵树到另一棵树下」。

### 1.2 用「帮派」理解并查集

把每个集合想象成一个帮派，帮派里每个成员都**只记录自己的上级**，帮主（根结点）的上级记为自己。

```
帮派 A:          帮派 B:
   1(帮主)          5(帮主)
   |  \              |
   2   3             6
       |             |
       4             7
```

- **查根**：从某成员出发不断找上级，直到「上级是自己」的人，那个人就是帮主；
- **合并**：让一个帮派的帮主认另一个帮派的帮主做上级；
- **判断同帮**：两人分别找到帮主，帮主相同则同帮。

### 1.3 两个关键优化

**（1）路径压缩（Path Compression）**

查根时把路径上所有结点的父亲**直接指向根**：

```
查找 4 之前:                  查找 4 之后:
      1                            1
      |                          / | \
      3                         2  3  4
      |
      4
```

`4` 的查根路径是 $4 \to 3 \to 1$，压缩后 $4$ 和 $3$ 都直接指向根 $1$；本来就连在根上的 $2$ 不受影响。路径压缩是「按需」的，只在查询时发生。

**（2）按秩合并（Union by Rank / Size）**

合并时把**较矮的树挂到较高的树下面**，避免树退化成链。用 `rank[]` 记录树高的上界，或用 `size[]` 记录集合大小。

如果只用路径压缩、不用按秩合并，最坏是均摊 $O(\log n)$；两者都用，单次操作的均摊复杂度为

$$
O(\alpha(n))
$$

其中 $\alpha$ 是阿克曼函数的反函数，对 $n \le 10^{18}$ 都有 $\alpha(n) \le 4$，可视为常数。

### 1.4 文字示意图：一次完整合并

初始 5 个元素各自独立。第一步 `union(1,2)`，2 挂到 1 下：

```
   1
   |
   2
```

第二步 `union(3,4)`，4 挂到 3 下：

```
   3
   |
   4
```

第三步 `union(2,3)`：2 的根是 1，3 的根是 3，两边集合大小都是 2；按大小合并把 3 挂到 1 下面：

```
        1
       / \
      2   3
          |
          4

5 仍然是独立的一个集合
```

### 1.5 并查集的典型应用

| 应用 | 说明 |
|---|---|
| 连通性判定 | 无向图加边过程中动态判断两点是否连通 |
| Kruskal 最小生成树 | 用并查集判断加入一条边是否会成环 |
| 等价关系传递 | 亲戚关系、朋友关系、变量相等关系 |
| 维护集合信息 | 带权并查集、可撤销并查集 |
| 染色/分组 | 敌人关系（种类并查集 / 扩展域并查集） |
| 区间合并 | 把相交区间合并成若干连通块 |

### 1.6 扩展：带权并查集与可撤销并查集

**带权并查集**：除了「是否同类」，还要维护「$x$ 与父亲相差多少」这类**相对信息**：`dis[x]` 记录 $x$ 到父结点的权值，查根时顺便累加并压缩，合并时反推两根间权值。典型应用：食物链、区间和查询、模意义下的相对距离。

**可撤销并查集**：题目需要「撤销上一次合并」时，路径压缩会破坏旧结构，只能用**按秩合并 + 栈记录合并历史**，撤销时弹栈恢复。单次操作 $O(\log n)$。

---

## 2. 模板代码

### 2.1 基础并查集（路径压缩 + 按大小合并）

```cpp
#include <bits/stdc++.h>
using namespace std;

class DSU {
    vector<int> parent;   // parent[x]：x 的父亲
    vector<int> sz;       // sz[x]：以 x 为根的集合大小
    int comps;            // 连通块个数
public:
    explicit DSU(int n) : parent(n + 1), sz(n + 1, 1), comps(n) {
        for (int i = 1; i <= n; ++i) parent[i] = i;   // 初始各自为根
    }
    int find(int x) {                                  // 查根 + 路径压缩
        if (parent[x] == x) return x;
        return parent[x] = find(parent[x]);
    }
    bool unite(int x, int y) {                         // 返回是否真的合并
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;                    // 已在同一集合
        if (sz[rx] < sz[ry]) swap(rx, ry);             // 小树挂到大树下
        parent[ry] = rx;
        sz[rx] += sz[ry];
        --comps;
        return true;
    }
    bool same(int x, int y) { return find(x) == find(y); }
    int size(int x) { return sz[find(x)]; }
    int count() const { return comps; }
};

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    DSU dsu(n);
    while (m--) {
        int op, x, y;
        cin >> op >> x >> y;
        if (op == 1) dsu.unite(x, y);
        else cout << (dsu.same(x, y) ? 'Y' : 'N') << '\n';
    }
    return 0;
}
```

输入 `5 6 / 1 1 2 / 1 3 4 / 2 1 2 / 2 1 3 / 1 2 3 / 2 1 4`，输出 `Y / N / Y`。

### 2.2 迭代版查根（防止递归爆栈）

$n$ 达到 $10^6$ 且数据构造得当时，递归 `find` 可能爆栈。改成迭代 + 两次循环：

```cpp
int find(int x) {
    int root = x;
    while (parent[root] != root) root = parent[root];   // 第一次：找根
    while (parent[x] != root) {                         // 第二次：路径压缩
        int nxt = parent[x];
        parent[x] = root;
        x = nxt;
    }
    return root;
}
```

### 2.3 带权并查集（食物链）

用 `rel[x]` 表示 $x$ 与其父亲的关系（0 同类、1 吃、2 被吃）。路径压缩时关系按模 3 累加：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 50005;
int parent[MAXN];
int rel[MAXN];   // 0 同类，1 x 吃父亲，2 x 被父亲吃

int find(int x) {
    if (parent[x] == x) return x;
    int root = find(parent[x]);              // 先递归，保证 rel[parent] 已是到根的关系
    rel[x] = (rel[x] + rel[parent[x]]) % 3;  // 再累加
    return parent[x] = root;
}

bool unite(int x, int y, int d) {
    // d = 1：x 与 y 同类；d = 2：x 吃 y
    int rx = find(x), ry = find(y);
    if (rx == ry) {
        if (d == 1) return rel[x] == rel[y];
        return (rel[x] - rel[y] + 3) % 3 == 1;
    }
    parent[rx] = ry;
    rel[rx] = ((d - 1) + rel[y] - rel[x] + 3) % 3;   // 反解两根间关系
    return true;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) parent[i] = i;
    int ans = 0;
    while (m--) {
        int d, x, y;
        cin >> d >> x >> y;
        if (x > n || y > n || (d == 2 && x == y)) { ++ans; continue; }
        if (!unite(x, y, d)) ++ans;
    }
    cout << ans << '\n';
    return 0;
}
```

### 2.4 可撤销并查集（按秩合并，不路径压缩）

```cpp
struct UndoDSU {
    vector<int> parent, rnk, siz;
    vector<pair<int,int>> history;      // (被挂接的结点, 它的原根)
    explicit UndoDSU(int n) : parent(n + 1), rnk(n + 1, 0), siz(n + 1, 1) {
        for (int i = 1; i <= n; ++i) parent[i] = i;
    }
    int find(int x) const {             // 不能路径压缩！
        while (parent[x] != x) x = parent[x];
        return x;
    }
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rnk[rx] < rnk[ry]) swap(rx, ry);
        history.emplace_back(ry, rx);
        parent[ry] = rx;
        siz[rx] += siz[ry];
        if (rnk[rx] == rnk[ry]) ++rnk[rx];
        return true;
    }
    void undo() {
        pair<int,int> last = history.back();  history.pop_back();
        int child = last.first, root = last.second;
        parent[child] = child;
        siz[root] -= siz[child];
    }
};
```

---

## 3. 经典例题

### 3.1 亲戚（裸并查集）

**题意**：$n$ 个人、$m$ 条「$x$ 与 $y$ 是亲戚」的关系，$q$ 次询问两人是否亲戚。

**思路**：关系具有传递性，直接并查集合并，询问时比较根。**复杂度**：近似 $O((n+m+q)\alpha(n))$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int parent[5005];
int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m, q;
    cin >> n >> m >> q;
    for (int i = 1; i <= n; ++i) parent[i] = i;
    for (int i = 0; i < m; ++i) { int x, y; cin >> x >> y; parent[find(x)] = find(y); }
    while (q--) {
        int x, y;
        cin >> x >> y;
        cout << (find(x) == find(y) ? "Yes" : "No") << '\n';
    }
    return 0;
}
```

### 3.2 Kruskal 最小生成树

**题意**：给定无向连通图，求最小生成树的边权和；若图不连通输出 `-1`。

**思路**：边按权值升序排序，依次尝试加入；若两端点不在同一集合则选它并合并，否则跳过。选够 $n-1$ 条边即完成。**复杂度**：$O(m \log m)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Edge {
    int u, v, w;
    bool operator<(const Edge& o) const { return w < o.w; }
};
int parent[100005];
int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    vector<Edge> edges(m);
    for (auto& e : edges) cin >> e.u >> e.v >> e.w;
    sort(edges.begin(), edges.end());
    for (int i = 1; i <= n; ++i) parent[i] = i;

    long long ans = 0;
    int cnt = 0;
    for (const auto& e : edges) {
        int ru = find(e.u), rv = find(e.v);
        if (ru == rv) continue;            // 成环，跳过
        parent[ru] = rv;
        ans += e.w;
        if (++cnt == n - 1) break;
    }
    if (cnt < n - 1) cout << -1 << '\n';   // 图不连通
    else cout << ans << '\n';
    return 0;
}
```

### 3.3 食物链

**题意**：动物分 A、B、C 三类，A 吃 B、B 吃 C、C 吃 A。给出若干「$x$ 与 $y$ 同类」或「$x$ 吃 $y$」的陈述，求假话条数。

**思路**：带权并查集，维护「相对于根的关系」，代码见 2.3。三条判假规则：根相同但关系不符；编号越界或「自己吃自己」；其余为真话并合并。**复杂度**：$O((n+m)\alpha(n))$。

### 3.4 连通块数量

**题意**：无向图有 $n$ 个点和 $m$ 条边，求连通块数量。**思路**：初始 $n$ 个连通块，每成功合并一次减一。**复杂度**：$O((n+m)\alpha(n))$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int parent[100005];
int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }

int main() {
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) parent[i] = i;
    int comps = n;
    for (int i = 0; i < m; ++i) {
        int u, v;
        cin >> u >> v;
        int ru = find(u), rv = find(v);
        if (ru != rv) { parent[ru] = rv; --comps; }
    }
    cout << comps << '\n';
    return 0;
}
```

---

## 4. 易错点

1. **忘记初始化 `parent[i] = i`**：所有结点初始应各自为根。
2. **`find` 未加路径压缩**：`parent[x] = find(parent[x])` 中把结果赋回 `parent[x]` 才是压缩。
3. **合并前没判断是否同集合**：`unite` 中若先不 `find` 就直接改 `parent`，会破坏已有结构。
4. **按秩合并时交换的是根**：`swap(rx, ry)` 而不是 `swap(x, y)`。
5. **递归 `find` 爆栈**：链较长时递归深度可达 $O(n)$，$n=10^6$ 时改迭代版。
6. **带权并查集的更新顺序**：必须先递归查根，再用已更新的 `rel[parent[x]]` 累加；顺序写反会用到旧值。
7. **可撤销并查集误用路径压缩**：路径压缩改动大量结点，无法逐条撤销，只能按秩合并。
8. **多组数据未重置**：每组数据都要重新初始化数组。
9. **`size` / `rank` 只在根上有效**：非根结点的 `sz` 可能是过期值，必须 `sz[find(x)]`。

---

## 5. 练习建议

### 5.1 由易到难的路线

1. **模板题**：亲戚、连通块数量、朋友的朋友。
2. **图论结合**：Kruskal 最小生成树、判断无向图是否有环。
3. **离线倒序**：关闭网络、星球大战（删点转加点）。
4. **带权 / 扩展域**：食物链、奇偶游戏、敌人朋友（种类并查集）。
5. **集合信息维护**：维护集合大小、最大值、异或和。
6. **可撤销并查集**：线段树分治 + 可撤销并查集（动态连通性）。
7. **进阶**：并查集合并历史、Kruskal 重构树、树上并查集。

### 5.2 推荐题目

| 难度 | 题目 | 考点 |
|---|---|---|
| 入门 | 洛谷 P1551 亲戚 | 模板 |
| 入门 | 洛谷 P1536 村村通 | 连通块计数 |
| 基础 | 洛谷 P3367【模板】并查集 | 模板 |
| 基础 | 洛谷 P1111 修复公路 | 并查集 + 排序 |
| 提高 | 洛谷 P2024 食物链 | 带权并查集 |
| 提高 | 洛谷 P1197 星球大战 | 离线倒序 |
| 提高 | 洛谷 P1525 关押罪犯 | 种类并查集 / 二分图 |
| 进阶 | 洛谷 P2147 / P3200 | 可撤销并查集 + 线段树分治 |

### 5.3 学习检查清单

- [ ] 能画出「森林表示集合」的示意图
- [ ] 能默写 `find`（递归 + 迭代两种）与 `unite`
- [ ] 能解释路径压缩和按秩合并各自的作用与复杂度
- [ ] 能独立完成 Kruskal 最小生成树
- [ ] 理解带权并查集中 `rel[]` 的更新顺序
- [ ] 知道「删除转倒序加点」的离线技巧
- [ ] 知道可撤销并查集为什么不能路径压缩

> **核心思想**：并查集用「森林 + 只记父亲」表示等价关系，用路径压缩把查询摊平、用按秩合并把树高压住，两个优化叠加后单次操作近似常数——是所有「动态连通性」问题的基础工具。
