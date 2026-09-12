# 2026 年 3—9 月博客补充规划

整理日期：2026-09-12。定位：Java 后端与架构经验，向 Agent 开发、Agent 架构和大模型应用延伸。

## 扫描结论

- 扫描时 content 下有 121 个 Markdown 文件，包含文章、关于页及索引；不是 121 篇原创文章。
- 最新正文为 2026-03-04 的上下文工程，已有 Agent 循环、工具、Skills、Multi-Agent、RAG、A2A 和 Spring AI / LangChain4j 相关介绍。
- Java 和架构内容涵盖 JVM、并发、数据库、消息队列、权限、高可用等。后续重点应是用业务案例展示取舍与验证，不再重复框架概念。
- 技术定位已在关于页表述为 Java 向 AI 应用拓展；本轮增加专题入口，不编造公司、工作年限和项目成绩。
- 主题为 Dream，沿用现有 author、avatar、cover、images、categories、tags、weight 元数据。新增内容放在独立的 content/agent-engineering 目录。

## 日期与数量口径

从 3 月 4 日到 9 月 12 日约六个月，但包含 3—9 月七个自然月，九月尚未结束。采用每月前四个周三的近似周更节奏；第五周不额外补文。

| 月份 | 已有计入 | 新增正文 | 后续草稿 | 月度合计 |
| --- | ---: | ---: | ---: | ---: |
| 3 月 | 1 | 3 | 0 | 4 |
| 4 月 | 0 | 4 | 0 | 4 |
| 5 月 | 0 | 4 | 0 | 4 |
| 6 月 | 0 | 4 | 0 | 4 |
| 7 月 | 0 | 4 | 0 | 4 |
| 8 月 | 0 | 4 | 0 | 4 |
| 9 月 | 0 | 2 | 2 | 4 |

新增 27 篇文章文件，其中 25 篇 draft: false，2 篇 draft: true；另有 1 篇系列导读，不计入月度篇数。所有日期为 2026 年，使用 +08:00 时区。

历史日期仅作补充归档，正文明确说明 2026-09-12 整理，lastmod 也记录该日期。不是伪造历史 Git 提交或真实周更经历。09-16 与 09-23 的未来正文保持草稿，默认 Hugo 构建不输出；届时审阅后将 draft 改为 false，并在系列导读中把对应计划条目改成文章链接。仓库有定时部署工作流，但 draft: true 不会在到期后自动发布。

## 月度选题与岗位关联

| 月份 | 主线 | 主要展示能力 |
| --- | --- | --- |
| 3 月 | 从模型调用转向业务工程 | 补齐输出契约、任务验收与架构边界；承接已有上下文工程文章。 |
| 4 月 | RAG 质量与数据治理 | 建立可更新、可隔离、可评测的知识链路。 |
| 5 月 | 工具与权限治理 | 把协议连通推进到可授权、可审批、可核对的业务动作。 |
| 6 月 | Java 可靠执行 | 展示状态恢复、事务消息、并发隔离和流式接口能力。 |
| 7 月 | 平台运维与容量 | 讨论模型路由、观测、成本和压测，形成工程闭环。 |
| 8 月 | 复杂 Agent 与质量保证 | 克制地引入协作和记忆，用发布评测与演练验证边界。 |
| 9 月 | 架构表达与作品集 | 汇总平台蓝图和 ADR，准备证据链与渐进迁移文章。 |

## 完整文章清单

