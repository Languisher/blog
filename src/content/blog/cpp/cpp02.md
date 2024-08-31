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
string s2 = "Value";
string s3("Value");
string s4(n, "c"); // = string s4("c...c") where c appears n times
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
	els本文简单介绍了 C++ 中的标准库顺序容器，主要参考 C++ Primer[^1] 第三和第九章的内容。

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
string s2 = "Value";
string s3("Value");
string s4(n, "c"); // = string s4("c...c") where c appears n times
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

### 常量迭代器 const_iterator

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

### 反向迭代器 reverse_iterator

顾名思义，反向迭代器对执行各种操作的含义都发生了颠倒，若要返回反向迭代器，则可以使用 `rbegin` 和 `rend`：

```cpp
list<string> a = {"A", "B", "C"};
auto it1 = a.rbegin(); // 类型是 list<string>::reverse_iterator
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

## 容器通用操作

### 容器初始化、拷贝容器

默认的初始化（即不显示初始化）是用其**默认构造函数**。

最基础的初始化就是显示传递容器内元素的列表，要求其元素类型两者必须向容：
```cpp
C c{a, b, c, d}; // 用花括号
C c = {a, b, c, d};
```

若想讲一个容器拷贝至新容器中，有两种可行条件：
1. 直接拷贝整个容器，要求**容器类型**和**其元素类型**都相互匹配。
2. 传递容器迭代器（`begin()` 和 `end()`），可以两者都不匹配，但要求元素能够被转换。

```cpp
// 方法 1
list<string> authors = {"M", "S"};
list<string> list1(authors); // 正确，list 和 string 都匹配
list<string> list2 = list1; // 也可以用 =

// 方法 2
vector<const char*> articles = {"a", "an", "the"};
forward_list<string> words(articles.begin(), articles.end()); // 正确，可以转换
```

最后，部分容器支持指定数目的同一元素作为初始化：

```cpp
vector<int> ivec(10, -1); // 10 个 -1 作为初始化
```

### 容器赋值和交换

**赋值**操作分成两种：
1. 要求两个容器类型一致，将一个容器内元素全部替换成另外一个元素容器的拷贝：`c1 = c2`
2. 两个容器类型可以不一致，用 `.assign(b, e)` 方法：

```cpp
list<string> names;
vector<const char*> oldstyle;

