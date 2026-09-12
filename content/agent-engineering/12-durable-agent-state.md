---
title: "长任务 Agent 状态机：检查点、租约与故障恢复"
date: 2026-06-03T10:00:00+08:00
lastmod: 2026-09-12T10:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/agent.png
images:
  - /img/agent.png
categories:
  - 架构设计
tags:
  - Agent
  - 架构设计
slug: durable-agent-state
weight: 1
description: "将任务事实持久化，用租约和版本校验处理进程重启与重复调度。"
---

将任务事实持久化，用租约和版本校验处理进程重启与重复调度。

<!--more-->

> 本文于 2026-09-12 补充整理，按 2026-06 专题归档。示例为方案设计，参数用于说明方法，不代表已上线项目或实测成果。

## 一次长任务包含多个短事务

排障 Agent 可能检索文档、查询工单、等待用户确认，然后继续执行。把这些步骤放在一个内存循环里，进程重启后就失去上下文；把整个过程包在数据库事务里，又会长时间占用连接和锁。

合理的持久化单位是一次确定的状态推进。外部调用发生在短事务之外，结果通过另一笔短事务写入；中间的不确定性由操作账本和恢复流程处理。

## 状态描述业务事实

建议至少包含 `READY`、`RUNNING`、`WAITING_INPUT`、`WAITING_APPROVAL`、`RECONCILING`、`SUCCEEDED`、`FAILED` 和 `CANCELLED`。其中等待和对账都不是失败。

任务表可以保存当前节点、业务状态、乐观锁版本、租约持有者、租约过期时间和最近检查点。检查点引用证据与工具结果，不必每次复制完整对话；大量原始内容放到受控对象存储。

## 用版本条件推进状态

以下是 PostgreSQL 风格的更新片段，参数由应用绑定：

~~~sql
UPDATE agent_task
SET status = :next_status,
    version = version + 1,
    updated_at = CURRENT_TIMESTAMP
WHERE task_id = :task_id
  AND tenant_id = :tenant_id
  AND version = :expected_version
  AND lease_owner = :worker_id
  AND lease_until > CURRENT_TIMESTAMP;
~~~

受影响行数为 0 时，当前工作进程已经失去推进资格，应停止提交结果并重新读取状态。不能忽略冲突后继续使用旧上下文执行写操作。

这段代码只是状态写入的一部分，完整实现还需验证允许的状态转换与调用者权限。

## 租约不等于外部动作隔离

工作进程 A 暂停过久，租约过期，B 接管；随后 A 恢复。数据库条件可以阻止 A 写回任务状态，但无法自动撤回 A 已经发送的外部请求。

因此写工具仍需稳定幂等键。对支持 fencing token 的自有下游，可携带递增任期并拒绝旧任期；不支持时要通过操作账本、结果查询和幂等契约降低重复风险。

租约续期频率需要与最长正常暂停、数据库延迟和任务超时协调。不能靠无限延长租约避免竞争，那会让真实故障迟迟无法恢复。

## 检查点应保存什么

保存模型输入版本、已接受的工具动作、工具结果引用、预算余额、当前提案和停止原因。模型再次生成同一步并不一定产生同样输出，因此恢复通常应重用已保存的已接受结果，而不是重新采样过去步骤。

检查点不需要保存模型隐藏推理。业务恢复依赖的是显式动作、观察结果和状态，而不是某个供应商内部的思考过程。

恢复时重新验证过期权限与资源状态。历史证据可用于审计，却不一定仍有资格进入新的模型上下文。

## 演练三个崩溃窗口

第一个窗口是外部请求前崩溃，恢复后可以重新调度；第二个是读请求返回后、检查点前崩溃，通常可在预算内重新读取；第三个是写请求成功后、检查点前崩溃，必须查询结果或依赖幂等。

记录每个窗口的任务终态、执行次数和恢复耗时。验收目标不是“永远不重试”，而是恢复路径清楚，且重试不会破坏业务约束。

相关的数据库与事件衔接可参考 [AWS 对事务 Outbox 的说明](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)。任务状态机与租约是本例需要额外实现的编排能力。

---

[查看系列导读与阅读路线]({{< relref "/agent-engineering/overview.md" >}})
