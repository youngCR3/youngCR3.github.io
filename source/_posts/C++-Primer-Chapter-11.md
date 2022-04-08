---
title: C++ Primer Chapter 11
date: 2022-02-17 15:29:54
tags: C++
categories: C++
mathjax: true
---

# 第11章 关联容器

- 关联容器支持高效的关键字查找和访问
- C++标准库提供了8个关联容器，其不同体现在了三个维度上
  - `map`或者`set`
  - 是否允许**重复关键字**
  - **顺序**保存元素，或**无序**保存
- `set`, `map`, `multiset`, `multimap`, `unordered_set`, `unordered_map`, `unordered_multiset`, `unordered_multimap`

<!--more-->

## 11.2 关联容器概述

### 定义关联容器

- 定义一个`map`时，必须同时指明**关键字类型和值类型**
- 定义一个`set`时，只需指明**关键字类型**

#### 构造函数

- 每个关联容器都有一个**默认构造函数**，创建一个指定类型的空容器

- 将关联容器初始化为**另一个同类型容器的拷贝**
- 提供**一对迭代器**，从该范围初始化关联容器
- 值初始化，提供**初始值列表**。使用此方法初始化一个`map`时，每个**关键字-值对**被包围在一对花括号中`{key, value}`
- `multiset`和`multimap`的初始化同样如此

### 关键字类型的要求

- 对于**有序容器**，关键字类型必须**定义元素比较的方法**，默认情况下有序容器**使用关键字类型的`<`运算符**比较两个关键字

- 提供自**己定义的操作来代替关键字上的`<`运算符**，要求所提供的操作必须在关键字类型上**定义一个严格弱序（strict weak ordering）**
  - 两个关键字不能同时“小于等于”对方
  - 如果`k1`小于等于`k2`，且`k2`小于等于`k3`，那么`k1`必须小于等于`k3`
  - 如果存在两个关键字，任何一个都不小于等于另一个，那么容器将它们视作相等来处理

#### 自定义比较操作

- **组织一个容器中元素的操作的类型也是该容器类型的一部分**，若使用自定义操作，则必须在定义关联容器类型时也提供此操作的类型（如传入一个比较函数，则为函数指针类型）

- 尖括号指定元素类型，**自定义操作类型紧跟元素类型给出**

- 当我们创建该类型关联容器的对象时，才真正提供该比较操作的实参，其类型应与定义的类型一致

- 函数指针类型，可以通过`decltype`简化代码

  ```c++
  set<Sales_data, bool(*)(const Sales_data&, const Sales_data&)> c(compareIsbn);
  set<Sales_data, decltype(compareIsbn)*> c(compareIsbn);
  ```

### pair类型

- 名为`pair`的标准库类型，定义在`utility`头文件中
- 一个`pair`保存两个数据成员，`pair`也是用于生成特定类型的模板，创建`pair`时必须提供两个类型名

#### 构造函数

- **默认构造函数**对数据成员进行**值初始化**
- 可以提供**初始值列表**进行初始化
- **接受两个参数**的构造函数，分别用于初始化两个数据成员

#### 数据成员

- `pair`的数据成员是`public`的
- 包含两个数据成员，分别是`first`和`second`

#### `make_pair(v1, v2)`

- 返回一个用`v1`和`v2`初始化的pair，`pair`的类型从`v1`和`v2`的类型推断出来

#### 返回一个`pair`

- 显式创建一个`pair`对象并返回
- 对返回值进行列表初始化，即直接`return {v1, v2};`

## 11.3 关联容器操作

### 关联容器的类型别名

- `key_type`，关联容器的关键字类型
- `mapped_type`，每个关键字关联的类型，只适用于`map`
- `value_type`
  - 对于`set`，与`key_type`相同
  - 对于`map`，为`pair<const key_type, mapped_type>`

### 关联容器迭代器

- 解引用一个关联容器迭代器，将得到一个类型为关联容器`value_type`的值的引用
- `set`的迭代器是`const`的，无法修改其中元素的值

- `map`和`set`类型都支持`begin`和`end`操作，可用于遍历关联容器 

- **通常不对关联容器使用泛型算法**，因为其迭代器是const的，无法修改或重排元素

- 使用泛型算法的情景
  - 将关联容器作为源序列，传入`copy`算法
  - 将关联容器作为目的序列，通过`inserter`将一个插入器绑定到一个关联容器

### 添加元素

#### `insert` 的三个版本

- 接受**一个值**，或**一个迭代器对**，或**一个初始化器列表**
  - `c.insert(v)`
  - `c.insert(b, e)`
  - `c.insert(li)`

#### `emplace(args)`

- 用`args`构造一个元素并插入关联容器

#### `insert(v)`或`emplace(args)`的返回值

- 对于不允许重复关键字的关联容器，接受单一元素的`insert`或`emplace`版本返回一个`pair`
  - `first`是**指向具有给定关键字元素的迭代器**
  - `second`是`bool`值，指示**插入是否成功**
