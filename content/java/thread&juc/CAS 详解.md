---
title: CAS 详解
date: 2024-10-08T15:21:26+08:00
lastmod: 2024-10-08T15:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/cas.png
images:
  - /img/cas.png
categories:
  - Java
tags:
  - Java
weight: 1
---

# Java CAS 机制详解

## 一、CAS 概述
• **定义**：Compare-And-Swap（比较并交换），一种无锁并发控制机制，属于乐观锁范畴。
• **核心思想**：通过比较内存值与期望值是否一致来决定是否更新，若一致则更新，否则重试或放弃。

---

## 二、Java 中 CAS 的实现
### 1. **Unsafe 类**
• **作用**：提供底层原子操作（位于 `sun.misc.Unsafe`），直接操作内存和硬件指令。
• **关键方法**：
  • `compareAndSwapObject()`：原子更新对象引用。
  • `compareAndSwapInt()`：原子更新 `int` 类型。
  • `compareAndSwapLong()`：原子更新 `long` 类型。
• **特点**：
  • **Native 方法**：通过 JNI 调用本地代码（C++/内联汇编），依赖 CPU 指令（如 x86 的 `CMPXCHG`）。
  • **不推荐直接使用**：设计为 JVM 内部使用，普通开发者应通过 `java.util.concurrent.atomic` 包间接使用。

### 2. **Atomic 原子类**
• **实现原理**：基于 `Unsafe` 的 CAS 方法实现无锁线程安全。
• **示例**：`AtomicInteger` 核心源码分析：
  • **字段偏移量**：静态初始化时通过 `Unsafe.objectFieldOffset()` 获取 `value` 字段的内存偏移量。
  • **volatile 保证可见性**：`private volatile int value`。
  • **原子方法**：
    ```java
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
    ```
  • **自旋机制**：例如 `getAndAddInt()` 通过循环 + CAS 实现原子递增。

---

## 三、CAS 的三大问题及解决方案
### 1. **ABA 问题**
• **原因**：变量被其他线程修改为 B 后又改回 A，CAS 无法感知中间变化。
• **解决方案**：
  • **版本号/时间戳**：使用 `AtomicStampedReference`，在比较值的同时检查版本戳。
  • **示例**：
    ```java
    public boolean compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp) {
        Pair<V> current = pair;
        return expectedReference == current.reference 
               && expectedStamp == current.stamp 
               && ((newReference == current.reference && newStamp == current.stamp) 
                   || casPair(current, Pair.of(newReference, newStamp)));
    }
    ```

### 2. **循环时间长开销大**
• **原因**：高并发下 CAS 失败频繁，导致自旋消耗 CPU 资源。
• **优化手段**：
  • **CPU 指令支持**：`pause` 指令（延迟流水线执行、避免内存顺序冲突）。
  • **适应性自旋**：JVM 根据历史成功率动态调整自旋次数。

### 3. **只能保证单一变量的原子性**
• **限制**：无法直接对多个变量进行原子操作。
• **解决方案**：
  • **封装对象**：使用 `AtomicReference` 将多个变量封装为单一对象。
  • **显式锁**：通过 `synchronized` 或 `ReentrantLock` 保证复合操作的原子性。

---

## 四、CAS 的应用场景
1. **原子类**：如 `AtomicInteger`、`AtomicLong` 等。
2. **并发容器**：`ConcurrentHashMap` 的桶节点操作。
3. **无锁数据结构**：无锁队列、栈等高性能数据结构。

---

## 五、总结
• **优势**：避免线程阻塞，提升高并发场景性能。
• **劣势**：需处理 ABA 问题、自旋开销、多变量限制。
• **适用性**：适合竞争不激烈的场景，替代部分锁的使用以提升效率。

