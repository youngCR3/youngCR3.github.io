---
title: C++ Primer Chapter 17
date: 2022-02-22 21:19:13
tags: C++
categories: C++
mathjax: true
---

**C++ Primer第17章笔记**

<!--more-->

# 第17章 标准库特殊设施

## 17.1 tuple类型

- 当我们想要将一些数据组合成单一对象，却又不想定义新的数据结构时，可以考虑使用`tuple`
- `tuple`类型可以有**任意数量**的成员，且每个成员也可以是**任意类型**的

### 定义和初始化`tuple`

- **默认构造函数**，对每个成员进行**值初始化**

  ```c++
  tuple<int, double, string> t;
  // t的三个成员分别被初始化为0, 0.0, 空string
  ```

- 为每个成员提供一个初始值，必须使用直接初始化

  ```c++
  tuple<int, double, string>{1, 3.14, "hello"};
  ```

- 使用`make_tuple`，使用初始值的类型推断`tuple`的类型，可直接通过`auto`定义`tuple`

  ```c++
  auto t = make_tuple(1, 3.14, "hello");
  ```

### 访问tuple的成员

- 使用标准库函数模板`get`，必须**传入显式模板实参**，指出**想要访问第几个成员**，从0开始编号，传入的模板实参必须是**整型常量表达式**

- 传递给`get`一个`tuple`对象，它将**返回指定成员的引用**

  ```c++
  tuple<int, double, string> t;
  get<2>(t);		// 访问t的第三个成员
  ```

- 通过`tuple_size`模板类的`value`成员，我们可以**获取`tuple`成员的数量**，需要传入`tuple`类型作为模板实参，最简单的方法就是使用`decltype`

  ```c++
  // tuple_size<tupleType>::value
  tuple<int, double, string> t;
  tuple_size<decltype(t)>::value;		// 表达式的值为3
  ```

- 通过`tuple_element`模板类的`type`成员，我们可以**获取某个成员的类型**，需要传入第几个成员、`tuple`类型作为模板实参

  ```c++
  // tuple_element<i, tupleType>::type
  tuple<int, double, string> t;
  tuple_element<2, decltype(t)>::type s = "hello";		// 获取的类型为string 
  ```

### 关系和相等运算符

- 两个`tuple`必须具有**相同数量的成员**，且**每对成员**使用**`==`运算符**都是**合法**时，我们才可以使用`tuple`的**相等或不等运算符**
- 类似地，两个`tuple`必须具有相同数量的成员，且每对成员使用**`<`运算符**都是合法时，我们才可以使用`tuple`的**关系运算符**

- 可以在**无序容器**中将`tuple`作为**关键字类型**

### 使用tuple返回多个值

- `tuple`的一个重要作用是从函数返回多个值

## 17.2 bitset类型

- `bitset`类型使得位运算更为容易，能够处理**超过最长整型类型大小的位集合**

### 定义和初始化`bitset`

- `bitset`具有固定的大小，定义`bitset`时需要通过**常量表达式**指明其**大小**

- `bitset`中的二进制位从0开始编号，从0开始的二进制位称为**低位**，编号到最后一位称为**高位**

- 默认初始化，`bitset`中所有位都设为0

- 用`unsigned long long`初始化，当我们用整型初始化`bitset`，整型将被转换为`unsigned long long`类型

  - 如果`bitset`大小大于`unsigned long long`的二进制位树，高位将置为0

- 用`string`初始化`bitset`

  ```c++
  string s = "111000";
  bitset<6> b(s);
  ```

### bitset操作

```c++
b.any();		// 存在任意置位的二进制位则返回true
b.all();
b.none();
b.count();		// b中置位的位数
b.size();		// b的位数

b.test(pos);	// pos位置是否置位
b.set(pos);		// 将pos置位
b.set();		// 将所有位置位
b.reset(pos);	// 将pos复位
b.reset();		// 将所有位复位
b.flip(pos);	// 将pos位反转
b.flip();		// 将所有位反转

b.to_ulong();	// 将b转为unsigned long, 若放不下会抛出异常
b.to_ullong();	// 将b转为unsigned long long, 若放不下会抛出异常

os << b; 		// 打印b
```

## 17.3 正则表达式

- 略

## 17.4 随机数

- 定义在头文件`random`的随机数库通过一组协作的类生成随机数
  - **随机数引擎类**，生成**随机`unsigned`整数序列**
  - **随机数分布类**，使用一个引擎类生成**指定类型**的、在**给定范围**内的、**服从特定概率分布**的随机数

### 给定的随机数发生器会一直生成相同的随机数序列

- 一个函数如果定义了**局部的随机数发生器**，应该将其**引擎和分布**对象都**定义为`static`**
- 否则，每次调用函数都会生成相同的序列

### 随机数发生器种子

- 通过**种子**，可以使得同一个随机数发生器**生成不同的序列**
- 但是种子的选择本身也是一个问题，常用方法是利用**当前时间**等

### 生成一定范围内的均匀分布的随机整数

```c++
default_random_engine e;
uniform_int_distribution<int> u(0, 9);
cout << u(e) << endl;
```

### 生成一定范围内的均匀分布的随机实数

```c++
default_random_engine e;
uniform_real_distribution<double> u(0, 1);
cout << u(e) << endl;
```



