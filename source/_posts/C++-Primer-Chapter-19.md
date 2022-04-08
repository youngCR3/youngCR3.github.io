---
title: C++ Primer Chapter 19
date: 2022-02-24 13:33:20
tags: C++
categories: C++
mathjax: true
---

**C++ Primer第19章笔记**

<!--more-->

# 第19章 特殊工具与技术

## 19.1 控制内存分配

### 19.1.1 重载new和delete

#### new和delete关键字干了什么？

- **new**表达式的**工作原理**

  ```c++
  string *sp = new string("a value");
  string *arr = new string[10];
  ```

  - `new`表达式调用了一个**名为`operator new`或者`operator new[]`的标准库函数**，**分配**一块足够大、**原始的、未命名的内存空间**以存储特定类型的对象（或对象数组）
  - 运行相应的构造函数以**构造这些对象**，传入初始值
  - 对象被分配空间并构造完成，**返回指向该对象的指针**

- **delete**表达式的**工作原理**

  ```c++
  delete sp;
  delete[] arr;
  ```

  - 对`sp`所指对象或者`arr`所指的数组中的元素执行相应的**析构函数**
  - 调用名为`opreator delete`或者`opreator delete[]`的标准库函**数释放内存空间**

- 编译器对new或delete的解析
  - 如果被分配或释放的对象是类类型，则编译器首先在**类及其基类的作用域**内查找可调用的`operator`函数
  - 若没找到，则在**全局作用域**查找匹配的函数
  - 若没找到，则调用**标准库**定义的版本

#### 重载operator new和operator delete

- 需要注意的是，调用opreator new只是new关键字所做工作的第一步，它会分配一段原始内存，我们**无法改变new关键字所做的工作**，只能通过重载operator new**改变其分配内存的方式**，operator delete同理

- **operator new**

  - 对于`opreator new`或者`opreator new[]`，必须**返回**`void*`类型，**第一个参数**的类型必须是`size_t`，代表**存储对象所需的字节数**，或存储对象数组中所有元素所需的字节数
  - 使用`opreator new`的自定义版本时，`new`表达式必须使用**`new`的定位形式**，**将实参传给新增的形参**
  - 用户**不能重载**以下版本，它只供标准库使用

  ```c++
  void *opreator new(size_t, void*);
  ```

- **operator delete**

  - 对于`opreator delete`，其返回类型必须是`void`，第一个参数必须是`void*`，允许有另外一个类型为`size_t`的形参，表示第一个形参所指对象的字节数
  - operator delete不允许抛出异常，原因与析构函数不允许抛出异常的原因一致

- `opreator new `和`opreator delete`定义为类的成员时，是**隐式静态**的，因为`opreator new`用在**对象构造之前**，而`opreator delete`用在**对象销毁之后**，因此必须是静态的，此外它们**不能操纵**任何类的**数据成员**

#### malloc和free

- `malloc`接受一个表示**待分配字节数**的`size_t`，**返回指向分配空间的指针**或返回**0表示分配失败**
- `free`接受一个`void*`，它是`malloc`返回的指针的副本，它将相关**内存返回给系统**

- operator new和operator delete的一种简单实现方式

  ```c++
  #include <cstdlib>
  
  void* operator new(size_t sz) {
      void* p = malloc(sz);
      if (p)
      	return p;
      else
          throw bad_alloc();
  }
  
  void operator delete(void* p) noexcept {
      free(p);
  }
  ```

  

### 19.1.2 定位new表达式

- 用户代码也可以直接调用`opreator new`函数，它将完成内存分配的工作，并返回指向该原始内存的指针
- 定位new表达式允许我们传入参数，它将根据传入的参数调用相应的`opreator new`版本进行内存分配

#### 定位new表达式传入单一地址参数

- 特别地，如果我们传入的是**单一的地址**，它将调用`void* operator new(size_t, void*)`
- 这是一个我们**无法自定义**的operator new版本，它将**简单地返回指针实参**，然后由**new表达式**继续**在指定的位置构造对象**
- 通过这种形式的定位new表达式，我们可以在一段**先前分配的内存**上**构造对象**

- 使用定位new表达式的目的和allocator的construct成员相似，都是在一段先前分配的内存上构造对象，不同之处在于**construct只能传入由同一个allocator分配的内存**，而**定位new表达式**接受的指针**不一定要指向operator new分配的内存**（甚至**不一定指向动态内存**）
- 当使用**只有一个指针类型形参**的**定位new表达式**时，它只**构造对象**，而**不分配内存**

