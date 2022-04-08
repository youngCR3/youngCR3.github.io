---
title: Modern C++
date: 2022-03-16 09:37:46
tags:
    - C++
    - 面试
categories: C++
mathjax: true
---

# 型别推导

## 1 模板类型推导

- 函数模板形如

```c++
template<typename T>
void f(ParamType param);
```

### ParamType是个指针或引用（不是万能引用）

- 类型实参的引用性会被忽略

```c++
template<typename T>
void f(T& param);

int x = 27;
const int cx = x;
const int& rx = x;
f(x);	// T为int, ParamType为int&
f(cx);	// T为const int, ParamType为const int&
f(cr);	// T为const int, ParamType为const int&
```



```c++
template<typename T>
void f(const T& param);

int x = 27;
const int cx = x;
const int& rx = x;
f(x);	// T为int, ParamType为const int&
f(cx);	// T为int, ParamType为const int&
f(cr);	// T为int, ParamType为const int&
```

### ParamType是个万能引用

- 类型实参是左值则推导为左值引用，则T和ParamType都会被推导为左值引用
- 类型实参是右值，T的类型会应用情况1的规则，ParamType被推导为右值引用

```c++
template<typename T>
void f(T&& param);

int x = 27;
const int cx = x;
const int& rx = x;
f(x);	// T为int&, ParamType为int&
f(cx);	// T为const int&, ParamType为const int&
f(cr);	// T为const int&, ParamType为const int&
f(27);	// T为int, ParamType为int&&
```

### ParamType既非指针也非引用

- 按值传递，param是传入实参的一个副本
- 参数的引用类型将被忽略
- 参数的const类型也将被忽略

```c++
template<typename T>
void f(T param);

int x = 27;
const int cx = x;
const int& rx = x;
f(x);	// T为int, ParamType为int
f(cx);	// T为int, ParamType为int
f(cr);	// T为int, ParamType为int
```

- 如果是指向常量对象的常量指针作为实参，则**顶层const忽略，底层const保留**

```c++
template<typename T>
void f(T param);

const char* const ptr = "hello";
f(ptr);		// T和paramType被推导为const char*
```

## 2 auto类型推导

### 类型修饰词非指针非引用

```c++
auto x = 27;		// int
const auto cx = x;	// const int
```

### 类型修饰词是指针或引用，但非万能引用

```c++
const auto& rx = x;	// const int&
```

### 类型修饰词是万能引用

```c++
auto&& r1 = x;		// int&
auto&& r2 = cx;		// const int&
auto&& r3 = 27;		// int&&
```

### auto类型推导的不同点

```c++
int x = 27;
int x1 = {27};
int x2{27};		// 都将得到值为27的int变量
```

```c++
auto x = 27;		// int
auto x1 = {27};		// std::initializer_list<int>
auto x2{27};		// std::initializer_list<int>
```

- auto声明变量的初始化表达式用大括号括起时，推导得到的类型是`std::initializer_list`

## 3 decltype类型推导

- 绝大多数情况会直接得到变量或表达式的类型
- 对于类型为T的左值表达式，若表达式仅有一个名字，则得到类型`T`，否则得到类型`T&`

```c++
int x = 28;
// decltype(x)得到类型int, 因为仅包含一个名字x
// decltype(x + 27)得到类型int&
// 特别地, decltype((x))也得到类型int&
```



- `decltype(auto)`作为函数返回值类型，按decltype的规则根据返回表达式进行推导

# auto

## 6 当auto推导的类型不符合要求时，使用带显式型别的初始化物习惯用法

- **隐形的代理类型**可以导致auto根据初始化表达式推导出错误的型别，如`std::vector<bool>`的下标表达式返回`std::vector<bool>::reference`，而非`bool&`
- 带显式型别的初始化物习惯用法强制auto推导出你想要的型别

```c++
auto highPriority = static_cast<bool>(features(w)[5]);
```

# Modern C++

## 7 创建对象时注意区分()和{}

- {}适用范围更宽泛，可以阻止隐式窄化类型转换

```c++
class T {
private:
    int c_;
    int x_;
public:
	T(int c, int x): c_(c), x_(x) {} 
};

T t1(1, 3.0);	// 通过
T t2{1, 3.0};	// 编译错误
```

- 只要有**任何可能**，**大括号**初始化就会与带有`std::initializer_list`类型的形参匹配，即使其他重载版本有着更加匹配的形参表

```c++
class T {
public:
	T(int i, bool b);
    T(int i, double d);
    T(std::initializer_list<bool> il);
};

T t{10, true};	
// 会匹配std::initializer_list版本, 而且因为要求窄化型别转换会导致编译错误
```

## 8 优先选用nullptr，而非0或NULL

- 0和NULL都非指针类型

## 9 优先选用别名声明，而非typedef（补充）

- typedef不支持模板化，但别名声明支持

## 10 优先选用限定作用域的枚举类型，而非不限作用域的枚举类型

- 优先使用`enum class `而非`enum`

```c++
// enum Color { black, white, red };
enum class Color { black, white, red };
```

**限定作用域的枚举类型不会泄露名字**：必须使用`Color::red`

