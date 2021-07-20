---
title: JVM 虚拟机
date: 2021-04-01T14:21:26+08:00
lastmod: 2021-04-01T14:21:26+08:00
author: Aaron
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

## 1、JDK、JRE、JVM之间的区别？

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718104756951.png" alt="image-20210718104756951" style="zoom: 67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718105138378.png" alt="image-20210718105138378" style="zoom:67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718110114455.png" alt="image-20210718110114455" style="zoom: 67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718110128559.png" alt="image-20210718110128559" style="zoom: 67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718110358517.png" alt="image-20210718110358517" style="zoom: 67%;" /> 

## 2、JVM类加载过程

![image-20210718111144533](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718111144533.png)

## 3、 **JDK1.6、JDK1.7、JDK1.8 内存模型演变** 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718112557280.png" alt="image-20210718112557280" style="zoom:80%;" /> 

这些版本的 JVM 内存模型主要有以下差异：

- JDK 1.6：有永久代，<font color='red'>**静态变量**</font>存放在永久代上
- JDK 1.7：有永久代，但已经把字符串常量池、静态变量，存放在堆上。逐渐的减少永久代的使用

- JDK 1.8：无永久代，运行时常量池、类常量池，都保存在元数据区，也就是常说的**<font color = "red">元空间</font>**。但字符串常量池仍然存放在堆上。

## 4、程序计数器、Java虚拟机栈、本地方法栈、堆和元空间、常量池

### 4.1 程序计数器

- 较小的内存空间、线程私有，记录当前线程所执行的字节码行号

- 如果执行 Java 方法，计数器记录虚拟机字节码当前指令的地址，本地方法则为空

- 这一块区域没有任何 OutOfMemoryError 定义。

### 4.2  Java虚拟机栈

- 每一个方法在执行的同时，都会创建出一个栈帧，用于存放局部变量表、操作数栈、动态链接、方法出口、线程等信息。
- 方法从调用到执行完成，都对应着栈帧从虚拟机中入栈和出栈的过程。
- 最终，栈帧会随着方法的创建到结束而销毁。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718125242359.png" alt="image-20210718125242359" style="zoom:67%;" /> 

### 4.3  本地方法栈

- 本地方法栈与 Java 虚拟机栈作用类似，唯一不同的就是本地方法栈执行的是Native 方法，而虚拟机栈是为 JVM 执行 Java 方法服务的。

- 另外，与 Java 虚拟机栈一样，本地方法栈也会抛出 StackOverflowError 和OutOfMemoryError 异常。

- JDK1.8 HotSpot 虚拟机直接就把本地方法栈和虚拟机栈合二为一。

### 4.4 堆和元空间

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718125506457.png" alt="image-20210718125506457" style="zoom:80%;" /> 

- JDK 1.8 JVM 的内存结构主要由三大块组成：堆内存、元空间和栈，Java 堆是内存空间占据最大的一块区域。

- Java 堆，由年轻代和年老代组成，分别占据 1/3 和 2/3。 

- 而年轻代又分为三部分，**Eden**、**From Survivor**、**To Survivor**，占据比例为 8:1:1，可调。

- 另外这里我们特意画出了元空间，也就是直接内存区域。在 JDK 1.8 之后就不在堆上分配方法区了。

- **元空间**从虚拟机 Java 堆中转移到本地内存，默认情况下，元空间的大小仅受本地内存的限制，说白了也就是以后不会因为永久代空间不够而抛出 OOM 异常出现了。jdk1.8 以前版本的 class 和 JAR 包数据存储在 PermGen 下面 ，PermGen 大小是固定的，而且项目之间无法共用，公有的 class，所以比较容易出现 OOM 异常。

### 4.5 常量池

从 JDK 1.7 开始把常量池从永久代中剥离，直到 JDK1.8 去掉了永久代。而<font color='red'>**字符串常量池一直放在堆空间**</font>，用于存储字符串对象，或是字符串对象的引用。

## 5、JVM故障处理工具

### 5.1 **jps 虚拟机进程状况**

jps（JVM Process Status Tool），它的功能与 ps 命令类似，可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main()函数所在的类）名 称 以 及 这 些 <font color='red'>**进 程 的 本 地 虚 拟 机 唯 一 ID** </font>（ Local Virtual Machine Identifier,LVMID），类似于 ps -ef | grep java 的功能。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718131529847.png" alt="image-20210718131529847" style="zoom:80%;" /> 

### 5.2 **jcmd 虚拟机诊断命令**

jcmd，是从 jdk1.7 开始新发布的 JVM 相关信息诊断工具，可以用它来导出堆和线程信息、查看 Java 进程、执行 GC、还可以进行采样分析（jmc 工具的飞行记录器）。注意其使用条件是只能在被诊断的 JVM 同台 sever 上，并且具有相同的用户和组(user and group).

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718131719650.png" alt="image-20210718131719650" style="zoom: 67%;" /> 

- **jcmd pid VM.flags，查看 JVM 启动参数** 
- **jcmd pid VM.uptime，查看 JVM 运行时长** 
- **jcmd pid PerfCounter.print，查看 JVM 性能相关参数** 、
- **jcmd pid GC.class_histogram，查看系统中类的统计信息** 
- **jcmd pid Thread.print，查看线程堆栈信息** 
- **jcmd pid VM.system_properties，查看 JVM 系统参数** 
- **jcmd pid GC.heap_dump 路径，导出 heap dump 文件**
- **jcmd pid help，列出可执行操作**
- **jcmd pid help JFR.stop，查看命令使用** 