- 假如关联容器已有该关键字，则什么也不做，且`second`为`false`；否则将关键字插入其中，且`second`为`true`

- 对于允许重复关键字的关联容器，如`multiset`，插入操作总是成功，**返回一个指向新元素的迭代器**

### 删除元素

- `erase(it)`，删除迭代器`it`指向的元素，返回`void`
- `erase(b, e)`, 删除迭代器对$[b,e)$指定的范围，返回`void`

#### `erase(k)`

- 删除**所有**匹配关键字`k`的元素，返回实际删除的数量

### map的下标操作

- `map`和`unordered_map`支持下标操作，`multimap`**不支持**下标操作
- 可使用下标运算符，或`at`成员函数，传入关键字`k`，返回关键字为`k`的元素
- 下标运算符和`at`成员函数的区别：**如果关键字`k`不在其中**
  - 使用**下标运算符**，将**添加关键字为`k`的元素将对其进行值初始化**
  - 使用`at`，将抛出一个out_of_range异常

- 返回值
  - **下标操作**的返回值是一个`mapped_type`的对象的引用
  - **解引用一个map迭代器**，将得到一个`value_type`对象

### 访问元素

- `lower_bound`和`upper_bound`不适用于无序容器

#### `find`

- `c.find(k)`返回指向第一个关键字为k的元素的迭代器

#### `count`

- `c.count(k)`返回关键字为k的元素的数量

#### `lower_bound`

- `c.lower_bound(k)`返回指向第一个关键字**不小于k**的元素的迭代器

#### `upper_bound`

- `c.upper_bound(k)`返回指向第一个关键字**大于k**的元素的迭代器

#### `equal_range`

- `c.equal_range(k)`返回一个迭代器对，代表关键字**等于k**的元素的范围，若容器中不含关键字为k的元素，则返回的迭代器对都指向`c.end()`

### 常见错误：在元素不存在时删除元素

- 错误写法

  ```c++
  c.erase(c.find(k));
  ```

- 假如`k`不存在于关联容器`c`中，则`find`将返回`c.end()`
- 将`c.end()`传入关联容器的`erase`成员函数会**导致错误**

- 正确写法

  ```c++
  if (c.find(k) != c.end())
      c.erase(c.find(k));
  ```

  

## 11.4 无序容器

- 无序容器不是使用比较运算符来组织元素，而是使用一个**哈希函数**和**关键字类型的`==`运算符**

- 标准库定义了四个无序容器：`unordered_map`, `unordered_set`, `unordered_multiset`, `unordered_multimap`

### 无序容器对关键字类型的要求

- 无序容器要求关键字类型的`==`运算符，同时要使用`hash<key_type>`类型的对象生成每个元素的哈希值
- 标准库只对内置类型，以及`string`、智能指针类型定义了`hash`

#### 为自定义类型提供函数代替`==`和哈希计算函数

- 与定义`map`时为自定义类型提供比较函数类似，在关键字类型和值类型后，**紧跟哈希函数指针、相等性判断运算符指针**

  ```c++
  size_t hasher(const Sales_data& sd) {
      return hash<string>()(sd.isbn());
  }
  
  bool eqOp(const Sales_data *lhs, const Sales_data &rhs) {
      return lhs.isbn() == rhs.isbn();
  }
  
  unordered_set<Sales_data, decltype(hasher)*, decltype(eqOp)*> s(42, hasher, eqOp);
  // 参数分别是桶大小、哈希函数指针和相等性判断运算符指针
  ```

### 管理桶

- 无序容器在存储上组织为一组桶，每个桶保存零或多个元素
- 使用哈希函数将元素映射到桶，具有一个特定哈希值的所有元素被保存到相同的桶中

#### 管理桶的函数（看看就好，需要理解哈希表存储，并阅读源码才懂）

- 桶接口
  - `bucket_count()`返回正在使用的桶的数目
  - `max_bucket_count()`返回能容纳的最多的桶的数量
  - `bucket_size(n)`返回第n个桶中有多少个元素
  - `bucket(k)`返回关键字k在哪个桶中
- 桶迭代
  - `local_iterator`用于访问桶中元素的迭代器类型
  - `const_local_iterator`桶迭代器的const版本
  - `c.begin(n)`, `c.end(n)`返回第n个桶的首元素迭代器和尾后迭代器
- 哈希策略
  - `c.load_factor()`，每个桶的平均元素数量，返回`float`
  - `c.max_load_factor()`，试图维护的平均桶大小
  - `c.rehash(n)`，重组存储，使得`bucket_count>=n`且`bucket_count > size / max_load_factor`
  - `c.reserve(n)`，重组存储，使得`c`可以保存`n`个元素且不必rehash

### 无序容器相对于有序容器的优缺点

- 优点：理论上能获得**更好的平均性能**，实际需要进行一些性能测试和调优工作 

- 缺点：无法排序
