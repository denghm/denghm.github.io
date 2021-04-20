---
layout: article
title: "Java并发编程"
modified: 2021-04-20 
author: Voyager
toc: false
comments: true
categories: technology
summary: Java并发编程的理论，工具
---

# 理论
## 并发问题来源
并发问题来源于CPU, 内存，IO之间的速度差异，以及为了提升系统速度而做出的努力。
为了平衡

## 实现机制
分为voting, commit两个阶段

```mermaid
sequenceDiagram
Title: 二阶段提交

participant Coordinator as 事务管理器
participant CohortA as 资源管理器A
participant CohortB as 资源管理器B

Coordinator->>CohortA: CanCommit
CohortA->>Coordinator: Yes
Coordinator->>CohortB: CanCommit
CohortB->>Coordinator: No

Coordinator->>CohortA: DoAbort
CohortA->>Coordinator: HaveCommitted
Coordinator->>CohortB: DoAbort
CohortB->>Coordinator: HaveCommitted
```

## 特性
### 单点故障
事务管理器发生故障

### 数据不一致
DoCommit时发生局部网络故障，接收到的参与者提交请求并执行提交操作，未接到提交请求的参与者则无法执行事务提交。

# 三阶段提交
## 参与者
事务管理器，资源管理器

## 实现机制
分为CanCommit、PreCommit、DoCommit三个阶段

```mermaid
sequenceDiagram
Title: 三阶段提交-正常流程

participant Coordinator as 事务管理器
participant CohortA as 资源管理器A
participant CohortB as 资源管理器B

Coordinator->>CohortA: CanCommit
CohortA->>Coordinator: Yes
Coordinator->>CohortB: CanCommit
CohortB->>Coordinator: Yes

Coordinator->>CohortA: PreCommit
CohortA-->>CohortA: 记录Undo和Redo信息到事务日志
CohortA->>Coordinator: ACK
Coordinator->>CohortB: PreCommit
CohortB-->>CohortB: 记录Undo和Redo信息到事务日志
CohortB->>Coordinator: ACK

Coordinator->>CohortA: DoCommit
CohortA->>Coordinator: ACK
Coordinator->>CohortB: DoCommit
CohortB->>Coordinator: ACK
```

## 特性
### 相比二阶段提交减少单点故障造成的系统阻塞
当参与者在预提交阶段向协调者发送 Ack 消息后，如果长时间没有得到协调者的响应，在默认情况下，参与者会自动将超时的事务进行提交，从而减少整个集群的阻塞时间

### 相比二阶段提交减少数据不一致
DoCommit时发生局部网络故障，参与者会自动将超时的事务进行提交，数据一致

# 基于分布式消息的最终一致性
## 参与者
消息中间件，资源管理器

## 特性
消息中间件会确认各系统的操作结果，若数据一致，则更新消息；若不一致，则回滚操作、删除消息。