**限定作用域的枚举类型不允许枚举量进行隐式类型转换**：`enum`版本允许枚举量隐式转换为整数，造成不必要的错误



## 12 为意在改写的函数添加override声明

- **改写override需要满足的要求**

  **虚函数**：基类中的函数必须是虚函数

  **名字相同**：基类和派生类中的函数名字必须完全相同

  **形参类型相同**：基类和派生类中的函数的形参类型必须完全相同

  **函数常量性相同**：基类和派生类的函数常量性必须完全相同（const限定符）

  **函数返回值兼容**

  **函数引用限定词相同**：`&`限定了只能通过左值调用成员函数，`&&`限定了只能通过右值调用成员函数

- `override`**声明的作用**：只有在成员函数加上`override`声明，编译器才会检查它是否改写了基类的虚函数，若没有改写则报错

- `&`限定词与`&&`限定词

  ```c++
  DataType& data() & { return values; }				// 返回左值
  
  DataType data() && { return std::move(values); }	// 返回右值
  ```

## 16 保证const成员函数的线程安全性

- **保证const成员函数的线程安全性**：const成员函数可能使用缓存，因此实际上会修改声明为mutable的成员变量

```c++
// 示例: 多项式类, 其中roots成员函数用于计算该多项式的根, 结果缓存在rootVals中
// 缓存根：由于求根不会改变多项式, 因此该成员声明为const, 但为了防止重复计算, 我们对根使用了缓存, 并将变量设为mutable, 从而可在const限定的roots成员函数中修改缓存
// 问题：如果两个线程同时进行求根, 则可能引发问题, 虽然由于roots是const成员函数, 但实际上它会修改缓存内容, 为此必须使用std::mutex保证const成员函数的线程安全性
class Polynomial {
public:
    RootsType roots() const {
        if (!rootsAreValid) {
            // ...
            rootsAreValid = true;
        }
        return rootVals;
    }
private:
    mutable bool rootsAreValid{false};
    mutable RootsType rootVals{};
};
```



# 智能指针

## shared_ptr、unique_ptr和weak_ptr

- unique_ptr实现的是专属所有权的语义，它是个只移类型，不能复制或共享，unique_ptr大小等同于一个指针
- shared_ptr采用引用计数，大小等同于两个指针，一个用于指向所共享的对象，另一个指向包含引用计数的共享控制块
- weak_ptr是结合shared_ptr使用的特例智能指针，它提供对shared_ptr所共享对象的访问，但不参与引用计数

## shared_ptr的线程安全性

- `shared_ptr`包含两个指针，一个指向**所共享的对象**，一个指向引用**计数器所在的内存**，`shared_ptr`的操作都涉及对这两个指针的修改，操作不具有原子性。因此多线程修改`shared_ptr`时应该通过`std::mutex`保证线程安全性

# 右值引用、移动语义和完美转发

## 23 理解std::move和std::forward

- **强制类型转换**：std::move不进行任何移动，std::forward不进行任何转发，它们不生成任何可执行代码，而是只执行强制类型转换的函数
- **std::move**：无条件地将实参强制转换成**右值**，它并不进行移动，也不保证经过强制类型转换后的对象具备可移动能力，它只保证结果是个右值
- **std::forward**：仅在某个**特定条件**满足时才执行右值强制转换，即传入的实参被绑定到右值时才进行向右值的强制转换
- **作用**：std::move常用于为移动操作做铺垫，而std::forward常用作传递或转发一个对象到另一个对象
- std::move实现具体原理
  - **引用折叠**：通过引用折叠原理，T&&类型参数，传入右值实参将得到右值，传入左值实参将得到左值
  - **移除引用**：通过remove_reference移除引用，得到具体类型T
  - **强制转换**：通过static_cast显式转换得到T&&右值引用类型

## 24 区分万能引用和右值引用

- T&&：有两种含义，右值引用或万能引用（绑定到左值时为左值引用，绑定到右值时为右值引用）
- **万能引用：函数模板的形参**

```c++
template<typename T>
void f(T&& param);		// param是个万能引用
```

- **万能引用：auto声明**

```c++
auto&& var2 = var1;		// var2是个万能引用
```

- **右值引用**：如果T&&形式，没有涉及类型推导，则为右值引用

## 25 针对右值引用实施std::move，针对万能引用实施std::forward

- 右值引用实施std::move，万能引用实施std::forward
- 若局部对象可能适用于返回值优化RVO，则不应实施std::move或std::forward

## 26 避免使用万能引用类型进行函数重载

- **避免使用万能引用类型作为函数参数**：万能引用作为重载函数的形参类型，几乎总会让该重载版本在始料未及的情况下被调用
- **避免使用完美转发构造函数**：对于非常量的左值类型，一般会形成比复制构造函数的更佳匹配

## 28 理解引用折叠

- **引用折叠发生的语境**：模板实例化、auto类型生成、typedef和别名声明，decltype
- **引用折叠**：编译器在引用折叠语境下生成引用的引用，结果会变为单个引用。若原始的引用中存在任意一个为左值引用，则结果为左值引用；否则，为右值引用

