---
title: Java 11 新特性概览
date: 2024-06-11T14:21:26+08:00
lastmod: 2024-06-11T14:21:26+08:00
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

# Java 11 新特性概览

## 发布与支持
- **发布时间**：2018年9月25日  
- **长期支持（LTS）**：Oracle官方支持至2026年9月  
- **历史定位**：Java 8之后首个长期支持版本  

---

## 核心新特性

### JEP 321: HTTP Client 标准化
- **标准化API**：从`jdk.incubator.http`迁移至`java.net.http`  
- **异步非阻塞支持**：基于`CompletableFuture`实现  
- **示例代码**：
  ```java
  // 同步请求
  var request = HttpRequest.newBuilder()
      .uri(URI.create("https://javastack.cn"))
      .GET()
      .build();
  var client = HttpClient.newHttpClient();
  HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
  
  // 异步请求
  client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println);

### JEP 333: ZGC（可伸缩低延迟垃圾收集器）
- **设计目标**：
  - GC停顿时间 ≤ 10ms  
  - 支持从几百MB到TB级堆内存  
  - 吞吐下降不超过15%（对比G1）  
- **特点**：
  - 实验性功能，仅支持Linux/x64  
  - 基于标记-复制算法的改进  
  - 显著减少STW（Stop The World）事件  

### JEP 323: Lambda 参数的局部变量语法
- **语法增强**：允许在Lambda表达式中使用`var`声明参数  
- **示例**：
  ```java
  Consumer<String> consumer = (var i) -> System.out.println(i); // 等价于显式类型声明


### JEP 330: 启动单文件源代码程序
- **功能**：直接运行未编译的`.java`文件  
- **约束**：所有相关类需定义在同一文件中  
- **适用场景**：快速测试脚本、教学场景，与`jshell`互补  


## 语言与API增强

### 字符串处理
- **新增方法**：
  - `isBlank()`：判断空或仅含空白字符  
  - `strip()` / `stripLeading()` / `stripTrailing()`：去除首尾/首部/尾部空格  
  - `repeat(int n)`：重复字符串`n`次  
  - `lines()`：按行终止符分割字符串，返回流  
- **示例**：
  ```java
  "  ".isBlank();                // true
  " Java ".strip();              // "Java"
  "Java".repeat(3);              // "JavaJavaJava"
  "A\nB\nC".lines().count();     // 3


### Optional 增强
- **方法**：`isEmpty()`判断Optional是否为空  
- **示例**：
  ```java
  Optional<String> op = Optional.empty();
  op.isEmpty();  // true


## 其他重要特性

### 新垃圾回收器
- **Epsilon GC**：消极GC实现，仅管理内存分配，无回收机制，适用于短生命周期任务和性能测试  

### 低开销堆分析
- **Heap Profiling**：通过JVMTI提供低开销的堆分配采样，支持获取对象分配信息  

### 安全增强
- **TLS 1.3**：默认支持RFC 8446协议，提升安全性与性能  
  - 兼容性改进：OCSP装订扩展、会话散列等  

### 工具与监控
- **Flight Recorder开源化**：原商业版功能转为免费，支持生产环境诊断  
  - 功能：低开销的性能事件收集与分析  

---

## 参考
- **官方Release Notes**：[Oracle JDK 11 Release Notes](https://www.oracle.com/java/technologies/javase/11-relnote-issues.html)

