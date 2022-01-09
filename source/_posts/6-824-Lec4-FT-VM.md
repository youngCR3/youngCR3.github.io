---
title: 6.824 Lec4 FT-VM
date: 2021-12-11 16:23:10
tags: 6.824
categories: Distributed System
---

# 复制

## 复制能解决的故障

- fail-stop fault，即一旦故障则停止运行，而不会产生错误结果的情况

## 复制不能够解决的故障

- 由于硬件漏洞、软件bug造成的运算结果错误

## 关联错误对复制的影响

- replicas之间的故障应该是相互独立的，否则复制可能无效，因为一个replica上的错误在另一replica上很可能再次发生

### 对实际应用的指导

- 两个replicas的服务器最好购自于不同厂商
- 两个servers的布置应保证物理隔离，有独立的供电系统
- 为了防止地震、台风等自然灾害影响，可将不同servers布置在不同城市

## replica是否值得

- 可从故障带来的经济损失进行考量

# 复制的模型

## State transfer

- primary将能表示当前状态的完整信息发送给backup，backup通过该信息执行并还原该状态
- 缺点是传输信息太多，降低性能

## 复制状态机

- 考虑有限状态机模型，若两个状态机的起始状态相同，且输入的内容和顺序一致，则必定产生相同的输出，因此primary只需将其输入传给backup即可
- 优点是传输信息少，缺点是VM过程中存在Non-deterministic event

## VMware的实现

- VMware使用的是**复制状态机模型**，为了应对非确定性事件或操作，primary必须将相关信息发送给backup，使得backup可以准确复制该状态
- 当任一VM故障时，剩下的primary VM必须在另一服务器上**启动新的backup VM，此时只能使用state transfer**保证该VM的状态与当前primary VM一致
- VMware FT的复制：**复制完整的机器状态，包括所有内存和寄存器**
- VMware没有考虑的因素：多核会带来不确定性，而**VMware只考虑两个VM都是单核的情况**

<!--more-->

# VMware公司介绍

## 核心技术

- 该公司是一个虚拟机公司
- 其核心技术是能在一台计算机上运行Virtual Machine Monitor, VMM，它能模拟出多个虚拟计算机，每个都可运行在不同的操作系统上

# 不确定性事件

## 客户端输入

- 客户端输入的**packet的内容**
- 客户端输入的packet到达VM触发**中断的具体时机**

## 怪异指令Wired Instructions

- 随机数生成器
- 获取当前CPU时钟
- 获取计算机唯一ID

## 多核

- omitted by VM-FT paper

## log channel传输的log entry需要包含的信息

- 事件发生时的指令序号，用于确定**事件的具体时机**
- log entry**类型**，客户端输入或怪异指令
- 数据，如果是客户端输入则是**packet content**，如果是**怪异指令就是其执行结果**

# Test-and-Set服务

- 为了保证primary和backup通信失败时，不会出现两台VM同时go live的情况，VM go live前必须调用Test-and-Set服务，该服务**保证了同时最多只有一个VM go live**
- VMware**以网络服务的方式提供Test-and-Set**，**Test-and-Set是单点故障**，最好以容错、复制系统的方式实现，否则Test-and-Set服务的故障将导致容错虚拟机的故障
