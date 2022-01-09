---
title: 'Map Reduce: Simplified Data Processing on Large Clusters'
date: 2021-12-04 11:23:07
tags: 6.824
categories: Distributed System
---

## 1 Google的机器配置

文章首先介绍了当时Google使用MapReduce的典型场景中的**机器配置环境**，其特点是**large clusters of commodity PCs connected together with switched Ethernet**

- **多个PC集群**通过以太网连接，每个PC集群包含成百上千台PC
- 使用商业网络硬件，**速度可达100MB/s-1GB/s**
- 数据存储在**普通IDE硬盘**上，通过**冗余配置**（文件拷贝）增加可靠性
- 由**调度系统**负责任务分发

## 2 任务执行流程

- 输入文件被划分为**M份**，每份大小是16-64MB

- master负责任务分发，共有**M个map任务和R个reduce任务**

- Map worker读取其对应的input file，将数据输入**map function**，产生intermediate K/V，并**缓存在内存中**

- 缓存的intermediate K/B被**定期写入local disks**，其**位置传入master**

- master将地址告知reduce worker，它们从对应地址的local disks读取intermediate data，并对数据**排序**

  > 排序是因为一个Reduce worker可能需要处理多个intermediate key对应的数据，而数据来自于多个Map workers，因此需要将相同intermediate key的数据放在一起

- reduce worker按intermediate key将数据分次输入**reduce function**，并将结果写入**output file**
- 所有Map和Reduce tasks完成后，master将返回**user program**

<!--more-->

## 3 Master数据结构

master需要存储的数据包括

- 每个Map/Reduce task的**状态（idle/in-progress/completed）**
- worker machine的标识
- map task完成后生成的R个**intermediate file的位置及其大小**

## 4 容错

- 由于MapReduce依赖于大量计算机，故障在所难免，因此必须考虑容错机制

### 4.1 Worker failure

1. 如何发现故障？

   master周期性地ping workers，若一定时间后仍未收到回复，则master将该worker标记为failed

2. 故障如何处置？

   （1）对于map worker，其任务都将被恢复为idle状态，并重新分配给其他workers

   （2）对于reduce worker，其未完成或进行中的任务恢复为idle状态，并分配给其他workers，**已完成的任务无需再次执行**

   > 这是因为map worker产生的intermediate file将写入local disks，若map worker故障导致其local disks无法读取，因此其任务（无论是否完成）必须重新执行。对于reduce worker，其输出文件将写入GFS，即使reduce worker故障能够被读取。

### 4.2 Master failure

1. master将定期地写其**data structures的checkpoints**
2. 当master故障时，可以将其任务复制并**从上一个checkpoint处继续执行**。当如果只有一个master，则其故障意味着整个MapReduce任务的终止

### 4.3 故障情况下的正确性保证

1. map和reduce都是其输入值的确定函数，即输出结果只与其输入参数有关

2. 这依赖于atomic commits of map and reduce task outputs

   - 对于map，当它完成任务时，会生成R个intermediate files，并将其名字告知master，master将在其data structure中查找该task的完成情况，若未完成则记录R个文件的名字
   - 对于reduce，完成任务时worker会将文件命名指定为output file的指定名字，而GFS提供了atomic rename operation

   以上机制保证了**对于每个map或reduce task，其输出结果是唯一的**

## 5 Locality

MapReduce如何减少网络带宽使用？

- 尽量将input file**分配给本地或相邻的worker**进行map操作
- 每份文件都有**若干份（如3份）拷贝**，因此可以对每个拷贝查询其所在worker或邻近worker是否空闲

所以**大部分input data都无需通过网络传输**即可执行map operation

## 6 任务的细粒度

1. map被划分为M份， reduce划分为R份

2. master需要进行`O(M+R)`个调度决定，并存储`O(MR)`个状态变量

3. 通常M的大小保证输入文件被划分为每份16-64MB，R的大小是总worker的几倍

   > We often perform MapReduce computations with M=200,000 and R=5,000 using 2,000 worker machines.

## 7 Backup tasks

1. 系统中可能出现个别的执行地异常慢的workers（**straggler**），可能原因是：bad disk/other tasks on the machine/bug等
2. **当一个操作快完成时，将其in-progress的任务分发给backup worker执行**，只要原worker或backup worker任一个完成任务，则将任务标记为completed
