---
title: 最重要的JVM参数总结
date: 2024-09-01T14:21:26+08:00
lastmod: 2024-09-01T14:21:26+08:00
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

## 最重要的JVM参数总结

### 堆内存配置
Java虚拟机所管理的内存中最大的一块，Java堆是所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。

与性能有关的最常见实践之一是根据应用程序要求初始化堆内存。推荐显示指定最小和最大堆大小：

#### 参数说明
- `-Xms<heap size>[unit]`  
  初始堆内存大小，单位：g/m/k  
  示例：`-Xms2G`

- `-Xmx<heap size>[unit]`  
  最大堆内存大小，单位：g/m/k  
  示例：`-Xmx5G`

组合示例：`-Xms2G -Xmx5G`

### 新生代内存配置
#### 配置方法
1. **独立指定**  
   - `-XX:NewSize=<young size>[unit]`  
     新生代初始大小  
   - `-XX:MaxNewSize=<young size>[unit]`  
     新生代最大大小  
     示例：`-XX:NewSize=256m -XX:MaxNewSize=1024m`

2. **统一指定**  
   - `-Xmn<young size>[unit]`  
     设置固定新生代大小  
     示例：`-Xmn256m`

#### 调优策略
- 通过`-XX:NewRatio=<int>`设置老年代与新生代内存比值  
  示例：`-XX:NewRatio=1`（老年代:新生代=1:1）

### 元空间配置
#### JDK 1.8+参数
- `-XX:MetaspaceSize=N`  
  触发Full GC的初始阈值（默认约20.8MB）
  
- `-XX:MaxMetaspaceSize=N`  
  元空间最大容量

#### 注意事项
- 实际初始容量由JVM根据平台决定（12-20MB）
- 达到`MetaspaceSize`值后会触发FGC并动态扩容
- 未设置`MaxMetaspaceSize`时可能耗尽系统内存

### 垃圾收集器配置
#### 收集器类型
- `-XX:+UseSerialGC`  
  串行垃圾收集器
  
- `-XX:+UseParallelGC`  
  并行垃圾收集器
  
- `-XX:+UseConcMarkSweepGC`  
  CMS收集器
  
- `-XX:+UseG1GC`  
  G1收集器

### GC日志配置
#### 基础配置
```shell
-XX:+PrintGCDetails 
-XX:+PrintGCDateStamps
-XX:+PrintTenuringDistribution
-XX:+PrintHeapAtGC
-XX:+PrintReferenceGC
-XX:+PrintGCApplicationStoppedTime
```

#### 高级配置
```shell
-Xloggc:/path/to/gc-%t.log
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=14
-XX:GCLogFileSize=50M
-XX:+PrintSafepointStatistics
-XX:PrintSafepointStatisticsCount=1
```

### 内存溢出处理
#### 关键参数
- `-XX:+HeapDumpOnOutOfMemoryError`  
  发生OOM时生成堆转储
  
- `-XX:HeapDumpPath=./java_pid<pid>.hprof`  
  堆转储文件路径
  
- `-XX:OnOutOfMemoryError="<command>"`  
  OOM时执行命令（如`shutdown -r`）
  
- `-XX:+UseGCOverheadLimit`  
  启用GC时间占比限制策略

### 其他重要参数
#### 性能优化
- `-server`  
  启用服务端模式（64位JVM默认）
  
- `-XX:+UseStringDeduplication`  
  字符串去重优化（Java 8u20+）

#### 内存管理
- `-XX:LargePageSizeInBytes=N`  
  大页内存大小
  
- `-XX:SurvivorRatio=N`  
  Eden与Survivor区比例（默认8）

#### 调试参数
- `-XX:+UseCompressedStrings`  
  启用字符串压缩存储
  
- `-XX:+OptimizeStringConcat`  
  优化字符串拼接操作
