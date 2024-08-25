---
title: "C++ 基础变量"
description: "历史资料，曾经写过的笔记放在这里存档。"
date: 2024-03-03
category: ["Software Engineering"]
tags: ["C++"]
---


本文简单介绍了 C++ 中变量的基本类型和复合类型。主要介绍了以下几个方面：
- 什么是变量？
	- 如何初始化/赋值变量
	- 在不同文件中使用同一变量：声明
- 什么是引用？
- 什么是指针？
- 如何解读复合类型？
- const 限定符
	- 如何初始化
	- const 引用/指针
	- 拷贝 const 限定符时底层 const 的限制
	- constexpr
- 类型别名
- 自动推断类型

## 变量
**变量**或者**对象**，意味着一个具有某种数据类型的内存空间。其定义的基本形式是**类型说明符**+一个或多个变量名组成的列表。

### 初始化

*初始化与赋值是不同的*：变量**初始化**是指在创建时获得一个初始的值；而赋值指的是擦除变量当前值（因此需要考虑是否具有擦除权限）随后以新值替代。

**列表初始化**：如果希望指定的变量在自动转化时避免丢失信息（例如从 `double` 存储在 `float`），可以以下方式拒绝执行初始化命令：
```cpp
long double ld = 3.1415926
int a(ld), b = ld; // 转换正常执行
int c{ld}, d = {ld}; // 转换不执行，因为会丢失信息
```

### 声明

*在不同文件中使用同一变量*：C++ 支持**分离式编译**，文件的变量可以在别处被定义，但前提是程序需要**声明**变量，方法是添加 `extern` 关键字且不能显式初始化：
```cpp
extern int iz; // 声明
extern int ip = 1024; // 声明并定义
```


## 复合类型

> 当有多个类型时，从右往左阅读变量的定义，例如 `int *& var` 意味着（对（指向整型变量）指针）的引用
### 引用

**引用 (reference)** 即别名：引用只是为一个已经存在的对象起了另外一个名字，引用类型引用另外一种类型。引用并非对象。

```c++
int ival = 1024;
int &refVal = ival; // refVal 指向 ival，即是 ival 的另外一个名字。
```

注意事项：
- 定义引用时，程序把引用和它的初始值 *绑定* 在一起（意味着调用时两者等价），而不是将初始值 *拷贝* 给引用。
- 无法令引用重新绑定到另外一个对象。
- 因此，引用必须初始化。
- 引用不能为字面值或某个表达式的计算结果绑定在一起。（不然可以通过引用来改变常量的值）

以下是一些常见错误。

```c++
int &refVal1; // 错误，没有初始化。
int &refVal2 = 100; // 错误，引用类型的初始值必须是对象。
double dval = 3.14;
int &refVal3 = dval; // 错误，引用类型初始值必须是 int 型对象。
```
### 指针

**指针 (pointer)** 基本同 C 语言的指针。

*赋值改变的是等号左边的对象*：`pi = &ival` 意味着指针的值被改变，而 `*pi = 0` 意味着指针指向的对象的值被改变。

与普通指针必须类型相匹配不同，`void *` 指针可以指向任何对象的地址，但与此同时这个指针也失去了操作变量的能力，只能做指针比较、作为函数输入输出等。

*指针是对象*，而引用不是：因此不存在指向引用的指针，但存在指针的引用：
```cpp
int *p, i = 42;
int *&r = p; // r 是对 p 指针的引用
r = &i; // 等价于 p = &i;
*r = 0; // 等价于 *p = 0; 也就是 i = 0
```


## const 限定符

const 限定符用于我们希望值不能被改变的变量。因为其值不能被改变，因此 *const 对象必须被初始化。*

**const 类型的对象的主要限制**：只能在该对象上执行不改变其内容的操作。

```c++
const int bufSize = 512;
const int i = get_size(); // 正确，可以是任意复杂的表达式。
```

由于 `const` 对象创建后值不再改变，因此 **`const` 对象创建后必须初始化**。

可以用 const 对象赋值非 const 对象，但反之不然。

在多个文件中出现的 `const` 对象：

- 若程序包含多个文件，为了满足每个文件在调用该对象时都能访问到他的初始值，为了防止重复定义，**`const` 对象仅在文件内有效，不同文件的同名对象实际上是不同的对象**。
- 当我们希望一个对象在文件之间共享 `const` 属性时，解决的方法是：**无论是定义还是声明对象，都添加 `extern` 关键字**。

```cpp
const int bufSize; // 错误，没有初始化
// file1.cc 定义并初始化常量，且能被其他文件访问
extern const int bufSize = fcn()
// file1.h 声明，且与前者变量是同一个
extern const int bufSize;
```

### const 的引用

**常量引用** 是对 const 变量的引用。本质上，引用本身就是常量，因为一旦绑定之后不可再改变。

⚠️ **无法让一个非常量引用指向一个常量对象**，否则就可以通过引用修改 const 变量。

常量引用仅对引用可参与的操作进行限定，引用“自以为是”其引用的变量是常量，而实际上被引用的变量可以是非 const.

```cpp
const int ci = 1024, int cj = 24;

const int &ri = ci; // 正确
int &rj = ci; // 错误，非常量引用无法指向 const 对象
const int &rk = cj; // 正确，引用对象可以非 const
```

### const 的指针

