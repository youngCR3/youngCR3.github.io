---
title: 6.824 Lec2 RPC and threads
date: 2021-12-09 14:46:49
tags: 6.824
---

# 1 线程

## 1.1 线程的特点

- share memory
- per-thread state: PC，registers，stack
- threads in Go are called **goroutines**

## 1.2 为什么需要线程

- I/O concurrency
- multicore performance
- `event-driven program`可以实现I/O concurrency，但是无法实现multicore performance，且difficult to program

## 1.3 线程使用中的注意点

- shared data需要使用lock等保护
- 避免deadlock



# 2 远程过程调用RPC

## 2.1 基本概念

- client：请求服务的进程
- server：提供服务的进程
- 对于不同的request，server需要提供不同的RPC handler，且需要有不同的Args and Reply struct

## 2.2 RPC handler的并发性

- 对于每个请求，server都会新建一个goroutine处理
- 因此RPC handler中对shared data的修改必须使用lock

## 2.3 RPC如何处理failure

### 2.3.1 best effort模型

- Call()等待**最大等待时间**后，若没收到则
- 重新发送request，设定**最大重新发送次数**
- 还没收到则放弃并**返回error**

### 2.3.2 at most once模型

如果failure并非发生在server，则server可能重复收到相同的request，并重复执行handler，因此提出了at most once模型

- 每个**request 都有唯一的标识码XID**
- RPC server会检查request是否重复，若重复则返回先前的reply，而不会重新调用handler
- 为了防止不同client的XID重复，可以**将client ID与XID组合**作为标识

### 2.3.3 GO中的RPC模型

- 只发送一次请求，若固定时间后没收到reply，则直接返回error

