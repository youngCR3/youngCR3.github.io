---
title: C++ 实现Vector
date: 2022-04-05 23:10:19
tags: C++
categories: C++
mathjax: true
---

# Vector

## 数据成员

- 容量：capacity，表示当前数组的大小
- 大小：sz，表示当前Vector存放的元素个数
- 存放数据的数组arr

## 接口

### 构造函数

- 默认构造函数
- 接受元素数量参数的构造函数
- 接受元素数量参数和元素初始值的构造函数
- 接受初始值列表的构造函数
- 拷贝构造函数
- 移动构造函数

### 赋值运算符

- 拷贝赋值运算符
- 移动赋值运算符

### 增加元素

- insert
- push_back

### 删除元素

- pop_back
- erase

### 获取信息

- size
- empty
- getCapacity

### 访问元素

- 重载[]运算符

### 扩容

- reserve，用户请求容量
- grow，当溢出时将容量扩大为之前的两倍（使得添加元素的均摊复杂度为O(1)）

## 实现

```c++
#include <iostream>
#include <cassert>
#include <initializer_list>

template<typename T>
class Vector {
public:
	// 默认构造函数
	Vector(): arr(nullptr), capacity(1), sz(0) {}

	// 接受元素数量的构造函数
	Vector(int cnt): arr(new T[std::max(1, cnt * 2)]), capacity(std::max(1, cnt * 2)), sz(cnt) {}

	// 接受元素数量和元素值的构造函数
	Vector(int cnt, T val): arr(new T[std::max(1, cnt * 2)]), capacity(std::max(1, cnt * 2)), sz(cnt) {
		for (int i = 0; i < sz; ++i) {
			arr[i] = val;
		}
	}

	// 接受初始值列表的构造函数
	Vector(std::initializer_list<T> li): arr(new T[std::max(1, (int)li.size() * 2)]), capacity(std::max(1, (int)li.size() * 2)), sz(li.size()) {
		int i = 0;
		for (auto &v: li) {
			arr[i] = v;
			++i;
		}
	}

	// 拷贝构造函数
	Vector(const Vector& other) {
		arr = new T[other.capacity];
		capacity = other.capacity;
		sz = other.sz;
		for (int i = 0; i < sz; ++i)
			arr[i] = other.arr[i];
	}

	// 移动构造函数
	Vector(Vector&& other) {
		arr = other.arr;
		capacity = other.capacity;
		sz = other.sz;
		other.arr = nullptr;
		other.capacity = 0;
		other.sz = 0;
	}

	// 拷贝赋值运算符
	Vector& operator=(const Vector& other) {
		arr = new T[other.capacity];
		capacity = other.capacity;
		sz = other.sz;
		for (int i = 0; i < sz; ++i)
			arr[i] = other.arr[i];
		return *this;
	}

	// 移动赋值运算符
	Vector& operator=(Vector&& other) {
		arr = other.arr;
		capacity = other.capacity;
		sz = other.sz;
		other.arr = nullptr;
		other.capacity = 0;
		other.sz = 0;
		return *this;
	}

	// 析构函数
	~Vector() {
		delete[] arr;
		arr = nullptr;
	}

	// 插入
	void insert(int pos, T val) {
		if (sz == capacity)
			grow();
		for (int i = sz; i > pos; --i)
			arr[i] = arr[i - 1];
		arr[pos] = val;
		++sz;
	}

	// 删除第一个等于val的元素
	void erase(T val) {
		for (int i = 0; i < sz; ++i) {
			if (arr[i] == val) {
				for (int j = i; j < sz - 1; ++j) {
					arr[j] = arr[j + 1];
				}
				--sz;
				return;
			}
		}
	}

	// 末尾添加/删除
	void push_back(T val) {
		if (sz == capacity)
			grow();
		arr[sz] = val;
		++sz;
	}

	void pop_back() {
		if (!empty()) {
			--sz;
		}
	}

	// 返回元素数量
	size_t size() {
		return sz;
	}

	// 返回是否空的vector
	bool empty() {
		return sz == 0;
	}

	// 随机访问
	T& operator[](int index) {
		if (index >= sz || index < 0)
			throw "Out of range";
		return arr[index];
	}

	// 扩容reserve
	void reserve(int cap) {
		if (cap <= capacity)
			return;
		T* newArrary = new T[cap];
		for (int i = 0; i < sz; ++i)
			newArrary[i] = arr[i];
		delete arr;
		arr = newArrary;
		capacity = cap;
	}

	// 测试接口, 获取当前容量
	int getCapacity() {
		return capacity;
	}

	// 测试接口, 获取当前数组
	T* getData() {
		return arr;
	}

private:
	T* arr;
	int capacity;
	int sz;

	// 溢出, 分配当前两倍的空间
	void grow() {
		T* newArrary = new T[capacity * 2];
		for (int i = 0; i < sz; ++i)
			newArrary[i] = arr[i];
		delete arr;
		arr = newArrary;
		capacity *= 2;
	}	


};

void BasicTest() {
	Vector<int> vec(10, 3);
	for (int i = 0; i < (int)vec.size(); ++i) {
		std::cout << vec[i] << ' ';
		assert(vec[i] == 3);
	}
	std::cout << '\n';
	vec.push_back(4);
	assert(vec.size() == 11);
	assert(vec[10] == 4);
	vec.pop_back();
	assert(vec.size() == 10);
	std::cout << "BasicTest passed\n";
}

void CopyTest() {
	Vector<int> v1(10, 1);
	Vector<int> v2 = v1;
	Vector<int> v3(v2);
	for (int i = 0; i < (int)v2.size(); ++i) {
		assert(v2[i] == 1);
	}
	for (int i = 0; i < (int)v3.size(); ++i) {
		assert(v3[i] == 1);
	}
	std::cout << "CopyTest passed\n";
}

void InsertTest() {
	Vector<int> vec(10, 1);
	vec.insert(0, 3);
	assert(vec.size() == 11);
	for (int i = 0; i < (int)vec.size(); ++i) {
		assert(vec[i] == (i == 0? 3: 1));
		std::cout << vec[i] << ' ';
	}
	std::cout << '\n';
	std::cout << "InsertTest passed\n";
}

void EraseTest() {
	Vector<int> vec{0, 1, 2, 3, 4};
	vec.erase(2);
	assert(vec.size() == 4);
	for (int i = 0; i < (int)vec.size(); ++i)
		std::cout << vec[i] << ' ';
	std::cout << '\n';
	std::cout << "EraseTest passed\n";
}

void ReserveTest() {
	Vector<int> vec{0, 1, 2, 3, 4};
	assert(vec.getCapacity() == 10);
	vec.reserve(8);
	assert(vec.getCapacity() == 10);
	vec.reserve(11);
	assert(vec.getCapacity() == 11);
	for (int i = 0; i < (int)vec.size(); ++i) {
		assert(vec[i] == i);
		std::cout << vec[i] << ' ';
	}
	std::cout << '\n';
	std::cout << "ReserveTest passed\n";
}

void GrowTest() {
	Vector<int> v{0};
	v.push_back(1);
	assert(v.getCapacity() == 2);
	v.push_back(2);
	assert(v.getCapacity() == 4);
	assert(v.size() == 3);
	std::cout << "GrowTest passed\n";
}	

void MoveTest() {
	Vector<int> v1{0, 1, 2, 3};
	Vector<int> v2(std::move(v1));
	assert(v1.size() == 0);
	assert(v1.getData() == nullptr);
	assert(v2.size() == 4);
	for (int i = 0; i < (int)v2.size(); ++i)
		assert(v2[i] == i);
	std::cout << "MoveTest passed\n";
}

int main() {
	BasicTest();
	CopyTest();
	InsertTest();
	EraseTest();
	ReserveTest();
	GrowTest();
	MoveTest();
}
```

