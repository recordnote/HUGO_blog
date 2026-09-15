---
title: "企业 Text-to-SQL 设计：语义口径、查询约束与结果验证"
date: 2026-09-30T08:00:00+08:00
publishDate: 2026-09-30T08:00:00+08:00
lastmod: 2026-09-15T08:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/sjk.png
images:
  - /img/sjk.png
categories:
  - AI
tags:
  - AI应用
  - 数据库
slug: text-to-sql-governance
weight: 1
description: "从业务语义层到只读执行网关，设计能够解释口径、限制成本并验证结果的自然语言查数链路。"
---

从业务语义层到只读执行网关，设计能够解释口径、限制成本并验证结果的自然语言查数链路。

<!--more-->

> 本文于 2026-09-15 提前整理。场景、配置和代码用于讨论设计，未声称已经完成生产验证；技术资料以整理时查阅的版本为准。

## SQL 能执行，不代表问题被正确回答

员工问“本月生产故障的平均处理时长”，模型生成了一条可以运行的 SQL，结果仍可能完全偏离业务：本月按创建时间还是关闭时间筛选？处理时长是否扣除等待客户反馈？重新打开的工单算一次还是两次？

Text-to-SQL 的首要难点是把自然语言翻译成明确统计口径。数据库可以证明语法和部分约束正确，却无法判断用户说的“处理时长”应该采用哪个定义。

可以先把输入变成语义查询计划，再由受控的编译器生成 SQL。复杂临时分析确实需要模型生成候选 SQL 时，也应经过同一套执行检查。

## 先定义业务语义层

以工单系统为例，维护一份版本化指标目录，而不是把全部数据库 DDL 一股脑交给模型：

~~~yaml
metricId: ticket_resolution_hours
version: 2
description: 已关闭工单从创建到最终关闭的自然时长
grain: ticket
timeDimension: closed_at
allowedDimensions:
  - project
  - environment
nullPolicy: exclude_unclosed
timezone: Asia/Shanghai
~~~

这是自定义配置示例，不是某个框架的标准格式。指标负责人应确认定义，并用固定数据验证。模型看到名称、解释和允许维度即可，不必知道数据库中的所有敏感字段。

“故障”的业务分类和项目范围也要来自领域目录。没有唯一解释时，先返回澄清问题。用默认字段偷偷补足歧义，会得到最难被发现的错误：答案合理、数字准确，统计的却不是用户想要的对象。

## 将查询计划与执行权限分开

模型可以提出如下计划：

~~~json
{
  "metricId": "ticket_resolution_hours",
  "metricVersion": 2,
  "timeRange": {
    "start": "2026-09-01T00:00:00+08:00",
    "endExclusive": "2026-10-01T00:00:00+08:00"
  },
  "filters": {"environment": "production"},
  "groupBy": ["project"]
}
~~~

这里的时间范围只是合成示例。实际请求的“本月”要根据业务时区和请求时间解析；结束时间采用开区间，避免用月末 23:59:59 遗漏更高精度时间。

租户、操作者和允许项目由认证上下文提供，不属于模型能自由决定的身份字段。查询计划中的项目过滤只能缩小授权范围，不能扩大它。

对于稳定指标，编译器可以从白名单中选择预定义表达式，再绑定时间和筛选参数。标识符不能简单当作普通参数绑定，表名、字段名和排序方向应从允许集合中选择。

## 查询网关不靠关键词判断安全

把 SQL 转成语法树，限制语句数量、引用对象、函数和操作类型。只检查开头是不是 SELECT 不够：公共表表达式、函数调用、子查询和系统对象访问都需要纳入规则。

数据库账号应只有分析所需的读取权限，必要时仅允许访问受控视图；租户和行权限继续由数据库或权威策略保护。只读事务提供额外约束，但不是通用 SQL 沙箱，不能替代对象权限、函数白名单或外部访问限制。[PostgreSQL 的事务模式说明](https://www.postgresql.org/docs/current/sql-set-transaction.html)明确了只读事务的约束范围。

在已创建合成分析表的实验环境中，执行器可以采用这样的事务包装：

~~~sql
BEGIN READ ONLY;
SET LOCAL statement_timeout = '2s';
SET LOCAL lock_timeout = '200ms';

SELECT project_id,
       AVG(EXTRACT(EPOCH FROM (closed_at - created_at)) / 3600.0)
           AS average_resolution_hours
FROM analytics_ticket_demo
WHERE tenant_id = 'tenant-demo'
  AND environment = 'production'
  AND closed_at >= TIMESTAMPTZ '2026-09-01 00:00:00+08'
  AND closed_at <  TIMESTAMPTZ '2026-10-01 00:00:00+08'
GROUP BY project_id
ORDER BY project_id
LIMIT 100;

ROLLBACK;
~~~

演示中的租户和时间使用字面量，应用应从可信上下文绑定参数；2 秒和 200 毫秒是实验值。超时后事务进入错误状态时，也要回滚并释放连接。[PostgreSQL 客户端配置文档](https://www.postgresql.org/docs/current/runtime-config-client.html)说明了语句与锁等待超时的不同作用。

LIMIT 只限制输出行数，不保证扫描、聚合和排序便宜。还需要限制时间范围、并发、返回字节数，必要时预先检查执行计划；真正执行的耗时和资源上限仍要在运行时生效。

## 回答必须包含数字的来源

返回给模型的结果应带上指标版本、授权范围、时间区间、单位、数据刷新时间和截断标记。生成器负责解释结果，不能把缺失数据补成零，也不能将“没有关闭工单”写成“没有发生故障”。

对于金额、时长和比率，尽可能由代码格式化最终数字；重要结论可将数值槽位与查询结果绑定，减少模型在复述时改写单位或小数点。

如果来自分析副本或离线表，说明数据截至何时。不能把延迟十分钟的副本结果说成实时状态。

## 如何验证语义与执行都正确

准备一组能揭露口径差异的数据：月底跨时区工单、未关闭工单、重新打开工单、其他租户同名项目和空结果。固定指标定义后，人工计算预期值，再对比完整查询链。

评测分为意图澄清、计划正确、执行授权和结果解释四层。两条 SQL 文字不同但结果与语义相同，不应因为字符串不一致而判失败；在当前数据上结果偶然相同的错误 SQL，也不能直接判正确。

这一方案可以衔接此前的[输出契约]({{< relref "/agent-engineering/02-structured-output-contract.md" >}})和[权限与引用设计]({{< relref "/agent-engineering/06-rag-permission-citations.md" >}})。最终需要证明的是数字可追溯、口径可解释、执行受约束，而不只是模型会写 SQL。
