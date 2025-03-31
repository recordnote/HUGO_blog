---
title: Atomic 原子类总结
date: 2024-10-26T15:21:26+08:00
lastmod: 2024-10-26T15:21:26+08:00
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

### Java Atomic 原子类总结

---

#### **一、概述**
1. **定义**  
   Atomic 原子类位于 `java.util.concurrent.atomic` 包下，提供线程安全的原子操作，通过 CAS（Compare And Swap）和 volatile 实现无锁并发，避免传统锁机制的性能开销。

2. **核心原理**  
   • **CAS 机制**：通过比较内存值与预期值是否一致来决定是否更新，避免线程阻塞。
   • **volatile 变量**：保证变量的可见性，确保多线程环境下能读取到最新值。

---

#### **二、原子类分类**
##### 1. **基本类型原子类**
• **作用**：原子更新基本类型变量。
• **类**：
  • `AtomicInteger`：整型原子类。
  • `AtomicLong`：长整型原子类。
  • `AtomicBoolean`：布尔型原子类。
• **常用方法**：
  ```java
  get()、set()、getAndSet()、getAndIncrement()、compareAndSet()
  ```

##### 2. **数组类型原子类**
• **作用**：原子更新数组中的单个元素。
• **类**：
  • `AtomicIntegerArray`：整型数组原子类。
  • `AtomicLongArray`：长整型数组原子类。
  • `AtomicReferenceArray`：引用类型数组原子类。
• **常用方法**：
  ```java
  get(int i)、set(int i, T newValue)、getAndUpdate(int i, IntUnaryOperator updateFunction)
  ```

##### 3. **引用类型原子类**
• **作用**：原子更新对象引用。
• **类**：
  • `AtomicReference`：普通引用类型原子类。
  • `AtomicStampedReference`：通过版本号解决 ABA 问题。
  • `AtomicMarkableReference`：通过布尔标记解决 ABA 问题。
• **ABA 问题**：  
  线程 1 读取值 A，线程 2 将值改为 B 后又改回 A，线程 1 的 CAS 操作会误认为值未变。通过版本号或标记解决。

##### 4. **对象属性修改类型**
• **作用**：原子更新对象的某个字段（需使用 `public volatile` 修饰）。
• **类**：
  • `AtomicIntegerFieldUpdater`：整型字段更新器。
  • `AtomicLongFieldUpdater`：长整型字段更新器。
  • `AtomicReferenceFieldUpdater`：引用类型字段更新器。
• **使用步骤**：
  1. 通过静态方法 `newUpdater()` 创建更新器。
  2. 调用更新器的原子方法（如 `getAndIncrement()`）。

---

#### **三、核心方法解析**
1. **CAS 方法**  
   ```java
   boolean compareAndSet(T expect, T update)
   ```
   • 若当前值等于 `expect`，则更新为 `update`，返回 `true`；否则返回 `false`。

2. **延迟写入方法**  
   ```java
   void lazySet(T newValue)
   ```
   • 最终将值设为 `newValue`，但不保证其他线程立即看到新值（弱内存语义）。

3. **弱一致性方法**  
   ```java
   boolean weakCompareAndSet(T expect, T update)
   ```
   • 可能虚假失败，不提供内存顺序保证（实际实现与 `compareAndSet` 相同）。

---

#### **四、典型示例**
##### 1. `AtomicInteger` 使用
```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet(); // 线程安全的自增
```

##### 2. `AtomicStampedReference` 解决 ABA
```java
AtomicStampedReference<Integer> ref = new AtomicStampedReference<>(0, 0);
int[] stampHolder = new int[1];
ref.compareAndSet(0, 100, 0, 1); // 检查值是否仍为 0，且版本号未变
```

##### 3. `AtomicIntegerFieldUpdater` 更新对象字段
```java
class User {
    public volatile int age;
}
AtomicIntegerFieldUpdater<User> updater = 
    AtomicIntegerFieldUpdater.newUpdater(User.class, "age");
User user = new User();
updater.getAndIncrement(user); // 原子更新 age 字段
```

---

#### **五、应用场景**
1. **计数器**：如统计请求次数。
2. **状态标志**：如原子更新布尔状态。
3. **对象引用更新**：如单例模式的双重检查锁优化。

---

#### **六、注意事项**
1. **ABA 问题**：在需要严格版本控制的场景，必须使用 `AtomicStampedReference`。
2. **字段可见性**：使用字段更新器时，字段必须声明为 `public volatile`。
3. **性能权衡**：高并发场景下，CAS 可能因自旋次数过多导致性能下降。

---

#### **七、参考资料**
• 《Java 并发编程的艺术》
• [Oracle 官方文档：java.util.concurrent.atomic](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/package-summary.html)

通过合理使用原子类，可以显著提升多线程程序的性能和可维护性。
