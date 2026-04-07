---
title: LLM Wiki Architecture
type: concept
created: 2026-04-06
related:
  - "[[LLM Wiki Philosophy]]"
  - "[[LLM Wiki Operations]]"
  - "[[INDEX]]"
---

# LLM Wiki Architecture 架构

## 三层架构

### 1. 原始资料层 (raw/)

你精心整理的原始文件收藏：
- 文章、论文、图片、数据文件
- 不可变 — LLM 从中读取，但从不修改
- 真相来源

### 2. Wiki 层 (wiki/)

LLM 生成的 markdown 文件目录：
- 摘要页
- 实体页
- 概念页
- 比较
- 概述
- 综合

LLM 完全拥有这一层，负责创建、更新、维护交叉引用、保持一致性。

### 3. 模式层 (Schema)

配置文件（如 CLAUDE.md、AGENTS.md）：
- 告诉 LLM Wiki 的结构、惯例
- 定义工作流程
- 让 LLM 成为有纪律的 Wiki 维护者

## 索引与记录

### index.md

以内容为导向 —— wiki 中所有内容的目录
- 每页附带链接和一句摘要
- 可选元数据（日期、来源数量）
- 按类别组织（实体、概念、来源）
- 回答查询时，LLM 先读索引找相关页面

### log.md

按时间顺序排列的活动记录
- 摄入、查询、健康检查的时间线
- 建议：每个条目以一致前缀开头，便于 Unix 工具解析
- 帮助 LLM 理解最近工作进展

## 来源

- 详见原始文档: [[llm-wiki]]
