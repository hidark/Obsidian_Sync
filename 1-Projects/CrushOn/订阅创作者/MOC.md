---
type: MOC
title: 订阅创作者功能 — 知识图谱索引
tags: [crushon, creator-subscription, MOC]
created: 2026-04-20
---

# 订阅创作者功能 — 知识图谱索引

## 原始需求文档

- [[订阅创作者]] — PRD 原稿（PDF 源文件）

---

## 产出文档（最终决策层）

> 阅读建议：先看 executive-summary，再看 prd，最后看 prd-supplement。

- [[executive-summary]] — 执行摘要（CEO/投资人视角）
- [[prd]] — 完整产品需求文档（6 阶段辩论产出，8 章）
- [[prd-supplement]] — PRD 补充文档（整合 zhuyue 讨论结论，升级 BLOCKER 条件）
- [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/decision-log]] — 所有关键决策及理由（D1.1-D6.4）

---

## 辩论过程 A：Claude 主导（6 阶段）

> 入口文件：[[debate-framework]] → [[core-commitments]]

### 框架与总览

- [[intent-brief]] — 意图澄清（评估目标、利益方、交付物）
- [[debate-framework]] — 辩论阶段设计（Phase 1-6）
- [[core-commitments]] — 核心承诺汇总（C1.1-C6.4，共 22 条）

### Phase 1：订阅机制完整性

- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/01-subscription-mechanism/subscription-mechanism-consensus]] — 共识（C1.1-C1.7）
- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/01-subscription-mechanism/subscription-mechanism-backtrack-check]] — 回溯验证

### Phase 2：创作者激励体系

- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/02-creator-incentives/creator-incentives-consensus]] — 共识（C2.x）
- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/02-creator-incentives/creator-incentives-backtrack-check]] — 回溯验证

### Phase 3：订阅者价值闭环

- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/03-subscriber-value/subscriber-value-consensus]] — 共识（C3.x）
- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/03-subscriber-value/subscriber-value-backtrack-check]] — 回溯验证

### Phase 4：平台商业模式

- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/04-platform-business/platform-business-consensus]] — 共识（C4.x）
- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/04-platform-business/platform-business-backtrack-check]] — 回溯验证

### Phase 5：利益平衡

- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/05-interest-balance/interest-balance-consensus]] — 共识（C5.x）
- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/05-interest-balance/interest-balance-backtrack-check]] — 回溯验证

### Phase 6：优化建议与最终 PRD

- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/06-optimization-recommendations/optimization-recommendations-consensus]] — 共识（C6.1-C6.4）
- [[1-Projects/CrushOn/订阅创作者/creator-subscription-analysis/phases/06-optimization-recommendations/optimization-recommendations-backtrack-check]] — 22 条承诺全量回溯（0 项违规）

---

## 辩论过程 B：zhuyue 主导（5 阶段）

> 入口文件：[[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/MOC]]（子索引）→ [[debate-summary]] → [[prd-review]]

### 总览

- [[debate-summary]] — 辩论全程摘要（49 条承诺，重大反转点）
- [[prd-review]] — PRD v1 综合评审报告（13/28 分，BLOCKER 阻断）
- [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/decision-log]] — zhuyue 版决策日志

### Phase 1：基线与 KPI 矩阵

- [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/01-baseline-metrics/baseline-metrics-consensus]] — C1.1-C1.8（Pixiv FANBOX 类比、毛利公式、稀缺性警报）

### Phase 2：用户订阅价值闭环

- [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/02-user-value-loop/user-value-loop-consensus]] — C2.1-C2.8（聊天惯性、AI 记忆身份权、iOS A'）

### Phase 3：创作者激励与冷启动

- [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/03-creator-incentive/creator-incentive-consensus]] — C3.1-C3.9（订阅者记忆基础设施、打赏筛选论）

### Phase 4：体系博弈 + 支付合规

- [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/04-system-payment/system-payment-consensus]] — C4.A（7 指标监控 + A/B 实验）+ C4.B（NSFW 发布硬门槛）

### Phase 5：风险与改进优先级

- [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/05-risks-priorities/round-01-proposer]] — 7 项 BLOCKER + 13 项 P0 终审

---

## 交叉比较

- [[output/comparison-with-zhuyue-discussion]] — 两份讨论的系统性对比（类比参照、MVP 条件、遗漏识别）

---

## 关键共识速查

| 承诺 | 内容 | 来源 |
|------|------|------|
| 类比参照 | Pixiv FANBOX + DLsite（非 OnlyFans） | [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/01-baseline-metrics/baseline-metrics-consensus]] |
| MVP 条件 | 7 项 BLOCKER（非 6 项 P0） | [[prd-supplement]] |
| 内容形态 | 世界观档案 + 剧情连载 + 投票（非美图包） | [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/01-baseline-metrics/baseline-metrics-consensus]] |
| 毛利反转 | 订阅替代打赏对平台毛利可能正向 | [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/04-system-payment/system-payment-consensus]] |
| 记忆基础设施 | 订阅者专属记忆须为平台级基础设施 | [[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/03-creator-incentive/creator-incentive-consensus]] |
| 成本估算 | $60K-$90K，8-10 周（修正后） | [[prd-supplement]] |
