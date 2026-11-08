# AC 自动机（Aho-Corasick Automaton）

> AC 自动机 = **Trie 树 + KMP 的失配指针**。它能在 $O(\sum|P_i| + |S|)$ 时间内，找出所有模式串在文本串中的所有出现位置。

---

## 目录

1. [结构](#1-结构)
2. [失配指针（fail）](#2-失配指针fail)
3. [匹配过程](#3-匹配过程)
4. [模板代码](#4-模板代码)
5. [优化：fail 树](#5-优化fail-树)
6. [经典例题](#6-经典例题)
7. [易错点与练习](#7-易错点与练习)

---

## 1. 结构

AC 自动机由两部分组成：

1. **Trie 树**：把所有模式串插入 Trie；
2. **fail 指针**：每个节点指向「最长的、是当前节点表示串的后缀的、其他节点的前缀」。

**例子**：模式串 `he`、`she`、`his`、`hers`

```
        root
       /  |  \
      h   s   (h)
     /   / \
    e   h   h
   /   /     \
  (r?) i      e
       |
      (s)
```

（括号表示单词结尾）

---

## 2. 失配指针（fail）

**定义**：节点 $u$ 的 fail 指针指向满足「$v$ 表示的串是 $u$ 表示的串的**真后缀**」的**最长**节点 $v$。

**求法**：BFS 按层计算。

- 根节点的所有儿子 fail 指向根；
- 对于节点 $u$ 的儿子 $c$（字符为 `ch`）：
  - 令 $f = fail[u]$；
  - 若 $f$ 有 `ch` 儿子，则 $fail[c] = f$ 的 `ch` 儿子；
  - 否则 $f = fail[f]$，继续找；
  - 若一直找到根还没有，$fail[c] = root$。

```cpp
void buildFail() {
    queue<int> q;
    for (int c = 0; c < 26; ++c)
        if (trie[0].ch[c]) q.push(trie[0].ch[c]);

    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int c = 0; c < 26; ++c) {
            int v = trie[u].ch[c];
            if (v) {
                trie[v].fail = trie[trie[u].fail].ch[c];
                q.push(v);
            } else {
                trie[u].ch[c] = trie[trie[u].fail].ch[c];   // 路径压缩
            }
        }
    }
}
```

> 路径压缩（把不存在的儿子指向 fail 的对应儿子）让匹配时不用跳 fail，直接走。

---

## 3. 匹配过程

从根开始，沿文本串逐字符走：

```cpp
int u = 0;
for (char c : s) {
    u = trie[u].ch[c - 'a'];
    for (int v = u; v; v = trie[v].fail) {
        // 处理以 v 结尾的模式串
        if (trie[v].cnt) { /* 匹配到 */ }
    }
}
```

**复杂度**：如果没有路径压缩，某些情况会退化；加上路径压缩后是 $O(|S| + \text{匹配次数})$。

---

## 4. 模板代码

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1000005;

struct Node {
    int ch[26];
    int fail;
    int cnt;      // 以该节点结尾的模式串数量
    int id;       // 模式串编号
} trie[N];

int tot = 0;

void insert(const string& s, int id) {
    int u = 0;
    for (char c : s) {
        int x = c - 'a';
        if (!trie[u].ch[x]) trie[u].ch[x] = ++tot;
        u = trie[u].ch[x];
    }
    trie[u].cnt++;
    trie[u].id = id;
}

void buildFail() {
    queue<int> q;
    for (int c = 0; c < 26; ++c) {
        int v = trie[0].ch[c];
        if (v) { trie[v].fail = 0; q.push(v); }
    }
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int c = 0; c < 26; ++c) {
            int v = trie[u].ch[c];
            if (v) {
                trie[v].fail = trie[trie[u].fail].ch[c];
                q.push(v);
            } else {
                trie[u].ch[c] = trie[trie[u].fail].ch[c];
            }
        }
    }
}

int ans[N];

void query(const string& s) {
    int u = 0;
    for (char c : s) {
        u = trie[u].ch[c - 'a'];
        for (int v = u; v; v = trie[v].fail) {
            if (trie[v].cnt) ans[trie[v].id] += trie[v].cnt;
        }
    }
}

int main() {
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) {
        string s;
        cin >> s;
        insert(s, i);
    }
    buildFail();
    string t;
    cin >> t;
    query(t);
    for (int i = 1; i <= n; ++i) printf("%d\n", ans[i]);
    return 0;
}
```

---

## 5. 优化：fail 树

**问题**：上面的 `for (int v = u; v; v = trie[v].fail)` 在极端情况下会超时（每次匹配都跳 fail 链）。

**优化**：建 fail 树（fail 指针构成一棵树），匹配时只把访问的节点计数 +1，最后在 fail 树上做子树和。

```cpp
vector<int> failTree[N];

// 建树
for (int i = 1; i <= tot; ++i)
    failTree[trie[i].fail].push_back(i);

// 匹配时只标记
void query(const string& s) {
    int u = 0;
    for (char c : s) {
        u = trie[u].ch[c - 'a'];
        vis[u]++;
    }
}

// DFS 求子树和
void dfs(int u) {
    for (int v : failTree[u]) {
        dfs(v);
        vis[u] += vis[v];
    }
}
// 最后 trie[u].cnt * vis[u] 就是该模式串出现次数
```

**复杂度**：$O(\sum|P| \cdot |\Sigma| + |S| + \text{节点数})$。

---

## 6. 经典例题

### 6.1 【模板】AC 自动机

**题意**：$n$ 个模式串，一个文本串，统计每个模式串出现次数。

**做法**：AC 自动机 + fail 树。

### 6.2 单词计数

**题意**：统计每个单词在文章中出现的次数。

**做法**：同上。

### 6.3 病毒检测

**题意**：判断文本串是否包含任何模式串。

**做法**：AC 自动机，匹配到就返回。

### 6.4 最短独有子串

**题意**：求最短的不包含任何模式串的字符串。

**做法**：AC 自动机 + BFS。

### 6.5 AC 自动机 + DP

**题意**：构造长度为 $n$ 的字符串，不包含任何模式串，求方案数。

**做法**：AC 自动机上 DP。

---

## 7. 易错点与练习

### 易错点

1. **字符集**：假设小写字母，若字符集更大要用 `map` 或离散化；
2. **fail 指针初始**：根的 fail 指向自己（或 0）；
3. **路径压缩**：BFS 时同时修改 `trie[u].ch[c]`；
4. **重复模式串**：同一模式串出现多次要合并；
5. **fail 树**：建树后再 DFS，别在匹配时暴力跳；
6. **多组数据**：清空 trie 数组。

### 练习建议

1. AC 自动机模板；
2. 单词计数（fail 树优化）；
3. 病毒检测；
4. AC 自动机 + DP；
5. 对比 KMP（单模式）与 AC 自动机（多模式）。
