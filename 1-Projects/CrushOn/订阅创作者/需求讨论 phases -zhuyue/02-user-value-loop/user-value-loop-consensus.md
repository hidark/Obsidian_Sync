---
type: process/consensus
title: zhuyue Phase 2 共识：用户订阅价值闭环
tags:
  - crushon
  - creator-subscription
  - zhuyue
  - phase-2
  - consensus
phase: 2
author: zhuyue
commitments:
  - C2.1
  - C2.2
  - C2.3
  - C2.4
  - C2.5
  - C2.6
  - C2.7
  - C2.8
key-decisions:
  - 聊天惯性 + AI 剧情记忆是 Crushon 独有订阅身份权
  - iOS 方案改为 A'（完全隐藏 + Web 教育）
created: 2026-04-16
up: "[[debate-summary]]"
prev: "[[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/01-baseline-metrics/baseline-metrics-consensus]]"
next: "[[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/03-creator-incentive/creator-incentive-consensus]]"
related:
  - "[[debate-summary]]"
---

---
type: process/consensus
title: zhuyue Phase 2 共识：用户订阅价值闭环
tags: [crushon, creator-subscription, zhuyue, phase-2, consensus]
phase: 2
commitments: [C2.1, C2.2, C2.3, C2.4, C2.5, C2.6, C2.7, C2.8]
key-decisions:
  - "聊天惯性 + AI 剧情记忆是 Crushon 独有订阅身份权"
  - "iOS 方案改为 A'（完全隐藏 + Web 教育）"
created: 2026-04-16
up: "[[debate-summary]]"
prev: "[[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/01-baseline-metrics/baseline-metrics-consensus]]"
next: "[[1-Projects/CrushOn/订阅创作者/需求讨论 phases -zhuyue/03-creator-incentive/creator-incentive-consensus]]"
related:
  - "[[debate-summary]]"
---

# Phase 2 共识（v2）：用户订阅价值闭环

## 最终评分（综合 Proposer + Reviewer 对齐）

| 维度 | 评分 | 关键判据 |
|---|---|---|
| **钱包外延**（权重 3 生死） | 1 | 首购有增量（站内黏性 + 首付新客），但稳态钱包外延受续费力牵制，首月后崩盘风险高 |
| **续费力**（权重 3 生死） | 1 | 不至于 0（FANBOX 基准错配 + 聊天惯性 + AI 剧情记忆身份权），但需补 P0 四件套才能稳定在 40-45% |
| **与打赏互补**（权重 2） | 1 | 分工定位正确，但稀缺性影响分工稳定；若订阅退化为美图包即转替代 |
| **内容稀缺性**（权重 1） | 0 | "付费相册+对话图"不支撑 $14.99+ 档；必须补世界观档案+连载+投票 |
| **价格档位一致性**（权重 1） | 1 | 创作者经济不需平台级 tier 对齐（OF/DLsite 同理），但同创作者内 tier 权益差异仍缺失 |

**Phase 2 加权总分**：1×3 + 1×3 + 1×2 + 0×1 + 1×1 = **9 / 10**（本阶段权重池 10）
**判定**：**部分达成**（生死项均 1 分，未触 0 分否决；但内容稀缺性 0 分是 Phase 3 必解问题）

---

## 达成共识（C2.x 写入 core-commitments）

### C2.1 聊天惯性 + AI 剧情记忆是 Crushon 独有的订阅身份权
- 区别于 FANBOX/DLsite（周/月访问周期），Crushon 是**日活聊天产品**，用户每日打开 app 形成天然回访
- AI 角色的"剧情记忆 / 模型记忆 / 订阅者专属记忆插槽"是**真人平台没有的身份权维度**
- Proposer 续费基准错配已修正：FANBOX 40-45% 是站外导流数据，站内黏性 + 聊天惯性加成 8-15pp
- **PRD 必补**：订阅者专属记忆持久化入口、订阅身份在聊天场景可见（徽章/标记）

### C2.2 续费 P0 四件套（不可缺）
- **到期前提醒**（到期前 7-3-1 天）
- **创作者新内容主动 push**（站内通知 + 邮件可选）
- **连载承诺 + 月更节奏可见**（创作者主页展示"本月已发布 X/月更承诺 Y"）
- **世界观档案入口**（把订阅从"图包"升级为"持续访问权"）
缺任一 → 订阅退化为"带 Auto Renew 的一次性图包"，续费 <30%

### C2.3 iOS 合规方案回归 A' — 完全隐藏 + Web 教育
- Proposer 推荐的方案 C（模糊订阅按钮跳 Web）违反 App Store 3.1.3
- 参考 Netflix/Spotify 做法：iOS app 内**完全不提订阅入口**，仅支持已订阅内容查看
- 配合 **Email/Web push 教育**用户"订阅需在 Web 完成"
- 转化损耗参照 Netflix 25-30%（非 Proposer 初估 40-60%）

