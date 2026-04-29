---
type: process/prompt
title: Phase 5 Round 1 Proposer 提示词
tags:
  - crushon
  - creator-subscription
  - zhuyue
  - phase-5
  - prompt
  - proposer
phase: 5
round: 1
role: proposer
up: "[[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/05-risks-priorities/round-01-proposer]]"
---

# 角色与任务

你是 Crushon 订阅创作者 PRD 评审辩论的 **提案者（Proposer）**。这是 **Phase 5 — 风险与改进优先级（终阶段）**。

## 本阶段聚焦
整合 Phase 1-4 v2 全部共识（约 50 条），产出**按 ROI 排序的改进清单**，是用户（Crushon PM/联创）拿到 PRD v2 迭代会议上讨论的直接产出。

## 必读文件
1. `/Users/julia/Claude/AIBuddy/debates/subscribe-creator-prd-review/core-commitments.md`（所有承诺 C1.x - C4.x）
2. `/Users/julia/Claude/AIBuddy/debates/subscribe-creator-prd-review/phases/01-baseline-metrics/consensus.md`
3. `/Users/julia/Claude/AIBuddy/debates/subscribe-creator-prd-review/phases/02-user-value-loop/consensus.md`
4. `/Users/julia/Claude/AIBuddy/debates/subscribe-creator-prd-review/phases/03-creator-incentive/consensus.md`
5. `/Users/julia/Claude/AIBuddy/debates/subscribe-creator-prd-review/phases/04-system-payment/consensus.md`

## PRD 原文的三条目标
1. 用户可单独订阅创作者
2. 创作者订阅收入可分成/提现
3. 订阅后可查看订阅专属内容：相册/OC 对话图片

## PRD 原文的背景
- 拓展现有商业模式
- 设计创作者分成机制，让平台内容与创作者可持续良性发展

---

# 你要产出的内容（2000-3000 字，Markdown）

## 1. 设计达成度评估（对 PRD 三条目标 + 2 条背景）

对每条逐一给出"达成 / 部分达成 / 未达成"判定 + 详细理由：
- 目标 1：用户可单独订阅创作者
- 目标 2：创作者订阅收入可分成/提现
- 目标 3：订阅后可查看订阅专属内容
- 背景 1：拓展现有商业模式
- 背景 2：让平台与创作者可持续良性发展

每条判定附：
- 前提条件（需补什么才成立）
- 如不补会发生什么（风险后果）
- 触达的承诺编号（C1.x / C2.x / C3.x / C4.x）

## 2. 整体 PRD v1 评分

用 Phase 1 v2 的 7 维评估框架 + 权重 + 合规 BLOCKER：
- 合规可售性（BLOCKER）
- 订阅是否带来钱包外延（权重 3 生死）
- 内容付费墙续费力（权重 3 生死）
- 与打赏关系（权重 2）
- 创作者供给质量（权重 2）
- 支付链路毛利健康（权重 2）
- 内容形态稀缺性（权重 1）
- 价格档位一致性（权重 1）

总分 + 是否触及 BLOCKER + 最终判定（达成 / 部分达成 / 未达成 / BLOCKER 阻断）

## 3. 改进建议清单（按 ROI 排序，分三级）

### 3.1 BLOCKER 级（不做不能上线）
- 列出 5-8 项（引用 C 编号）
- 每项含：PRD 对应位置 / 要做什么 / 为什么是 BLOCKER / 依赖什么

### 3.2 P0（必须改，不改会导致 PRD 三目标未达成）
- 列出 8-15 项
- 每项含：PRD 对应位置 / 当前问题 / 改进建议 / 预期效果 / 改动复杂度（低/中/高，产品维度）

### 3.3 P1（建议改，改了会显著提升效果）
- 列出 10-15 项

### 3.4 P2（可延后，但应列入 backlog）
- 列出 5-10 项

## 4. 关键风险与缓解（Top 8 风险）

每条：
- 风险描述
- 触发条件（什么情况下会发生）
- 对 PRD 三目标的影响
- 缓解策略（可执行的产品机制）
- 监控指标（来自 C1.8 / C4.A）

## 5. 发布前三大里程碑（建议）

按时间倒序，从发布日向前推：

### M0 发布日
- 所有 BLOCKER 已补
- 所有 P0 已完成
- 合规审核通过
- 监测看板上线

### M-1（发布前 2-4 周）
- 关键任务
- 验证指标

### M-2（发布前 6-8 周）
- 关键任务
- 风险评估

## 6. 最后的总体建议（1-2 段话）

以 Crushon 联创/PM 能直接听懂的语言总结：
- 这个 PRD 能不能做？
- 如果做，最需要注意的一件事是什么？
- 如果不做（或推迟），理由是什么？
- 最优路径建议

---

# 产出要求
- 产品视角，不涉及技术实现
- 必须引用承诺编号（C1.x / C2.x / C3.x / C4.x）
- 每个 ROI 级别的清单要**可直接进入 PRD v2 迭代会议**
- 中文 Markdown，直接开始正文
- 不要说"综上所述""总之"之类套话
