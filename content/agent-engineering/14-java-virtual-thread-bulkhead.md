---
title: "Java 21 虚拟线程接入大模型：并发隔离与容量边界"
date: 2026-06-17T10:00:00+08:00
lastmod: 2026-09-12T10:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/vt.png
images:
  - /img/vt.png
categories:
  - 架构设计
tags:
  - Java
slug: java-virtual-thread-bulkhead
weight: 1
description: "把线程调度能力与下游资源配额分开治理，避免更轻量的线程放大依赖压力。"
---

把线程调度能力与下游资源配额分开治理，避免更轻量的线程放大依赖压力。

<!--more-->

> 本文于 2026-09-12 补充整理，按 2026-06 专题归档。示例为方案设计，参数用于说明方法，不代表已上线项目或实测成果。

## 轻量线程不会增加供应商配额

模型调用往往等待网络，传统线程池可能在 CPU 不高时先耗尽线程。虚拟线程可以改善阻塞式代码的并发承载方式，但数据库连接数、HTTP 连接数、供应商请求配额和 token 配额都不会因此增加。

[Java 21 的 JEP 444](https://openjdk.org/jeps/444)明确建议为任务创建虚拟线程，并使用专门的并发控制机制约束稀缺资源。本文采用 Java 21 作为示例基线，不将较新 JDK 的行为混写到 Java 21 中。

## 把执行线程与下游许可分开

下面是隔离入口的 Java 21 类片段，假定 HTTP 客户端本身也配置了连接和请求超时：

~~~java
import java.util.concurrent.Callable;
import java.util.concurrent.RejectedExecutionException;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

final class ModelBulkhead {
    private final Semaphore permits = new Semaphore(20);

    <T> T call(Callable<T> action) throws Exception {
        if (!permits.tryAcquire(100, TimeUnit.MILLISECONDS)) {
            throw new RejectedExecutionException("model capacity exhausted");
        }
        try {
            return action.call();
        } finally {
            permits.release();
        }
    }
}
~~~

20 个许可和 100 毫秒等待仅是说明配置。方法应在应用管理的执行器中调用；信号量按应用实例共享，不能每个请求创建一个。多实例时，本地 20 个许可并不等于全局上限 20，还需要全局配额分配或网关限流。

这段代码控制同时进行的调用数，不控制每分钟请求数或 token 数，也不负责中断失控的网络调用。

## 超时应覆盖整个等待链

用户的 15 秒预算，可能消耗在许可等待、连接池等待、连接建立、模型生成和工具调用中。每层单独设置 15 秒，会让总时长远超用户预期。

应用应维护截止时间，调用下游前计算剩余预算，并设置实际客户端超时。`Future` 超时返回不代表网络请求已停止，也不保证对方没有产生费用；取消需要沿客户端能力传播，并记录可能仍在运行的请求。

入口还要限制待处理任务数量，避免大量虚拟线程同时等待许可并保留大段请求上下文。

## 迁移时留意版本差异

Java 21 下，需要观察某些阻塞操作与 synchronized 区域结合时的 pinning 情况。[JEP 491](https://openjdk.org/jeps/491)在 JDK 24 处理了 synchronized 相关的主要固定载体问题，但不能据此推断 Java 21 已有同样行为，也不能推断所有阻塞路径都没有限制。

ThreadLocal 中不宜放大型缓存。虚拟线程数量增加后，每线程附带状态也会放大内存占用。数据库事务、MDC 和安全上下文的传播需要分别验证，不能假设换执行器后自动全部正确。

## 怎样比较三种并发方案

在同一模拟下游和相同并发上限下，比较平台线程池、虚拟线程和现有响应式实现。记录任务成功率、排队时间、堆内存、连接池等待和 p95/p99，而不是只比较启动了多少线程。

模拟下游分别设置正常延迟、长尾延迟和限流响应。若虚拟线程方案在不限流时吞吐较高，但一遇到下游限流就持续堆积，说明治理尚未完成。

对已有成熟 WebFlux 链路，迁移不是必选项。选择应由团队的调试经验、依赖库阻塞特性和实际瓶颈决定。虚拟线程的价值是让合适场景下的并发代码更直接，不是消除资源容量边界。

---

[查看系列导读与阅读路线]({{< relref "/agent-engineering/overview.md" >}})
