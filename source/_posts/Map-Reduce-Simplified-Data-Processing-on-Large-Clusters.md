---
title: 'Map Reduce: Simplified Data Processing on Large Clusters'
date: 2021-12-04 11:23:07
tags:
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

## 3 Master数据结构

master需要存储的数据包括

- 每个Map/Reduce task的**状态（idle/in-progress/completed）**
- worker machine的标识
- map task完成后生成的R个**intermediate file的位置及其大小**
