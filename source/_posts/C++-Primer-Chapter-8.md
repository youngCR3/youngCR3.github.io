---
title: C++ Primer Chapter 8
date: 2022-02-15 11:05:10
tags: C++
categories: C++
mathjax: true
---

# 第八章 IO库

## 回顾IO库设施

- `istream`类型，提供输入操作
- `ostream`类型，提供输出操作
- `cin`，`istream`对象，从标准输入读取数据
- `cout`，`ostream`对象，向标准输出写入数据
- `cerr`，`ostream`对象，用于输出程序错误信息，写入到标准错误
- `>>`运算符，用于从一个istream对象读取输入数据
- `<<`运算符，用于从一个ostream对象写入输出数据
- `getline`函数，从一个给定的istream读取一行数据，存入一个给定的string对象中

## 8.1 IO类

### 三种IO类型

- `iostream`定义用于**读写流**的基本类型，包含`istream`和`ostream`
- `fstream`定义用于**读写命名文件**的类型，包含`ifstream`和`ofstream`
- `sstream`定义了**读写内存string对象**的类型，包含`istringstream`和`ostringstream`
- 以上类型都用于处理char数据，为了支持宽字符类型，以上每种IO类型都对应的宽字符版本，用于处理`wchar_t`类型的数据

#### IO类型之间的关系（继承）

- 类型`ifstream`和`istringstream`都**继承**自`istream`，我们可以像使用`istream`对象一样使用`ifstream`和`istringstream`对象

### IO对象无拷贝或赋值

- 不能拷贝或对IO对象赋值

- 因为不能拷贝IO对象，因此**不能将函数形参或返回类型设置为流类型**，只能以引用的方式传递IO对象

- 另外，读写IO对象会改变其状态，因此传递和返回的引用不能是const的

  ```c++
  ofstream out1, out2;
  out1 = out2;		// 非法, 不能用一个io对象赋值给另一个io对象
  print(out1);		// 非法, 不能拷贝io对象
  ```

### 条件状态

- IO操作可能发生错误，**IO类提供了一些函数与标志**，用于访问和操纵流的条件状态

- 条件状态

  - `strm::iostate`，提供了表达条件状态的完整功能
  - `strm::badbit`，指出**流已崩溃**
  - `strm::failbit`，指出一个**IO操作失败**
  - `strm::eofbit`，指出流到达了文件结束
  - `strm::goodbit`，指出流未处于错误状态，保证为0
  - `s.eof()`，若流s的`eofbit`置位则为true
  - `s.fail()`，若流s的`failbit`或`badbit`置位则为true
  - `s.bad()`，若流s的`badbit`置位则为true
  - `s.good()`，若流s处于有效状态则返回true
  - `s.clear()`，将流s的所有条件状态复位，流的状态设置为有效，返回void
  - `s.clear(flags)`，根据给定的`flags`标志位，将流s的状态复位，`flags`的类型为`strm::iostate`，返回void
  - `s.setstate(flags)`，根据给定的`flags`标志位，将流s的状态置位
  - `s.rdstate()`，返回流s的当前条件状态，返回值类型为`strm::iostate`

- IO错误的例子

  ```c++
  int ival;
  cin >> ival;
  // 标准输入Boo, 读操作会失败, cin进入错误状态
  ```

### 在使用IO类型前检查其是否有效状态

- 直接将IO对象作为一个条件使用，可检查其是否有效状态

  ```c++
  while (cin >> val) {
      // 读操作成功
  }
  ```

### 查询流的状态

- 使用IO对象的`good()`方法和`fail()`方法
- 若所有错误位均未置位，则`good()`返回true
- 若`badbit`被置位或`failbit`被置位，则`fail()`返回true。另外，到达文件结尾时，`failbit`、`eofbit`都会被置位，因此`fail()`包括了所有错误
- 当我们直接使用IO对象作为条件时，等价于使用`!s.fail()`

### 管理条件状态

- 利用`rdstate()`和`flag()`方法，可以实现将部分标志位复位，其他不变的效果

