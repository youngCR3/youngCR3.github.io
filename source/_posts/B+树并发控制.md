---
title: B+树并发控制
date: 2022-04-09 01:16:27
tags:
    - Database System
    - 面试
mathjax: true
categories: Database System
---

**索引查找的多线程并发控制原理**

<!--more-->

# B+树并发控制

## 并发控制协议

## DBMS中Lock与Latch的区别

- Lock是事务层次的锁，需要具备rollback changes的能力
- Latch是操作层面的锁，相当于访问共享资源的临界区前需要加索，无需具备rollback changes的能力

## Latch mode

- read mode：共享锁，可以有多个线程同时获取read mode的latch
- write mode：独占锁，线程只能独自地获取write mode的latch，此时其他线程无法获取read/write mode的latch

## Latch实现

### Blocking OS mutex

- 概念：加锁时如果需要等待则进入内核，内核将当前线程阻塞并调度其他线程，由于需要切换到内核态，开销昂贵

- 示例：`std::mutex`

  ```c++
  std::mutex mtx;
  mtx.lock();
  // 访问共享资源
  mtx.unlock();
  ```

### Test-And-Set Spin Latch

- 概念：通过Test and set指令尝试加锁，若锁被占有则不断循环重试（spin），由于CPU资源消耗在了spin上，因此CPU利用率低

- 示例：`std::atomic`

  ```c++
  std::atomic<bool> latch;
  while (latch.test_and_set(...)) {
    // retry? Yield? Abort?
  }
  // 访问共享资源
  latch.unlock();
  ```

### Reader-Writer Latch

- 概念：多个线程可以同时获取读锁，而某个线程获取了写锁将阻塞其他所有读锁/写锁的获取

## Hash table latching

- 不会发生死锁：以linear probe hashing为例，查询数据项时，总是按同一个方向进行linear search，并以此对各个槽上锁/解锁，因此不满足死锁的环路等待条件，不会发生死锁
- table resize：获取整个表的global latch，调整大小并对所有数据rehash后解锁即可

### 方法一：page latches

- 概念：对每个page维护一个latch，一个page包含多个slots，为了提高并发度，该latch一般为reader-writer latch
- 特点：需要管理的锁更少，但颗粒度较粗，并发度降低

### 方法二：slot latches

- 概念：对每个slot维护一个latch，由于颗粒度较细，为减少需要维护的信息，该latch一般为single mode latch
- 特点：需要维护的锁更多，但锁的颗粒度细，并发度更高

## B+ tree latching

### B+树并发控制需要解决的问题

- 为了提高并发度，允许多个线程同时read/update B+树

- 为了保证正确性，需要防止：

  - 多线程同时修改同一个节点的内容

  - 一个线程搜索B+树的同时，另一个线程进行合并/拆分节点

### Latch crabbing/coupling

- Find：从根节点往叶子节点搜索，先获取子节点的R latch，再释放父节点的R latch

![B_plus_tree_find.drawio](B_plus_tree_find.drawio.svg)

- Insert/Delete：从根节点往叶子节点搜索，获取各个节点的W latch，获取了子节点的W latch后检查其是否安全，若安全则释放所有祖先节点的W latches

![B_plus_tree_insert.drawio](B_plus_tree_insert.drawio.svg)

- 安全
  - 对于Insert，指不会溢出
  - 对于Delete，指删除后所有节点都at least half-full
  - 确认安全的时机，并非总是到达叶子节点才确认，对于insert，若某个inner node当前非满则可以确保插入一个数据项不会导致溢出；对与delete，若某个inner node拥有的keys严格超过一半，则删除一个数据项也能保证at least half-full，则可以在inner node确认安全

- 释放祖先节点锁的顺序，不会影响正确性，但是由于更高层次的锁覆盖了更多的节点，因此优先释放高层次节点的锁能提高并发度，因此可以**将获取的锁存放在queue中**

### Better Latching Algorithm（乐观锁）

- Find：与上述一致，即先获取子节点的R latch，再释放父节点的R latch
- Insert/Delete
  - 乐观锁思想：假设最终到叶子节点是safe的，无需merge/split
  - 根节点到叶子节点前的inner node的遍历：获取子节点到R latch，再释放父节点到R latch
  - 叶子节点safe：获取其W latch，检查是否safe，如果safe则insert/delete并执行成功
  - 叶子节点unsafe：释放当前的latch，然后重新从根节点到叶节点遍历，并按上述普通策略获取路径上节点到W latches

- 工程实际中，B+树每个节点一般容纳多个数据项，因此合并/拆分发生的几率相对较小，使用乐观锁的思想是合理且能提高效率的

![Optimistic lock.drawio](Optimistic lock.drawio.svg)

### 死锁

- **Root-To-Leaf traversal不会死锁**：目前为止，只考虑了从top到bottom的遍历，总是沿同一方向加锁，不满足死锁的环路等待条件，不会出现死锁
- **Leaf node scan可能死锁**：但还存在leaf node之间的遍历，可能引起死锁？可能，因为leaf node之间的遍历可能是从左到右，也可能是从右到左的

### Leaf node scan

- **latch不支持死锁检测/避免**：操作层面的latch不支持死锁检测/死锁避免，它只依赖于程序员编写正确的代码来解决死锁
- **终止并retry**：在进行相邻叶节点的遍历时，对latch的获取必须支持no-wait mode，如等待一段时间仍未获取到则终止并重做，或直接终止并重做，而不能一直等待，因为我们不知道获取了该锁的线程的任何信息，因而无法判断是否出现了死锁

![Leaf_node_scan.drawio](Leaf_node_scan.drawio.svg)