**指向常量的指针**不能用于改变其对象的值，同常量引用；**常量指针**指指针作为一个变量，其值（即存放在指针中的那个地址）无法再修改，将一直指向那个位置。

无法通过**指向常量的指针**修改其存放的对象的值，而对象本身可以不是一个常量。

常量指针不与常量引用对应，与之对应的是指向常值的指针，这点也可以从缩写中看出来。

```cpp
int errNumb = 0;  

const int * curErr = &errNumb; // curErr 无法修改 errNumb 的值
int *const currErr = &errNumb; // currErr 将一直指向 errNumb，常量指针
const int *const currrErr = &errNumb; // 指向常量的常量指针，同时满足以上两点
```

### 顶层 const 和底层 const

指针本身是一个变量，（1）指针本身是不是常量（2）指针指向的对象是不是常量是两个独立的问题。

对于任意对象而言，**顶层 const** 指其本身（比如指针本身）是常量，**底层 const** 指其基本类型（比如指针指向的对象）是常量。

**拷贝需要注意底层 const 限制**：当拷贝时，底层 const 限制不能忽视——不能将指向常量变量的指针拷贝给指向非常量变量的指针。

举例说明：
```cpp
int i = 42;
const int *cp = &i, &r = i;
int *p = cp; // 错误，cp 有底层 const，不能赋值给非常量。
int &r3 = r; // 错误，理由同上。

int ci = 42;
const int *p1 = &ci;
const int *const p2 = p1;
p1 = p2; // 正确，顶层 const 不影响
```

### 常量表达式和 constexpr

**常量表达式 (const expression)** 指值不会改变且编译时即可以得到计算结果的表达式。函数运算结果的值一定不是常量表达式。

可以在编译时就得到计算结果的类型称之为**字面值类型**，算术类型、引用和指针属于字面值类型，而像 string 类型就不属于字面值类型。


```cpp
const int max_files = 20;  
const int limit = max_files + 1;  
int staff_size = 27; // 不是常量表达式，因为不是常值  
const int sz = get_size(); // 不是常量表达式，因为不能在编译时得到
```

如果认定变量是常量表达式，就将其声明为 `constexpr`，来验证其值是否是常量表达式。

⚠️ `constexpr` 将对象设置为顶层 `const`，因此 `const int *` 不等于 `constexpr int *`. 前者是指向常量的指针，后者是常量指针。

新定义规定新的 `constexpr` 函数：

```cpp
constexpr int sz = size(); // 仅当 size() 是 constexpr 函数
```

## 类型别名

### 定义同义词

有以下几种方式来定义某种类型的同义词：

```cpp
typedef double wages; // wages 是 double 的同义词
using ii = int; // ii 是 int 的同义词
```

*复合类型别名的陷阱*：

```cpp
typedef char *pstring;
const pstring *ps = 0; // ps 是一个指针，其对象是【（指向char的）（常量指针）】
					   // 而不是 const char *，要分开来理解
```

### 自动推断类型

第一种是 `auto`

```cpp
auto item = val1 + val2; // 自动推断类型

// auto 会自动忽略顶层 const，因此如果需要需要显式指明
const int ci = 42;
const auto f = ci; // 如果希望 f 是常量， const 必不可少
```

第二种是 `decltype`：创建一个和某表达式/变量结果类型相同的变量

```cpp
decltype(f()) sum = x; // sum 是函数 f 的返回类型

// decltype 保留顶层 const
const int ci = 0;
decltype(ci) x = 0;

// 表达式内容是解引用操作，decltype 结果是引用
int i = 42, *p = &i, &r = i;
decltype(r + 0) b; // b 是 int，表达式结果
decltype(*p) c; // c 是 int&，必须初始化
```

## 自定义数据结构

```cpp
struct STURCT_NAME {
	std::string sth;
	// ...
}
```
## 抽象数据类型

### 可变长字符串：string

在实践中，字符序列往往是长度可变的，因此诞生 `string`. 

`string` 在标准库 `string` 头文件中定义，因此需要：

```cpp
#include <string>
using std::string;
```

略。

#### 定义和初始化

### 可变长集合：vector

#### 定义和初始化

我们需要一个能够存储给定类型的可变长的集合，且各元素都能被下标索引，因此诞生 `vector`.

`vector` 在 `vector` 头文件中定义，因此需要：

```cpp
#include <vector>
using std::vector;
```

`vector` 是一个类模版，我们需要提供 `vector` 所存放的类型以实例化类。`vector` 能容纳绝大部分类型的*对象*，但不能包含引用。

以下是一些定义和初始化方式：

```cpp
vector<T> v1(n, val); // 包含 n 个值都是 val 类型为 T 的变量 
vector<T> v2(v1); // 用 v1 元素的副本初始化 v2
```

#### 操作

以下是一些操作：

```cpp
v.push_back(t) // 向 v 的尾端添加值为 t 的元素
v[n] // 返回 v 中第 n 个位置的引用
v1 = v2 // 用 v2 的拷贝替换元素
<, <=, >, >= // 以字典方式比较

// 对 vector 内每个元素操作
for (auto &i : v)
	i *= i;
	
// 输出 vector 内所有元素
for (auto i : v)
	cout << i << " ";
```

⚠️所有的初始化和赋值都是拷贝副本后操作的。

注意事项：
- 如果循环体内有向 `vector` 对象添加元素的语句，则不可以使用范围 `for` 循环
