---
title: "Agent 工具契约设计：参数、错误与业务边界"
date: 2026-05-06T10:00:00+08:00
lastmod: 2026-09-12T10:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/mcp.png
images:
  - /img/mcp.png
categories:
  - AI
tags:
  - Agent
slug: tool-contract-design
weight: 1
description: "把粗粒度存量接口拆成语义明确、可校验且可审计的 Agent 工具。"
---

把粗粒度存量接口拆成语义明确、可校验且可审计的 Agent 工具。

<!--more-->

> 本文于 2026-09-12 补充整理，按 2026-05 专题归档。示例为方案设计，参数用于说明方法，不代表已上线项目或实测成果。

## 工具数量不等于工具能力

把几十个内部 API 全部交给模型，并不一定让助手更能干。接口名称含糊、参数意义重复、返回值过大，都会增加选择错误。工具设计需要回答三个问题：什么时候使用、输入代表什么、结果允许下一步做什么。

以工单系统为例，`executeOperation(action, payload)` 看似通用，却把权限和业务语义藏进字符串。拆成 `searchTickets`、`getTicket`、`prepareTicket`、`createApprovedTicket`，通常更容易分别约束读写能力。

## 参数应贴近任务语义

~~~json
{
  "name": "searchTickets",
  "description": "在当前用户可访问的项目内查询工单，不创建或修改工单",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {"type": "string", "maxLength": 200},
      "status": {"type": "string", "enum": ["OPEN", "CLOSED"]},
      "limit": {"type": "integer", "minimum": 1, "maximum": 20}
    },
    "required": ["query"],
    "additionalProperties": false
  }
}
~~~

这是工具描述示例，省略协议封装。租户和操作人不出现在模型可写参数中，由执行上下文注入。查询范围若由项目决定，服务端还要验证项目权限，不能仅相信模型选择了某个候选项目。

工具描述应说明分页、空结果和时间范围。涉及时间时明确时区；涉及金额时明确单位。避免一个接口返回“100”，调用者却不知道它是分、元还是百分比。

## 返回值要帮助下一步决策

工具返回可以包含 `status`、`items`、`nextCursor` 和 `truncated`。明确标记截断比悄悄裁掉尾部更好；否则模型可能把“只看到了前 20 条”误认为“系统只有 20 条”。

错误分类建议区分：

| 错误 | 模型或编排器的下一步 |
| --- | --- |
| INVALID_ARGUMENT | 根据字段错误修正参数 |
| NOT_FOUND | 说明未找到，必要时换查询条件 |
| FORBIDDEN | 停止该操作，不换身份继续尝试 |
| RATE_LIMITED | 在总预算内延后，遵守服务端建议 |
| OUTCOME_UNKNOWN | 查询操作结果，禁止盲目重放写入 |

这些是应用层错误语义，不代表 MCP 或某个 SDK 自动提供同名枚举。

## 读写边界要在执行器落实

工具描述中的“只读”是说明，最终限制应由只读接口、权限令牌和服务端逻辑落实。模型看到的工具集合可以按角色缩减，但执行时仍应鉴权，防止旧会话继续使用已经撤销的能力。

[MCP 规范](https://modelcontextprotocol.io/specification/2025-06-18)也将工具描述和注解放在需要评估信任的范围内。不能因为远端服务器给工具标了低风险，就自动赋予写权限。

对于创建工单，不应让模型重新生成已获审批的全部参数。让它引用服务端保存的提案编号，由执行器读取不可变提案，会更容易验证执行内容与审批内容一致。

## 如何衡量工具是否好用

准备“查状态”“查相似问题”“准备新工单”三类输入，比较粗粒度工具与拆分工具的选择正确率、参数修复次数和总调用数。评测中应加入相近名称、空结果和超长结果。

单次调用少不一定更好。一个巨型工具可能减少次数，却扩大权限和错误范围。更值得观察的是：失败后能否根据明确错误恢复，以及执行器能否独立判定操作合法性。

最终的工具目录应该是一组业务能力契约，而不是内部 HTTP 路由表的直接复制。

---

[查看系列导读与阅读路线]({{< relref "/agent-engineering/overview.md" >}})
