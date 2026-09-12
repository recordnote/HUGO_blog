---
title: "PostgreSQL 与 pgvector：知识库索引选型和容量估算"
date: 2026-04-22T10:00:00+08:00
lastmod: 2026-09-12T10:00:00+08:00
draft: false
author: Lin
avatar: /me/yy.jpg
cover: /img/sjk.png
images:
  - /img/sjk.png
categories:
  - 架构设计
tags:
  - PostgreSQL
  - RAG
slug: rag-postgres-pgvector
weight: 1
description: "以过滤选择性、召回质量和内存预算决定索引方案，而不是只比较向量检索耗时。"
---

以过滤选择性、召回质量和内存预算决定索引方案，而不是只比较向量检索耗时。

<!--more-->

> 本文于 2026-09-12 补充整理，按 2026-04 专题归档。示例为方案设计，参数用于说明方法，不代表已上线项目或实测成果。

## 先描述负载，再讨论数据库

对于已经使用 PostgreSQL 的团队，把业务元数据和向量放在一个数据库里，可以减少一种基础设施的运维成本。但能否适用，取决于数据规模、过滤条件、更新频率和隔离要求，不能只凭“支持向量”判断。

[pgvector 官方说明](https://github.com/pgvector/pgvector)提供精确检索、近似检索以及 HNSW、IVFFlat 等索引能力。下面关注如何评估这些能力，而不是宣称某种索引在所有场景下最好。

## 从最小精确基线开始

以下 SQL 用于已安装 pgvector 的实验数据库。3 维向量只是为了展示语法，真实维度要与 embedding 模型一致。

~~~sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE kb_chunk_demo (
    id bigint PRIMARY KEY,
    tenant_id text NOT NULL,
    body text NOT NULL,
    embedding vector(3) NOT NULL
);

CREATE INDEX ON kb_chunk_demo (tenant_id);

SELECT id, body
FROM kb_chunk_demo
WHERE tenant_id = 'tenant-a'
ORDER BY embedding <=> '[0.1,0.2,0.3]'::vector
LIMIT 5;
~~~

实验数据不要使用真实内部文档。`<=>` 表示余弦距离，数值越小越相近；不能拿另一个距离函数的阈值直接复用。

这个查询只演示租户过滤和距离排序，不包含项目级权限、参数绑定与生产账号配置。Java 中应使用绑定参数，身份值来自服务端。

## 建索引前保留正确答案

在代表性数据上保存精确检索结果，再创建近似索引作为对照。要观察的不只是查询耗时，还有相对于精确结果的召回、不同租户的返回数量、构建时间和更新影响。

HNSW 会增加内存和构建成本；IVFFlat 涉及列表划分与探测参数。调参应记录扩展版本和实际执行计划。带过滤的近似检索可能返回不足 K 条，不能把 LIMIT 5 理解为总能得到 5 个授权结果。

对于很小的租户子集，先利用普通索引过滤再精确排序，可能已足够。对于规模差异明显的租户，需要比较分区、部分索引或单独资源隔离的代价。

## 容量估算只是起点

假设有 100 万个切片，每个向量为 1536 维单精度浮点，仅向量数值约需：

~~~text
1,000,000 × 1,536 × 4 bytes
= 6,144,000,000 bytes
≈ 6.14 GB，或 5.72 GiB
~~~

这个数字不包含行存储开销、正文、索引、WAL、备份和副本，因此不能直接当作数据库内存需求。索引重建期间可能需要新旧版本共存，容量预算还应覆盖这个峰值。

输入维度改变时，不是简单更新模型名称就能继续混用。不同 embedding 空间的向量不可直接比较，需要新列、新表或新索引版本，以及成套重建和切换。

## 怎样做一个有用的选型实验

选取三个租户规模和三种权限选择性；分别测精确查询与候选近似索引。每组记录 Recall@K、p50/p95、返回不足 K 的比例、索引大小和批量更新期间的查询变化。

执行计划在实验环境用 `EXPLAIN (ANALYZE, BUFFERS)` 检查；它会真实执行查询，负载安排应与测量目的匹配。冷缓存与热缓存结果分开保存。

如果瓶颈来自数据库连接池或原文读取，替换向量索引可能并没有帮助。选型结论应明确“在哪个规模和约束内足够用”，再列出触发独立检索服务的条件，例如资源争用持续影响核心事务、隔离要求提高或扩容方式不再适合。

---

[查看系列导读与阅读路线]({{< relref "/agent-engineering/overview.md" >}})