- 如将`failbit`和`badbit`复位，而其他标志位不变，代码如下

  ```c++
  cin.clear(cin.rdstate() & (~cin.badbit) & (~cin.failbit));
  ```

- `clear()`不接受参数的版本，将清除所有错误标志位

### 管理输出缓冲

#### 缓冲区的作用

- 每个输出流都有一个缓冲区，用来保存程序读写的数据
- 缓冲区的作用：允许操作系统将程序的多个输出操作组合成单一的系统级写操作，可能带来**性能提升**

#### 缓冲刷新（数据真正写到输出设备或文件）的原因

- **程序正常结束**时，将缓冲刷新
- **缓冲区满**时
- 使用**操纵符`endl`**显式刷新缓冲区
- 用**操纵符`unitbuf`设置流的内部状态**，以清空缓冲区
- 一个输出流可能关联到其他流，这时**读写被关联的流时，关联的流的缓冲区将被刷新**。如`cin`和`cerr`都被关联到`cout`，因此读`cin`或写`cerr`都会导致`cout`的缓冲区被刷新

#### 显式刷新输出缓冲区的操纵符

- **`endl`换行**并刷新缓冲区
- **`flush`**刷新缓冲区，但**不输出任何额外字符**
- **`ends`向缓冲区插入一个空字符**，刷新缓冲区

#### `unitbuf`和`nounitbuf`操纵符

- `unitbuf`操纵符告诉流在接下来的每次写操作后都进行依次flush操作
- 而`nounitbuf`操纵符则重置流，恢复正常的系统管理的缓冲区刷新机制

#### 如果程序崩溃，输出缓冲区将不会被刷新

- 程序异常终止，输出缓冲区不会刷新。因此调试时，你可能会因为没有输出，误以为程序没有执行相应的代码，但实则程序已经执行代码，但在输出前崩溃了

### 关联输入流和输出流

- `cin`被关联到`cout`，这意味着所有读取标准输入的操作前都会先刷新`cout`的缓冲区
- 交互式系统通常应该关联输入流和输出流

#### `tie`方法

- `s.tie()`，假如s被关联到一个输出流，将返回指向该输出流的指针；若s未被关联到输出流，将返回空指针
- `s.tie(ostrem*)`，将自己关联到指针指向的`ostream`，返回指向s原来关联的ostream的指针，若无则返回空指针
- 每个流最多同时关联到一个ostream，但同一个ostream可以被多个不同的流关联

<!--more-->

## 8.2 文件输入输出

### 文件IO类型

- `ifstream`从给定文件中读取数据
- `ofstream`向一个给定文件写入数据
- `fstream`可以读写给定文件

### fstream特有的操作

- 除了继承自`iostream`类型的行为外，`fstream`还有一些特有操作

### 使用文件流对象

#### 创建文件流对象

```c++
fstream fstrm;
fstream fstrm(s);
fstream fstrm(s, mode);
```

- 第一行定义一个文件流对象
- 第二行定义一个文件流对象，并**将其与名为`s`的文件关联**，创建文件流对象时将自动调用其`open`方法打开该文件，`s`可以是**string对象或指向C风格字符串的指针**
- 第三行定义一个文件流对象，将其与名为`s`的文件关联，并以`mode`方式打开文件

#### 打开文件

- 假如创建文件流对象时使用了第二或第三种，将自动打开文件
- `fstrm.open(s)`，打开名为`s`的文件，并将文件与`fstrm`绑定，**返回void**

#### 成员函数`open`和`close`

- 调用`open`可以将文件流对象与文件关联，并打开该文件，`fstrm.open(s)`

- **如果调用open失败，则`failbit`会被置位**，因此最好在调用open后进行检查open是否成功的检测

  ```c++
  fstrm.open(s);
  if (fstrm) {}	// 检测fstrm调用open是否成功
  ```

  - 注意open的返回值是void，因此应该在调用后直接将文件流对象作为条件进行判断

