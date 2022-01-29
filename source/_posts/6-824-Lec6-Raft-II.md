---
title: 6.824 Lec6 Raft II
date: 2021-12-16 21:04:28
tags: 6.824
categories: Distributed System
---

# 日志恢复

## `NextIndex`的作用

- `NextIndex`存储了在leader看来，某个follower下一个应该匹配的log entry位置
- 当leader发送AppendEntries RPC给follower时，follower会对比`prevLogIndex`位置（该位置是`NextIndex`的前一位置）上的log entry，如果该log匹配则从该位置之后开始添加leader发送过来的log entries，**如果不匹配则拒绝该RPC并返回false**
- 如果leader收到来自follower的拒绝，则leader知道该follower在`prevLogIndex`上的log entry不匹配，因此leader会**将`NextIndex`减一**，从而对比更前面的位置是否匹配
- 该过程将持续**直到follower成功匹配`prevLogIndex`上的log entry**，并将leader发送过来的log entry添加到自己的log中

# 选举约束

## 成为leader的限制条件

- 当candidate向其他server发送`RequesetVote` RPC时，只有当candidate的log is **at least as up-to-date as receiver's log**时，server才会向其投票
- up-to-date
  - 如果candidate的**last term大于自己的last term**
  - 或两者一样大但candidate的**last index大于等于自己的last index**时，就认为它是at least as up-to-date as receiver
- 当leader收到过半数选票，即它**比过半数的服务器更加up-to-date时**，才有可能成为leader

# 快速恢复

## 日志恢复存在的问题

- 日志恢复中`NextIndex`每次减一，这对于实际场景太慢

## 快速恢复

- 当follower因为log inconsistent而拒绝leader时，可携带额外信息来加速日志恢复

### ConflictTerm

- follower中与leader冲突的log对应的term，如果follower在对应位置没有log则设为-1

### ConflictIndex

- follower中对应任期号为ConflictTerm的第一条log的index，当follower在对应位置没有log时则返回log的长度

## leader收到reply后的操作

- 搜索其log，找到term等于ConflictTerm的log
  - 若找到了则将nextIndex设为该term最后一个log entry之后的位置
  - 若没有找到则将nextIndex设为ConflictIndex

<!--more-->

# 持久化

## 持久化的原因

- 当一些服务器因为故障（如断电）而退出时，启动另一些服务器去替换它，为此需要将某些**状态信息持久化存储，以便新启动的服务器恢复状态**

## 持久化状态：votedFor、currentTerm、log

- 持久化votedFor和currentTerm是因为我们必须保证server在一个任期内只给一个candidate投票，否则可能产生两个leader
- 持久化log是因为这是能够恢复状态的唯一信息

# 日志快照 log snapshot

## 目的

- 对于一个长期运行的系统，如果让其log持续增长，则最终会导致磁盘空间耗尽
- 此外，如果服务器重启了，需要重投开始执行log来恢复状态，将花费大量时间
- 为此，引入快照snapshot

## 思想

- 应用程序将其状态的拷贝作为一种特殊的log条目（snapshot）存储下来

- **对于大多数应用程序，其状态远小于log的大小**，因此将其状态snapshot并存储下来，可以压缩存储空间

- 当snapshot后，之前的snapshots，以及当前快照对应的最后一个index之前的log entry都可以被**丢弃**，因为恢复时只需要直接加载当前snapshot即可，无需执行之前的log entry

## InstallSnapshot RPC

- 若某些follower远远落后于leader，以至于其nextIndex对应的log entry早已被leader丢弃，此时leader会发送InstallSnapshot RPC并携带snapshot给follower
- follower收到该snapshot后，将其发送给应用程序使其加载该snapshot，从而跟上leader的步伐

# 线性一致

## 正确=线性一致/强一致

- 如果一个服务表现得就像只有一个服务器，并且该服务器没有故障，每次只执行一个客户端请求并正确返回，则认为该服务是**线性一致**的
