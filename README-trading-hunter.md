# 猎手模拟交易 — Agent 化实战案例

> **场景**：量化策略 + 模拟仓 + 日频 pipeline + 用户零运维  
> **方法论对齐**：[agent-methodology](../README.md) 证据链 · 门卫 · 教训闭环 · Verifier 分离  
> **工程化参考**：[SCALE Engine Shield](https://gitee.com/hongmaple/scale-engine) 轻量门禁（退出码 + Cursor Hook）

---

## 改造前痛点

| 问题 | 表现 |
|------|------|
| 证据在人脑 | 门禁状态、持仓、Phase1 进度每次对话重讲 |
| 改策略无核验 | 动 `signal_filter` 后单测/集成无人跑 |
| 展示与执行混淆 | 主页「看多」与 `can_open_new=false` 被当成矛盾 |
| 用户被迫运维 | 烤机 accept、看 postmortem 网页、跑 shell |
| 会话失忆 | Agent 每轮重新「发明」架构决策 |

---

## 改造后架构

```
日频 pipeline（cron）
    │
    ├─ 行情/候选/信号/路由 ──→ daily/*.json（P0 证据自动刷新）
    ├─ signal_postmortem ────→ 板块/来源命中率（只读观察）
    ├─ strategy_grill ───────→ 指纹 + 单测 + Phase1 影响
    ├─ health_check ─────────→ 24 项集成验收
    ├─ phase1_journal ───────→ 给主人的唯一摘要
    └─ zhuli_deploy ─────────→ 主页同步

Agent 改代码
    │
    ├─ preToolUse Hook ──────→ agent_gate（受保护路径须 grill PASS）
    ├─ afterFileEdit Hook ───→ needs_grill 标记
    ├─ grill + health ───────→ BLOCKER 则 Agent 自修
    └─ memory/lessons.md ────→ confirmed 教训升格
```

---

## 与经典数仓模式的差异

| 维度 | 数仓（企业数据平台） | 猎手交易 |
|------|----------------------|----------|
| **P0 证据来源** | 用户放 model_csv（看 mtime） | **pipeline 自动写 JSON** |
| **用户角色** | 按模板交设计稿 | **零运维**：不跑命令、不点 accept |
| **Verifier** | warehouse_pre_edit_guard | strategy_grill + health_check |
| **业务复盘** | 行数/金额抽样 | signal_postmortem 分桶命中率 |
| **冻结期** | 口径变更窗口 | Phase1 验证期冻结门禁参数 |
| **Hook 策略** | 改表前注入 guard 摘要 | **拦截**受保护路径直至烤机通过 |

核心洞察：**证据不必等人交——能脚本化的日频真相，应由 pipeline 产出，Agent 只读磁盘。**

---

## 文件地图（可复制骨架）

| 层级 | 路径 | 作用 |
|------|------|------|
| 索引 | `AGENTS.md` | 启动读 rules + lessons |
| Rules | `.cursor/rules/evidence-priority.mdc` | P0/P1/P2 表 |
| Rules | `.cursor/rules/agent-loops-workflow.mdc` | Agent-only、改后收尾 |
| 教训 | `memory/lessons.md` | draft → confirmed |
| 烤机 | `tools/strategy_grill.py` | WATCHED 指纹 + 单测 |
| 健康 | `tools/health_check.py` | BLOCKER/SHOULD/INFO |
| 复盘 | `tools/signal_postmortem.py` | 看错/看对分桶 |
| 门禁 | `tools/agent_gate_hook.py` | SCALE Shield 轻量版 |
| Hook | `.cursor/hooks.json` | preToolUse + afterFileEdit |
| 日记 | `daily/phase1_journal.md` | 主人唯一必读摘要 |

模板见 [`examples/trading-hunter/`](examples/trading-hunter/)。

---

## 证据优先级（P0 / P1 / P2）

```
P0（当日真相，pipeline 刷新）
  portfolio.json · strategy_routing.json · market_context.json · phase1_validation.json

P1（改策略时读）
  position_config_v4.json · candidate_stocks.json · unified_signals.json · etf_watch.json

P2（辅助，不替代 P0）
  lessons.md · signal_postmortem.json · strategy_grill.json · health_check.json
```

冲突：**portfolio + routing** 为准，记入 `memory/YYYY-MM-DD.md`。

---

## Loops 工作流（Boris Cherny）

> 写 **loops**（可重复脚本），不要堆 **IDE 改代码**。

| Loop | 脚本 | 产出 |
|------|------|------|
| 烤机 | `strategy_grill.py --pipeline` | `daily/strategy_grill.json` |
| 健康 | `health_check.py --soft` | `daily/health_check.json` |
| 复盘 | `signal_postmortem.py` | `daily/signal_postmortem.json` |
| 日记 | `phase1_process_journal.py` | `daily/phase1_journal.md` |

**严重级别**：grill `FAIL` / health `critical` → **BLOCKER**（Agent 必须修完再汇报）；全 PASS → 不必打扰主人。

---

## 已沉淀教训（节选）

| ID | 规则摘要 |
|----|----------|
| L-003 | 个股与 ETF 必须分轨，禁止混池 |
| L-004 | 门禁关闭 ≠ 信号消失；执行只听 routing + filter |
| L-005 | 复盘样本 &lt;20 不调权重，`observe_only` |
| L-006 | 证据 pipeline 产出；主人零运维 |
| L-007 | Phase1 冻结期不改门禁/辩论阈值 |
| L-008 | 受保护文件须过 grill + agent_gate |

完整条目见 `examples/trading-hunter/lessons-template.md`。

---

## 门卫：agent_gate + strategy_grill

```
编辑 WATCHED 内文件
    → afterFileEdit: needs_grill=true
    → 再次编辑: preToolUse deny (exit 2)
    → Agent 跑 strategy_grill --pipeline
    → PASS → clear_gate + 刷新基线
```

受保护路径与 `strategy_grill.WATCHED` 同步（配置、gate、filter、router 等）。

---

## 搭建提示词（复制即用）

```markdown
# 任务：为【量化/交易】项目搭建 Agent 协作体系（猎手模式）

约束：
- 用户零运维：证据由日频 pipeline 写 JSON，用户不交 model_csv
- 改策略后自动：grill → health → journal → lessons
- Phase1 验证期列出冻结参数，Agent 不得擅自改
- 展示层信号与执行层门禁分离文档化

请产出：
1. AGENTS.md（参考 AGENTS-trading-hunter.md）
2. .cursor/rules/evidence-priority.mdc + agent-loops-workflow.mdc
3. tools/strategy_grill.py 骨架（WATCHED 列表 + 单测入口）
4. tools/health_check.py 骨架（BLOCKER/SHOULD 分级）
5. memory/lessons.md 模板（draft / confirmed）
6. .cursor/hooks.json + agent_gate_hook.py 骨架

验收：改 WATCHED 内文件 → Hook 拦截 → grill 通过后解除。
```

---

## 量化收益（主观）

- Agent 改策略**返工率**明显下降（烤机 FAIL 在合并前拦住）
- 主人从「看报告、点 accept」变为**只看异常**
- 教训库 8 条 confirmed，避免重复踩坑（银行看多、ETF 混池等）
- 与 SCALE 完整引擎相比：**只借 Shield 思路**，不引入过重治理栈

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [`AGENTS-trading-hunter.md`](AGENTS-trading-hunter.md) | 交易场景 AGENTS 模板 |
| [`docs/04-agent-loops-zero-ops.md`](docs/04-agent-loops-zero-ops.md) | Loops + 零运维模式详解 |
| [`examples/trading-hunter/`](examples/trading-hunter/) | 可复制的 rules / hooks 模板 |
| [`agent_setup_prompts_by_scenario.md`](agent_setup_prompts_by_scenario.md) §15 | 场景 K 提示词 |

---

*案例来源：猎手模拟交易 Phase1（2026-06）· 方法论仓库同步*
