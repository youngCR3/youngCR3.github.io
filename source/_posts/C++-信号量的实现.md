---
title: C++ 信号量的实现
date: 2022-02-25 23:16:57
tags: C++
categories: C++
mathjax: true
---

**C++没有提供信号量，可以通过标准库的mutex、unique_lock和condition_variable实现信号量机制**

<!--more-->

# Mutex类

- Mutex又称互斥量，C++11中与Mutex相关的类（包括锁类型Lock类）和函数都声明在**`mutex`头文件**中

- **定义与初始化**

  ```c++
  mutex mtx;
  ```

- `lock()`成员，调用线程锁住该互斥量

  ```c++
  mtx.lock();
  ```

- `unlock()`成员，调用线程释放对互斥量的所有权

  ```c++
  mtx.unlock();
  ```

- `try_lock()`成员，尝试锁住互斥量，若被其他线程占有也不会阻塞。返回**false代表互斥量被其他线程占有**

  ```c++
  if (mtx.try_lock()) {
      // ...
  } else {
      // ...
  }
  ```


# unique_lock类

- 构造函数接受一个`mutex`，将其锁定
- 析构函数将`mutex`解锁

# 条件变量类

- 通过上述互斥量和条件变量，可以构造**管程**
- 定义在**头文件**`condition_variable`中

- 定义与初始化

  ```c++
  condition_variable cr;
  ```

- 条件变量的`wait`成员

  ```c++
  void wait(unique_lock<mutex>& lock);
  ```

- 条件变量的`notify_one`成员

  ```c++
  void notify_one();
  ```



# 自定义信号量类Semaphore

```c++
class Semaphore {
public:
    Semaphore(int c): _cnt(c) {}
    
    void set(int c) { _cnt = c; }
    
    void signal() {
        unique_lock<mutex> lock(_mtx);
        ++_cnt;
        _cv.notify_one();
    }

    void wait() {
        unique_lock<mutex> lock(_mtx);
        while (_cnt <= 0)
            _cv.wait(lock);
        --_cnt;
    }

private:
    int _cnt;
    mutex _mtx;
    condition_variable _cv;
};
```

