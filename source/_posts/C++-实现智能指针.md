---
title: C++-实现智能指针
date: 2022-04-05 20:49:44
tags: C++
categories: C++
mathjax: true
---

# 智能指针原理

## shared_ptr

### 引用计数

- shared_ptr包含两个指针，一个指向所共享的对象，另一个指向引用计数器，此外还保存着指向删除器的指针
- 当对shared_ptr进行拷贝时，引用计数加1，当shared_ptr被销毁时，引用计数减1。特别地，如果某个shared_ptr在销毁时引用计数为0，则需要对所共享的对象进行销毁并释放相应的内存

### 线程安全性

- 基于上述原理，对shared_ptr的操作包含对其指向对象的指针，以及指向引用计数的指针的修改，操作不具有原子性，因此多线程修改shared_ptr时可能出现race condition，不具有线程安全性

## unique_ptr

- 包含指向所拥有对象的指针，不支持拷贝操作，支持移动操作
- 包含销毁所指对象的删除器
- 若自定义删除器，则删除器类型也是unique_ptr的模板类型

# shared_ptr的实现

## 成员变量

- T* pointer，指向所共享的对象
- D* deleter，指向删除器，即销毁所指对象时调用的函数
- Counter* ctr，指向引用计数器

## 接口实现

- 构造函数：接受指针的构造函数、拷贝构造函数、移动构造函数
- 赋值运算符：拷贝赋值运算符、移动赋值运算符
- 析构函数：引用计数减1，若为0则调用删除器销毁所指对象
- 重载bool运算符
- 重载解引用运算符
- 重载箭头运算符
- get()获取裸指针
- reset()，reset(ptr)，reset(ptr, del)，放弃对对象的引用
- unique()，返回是否唯一引用对象的智能指针
- use_count()，返回引用该对象的智能指针数目
- swap(shared_ptr& rhs)，与另一个智能指针交换所指对象

# unique_ptr的实现

## 成员变量

- T* pointer，指向所共享的对象
- D deleter，删除器，即销毁所指对象时调用的函数，默认采用delete删除

## 接口实现

- 构造函数：接受指针的构造函数、移动构造函数
- 赋值运算符：移动赋值运算符
- 析构函数：采用删除器销毁对象
- 拷贝构造函数合拷贝赋值运算符定义为delete，表示不支持拷贝操作
- 重载bool运算符
- 重载解引用运算符
- 重载箭头运算符
- get()获取裸指针
- reset()，reset(ptr)
- release()放弃对指针的控制权，返回指针并置空

# 总结

- **构造函数、赋值运算符**的使用
- **重载bool、解引用和箭头运算符**的使用
- **模板类**的使用
- shared_ptr和unique_ptr的不同实现原理
- shared_ptr和unique_ptr对**删除器的管理方式**：shared_ptr内部包含指向删除器的指针，删除器可以改变，不影响shared_ptr的类型；unique_ptr内部存放了删除器本身，删除器类型不可改变，删除器类型也作为unique_ptr类型的一部分

# 代码

## shared_ptr

