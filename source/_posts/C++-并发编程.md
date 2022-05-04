---
title: C++并发编程
date: 2022-04-19 22:02:47
tags:
    - C++
    - 多线程
categories: C++
mathjax: true  
---

C++并发编程

<!--more-->

# std::thread

## 基本用法

- `join()`方法：结合，等待线程执行完成
- `detach()`方法：分离，显式地声明不再等待该线程完成

```c++
class Task {
public:
    // 重载调用运算符
  	void operator() () const {
      	cout << "Call Task()" << endl;
    }
};

// 使用可调用对象构建myThread
Task f1, f2;
std::thread myThread1(f1);
std::thread myThread2(f2);
myThread1.join();			// 结合，等待线程执行完成
myThread2.detach();		// 分离，显式地声明不再等待它
```



## 传递参数给线程函数

```c++
class Task {
public:
    // 重载调用运算符
  	void operator() (int x) const {
      	cout << x << endl;
    }
};

// 使用可调用对象构建myThread
Task f1, f2;
std::thread myThread1(f1, 10);
std::thread myThread2(f2, 5);
myThread1.join();			// 结合，等待线程执行完成
myThread2.join();
```

## 转移线程所有权

- 需要转移线程所有权的场景

  编写一个函数，创建一个后台运行的线程，向调用函数者会穿新线程的所有权

  创建一个线程，并将所有权传递给要等待它完成的函数

- std::thread支持移动操作，而且是只移类型，不可复制

- 从函数中返回std::thread

```c++
// 从函数中返回线程
std::thread getThread() {
    std::thread t(f);
    return std::move(t);
}
```

- 将std::thread作为参数传递给函数

```c++
void f() {
    std::cout << "f() is called\n";
}

// 传递线程作为函数参数
void watchThread(std::thread t) {
    t.join();
    std::cout << "thread done\n";
}

void watchThreadTest() {
    std::thread t1(f);
    watchThread(std::move(t1));
    // 下面语句是错误的, std::thread无法拷贝
    // watchThread(t2)
}
```

## 选择线程数量

- 获取给定程序执行时能够真正并发运行的线程数量提示

```c++
int threadCount = std::thread::hardware_currency();
```

## 实例：并行累加

```c++
#include <iostream>
#include <numeric>
#include <vector>
#include <cassert>
#include <thread>
using ULL = unsigned long long;

template<typename T>
void LOG_DEBUG(T msg) {
#ifndef NDEBUG
    std::cout << "LOG_DEBUG: " << msg;
#endif
}

template<typename Iterator, typename T>
class accumulate_block {
public:
    void operator() (Iterator first, Iterator last, T& result) {
        result = std::accumulate(first, last, result);
    }
};

template<typename Iterator, typename T>
T parallel_accumulate(Iterator first, Iterator last, T init) {
    // 迭代器范围内的元素数量
    const ULL length = std::distance(first, last);
    if (!length)
        return init;
    // 每个线程至少处理25个元素
    const ULL min_per_thread = 25;
    // 运行的线程数不超过硬件所能支持的线程(超额订阅), 因为上下文切换意味着更多的线程会降低性能; 所运行的线程数也不应该过少, 否则可能错过可用的并发
    const ULL max_threads = (length + min_per_thread - 1) / min_per_thread;
    const ULL hardware_threads = std::thread::hardware_concurrency();
    const ULL num_threads = std::min(hardware_threads != 0? hardware_threads: 2, max_threads);
    const ULL block_size = length / num_threads;
    std::vector<T> results(num_threads); 
    LOG_DEBUG("num_threads is " + std::to_string(num_threads) + ", block_size is " + std::to_string(block_size) + "\n");
    // 启动比num_threads少一个的线程, 因为已经有一个了
    std::vector<std::thread> threads(num_threads - 1);
    Iterator block_start = first;
    for (ULL i = 0; i < num_threads - 1; ++i) {
        Iterator block_end = block_start;
        std::advance(block_end, block_size);
        threads[i] = std::thread(accumulate_block<Iterator, T>(), block_start, block_end, std::ref(results[i]));
        block_start = block_end;
    }
    // 当前线程计算最后一部分
    accumulate_block<Iterator, T>() (block_start, last, results[num_threads - 1]);
    std::for_each(threads.begin(), threads.end(), std::mem_fn(&std::thread::join));
    return std::accumulate(results.begin(), results.end(), init);
}

void basicTest() {
    std::vector<int> arr(100, 10);
    int sum = parallel_accumulate(arr.begin(), arr.end(), 0);
    assert(sum == 1000);
    std::cout << "Test passed\n";
}

int main() {
    basicTest();
}
```

