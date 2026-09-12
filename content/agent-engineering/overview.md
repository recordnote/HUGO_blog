---
title: "Java Agent 工程化：从知识库问答到可靠业务执行"
date: 2026-09-12T10:00:00+08:00
lastmod: 2026-09-12T10:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/agent.png
images:
  - /img/agent.png
categories:
  - AI
  - 架构设计
tags:
  - Java
  - Agent
  - 大模型应用
slug: overview
weight: 1
description: "以企业知识库与工单助手为例，梳理 RAG、工具治理、可靠执行、评测和 Java 平台架构。"
---

这组文章围绕一个企业知识库与工单助手展开：查到可访问的证据、解释问题、生成提案，经确认后执行，并在故障后核实业务结果。重点是把大模型能力接入可靠的 Java 系统。

<!--more-->

## 适合怎样阅读

如果关注 Agent 开发，先看输出契约、工具设计、状态机和 SSE；如果关注 Agent 架构，先看平台蓝图、ADR、模型网关和发布流程；如果关注大模型应用，沿着四月的 RAG 专题阅读；如果关注 Java 架构，从事务消息、幂等、虚拟线程与容量规划进入。

文章中的系统为贯穿系列的设计案例，尚不代表本仓库包含对应业务实现。参数、容量算例和故障场景用于说明方法；实际效果需要通过文中实验验证。示例的官方资料链接放在相关论述附近，框架能力以接入时锁定版本为准。

## 归档说明

本系列在 2026-09-12 集中补充整理，按 3—9 月的专题节奏归档；文章日期用于归档排序，更新时间记录实际整理日期。3 月 4 日已有的[上下文工程文章]({{< relref "/ai/Java Agent Framework 进阶：Context Engineering 上下文工程与生产级治理.md" >}})作为衔接。新增 25 篇截至当前归档的正文，另准备 2 篇九月下半月草稿，本导读不计入每月四篇。

## 3 月：从模型调用转向业务工程

补齐输出契约、任务验收与架构边界；承接已有上下文工程文章。

- 03-04 · 已有：[Context Engineering 上下文工程与生产级治理]({{< relref "/ai/Java Agent Framework 进阶：Context Engineering 上下文工程与生产级治理.md" >}})
- 03-11 · [从业务流程到 Agent：企业知识库与工单助手的架构边界]({{< relref "/agent-engineering/01-workflow-or-agent.md" >}})
- 03-18 · [Java 大模型应用的输出契约：从 JSON 到可执行业务命令]({{< relref "/agent-engineering/02-structured-output-contract.md" >}})
- 03-25 · [Agent 评测基线：把演示效果变成可重复的验收条件]({{< relref "/agent-engineering/03-agent-evaluation-baseline.md" >}})

## 4 月：RAG 质量与数据治理

建立可更新、可隔离、可评测的知识链路。

- 04-01 · [RAG 文档入库设计：切分、版本与增量更新]({{< relref "/agent-engineering/04-rag-ingestion-versioning.md" >}})
- 04-08 · [混合检索调优：用失败样本决定召回与重排策略]({{< relref "/agent-engineering/05-rag-hybrid-retrieval.md" >}})
- 04-15 · [企业 RAG 的权限与引用：检索结果如何成为可信证据]({{< relref "/agent-engineering/06-rag-permission-citations.md" >}})
- 04-22 · [PostgreSQL 与 pgvector：知识库索引选型和容量估算]({{< relref "/agent-engineering/07-rag-postgres-pgvector.md" >}})

## 5 月：工具与权限治理

把协议连通推进到可授权、可审批、可核对的业务动作。

- 05-06 · [Agent 工具契约设计：参数、错误与业务边界]({{< relref "/agent-engineering/08-tool-contract-design.md" >}})
- 05-13 · [MCP 接入企业系统：身份传播与最小权限设计]({{< relref "/agent-engineering/09-mcp-identity-authorization.md" >}})
- 05-20 · [Agent 提示词注入防护：把外部内容留在数据边界内]({{< relref "/agent-engineering/10-prompt-injection-defense.md" >}})
- 05-27 · [人工审批与幂等执行：让 Agent 安全创建工单]({{< relref "/agent-engineering/11-approval-idempotency.md" >}})

## 6 月：Java 可靠执行

展示状态恢复、事务消息、并发隔离和流式接口能力。

- 06-03 · [长任务 Agent 状态机：检查点、租约与故障恢复]({{< relref "/agent-engineering/12-durable-agent-state.md" >}})
- 06-10 · [Agent 异步任务的事务消息：Outbox 与幂等消费]({{< relref "/agent-engineering/13-transactional-outbox-agent.md" >}})
- 06-17 · [Java 21 虚拟线程接入大模型：并发隔离与容量边界]({{< relref "/agent-engineering/14-java-virtual-thread-bulkhead.md" >}})
- 06-24 · [Java 流式 AI 接口：SSE、取消传播与慢客户端治理]({{< relref "/agent-engineering/15-sse-streaming-cancellation.md" >}})

## 7 月：平台运维与容量

讨论模型路由、观测、成本和压测，形成工程闭环。

- 07-01 · [企业模型网关设计：能力路由、配额与降级]({{< relref "/agent-engineering/16-model-gateway-routing.md" >}})
- 07-08 · [Agent 可观测性设计：从一次请求追踪到业务终态]({{< relref "/agent-engineering/17-agent-observability.md" >}})
- 07-15 · [大模型应用成本治理：Token 预算、缓存与质量约束]({{< relref "/agent-engineering/18-llm-cost-cache-budget.md" >}})
- 07-22 · [AI 服务容量规划：长尾延迟、排队与压测模型]({{< relref "/agent-engineering/19-ai-capacity-load-test.md" >}})

## 8 月：复杂 Agent 与质量保证

克制地引入协作和记忆，用发布评测与演练验证边界。

- 08-05 · [多 Agent 协作的工程取舍：何时拆分，如何收敛]({{< relref "/agent-engineering/20-multi-agent-coordination.md" >}})
- 08-12 · [Agent 长期记忆治理：写入条件、过期与删除]({{< relref "/agent-engineering/21-agent-memory-governance.md" >}})
- 08-19 · [Agent 版本发布：离线评测、影子流量与灰度回滚]({{< relref "/agent-engineering/22-agent-release-evaluation.md" >}})
- 08-26 · [Agent 故障演练设计：重复执行、越权检索与预算失控]({{< relref "/agent-engineering/23-agent-failure-drills.md" >}})

## 9 月：架构表达与作品集

汇总平台蓝图和 ADR，准备证据链与渐进迁移文章。

- 09-02 · [Java 企业 Agent 平台蓝图：从模块边界到部署拓扑]({{< relref "/agent-engineering/24-java-agent-platform-blueprint.md" >}})
- 09-09 · [Agent 架构决策记录：用 ADR 解释五个关键取舍]({{< relref "/agent-engineering/25-agent-architecture-decisions.md" >}})
- 09-16 · 计划草稿：Agent 项目展示设计：把架构文章连接到可验证证据
- 09-23 · 计划草稿：存量 Java 系统接入 Agent：渐进迁移与演进路线

## 可以继续验证的三个问题

1. 同一工单创建请求遇到超时和重启后，是否仍能获得唯一且可信的业务结果？
2. 文档更新或权限撤销后，检索、缓存和最终回答是否共同遵守新状态？
3. 一次架构变化是否在固定样本和相同预算下改善了业务成功率？

后续可以为这三个问题补上独立示例仓库、原始评测结果与故障演练记录，再把链接接入对应文章。

