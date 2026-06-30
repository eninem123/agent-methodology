# 【项目名】— 已确认教训库

> 对齐 [agent-methodology 教训闭环](https://github.com/eninem123/agent-methodology#5-教训闭环-loop经验传承)  
> **Agent 改策略前必读**；主人不维护本文件。  
> 新教训：postmortem / grill FAIL / Phase1 未达标 → Agent 追加，`status: confirmed` 后方可当红线引用。

---

## 条目格式

```markdown
## L-00N · 标题

- **status**: confirmed | draft
- **source**: 【signal_postmortem / grill / 架构决策 + 日期】
- **fact**: 【可验证事实，含样本量 n=】
- **rule**: 【Agent 可执行的一句话规则】
- **tags**: 【sector, gate, architecture …】
```

---

## 示例（来自猎手交易）

## L-003 · 个股与 ETF 必须分轨

- **status**: confirmed
- **source**: 架构决策 2026-06-29
- **fact**: ETF 与个股混池导致观察混乱
- **rule**: 个股走 candidate/signal_hub；ETF 走独立 etf_watch；禁止混池
- **tags**: architecture, etf

## L-006 · 主人零运维

- **status**: confirmed
- **source**: 主人明确要求
- **fact**: 主人不跑 accept、不看复盘网页
- **rule**: 证据由 pipeline 产出；异常只报 BLOCKER；结论写 phase1_journal
- **tags**: ops, agent-only

## L-008 · 策略关键文件须过烤机门禁

- **status**: confirmed
- **source**: agent_gate_hook（SCALE Shield 轻量版）
- **fact**: 未烤机即改 WATCHED 内文件导致基线漂移
- **rule**: Hook 拦截；needs_grill 时先 strategy_grill --pipeline
- **tags**: grill, agent_gate

---

## 待确认（draft，不可当红线）

| id | 摘要 | 待验证 |
|----|------|--------|
| D-001 | 【示例】某板块 n=2 命中率 100% | 样本≥5 |

---

_自动复盘见 `daily/signal_postmortem.md`；本文件只保留 confirmed 教训。_