## 标识线程

- 线程标识符是`std::thread::id`类型的，有两种获取方式

  通过std::thread对象的`get_id()`成员函数获取

  当前线程的标识符，可以通过`std::this_thread::get_id()`获得

- 若某个std::thread对象没有线程，则调用get_id()将返回特殊值代表没有线程

```c++
Task f1, f2;
std::thread myThread1(f1, 10);
std::thread myThread2(f2, 5);
std::cout << "myThread1 id is " << myThread1.get_id() << std::endl;
std::cout << "myThread2 id is " << myThread2.get_id() << std::endl;
std::cout << "main thread id is " << std::this_thread::get_id() << std::endl;
std::thread myThread3;
std::cout << "myThread3 id is " << myThread3.get_id() << std::endl;
// 返回特殊值代表没有线程，如0x0
```

- std::thread::id类型定义了比较运算，同时标准库提供了`std::hash<std::thread::id>`，因此std::thread::id类型可以作为有序容器的键，也可以作为无需容器的键使用

# 线程之间共享数据

## 共享数据可能出现的问题

- 不变量：对于特定数据结构总是为真的语句，不变量在更新数据时可能被打破

- 示例：双向链表的不变量“如果A的next指针指向B，则B的prev指针指向A”，该不变量在插入或删除节点时可能被打破，如删除节点的步骤如下

  1. 标识要删除的节点N
  2. 将N的前一节点的next指针指向N的后一节点
  3. 将N的后一节点的prev指针指向N的前一节点
  4. 删除节点N

  在上述更新步骤的2和3之间，不变量被打破

- 竞争条件race condition，通常表示有问题的竞争条件
- 数据竞争data race，表示因单个对象的并发修改而产生的特定类型的竞争条件，数据竞争造成未定义行为UB

## 使用互斥元保护共享数据

- C++通过构造std::mutex的实例创建互斥元，调用lock()成员锁定，调用unlock()成员解锁
- C++提供std::lock_guard类模版，实现互斥元的RAII管用语法，在构造时锁定所给的互斥元，在析构时将互斥元解锁，从而保证被锁定的互斥元始终被正确解锁

```c++
#include <mutex>

std::mutex mtx;

int accessDataWithProtection() {
    std::lock_guard<std::mutex> guard(mtx);
    // 临界区代码
    // doSomething()
}
```

### 使用互斥元时的注意事项

- 不要将对受保护数据的指针或引用传递到锁的范围之外，包括
  1. 函数中返回它们
  2. 将其存放在外部可见的内存
  3. 作为参数传递给用户提供的函数

### 死锁及其避免

- C++提供的std::lock可以“同时”锁定多个互斥元

```c++
std::mutex mtx1, mtx2;
// std::lock同时锁定多个互斥元
std::lock(mtx1, mtx2);
// std::lock_guard传递第二个参数std::adopt_lock告知std::lock_guard对象该互斥元已被锁定，无需在构造函数中锁定该互斥元
std::lock_gurad<std::mutex> gurad1(mtx1, std::adopt_lock);
std::lock_gurad<std::mutex> gurad2(mtx2, std::adopt_lock);
// do something
```

- 避免嵌套锁：已经持有一个锁时，避免获取其他锁，若需要获取多个锁，则使用std::lock的单个动作执行
- 持有锁时，避免调用用户提供的代码：用户提供代码的行为是未知的，它可能尝试获取一个锁，从而违反了“避免嵌套锁”的原则
- 以固定顺序获取锁：如果必须要获取多个锁，且无法通过std::lock的单个操作获取，则每个线程中都以相同的顺序获取这多个锁
- 使用锁层次（类似固定顺序获取锁）

### 使用std::unique_lock灵活锁定

