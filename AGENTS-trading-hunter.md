# AGENTS.md — 猎手交易 / 量化策略（模板）

> 复制到项目根目录，把【】替换为实际值。  
> 配套案例：[`README-trading-hunter.md`](README-trading-hunter.md)

---

## §0 自动化触发表

| 触发 | 动作 | 谁执行 |
|------|------|--------|
| 日频 cron | `run_trading_pipeline.sh` | 机器 |
| Agent 改 WATCHED 内文件 | `strategy_grill.py --pipeline` | Agent |
| grill FAIL / health critical | Agent 自修后再汇报 | Agent |
| postmortem 新规律 | 写 `lessons.md` draft → 样本够再 confirmed | Agent |
| 用户 @ 文件 | 覆盖同路径磁盘默认 | 对话优先 |

**主人约定**：【零运维 / 仅异常汇报 / Phase1 冻结参数列表】

---

## §1 证据优先级

### P0 — 必须先读（当日真相，pipeline 刷新）

| 证据 | 路径 |
|------|------|
| 模拟仓 | `data/portfolio.json` |
| 门禁路由 | `daily/strategy_routing.json` |
| 市场上下文 | `daily/market_context.json` |
| 验证冲刺 | `daily/phase1_validation.json` |
| 用户 @ | 对话显式引用 |

**冲突**：以 `portfolio.json` + `strategy_routing.json` 为准。

### P1 — 改策略/信号/执行时读

`position_config_v4.json` · `candidate_stocks.json` · `unified_signals.json` · `daily/etf_watch.json`

### P2 — 辅助（不替代 P0/P1）

`memory/lessons.md` · `daily/signal_postmortem.json` · `daily/strategy_grill.json` · `daily/health_check.json` · `MEMORY.md`

---

## §2 单一事实来源

| 领域 | 权威路径 | 非权威 |
|------|----------|--------|
| 能否开仓 | `strategy_routing.json` | 主页展示文案 |
| 候选池 | `candidate_stocks.json` | 聊天记录 |
| ETF 观察 | `daily/etf_watch.json` | `candidate_stocks` 内 etfs |
| 策略参数 | `position_config_v4.json` | 口述 |

---

## §3 红线索引

1. **个股 / ETF 分轨** — 禁止混池（见 lessons L-003）
2. **执行只听 routing + signal_filter** — 展示信号不等于可交易（L-004）
3. **Phase1 冻结** — 不改【门禁 / 辩论阈值 / 探针】除非主人明确要求（L-007）
4. **复盘 observe_only** — verified&lt;20 不调 `adaptive_state`（L-005）
5. **受保护文件须 grill** — Hook 拦截直至 PASS（L-008）
6. **禁止改** 【/var/www/、.env、cron、运维配置】
7. **实盘** — 需主人明确指令；模拟仓默认

细则：`.cursor/rules/evidence-priority.mdc` · `.cursor/rules/agent-loops-workflow.mdc`

---

## §4 冲突处理

1. P0 矛盾 → 记 `memory/YYYY-MM-DD.md`，以 portfolio + routing 为准执行
2. 设计 vs 实现歧义 → 列 QUESTION，**禁止**先改门禁/仓位
3. postmortem 样本 &lt;2 → 不出板块结论

---

## §5 开发后收尾（Agent 独占，禁止让人类跑命令）

1. `strategy_grill.py --force` 或 `--pipeline`
2. `health_check.py`
3. `phase1_process_journal` + `memory/YYYY-MM-DD.md` 一行
4. 新教训 → `memory/lessons.md`（`status: confirmed` 后方可引用为红线）
5. grill FAIL → **自修**，不要让主人执行 shell

---

## §6 环境与工具

- Python 3 · 项目根 `tools/`
- 烤机：`tools/strategy_grill.py`（`WATCHED` 列表同仓库）
- 门禁：`tools/agent_gate_hook.py` + `.cursor/hooks.json`
- 部署：【zhuli_deploy.sh 或等价】

---

## §7 扩展入口

| 路径 | 用途 |
|------|------|
| `.cursor/rules/` | evidence-priority · agent-loops-workflow |
| `memory/lessons.md` | 教训库 |
| `daily/phase1_journal.md` | 主人摘要 |
| `examples/trading-hunter/` | 本方法论仓库模板 |

---

## 会话启动 checklist

1. 读 `SOUL.md` / `USER.md`（若有）
2. 读 `memory/今日.md` + `memory/昨日.md`
3. 主会话读 `MEMORY.md`
4. 读 `.cursor/rules/agent-loops-workflow.mdc`
5. 改策略前读 `memory/lessons.md`

---

*模板版本：2026-06-30 · 对齐 [agent-methodology](https://github.com/eninem123/agent-methodology)*
