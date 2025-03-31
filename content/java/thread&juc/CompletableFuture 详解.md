---
title: CompletableFuture 详解
date: 2024-11-30T15:21:26+08:00
lastmod: 2024-11-30T15:21:26+08:00
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

## CompletableFuture 详解

### **1. 核心概念**
• **异步编程模型**：通过异步执行任务提升程序性能，避免线程阻塞等待。
• **Future 的局限性**：
  • 不支持任务编排（如链式调用、组合任务）。
  • `get()` 方法阻塞线程，影响效率。
• **CompletableFuture 的优势**：
  • 支持函数式编程和任务编排（如链式调用、并行执行）。
  • 提供丰富的 API 处理异步结果和异常。

---

### **2. 核心功能与使用场景**
• **典型场景**：
  • **并行调用无依赖服务**：例如同时获取用户信息、商品详情、物流信息。
  • **任务编排**：例如先获取用户信息，再并行调用商品和物流接口，最后汇总结果。
• **核心目标**：通过异步化减少接口响应时间，提升吞吐量。

---

### **3. 创建 CompletableFuture**
### **3.1 静态工厂方法**
• **`runAsync()`**：执行无返回值的任务（`Runnable`）。
  ```java
  CompletableFuture.runAsync(() -> System.out.println("Task"));
  ```
• **`supplyAsync()`**：执行有返回值的任务（`Supplier`）。
  ```java
  CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Result");
  ```
• **自定义线程池**（推荐）：避免使用默认的 `ForkJoinPool`，防止资源竞争。
  ```java
  Executor executor = Executors.newFixedThreadPool(10);
  CompletableFuture.supplyAsync(() -> "Result", executor);
  ```

### **3.2 手动创建**
• **`new` 关键字**：用于需要手动控制任务完成时机的场景。
  ```java
  CompletableFuture<String> future = new CompletableFuture<>();
  future.complete("Result"); // 手动设置结果
  ```

---

### **4. 处理异步结果**
##### **4.1 链式处理**
• **`thenApply()`**：转换结果（同步执行）。
  ```java
  future.thenApply(s -> s + " processed");
  ```
• **`thenAccept()`**：消费结果（无返回值）。
  ```java
  future.thenAccept(System.out::println);
  ```
• **`thenRun()`**：任务完成后执行操作（不访问结果）。
  ```java
  future.thenRun(() -> System.out.println("Done"));
  ```

##### **4.2 异常处理**
• **`exceptionally()`**：捕获异常并返回默认值。
  ```java
  future.exceptionally(ex -> "Fallback");
  ```
• **`handle()`**：统一处理结果和异常。
  ```java
  future.handle((res, ex) -> ex != null ? "Error" : res);
  ```

##### **4.3 组合任务**
• **`thenCompose()`**：链式依赖（前一个任务的结果作为下一个任务的输入）。
  ```java
  future.thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " chained"));
  ```
• **`thenCombine()`**：并行执行两个任务并合并结果。
  ```java
  future1.thenCombine(future2, (a, b) -> a + b);
  ```

---

### **5. 并行执行与结果聚合**
• **`allOf()`**：等待所有任务完成。
  ```java
  CompletableFuture.allOf(future1, future2).join();
  ```
• **`anyOf()`**：任意一个任务完成即返回。
  ```java
  CompletableFuture.anyOf(future1, future2).thenAccept(System.out::println);
  ```

---

### **6. 最佳实践**
1. **自定义线程池**：避免全局线程池资源竞争。
2. **避免阻塞 `get()`**：优先使用 `join()` 或带超时的 `get()`。
   ```java
   future.get(5, TimeUnit.SECONDS); // 超时处理
   ```
3. **合理编排任务**：
   • 无依赖任务使用 `thenCombine` 或 `allOf` 并行执行。
   • 有依赖任务使用 `thenCompose` 链式调用。
4. **异常传播**：通过 `whenComplete` 或 `handle` 确保异常被正确处理。

---

### **7. 实际案例**
##### **订单信息聚合**
```java
// 1. 获取用户信息（异步）
CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(this::fetchUser);

// 2. 并行获取商品和物流信息
CompletableFuture<Product> productFuture = userFuture.thenCompose(user -> 
    CompletableFuture.supplyAsync(() -> fetchProduct(user)));
CompletableFuture<Logistics> logisticsFuture = userFuture.thenCompose(user -> 
    CompletableFuture.supplyAsync(() -> fetchLogistics(user)));

// 3. 合并商品和物流结果后获取推荐
CompletableFuture<Recommendation> recommendationFuture = productFuture
    .thenCombine(logisticsFuture, (product, logistics) -> aggregateData(product, logistics))
    .thenCompose(data -> CompletableFuture.supplyAsync(() -> fetchRecommendation(data)));

// 4. 所有结果汇总
return recommendationFuture.thenApply(recommendation -> buildResponse(recommendation));
```

---

### **8. 参考资料**
1. [CompletableFuture 原理与实践 - 美团技术团队](https://tech.meituan.com/2022/05/12/principles-and-practices-of-completablefuture.html)
2. [RocketMQ 中的 CompletableFuture 应用](https://juejin.cn/post/6844904195703502862)
3. [京东 asyncTool 框架](https://github.com/jdpxiaoming/asyncTool)
