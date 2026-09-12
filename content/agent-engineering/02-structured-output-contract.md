---
title: "Java 大模型应用的输出契约：从 JSON 到可执行业务命令"
date: 2026-03-18T10:00:00+08:00
lastmod: 2026-09-12T10:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/java.png
images:
  - /img/java.png
categories:
  - AI
tags:
  - Java
  - 大模型应用
slug: structured-output-contract
weight: 1
description: "用结构、语义和权限三层校验，阻止一段格式正确的模型输出直接变成错误业务操作。"
---

用结构、语义和权限三层校验，阻止一段格式正确的模型输出直接变成错误业务操作。

<!--more-->

> 本文于 2026-09-12 补充整理，按 2026-03 专题归档。示例为方案设计，参数用于说明方法，不代表已上线项目或实测成果。

## 格式正确只是第一层条件

在工单助手中，模型可能输出一个字段齐全的 JSON，但把测试环境识别成生产环境，或者填入当前用户无权访问的项目。反序列化成功只能说明程序读懂了输出，不能说明可以执行业务操作。

[Spring AI 1.0 系列的结构化输出文档](https://docs.spring.io/spring-ai/reference/1.0/api/structured-output-converter.html)说明了对象转换方式以及尽力生成格式的限制。本文采用独立于具体模型供应商的业务契约；接入时仍需固定实际使用的依赖版本和模型能力。

## 将建议对象和命令对象分开

下面是 Java 21 的类型设计片段，不是完整应用。`TicketSuggestion` 允许信息缺失；`CreateTicketCommand` 只能由服务端在校验后构造。

~~~java
record TicketSuggestion(
    String projectCode,
    String environment,
    String summary,
    java.util.List<String> evidenceIds
) {}

record CreateTicketCommand(
    String tenantId,
    String actorId,
    String projectId,
    String environment,
    String summary,
    String approvalId,
    String idempotencyKey
) {}
~~~

不要让模型直接生成后者。租户和操作人来自认证会话，项目 ID 来自受权限约束的查询，审批编号由审批服务生成，幂等键绑定具体业务操作。

如果用户没有提供环境，建议对象应保留缺失状态。给字段设置“production”之类的默认值，会将信息不足伪装成有效输入。

## 建立三层校验

| 层次 | 检查内容 | 失败处理 |
| --- | --- | --- |
| 结构 | 必需字段、类型、长度、枚举 | 记录解析失败，必要时有限修复 |
| 语义 | 项目存在、环境有效、证据关联当前任务 | 要求澄清或重新检索 |
| 权限与状态 | 用户权限、审批有效、工单是否已创建 | 拒绝执行或返回已有结果 |

结构化输出、JSON Schema 和 Java Bean Validation 可以互相补充，但并不互相替代。反序列化为 record 不会自动完成所有业务校验；Bean Validation 注解也需要实际调用校验器或经过相应框架入口。

证据 ID 不仅要存在，还要属于本次任务允许使用的证据集。否则模型可以引用另一个请求里真实存在、但当前用户无权看到的文档。

## 修复输出也需要预算

格式错误时，可以把精简后的校验错误反馈给模型，并最多修复一次。反馈应说明字段问题，不应包含数据库堆栈或内部凭据。语义错误不能简单重试同一提示词，例如项目不存在时应重新查询候选项目。

一次修复后仍失败，返回结构化失败结果，保存原输出的受控引用用于排查。日志默认记录错误类型和契约版本，原文需要脱敏和访问控制。

不要把大段失败输出不断塞回上下文。这样既扩大成本，也可能让来自外部内容的指令进入下一轮执行。

## 契约版本如何演进

给业务建议标注 `schemaVersion`。新增可选字段可以在兼容窗口中逐步启用；改变枚举含义、重命名关键字段或修改单位时，应升级版本并验证旧任务恢复。历史任务里保存的是当时实际使用的契约，不能在恢复时悄悄按新规则解释。

建议准备四个反例：合法 JSON 中的未知环境、超长摘要、其他租户证据 ID、重复审批编号。验收重点是这些输入都不能直接创建工单，并且调用者能够得到可处理的错误分类。

这篇设计要表达的能力是：把概率性建议转成确定性业务命令。模型输出越容易被下游系统执行，契约边界就越需要清晰。

---

[查看系列导读与阅读路线]({{< relref "/agent-engineering/overview.md" >}})
