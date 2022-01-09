---
title: Raft
date: 2021-12-11 17:35:51
tags: 6.824
mathjax: true
categories: Distributed System
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

<!--more-->

# 处于可理解性的设计

## Raft的设计目标

- 和其他一致性算法一样的基本特征：safety、efficiency等
- Raft的最重要目标是可理解性

## Decomposition

- 将问题分解为多个可被独立理解并解决的部分
- 例如，Raft中分解了leader election，log replication，safety和membership change

## 减少需要考虑的状态数

- 减少系统的不一致情况，从而简化state space
- 大部分情况下需要减少不确定性，但有时引入不确定性可以提高可理解性

# Raft

## 基本概念

### Term

- term是raft算法中的基本时间单元，由election和normal operation两部分组成

### 服务器角色

- 分为follower、candidate、leader
- 在normal operation阶段，1个leader，其余为follower
- 在election阶段，参与竞选leader的服务器为candidate

### 通信方式

- servers通过RPC通信，又可以分为两种

  - `RequestVoteRPC`，由candidate发送给其他servers，请求他们为自己投票

  - `AppendEntriesRPC`，由leader发送给其他servers，有两个用途

    （1）**heartbeat**

    （2）使其向自己的log添加log entries，用于**log replication**

## Leader Election

### 状态转移图

![](role.png)

### heartbeat机制

- leader会定期向followers发送空的`AppendEntriesRPC`，以保持自己的authority

### Start Election

- follower超过限时仍未收到leader heartbeat，则会发起选举

#### follower转变为candidate时会做的事

- term加一
- vote for itself
- 并行地向其他各个servers发送`RequestVoteRPCs`

### Election的可能结果

对于一个candidate，可能的结果是

- 赢得选举，candidate转变为leader
- 其他服务器获胜（收到一个声称它为leader的`AppendEntriesRPC`，且该服务器的term不小于自己），candidate转变为follower
- 其他服务器诈胡（收到一个声称它为leader的`AppendEntriesRPC`，但是服务器的term小于自己），则candidate会拒绝该RPC，并重新开始选举，candidate转变为candidate
- 超过限时后仍没有赢家，没有人获得超过半数的投票，则candidate会发起新的选举，candidate转变为candidate

### 防止无限Start Election的机制

- 若始终没有服务器获得超过半数的选票，则会无休止的重新start election，为此，raft使用了

#### 随机的超时时间

- follower超过一定限时没收到heartbeat，会成为candidate开始选举
- candidate超过一定限时没有赢家，会重新发起选举
- 这两个限时都是随机的结果，这样保证了不会有无休止的start election

## Log replication

### 发送AppendEntriesRPC

- leader收到client的请求后，首先将包含command的log entry添加至自己的log
- 然后并行地向所有followers发送AppendEntriesRPC
- 若没有收到来自followers的确认，则不停地重新发送

### log entry包含的信息

- state machine command
- term number，当前任期
- integer index，在log中的位置

### commit

- 当超过半数的servers拥有该log entry时，则可以commit
- leader将该log entry送入state machine，并将输出结果返回给client

### 如何保持一致性

- 当leader与follower的log不一致时，leader会让follower复制自己的log
- leader对每个follower都维持了一个**nextIndex数组元素**，当前index的log entry不一致时，leader会**回退该follower的nextIndex变量，重新执行AppendEntriesRPC**，直至log entry一致，然后从此处开始让follower复制自己的log entry

## Safety

### 选举的限制

- 成为leader的candidate的log，必须包含所有已提交的log entries（**hold all the committed entries**）
- 推论：If candidate's log is **at least as up-to-date as any other logs** in that majority, it will hold all the committed entries.
- **RequestVoteRPC实现**
  - 如果最后一个entry的term更大，则更加up-to-date
  - 如果最后一个entry的term相等，则log长度更长的更up-to-date

### 提交之前term的entries的限制

- 对于非当前term的entries，leader不能根据其在服务器超过半数而判断其能被commit

- 反例

  ![image](figure8.png)

- leader只能通过**Log matching property间接地提交非当前term的entries**（即在当前term提交一个entry后，比当前term更小的term的log entry都可以安全提交）

### 证明Leader Completeness property

- If a log entry is committed in a given term, then that entry will be present in the logs of the leaders for all higher-numbered terms
- 使用反证法证明，详见论文5.4.3

## Timing and Availability

- 当下式满足时，可认为Raft可以稳定地维持一个leader

  $$broadcastTime \ll electionTimeout \ll MTBF$$

  - broadcastTime是一台服务器向其他所有服务器发送RPCs并收到回复的平均用时
  - MTBF是服务器的平均故障时间间隔

