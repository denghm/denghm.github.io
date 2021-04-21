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
1. 为了弥补CPU与内存的速度差异，引入缓存，这带来了**可见性**问题；
2. 为了弥补CPU与IO设备的速度差异，引入进程，线程，CPU分时复用，这带来了**原子性**问题；
3. 为了合理利用缓存，编译程序优化指令执行，这带来了**有序性**问题。

## 解决可见性和有序性问题——Java内存模型
因为缓存和编译优化带来了可见性和有序性问题，为了解决这两个问题，就需要**部分禁用缓存和编译优化**，这一解决方案统称为**Java内存模型**。
具有的方法包括: **volatile**、**sychronized**、**final**三个关键字，以及六项happens-before规则。
1. 程序的顺序性规则
前面的操作Happens-Before后面的操作
2. volatile变量规则
对一个volatile变量的写操作，Happens-Before于后面的读操作
3. 传递性规则
这条规则是指如果 A Happens-Before B，且 B Happens-Before C，那么 A Happens-Before C。
4. 管程的锁规则
对一个锁的解锁Happens-Before后续对这个锁的加锁。
5. 线程start()规则
主线程启动子线程后，子线程可以看到主线程在启动之前的操作
6. 线程的join()规则
它是指主线程 A 等待子线程 B 完成（主线程 A 通过调用子线程 B 的 join() 方法实现），当子线程 B 完成后（主线程 A 中 join() 方法返回），主线程能够看到子线程的操作。

## 解决原子性问题——互斥锁
互斥锁确保同一时刻只有一个线程执行某一段代码，该段需要互斥执行的代码称为**临界区**。
锁与其保护的资源存在1:N关系。

![锁与资源的关系](https://raw.githubusercontent.com/denghm/denghm.github.io/master/images/technology/locked-resource.png "锁与资源的关系")

### Java语言提供的锁 Synchronized
Java 语言提供的 synchronized 关键字，就是锁的一种实现, synchronized 关键字可以用来修饰方法，也可以用来修饰代码块。
Java对象的对象头是synchronized的实现基础。对象头主要结构是由Mark Word 和 Class Metadata Address组成，其中Mark Word存储对象的hashCode、锁信息或分代年龄或GC标志等信息，Class Metadata Address是类型指针指向对象的类元数据，JVM通过该指针确定该对象是哪个类的实例。
锁也分不同状态，JDK6之前只有两个状态：无锁、有锁（重量级锁），而在JDK6之后对synchronized进行了优化，新增了两种状态，总共就是四个状态：无锁状态、偏向锁、轻量级锁、重量级锁，其中无锁就是一种状态了。锁的类型和状态在对象头Mark Word中都有记录，在申请锁、锁升级等过程中JVM都需要读取对象的Mark Word数据。
每一个锁都对应一个monitor对象，在HotSpot虚拟机中它是由ObjectMonitor实现的（C++实现）。每个对象都存在着一个monitor与之关联，对象与其monitor之间的关系有存在多种实现方式，如monitor可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个monitor被某个线程持有后，它便处于锁定状态。
```C++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0;  //锁计数器
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```
ObjectMonitor中有两个队列\_WaitSet和\_EntryList,用来保存ObjectWaiter对象列表，\_owner持有指向ObjectMonitor对象的线程。
当多个线程访问同步代码时，首先会进入\_EntryList集合，当线程获取到对象的monitor后进入\_Owner区域并把monitor中的\_owner变量设置为当前线程，同时monitor中的计算器\_count加1。
若线程调用wait()方法，将释放当前持有的monitor, \_owner变量恢复为null,  \_count减一, 该线程进入 \_WaitSet集合中等待被唤醒。

|Thread| \_EntryList | \_Owner | \_count | \_WaitSet |
|----------|------|---------|---------|---------------|
|ThreadA|ThreadA|ThreadA|+1||
|ThreadA.wait()||null|-1|ThreadA|

## 死锁及其解决方案
### 死锁的形成条件
要形成死锁，必须满足下面的4个条件
1. 互斥
2. 占有且等待
3. 不可抢占
4. 循环等待
破坏其中一个，就可以避免死锁的发生。

1. 破坏“占有且等待”，可以一次申请所有资源；
2. 破坏”不可抢占“，占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源；
3. 破坏"循环等待”，可以按序申请资源。

### 占用且等待的破坏方案
#### 死循环来等待释放锁
#### 等待-通知机制
![等待-通知机制](https://raw.githubusercontent.com/denghm/denghm.github.io/master/images/technology/thread-wait-notify.png "等待-通知机制")

## 并发编程的问题
### 安全性
本质上就是正确性，就是程序按照我们期望的执行，不要让我们感到意外。
数据竞争，存在共享数据，多个线程读取同一份数据
竞态条件, 程序执行的结果依赖程序执行的顺序
### 活跃性
#### 活锁
#### 解决办法
需要在谦让时，等待一个随机的时间

### 性能
#### 阿姆达尔（Amdahl）定律
#### 解决办法
#### 度量指标
吞吐量: 单位时间内处理的请求数量
延迟: 发出请求到收到响应的时间
并发量： 同时处理的请求数量

## 管程
### 管程
管程，指的是管理共享变量以及对共享变量的操作过程，让它们支持并发。管程支持两类并发领域的经典问题：互斥和并发。

互斥帮助线程避免彼此干扰，并发帮助线程共同完成同一任务。



### MESA模型










