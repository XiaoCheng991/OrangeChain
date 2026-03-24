# 学术论文助手

## What This Is

一个基于LangChain的学术文献问答系统，帮助研究者快速理解论文、提取关键信息、跨文献关联。支持中英双语，可解析PDF和arXiv链接，提供混合检索、引用溯源、图文多模态解析等核心能力。

**核心价值**：用AI加速科研阅读，让复杂论文几分钟内转化为可执行的知识图谱。

## Core Value

让研究者用自然语言快速获取论文核心信息，答案可靠可溯源。

## Requirements

### Validated

(None yet — ship to validate)

### Active

See: [REQUIREMENTS.md](./REQUIREMENTS.md) (16 requirements, v1.0)

**Summary**:
- **P0 (MVP)**: 12个需求 - PDF/arXiv解析、向量检索、BM25、基础问答、引用溯源、流式输出
- **P1 (增强)**: 4个需求 - 混合检索、重排序、表格提取、多轮对话、检索评估

### Out of Scope

### Out of Scope

- [实时arXiv推送订阅] — 聚焦核心问答能力，不进入内容分发领域
- [论文写作生成] — 辅助阅读理解，不替代原创写作
- [多语言支持超出中英] — 先做深，后续根据需求扩展

## Context

**学习者背景**：
- 编程基础：熟悉Java、Python，有Web开发经验
- AI基础：了解大模型基本概念，使用过ChatGPT/Claude
- 目标：掌握LangChain核心 → RAG系统 → Agent架构 → 生产化部署
- 周期：1个月，每周15-20小时投入

**求职目标**：
- 目标公司：互联网公司或AI初创
- 面试偏好：问答系统、知识管理话题
- 能力展示：快速掌握能力、极强学习能力

**技术决策记录**：
- 模型选择：minimaxai/minimax-m2.1（NVIDIA提供免费API）
- 技术亮点聚焦：检索优化 + 自动评估 + 多模态
- 部署目标：在线SaaS，可分享链接给面试官

## Constraints

- **时间周期**：1个月完成 — 需快速迭代，功能做减保核心
- **投入时间**：每周15-20小时 — 按4周规划，约60-80小时总投入
- **成本控制**：使用免费的minimax-m2.1 API — 避免高额token费用
- **本地部署**：最终需要可在线访问的demo — 影响部署方案选择
- **简历价值**：项目需足够亮眼，能写入简历并现场演示 — 需要完整的架构和可展示的技术深度

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| 选择学术论文助手场景 | 数据质量高、展示点多、面试影响力大 | — Pending |
| 聚焦检索优化 | 展示对LangChain底层机制的精通能力 | — Pending |
| 加入自动评估 | 展示工程化思维（很多开发者忽略） | — Pending |
| 支持多模态解析 | 满足论文中大量公式图表的实际需求 | — Pending |
| 使用minimax-m2.1模型 | 免费API、降低成本，同时展示模型集成能力 | — Pending |
| 中英双语支持 | 国际学术论文普遍为英文，兼顾国内需求 | — Pending |
| 在线SaaS部署 | 可分享链接给面试官，提升项目影响力 | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-03-24 after initialization*
