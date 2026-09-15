---
title: "Spring 接入 Agent 的事务边界：长调用、代理与异步执行"
date: 2026-10-14T08:00:00+08:00
publishDate: 2026-10-14T08:00:00+08:00
lastmod: 2026-09-15T08:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/spring.jpg
images:
  - /img/spring.jpg
categories:
  - 架构设计
tags:
  - Java
  - 架构设计
slug: spring-agent-transaction-boundaries
weight: 1
description: "拆开模型等待与数据库事务，用短事务、状态版本和业务幂等处理跨线程、进程重启与外部调用。"
---

拆开模型等待与数据库事务，用短事务、状态版本和业务幂等处理跨线程、进程重启与外部调用。

<!--more-->

> 本文于 2026-09-15 提前整理。场景、配置和代码用于讨论设计，未声称已经完成生产验证；技术资料以整理时查阅的版本为准。

## 一个注解可能包住几十秒的等待

在工单助手中，很容易写出这样的服务方法：加上 `@Transactional`，读取任务、调用模型、创建工单，最后保存结果。代码看起来顺序清楚，却把远程等待纳入了本地事务的生命周期。

连接可能在执行首条 SQL 后持续被占用；如果提前锁定任务行，模型等待期间锁也可能一直存在。即使外部请求超时，本地事务回滚，也不能撤回已在工单系统完成的写入。

正确边界应围绕可以原子提交的数据库事实划分。模型生成与远程工具调用通常放在事务之外，失败窗口由任务状态和幂等协议处理。

## 分成认领、执行、提交三个阶段

~~~text
短事务一：校验状态 → 认领任务 → 保存版本和执行令牌
事务之外：检索 → 调用模型 → 受控工具执行
短事务二：校验版本和令牌 → 保存结果 → 写入后续事件
~~~

“事务之外”并不意味着不受控制。执行阶段仍要遵守截止时间、租约和操作权限；完成后也不能无条件覆盖数据库中的当前状态。

如果主任务已被取消或被另一个工作进程接管，旧执行者保存结果时应发生版本冲突。外部动作可能已经发生的情况，则需要进入结果核对，而不是通过一次失败的 UPDATE 假装它没有执行过。

## 使用独立 Bean 暴露短事务

下面是 Spring 应用的接口与服务骨架，省略存储实现、异常处理和依赖注入构造器，不能单独当作完整项目运行：

~~~java
public class AgentWorker {
    private final TaskTransactions transactions;
    private final ModelPlanner planner;

    public void execute(String taskId) {
        TaskClaim claim = transactions.claim(taskId);
        PlanResult result = planner.plan(claim);
        transactions.complete(claim, result);
    }
}

public class TaskTransactions {
    private final TaskRepository repository;

    @org.springframework.transaction.annotation.Transactional
    public TaskClaim claim(String taskId) {
        return repository.claimEligibleTask(taskId);
    }

    @org.springframework.transaction.annotation.Transactional
    public void complete(TaskClaim claim, PlanResult result) {
        repository.completeIfOwned(claim, result);
    }
}
~~~

这里假定两个类都已注册为 Spring Bean，Worker 经代理调用 TaskTransactions，且 execute 的调用方没有开启外层事务。`TaskClaim`、`PlanResult`、`TaskRepository` 和 `ModelPlanner` 是项目自定义类型。

存储层的 `completeIfOwned` 必须验证租约、版本和状态，并检查受影响行数。若还要发送下一步事件，可以在同一短事务里写 outbox，由投递器负责发送。

## 自调用和外层事务容易打破设计

在默认代理模式下，一个对象通过 `this.claim()` 调用自己的事务方法，不会经过相应代理拦截。把方法拆开但仍留在同一对象里自调用，可能只是视觉上分层。[Spring 的声明式事务文档](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)说明了这个边界。

默认 REQUIRED 传播还可能加入调用方的事务。若外层方法已经开启长事务，内部两个短方法并不会自动缩短它。可以限制编排入口不带事务，或通过明确的事务传播和程序式边界隔离，但应结合调用链验证。

不能把所有方法都改成 REQUIRES_NEW 作为通用修复。它可能挂起外层事务并额外占用连接；外层资源仍未释放，连接池压力甚至可能变大。

## 异步执行需要新的明确事务

命令式 Spring 事务通常绑定当前执行线程，不会自动传播到新建线程。将数据库操作放进 `CompletableFuture` 后，不能假设它还属于原事务。[Transactional 注解文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html)区分了线程绑定事务与响应式事务的上下文机制。

异步工作应在其实际执行入口建立需要的短事务。安全身份、租户和 trace 也要明确传递；不能把会话对象、数据库连接或 JPA EntityManager 直接交给另一个线程共享。

响应式事务采用自己的上下文传播规则，不能把这篇命令式示例原样套到 Reactor 流上。需要核对事务管理器、返回类型和操作是否处于同一响应式链路。

## 失败语义比回滚注解更重要

在 Spring 默认配置下，运行时异常和 Error 通常触发回滚，受检异常未必自动回滚；项目可以通过规则改变这个行为。应用若捕获异常后正常返回，也可能让事务提交，所以应把预期失败和异常路径写清楚。

但即使所有本地异常都能回滚，跨系统动作依然不具备本地原子性。工单已经创建、保存结果失败时，重试必须沿用原操作 ID，通过下游幂等或对账恢复。

可以衔接此前的[审批与幂等设计]({{< relref "/agent-engineering/11-approval-idempotency.md" >}})和[Outbox 设计]({{< relref "/agent-engineering/13-transactional-outbox-agent.md" >}})，分别解决允许执行、外部重复和事件可靠投递。

## 如何验证事务真的变短

将模拟模型延迟设置为 20 秒，在执行期间观察数据库活跃事务、连接池占用和任务行锁。改造后的编排不应因为等待模型而持续持有一笔已经开始的数据库事务。

再在认领后、外部写入后和结果提交前分别注入崩溃，检查恢复状态。用两个工作进程同时尝试完成同一任务，确认旧版本不能覆盖新结果。

报告应记录真实事务边界和资源指标。只检查代码上有几个 `@Transactional`，无法证明系统已经获得所需的并发和恢复行为。
