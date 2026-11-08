# 模板速查（Template Cheatsheet）

> 竞赛常用模板代码合集。**建议背熟并能手写**，开考时直接敲进程序。

---

## 目录

1. [基础框架](#1-基础框架)
2. [快速幂与数论](#2-快速幂与数论)
3. [并查集](#3-并查集)
4. [树状数组](#4-树状数组)
5. [线段树](#5-线段树)
6. [ST 表](#6-st-表)
7. [堆与优先队列](#7-堆与优先队列)
8. [图的存储与最短路](#8-图的存储与最短路)
9. [最小生成树](#9-最小生成树)
10. [拓扑排序与强连通分量](#10-拓扑排序与强连通分量)
11. [LCA 与树剖](#11-lca-与树剖)
12. [字符串哈希与 KMP](#12-字符串哈希与-kmp)
13. [单调栈与单调队列](#13-单调栈与单调队列)
14. [二分与三分](#14-二分与三分)
15. [常用小技巧](#15-常用小技巧)

---

## 1. 基础框架

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef pair<int, int> pii;
#define rep(i, a, b) for (int i = (a); i <= (b); ++i)
#define rrep(i, a, b) for (int i = (a); i >= (b); --i)
#define all(x) (x).begin(), (x).end()

const int N = 100005;
const int INF = 0x3f3f3f3f;
const ll LINF = 0x3f3f3f3f3f3f3f3fLL;
const double EPS = 1e-9;
const int MOD = 1000000007;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    // freopen("xxx.in", "r", stdin);
    // freopen("xxx.out", "w", stdout);

    return 0;
}
```

---

## 2. 快速幂与数论

### 2.1 快速幂

```cpp
ll qpow(ll a, ll b, ll mod) {
    ll res = 1 % mod;
    a %= mod;
    while (b) {
        if (b & 1) res = res * a % mod;
        a = a * a % mod;
        b >>= 1;
    }
    return res;
}
```

### 2.2 扩展欧几里得

```cpp
ll exgcd(ll a, ll b, ll& x, ll& y) {
    if (!b) { x = 1; y = 0; return a; }
    ll d = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return d;
}
```

### 2.3 乘法逆元

```cpp
// 模数为质数
ll inv(ll a, ll p) { return qpow(a, p - 2, p); }

// 任意互质
ll inv2(ll a, ll p) {
    ll x, y;
    exgcd(a, p, x, y);
    return (x % p + p) % p;
}
```

### 2.4 线性筛

```cpp
const int N = 10000005;
int primes[N], cnt = 0;
bool isComp[N];

void sieve(int n) {
    isComp[1] = true;
    for (int i = 2; i <= n; ++i) {
        if (!isComp[i]) primes[++cnt] = i;
        for (int j = 1; j <= cnt && (ll)i * primes[j] <= n; ++j) {
            isComp[i * primes[j]] = true;
            if (i % primes[j] == 0) break;
        }
    }
}
```

### 2.5 组合数（预处理阶乘）

```cpp
const int N = 1000005;
ll fac[N], ifac[N];

void initC(int n, ll p) {
    fac[0] = 1;
    for (int i = 1; i <= n; ++i) fac[i] = fac[i-1] * i % p;
    ifac[n] = qpow(fac[n], p - 2, p);
    for (int i = n - 1; i >= 0; --i) ifac[i] = ifac[i+1] * (i + 1) % p;
}

ll C(int n, int m, ll p) {
    if (m < 0 || m > n) return 0;
    return fac[n] * ifac[m] % p * ifac[n-m] % p;
}
```

---

## 3. 并查集

```cpp
int fa[N], rnk[N];

void init(int n) {
    for (int i = 1; i <= n; ++i) fa[i] = i, rnk[i] = 1;
}

int find(int x) {
    return fa[x] == x ? x : fa[x] = find(fa[x]);
}

bool merge(int x, int y) {
    x = find(x); y = find(y);
    if (x == y) return false;
    if (rnk[x] < rnk[y]) swap(x, y);
    fa[y] = x;
    if (rnk[x] == rnk[y]) ++rnk[x];
    return true;
}
```

---

## 4. 树状数组

### 4.1 单点修改 + 区间查询

```cpp
ll bit[N];
int n;

void add(int i, ll v) {
    for (; i <= n; i += i & -i) bit[i] += v;
}

ll sum(int i) {
    ll s = 0;
    for (; i > 0; i -= i & -i) s += bit[i];
    return s;
}

ll query(int l, int r) { return sum(r) - sum(l - 1); }
```

### 4.2 区间修改 + 单点查询（差分）

```cpp
void rangeAdd(int l, int r, ll v) {
    add(l, v);
    add(r + 1, -v);
}
ll pointQuery(int i) { return sum(i); }
```

### 4.3 求第 k 小

```cpp
int kth(int k) {          // 返回最小的 i 使 sum(i) >= k
    int pos = 0;
    for (int i = 20; i >= 0; --i) {
        int nxt = pos + (1 << i);
        if (nxt <= n && bit[nxt] < k) {
            k -= bit[nxt];
            pos = nxt;
        }
    }
    return pos + 1;
}
```

---

## 5. 线段树

### 5.1 区间加 + 区间求和

```cpp
struct Node {
    ll sum, tag;
} tr[N << 2];

void pushup(int p) { tr[p].sum = tr[p<<1].sum + tr[p<<1|1].sum; }

void pushdown(int p, int l, int r) {
    if (!tr[p].tag) return;
    int mid = (l + r) >> 1;
    tr[p<<1].sum += tr[p].tag * (mid - l + 1);
    tr[p<<1].tag += tr[p].tag;
    tr[p<<1|1].sum += tr[p].tag * (r - mid);
    tr[p<<1|1].tag += tr[p].tag;
    tr[p].tag = 0;
}

void build(int p, int l, int r, ll a[]) {
    if (l == r) { tr[p].sum = a[l]; return; }
    int mid = (l + r) >> 1;
    build(p<<1, l, mid, a);
    build(p<<1|1, mid+1, r, a);
    pushup(p);
}

void update(int p, int l, int r, int ql, int qr, ll v) {
    if (ql <= l && r <= qr) {
        tr[p].sum += v * (r - l + 1);
        tr[p].tag += v;
        return;
    }
    pushdown(p, l, r);
    int mid = (l + r) >> 1;
    if (ql <= mid) update(p<<1, l, mid, ql, qr, v);
    if (qr > mid)  update(p<<1|1, mid+1, r, ql, qr, v);
    pushup(p);
}

ll query(int p, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) return tr[p].sum;
    pushdown(p, l, r);
    int mid = (l + r) >> 1;
    ll res = 0;
    if (ql <= mid) res += query(p<<1, l, mid, ql, qr);
    if (qr > mid)  res += query(p<<1|1, mid+1, r, ql, qr);
    return res;
}
```

### 5.2 区间加 + 区间最大值

```cpp
struct Node { ll mx, tag; } tr[N << 2];
void pushup(int p) { tr[p].mx = max(tr[p<<1].mx, tr[p<<1|1].mx); }
void pushdown(int p) {
    if (!tr[p].tag) return;
    tr[p<<1].mx += tr[p].tag;  tr[p<<1].tag += tr[p].tag;
    tr[p<<1|1].mx += tr[p].tag; tr[p<<1|1].tag += tr[p].tag;
    tr[p].tag = 0;
}
```

---

## 6. ST 表

```cpp
int st[20][N], lg[N];

void buildST(int n, int a[]) {
    lg[1] = 0;
    for (int i = 2; i <= n; ++i) lg[i] = lg[i >> 1] + 1;
    for (int i = 1; i <= n; ++i) st[0][i] = a[i];
    for (int k = 1; (1 << k) <= n; ++k)
        for (int i = 1; i + (1 << k) - 1 <= n; ++i)
            st[k][i] = max(st[k-1][i], st[k-1][i + (1 << (k-1))]);
}

int queryMax(int l, int r) {
    int k = lg[r - l + 1];
    return max(st[k][l], st[k][r - (1 << k) + 1]);
}
```

---

## 7. 堆与优先队列

```cpp
// 大根堆
priority_queue<int> maxq;
// 小根堆
priority_queue<int, vector<int>, greater<int>> minq;
// 结构体小根堆
struct Node { int d, id; };
struct Cmp { bool operator()(const Node& a, const Node& b) const { return a.d > b.d; } };
priority_queue<Node, vector<Node>, Cmp> pq;
```

---

## 8. 图的存储与最短路

### 8.1 链式前向星

```cpp
int head[N], to[M], nxt[M], w[M], ecnt = 0;

void addEdge(int u, int v, int c) {
    to[++ecnt] = v; w[ecnt] = c; nxt[ecnt] = head[u]; head[u] = ecnt;
}
```

### 8.2 Dijkstra（堆优化）

```cpp
ll dist[N];
bool vis[N];

void dijkstra(int s, int n) {
    for (int i = 1; i <= n; ++i) dist[i] = LINF, vis[i] = false;
    priority_queue<pair<ll,int>, vector<pair<ll,int>>, greater<pair<ll,int>>> pq;
    dist[s] = 0;
    pq.push({0, s});
    while (!pq.empty()) {
        int u = pq.top().second; pq.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (int e = head[u]; e; e = nxt[e]) {
            int v = to[e];
            if (dist[v] > dist[u] + w[e]) {
                dist[v] = dist[u] + w[e];
                pq.push({dist[v], v});
            }
        }
    }
}
```

### 8.3 Floyd

```cpp
ll d[N][N];
void floyd(int n) {
    for (int k = 1; k <= n; ++k)
        for (int i = 1; i <= n; ++i)
            for (int j = 1; j <= n; ++j)
                if (d[i][j] > d[i][k] + d[k][j])
                    d[i][j] = d[i][k] + d[k][j];
}
```

### 8.4 Bellman-Ford / 判负环

```cpp
ll dist[N];
bool bellman(int s, int n, int m) {   // 返回是否无负环
    for (int i = 1; i <= n; ++i) dist[i] = LINF;
    dist[s] = 0;
    for (int i = 1; i < n; ++i) {
        bool flag = false;
        for (int j = 1; j <= m; ++j)
            if (dist[u[j]] < LINF && dist[v[j]] > dist[u[j]] + w[j]) {
                dist[v[j]] = dist[u[j]] + w[j];
                flag = true;
            }
        if (!flag) break;
    }
    for (int j = 1; j <= m; ++j)
        if (dist[u[j]] < LINF && dist[v[j]] > dist[u[j]] + w[j]) return false;
    return true;
}
```

---

## 9. 最小生成树

```cpp
struct Edge { int u, v, w; };
Edge e[M];

bool cmp(const Edge& a, const Edge& b) { return a.w < b.w; }

ll kruskal(int n, int m) {
    sort(e + 1, e + m + 1, cmp);
    init(n);                     // 并查集初始化
    ll ans = 0; int cnt = 0;
    for (int i = 1; i <= m; ++i) {
        if (merge(e[i].u, e[i].v)) {
            ans += e[i].w;
            if (++cnt == n - 1) break;
        }
    }
    return cnt == n - 1 ? ans : -1;
}
```

---

## 10. 拓扑排序与强连通分量

### 10.1 拓扑排序

```cpp
int indeg[N], q[N];

bool topo(int n) {
    int hh = 0, tt = -1;
    for (int i = 1; i <= n; ++i) if (!indeg[i]) q[++tt] = i;
    while (hh <= tt) {
        int u = q[hh++];
        for (int e = head[u]; e; e = nxt[e])
            if (--indeg[to[e]] == 0) q[++tt] = to[e];
    }
    return tt == n - 1;
}
```

### 10.2 Tarjan 求强连通分量

```cpp
int dfn[N], low[N], stk[N], scc[N], top = 0, timer = 0, sccCnt = 0;
bool inStk[N];

void tarjan(int u) {
    dfn[u] = low[u] = ++timer;
    stk[++top] = u; inStk[u] = true;
    for (int e = head[u]; e; e = nxt[e]) {
        int v = to[e];
        if (!dfn[v]) { tarjan(v); low[u] = min(low[u], low[v]); }
        else if (inStk[v]) low[u] = min(low[u], dfn[v]);
    }
    if (dfn[u] == low[u]) {
        ++sccCnt;
        int v;
        do {
            v = stk[top--];
            inStk[v] = false;
            scc[v] = sccCnt;
        } while (v != u);
    }
}
```

---

## 11. LCA 与树剖

### 11.1 倍增 LCA

```cpp
int dep[N], up[20][N];

void dfs(int u, int p) {
    dep[u] = dep[p] + 1;
    up[0][u] = p;
    for (int k = 1; k < 20; ++k) up[k][u] = up[k-1][up[k-1][u]];
    for (int e = head[u]; e; e = nxt[e])
        if (to[e] != p) dfs(to[e], u);
}

int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);
    int d = dep[u] - dep[v];
    for (int k = 0; k < 20; ++k)
        if (d >> k & 1) u = up[k][u];
    if (u == v) return u;
    for (int k = 19; k >= 0; --k)
        if (up[k][u] != up[k][v]) { u = up[k][u]; v = up[k][v]; }
    return up[0][u];
}
```

### 11.2 树链剖分

```cpp
int fa[N], dep2[N], sz[N], son[N], top2[N], dfn2[N], rk[N], timer2 = 0;

void dfs1(int u) {
    sz[u] = 1; son[u] = 0;
    for (int e = head[u]; e; e = nxt[e]) {
        int v = to[e];
        if (v == fa[u]) continue;
        fa[v] = u; dep2[v] = dep2[u] + 1;
        dfs1(v);
        sz[u] += sz[v];
        if (sz[v] > sz[son[u]]) son[u] = v;
    }
}

void dfs2(int u, int tp) {
    top2[u] = tp; dfn2[u] = ++timer2; rk[timer2] = u;
    if (son[u]) dfs2(son[u], tp);
    for (int e = head[u]; e; e = nxt[e])
        if (to[e] != fa[u] && to[e] != son[u]) dfs2(to[e], to[e]);
}

// 路径查询（配合线段树）
ll pathQuery(int u, int v) {
    ll res = 0;
    while (top2[u] != top2[v]) {
        if (dep2[top2[u]] < dep2[top2[v]]) swap(u, v);
        res += query(1, 1, n, dfn2[top2[u]], dfn2[u]);
        u = fa[top2[u]];
    }
    if (dep2[u] > dep2[v]) swap(u, v);
    res += query(1, 1, n, dfn2[u], dfn2[v]);
    return res;
}
```

---

## 12. 字符串哈希与 KMP

### 12.1 字符串哈希

```cpp
typedef unsigned long long ull;
const ull BASE = 131;
ull h[N], pw[N];

void initHash(const string& s) {
    int n = s.size();
    pw[0] = 1;
    h[0] = 0;
    for (int i = 1; i <= n; ++i) {
        h[i] = h[i-1] * BASE + s[i-1];
        pw[i] = pw[i-1] * BASE;
    }
}

ull getHash(int l, int r) {          // 1-indexed，闭区间
    return h[r] - h[l-1] * pw[r - l + 1];
}
```

### 12.2 KMP

```cpp
int nxtArr[N];

void getNext(const string& p) {
    int m = p.size();
    nxtArr[0] = -1;
    for (int i = 1, j = -1; i < m; ++i) {
        while (j >= 0 && p[i] != p[j+1]) j = nxtArr[j];
        if (p[i] == p[j+1]) ++j;
        nxtArr[i] = j;
    }
}

void kmp(const string& s, const string& p) {
    int n = s.size(), m = p.size();
    getNext(p);
    for (int i = 0, j = -1; i < n; ++i) {
        while (j >= 0 && s[i] != p[j+1]) j = nxtArr[j];
        if (s[i] == p[j+1]) ++j;
        if (j == m - 1) {
            cout << i - m + 2 << "\n";   // 匹配位置（1-indexed）
            j = nxtArr[j];
        }
    }
}
```

---

## 13. 单调栈与单调队列

### 13.1 单调栈（下一个更大元素）

```cpp
int a[N], nxtGreater[N], stk[N], top = 0;

void nextGreater(int n) {
    for (int i = n; i >= 1; --i) {
        while (top && a[stk[top]] <= a[i]) --top;
        nxtGreater[i] = top ? stk[top] : 0;
        stk[++top] = i;
    }
}
```

### 13.2 单调队列（滑动窗口最大值）

```cpp
int q[N], a[N];

void solve(int n, int k) {
    int hh = 0, tt = -1;
    for (int i = 1; i <= n; ++i) {
        while (hh <= tt && q[hh] <= i - k) ++hh;      // 弹出过期
        while (hh <= tt && a[q[tt]] <= a[i]) --tt;    // 维护单调
        q[++tt] = i;
        if (i >= k) printf("%d ", a[q[hh]]);
    }
}
```

---

## 14. 二分与三分

### 14.1 整数二分

```cpp
// 找第一个 >= x 的位置
int lowerBound(int a[], int n, int x) {
    int l = 1, r = n + 1;
    while (l < r) {
        int mid = l + (r - l) / 2;
        if (a[mid] >= x) r = mid;
        else l = mid + 1;
    }
    return l;
}

// 找最后一个 <= x 的位置
int upperBound(int a[], int n, int x) {
    int l = 0, r = n;
    while (l < r) {
        int mid = l + (r - l + 1) / 2;
        if (a[mid] <= x) l = mid;
        else r = mid - 1;
    }
    return l;
}
```

### 14.2 浮点二分

```cpp
double binarySearch(double l, double r) {
    for (int it = 0; it < 100; ++it) {
        double mid = (l + r) / 2;
        if (check(mid)) r = mid;
        else l = mid;
    }
    return l;
}
```

### 14.3 三分（单峰函数）

```cpp
double ternarySearch(double l, double r) {
    for (int it = 0; it < 200; ++it) {
        double m1 = l + (r - l) / 3;
        double m2 = r - (r - l) / 3;
        if (f(m1) < f(m2)) l = m1;
        else r = m2;
    }
    return (l + r) / 2;
}
```

---

## 15. 常用小技巧

### 15.1 离散化

```cpp
vector<int> xs;
// xs 存所有需要的值
sort(xs.begin(), xs.end());
xs.erase(unique(xs.begin(), xs.end()), xs.end());
int id = lower_bound(xs.begin(), xs.end(), x) - xs.begin() + 1;
```

### 15.2 随机数

```cpp
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
int rnd(int l, int r) { return uniform_int_distribution<int>(l, r)(rng); }
```

### 15.3 字符串分割

```cpp
vector<string> split(const string& s, char delim) {
    vector<string> res;
    string cur;
    istringstream iss(s);
    while (getline(iss, cur, delim)) res.push_back(cur);
    return res;
}
```

### 15.4 位运算常用

```cpp
x & -x        // lowbit
x >> i & 1    // 第 i 位
x | (1 << i)  // 置位
x & ~(1 << i) // 清位
__builtin_popcount(x)   // 二进制中 1 的个数
__builtin_ctz(x)        // 末尾 0 的个数
__builtin_clz(x)        // 前导 0 的个数
```

### 15.5 取模

```cpp
inline int add(int a, int b, int p) { return (a + b >= p) ? (a + b - p) : (a + b); }
inline int sub(int a, int b, int p) { return (a < b) ? (a - b + p) : (a - b); }
inline int mul(int a, int b, int p) { return (ll)a * b % p; }
```

### 15.6 邻接矩阵转邻接表

```cpp
for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= n; ++j)
        if (g[i][j] != INF) addEdge(i, j, g[i][j]);
```

### 15.7 最大公约数 / 最小公倍数

```cpp
ll gcd(ll a, ll b) { return b ? gcd(b, a % b) : a; }
ll lcm(ll a, ll b) { return a / gcd(a, b) * b; }   // 防溢出
```

### 15.8 读入一行

```cpp
string s;
getline(cin >> ws, s);   // 跳过前导空白
```

### 15.9 结构体排序

```cpp
struct Node { int a, b; };
bool operator<(const Node& x, const Node& y) {
    if (x.a != y.a) return x.a < y.a;
    return x.b < y.b;
}
sort(v.begin(), v.end());
```

### 15.10 大整数（Python 可用时）

```python
print(a + b)      # 任意精度
```

---

## 附：模板背诵优先级

| 优先级 | 模板 | 出现频率 |
|---|---|---|
| ⭐⭐⭐ | 快速幂、并查集、最短路、排序 | 几乎每题 |
| ⭐⭐⭐ | 线段树（区间加+区间和） | 高 |
| ⭐⭐ | 树状数组、ST 表、LCA | 中高 |
| ⭐⭐ | KMP、字符串哈希 | 中 |
| ⭐ | 树剖、Tarjan、网络流 | 中低 |
| ⭐ | 平衡树、多项式 | 低（但考到就是压轴） |
---

## 附：经典案例与易错点

### 经典案例

**案例 1：模板题验证**
拿到任何模板后，先用题目样例跑一遍，再用暴力对拍小数据。模板本身正确不代表你的**调用方式**正确。

**案例 2：模板组合**
一道题往往同时用到多个模板（如「树剖 + 线段树」）。要注意：
- 各模板的下标体系是否一致（0 起始 vs 1 起始）；
- 全局数组是否冲突（重名变量）；
- 多组数据是否所有结构都清空。

**案例 3：模板定制**
出现以下情况时必须改模板：
- 需要区间最大值而不是区间和；
- 需要取模；
- 需要支持负数；
- 需要多组数据。

### 易错点

1. **下标体系混用**：0 起始与 1 起始混写是最常见的错误来源。
2. **数组开小**：线段树要开 `4n`，ST 表要开 `log n` 层。
3. **忘记取模**：乘法与累加都可能溢出或超范围。
4. **多组数据不清空**：全局数组必须重置。
5. **复制模板不改常量**：`N`、`MOD`、`LOG` 要按题目范围调整。
6. **模板中用了 `using namespace std` 的变量名**：如 `next`、`rank`、`y1` 会与标准库冲突。