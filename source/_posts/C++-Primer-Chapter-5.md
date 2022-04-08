---
title: C++ Primer Chapter 5
date: 2022-02-13 00:21:09
tags: C++
categories: C++
mathjax: true
---

# 第五章 语句

## 5.1 简单语句

### 空语句

```cpp
; 		// 最好加上注释解释为啥是空语句
```

- 常用于循环的全部工作再条件部分就可以完成时，循环体则为一条空语句
- 如以下代码，将不断读入数据直至eof或者输入值等于`sought`

```c++
while(cin >> s && s != sought) ;	// 空语句
```



### 复合语句/块

- 用花括号括起的语句和声明序列称为**复合语句**，或者**块**

- 块不以分号结束
- **空块**指内部没有任何语句的一对花括号，空块等价于空语句



## 5.2 语句作用域

- 在`if`、`switch`、`while`、`for`语句的控制结构内定义的变量，只在语句内部可见

<!--more-->

## 5.3 条件语句

### 悬垂else

- else总是与离它最近且尚未匹配的if匹配

```c++
if (grade % 10 >= 3)
    if (grade % 10 > 7)
		lettergrade += '+';
else
    lettergrade += '-';
// 看起来else会与第一个if匹配，实则与第二个if匹配
// 使用花括号能避免以上问题
```



### switch语句

```c++
switch (expr) {
    case v1:
        //...
        break;
    case v2:
        //...
        break;
    default:
        ;
}
```

- switch语句首先对括号里的表达式求值，**表达式的值转换成整数类型**，并与每个case标签的值进行比较
- 如果表达式的值跟某个case标签的值匹配成功，程序从该标签后的第一条语句开始执行，直到达到了switch结尾或者遇到一条break语句为止

#### case标签

- case关键字和它对应的值一起被称为case标签，case标签必须是**整型常量表达式**
- 任何两个case标签的值不能相同
- default是一种特殊的case标签

#### switch内部的控制流

- 大多数情况，每个case标签最后都应加上一个break语句，否则它会执行后面的case标签中的语句，直至switch结尾或遇到一条break语句
- 如果刻意没写break语句，最后加上注释说明程序逻辑
- 虽然最后一个分支的break语句不是必须的，因为已经到达了switch结尾，但为了后续新增case分支不引起错误，最好还是加上break语句

#### default标签

- default标签是一种特殊的case标签
- 若switch语句以一个空的default标签结束，则必须在后面加上一条空语句或一个空块

#### switch语句内部的变量定义

- 应该在第一个case标签之前进行定义，保证所有变量都正确地被初始化

- 假如在某个case标签内进行定义，且之后的其他case分支又使用了该变量，则可能由于作用域规则该变量能够被定义，但是没有正确初始化（因为初始化语句所在的case分支被跳过了）

  ```c++
  switch (expr) {
      case v1:
          int i = 1;
          break;
      case v2:
          cout << i << '\n';
          break;
  }
  // 假如进入v2分支, 则变量i被定义, 但是初始化并未执行, 它的值是未定义的
  ```

- 更准确的表述是，C++规定，**不允许跨过变量的初始化语句直接跳转到该变量作用域内的另一个位置**

- 值得一提的是，假如在某个case分支内定义该变量，但不执行初始化，则不会引起错误，因为并没有跳过初始化语句。但最好还是在case标签前进行变量定义与初始化

  ```c++
  switch (expr) {
      case v1:
          int i;			// 只定义, 不初始化, 则i可在switch语句中使用, 不会引起错误
          break;
      case v2:
          cout << i << '\n';
          break;
  }
  ```

  

## 5.4 迭代语句

### 范围for语句

- 语法形式

  ```c++
  for (declaration : expression)
      statement
  ```

- expression必须是一个序列，如数组、vector、string等，其共同点是**拥有能返回迭代器的`begin()`和`end()`成员**

- declaration定义一个变量，可定义为auto类型，若需要修改序列中元素，则将变量声明成**引用类型**

#### 范围for语句的等价形式

```c++
for (auto beg = v.begin(), end = v.end(); beg != end; ++beg) {
    //...
}
```

#### 范围for语句运存了end()的值，不能在范围for语句中修改容器大小

- 不能在范围for语句中改变容器的大小，即增加或删除容器元素，因为**范围for预存了end()的值，改变容器大小导致end()无效**

### do while语句

- 基本形式

  ```c++
  do
      statement
  while (condition);
  ```

- 括号括起的条件后面需要**加上分号，以表示语句结束**

## 5.5 跳转语句

- break

- continue

- goto



## 5.6 try语句块和异常处理

### 异常检测和异常处理

- 异常处理机制为程序中**异常检测**和**异常处理**两部分的协作提供支持
  - **throw表达式**，异常检测部分使用throw表示遇到了无法处理的问题，我们说throw引发了异常
  - **try语句块**，异常处理部分使用try语句块处理异常，它以关键字try开始，并以一个或多个**catch子句**结束
  - **异常类**，用于在throw表达式和相关的catch子句之间传递异常的具体信息

### throw表达式

- throw后紧跟一个表达式，**表达式的类型就是抛出的异常类型**

  ```c++
  if (item1.isbn() != item2.isbn())
      throw runtime_error("Data must refer to same ISBN");
  ```

### try语句块

- 形式

```cpp
try {
    program-statements
} catch (exception-declaration) {
    handler-statements
} catch (exception-declaration) {
    handler-statements
}
```

- 函数在寻找处理代码的过程中退出
  - 异常被抛出时，首先搜索抛出异常的函数，若没找到匹配的catch子句则终止函数，并在调用该函数的函数中继续寻找匹配的catch子句，即**由内到外搜索匹配的catch子句**
  - 若最终找不到catch子句，则**转到名为terminate的标准库函数**，一般情况下会导致非正常退出。**对于没有try语句的异常，也将转到terminate**

### 标准异常

- 异常类头文件
  - `exception`头文件定义了最通用的异常类exception，只报告异常发生，不提供额外信息
  - `stdexcept`头文件定义了常用异常类
  - `new`头文件定义了`bad_alloc`异常类型
  - `type_info`头文件定义了`bad_cast`异常类型

- `stdexcept`头文件的异常类
  - exception
  - runtime_error，运行时错误
  - range_error，结果超出了有意义的值域范围
  - overflow_error，计算上溢
  - underflow_error，计算下溢
  - logic_error，程序逻辑错误
  - domain_error，参数对应的结果值不存在
  - invalid_argument，无效参数
  - length_error，试图创建一个超出该类型最大长度的对象
  - out_of_range，使用一个超出有效范围的值

- 只能以默认初始化形式初始化`exception`、`bad_alloc`、`bad_cast`对象，**只能以string或字符串字面值初始化其他异常对象**

- 异常类型只有**`what()`成员函数**，对于string或字符串字面值初始化的异常对象，返回该字符串；对于默认初始化的异常对象，返回类型与编译器有关

### 异常处理代码示例

- 输入两个数相除，若除数为0则抛出异常信息，提示用户重新输入

```c++
while (cin >> v1 >> v2) {
    try {
        if (v2 == 0)
            throw runtime_error("Divisor can't be zero, please re-enter the operands");		// throw语句抛出异常
        else {
            cout << "v1 divides v2 is " << v1 / v2 << endl; 
            break;
        }
    } catch (runtime_error err) {			// catch语句处理除数为0的异常
        cout << err.what() << endl;
    }
}
```