names.assign(oldstyle.cbegin(), oldstyle.cend());
names.assign(10, "Haya"); // 替换成 n 个值为 t 的元素
```

**交换**操作要求两个容器类型一致：`c1.swap(c2)` 或 `swap(c1, c2)`

**注意：赋值会导致容器内部【迭代器、引用和指针】全部失效，但交换不会！（除 string 和 array）**


### 容器大小操作

根据以下几个原则比较：
1. 一一比较，遇到元素不同时，含有较大元素的容器较大
2. 若没有不同的元素，但容器大小不同时，含有较多元素的容器较大
3. 若两者大小相同且所有元素对应相等，则两者相等，否则不等

## 顺序容器操作

### 顺序容器种类一览

（自己查

### 顺序容器添加元素

| 操作                    | 命令                      | 注释                      |
| --------------------- | ----------------------- | ----------------------- |
| *尾部*添加单个元素，传值方式       | `c.push_back(t)`        | `forward_list` 不支持      |
| *尾部*添加单个元素，构造函数方式     | `c.emplace_back(args)`  | `forward_list` 不支持      |
| *头部*添加单个元素，传值方式       | `c.push_front(t)`       | `vector` 和 `string` 不支持 |
| *头部*添加单个元素，构造函数方式     | `c.emplace_front(args)` | `vector` 和 `string` 不支持 |
| *迭代器*之前添加单个元素，传值方式    | `c.insert(p, t)`        | `p` 是迭代器，下同             |
| *迭代器*之前添加单个元素，构造函数方式  | `c.emplace(p, args)`    |                         |
| *迭代器*之前添加 n 个相同的元素，传值 | `c.insert(p, n, t)`     |                         |
| *迭代器*之前添加迭代器指定范围内元素   | `c.insert(p, b, e)`     | `b`, `e` 是迭代器且不指向 `c`   |
| *迭代器*之前添加花括号范围内的元素    | `c.insert(p, il)`       |                         |
使用**迭代器**添加元素的方式，返回的是指向新元素/新添加的第一个元素的迭代器；在**头部/尾部**添加元素返回 `void`.

### 顺序容器访问元素

| 操作                     | 命令          | 注释                 |
| ---------------------- | ----------- | ------------------ |
| *尾部*元素的引用              | `c.back()`  | `forward_list` 不支持 |
| *头部*元素的引用              | `c.front()` |                    |
| *索引*为 n 元素的访问，若越界则为定义  | `c[n]`      | 只适用部分              |
| *索引*为 n 元素的访问，若越界则抛出异常 | `c.at(n)`   | 只适用部分              |
注意：不能对空容器调用 `front` 和 `back`.

### 顺序容器删除元素

删除元素会影响容器大小，因此全部不能使用在 array 上。

| 操作           | 命令              | 注释                      |
| ------------ | --------------- | ----------------------- |
| *尾部*元素删除     | `c.pop_back()`  | `forward_list` 不支持      |
| *头部*元素删除     | `c.pop_front()` | `vector` 和 `string` 不支持 |
| *迭代器*指向元素删除  | `c.erase(p)`    | `p` 是迭代器                |
| *迭代器*范围内元素删除 | `c.erase(b, e)` | `b`, `e` 是迭代器且指向 `c`    |
| *所有*元素删除     | `c.clear()`     |                         |
使用**迭代器**删除元素的方式，返回的是被删元素之后/被删的最后一个元素之后迭代器；其他方式均是返回 `void`，因此若要实现类似 `pop` 操作必须先保存该元素。

### 改变顺序容器大小

改变顺序容器大小 `resize` 会导致两个后果：
1. 若容器元素数目小于要求，则会将新元素填充到容器后部
2. 若容器元素数目大于要求，则会删除后部的元素直至满足要求

```cpp
list<int> ilist(10, 42);
ilist.resize(25, -1); // 填充了 15 个 -1
ilist.resize(5); // 删除 20 个元素
```

### 改变顺序容器元素导致迭代器、指针和引用失效

原则是：当添加/删除元素之后，原来的容器存储空间被重新分配，则指向容器的迭代器、指针和引用可能会失效。

- `list` 和 `forward_list` 的链表结构决定无论什么操作都不会使其失效
- `deque` 双向列表导致除了首尾位置的插入/删除都会失效。神奇的是，如果首尾位置插入，迭代器会失效但指针和引用不会；首位置和尾位置的删除（除了尾位置删除时的尾后迭代器）都不会失效
- `vector` 和 `string` 更复杂：
	- 添加元素之后导致存储空间再分配，全部失效
	- 添加元素之后存储空间未再分配，插入位置之前仍有效，之后都会失效
	- 删除元素之前仍有效，之后都会失效

通常情况下，`end` 迭代器会被持续更新，因此*不要保存 end 返回的迭代器*。

## 特殊的容器适配器：队列和优先级队列 queue, priority_queue 和栈 stack

| 操作                     | 命令                | 注释                 |
| ---------------------- | ----------------- | ------------------ |
| *删除*首元素，但不将其返回         | `q.pop()`         | 对于优先级队列，返回优先级最高的元素 |
| *返回*首元素或尾元素，但不删除该元素    | `q.front()`       | 仅队列、优先级队列          |
| *返回*最高优先级或栈顶元素，但不删除该元素 | `q.top(n)`        | 仅栈、优先级队列           |
| 在队列末尾*添加*某元素           | `q.push(item)`    |                    |
| 在队列末尾由构造器*添加*某元素       | `q.emplace(args)` |                    |

这些类型都是**适配器**，是基于以上类型实现的。适配器类型不能使用其基于的底层容器操作，必须使用以上自己独有的特殊操作，例如不能使用 `push_back` 而必须使用 `push` 添加元素。