---
title: "C++ 基础（二）：标准库顺序容器"
description: "历史资料，曾经写过的笔记放在这里存档。"
date: 2024-08-26
category: ["Programming Basics"]
tags: ["C++"]
---

本文简单介绍了 C++ 中的标准库顺序容器，主要参考 C++ Primer[^1] 第三和第九章的内容。

[^1]: Lippman, S. B., Lajoie, J., & Moo, B. E. (2012). _C++ Primer_ (5th ed.). Addison-Wesley.
## 标准库类型 string：处理字符串

`string` 表示可变长的字符序列，首先使用包含 `string` 头文件

```cpp
#include <string>
using std::string;
```

### 初始化 string

除了基础的赋值之外，提供特殊的直接初始化方式：
`
```cpp
string s2 = "Value"
string s3("Value")
string s4(n, "c") // = string s4("c...c") where c appears n times
```

### 对 string 对象的整体操作

| 操作                                           | 命令                         |
| -------------------------------------------- | -------------------------- |
| 从流中读写字符                                      | `is >> s`, `os << s`       |
| 从流中读取一行                                      | `getline(is, s)`           |
| 判断是否为空，返回 Bool                               | `s.empty()`                |
| 计算字符串大小（含有多少字符），*注意返回的是 `string::size_type`* | `s.size()`                 |
| 对字符串中字符的引用                                   | `s[n]`                     |
| *连接两个字符串*                                    | `s1 + s2`                  |
| *赋值字符串 （即用字符串副本覆盖）*                          | `s1 = s2`                  |
| *判断两个字符串是否完全相等（字符一一匹配，长度一致）*                 | `s1 == s2`, `s1 != s2`     |
| *根据字符在字典顺序比较字符串*                             | `s1 < s2`, `<=`, `>`, `>=` |
#### 操作额外注意事项

计算字符串大小函数返回的是特殊类型 `string::size_type`，这是一个无符号类型，因此不能将其与负值比较。与此同时，建议使用 `auto` 来为之赋值。

字符串可以两者通过加号连接，字符串和字符字面值常量也可以通过加号连接，但是两个字符字面值常量之间不可以通过加号连接！！（解决方法：强行之间插入 `string` 变量）

```cpp
string s1, s2;
auto len = s1.size();
string s2 = s1 + ", " + "hello";
string s3 = "hello" + ", " + s2; // 错误
```

### 选择 string 对象中的单个字符

两种方法：（1）下标选择特定字符；（2）（通常是希望一一选择字符串的全部字符）使用范围 for 语句：`for (declaration : expression)`，其中 `expression` 是一个表示一个队列的对象；`declaration` 则是一个变量用于访问该队列的*基础元素*，每个队列的元素都被*拷贝*到该元素上。

如果我们希望直接修改原队列的元素而不是对其*拷贝*的值操作，则需要将基础元素定义为引用类型。

```cpp
string s1("some string");
// 按顺序选中 s1 的每个字符，一行行分别将其输出
for (auto c : s1) // for(declaration : expression)
	cout << c << endl;
for (auto &c : s1)
	c = toupper(c); // toupper(c) 将字符转换成大写
```

### 对 string 对象中字符操作

请上网络查询，太多了。

## 标准库类型 vector：顺序容器——对象的按序集合

`vector` 容纳一些对象，其中这些对象类型都完全相同，且可以通过索引访问它们，首先需要包含头文件：

```cpp
#include <vector>
using std::vector;
```

### vector 是一个类模版

`vector` 是一个什么都装得下的容器——它能够容纳各种类型。这种容器称之为**类模版**，指定其装载变量的类型的过程称之为将其**实例化**：`vector<T>`，例如：

```cpp
vector<int> ivec;
vector<vector<int>> file;
```

### 初始化 vector

除了基础的赋值之外，还有一下方式：

```cpp
vector<T> v3(n, val); // 存储 n 个重复的元素且元素值都是 val
vector<T> v4(n); // 同上，元素值被默认初始化
vector<T> v5{a, b, c, d}; // 存储初始值个数的元素，且每个元素被赋予对应值
vector<T> v6 = {a, b, c, d}; // 同上
```

注意列表初始化不能将花括号改成圆括号。区分：
- 圆括号中是 n 个 val 的元素
- 花括号是列表元素个数的元素，被一一赋值

### 向 vector 中添加元素

`push_back` 负责将一个值当作 vector 对象的尾元素压到 vector 容器的尾端：

```cpp
string word;
vector<string> txt;

