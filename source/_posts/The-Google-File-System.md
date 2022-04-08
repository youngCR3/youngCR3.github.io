---
title: The Google File System
date: 2021-12-09 16:42:41
tags: 6.824
categories: Distributed System
---

# 1 Introduction

## 1.1 difference between GFS and previous distributed file system

- failures are the norm, because of the large quantity of servers
- files are huge, multi-GB files are common
- most files are mutated by appending new data rather than overwriting existing data
- co-designing the application and the file system API is helpful

# 2 Design Overview

## 2.1 Assumption

- components often **fail**
- a modest number of **large files**, multi-GB files are common
- **workload**主要分为两种：
  - **large streaming reads**
  - **small random reads**
- large, sequential writes that **append** data to files are also common
- **high sustained bandwidth** is more important than low latency

## 2.2 Interface

- 传统文件系统的结构
  - organized hierarchically in **directories**
  - **pathnames**
  - **create, delete, open, close, read and write** files

- 文件快照**snapshot**

  creates a copy of a file or a directory tree at low cost, using copy on write technique.

  > Recall copy-on-write  in 6.S081: Fork()  defers allocating and copying physical memory pages for the child until the copied pages are actually needed.

- 文件追加**Record append**

  allows multiple clients to append data to the same file concurrently while guaranteeing the atomicity of each individual client's append.

<!--more-->

## 2.3 Architecture

- **Single master**, **multiple chunk-servers**, accessed by **multiple clients**
- Each of master/server/client is typically a **commodity Linux machine**. It's possible to run both a chunk-server and a client on the same machine.
- Files are divided into **fix-size chunks**, each identified by a unique 64 bit **chunk handle**.
- Every chunk has **three replicas**.
- Master maintains all file system metadata.
- Communication
  - Client communicates with master for metadata operations
  - All data-bearing communication goes directly to the chunk-servers
- Cache
  - Client and chunk-server don't cache file data
  - Client caches metadata

## 2.4 Single Master

- Master **Data Structures**
  - 文件名到chunk ID array的对应
  - chunk ID到chunk数据的对应，包含每个chunk存储的servers list、每个chunk的版本号、primary chunk、租约时间
- data的存储
  - data structures存储在memory中，为防止master failure，将部分数据**写入disk中，包括：文件名到chunk ID array的对应、chunk的版本号**
  - chunk对应的server lists无需写入disk，因为master重启后会与每个chunk-server通信以获取其存储的chunk
  - primary chunk和租约时间也无需写入disk，master重启后只要确保所有租约到期，并重新分配primary chunk即可
- Log
  - 当GFS create chunk或更新chunk的版本号时，会向磁盘追加一条log记录
  - 维护log而不是数据库的原因：数据库本质是B+ tree或hash table，其写入效率不如log
  - crash recovery：**从最近的checkpoint开始逐条执行log，直至恢复状态**

## 2.5 Chunk size

- GFS规定**chunk大小为64MB**，远大于一般文件系统的block sizes
- 优点
  - 减少了clients与master交互的次数（因为client获取metadata信息后会cache，下次读写同一个chunk无需再次获取）
  - 减少了master存储metadata的大小
- 缺点
  - 小文件可能只持有1个chunk，持有这些chunk的chunk-servers可能成为**hot spots**

## 2.6 GFS读写文件

### 2.6.1 读文件

- client得到filename和offset，发送请求给master
- master根据filename确定chunk ID array，根据offset确定为哪个数组元素，得到chunk ID后可查询到存储chunk的server list，将其返回client
- client从server list中选取一个server（从最近的开始），将chunk ID和offset发送给chunk-server
- chunk-server收到请求后，在local disk读取数据并返回client

### 2.6.2 写文件

- 只考虑向文件append的情况（Record Append）
- client向master查询filename文件的最后一个chunk（因为是将数据追加在文件末尾）所在的server
- master查看filename是否指定了primary chunk，若没有则：
  - 从各个replica选取最新的一个（版本号与master保存的一致的replica视为最新的）作为primary replica，其余replica作为secondary replica
  - master更新chunk的版本号并写入disk
  - master向各个server发送消息告知其primary/secondary身份及新版本号
  - 对primary赋予一个租约
- 然后master向server返回primary和secondary chunk所在的server
- client将要追加的数据发送给Primary和Secondary服务器，这些服务器将数据写入一个临时位置（不会直接追加到文件中）
- client向primary服务器发送消息，告知所有primary和secondary服务器都有了需要append的数据，可以将数据追加到文件中
- Primary查看当前chunk有足够的剩余空间，然后将数据写入chunk末尾，并通知所有Secondary服务器也执行追加操作
- Secondary追加成功后会返回成功，否则返回写入失败，只有当所有Secondary都返回下写入成功，Primary才会告诉client写入成功，否则告知client写入失败

## 2.7 GFS一致性

GFS并不保证强一致性，从同一个chunk的不同replicas读取到不同数据是正常的

### 2.7.1 GFS对写失败的处理

- 如果写失败，各个replica的状态可能不一致，GFS允许这种情况发生。若最近的一次record append操作失败，则client读到的数据可能包含新append的数据，也可能不包含，取决于其向哪个replica读取数据

### 2.7.2 master如何处理无法联系primary的情况

- master向primary发送租约后，会定期ping primary已确保其存活
- 若master ping primary没收到回复，不一定是primary故障了，可能是primary与secondary之间、primary和client之间仍可以正常通信，但master与primary之间的network通信出现问题，这是由于**网络分区引起的问题**
- 因此master ping primary无法收到回复后不能直接认为primary down，master会直接等待primary的**租约时间结束后**，再分配primary，从而**确保不会同时出现两个primary**

### 2.7.3 应用程序

- 由于GFS不确保强一致性，应用程序必须接受此特性，它可以在写入时为所有记录写入各自的**校验和**，并在读取时进行校验，已确定该数据是否有效
- 由于GFS对Record append的优化，应用程序应尽可能采用Record append而不是write

# 3 System interactions

## 3.3 Atomic Record Append 文件追加

- GFS提供at-least-once保证

  GFS guarantees that the data is written at least once as an atomic unit.

- Client在写失败后尝试重新执行

  Client retries record append if it fails at any replica. As a result, replicas of the same chunk may contain different data possibly including duplicates of the same record in whole or in part.

## 3.4 snapshot 文件快照

- snapshot makes a copy of a file or a directory tree almost instantaneously, using **copy on write （COW）technique**

  > Recall: COW in 6.S081
  >
  > Fork() defers allocating and copying physical memory pages for the child until the copies are actually needed.

- Snapshot

  - master撤回file或directory对应的chunk的租约，因此任意访问这些chunks的clients都必须先经过master
  - master复制其metadata，相应的filename to chunk ID array等数据会先指向先前文件的数据
  - 当执行snapshot后，若有client想写入chunk C，master会注意到其reference count大于1，从而要求相关的chunk-servers都复制一份chunk C'，从而**将复制chunk的时机延迟到真正使用时，减少了不必要的操作**
