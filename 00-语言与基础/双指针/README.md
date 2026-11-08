# 双指针

> 一句话概括：**双指针是用两个下标在序列上移动来替代嵌套循环的技巧，把大量 $O(n^2)$ 的区间问题降到 $O(n)$。**

---

## 目录

1. [什么是双指针](#1-什么是双指针)
2. [同向双指针与快慢指针](#2-同向双指针与快慢指针)
3. [对撞双指针](#3-对撞双指针)
4. [滑动窗口（尺取法）](#4-滑动窗口尺取法)
5. [双指针的正确性](#5-双指针的正确性)
6. [模板代码](#6-模板代码)
7. [经典例题](#7-经典例题)
8. [易错点](#8-易错点)
9. [练习建议](#9-练习建议)

---

## 1. 什么是双指针

普通的双重循环枚举所有区间：

```cpp
for (int l = 0; l < n; ++l)
    for (int r = l; r < n; ++r)
        check(l, r);          // O(n^2) 个区间
```

如果题目具有某种**单调性**，不必检查所有 $O(n^2)$ 个区间——可以让 `l`、`r` 两个指针**只向前移动、从不回头**，总移动次数 $O(n)$，从而把复杂度降到 $O(n)$。

### 双指针的三种形态

| 形态 | 指针移动方向 | 典型问题 |
|---|---|---|
| 同向双指针（快慢指针） | 都向右，一快一慢 | 去重、判环、删除元素 |
| 对撞双指针 | 一个从左、一个从右，相向而行 | 有序数组两数之和、回文判断 |
| 滑动窗口 | 左右边界同向移动，维护区间 | 最短/最长满足条件的子数组 |

### 双指针的两个前提

1. **单调性**：某个量随着指针移动单调变化（如窗口和只增不减）；
2. **不回退**：左指针不会因为右指针前进而需要后退。

只要满足这两点，双指针才正确。

---

## 2. 同向双指针与快慢指针

### 2.1 快慢指针删除元素

**题目**：原地删除数组中所有等于 `val` 的元素，返回新长度。

```cpp
int removeElement(vector<int>& a, int val) {
    int slow = 0;
    for (int fast = 0; fast < (int)a.size(); ++fast)
        if (a[fast] != val) a[slow++] = a[fast];   // 把保留的元素往前挪
    return slow;                                   // slow 就是新长度
}
```

`fast` 负责扫描，`slow` 负责写入。所有被保留的元素按原顺序挤在 `[0, slow)`。

### 2.2 快慢指针判断链表环

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int v) : val(v), next(nullptr) {}
};
bool hasCycle(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;             // 慢指针走一步
        fast = fast->next->next;       // 快指针走两步
        if (slow == fast) return true; // 追上说明有环
    }
    return false;
}
```

**原理**：若有环，快指针每轮比慢指针多走一步，最终一定在环内追上；若无环，快指针先到 `nullptr`。

### 2.3 快慢指针找链表中点

```cpp
ListNode* middleNode(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;      // 偶数个节点时返回靠后的中间节点
}
```

### 2.4 数组去重

```cpp
int removeDuplicates(vector<int>& a) {
    if (a.empty()) return 0;
    int slow = 1;
    for (int fast = 1; fast < (int)a.size(); ++fast)
        if (a[fast] != a[slow - 1]) a[slow++] = a[fast];
    return slow;
}
```

---

## 3. 对撞双指针

### 3.1 有序数组两数之和

**题目**：升序数组 $a$，找两个数使其和等于 $target$。

```cpp
pair<int,int> twoSum(const vector<int>& a, int target) {
    int l = 0, r = (int)a.size() - 1;
    while (l < r) {
        int s = a[l] + a[r];
        if (s == target) return {l, r};
        if (s < target) ++l;      // 和太小，左指针右移增大
        else --r;                  // 和太大，右指针左移减小
    }
    return {-1, -1};
}
```

**为什么正确**：数组有序，`a[l]` 是当前可选的最小值，`a[r]` 是最大值。

- 若 $a_l + a_r < target$，则 `a[l]` 与任何更小的数（不存在）都无法凑成 target，所以 `l` 可以安全右移；
- 若和大于 target，同理 `r` 可以安全左移。

每个元素最多被访问一次，复杂度 $O(n)$。

### 3.2 回文串判断与反转

```cpp
bool isPalindrome(const string& s) {
    int l = 0, r = (int)s.size() - 1;
    while (l < r) {
        if (s[l] != s[r]) return false;
        ++l; --r;
    }
    return true;
}
void reverseArray(vector<int>& a) {
    int l = 0, r = (int)a.size() - 1;
    while (l < r) swap(a[l++], a[r--]);
}
```

### 3.3 三数之和

**题目**：无序数组，找所有和为 0 的三元组（不重复）。先排序，枚举第一个数 $a_i$，剩余部分用对撞双指针找两数之和为 $-a_i$。复杂度 $O(n^2)$。

```cpp
vector<vector<int>> threeSum(vector<int>& a) {
    sort(a.begin(), a.end());
    int n = a.size();
    vector<vector<int>> ans;
    for (int i = 0; i < n - 2; ++i) {
        if (i > 0 && a[i] == a[i - 1]) continue;          // 跳过重复的第一个数
        int l = i + 1, r = n - 1;
        while (l < r) {
            int s = a[i] + a[l] + a[r];
            if (s < 0) ++l;
            else if (s > 0) --r;
            else {
                ans.push_back({a[i], a[l], a[r]});
                while (l < r && a[l] == a[l + 1]) ++l;      // 跳过重复
                while (l < r && a[r] == a[r - 1]) --r;
                ++l; --r;
            }
        }
    }
    return ans;
}
```

---

## 4. 滑动窗口（尺取法）

### 4.1 模板

维护一个区间 `[l, r]`，右端点不断扩张，当区间不满足条件时左端点收缩：

```cpp
int l = 0;
long long window = 0;              // 窗口内的统计量
for (int r = 0; r < n; ++r) {
    window += a[r];                 // 加入 a[r]
    while (/* 窗口不满足条件 */) {
        window -= a[l];             // 移除 a[l]
        ++l;
    }
    // 此时窗口 [l, r] 满足条件，更新答案
}
```

**关键**：`l` 和 `r` 都只增不减，所以总移动次数 $O(n)$。窗口内的统计量通常写成

$$\text{sum}(l, r) = \sum_{k=l}^{r} a_k$$

### 4.2 最短子数组（和 ≥ target）

```cpp
int minSubArrayLen(int target, const vector<int>& a) {
    int n = a.size(), l = 0, ans = INT_MAX;
    long long sum = 0;
    for (int r = 0; r < n; ++r) {
        sum += a[r];
        while (sum >= target) {          // 满足条件，尝试收缩
            ans = min(ans, r - l + 1);
            sum -= a[l++];
        }
    }
    return ans == INT_MAX ? 0 : ans;
}
```

### 4.3 最长无重复子串

```cpp
int lengthOfLongestSubstring(const string& s) {
    vector<int> last(256, -1);       // 每个字符上次出现的位置
    int l = 0, ans = 0;
    for (int r = 0; r < (int)s.size(); ++r) {
        if (last[(unsigned char)s[r]] >= l)
            l = last[(unsigned char)s[r]] + 1;   // 左指针跳到重复字符之后
        last[(unsigned char)s[r]] = r;
        ans = max(ans, r - l + 1);
    }
    return ans;
}
```

### 4.4 滑动窗口的适用条件

窗口的统计量必须**可增可减**（如和、计数）；条件关于窗口长度必须**单调**（窗口越长越容易满足，或越难满足）；若统计量无法在 $O(1)$ 内更新（如中位数），需要 `multiset` 等结构辅助。

---

## 5. 双指针的正确性

### 5.1 单调性论证

以「最短子数组和 ≥ target」为例（数组元素均为正）：固定右端点 $r$，随着 $l$ 增大窗口和单调减小，所以满足「和 ≥ target」的 $l$ 形成一个后缀，最优的 $l$ 是其中最大的那个；当 $r$ 增大时，最优的 $l$ 不会减小（新加入的元素只会让和更大）。这就是「左指针不回退」的依据。

### 5.2 什么时候不能用

数组含**负数**时，「最短子数组和 ≥ target」不满足单调性，双指针失效（要用前缀和 + 单调队列）；需要枚举**所有**区间而不是找最优时，双指针只覆盖了一部分；窗口统计量无法快速维护时也不能用。

### 5.3 与其他方法的对比

| 方法 | 复杂度 | 适用 |
|---|---|---|
| 双指针 | $O(n)$ | 单调性成立，求最优区间 |
| 二分 | $O(n\log n)$ | 答案有单调性，或值域二分 |
| 前缀和 + 哈希 | $O(n)$ | 求和为定值的子数组（可含负数） |

---

## 6. 模板代码

### 6.1 对撞双指针模板

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    int n, target;
    cin >> n >> target;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    sort(a.begin(), a.end());
    int l = 0, r = n - 1;
    while (l < r) {
        int s = a[l] + a[r];
        if (s == target) { cout << l << ' ' << r << '\n'; return 0; }
        if (s < target) ++l;
        else --r;
    }
    cout << "No solution\n";
    return 0;
}
```

### 6.2 滑动窗口模板（最短区间）

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    int n, target;
    cin >> n >> target;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    int l = 0, ans = INT_MAX;
    long long sum = 0;
    for (int r = 0; r < n; ++r) {
        sum += a[r];
        while (sum >= target) {
            ans = min(ans, r - l + 1);
            sum -= a[l++];
        }
    }
    cout << (ans == INT_MAX ? 0 : ans) << '\n';
    return 0;
}
```

### 6.3 滑动窗口模板（最长区间）

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    int n, k;
    cin >> n >> k;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    int l = 0, ans = 0;
    long long sum = 0;
    for (int r = 0; r < n; ++r) {
        sum += a[r];
        while (sum > k) sum -= a[l++];   // 不满足条件就收缩
        ans = max(ans, r - l + 1);       // 此时窗口一定满足条件
    }
    cout << ans << '\n';
    return 0;
}
```

### 6.4 快慢指针模板

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int& x : a) cin >> x;
    int slow = 0;
    for (int fast = 0; fast < n; ++fast)
        if (a[fast] % 2 == 1) {          // 把所有奇数移到前面
            swap(a[slow], a[fast]);
            ++slow;
        }
    for (int x : a) cout << x << ' ';
    return 0;
}
```

---

## 7. 经典例题

### 例题 1：有序数组两数之和

**题意**：给定升序数组 $a$（$n \le 10^5$）与目标值 $target$，找一对下标 $i < j$ 使 $a_i + a_j = target$。

**思路**：对撞双指针，和小于目标左移，和大于目标右移。**复杂度**：$O(n)$。（代码见 6.1。）

### 例题 2：长度最小的子数组

**题意**：给定正整数数组 $a$ 和目标 $S$，找出和 $\ge S$ 的最短连续子数组长度。

**思路**：滑动窗口。右端点扩张，和达到 $S$ 后不断收缩左端点并更新答案。**复杂度**：$O(n)$。（代码见 4.2。）

### 例题 3：最长无重复字符子串

**题意**：给定字符串 $s$，求不含重复字符的最长子串长度。

**思路**：滑动窗口 + `last` 数组记录每个字符上次出现位置。右端点遇到重复时，左端点跳到重复字符后一位。**复杂度**：$O(n)$。（代码见 4.3。）

### 例题 4：三数之和

**题意**：给定数组，找出所有和为 0 的三元组，不能重复。

**思路**：排序后枚举第一个数，再用对撞双指针找后两个数；注意跳过重复元素。**复杂度**：$O(n^2)$。（代码见 3.3。）

### 例题 5：判断链表是否有环

**题意**：给定链表头节点，判断链表中是否有环。**思路**：快慢指针，慢指针走一步、快指针走两步，若相遇则有环。**复杂度**：时间 $O(n)$，空间 $O(1)$。（代码见 2.2。）

---

## 8. 易错点

1. **左指针回退**：写成 `l = 0` 重置会让复杂度退化为 $O(n^2)$。
2. **`while` 与 `if` 混用**：收缩可能要进行多次，必须用 `while` 而不是 `if`。
3. **窗口不满足条件时的处理顺序**：先扩张再收缩，还是先收缩再扩张，不同题目不同，要理清「窗口不变式」。
4. **数组含负数时用滑动窗口**：单调性被破坏，结果错误。
5. **对撞指针的终止条件**：`l < r` 还是 `l <= r`，取决于是否允许同一个元素用两次。
6. **去重时漏掉边界**：三数之和中 `while (l < r && a[l] == a[l+1])` 的内层判断不能少 `l < r`。
7. **快慢指针判环的空指针**：`fast->next` 前要确保 `fast` 非空。
8. **链表找中点返回哪个**：快指针走两步时偶数节点返回靠后的中点，需要靠前的要改条件。
9. **窗口长度计算**：`r - l + 1`，不要漏 `+1`。
10. **答案初始化**：求最小值时初始化为 `INT_MAX`，求最大值时初始化为 0 或负无穷。
11. **`int` 溢出**：窗口和可能超过 `int`，用 `long long`。
12. **字符下标越界**：用 `s[r]` 作数组下标时先转 `unsigned char`。

---

## 9. 练习建议

### 9.1 学习路线（由易到难）

1. **对撞双指针**：有序数组两数之和、回文判断、反转数组。
2. **快慢指针**：数组去重、删除元素。
3. **滑动窗口（固定长度）**：窗口最大值、固定长度子数组和。
4. **滑动窗口（可变长度）**：最短子数组、最长无重复子串。
5. **多指针**：三数之和、四数之和。
6. **链表双指针**：判环、找中点、找倒数第 k 个。
7. **综合**：双指针 + 排序、双指针 + 前缀和。

### 9.2 刷题清单

| 难度 | 题目 | 训练点 |
|---|---|---|
| 入门 | LeetCode 167 两数之和 II | 对撞双指针 |
| 入门 | LeetCode 26 删除有序数组中的重复项 | 快慢指针 |
| 普及− | 洛谷 P1102 A-B 数对 | 排序 + 双指针 |
| 普及− | LeetCode 209 长度最小的子数组 | 滑动窗口 |
| 普及/提高− | LeetCode 3 无重复字符的最长子串 | 滑动窗口 + 哈希 |
| 普及/提高− | LeetCode 15 三数之和 | 排序 + 对撞双指针 |

### 9.3 学习检查清单

- [ ] 能区分同向双指针、对撞双指针、滑动窗口
- [ ] 能说清双指针正确性依赖的单调性
- [ ] 能写出滑动窗口的通用模板
- [ ] 知道什么时候双指针会失效（如含负数）
- [ ] 能用快慢指针判断链表环
- [ ] 能处理三数之和的去重
- [ ] 能分析双指针的复杂度为什么是 $O(n)$

> **核心思想**：双指针的本质是**「利用单调性，让两个下标各自只走一遍」**。看到「连续子数组」「有序数组两数之和」「链表环」，第一时间想双指针。