#### 显式的析构函数调用

- 与调用类的普通成员函数一样，可以通过类对象、对象的指针或引用调用
- 调用**析构函数**只**销毁对象**，**不释放内存空间**

## 19.2 运行时类型识别

- 运行时类型识别**（run-time type identification, RTTI）**的功能由**两个运算符**组成
  - `typeid`运算符，返回**表达式的类型**
  - `dynamic_cast`运算符，用于将**基类指针或引用**安全地转换为**派生类的指针或引用**

- 当我们将这两个运算符用于**某种类型的指针或引用**，且该**类型含有虚函数**时，它将使用指针或引用**所绑定对象的动态类型**
- 使用场景：想使用**基类对象的指针或引用**执行某个**派生类操作**，但该操作**不是虚函数**时

> 在可能的情况下，最好定义虚函数，而不要使用RTTI运算符

### 19.2.1 `dynamic_cast`运算符

- `dynamic_cast`运算符的**三种形式**

  ```c++
  dynamic_cast<type*>(e);		// e必须是一个有效指针
  dynamic_cast<type&>(e);		// e必须是一个左值
  dynamic_cast<type&&>(e);	// e不能是左值
  ```

- **e的类型**必须满足以下三种的任意一个：

  - e的类型是目标type的**公有派生类**
  - e的类型是目标type的**公有基类**
  - e的类型**就是**目标type的**类型**

- 但是，**e所指向的对象类型**必须是**type类**或**type类的派生类**，而**不能是type类的基类**

- **指针类型**的`dynamic_cast`，如果类型转换**失败则返回0**

  ```c++
  if (Derived* dp = dynamic_cast<Derived*>(bp)) {
      // ... 使用do指向的Derived对象
  } else {
      // ... 使用bp指向的Base对象
  }
  ```

  在`if`条件中同时完成了类型转换和类型检查，若转换失败将返回0，从而进入else分支

- **引用类型**的`dynamic_cast`，如果类型转换**失败则抛出`bad_cast`异常**

  ```c++
  try {
      const Derived& d = dynamic_cast<const Derived&>(b);
  } catch(bad_cast) {
      // ...
  }
  ```

  

### 19.2.2 typeid运算符

- 形式为`typeid(e)`，e可以是任意表达式或类型名字，返回一个常量对象的引用，对象类型是标准库类型`type_info`
- 如果表达式e是一个**引用**，将得到**该引用所指对象的类型**
- 如果表达式e是一个**数组或函数**，得到的是**数组或函数的类型**，而非数组指针或函数指针类型
- 如果运算对象**不是类类型**，或该**类不含虚函数时**，将得到其**静态类型**。如果运算对象是**定义了至少一个虚函数**的**类类型的左值**时，typeid的结果会在**运行时求得**

- typeid作用于**指针**，而非指针指向的对象时，将得到该**指针的静态编译类型**

#### 使用typeid运算符

- 通常，通过typeid比较**两个表达式的类型是否相同**，或比较一条表达式的类型与指定类型是否相同

### 19.2.3 使用RTTI

- 为具有继承关系的类实现**相等运算符**。对于**两个对象**，只有它们**类型相同**，且对应的**数据成员值也相等**时，才说这两个对象是相等的

- 定义**相等运算符**的形参是**基类的引用**，使用**typeid**检查**两个运算对象的类型**是否一致，若不一致则返回false

- 若一致，则**调用虚函数`equal`**。每个类的`equal`函数负责**比较类型自己的成员**，**接受`Base&`参数**，并在比较前**先用`dynamic_cast`进行类型转换**

  > 此处的`dynamic_cast`是必定会成功的，因为我们已经通过typeid运算保证了两个对象的类型相同

- 示例代码

  ```c++
  class Base {
      bool operator=(const Base& rhs) const{
          return typeid(*this) == typeid(rhs) && lhs.equal(rhs);
      }
      
      virtual bool equal(const Base& rhs) const {
          // 比较Base对象
      }
  };
  
  class Derived: public Base {
      bool equal(const Base& rhs) const {
          const Derived& r = dynamic_cast<const Derived&>(rhs);
          // 比较两个Derived对象
      }
  };
  
  ```

  
