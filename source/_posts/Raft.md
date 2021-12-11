---
title: Raft
date: 2021-12-11 17:35:51
tags: 6.824
---

# Intro

## 从Paxos到Raft

- Paxos算法一经提出，便成为了一致性算法的代言词，但是它晦涩难懂，对分布式系统教学、实际应用中的分布式系统搭建非常不友好，因此作者提出了一种新的易于理解的一致性算法，即Raft

## Raft的一些特征

- Strong leader
- Leader election
- Membership change

# Replicated State Machine复制状态机

## 复制状态机的应用

- replicated state machine被广泛应用于分布式系统的容错问题

## 复制状态机模型

- server由**consensus module、log、state machine**等模块组成
- client向server发送请求后，其中一个server的consensus module收到指令将其添加在log，同时与其他server的consensus module通信，以保证**所有server的log都以同样的顺序保存了同样的请求**
- state machine执行该请求，并将输出返回client

## 一致性算法的作用

- consensus algorithm必须保证所有replicated logs的一致性

## 对一致性算法的要求

- 在所有非拜占庭情况下，保证safety（never returning an incorrect result），包括网络延迟、分区、丢包、重复包、乱序等
- 高可用性，只要集群中大多数机器能运行，且能够相互通信以及和客户端通信，集群就是可用的
- 一致性不依赖于timing，faulty clocks和极端情况下的消息延迟最多只引起可用性问题
- 少部分慢的机器不影响系统的整体性能

# Paxos的缺点

- 晦涩难懂
- 难以用于建立实际系统，如没有提供multi-Paxos的具体做法

## 处于可理解性的设计

## Raft的设计目标

- 和其他一致性算法一样的基本特征：safety、efficiency等
- Raft的最重要目标是可理解性

## Decomposition

- 将问题分解为多个可被独立理解并解决的部分
- 例如，Raft中分解了leader election，log replication，safety和membership change

## 减少需要考虑的状态数

- 减少系统的不一致情况，从而简化state space
- 大部分情况下需要减少不确定性，但有时引入不确定性可以提高可理解性