### C2.4 订阅不含聊天权益 → 前置教育三层文案
- **支付前**：订阅按钮下方固定文案 `Unlock subscriber-only albums and story content. Chat is not included.`
- **支付确认页**：明确"本订阅不包含聊天权益、会员权益、credits"
- **支付成功页**：第一屏直接带去"你刚解锁的专属内容"，避免用户先聊天再撞墙
- 认知风险不是主要矛盾；**真正风险是"价值感太纯"**（C1.5 稀缺性问题），不是 C1.2 认知问题

### C2.5 创作者经济不需平台级 tier 对齐
- 撤回"四档价格会造成跨创作者价格迷惑"的判断
- OF/DLsite/Fanbox 均是创作者自主定价；用户通常不跨创作者比价
- **仍需补**：同一创作者内不同价格档对应权益差异（Patreon tier 模式）—— 解决"付$29.99 和 $4.99 区别在哪"的问题

### C2.6 Phase 2 新增必补漏洞（合并 Proposer + Reviewer）
1. **订阅 vs 关注心智分离**：订阅入口必须在关注 UI 上方，权益差异一句话讲清
2. **Gift Sub（订阅赠送）**：PRD 空白。参考 Twitch Gift Sub 占 20-30% 增长贡献。AI 角色场景传播力可能更强
3. **订阅试用期 / 首月折扣**：C1.1 首购率的直接阻塞；DLsite/FANBOX/Netflix 首月折扣是标配
4. **订阅专属评论区**：身份权最低成本实现，应升为 P0
5. **订阅期内 vs 订阅前内容可见范围**：
   - 订阅前发布的存量内容 → 订阅后可访问（FANBOX 业态共识）
   - 订阅期内发布的内容 → 退订后仍可查看该期内容（降低退订心理成本，但保留续费理由）
6. **跨端权益同步**：iOS 买了 Android/Web 能看吗？账号绑定还是 receipt 绑定？
7. **创作者下架/封号处理**：剩余订阅周期退款/折算/消失？PRD 必明确
8. **90 天切档冷却 + 中断 Auto Renew 是设计错误**：老订阅者应价格保护，新订阅者新价生效
9. **订阅管理页缺月度消费总结**："本月你因订阅已解锁了什么"
10. **取消入口放错位置**：应在订阅列表，不在创作者页
11. **模糊预览缺信息**：应显示系列名/张数/发布日期，不只是高斯模糊

### C2.7 AI 内容首月价值密度评估
基于当前 PRD "相册 + 对话图" 形态：
- 头部创作者（10-30 组相册 + 5-15 对话图节点）：$4.99 成立，$14.99 勉强，$29.99/$49.99 危险
- 腰部（4-10 组相册 + 2-6 对话图）：仅 $4.99 试水心智成立
- 长尾（0-3 组 + 0-2 对话图）：退化为"花钱开一个空柜子"
- 在 PRD 不补世界观档案+连载+投票前，**$14.99 以上档位是创作者陷阱**

### C2.8 首月续费率估算
- 未补 P0 四件套 + C2.1 身份权：20-30%（崩盘）
- 补完 P0 四件套 + 聊天惯性 + 站内黏性：35-45%（触及 C1.4 下限）
- 补完 + AI 剧情记忆身份权 + 订阅者评论区：45-55%（达成 C1.4 目标）

---

## 回溯验证 vs Phase 1 承诺

| 承诺 | Phase 2 验证结果 |
|---|---|
| C1.1 钱包扩张 | ⚠️ 部分验证：首购可增长，但稳态依赖续费力，需 Phase 3 供给侧配合 |
| C1.2 订阅-打赏互补 | ✅ 支持：PRD 分工清楚，订阅身份权如补齐可形成真实互补 |
| C1.3 毛利公式 | ⚠️ 未直接验证（Phase 4 专题） |
| C1.4 续费阈值 40-45% | ⚠️ 当前 PRD 下 20-30%，需补 P0 四件套才能达成 |
| C1.5 稀缺性需转形态 | ❌ 强烈验证：当前"美图包"形态不支撑高价档，Phase 3 必须解决 |
| C1.6 合规 BLOCKER | ✅ 支持：iOS 方案 A' 是唯一合规路径 |

---

## 遗留分歧（小范围，不阻塞推进）
- **DV2.1 续费力评分（0 vs 1）**：已采用 1（综合站内黏性 + 身份权），Phase 3 创作者供给侧将验证 P0 四件套是否切实推进
- **DV2.2 iOS 方案 A 的 Web 教育效果**：具体 push 文案、频次、转化损耗待 PRD v2 验证

---

## 状态
- 本阶段：已完成（Round 1 后直接收敛，跳过 Round 2）
- 新增承诺：C2.1 - C2.8
- 下一阶段：Phase 3 - 创作者激励与冷启动（验证 C1.5 稀缺性落地 + C3 创作者 M2 留存）
