---
title: Fault tolerant virtual machine
date: 2021-12-10 20:22:41
tags: 6.824
---

# Intro

- 研究对象：具有容错机制的虚拟机，Fault-Tolerant Virtual Machine
- 架构：primary/backup approach，primary故障时，backup上线，且应用程序感受不到切换过程

# Basic Fault-Tolerant Design

![img](https://img2020.cnblogs.com/blog/1616773/202008/1616773-20200814212404116-959184244.png)

## Deterministic Replay Implementation

### 有限状态机

- 对于两个有限状态机，如果起始状态相同，经过相同内容和顺序的输入后，会产生相同的输出

### Non-deterministic

- VM执行过程中存在不确定因素，主要包含
  - Non-deterministic events（如中断的确切发生时机）
  - Non-deterministic operations（如获取CPU时钟）

### VMware vSphere平台提供的VMware deterministic replay功能

- 该功能会记录所有input和执行中的不确定因素（如中断的确切发生时机，CPU时钟的值等），并将其放在一系列log entries中，最终写入一个log file
- 通过重新执行log，即可实现replay

## Fault-Tolerant Protocol

### log channel

- Primary与Backup通过log channel通信，primary向backup发送log entries以保证同步执行

### Output Rule

- Output requirement：使backup在primary故障后接手时，仍能保证与primary的输出具有一致的行为
- Primary输出到external world前，必须先向backup发送log并收到backup返回的acknowledgement

<img src="C:\Users\young\AppData\Roaming\Typora\typora-user-images\image-20211210204305175.png" alt="image-20211210204305175" style="zoom:80%;" />

### Duplicate output

- backup接受primary后，由于无法确定primary故障时机是在output前还是output后，因此它会重新output
- 由于网络连接机制能够处理duplicate packets（如TCP），因此无需特殊处理

## Detecting and Responding to Failure

### How to detect?

- UDP heartbeating between servers running the primary and backup servers
- A halt in the flow of log entries of acknowledgements

### How to handle?

- **go live**：当一方故障时，另一VM会退出recording mode，停止发送log entries或acknowledgement，像普通虚拟机一样执行
- **Primary fail**：backup会继续执行完log channel的所有log entries，然后提升为primary，并go live
- **Backup fail**：Primary go live即可
- go live后，primary会在另一主机启动新的backup VM

### Split-brain problem

- 如果primary和backup之间的连接丢失，但是primary和backup均未故障，则**两个虚拟机都可能尝试go live**，必须避免出现这种情况
- 因此，规定**go live前必须在shared storage中执行atomic test-and-set操作**，该操作保证同时只有一台VM可能go live

# 具体实现中的问题

## Backup VM的启动

- 借助VMware vSphere平台的VMotion功能
- 对primary的中断一般少于1s

## 管理logging channel

### Buffer

- 虚拟机管理程序维护了log buffer
- Primary将log写入log buffer，Backup从log buffer读取log
- 每当backup从log buffer读取一个log，它就会向primary返回一个ACknowledgement

### Primary buffer full

- 当primary VM的buffer满了，它无法再写入log，因此必须等待任意log entry发送并flush out后才能继续执行
- FT protocol中，log和ACK会携带额外信息，以计算**primary和backup之间的lag**
- primary和backup执行的**闭环控制**
  - 若lag大于一定值，则会通过降低CPU资源分配等措施让primary执行速度下降
  - 若lag小于一定值，则会通过提升CPU资源分配等措施让primary执行速度上升

## 虚拟机层面的操作

- 关机：Primary关机后，backup也应该关机，不要尝试go live
- 资源管理：应该在primary和backup之间同步资源管理操作，如primary提升CPU频率，backup也应该执行同样操作
- VMotion
