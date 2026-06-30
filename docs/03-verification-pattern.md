# 验证模式：从竞赛到工程

> 灵感来源：IMO 2025 Gemini 金牌验证提示词、ICLR APE 竞赛优化、社区多轮反思验证

---

## 核心洞察

竞赛和工程的共同痛点：**AI 生成的答案看起来对，但可能有隐藏缺陷**。

解决方案：**生成（Solver）和核验（Verifier）分离**，强制独立复核。

```
┌─────────────────────────────────────────────────────────┐
│                    验证闭环 Loop                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐            │
│   │  生成   │───→│ 独立核验 │───→│ 修正复核 │            │
│   │ (Solver)│    │(Verifier)│   │ (Fix)   │            │
│   └─────────┘    └─────────┘    └─────────┘            │
│        ↑                              │                 │
│        └──────────────────────────────┘                 │
│              不通过则循环                                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 从竞赛提示词提取的 5 个核心模式

### 模式 1：角色分离（Solver / Verifier）

**竞赛原文**：
> You must act as a verifier, NOT a solver. Do NOT attempt to correct the errors.

**工程迁移**：
- AI 生成代码 → 门卫脚本独立校验（不依赖 AI 自己的判断）
- Code Review → Reviewer 不改代码，只列问题

**在 agent-methodology 中的位置**：
- AGENTS §0：门卫自动触发
- tools/warehouse_pre_edit_guard.py：独立 Verifier
- .cursor/skills/code-review：Reviewer 角色

### 模式 2：分类标注（致命错误 vs 论证缺失）

**竞赛原文**：
> Critical Error: Logical fallacy, factual arithmetic mistake
> Justification Gap: Conclusion may be plausible but lacks rigorous proof

**工程迁移**：
- **BLOCKER**（致命错误）：必须改，不改不能合并
- **SHOULD**（论证缺失）：建议改，可合并但有风险
- **QUESTION**（待确认）：需要业务确认

**在 agent-methodology 中的位置**：
- .cursor/skills/code-review/SKILL.md 输出格式

### 模式 3：逐步核验（Step-by-step）

**竞赛原文**：
> You must perform a step-by-step check of the entire reasoning.

**工程迁移**：
- SQL 开发：逐字段核对 model_csv
- 门卫脚本：逐表检查设计稿 mtime + 字段清单

**在 agent-methodology 中的位置**：
- 门卫脚本输出格式
- 单表开发提示词（low_token_prompt_templates.md）

### 模式 4：不修改只审计（Audit-only）

**竞赛原文**：
> Do not add new solution work, only audit and report flaws.

**工程迁移**：
- 门卫只报错，不自动修代码
- Code Review 只列问题，不改代码
- 修正由 Solver（AI 或人）执行

**在 agent-methodology 中的位置**：
- 门卫脚本设计原则
- Review Skill 设计原则

### 模式 5：交叉验证（Cross-check）

**竞赛原文**：
> Perform a final cross-check: test with small examples/alternative calculation methods

**工程迁移**：
- SQL 结果：行数/金额/明细抽样核对
- 关键报表：多源交叉验证

**在 agent-methodology 中的位置**：
- warehouse_on_demand.md §5 收尾与质量
- AGENTS §5 开发后

---

## 整合方案：分层验证体系

### 第 1 层：AGENTS.md 红线（预防）

```
改代码前必须读设计稿 → 避免「无证据开发」
```

### 第 2 层：门卫脚本（自动核验）

```
改代码前自动跑 guard → 校验设计稿/mtime/字段
```

### 第 3 层：Code Review（独立审计）

```
PR 提交后 → Reviewer 独立核验 → 分类标注问题
```

### 第 4 层：教训闭环（经验沉淀）

```
踩坑 → 写教训 → 确认 → 写入红线 → 下次自动避开
```

---

## 对应关系

| 竞赛模式 | agent-methodology 实现 | 文件位置 |
|----------|------------------------|----------|
| Solver/Verifier 分离 | AI 生成 + 门卫校验 | AGENTS §0, tools/ |
| 分类标注 | BLOCKER/SHOULD/QUESTION | .cursor/skills/code-review/ |
| 逐步核验 | 逐字段检查 model_csv | tools/warehouse_pre_edit_guard.py |
| 不修改只审计 | 门卫只报错不修代码 | tools/ 设计原则 |
| 交叉验证 | 行数/金额/明细抽样 | warehouse_on_demand.md §5 |

---

*版本：2026-06-30 · 竞赛提示词到工程方法论的迁移*