- std::unique_lock实例并不总是拥有与之相关联的互斥元
- 把std::defer_lock作为第二参数传递给std::unique_lock对象表示该互斥元在构造时应保持未被锁定，该互斥元的锁之后可通过std::unique_lock对象调用lock()获取
- 与std::lock_guard相似，std::unique_lock也实现了RAII，在构造时会对关联的互斥元锁定，在析构时会对关联的互斥元解锁

```c++
std::mutex mtx1, mtx2;
void unique_lock_test() {
    std::unique_lock<std::mutex> lock1(mtx1, std::defer_lock);
    std::unique_lock<std::mutex> lock2(mtx2, std::defer_lock);
    std::lock(mtx1, mtx2);
    // do something
    std::cout << "unique_lock_test passed\n";
}
```

- std::unique_lock对象提供了lock()、try_lock()、unlock()三个成员函数，它们会更新std::unique_lock实例内部的一个标识，该标识可以通过owns_lock()成员函数查询。由于需要更新该标识，因此std::unique_lock性能一般差于std::lock_gurad

### 在作用域之间转移锁的所有权

- std::unique_lock是只移类型，允许函数锁定一个互斥元，并将该锁的所有权转移给调用者，调用者可以在同一个锁的保护下执行额外的操作

```c++
std::unique_lock<std::mutex> get_lock() {
    extern std::mutex mtx;
    std::unique_lock<std::mutex> lk(mtx);
    prepare_data();
    // lk可以直接返回而无需调用std::move(), 编译器负责调用移动构造函数
    return lk;
}

void process_data() {
    std::unique_lock<std::mutex> lk(get_lock());
    do_something();
}
```

## 用于共享数据保护的替代工具

### 在初始化时保护共享数据

- 假设某个共享资源的构造非常昂贵，可以采用延迟初始化，即在第一次使用该资源时才构造
- 单线程的情况，只需要检查该资源是否被初始化，若未被初始化则进行初始化即可

```c++
std::shared_ptr<some_resource> resource_ptr;

void foo() {
    // 使用some_resource对象调用其do_something()成员, 检查resource_ptr是否为空, 若为空则创建some_resource对象
    if (!resource_ptr) {
        resource_ptr.reset(new some_resource);
    }
    resoure_ptr->do_something();
}
```

- 多线程场景下，假设对共享资源本身的并发访问是安全的（即允许并发调用`resource_ptr->do_something()`)，但对resource_ptr初始化应该被互斥元保护，以保证只有一个线程执行初始化

```c++
void foo() {
    // 所有线程在此处被序列化, 降低了并发性能
    std::unique_lock<std::mutex> lk(resource_mutex);
    if (!resource_ptr) {
        resource_ptr.reset(new some_resource);
    }
    lk.unlock();	// 只有初始化some_resource需要被保护
    resource_ptr->do_something();	
}
```

### std::once_flag和std::call_once

- C++标准库提供了std::once_flag和std::call_once来处理该情况
- 允许每个线程都使用std::call_once，std::call_once返回时，指针将会被某个线程初始化

```c++
std::shared_ptr<some_resource> resource_ptr;
std::once_flag resource_flag;

void init_resource() {
    resource_ptr.reset(new some_resource);
}

void foo() {
    // 初始化会被正好调用一次
    std::call_once(resource_flag, init_resource);
    resource_ptr->do_something();
}
```

### 保护很少更新的数据结构

- 读写互斥元：多个读线程可以并发访问，而写线程会阻塞其他任意读/写线程

```c++
boost::shared_mutex entry_mutex;

// 获取共享锁
boost::shared_lock<boost::shared_mutex> lk(entry_mutex);

// 获取独占锁
std::lock_guard<boost::shared_mutex> lk(entry_mutex);
```

### 递归锁

- 使用std::mutex时，一个线程试图锁定已经拥有的互斥元是错误的，试图这么做会导致未定与行为
- C++提供的std::recursive_mutex像std::mutex一样，但可以在同一个线程的单个实例上获取多个锁，并且应该正确地释放所有锁，如调用了3次lock则应该调用3次unlock
- 大多数情况std::recursive_mutex都能够用std::mutex代替，应尽量避免使用递归锁

# 同步并发操作