### 5.3 **jinfo Java 配置信息工具** 

jinfo（Configuration Info for Java），实时查看和调整 JVM 的各项参数。在上面讲到 jps -v 指令时，可以看到它把虚拟机启动时显式的参数列表都打印出来了，但如果想更加清晰的看具体的一个参数或者想知道未被显式指定的参数时，就可以通过 jinfo -flag 来查询了。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132029912.png" alt="image-20210718132029912" style="zoom: 67%;" /> 

### 5.4  **jstat 收集虚拟机运行数据** 

jstat（JVM Statistics Monitoring Tool），用于监视虚拟机各种运行状态信息。它可以查看本地或者远程虚拟机进程中，类加载、内存、垃圾收集、即时编译等运行时数据。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132305683.png" alt="image-20210718132305683" style="zoom:67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132324814.png" alt="image-20210718132324814" style="zoom:67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132339965.png" alt="image-20210718132339965" style="zoom:80%;" /> 

### 5.5 **jmap 内存映射工具** 

jmap（Memory Map for Java），用于生成堆转储快照（heapdump 文件）。jmap 的作用除了获取堆转储快照，还可以查询 finalize 执行队列、Java 堆和方法区的详细信息。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132459051.png" alt="image-20210718132459051" style="zoom:67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132508386.png" alt="image-20210718132508386" style="zoom:67%;" /> 

### 5.6 **jhat 堆转储快照分析工具** 

jhat（JVM Heap Analysis Tool），与 jmap 配合使用，用于分析 jmap 生成的堆转储快照。

jhat 内置了一个小型的 http/web 服务器，可以把堆转储快照分析的结果，展示在浏览器中查看。不过用途不大，基本大家都会使用其他第三方工具。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132630003.png" alt="image-20210718132630003" style="zoom: 80%;" /> 

### 5.7 **jstack Java 堆栈跟踪工具** 

jstack（Stack Trace for Java），用于生成虚拟机当前时刻的线程快照（线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的目的通常是定位线程出现长时间停顿的原因，如：线程死锁、死循环、请求外部资源耗时较长导致挂起等。）

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132942568.png" alt="image-20210718132942568" style="zoom: 67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718132948336.png" alt="image-20210718132948336" style="zoom:80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718133021740.png" alt="image-20210718133021740" style="zoom:80%;" /> 

### 5.8 可视化处理工具

#### 5.8.1 **jconsole，Java 监视与管理控制台** 

JConsole（ Java Monitoring and Management Console），是一款基于 JMX（ Java Manage-ment Extensions） 的可视化监视管理工具。它的功能主要是对系统进行收集和参数调整，不仅可以在虚拟机本身管理还可以开发在软件上，是开放的服务，有相应的代码 API 调用。

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718133348156.png" alt="image-20210718133348156" style="zoom:80%;" />

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718133411259.png" alt="image-20210718133411259" style="zoom:67%;" /> 

#### 5.8.2 **VisualVM，多合故障处理工具** 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718133447437.png" alt="image-20210718133447437" style="zoom:80%;" /> 

## 6、GC垃圾回收

#### 6.1 判断对象是否死亡

##### 6.1.1 引用计数器

1.  为每一个对象添加一个引用计数器，统计指向该对象的引用次数。

2.  当一个对象有相应的引用更新操作时，则对目标对象的引用计数器进行增减。

3.  一旦当某个对象的引用计数器为 0 时，则表示此对象已经死亡，可以被垃圾回收。

ps：<font color='red'>**在主流的 Java 虚拟机中并没有选用引用计数算法来管理内存**</font>，主要是因为这个简单的计数方式在处理一些相互依赖、循环引用等就会非常复杂。可能会存在不再使用但又不能回收的内存，造成内存泄漏

##### 6.1.2 可达性分析法

通过定义一系列称为 <font color='red'>**GC Roots 根对象**</font>作为起始节点集，从这些节点出发，穷举该集合引用到的全部对象填充到该集合中（live set）。这个过程叫做标记，只标记那些存活的对象，未被标记的对象就是可以被回收的对象了。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718135027465.png" alt="image-20210718135027465" style="zoom:80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718135111979.png" alt="image-20210718135111979" style="zoom:80%;" /> 

#### 6.2  **垃圾回收算法**

##### 6.2.1  **标记-清除算法**

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718141243775.png" alt="image-20210718141243775" style="zoom:80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718135648648.png" alt="image-20210718135648648" style="zoom:80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718135657305.png" alt="image-20210718135657305" style="zoom:80%;" /> 

##### 6.2.2  **标记-复制算法**

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718135918861.png" alt="image-20210718135918861" style="zoom:80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718135941706.png" alt="image-20210718135941706" style="zoom: 80%;" /> 

##### 6.2.3  **标记-压缩算法**	

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718140045610.png" alt="image-20210718140045610" style="zoom:67%;" /> 

   <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718140122609.png" alt="image-20210718140122609" style="zoom:80%;" />

####  6.3 垃圾回收器

##### 6.3.1 新生代

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718140412254.png" alt="image-20210718140412254" style="zoom: 80%;" /> 

##### 6.3.2 老年代

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718140746607.png" alt="image-20210718140746607" style="zoom:80%;" /> 

##### 6.3.3  G1

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718140802932.png" alt="image-20210718140802932" style="zoom: 80%;" /> 