```c++
#include <iostream>
#include <functional>
#include <cassert>

namespace A {
template<typename T>
class SharedPointer;

template<typename T>
void swap(SharedPointer<T>& lhs, SharedPointer<T>& rhs) {
	std::swap(lhs.pointer, rhs.pointer);
	std::swap(lhs.ctr, rhs.ctr);
	std::swap(lhs.deleter, rhs.deleter);
} 

template<typename T>
void Delete(T* ptr) {
	delete ptr;
}

class Counter {
public:
	Counter(int cnt = 1): counter(cnt) {}
	int counter;
};


template<typename T>
class SharedPointer {
public:
	// 默认构造函数
	SharedPointer(T* ptr = nullptr, std::function<void(T*)> del = A::Delete<T>): pointer(ptr), ctr(new Counter(1)), deleter(del) {}

	// 拷贝构造函数
	SharedPointer(const SharedPointer& other): pointer(other.pointer), ctr(other.ctr), deleter(other.deleter) {
		++ctr->counter;
	}

	// 移动构造函数
	SharedPointer(SharedPointer&& other): pointer(other.pointer), ctr(other.ctr), deleter(other.deleter) {
		other.pointer = nullptr;
		other.ctr = nullptr;
	}

	// 拷贝赋值运算符
	SharedPointer& operator=(const SharedPointer& other) {
		decrementAndDestory();
		pointer = other.pointer;
		ctr = other.ctr;
		deleter = other.deleter;
		++ctr->counter;
		return *this;
	}

	// 移动赋值运算符
	SharedPointer& operator=(SharedPointer&& other) {
		decrementAndDestory();
		swap(other);
	}

	~SharedPointer() {
		decrementAndDestory();
	}

	// 将SharedPtr对象作为bool值
	operator bool() {
		return pointer? true: false;
	}

	// 重载解引用运算符
	T& operator*() {
		if (!pointer)
			throw "Bad operator*";
		return *pointer;
	}

	// 重载箭头运算符
	T* operator->() {
		return &this->operator*();
	}


	T* get() {
		return pointer;
	}

	void reset() {
		decrementAndDestory();
	}

	void reset(T* ptr) {
		decrementAndDestory();
		pointer = ptr;
		ctr = new Counter(1);
	}

	void reset(T* ptr, const std::function<void(T*)> del) {
		reset(ptr);
		deleter = del;
	}

	bool unique() {
		return ctr->counter == 1;
	}

	int use_count() {
		return ctr->counter;
	}

	void swap(SharedPointer& rhs) {
		A::swap(*this, rhs);
	}

private:
	T* pointer;
	Counter* ctr;
	std::function<void(T*)> deleter;

	// 递减计数器, 若计数器为0则销毁所指对象, 重置指针值为nullptr
	void decrementAndDestory() {
		if (pointer) {
			--ctr->counter;
			if (ctr->counter == 0) {
				deleter(pointer);
				delete ctr;
			}
		} else {
			delete ctr;
		}
		pointer = nullptr;
		ctr = nullptr;
	}

};

};

void copySharedPointerTest() {
	using namespace A;
	int i = 2022;
	SharedPointer<int> sp1(&i);
	SharedPointer<int> sp2(sp1);
	assert(*sp1 == 2022);
	assert(*sp2 == 2022);
	assert(sp1.use_count() == 2);
	std::cout << "copyTest passed" << std::endl;
}

void moveSharedPointerTest() {
	using namespace A;
	int i = 2022;
	SharedPointer<int> sp1(&i);
	SharedPointer<int> sp2(std::move(sp1));
	assert(sp1.get() == nullptr);
	assert(*sp2 == 2022);
	assert(sp2.use_count() == 1);
	std::cout << "moveTest passed" << std::endl;
}

int main() {
	copySharedPointerTest();
	moveSharedPointerTest();
}
```

## unique_ptr

```c++
#include <iostream>
#include <functional>
#include <cassert>

namespace A {

template<typename T>
void Delete(T* ptr) {
	delete ptr;
}

template<typename T, typename D = std::function<void(T*)>>
class UniquePointer {
public:
	// 默认构造函数
	UniquePointer(T* ptr = nullptr, D del = A::Delete<T>): pointer(ptr), deleter(del) {}

	// 移动构造函数
	UniquePointer(UniquePointer&& rhs) {
		pointer = rhs.pointer;
		deleter = std::forward<D>(rhs.deleter);
		rhs.pointer = nullptr;
	}

	// 移动赋值运算符
	UniquePointer& operator=(UniquePointer&& rhs) {
		pointer = rhs.pointer;
		deleter = std::forward<D>(rhs.deleter);
		rhs.pointer = nullptr;
	}

	// 析构函数
	~UniquePointer() {
		destroy();
	}

	// 拷贝构造函数
	UniquePointer(const UniquePointer& rhs) = delete;

	// 拷贝赋值运算符
	UniquePointer& operator=(const UniquePointer& rhs) = delete;

	// 裸指针赋值
	UniquePointer& operator=(T* ptr) = delete;

	// 重载bool
	operator bool() {
		return pointer? true: false;
	}

	// 重载解引用运算符
	T& operator*() {
		return *pointer;
	}

	// 重载箭头运算符
	T* operator->() {
		return &this->operator*();
	}

	// 获取裸指针
	T* get() {
		return pointer;
	}

	// 放弃对指针的控制权, 返回指针并置空
	T* release() {
		T* temp = pointer;
		pointer = nullptr;
		return temp;
	}
	
	// 释放所指对象
	void reset() {
		destroy();
	}

	// 释放所指对象, 并指向另一个对象
	void reset(T* ptr) {
		reset();
		pointer = ptr;
	}

private:
	T* pointer;
	D deleter;

	void destroy() {
		if (pointer) {
			deleter(pointer);
			pointer = nullptr;
		}
	}
};

};

void basicTest() {
	using namespace A;
	UniquePointer<int> up1(new int(10));
	assert(*up1 == 10);
	std::cout << "Basic test passed\n";
}

void moveTest() {
	using namespace A;
	UniquePointer<int> up1(new int(10));
	UniquePointer<int> up2(std::move(up1));
	assert(*up2 == 10);
	assert(up1.get() == nullptr);
	std::cout << "Move test passed\n";
}

void resetTest() {
	using namespace A;
	UniquePointer<int> up(new int(10));
	up.reset();
	assert(up1.get() == nullptr);
	up.reset(new int(2022));
	assert(*up1 == 2022);
	std::cout << "Reset test passed\n";
}

int main() {
	basicTest();
	moveTest();
	resetTest();
}
```

