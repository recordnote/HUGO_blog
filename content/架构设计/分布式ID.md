---
title: 分布式ID
date: 2023-02-04T14:21:26+08:00
lastmod: 2023-02-04T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/java.png
images:
  - /img/java.png
categories:
  - 架构设计
tags:
  - 架构设计
weight: 1
---

## 1.业务背景：

传统方案在复杂分布式系统中，往往需要对大量的数据和消息进行唯一标识。如在美团点评的金融、支付、餐饮、酒店、猫眼电影等产品。对数据分库分表后需要有一个唯一ID来标识一条数据或消息。

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144239032.png" alt="image-20230711144239032" style="zoom:50%;" />  

## 2.一个最基本的分布式 ID 需要满足下面这些要求：

- **全局唯一** ：ID 的全局唯一性肯定是首先要满足的！
- **高性能** ：分布式 ID 的生成速度要快，对本地资源消耗要小。
- **高可用** ：生成分布式 ID 的服务要保证可用性无限接近于 100%。
- **方便易用** ：拿来即用，使用方便，快速接入！

除了这些之外，一个比较好的分布式 ID 还应保证：

- **安全** ：ID 中不包含敏感信息。

- **有序递增** ：如果要把 ID 存放在数据库的话，ID 的有序性可以提升数据库写入速度。并且，很多时候 ，我们还很有可能会直接通过 ID 来进行排序。

- **有具体的业务含义** ：生成的 ID 如果能有具体的业务含义，可以让定位问题以及开发更透明化（通过 ID 就能确定是哪个业务）。

- **独立部署** ：也就是分布式系统单独有一个发号器服务，专门用来生成分布式 ID。这样就生成 ID 的服务可以和业务相关的服务解耦。不过，这样同样带来了网络调用消耗增加的问题。总的来说，如果需要用到分布式 ID 的场景比较多的话，独立部署的发号器服务还是很有必要的。

  ##### 数据库号段模式

  数据库主键自增这种模式，每次获取 ID 都要访问一次数据库，ID 需求比较大的时候，肯定是不行的。

  如果我们可以批量获取，然后存在在内存里面，需要用到的时候，直接从内存里面拿就舒服了！这也就是我们说的 **基于数据库的号段模式来生成分布式 ID。**

  数据库的号段模式也是目前比较主流的一种分布式 ID 生成方式。像+

  滴滴开源的**Tinyid**就是基于这种方式来做的。不过，TinyId 使用了双号段缓存、增加多 db 支持等方式来进一步优化。

## 3.方案

![image-20230711144321090](https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144321090.png)

### 自研算法：

时间戳（毫秒）+ 机器序列 + 自增数

###  号段模式：

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144414664.png" alt="image-20230711144414664" style="zoom:67%;" /> 

### 雪花算法：

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144445043.png" alt="image-20230711144445043" style="zoom:67%;" /> 

核心思想是： 使用41bit作为毫秒数，10bit作为机器的ID(5ge bit是数据中心，5个bit的机器ID),12bit 作为毫秒数内的流水号，最后一个符号位，永远是0.

符号位：0（表示正数）

工作进程位（类似服务器序号）：

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144527590.png" alt="image-20230711144527590" style="zoom:50%;" />    

常见的开源组件：

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144556996.png" alt="image-20230711144556996" style="zoom:67%;" /> 

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144641923.png" alt="image-20230711144641923" style="zoom:67%;" /> 

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20230711144710698.png" alt="image-20230711144710698" style="zoom:67%;" /> 

ps：雪花算法会跟机器标识做绑定，尤其是服务器IP，需要唯一