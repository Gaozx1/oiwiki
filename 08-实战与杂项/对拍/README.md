# 对拍（Stress Testing）

> 对拍 = 用「暴力程序」验证「正解程序」：随机造数据 → 两个程序同时跑 → 比较输出。比赛调不出错时的终极武器。

---

## 目录

1. [为什么需要对拍](#1-为什么需要对拍)
2. [对拍的三要素](#2-对拍的三要素)
3. [Windows 下的对拍脚本](#3-windows-下的对拍脚本)
4. [Linux 下的对拍脚本](#4-linux-下的对拍脚本)
5. [数据生成器怎么写](#5-数据生成器怎么写)
6. [常见对拍套路](#6-常见对拍套路)
7. [对拍的局限](#7-对拍的局限)
8. [实战建议](#8-实战建议)

---

## 1. 为什么需要对拍

人工手造数据有两个致命问题：

- **造不出边界**：你想到的样例，你的代码大概率也能过；
- **造不出规模**：小数据过了不代表大数据对。

对拍的价值在于**用随机数据自动覆盖你没想到的情况**。它不能证明程序正确，但能以极高概率暴露错误。

> 经验法则：**只要能写出暴力，就应该对拍**。对拍十分钟，胜过盯代码一小时。

---

## 2. 对拍的三要素

对拍需要三个程序：

| 程序 | 作用 | 要求 |
|---|---|---|
| `gen.exe` | 数据生成器 | 随机、且能按参数控制数据规模 |
| `brute.exe` | 暴力/标准程序 | **必须正确**，可以慢 |
| `sol.exe` | 你的程序 | 待验证 |

流程：

```
循环 1000 次:
    gen > in.txt
    brute < in.txt > out1.txt
    sol   < in.txt > out2.txt
    if out1 != out2:
        保存 in.txt 并退出
```

---

## 3. Windows 下的对拍脚本

### 3.1 批处理（.bat）

```bat
@echo off
setlocal enabledelayedexpansion
for /l %%i in (1,1,1000) do (
    gen.exe > in.txt
    brute.exe < in.txt > out1.txt
    sol.exe < in.txt > out2.txt
    fc out1.txt out2.txt > nul
    if errorlevel 1 (
        echo Wrong Answer on test %%i
        pause
        exit /b
    )
    echo Test %%i OK
)
echo All tests passed!
pause
```

要点：

- `fc a b > nul` 比较两个文件，不同时 `errorlevel` 为 1；
- 出错时 **不要自动删除 `in.txt`**，方便复现；
- 把 `gen.exe`、`brute.exe`、`sol.exe` 和脚本放同一目录。

### 3.2 PowerShell 版本（更灵活）

```powershell
$rounds = 1000
for ($i = 1; $i -le $rounds; $i++) {
    .\gen.exe > in.txt
    .\brute.exe < in.txt > out1.txt
    .\sol.exe  < in.txt > out2.txt

    $a = Get-Content out1.txt -Raw
    $b = Get-Content out2.txt -Raw
    if ($a -ne $b) {
        Write-Host "WA on test $i" -ForegroundColor Red
        break
    }
    Write-Host "test $i OK"
}
```

> 注意：PowerShell 的 `<` 重定向不是原生支持的，某些版本会报错。**推荐用 cmd 运行 .bat**，或改用 C++ 写的对拍器（见 3.3）。

### 3.3 C++ 写的对拍器（跨平台最稳）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main(int argc, char** argv) {
    int rounds = (argc > 1) ? atoi(argv[1]) : 1000;
    for (int i = 1; i <= rounds; ++i) {
        system("gen.exe > in.txt");
        system("brute.exe < in.txt > out1.txt");
        system("sol.exe   < in.txt > out2.txt");
        if (system("fc out1.txt out2.txt > nul") != 0) {
            printf("WA on test %d\n", i);
            return 0;
        }
        printf("test %d OK\n", i);
    }
    printf("All tests passed!\n");
    return 0;
}
```

Linux 下把 `fc a b > nul` 换成 `diff a b > /dev/null`，`.exe` 换成无后缀即可。

---

## 4. Linux 下的对拍脚本

```bash
#!/bin/bash
for ((i=1; i<=1000; i++)); do
    ./gen > in.txt
    ./brute < in.txt > out1.txt
    ./sol   < in.txt > out2.txt
    if ! diff -q out1.txt out2.txt > /dev/null; then
        echo "WA on test $i"
        break
    fi
    echo "test $i OK"
done
```

在 OI 比赛中（NOI Linux），用 `bash` 写对拍通常比 bat 方便。

---

## 5. 数据生成器怎么写

**核心要求：每次运行结果不同**（用 `srand(time(0))` 或 `mt19937` 随机种子）。

### 5.1 基础模板

```cpp
#include <bits/stdc++.h>
using namespace std;

mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());

int rnd(int l, int r) {                      // [l, r] 均匀随机
    return uniform_int_distribution<int>(l, r)(rng);
}

int main() {
    int n = rnd(1, 10);
    int m = rnd(1, 10);
    printf("%d %d\n", n, m);
    for (int i = 1; i <= n; ++i) printf("%d ", rnd(-5, 5));
    puts("");
    return 0;
}
```

### 5.2 造出「有梯度」的数据

好的生成器应该分档：

```cpp
#include <bits/stdc++.h>
using namespace std;

mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());
int rnd(int l, int r) { return uniform_int_distribution<int>(l, r)(rng); }

int main(int argc, char** argv) {
    int type = (argc > 1) ? atoi(argv[1]) : 0;   // 用命令行参数选档
    int n;
    if (type == 0) {                             // 小数据
        n = rnd(1, 8);
    } else if (type == 1) {                      // 边界数据
        n = rnd(1, 2);
    } else if (type == 2) {                      // 大数据
        n = 100000;
    } else {                                     // 随机中等
        n = rnd(1, 2000);
    }
    printf("%d\n", n);
    for (int i = 1; i <= n; ++i) printf("%d ", rnd(-1000, 1000));
    puts("");
    return 0;
}
```

### 5.3 特殊结构的构造

| 想测什么 | 怎么造 |
|---|---|
| 全相同 | `a[i] = 1` |
| 已有序 | `a[i] = i` |
| 逆序 | `a[i] = n - i + 1` |
| 链状树 | `i` 连 `i-1` |
| 菊花树 | `1` 连所有点 |
| 星形图/稠密图 | 特殊边集 |
| 卡哈希 | 构造碰撞字符串（如 `a` 与 `aa...`） |

---

## 6. 常见对拍套路

### 6.1 暴力对正解

最常用。例如：线段树题写一个 $O(nq)$ 的暴力。

### 6.2 小数据暴力 + 大数据随机

当暴力在 $n=1000$ 就跑不动时，把 `gen` 的 $n$ 限制在 $1 \sim 10$，只验证**逻辑正确性**。

### 6.3 双解对拍

两个不同做法的程序互相验证（一个用 `map` 一个用线段树），比暴力更容易在大数据上跑。

### 6.4 特殊性质对拍

只测「链」「菊花」「全相同」等特殊数据，往往能快速定位边界 bug。

### 6.5 与其他工具配合

- **ASan / UBSan**（Linux）：编译时加 `-fsanitize=address,undefined`，能抓越界与溢出；
- **`gdb`**：对拍出错后直接 `gdb ./sol` 复现；
- **输出比对工具**：实数题用 `diff -w` 或专用 checker。

---

## 7. 对拍的局限

1. **暴力必须正确**：暴力错则对拍无意义；
2. **随机数据未必覆盖稀有结构**：需要人为构造特殊数据；
3. **对拍通过 ≠ 正确**：只是没找到反例；
4. **大数据下的暴力可能超时**：需要控制规模；
5. **多解题目**：输出不唯一时不能直接比较，需要写 SPJ（Special Judge）。

---

## 8. 实战建议

- **比赛一开始就写暴力**：既是保底分，也是对拍基础；
- **对拍前先自己想 3 组数据**：很多错误肉眼可见；
- **保留出错数据**：`in.txt` 不要覆盖，出错后手动缩小数据（删点、缩小值域）来定位；
- **数据缩小技巧**：二分删除元素，直到最小复现；
- **写对拍器时固定随机种子**：出问题后可复现同一组数据；
- **NOI Linux 下优先用 shell 脚本**，Windows 下优先 bat。

---

## 附：最小可复现工程模板

```
stress/
├── gen.cpp
├── brute.cpp
├── sol.cpp
└── stress.bat
```

编译命令（Windows + MinGW）：

```bat
g++ -O2 -std=c++14 gen.cpp -o gen.exe
g++ -O2 -std=c++14 brute.cpp -o brute.exe
g++ -O2 -std=c++14 sol.cpp -o sol.exe
stress.bat
```

Linux：

```bash
g++ -O2 -std=c++14 -o gen gen.cpp
g++ -O2 -std=c++14 -o brute brute.cpp
g++ -O2 -std=c++14 -o sol sol.cpp
bash stress.sh
```

---

## 附：常见错误与练习清单

### 常见错误

1. **忽略边界情形**：空输入、最小规模、极大值、全相同元素。
2. **想当然地套模板**：没有先确认问题的前提条件是否满足。
3. **调试输出忘记删除**：提交前检索 `cerr`、`printf("debug` 等。
4. **变量作用域混乱**：局部变量遮蔽全局变量，多组数据不清空。
5. **时间估算错误**：只看复杂度，忽略常数与评测机性能。

### 练习建议

- 先用最小规模数据手算一遍，确认思路无误；
- 写完后用对拍脚本自测（见 [对拍](../对拍/README.md)）；
- 整理一份自己的错题本，定期回看；
- 把本文的方法固化成checklist，每次比赛前过一遍。