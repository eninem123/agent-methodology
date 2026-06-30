# 企业数据平台 能力对应表

## 核心架构（6 层）

| 层级 | 新技术 | 企业数据平台 实现 | 说明 |
|------|--------|----------------|------|
| 索引层 | 分层规则 | `AGENTS.md` + `.cursor/rules/*.mdc` | 红线索引、证据优先级、自动化表 |
| 守卫层 | 结构化门禁 | `tools/warehouse_pre_edit_guard.py` + `.cursor/hooks.json` | 改表前校验设计稿、mtime、字段冲突 |
| 知识层 | TF-IDF 教训库 | `local_knowledge_model/` + `ai_learning/lessons_learned/` | 本地语义检索，教训沉淀 |
| 对话层 | MCP 查数 | gpustack MCP（生产查数） | 禁止未授权直连 |
| 技能层 | 场景 Skill | `.cursor/skills/` + `.agents/skills/` | 面试、review、决策等 |
| 决策层 | Bayesian Skill | `yao-bayesian-skill/` | 不确定决策报告 |

## 关键映射表

| 技术点 | 企业数据平台 路径 | 猎手交易 路径 | 优先级 |
|--------|----------------|--------------|--------|
| MCP 工具 | gpustack MCP | —（按需行情 API） | P0 - 生产查数 |
| 分层规则 | AGENTS.md | AGENTS.md + rules/*.mdc | P0 - 索引层核心 |
| 结构化门禁 | warehouse_pre_edit_guard.py | strategy_grill + agent_gate_hook | P0 - 改关键文件前 |
| Cursor Hook | .cursor/hooks.json | .cursor/hooks.json | P0 - 自动触发 |
| 集成验收 | — | health_check.py | P0 - BLOCKER 分级 |
| 业务复盘 | SQL 抽样 | signal_postmortem.py | P1 - 只读观察 |
| TF-IDF 知识库 | local_knowledge_model/ | — | P1 - 教训检索 |
| 教训库 | ai_learning/lessons_learned/ | memory/lessons.md | P1 - 闭环 |
| 场景 Skill | .cursor/skills/ | — | P1 - 能力扩展 |
| Bayesian 决策 | yao-bayesian-skill/ | — | P2 - 不确定决策 |

## 自动化闭环

```
数仓：改代码 → Hook → guard 校验 → 教训 → KB

交易：改代码 → Hook(agent_gate) → grill + health → postmortem → lessons
      日频 pipeline 写 P0 JSON → Agent 只读磁盘
```

## 数据流图

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   设计稿     │    │   守卫脚本   │    │   门卫规则   │
│  (model_csv) │───→│  (guard.py)  │───→│  (AGENTS §3) │
└──────────────┘    └──────────────┘    └──────────────┘
        ↓                   ↓                   ↓
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  当日版本    │    │   校验结果   │    │   红线拦截   │
│  (mtime)     │    │  (pass/fail) │    │  (BLOCKER)   │
└──────────────┘    └──────────────┘    └──────────────┘
```

## 价值量化

| 指标 | 旧方案（全 RAG） | 新方案（分场景） | 提升 |
|------|------------------|------------------|------|
| 幻觉率 | 高 | 低 | ↓ 80%+ |
| Token 消耗 | 高 | 低 | ↓ 60%+ |
| 响应时间 | 慢 | 快 | ↑ 3-5x |
| 可维护性 | 低 | 高 | ↑ 显著 |

---

*版本：2026-06-30 · 源自 企业数据平台 实践*