- **对一个已经打开的文件流调用open会失败**，并会导致`failbit`被置位

- 假如文件流对象已经打开了一个文件，则想让它打开另一个文件前，应**先关闭已经关联的文件**

  ```c++
  fstrm.close();		// 关闭fstrm绑定的文件, 返回void
  ```

#### `is_open()`

- 返回bool值，指出与 fstrm关联的文件是否成功打开并且尚未关闭
- 假如该文件流对象**未关联任何文件，将返回false**

### 自动构造和析构

- fstream对象被销毁时，其**close方法将被自动调用**

### 文件模式

- 每个流都有关联的**文件模式，指出如何使用文件**

  - `in`，以读方式打开
  - `out`，以写方式打开
  - `app`，每次写操作前均定位到文件末尾

  - `ate`，打开文件后立刻定位到文件末尾
  - `trunc`，截断文件
  - `binary`，以二进制方式进行IO

- 调用**open**打开文件或**用文件名初始化流对象**时，我们都可以指定文件模式

- 文件模式的限制

  - 只可以对`ofstream`或`fstream`对象设定out模式
  - 只可以对`ifstream`或`fstream`对象设定in模式
  - 当`out`被设定时，才可以设定`trunc`
  - 只要`trunc`没被设定，就可以设定`app`模式，`app`模式下文件总是以输出方式被打开
  - `out`模式打开的文件默认地被截断，若想保留以`out`方式打开的文件内容，必须同时指定`app`模式，或同时指定`in`模式
  - `ate`和`binary`可以用于任何类型的文件流对象，且可以与任何其他模式组合使用

#### 每个文件流类型都有默认的文件模式

- 当我们创建文件流对象时，若不指定文件模式，将使用该默认的文件模式
- `ifstream`关联的文件默认以`in`模式打开，`ofstream`关联的文件默认以`out`模式打开，`fstream`关联的文件默认以`in`和`out`模式打开

#### 以out模式打开的文件默认地丢弃已有数据

- 想要保留`ofstream`打开的文件中已有数据，则必须指定`app`模式或`in`模式

#### 使用位或运算符`|`指定多个文件模式

- 通过按位或运算符`|`可以指定多个文件模式

  ```c++
  std::ofstream fout("a.txt", std::ofstream::out | std::ofstream::app);
  ```

## 8.3 string流

- `sstream`头文件定义了三个IO类型，用于向string对象写入数据或读取数据，就像string是一个IO流一样
  - `istringstream`从string读取数据
  - `ostringstream`将数据写入string
  - `stringstream`既可以从string读数据也可向string写数据

### stringstream的操作

- `sstream strm`，定义一个`stringstream`对象，它是未绑定的
- `sstream strm(s)`，定义一个`stringstream`对象，**保存string对象s的一个拷贝**
- `strm.str()`，返回其保存的string对象的拷贝
- `strm.str(s)`，**将string对象拷贝到strm中，返回void**

## 习题

- 编写函数，接收istream&，从给定流中读取数据，直到遇到文件结束，将读取的数据打印到标准输出，然后对流进行复位使其位于有效状态，最后返回istream&

  ```c++
  std::istream& f(std::istream& is) {
      std::string s;
      while (is >> s) {
          std::cout << s << std::endl;
      }
      is.clear();
      return is;
  }
  ```

  

- 编写函数，以读模式打开文件，将其内容读入到`vector<string>`中，将每行作为独立元素存于vector对象中

  ```c++
  void f() {
      std::ifstream fin("a.txt");
      std::vector<std::string> a;
      std::string s;
      while (getline(fin, s)) {
          a.push_back(s);
      }
      for (auto &s: a)
          std::cout << s << std::endl;
  }
  ```

- 重写上述函数，将每个单词存于vector对象中

  ```c++
  void f() {
      std::ifstream fin("a.txt");
      std::vector<std::string> a;
      std::string s;
      while (fin >> s) {
          a.push_back(s);
      }
      for (auto &s: a)
          std::cout << s << std::endl;
  }
  ```

  
