# 排序算法

> 一句话概括：**排序是把无序数据整理成有序序列的过程，既是 $O(n\log n)$ 世界的入口，也是二分、双指针、贪心等算法的前置条件。**

---

## 目录

1. [为什么排序如此重要](#1-为什么排序如此重要)
2. [排序的稳定性与比较](#2-排序的稳定性与比较)
3. [冒泡、选择、插入排序](#3-冒泡选择插入排序)
4. [归并排序](#4-归并排序)
5. [快速排序](#5-快速排序)
6. [堆排序与计数排序](#6-堆排序与计数排序)
7. [STL 的 sort](#7-stl-的-sort)
8. [模板代码](#8-模板代码)
9. [经典例题](#9-经典例题)
10. [易错点](#10-易错点)
11. [练习建议](#11-练习建议)

---

## 1. 为什么排序如此重要

排序能把「无序」变成「有序」，而有序带来一系列强大的能力：**二分查找**（$O(\log n)$ 查值）、**双指针**（两数之和从 $O(n^2)$ 降到 $O(n)$）、**贪心**（按结束时间选活动）、**去重**（相同元素相邻）、**取第 k 小**（排序后直接取）。一句话：**排序是性价比最高的预处理。**

排序算法分两类：比较排序（通过比较元素大小）与非比较排序（利用值的范围，如计数排序、基数排序、桶排序）。

---

## 2. 排序的稳定性与比较

### 2.1 稳定性

若排序前 $a$ 在 $b$ 前面且 $a = b$，排序后 $a$ 仍在 $b$ 前面，则称该排序**稳定**。

| 算法 | 稳定性 | 平均时间 | 最坏时间 | 空间 |
|---|---|---|---|---|
| 冒泡排序 | 稳定 | $O(n^2)$ | $O(n^2)$ | $O(1)$ |
| 选择排序 | 不稳定 | $O(n^2)$ | $O(n^2)$ | $O(1)$ |
| 插入排序 | 稳定 | $O(n^2)$ | $O(n^2)$ | $O(1)$ |
| 归并排序 | 稳定 | $O(n\log n)$ | $O(n\log n)$ | $O(n)$ |
| 快速排序 | 不稳定 | $O(n\log n)$ | $O(n^2)$ | $O(\log n)$ |
| 堆排序 | 不稳定 | $O(n\log n)$ | $O(n\log n)$ | $O(1)$ |
| 计数排序 | 稳定 | $O(n+V)$ | $O(n+V)$ | $O(V)$ |

**为什么需要稳定性？** 多关键字排序时：先按第二关键字排，再按第一关键字稳定排序，结果就是「第一关键字优先、第二关键字次之」。

### 2.2 比较函数与严格弱序

```cpp
bool cmp(const Node& a, const Node& b) {
    if (a.x != b.x) return a.x < b.x;
    return a.y < b.y;
}
```

必须满足**不可自反**（`cmp(a,a) == false`）、**反对称**、**传递**。写成 `a.x <= b.x` 就违反了第一条，`std::sort` 可能越界崩溃。

---

## 3. 冒泡、选择、插入排序

### 3.1 冒泡排序

相邻两元素比较，逆序就交换，每一轮把当前最大值「冒」到末尾：

```
初始: 5 3 8 1
第1轮: 3 5 1 [8]
第2轮: 3 1 [5 8]
第3轮: 1 [3 5 8]
```

```cpp
void bubbleSort(vector<int>& a) {
    int n = a.size();
    for (int i = 0; i < n - 1; ++i) {
        bool swapped = false;
        for (int j = 0; j < n - 1 - i; ++j)
            if (a[j] > a[j + 1]) { swap(a[j], a[j + 1]); swapped = true; }
        if (!swapped) break;      // 已有序，提前结束
    }
}
```

平均与最坏 $O(n^2)$，最好 $O(n)$（已有序），空间 $O(1)$，稳定。

### 3.2 选择排序

每轮从未排序部分选出最小值，放到已排序部分末尾：

```cpp
void selectionSort(vector<int>& a) {
    int n = a.size();
    for (int i = 0; i < n - 1; ++i) {
        int minPos = i;
        for (int j = i + 1; j < n; ++j)
            if (a[j] < a[minPos]) minPos = j;
        if (minPos != i) swap(a[i], a[minPos]);
    }
}
```

比较次数固定，无论数据如何都是 $O(n^2)$，空间 $O(1)$，**不稳定**（交换会打乱相等元素顺序）。

### 3.3 插入排序

维护已排序前缀，每次把下一个元素插入合适位置：

```cpp
void insertionSort(vector<int>& a) {
    int n = a.size();
    for (int i = 1; i < n; ++i) {
        int key = a[i], j = i - 1;
        while (j >= 0 && a[j] > key) {   // 严格大于，保证稳定
            a[j + 1] = a[j];
            --j;
        }
        a[j + 1] = key;
    }
}
```

对**近乎有序**的数据极快（最好 $O(n)$），常数小，是小数组排序的常用选择（`std::sort` 在小段上就退化为插入排序）。空间 $O(1)$，稳定。

---

## 4. 归并排序

**分治**：把数组分成两半分别排序，再合并两个有序数组。

```
        [5 3 8 1]
       /         \
   [5 3]        [8 1]
   /   \        /   \
 [5]  [3]     [8]  [1]
   \   /        \   /
  [3 5]        [1 8]
       \        /
      [1 3 5 8]
```

```cpp
void mergeSort(vector<int>& a, int l, int r, vector<int>& tmp) {
    if (l >= r) return;
    int mid = (l + r) >> 1;
    mergeSort(a, l, mid, tmp);
    mergeSort(a, mid + 1, r, tmp);

    int i = l, j = mid + 1, k = l;
    while (i <= mid && j <= r) {
        if (a[i] <= a[j]) tmp[k++] = a[i++];   // <= 保证稳定
        else              tmp[k++] = a[j++];
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r)   tmp[k++] = a[j++];
    for (int p = l; p <= r; ++p) a[p] = tmp[p];
}
```

时间 $O(n\log n)$（每层合并 $O(n)$，共 $\log n$ 层），空间 $O(n)$，稳定。写成递归式就是

$$T(n) = 2T\left(\frac{n}{2}\right) + O(n) = O(n\log n)$$

应用：求逆序对、外部排序。

---

## 5. 快速排序

**思想**：选基准值 `pivot`，把数组分成「小于 pivot」和「大于 pivot」两部分，再递归排序两边。

```cpp
void quickSort(vector<int>& a, int l, int r) {
    if (l >= r) return;
    int pivot = a[(l + r) >> 1];      // 取中间元素作为基准
    int i = l, j = r;
    while (i <= j) {
        while (a[i] < pivot) ++i;
        while (a[j] > pivot) --j;
        if (i <= j) { swap(a[i], a[j]); ++i; --j; }
    }
    quickSort(a, l, j);
    quickSort(a, i, r);
}
```

平均 $O(n\log n)$；最坏 $O(n^2)$，发生在每次 pivot 都取到最值时（如已排序数组取首元素）。优化：随机选 pivot、三数取中、小区间用插入排序。

---

## 6. 堆排序与计数排序

### 6.1 堆排序

建大根堆，每次把堆顶（最大值）与末尾交换，再对剩余部分下沉：

```cpp
void siftDown(vector<int>& a, int n, int i) {
    while (true) {
        int l = 2 * i + 1, r = 2 * i + 2, largest = i;
        if (l < n && a[l] > a[largest]) largest = l;
        if (r < n && a[r] > a[largest]) largest = r;
        if (largest == i) break;
        swap(a[i], a[largest]);
        i = largest;
    }
}

void heapSort(vector<int>& a) {
    int n = a.size();
    for (int i = n / 2 - 1; i >= 0; --i) siftDown(a, n, i);   // O(n) 建堆
    for (int i = n - 1; i > 0; --i) {
        swap(a[0], a[i]);
        siftDown(a, i, 0);
    }
}
```

$O(n\log n)$，空间 $O(1)$，不稳定。

### 6.2 计数排序

值域 $V$ 不大时，统计每个值出现次数再按顺序输出：

```cpp
void countingSort(vector<int>& a, int maxV) {
    vector<int> cnt(maxV + 1, 0);
    for (int x : a) ++cnt[x];
    int k = 0;
    for (int v = 0; v <= maxV; ++v)
        while (cnt[v]--) a[k++] = v;
}
```

$O(n + V)$，空间 $O(V)$，适合值域小的整数（成绩、年龄）。

---

## 7. STL 的 sort

```cpp
sort(a.begin(), a.end());                        // 升序
sort(a.begin(), a.end(), greater<int>());        // 降序
sort(a, a + n);                                  // 数组
sort(a.begin(), a.end(), [](int x, int y) { return x > y; });  // lambda
```

`std::sort` 是**内省排序**（快排 + 堆排 + 插入排序的混合），保证最坏 $O(n\log n)$。需要稳定排序时用 `stable_sort`（归并实现，$O(n\log n)$ 但常数更大、需 $O(n)$ 额外空间）。

### 自定义比较器

```cpp
struct Node {
    int x, y;
    bool operator<(const Node& o) const {     // 成员函数写法
        return x != o.x ? x < o.x : y < o.y;
    }
};
sort(v.begin(), v.end());                     // 自动使用 operator<

sort(v.begin(), v.end(), [](const Node& a, const Node& b) {   // lambda 写法
    return a.x != b.x ? a.x < b.x : a.y < b.y;
});
```

---

## 8. 模板代码

### 8.1 归并排序与逆序对

```cpp
#include <bits/stdc++.h>
using namespace std;

long long mergeCount(vector<int>& a, int l, int r, vector<int>& tmp) {
    if (l >= r) return 0;
    int mid = (l + r) >> 1;
    long long cnt = mergeCount(a, l, mid, tmp) + mergeCount(a, mid + 1, r, tmp);

    int i = l, j = mid + 1, k = l;
    while (i <= mid && j <= r) {
        if (a[i] <= a[j]) {
            tmp[k++] = a[i++];
        } else {
            tmp[k++] = a[j++];
            cnt += mid - i + 1;      // 右侧 a[j] 比左侧剩余的都小
        }
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r)   tmp[k++] = a[j++];
    for (int p = l; p <= r; ++p) a[p] = tmp[p];
    return cnt;
}

int main() {
    int n;
    cin >> n;
    vector<int> a(n), tmp(n);
    for (int& x : a) cin >> x;
    cout << mergeCount(a, 0, n - 1, tmp) << '\n';
    return 0;
}
```

### 8.2 多关键字排序

```cpp
struct Student {
    string name;
    int score, id;
    bool operator<(const Student& o) const {
        if (score != o.score) return score > o.score;   // 分数降序
        return id < o.id;                                // 同分按输入顺序
    }
};

int main() {
    int n;
    cin >> n;
    vector<Student> v(n);
    for (int i = 0; i < n; ++i) {
        cin >> v[i].name >> v[i].score;
        v[i].id = i;
    }
    sort(v.begin(), v.end());
    for (auto& s : v) cout << s.name << ' ' << s.score << '\n';
    return 0;
}
```

---

## 9. 经典例题

### 例题 1：排序模板题

**题意**：给定 $n$（$n \le 10^5$）个整数，按从小到大输出。

**思路**：直接 `sort`。**复杂度**：$O(n\log n)$。

```cpp
int main() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    sort(a.begin(), a.end());
    for (int x : a) cout << x << ' ';
    return 0;
}
```

### 例题 2：逆序对

**题意**：给定长度为 $n$ 的序列，求逆序对个数（满足 $i < j$ 且 $a_i > a_j$ 的对数）。

**思路**：归并排序合并时统计：当右侧元素先出队时，左侧剩余元素都与它构成逆序对，贡献 `mid - i + 1`。**复杂度**：$O(n\log n)$。（代码见上文模板。）

### 例题 3：成绩排序

**题意**：$n$ 个学生，每人有姓名和分数。按分数从高到低排序，同分按输入顺序。

**思路**：给每个学生记原始下标 `id`，比较器里分数降序、`id` 升序。**复杂度**：$O(n\log n)$。（代码见上文模板。）

### 例题 4：第 k 小元素

**题意**：给定 $n$ 个整数，求第 $k$ 小的数（$1 \le k \le n$）。

**思路 1**：排序后取 `a[k-1]`，$O(n\log n)$。**思路 2**：`nth_element`，平均 $O(n)$ 且不需要完全排序。

```cpp
int main() {
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    nth_element(a.begin(), a.begin() + k - 1, a.end());
    cout << a[k - 1] << '\n';
    return 0;
}
```

---

## 10. 易错点

1. **比较器写 `<=`**：违反严格弱序，`std::sort` 可能段错误。
2. **快排基准取首元素**：已排序数据会退化成 $O(n^2)$，随机化或取中间。
3. **归并排序忘记拷贝回原数组**：`tmp` 只是临时空间，最后要写回。
4. **归并排序的稳定性**：合并时用 `<=` 而不是 `<`。
5. **冒泡排序的边界**：内层循环到 `n-1-i`，不要写成 `n-1`。
6. **计数排序值域太大**：$V = 10^9$ 时开不出数组，要先离散化。
7. **`sort` 对 `pair` 的默认比较**：先比 `first` 再比 `second`，有时不符合预期。
8. **多关键字排序顺序反了**：先写的条件优先级最高。
9. **`nth_element` 后数组不是有序的**：它只保证第 $k$ 位是第 $k$ 小。
10. **排序结构体时没有定义 `<`**：编译报错；用 lambda 时注意捕获。
11. **`stable_sort` 需要更多内存**：内存紧张时可能 MLE。
12. **`sort` 比较浮点数**：精度问题可能导致顺序不稳定，用 `eps` 比较。

---

## 11. 练习建议

### 11.1 学习路线（由易到难）

1. **`std::sort` 基本用法**：升序、降序、结构体排序。
2. **手写 $O(n^2)$ 排序**：冒泡、选择、插入，理解过程。
3. **手写归并排序**：理解分治与合并。
4. **手写快速排序**：理解 partition。
5. **应用**：逆序对、第 k 小、去重、离散化。
6. **非比较排序**：计数排序、基数排序。
7. **综合**：多关键字排序、稳定排序。

### 11.2 刷题清单

| 难度 | 题目 | 训练点 |
|---|---|---|
| 入门 | 洛谷 P1177 快速排序 | `sort` 模板 |
| 入门 | 洛谷 P1059 明明的随机数 | 排序 + 去重 |
| 普及− | 洛谷 P1093 奖学金 | 多关键字排序 |
| 普及− | 洛谷 P1908 逆序对 | 归并排序 |
| 普及/提高− | 洛谷 P1923 求第 k 小的数 | `nth_element` |
| 普及/提高− | 洛谷 P2676 Bookshelf B | 贪心 + 排序 |

### 11.3 学习检查清单

- [ ] 能说出各排序算法的时间、空间、稳定性
- [ ] 能手写归并排序与快速排序
- [ ] 能用归并排序求逆序对
- [ ] 能写正确的比较器（严格弱序）
- [ ] 知道 `std::sort` 是内省排序
- [ ] 会用 `nth_element` 求第 k 小
- [ ] 知道什么时候需要稳定排序

> **核心思想**：排序不只是「把数排好」，更是**把问题变成有序结构以便用更快的算法**。理解每种排序的代价，才能在赛场上选对工具。
