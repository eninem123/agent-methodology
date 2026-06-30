# 变更说明 · 2026-06-30

## 摘要

基于 **猎手模拟交易** Phase1 实践，补充「量化/交易策略」场景的 Agent 化文档与可复制模板，并引入 **Loops + 零运维** 模式（pipeline 自动产 P0 证据）。

## 新增文件

| 文件 | 说明 |
|------|------|
| `README-trading-hunter.md` | 猎手交易 Agent 化完整案例 |
| `AGENTS-trading-hunter.md` | 交易场景 AGENTS.md 模板 |
| `docs/04-agent-loops-zero-ops.md` | Boris Cherny Loops + 零运维契约 |
| `examples/trading-hunter/*` | rules、lessons、hooks、pipeline 模板 |
| `optimization_changelog_20260630.md` | 本文件 |

## 更新文件

| 文件 | 变更 |
|------|------|
| `README.md` | 案例、适用场景、文件结构、P0 零运维扩展 |
| `agent_setup_prompts_by_scenario.md` | §15 场景 K（量化/交易）+ 速查表 |
| `docs/02-capability-mapping.md` | 猎手交易能力映射列 |
| `docs/03-verification-pattern.md` | 交易场景 Verifier 对照行 |

## 核心经验沉淀

1. **证据不必等人交**：日频 pipeline 写的 JSON 等价于数仓的 model_csv
2. **四层 Loop**：grill → health → postmortem → lessons
3. **Agent-only**：用户不跑 accept、不看网页；仅 BLOCKER 汇报
4. **Shield 轻量版**：`agent_gate_hook` + Cursor Hook，不必上完整 SCALE Engine
5. **Phase1 冻结**：验证期参数写入 lessons，防 Agent 污染样本
6. **展示 vs 执行分离**：门禁关闭时信号仍可展示，执行只听 routing

## 与 SCALE Engine 关系

- 方法论层：本仓库（证据、规则、教训）
-  enforcement 层：可选 SCALE Engine 或轻量 Hook + exit 2
- 猎手实践选择后者，见 `README-trading-hunter.md` §门卫

---

*维护者：agent-methodology · 案例来源：猎手模拟交易 2026-06*
