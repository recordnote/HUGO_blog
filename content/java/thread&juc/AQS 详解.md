---
title: AQS 详解
date: 2024-10-15T15:21:26+08:00
lastmod: 2024-10-15T15:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/juc.png
images:
  - /img/juc.png
categories:
  - Java
tags:
  - Java
weight: 1
---

### AQS（AbstractQueuedSynchronizer）核心原理与源码解析

#### 一、AQS概述
1. **定义与作用**  
   • AQS是Java并发包中的基础框架（`java.util.concurrent.locks`包下的抽象类），用于构建锁和同步器。
   • 提供通用功能实现（如线程排队、状态管理），简化同步器的开发，如`ReentrantLock`、`Semaphore`、`CountDownLatch`等均基于AQS。

2. **解决的问题**  
   • 封装底层线程同步机制，隐藏线程管理复杂性。
   • 通过模板方法模式定义资源获取/释放的通用流程，开发者只需关注具体同步逻辑。

---

#### 二、AQS核心组件
1. **同步状态（`state`变量）**  
   • **作用**：通过`volatile int state`表示共享资源状态（如锁的持有次数、信号量许可数等）。
   • **操作方法**：  
     ◦ `getState()`：获取当前状态。
     ◦ `setState(int)`：直接设置状态。
     ◦ `compareAndSetState(int, int)`：原子更新状态（CAS操作）。

2. **CLH变体队列**  
   • **结构**：双向队列（原CLH为单向），每个节点代表等待线程，包含`thread`、`waitStatus`、`prev`、`next`字段。
   • **改进**：
     ◦ **自旋+阻塞**：减少CPU占用，先短暂自旋尝试获取资源，失败后阻塞。
     ◦ **双向指针**：便于唤醒后继节点，提升效率。

3. **节点状态（`waitStatus`）**  
   | 状态值           | 含义                                                       |
   | ---------------- | ---------------------------------------------------------- |
   | `SIGNAL` (-1)    | 后继节点需被唤醒，当前节点释放资源后会通知后继节点。       |
   | `CANCELLED` (1)  | 节点因超时或中断取消等待，需从队列中移除。                 |
   | `CONDITION` (-2) | 节点处于条件等待队列，等待`signal()`后被转移到同步队列。   |
   | `PROPAGATE` (-3) | 共享模式下传播唤醒操作，解决并发场景下线程无法唤醒的问题。 |
   | 0                | 初始状态，新节点加入队列时的默认值。                       |

---

#### 三、AQS工作流程
1. **模板方法模式**  
   • **钩子方法**：子类需重写以下方法实现具体同步逻辑：
     ◦ `tryAcquire(int)` / `tryRelease(int)`：独占模式获取/释放资源。
     ◦ `tryAcquireShared(int)` / `tryReleaseShared(int)`：共享模式获取/释放资源。
     ◦ `isHeldExclusively()`：判断当前线程是否独占资源。

2. **独占模式（如`ReentrantLock`）**  
   • **获取资源**：
     1. 调用`acquire(int)`尝试获取资源。
     2. 失败后通过`addWaiter()`将线程封装为节点加入队列。
     3. 在`acquireQueued()`中自旋或阻塞，等待前驱节点释放资源。
   • **释放资源**：
     1. `release(int)`调用`tryRelease(int)`释放资源。
     2. 若资源完全释放（`state=0`），唤醒后继节点。

3. **共享模式（如`Semaphore`）**  
   • **获取资源**：
     ◦ `acquireShared(int)`尝试获取资源，失败后加入队列。
     ◦ 被唤醒后若剩余资源足够，传播唤醒后续节点（`setHeadAndPropagate()`）。
   • **释放资源**：
     ◦ `releaseShared(int)`释放资源，通过`doReleaseShared()`唤醒后续节点。
     ◦ **PROPAGATE状态**：解决并发释放资源时可能出现的唤醒遗漏问题。

---

#### 四、关键源码分析
1. **独占模式获取资源（以`ReentrantLock`为例）**  
   • **非公平锁`nonfairTryAcquire()`**：
     ```java
     final boolean nonfairTryAcquire(int acquires) {
         Thread current = Thread.currentThread();
         int c = getState();
         if (c == 0) { // 资源未被占用
             if (compareAndSetState(0, acquires)) {
                 setExclusiveOwnerThread(current); // 设置持有线程
                 return true;
             }
         } else if (current == getExclusiveOwnerThread()) { // 重入
             setState(c + acquires);
             return true;
         }
         return false;
     }
     ```

2. **共享模式释放资源（以`Semaphore`为例）**  
   • **释放逻辑**：
     ```java
     protected boolean tryReleaseShared(int releases) {
         for (;;) {
             int current = getState();
             int next = current + releases;
             if (compareAndSetState(current, next)) {
                 return true;
             }
         }
     }
     ```

---

#### 五、基于AQS的同步工具
1. **`ReentrantLock`**  
   • **特性**：可重入、支持公平/非公平锁。
   • **实现**：通过`state`记录锁的持有次数，独占模式实现。

2. **`Semaphore`**  
   • **作用**：控制并发线程数，`state`表示可用许可数量。
   • **模式**：公平/非公平，通过共享模式实现。

3. **`CountDownLatch`**  
   • **原理**：`state`初始化为任务数，线程调用`countDown()`递减，`await()`阻塞至`state=0`。

4. **`CyclicBarrier`**  
   • **特点**：可循环使用，基于`ReentrantLock`和`Condition`实现。
   • **与`CountDownLatch`区别**：支持重置计数器，允许执行屏障任务。

---

#### 六、总结
• **AQS核心思想**：通过CLH队列管理等待线程，结合CAS操作和状态机实现高效同步。
• **设计优势**：模板方法模式提供扩展性，双向队列和状态流转优化唤醒效率。
• **应用场景**：适用于需要精确控制资源访问的同步工具，如锁、信号量、倒计时器等。

通过深入理解AQS的设计，开发者可以更高效地实现自定义同步器，并掌握Java并发工具底层原理。
