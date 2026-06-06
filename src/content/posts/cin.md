---
title: 输入读取方式
published: 2026-06-06
description: cin,getline,get的使用
tags: [getline,cin,get]
image: ./images/繁星、飞鸟与鸢尾花.jpg
category: 输入
draft: false
---

下面按 **ACM/ICPC 常用输入场景**把 `cin` / `getline` / `get`（以及经常配套的 `ignore/peek/read`）的主要用法一次讲清。默认都是 C++（`<iostream>`, `<string>`）。

---

## 0) 先给结论：ACM 最常用的三套写法

### A. 纯 token（按空白分隔）输入：最稳
```cpp
int n; 
while (cin >> n) { ... }   // 读到 EOF 结束
```

### B. 需要读整行（含空格）：用 getline
```cpp
string line;
while (getline(cin, line)) { ... }
```

### C. 既有数字/单词又有整行：先吃掉换行再 getline
```cpp
int n; cin >> n;
cin.ignore(numeric_limits<streamsize>::max(), '\n'); // 清掉行尾
string line; getline(cin, line);
```

---

## 1) `cin`（`operator>>`）——“按空白分隔”的输入核心

### 1.1 基本读法
- 读 `int/long long/double/string(不含空格)` 等：
```cpp
int a; long long b; double x; string s;
cin >> a >> b >> x >> s;
```
特点：
- 自动跳过前导空白（空格/换行/制表）
- 对 `string s; cin >> s;`：只读到下一个空白为止（所以读不了带空格的句子）

### 1.2 读到 EOF（ACM 经典）
```cpp
int a, b;
while (cin >> a >> b) {
  cout << a + b << "\n";
}
```
当输入耗尽或格式不匹配时，`cin >> ...` 返回失败，循环结束。

### 1.3 读一个字符（注意：会跳过空白！）
```cpp
char c;
cin >> c;   // 会跳过空格和换行，读到下一个非空白字符
```
如果你想把空格/换行也读进来，不用 `>>`，用 `get()`（见后面）。

### 1.4 常见坑：`cin >> n` 后紧接 `getline`
`>>` 读完数字后，行尾的 `'\n'` 还留在缓冲区，下一次 `getline` 会立刻读到空行。

解决：在 `getline` 前清掉这一行剩余部分：
```cpp
cin.ignore(numeric_limits<streamsize>::max(), '\n');
getline(cin, line);
```

### 1.5 速度（ACM 常见要求）
如果数据量大，建议一开头加：
```cpp
ios::sync_with_stdio(false);
cin.tie(nullptr);
```
并且 **不要混用** `scanf/printf` 与 `cin/cout`（关同步后混用更危险）。

---

## 2) `std::getline` ——“读整行”的输入核心（含空格）

### 2.1 默认用法：读到 `\n`，并丢弃换行符
```cpp
string line;
getline(cin, line); // line 不含 '\n'
```

### 2.2 循环读所有行（到 EOF）
```cpp
string line;
while (getline(cin, line)) {
  // 可能为空行：line == ""
}
```

### 2.3 自定义分隔符（不常见但有用）
```cpp
string part;
getline(cin, part, ','); // 读到逗号为止，逗号被丢弃
```
适合简单 CSV / 自定义格式。

### 2.4 空行、行首空格会不会被保留？
会。`getline` 不会像 `>>` 那样跳过空白。
- 输入行是 `"   abc  def"`，`line` 会保留开头空格。

---

## 3) `cin.get()` / `istream::get` ——“按字符”读（更底层）

这里有几种重载，ACM 常用主要三类：

### 3.1 `int ch = cin.get();`（推荐用来判断 EOF）
```cpp
int ch = cin.get();
if (ch == EOF) ...
else char c = (char)ch;
```
特点：
- **不跳过空白**（空格/换行都能读到）
- 返回 `int`，能区分所有 `unsigned char` 值和 `EOF`

### 3.2 `cin.get(char& c);`
```cpp
char c;
cin.get(c); // 读一个字符（包括空白）
```
如果失败（EOF），`cin` 状态会变坏，需要判断：
```cpp
char c;
if (cin.get(c)) { ... }
```

### 3.3 读一行到 C 风格数组：`cin.getline(buf, n)`
```cpp
char buf[1000];
cin.getline(buf, 1000); // 不包含 '\n'，读到 '\n' 或 n-1
```
注意：
- 如果一行长度 ≥ n，会触发 `failbit`，且缓冲区里可能还残留没读完的字符；这在 ACM 里容易出坑（所以更推荐 `std::string + getline`）。

### 3.4 `cin.get(buf, n)` 与 `getline` 的区别（容易考/踩）
```cpp
cin.get(buf, n);   // 读到 '\n' 之前停止，但 **不会丢弃 '\n'**，'\n' 还在流里
cin.getline(buf,n);// 读到 '\n' 停止，并且 **丢弃 '\n'**
```
这点很关键：`get` 之后你可能还要再 `cin.get()` 把那一个 `'\n'` 拿走。

---

## 4) 其他常配套成员：`ignore / peek / putback / read`

### 4.1 `ignore`：跳过字符（ACM 超常用）
最经典就是清掉行尾换行：
```cpp
cin.ignore(numeric_limits<streamsize>::max(), '\n');
```
也可以跳过固定个数：
```cpp
cin.ignore(1); // 跳过一个字符
```

### 4.2 `peek`：偷看下一个字符但不取走
```cpp
int p = cin.peek();
if (p == '\n') { ... }
```
适合自己写解析器时判断分隔符。

### 4.3 `putback` / `unget`：把字符放回去（少用）
```cpp
char c; cin.get(c);
cin.putback(c); // 退回一个字符
```

### 4.4 `read`：按字节块读（二进制/大块数据才用）
```cpp
char buf[4096];
cin.read(buf, 4096);
auto got = cin.gcount();
```
ACM 大多不需要，除非题目是二进制或极端 IO 优化。

---

## 5) 按 ACM 题型总结“该用哪个”

### 5.1 “每行两个整数”/“若干数字直到 EOF”
用 `cin >>`：
```cpp
long long a, b;
while (cin >> a >> b) cout << a + b << "\n";
```

### 5.2 “一行是一段文本，可能有空格/空行”
用 `getline`：
```cpp
string s;
while (getline(cin, s)) { ... }
```

### 5.3 “输入混合：先给 n，再给 n 行字符串（可含空格）”
```cpp
int n;
cin >> n;
cin.ignore(numeric_limits<streamsize>::max(), '\n');
for (int i=0;i<n;i++){
  string line;
  getline(cin, line);
}
```

### 5.4 “需要把空格、换行都当成有效字符处理（例如画图/网格含空格）”
用 `cin.get()` 或 `getline`（看题意）：
- 行为单位：`getline`
- 字符流单位：`cin.get()`

---

## 6) 一份“ACM 通用输入模板”（建议背下来）

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // 1) token 输入例子：
    // int a, b;
    // while (cin >> a >> b) cout << a + b << "\n";

    // 2) 行输入例子：
    // string line;
    // while (getline(cin, line)) { ... }

    // 3) 混合例子：
    int n;
    if (!(cin >> n)) return 0;
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
    for (int i = 0; i < n; i++) {
        string line;
        getline(cin, line);
        // process line
    }
    return 0;
}
```

---