while (cin >> word) {
	txt.push_back(word);
}
```

不能以下标方式添加元素。
### 对 vector 的操作

基本同 `string` 但没有从流读取以及读取整行的操作，替代的添加元素方式是 `push_back(t)`

| 操作                                    | 命令                     |
| ------------------------------------- | ---------------------- |
| 判断是否为空，返回 bool                        | `v.empty()`            |
| 返回容器内元素的个数，类型是 `vector<T>::size_type` | `v.size()`             |
| 对容器中元素的引用                             | `v[n]`                 |
| 拷贝替换                                  | `v1 = v2`              |
| 判断两个容器元素数量完全相同且其值完全相同                 | `v1 == v2`, `v1 != v2` |
| 以字典顺序比较                               | `<`, `<=`, `>`, `>=`   |

### 选择 vector 容器中的每个元素

基本同 `string`.

```cpp
vector<int> ivec{1, 2, 3, 4, 5};
for (auto &i : v) // 对容器元素进行修改
	i *= i;
for (auto i : v) // 拷贝元素
	cout << i << endl;
```

## 迭代器：对对象的间接访问

对容器内对象的访问，最常见是使用**迭代器**方式，而下标访问并不被所有（大部分）容器支持。迭代器有两种状态：（1）指向容器内的元素，及尾元素的下一个位置，称之为**有效**（2）其它情况均是**无效**的

```cpp
// b 指向容器的第一个元素，e 指向容器的尾元素的下一个位置
auto b = v.begin(), e = v.end();
```

### 对迭代器的操作

主要包含：迭代器的移动、访问迭代器指向的元素、两个迭代器之间的比较操作

| 操作                                  | 命令                                 |
| ----------------------------------- | ---------------------------------- |
| 判断两个迭代器是否指向同一位置                     | `iter1 == iter2`, `iter1 != iter2` |
| 移动迭代器使其指向下/上一个位置                    | `++iter1`, `--iter2`               |
| 移动迭代器使其指向下/上n个位置                    | `iter1 += n`, `iter2 -= n`         |
| 计算两个迭代器之间的距离                        | `iter2 - iter1`                    |
| 返回迭代器指向元素的*引用*（而非复制）                | `*iter`                            |
| 解引用迭代器，并返回名为 `mem` 的成员（通常用在对象是类的情况） | `iter->mem` (=`*iter.mem`)         |
常见组合技：依次处理字符串的字符直到所有字符处理完成或遇到空白：

```cpp
string s; // After initialization
for (auto it = s.begin(); it != s.end() && !isspace(*it); ++it) 
	*it = toupper(*it);
```

### 迭代器类型 const_iterator

`const_iterator` 与常量指针（指向常量的指针）相似，可以访问读取但不能修改其指向的元素。

拥有迭代器的标准库类型使用 `iterator` 和 `const_iterator` 指定类型：
```cpp
vector<int>::iterator it;
string::const_iterator it2;
```

迭代器是否是常量根据对象（这里指容器）是否常量决定。如果要创建一个非常量对象（容器）的常量迭代器，则可以用 `cbegin` 和 `cend`：
```cpp
const vector<int> cv;
auto it2 = v.begin(); // 类型是 vector<int>::const_iterator

vector<int> v;
auto it3 = v.cbegin(); // 显式指明常量迭代器
```

### 使用迭代器的例子：二分搜索

```cpp
// text 是 string 类型，且有序排列
auto beg = text.begin(), end = text.end();
auto mid = text.begin() + (end - beg)/2;

while (mid != end && *mid != sought) {
	if (sought < *mid)
		end = mid;
	else
		beg = mid + 1;
	mid = beg + (end - beg)/2; // update mid
}
```