| 归档日期 | 文章 | 定位 | 状态 |
| --- | --- | --- | --- |
| 2026-03-11 | [从业务流程到 Agent：企业知识库与工单助手的架构边界](../content/agent-engineering/01-workflow-or-agent.md) | 架构设计 / Agent | 正文初稿，可随构建输出 |
| 2026-03-18 | [Java 大模型应用的输出契约：从 JSON 到可执行业务命令](../content/agent-engineering/02-structured-output-contract.md) | Java / 大模型应用 | 正文初稿，可随构建输出 |
| 2026-03-25 | [Agent 评测基线：把演示效果变成可重复的验收条件](../content/agent-engineering/03-agent-evaluation-baseline.md) | Agent / 评测 | 正文初稿，可随构建输出 |
| 2026-04-01 | [RAG 文档入库设计：切分、版本与增量更新](../content/agent-engineering/04-rag-ingestion-versioning.md) | RAG / 数据工程 | 正文初稿，可随构建输出 |
| 2026-04-08 | [混合检索调优：用失败样本决定召回与重排策略](../content/agent-engineering/05-rag-hybrid-retrieval.md) | RAG / 检索 | 正文初稿，可随构建输出 |
| 2026-04-15 | [企业 RAG 的权限与引用：检索结果如何成为可信证据](../content/agent-engineering/06-rag-permission-citations.md) | RAG / 权限设计 | 正文初稿，可随构建输出 |
| 2026-04-22 | [PostgreSQL 与 pgvector：知识库索引选型和容量估算](../content/agent-engineering/07-rag-postgres-pgvector.md) | PostgreSQL / RAG | 正文初稿，可随构建输出 |
| 2026-05-06 | [Agent 工具契约设计：参数、错误与业务边界](../content/agent-engineering/08-tool-contract-design.md) | Agent / 工具调用 | 正文初稿，可随构建输出 |
| 2026-05-13 | [MCP 接入企业系统：身份传播与最小权限设计](../content/agent-engineering/09-mcp-identity-authorization.md) | MCP / 权限设计 | 正文初稿，可随构建输出 |
| 2026-05-20 | [Agent 提示词注入防护：把外部内容留在数据边界内](../content/agent-engineering/10-prompt-injection-defense.md) | Agent / 安全设计 | 正文初稿，可随构建输出 |
| 2026-05-27 | [人工审批与幂等执行：让 Agent 安全创建工单](../content/agent-engineering/11-approval-idempotency.md) | Agent / 分布式事务 | 正文初稿，可随构建输出 |
| 2026-06-03 | [长任务 Agent 状态机：检查点、租约与故障恢复](../content/agent-engineering/12-durable-agent-state.md) | Agent / 状态机 | 正文初稿，可随构建输出 |
| 2026-06-10 | [Agent 异步任务的事务消息：Outbox 与幂等消费](../content/agent-engineering/13-transactional-outbox-agent.md) | 消息队列 / 分布式事务 | 正文初稿，可随构建输出 |
| 2026-06-17 | [Java 21 虚拟线程接入大模型：并发隔离与容量边界](../content/agent-engineering/14-java-virtual-thread-bulkhead.md) | Java / 并发 | 正文初稿，可随构建输出 |
| 2026-06-24 | [Java 流式 AI 接口：SSE、取消传播与慢客户端治理](../content/agent-engineering/15-sse-streaming-cancellation.md) | Java / SSE | 正文初稿，可随构建输出 |
| 2026-07-01 | [企业模型网关设计：能力路由、配额与降级](../content/agent-engineering/16-model-gateway-routing.md) | 大模型应用 / 架构设计 | 正文初稿，可随构建输出 |
| 2026-07-08 | [Agent 可观测性设计：从一次请求追踪到业务终态](../content/agent-engineering/17-agent-observability.md) | Agent / 可观测性 | 正文初稿，可随构建输出 |
| 2026-07-15 | [大模型应用成本治理：Token 预算、缓存与质量约束](../content/agent-engineering/18-llm-cost-cache-budget.md) | 大模型应用 / 成本治理 | 正文初稿，可随构建输出 |
| 2026-07-22 | [AI 服务容量规划：长尾延迟、排队与压测模型](../content/agent-engineering/19-ai-capacity-load-test.md) | Java / 性能测试 | 正文初稿，可随构建输出 |
| 2026-08-05 | [多 Agent 协作的工程取舍：何时拆分，如何收敛](../content/agent-engineering/20-multi-agent-coordination.md) | Multi-Agent / 架构设计 | 正文初稿，可随构建输出 |
| 2026-08-12 | [Agent 长期记忆治理：写入条件、过期与删除](../content/agent-engineering/21-agent-memory-governance.md) | Agent / 上下文工程 | 正文初稿，可随构建输出 |
| 2026-08-19 | [Agent 版本发布：离线评测、影子流量与灰度回滚](../content/agent-engineering/22-agent-release-evaluation.md) | Agent / 评测 | 正文初稿，可随构建输出 |
| 2026-08-26 | [Agent 故障演练设计：重复执行、越权检索与预算失控](../content/agent-engineering/23-agent-failure-drills.md) | Agent / 可靠性 | 正文初稿，可随构建输出 |
| 2026-09-02 | [Java 企业 Agent 平台蓝图：从模块边界到部署拓扑](../content/agent-engineering/24-java-agent-platform-blueprint.md) | Java / Agent架构 | 正文初稿，可随构建输出 |
| 2026-09-09 | [Agent 架构决策记录：用 ADR 解释五个关键取舍](../content/agent-engineering/25-agent-architecture-decisions.md) | 架构设计 / Agent | 正文初稿，可随构建输出 |
| 2026-09-16 | [Agent 项目展示设计：把架构文章连接到可验证证据](../content/agent-engineering/26-agent-portfolio-evidence.md) | Agent / 工程实践 | 未来草稿 |
| 2026-09-23 | [存量 Java 系统接入 Agent：渐进迁移与演进路线](../content/agent-engineering/27-java-agent-migration.md) | Java / 架构设计 | 未来草稿 |

