---
title: Lec5 Raft-I
date: 2021-12-12 17:00:30
tags: 6.824
---

# Split brain问题

## Primary结构

- MapReduce、GFS、FT-VM都采用了primary结构

## Split brain in FT-VM

![](split brain.png)

- S1和S2通过网络形式提供Test and Set服务，C1和C2相当于FT-VM中的primary和backup
- 假设网络故障，C1和C2无法通信，则它们会尝试go live
- 本来C1应该同时与S1和S2通信，并进行Test and Set，但是C1现在只能与S1通信，同理C2只能与S2通信
- 若允许client只与一台服务器通信执行Test and Set便上线，则上述情况可能导致C1和C2同时go live，**造成split brain问题**；若必须与两台服务器通信，则Test and Set**无法容错**，且两台replica反而使得可靠性下降

## Solution

上述问题主要通过两种方式解决

- **无故障网络**：保证网络可靠无故障，需要大量资金
- **人工维护**：当C无法与其中一台服务器联系时，则联系机房人员，将该服务器关闭，从而避免了Split brain

# Majority Vote

- primary的决策只需得到过半数投票即可执行，无需得到全部服务器投票，从而实现容错
- 若系统有2F+1台服务器，则最多可接受F台服务器故障

# Raft初探

- Raft以库的形式存在于服务中，其上层是应用程序
- 对于客户端来说，多个Raft节点是透明的，它们只会认为其在向一个应用程序请求服务

# log同步时序

- client向leader发送请求，leader在add log之后，同时向其他服务器发送AE
- 得到过半数服务器的确认后，则leader commit log，让state machine执行相应的command并将结果返回client
- leader还会通知其他所有server也commit log，但是Raft不会发送额外消息，它会将该信息附在下一次AE中

# Leader election

## Election timer

- 若当前节点长时间没收到leader的心跳信号，计时器time out时，会开始选举

## Election的产生

- leader故障，会导致新的选举
- 即使leader不故障，由于网络延迟导致follower超时没收到leader的心跳信号，也会开始新的选举

## 如何确保旧leader的行为正确

- 如果是由于网络分区等原因导致的新选举，则可能旧leader无法得知新leader的产生
- 但由于Raft的思想是leader的决策必须得到过半数投票，因此新leader的产生意味着它得到了过半数投票，此时旧leader的任何行为都无法得到过半数投票（这很有可能是由于网络分区，且旧leader所在的网络分区节点数较少，而新leader所在的网络分区节点数过半导致的）

# Election timer

## reset

- 每次follower收到AppendEntriesRPC便会重设timer

## Randomized timeout

- 为了避免Split vote，被刺reset时都会随机选择time out

## timeout的合理取值

- 大于心跳信号间隔，一般选择为心跳信号间隔的几倍
- 小于server的平均故障时间间隔

