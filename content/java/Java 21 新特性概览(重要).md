---
title: Java 21 新特性概览(重要)
date: 2024-07-15T14:21:26+08:00
lastmod: 2024-07-16T14:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/java.png
images:
  - /img/java.png
categories:
  - Java
tags:
  - Java
weight: 1
---

### Java 21 新特性概览

#### **1. JDK 21 概述**
• **发布信息**：2023年9月19日发布，是长期支持版本（LTS），目前LTS版本包括JDK 8、11、17和21。
• **核心目标**：增强语言表达能力、优化性能、简化开发流程。
• **新特性总数**：共15个JEP（JDK Enhancement Proposal），以下选取关键特性详细介绍。

---

#### **2. JEP 430: 字符串模板（预览）**
• **核心功能**：通过占位符`${}`动态构建字符串，支持嵌入变量、方法调用和表达式。
• **模板处理器**：
  • **STR**：自动插值，如`STR."Greetings \{name}!"`。
  • **FMT**：支持格式化（类似`printf`），如`FMT."Greetings %-12s\{name}!"`。
  • **RAW**：返回`StringTemplate`对象，需手动处理。
• **优势**：简化字符串拼接，提升可读性和灵活性。
• **示例**：
  ```java
  String name = "Java";
  String message = STR."Hello \{name.toUpperCase()}!"; // 输出：Hello JAVA!
  ```

---

#### **3. JEP 431: 序列化集合（Sequenced Collections）**
• **目标**：统一有序集合的API，强化对首尾元素和反向视图的操作。
• **新接口**：
  • **SequencedCollection**：扩展`Collection`，提供`addFirst`、`getLast`、`reversed()`等方法。
  • **SequencedSet**：专为集合设计，如`LinkedHashSet`。
  • **SequencedMap**：扩展`Map`，支持`putFirst`、`pollLastEntry`等。
• **示例**：
  ```java
  LinkedHashSet<Integer> set = new LinkedHashSet<>(List.of(3, 1, 2));
  set.addFirst(0); // 结果：[0, 3, 1, 2]
  SequencedSet<Integer> reversed = set.reversed(); // 反向视图：[2, 1, 3, 0]
  ```

---

#### **4. JEP 439: 分代 ZGC**
• **改进点**：引入分代垃圾回收机制，减少停顿时间。
• **启用方式**：需显式开启参数`-XX:+UseZGC -XX:+ZGenerational`。
• **未来计划**：分代ZGC将设为默认，最终移除非分代ZGC。
• **适用场景**：适合大型应用和高并发环境，提升吞吐量和响应速度。

---

#### **5. JEP 440: 记录模式（转正）**
• **功能**：允许在模式匹配中解构记录（Record）对象。
• **示例**：
  ```java
  record Point(int x, int y) {}
  if (obj instanceof Point(int x, int y)) {
    System.out.println(x + ", " + y); // 解构Point对象
  }
  ```

---

#### **6. JEP 441: switch 模式匹配（转正）**
• **增强功能**：支持类型模式和守卫条件。
• **示例**：
  ```java
  static String format(Object obj) {
    return switch (obj) {
      case Integer i -> "int: " + i;
      case String s && !s.isEmpty() -> "非空字符串: " + s; // 守卫条件
      default -> "其他类型";
    };
  }
  ```

---

#### **7. JEP 442: 外部函数与内存API（第三次预览）**
• **目标**：替代JNI，安全高效地调用本地代码和管理堆外内存。
• **关键类**：`MemorySegment`、`SymbolLookup`、`Linker`。
• **应用场景**：与C/C++库交互，如调用系统函数或处理二进制数据。

---

#### **8. JEP 443: 未命名模式与变量（预览）**
• **语法改进**：使用`_`忽略未使用的变量或模式组件。
• **示例**：
  ```java
  try (var _ = resource.acquire()) { ... } // 忽略资源变量
  if (point instanceof Point(_, int y)) { ... } // 忽略x坐标
  ```

---

#### **9. JEP 444: 虚拟线程（转正）**
• **核心价值**：轻量级线程，大幅提升高并发应用性能。
• **使用方式**：
  ```java
  try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> System.out.println("虚拟线程任务"));
  }
  ```
• **优势**：降低线程创建和上下文切换开销。

---

#### **10. JEP 445: 未命名类与实例main方法（预览）**
• **简化入口**：允许省略类名和`static`修饰符。
• **示例**：
  ```java
  void main() { // 无需类名和public static
    System.out.println("Hello, Simplified World!");
  }
  ```

---

#### **其他JEP概览**
• **JEP 433: 结构化并发（预览）**：统一多线程任务管理。
• **JEP 446: Scoped Values（预览）**：替代线程局部变量，提升性能。
• **JEP 448: Vector API（第六次孵化）**：优化向量计算。

---

#### **参考与总结**
• **升级建议**：LTS版本适合生产环境，建议评估虚拟线程、分代ZGC等特性。
• **注意事项**：预览功能需通过`--enable-preview`启用，可能存在变更。
• **官方文档**：[Oracle JDK 21 Release Notes](https://docs.oracle.com/en/java/javase/21/)