## 优先精修的六篇

如果先选择少量文章用于简历链接，建议从以下六篇中选择与真实经验匹配的内容，并补上自己的实现与实验：

- [人工审批与幂等执行：让 Agent 安全创建工单](../content/agent-engineering/11-approval-idempotency.md)：把审批绑定到具体命令，用业务幂等和结果对账处理超时后的不确定执行状态。
- [长任务 Agent 状态机：检查点、租约与故障恢复](../content/agent-engineering/12-durable-agent-state.md)：将任务事实持久化，用租约和版本校验处理进程重启与重复调度。
- [Java 21 虚拟线程接入大模型：并发隔离与容量边界](../content/agent-engineering/14-java-virtual-thread-bulkhead.md)：把线程调度能力与下游资源配额分开治理，避免更轻量的线程放大依赖压力。
- [AI 服务容量规划：长尾延迟、排队与压测模型](../content/agent-engineering/19-ai-capacity-load-test.md)：用到达率、服务时间和资源池解释吞吐上限，设计能暴露排队问题的负载实验。
- [Java 企业 Agent 平台蓝图：从模块边界到部署拓扑](../content/agent-engineering/24-java-agent-platform-blueprint.md)：串起身份、编排、知识、工具和模型网关，给出可以分阶段实现的平台设计。
- [Agent 架构决策记录：用 ADR 解释五个关键取舍](../content/agent-engineering/25-agent-architecture-decisions.md)：围绕流程、存储、协议、并发和模型路由记录约束、代价与重新评估条件。

## 内容标准

每篇用具体问题开场，给出方案取舍、数据或接口示例、失败边界以及验证方法。正文为 AI 辅助编写的技术初稿，应在正式求职使用前逐篇阅读并验证。不要把说明性代码称为完整可运行应用，也不要把设计参数写成实测成果。

代码示例以 Java 21 类型和标准并发 API、PostgreSQL SQL、JSON/YAML 契约为主。Java 类片段没有配套完整业务工程；SQL 涉及扩展或演示表时注明前提。工具协议、模型能力和框架行为引用官方资料，不宣传未核实的最新版本。

文章未声称真实公司的收益、用户量、生产 QPS 或上线经历。后续有真实实验时，应补充环境、版本、样本和原始结果，再更新 lastmod。

## 阅读入口与本地检查

[系列导读源文件](../content/agent-engineering/overview.md)会在站点生成 /agent-engineering/overview/。关于页也提供专题链接。历史文章保持原样。

正常检查使用 hugo，草稿检查使用 hugo --buildDrafts --buildFuture。建议用 --destination 指向仓库外独立临时目录，避免生成文件混入源文件改动。

本机命令路径虽然包含 0.145.0，实际 hugo version 返回 v0.89.1；GitHub Actions 配置使用 0.119.0。验证结果应以实际运行的版本为准，不能将本地构建视为云端部署完成。本轮不提交、不推送、不触发站点部署。

## 本轮验证记录

- 本机 Hugo v0.89.1 普通构建与 --buildDrafts --buildFuture 构建均通过。新增文章未使用需要额外 taxonomy 配置的 series 字段，以兼容当前主题和本机版本。
- 普通构建输出 25 篇新增文章和 1 篇导读；包含草稿时输出全部 28 个新页面。两篇未来草稿未进入普通构建的首页搜索 JSON、RSS 和 sitemap。
- 27 篇正文含约 27,781 个汉字（不包含前置元数据，统计包含正文说明与导航），每篇约 906—1,171 个汉字，另外包含代码、表格和英文术语。
- 已检查新文章日期、周三归档安排、封面文件、内部引用、JSON 代码块和 Markdown 围栏；已验证生成 HTML 中的 76 处系列内部链接、9 个表格和 28 个代码块。
- git diff --check 通过。没有运行完整 Java / 数据库示例系统，没有进行真实模型效果评测、负载测试或云端部署；文中明确将这些内容标为方案和待实施实验。

## 存量问题说明

扫描发现两处历史日期不是合法日历日期：分布式任务调度为 2022-11-31，分布式 session 为 2022-06-31。本地旧版 Hugo 仍可构建；本轮没有擅自猜测原发布日期并修改。两个高可用文章标题相同、社交链接有 Lin 占位值，也适合后续内容清理时核实。
