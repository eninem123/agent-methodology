# Agent Loops 与零运维模式

> 源自猎手模拟交易实践 · 补充经典「用户按模板交证据」模式  
> 灵感：[Boris Cherny — Write loops, not code in the IDE](https://x.com/bcherny) · [agent-methodology](../README.md) 验证闭环

---

## 问题：两种失败模式

### 模式 A — 堆 IDE 改代码

```
用户：帮我把银行权重调高一点
Agent：改 signal_filter.py … 改 routing … 再改一层 …
（无单测、无复盘、下轮会话又改回去）
```

### 模式 B — 用户当运维

```
Agent：请运行 strategy_grill.py --accept
用户：忘了 / 不想点 / accept 了错误基线
```

**Loops 模式**同时解决两者：**可重复脚本闭环** + **证据与验收机器产出**。

---

## 核心公式

```
Loop = 触发器 + 独立 Verifier + 结构化产出 + 严重级别

零运维 = P0 证据由 pipeline 写盘 + Agent 独占收尾 + 仅 BLOCKER 打扰用户
```

---

## 四层 Loop 栈（猎手交易实例）

```
┌────────────────────────────────────────────────────────────┐
│  L4 教训 Loop    postmortem/grill FAIL → lessons.md        │
├────────────────────────────────────────────────────────────┤
│  L3 复盘 Loop    signal_postmortem → 分桶命中率（只读）     │
├────────────────────────────────────────────────────────────┤
│  L2 健康 Loop    health_check → 24 项集成 BLOCKER/SHOULD   │
├────────────────────────────────────────────────────────────┤
│  L1 烤机 Loop    strategy_grill → 指纹 + 单测 + 基线      │
└────────────────────────────────────────────────────────────┘
         ↑                                    │
         └──────── grill FAIL 回到 Agent 修 ──┘
```

### L1 烤机（strategy_grill）

- **输入**：`WATCHED` 路径 SHA256 vs 基线
- **Verifier**：单元测试 + 静态风险扫描 + Phase1 影响摘要
- **产出**：`daily/strategy_grill.json`（`verdict`: PASS/WARN/FAIL）
- **基线**：pipeline 通过时自动 `accept`（用户不点）

### L2 健康（health_check）

- **输入**：关键 JSON 存在性、schema、交叉引用
- **产出**：`severity`: BLOCKER / SHOULD / INFO

### L3 复盘（signal_postmortem）

- **输入**：归档候选 + N 日行情验证
- **产出**：按板块/来源/门禁分桶；`observe_only` 直至样本够
- **原则**：**只读**，不自动改权重（防过拟合单周）

### L4 教训（lessons）

- **流程**：draft → 样本/业务确认 → `status: confirmed` → AGENTS §3 引用
- **来源**：postmortem、grill FAIL、Phase1 未达标

---

## Pipeline 串联（日频）

```bash
# 概念顺序（实际以 run_trading_pipeline.sh 为准）
交易逻辑 … → candidate_merge → signal_postmortem
    → strategy_grill --pipeline
    → health_check --soft
    → phase1_process_journal
    → deploy
```

**Agent 改代码后**应跑同一链条（或等 cron），禁止让人类「帮忙 accept」。

---

## 零运维契约

| 用户不做 | 机器/Agent 做 |
|----------|----------------|
| 跑 grill/health | pipeline 末尾或 Agent 改后自动跑 |
| 看 postmortem 网页 | Agent 读 JSON；摘要写 journal |
| 点 accept 基线 | `--pipeline` 通过自动刷新 |
| 日常听汇报 | 仅 BLOCKER 简要说明 |

**汇报原则**：

- 正常：沉默或一句「pipeline 正常」
- 异常：什么坏了 + Agent 已/将如何处理
- **禁止**向用户列出要执行的 shell 命令

---

## Cursor Hook：Shield 轻量版

完整 [SCALE Engine](https://gitee.com/hongmaple/scale-engine) 过重时，可只借 **Shield** 思路：

| 组件 | 作用 |
|------|------|
| `preToolUse` | 编辑受保护路径前检查 grill / needs_grill |
| `afterFileEdit` | 标记 dirty，强制下次编辑前先烤机 |
| `exit 2` | 阻断工具调用（不依赖模型自律） |

详见 `examples/trading-hunter/hooks.json.example`。

---

## 与证据优先级的关系

经典方法论：

```
P0 = 用户 @ / 当日设计稿（mtime）
```

零运维扩展：

```
P0 = pipeline 当日 JSON + 用户 @（@ 优先）
P2 = grill/health/postmortem（Verifier 产出，辅助决策）
```

**不矛盾**：设计稿等价物从「人放的 CSV」变成「机器写的 routing/portfolio」。

---

## 何时用本模式

| 适合 | 不适合 |
|------|--------|
| 日频/周频自动任务 | 一次性脚本 |
| 有单测或集成检查 | 纯原型无验收 |
| 主人明确不想运维 | 需要人工审批每步 |
| 长周期验证（Phase1） | 频繁口头改需求 |

---

## 搭建清单（1 天 MVP）

- [ ] `AGENTS.md` 写零运维契约 + P0 表
- [ ] `tools/strategy_grill.py`：`WATCHED` + 至少 1 个单测
- [ ] `tools/health_check.py`：5–10 项 BLOCKER
- [ ] `memory/lessons.md` 模板
- [ ] pipeline 末尾挂 grill + health
- [ ] （可选）`agent_gate_hook.py` + `.cursor/hooks.json`

---

## 对照表

| 概念 | 数仓 | 猎手交易 |
|------|------|----------|
| P0 | model_csv mtime | portfolio + routing JSON |
| Verifier | warehouse_pre_edit_guard | strategy_grill + health_check |
| 复盘 | 行数金额抽样 | signal_postmortem |
| Hook | 注入 guard 摘要 | deny 直至 grill PASS |
| 用户 | 交设计稿 | 零运维 |

---

*版本：2026-06-30*
