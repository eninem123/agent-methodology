---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/agent_setup_prompts_by_scenario-4961d0f7f2.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779193913
 ReservedCode2: ""
---
# 智能体搭建提示词手册（分场景）

> 基于 企业数据平台 数仓协作实践沉淀。日常「低 token 提问」见 [`low_token_prompt_templates.md`](low_token_prompt_templates.md)；本文聚焦 **新项目/新场景如何搭 Agent 骨架** 与 **可复制提示词**。
> **纯通用、无 StarRocks/SAP 绑定**：整包复制见 [`agent_setup_prompts_generic_export.md`](agent_setup_prompts_generic_export.md)。

---

## 1. 为什么要分层

企业数据平台 跑通后的经验：**不要把所有规则塞进一条 System Prompt**。分层后 token 可控、可维护、可自动化。

```text
AGENTS.md（索引，常驻）
 ├── .cursor/rules/*.mdc（按 glob 按需注入）
 ├── docs/ai_guides/*.md（长规范，命中再 Read）
 ├── .cursor/skills/（轻场景 Skill）
 ├── .agents/skills/（重 Skill：脚本/模板/报告）
 ├── tools/ + .cursor/hooks.json（机器守卫）
 └── ai_learning/lessons_learned/（教训 → KB 闭环）
```

| 层级 | 放什么 | 不放什么 |
|------|--------|----------|
| AGENTS.md | 红线索引、证据优先级、自动化表、入口路径 | 长篇语法细则、完整案例 |
| rules | 开发前/中/后 checklist、globs 绑定 | 与 glob 无关的全局废话 |
| on_demand 文档 | StarRocks 细则、SAP 清洗、Python ETL | 每轮都需要的 3 条红线 |
| Skill | 触发词、流程、禁止项、输出契约 | 整个仓库规范 |
| tools/hooks | 可脚本化校验 | 需要人脑判断的业务口径 |

**三条铁律（企业数据平台 验证有效）**

1. **证据链**：用户 @ / 当日设计稿 > 代码与 schema > 历史 KB（KB 只辅助教训，不替代设计）。
2. **自动化**：能 Hook/脚本的步骤，不写「请用户执行」。
3. **冲突停手**：设计 vs 实现有歧义 → 列待确认项，禁止先落库/先合并。

---

## 2. 通用骨架提示词（任何新项目第一步）

适用：空仓库或「还没有 AGENTS.md」的项目。复制到 Cursor Agent 对话即可。

```markdown
# 任务：为本仓库搭建「项目专用 AI 智能体」协作体系

你是长期协作 Agent。请扫描现有目录，复用已有约定，再补齐缺失层。

## 项目画像（我先填）

- 项目名称：【】
- 技术栈：【】
- Agent 主职责：【写代码 / 评审 / 答疑 / 运维脚本 …】
- 禁止 Agent 做：【直连生产写库、改 secrets、未经确认改对外 API …】
- 人类必须确认：【破坏性迁移、计费口径、安全策略 …】

## 必须产出

1. `AGENTS.md`（~120 行内）含：
 - §0 自动化触发表
 - §1 证据优先级（P0 @/当日设计 > P1 代码 schema > P2 历史教训）
 - §2 单一事实来源（哪些路径是设计依据；哪些 AI 文档不算已确认）
 - §3 红线索引（5–15 条，链到详细文档）
 - §4 冲突处理（列待确认 → 禁止先改）
 - §5 开发后收尾（文档同步、验证、教训沉淀）
 - §6 环境与工具（lint、测试、MCP 授权范围）
 - §7 扩展入口（rules/skills/tools/hooks 路径表）

2. `.cursor/rules/<domain>.mdc`：`alwaysApply: false`，`globs` 绑定核心代码路径

3. `docs/ai_guides/on_demand.md`：承接装不下的长规范

4. `ai_learning/lessons_learned/_TEMPLATE.md`：draft | confirmed 升格机制

## 行为契约（写入 AGENTS）

- 改代码前：Read 守卫/设计稿全文（含路径与 mtime）
- 无证据：标「待确认」，禁止编造字段/枚举/数字
- 改代码后：最小 diff；同步 readme/CHANGELOG；可量化验证
- 新教训：Agent 执行索引命令，不要求用户手敲

## 验收

- [ ] 新人只读 AGENTS.md 可知证据从哪来、红线是什么
- [ ] 至少 1 个可执行 guard 或等价检查
- [ ] 至少 1 个带触发词的 Skill
- [ ] 细则不在 AGENTS 堆长文

先输出骨架说明与文件清单，再逐个创建。业务未知处用 `【TODO: 待业务确认】`，不要编造。
```

---

## 3. 场景 A：数据中台 / 数仓（企业数据平台 模式）

**适用**：ETL、SQL 建模、调度任务、StarRocks/Hive 等。企业数据平台 已落地的参考路径：

| 组件 | 本项目路径 |
|------|------------|
| 总索引 | `AGENTS.md` |
| 按需细则 | `.cursor/rules/warehouse-workflow-extended.mdc` |
| 长规范 | `docs/ai_guides/warehouse_on_demand.md` |
| 改表守卫 | `tools/warehouse_pre_edit_guard.py` |
| Hook | `.cursor/hooks.json` → `warehouse_sql_guard_hook.py` |
| 面试 Skill | `.cursor/skills/warehouse-interview/SKILL.md` |
| 决策 Skill | `.agents/skills/yao-bayesian-skill/` |

### A1. 从零搭数仓 Agent

```markdown
# 任务：搭建「数仓专用」Agent 协作体系（参照 企业数据平台 分层）

技术栈：【StarRocks 3.x / Spark / Airflow …】
代码根目录：【如 model_project/src/prod/】
设计稿目录：【如 docs/model_csv/，须每日手更】
数据字典：【如 dim_data_dictionary_df.csv】

请创建：

1. `AGENTS.md` 数仓版，证据优先级必须是：
 - P0：用户 @ 的 CSV/Excel/SQL
 - P0：磁盘**当日**设计稿（看 mtime，禁止用 KB 里旧 model_csv 开发）
 - P1：数据字典 + 现有 prod SQL
 - P2：lessons_learned / KB（仅教训，非设计依据）

2. `tools/warehouse_pre_edit_guard.py`（或改名）：
 - 输入 `--tables <表名> --markdown --write-cache`
 - 输出：设计稿路径、mtime、字段清单、冲突提示
 - 可选 `--with-kb` 只查历史教训

3. `.cursor/hooks.json`：Write/StrReplace 触及 `src/**/layer/*.sql` 时自动跑 guard

4. `.cursor/rules/warehouse-workflow-extended.mdc`：
 - globs: `**/src/**/*.sql`, `**/readme.md`
 - 开发前/中/后 + 链到 `warehouse_on_demand.md`

5. SQL 红线索引（写入 AGENTS §3，示例）：
 - 禁 SELECT *；SELECT/CASE 禁子查询；无 QUALIFY
 - 全量默认 INSERT OVERWRITE；明细禁 UPDATE
 - 先查来源类型再 CAST/TRIM，禁止 blanket 清洗
 - CSV「直接获取」维度 → GROUP BY 直出，禁滥用 MAX 揉维度

6. `ai_learning/lessons_learned/_TEMPLATE.md` + 约定：
 - Agent 写入教训后自动跑 `build_knowledge_enhanced.py --lessons-only`
 - 仅 confirmed 教训可升格进 AGENTS 红线表

参考 企业数据平台 的 `AGENTS.md` 表格式，不要复制 StarRocks/SAP 专有条款除非我明确需要。
```

### A2. 单表 SQL 开发（日常任务，低 token）

```markdown
任务：为 <目标表名> 开发/修改 SQL（仅此任务）。

范围（禁止全仓搜索）：
- @model_project/src/prod/<layer>/<target>.sql
- @model_project/docs/model_csv/<target>.csv

改代码前：
- 先 Read 上述 model_csv 全文（含 mtime）
- 或执行：python tools/warehouse_pre_edit_guard.py --tables <target> --markdown

已知口径：
- 粒度：【】
- 主键：【】
- 过滤：【】

约束：遵守 AGENTS.md §3 红线；冲突先列待确认，禁止直接实现。

输出：
1) 三条结论（每条 ≤20 字）
2) 改动文件列表
3) 待确认项（无则写「无」）
```

### A3. SAP / 特殊源表画像门禁

```markdown
任务：为引用 ods_sap_erp_* 的模型 <模型名> 补齐空格/类型画像，再写 ETL。

步骤（按序，不可跳过）：
1. python tools/ensure_sap_erp_space_profile.py --from-model <模型名> --prod
2. Read 报告「字段详情」页签，按 T0–T4 分型
3. 再改 @model_project/src/prod/<layer>/<表>.sql

禁止：未跑画像前 blanket TRIM/CAST；两侧 JOIN 键清洗不对称。

输出：字段分型表（列名 | 类型 | 推荐模板 | 是否需 TRIM 证据 SQL）
```

---

## 4. 场景 B：后端 API / 微服务

```markdown
# 任务：搭建「后端服务」Agent 体系

事实来源：
- P0：用户 @ 的 PRD / OpenAPI YAML / 接口设计稿
- P0：`api/openapi.yaml` 或 `docs/api/`（当日版本）
- P1：`src/`、`migrations/`、`schema/`
- P2：`ai_learning/lessons_learned/`

请创建：

1. `AGENTS.md` 后端版 §3 红线示例：
 - 破坏性 migration 须人类确认
 - 对外 API 变更须同步 OpenAPI + CHANGELOG
 - 禁止日志/响应泄露 secrets、PII
 - 数据库写操作默认走 migration，禁止 Agent 直连生产

2. `tools/pre_change_guard.py`：
 - `--module <服务名>` 检查 OpenAPI 是否存在、handler 与路由是否一致
 - `--migration` 检查 pending migration 与 rollback 说明

3. `.cursor/rules/backend-api.mdc`：
 - globs: `src/**/*.go`, `**/*_test.go`, `migrations/**`

4. `.cursor/skills/api-review/SKILL.md`：
 - 触发词：PR、review、接口评审、breaking change
 - 输出：契约一致性、错误码、幂等、鉴权 checklist

5. 开发后：跑 `make test` / `go test ./...`；关键接口给 curl 示例
```

---

## 5. 场景 C：前端 / 全栈

```markdown
# 任务：搭建「前端」Agent 体系

事实来源：
- P0：Figma 链接或 @ 的设计标注 / 组件规范
- P1：`src/components/`、设计 token、`package.json`
- P2：Storybook / 视觉回归基线

AGENTS §3 红线示例：
- 禁止内联硬编码色值/间距，用 design token
- 新组件须对齐现有目录与命名
- 可访问性：表单须有 label、按钮须有 aria
- 禁止把 mock 数据当生产契约

`.cursor/rules/frontend.mdc` globs: `src/**/*.{tsx,vue,css}`

Skill `ui-implementation` 触发词：页面、组件、样式、交互
```

---

## 6. 场景 D：Code Review / 质量门禁

```markdown
# 任务：创建 PR/Code Review Skill

路径：.cursor/skills/code-review/SKILL.md

触发词：review、PR、合并前检查、看看这段代码

流程：
1. 只读 PR diff 与用户 @ 的文件，不全仓扫描
2. 按 AGENTS.md §3 红线逐条对照（无红线则按 rules）
3. 区分：必须改 / 建议改 / 待确认（业务口径）
4. 禁止：脱离仓库编造规范；把 readme 当已确认业务依据

输出格式：
| 级别 | 文件:行 | 问题 | 依据（AGENTS/lesson 路径） |
|------|---------|------|---------------------------|
| BLOCKER | … | … | … |
```

---

## 7. 场景 E：不确定决策（选型 / 上不上线）

```markdown
# 任务：为「技术选型/上线决策」接入决策 Skill

场景示例：先 test 还是先 prod？用 A 库还是 B 库？这期要不要上？

Skill 要求：
- 输入契约：假设、证据分级、先验、成功指标、行动阈值
- 多轮：信息不足时弱先验 + 最少追问
- 输出：中文结论在前 + Markdown/HTML 报告；数字标 observed/estimated/assumed
- 禁止：最终医疗/法律/投资建议；纯贝叶斯作业辅导
```

---

## 8. 场景 F：面试 / 答辩 / 知识教练

```markdown
# 任务：搭建「项目面试教练」Skill

路径：.cursor/skills/<project>-interview/SKILL.md
题库：docs/interview/<project>/interview-bank.md

作答原则（强制）：
1. 证据优先：答案须指向仓库真实路径（AGENTS、src、lessons_learned）
2. 区分「规范」与「业务口径」；无 CSV/SQL 支撑时标明勿编造
3. 结构：结论一句 → 项目怎么做（表名/路径）→ 为什么（规范/教训）→ 可追问点

触发词：面试、八股、答辩、STAR、简历项目深挖
```

---

## 9. 场景 G：知识库与教训闭环

```markdown
# 任务：搭建「教训 → 检索」闭环

目录：
- ai_learning/lessons_learned/
- ai_learning/errors/
- local_knowledge_model/

规则：
1. 教训模板必填：现象、根因、禁止项、检测 SQL/步骤、status: draft|confirmed
2. Agent 保存 lessons_learned 后自动执行：
 python local_knowledge_model/scripts/build_knowledge_enhanced.py --lessons-only
3. 禁止：用全量构建代替 --lessons-only；禁止用 KB 替代当日设计稿
4. 仅 confirmed 可写入 AGENTS §3 红线索引
```

---

## 10. 场景 H：MCP / 外部工具接入

```markdown
# 任务：为 Agent 配置 MCP 工具边界

外部能力：【如：只读 StarRocks、Jira、内部 API】

请写入 AGENTS §6：
- 允许：【工具名 + 只读/读写 + 环境：test/prod】
- 禁止：【未授权的库、写操作、hone_meta 等】
- 查数前：先 EXPLAIN / LIMIT 100 / 脱敏列清单
```

---

## 11. 场景 I：Hook + 守卫自动化

```markdown
# 任务：为【路径 glob】添加 Cursor Hook 自动守卫

触发：preToolUse / postToolUse on Write|StrReplace|EditNotebook

匹配路径：【如 **/src/**/*.sql】

脚本逻辑：
1. 从 tool_input 解析目标文件路径
2. 若非匹配路径则 exit 0
3. 调用 tools/<guard>.py --tables <从文件名推断> --markdown --write-cache
4. preToolUse：将 guard 摘要注入 agent_message
5. postToolUse：刷新 .cursor/cache/<guard>.json

要求：超时 120s；失败不阻断写入但须在 additional_context 标明「守卫失败」
```

---

## 12. 场景 J：老项目「Agent 化」改造

```markdown
# 任务：将现有仓库改造为 Agent 友好（不大重构）

阶段 1（1 天）：只加 AGENTS.md + 1 个 rules + lessons 模板
阶段 2：挑最高频路径加 pre_change_guard
阶段 3：加 1 个 Skill（面试或 review）+ hooks
阶段 4：KB 只索引 lessons_learned + AGENTS

请先扫描仓库列出：
- 已有规范文档（可升为 on_demand）
- 核心代码 glob（可绑 rules）
- 最高频人工重复步骤（可脚本化）

输出：分阶段文件清单 + 每阶段验收标准
```

---

## 13. 场景对照速查

| 场景 | 优先建什么 | 设计稿等价物 | 典型 guard | 典型 Skill |
|------|------------|--------------|------------|------------|
| 数仓 | AGENTS + guard + hook | model_csv | warehouse_pre_edit_guard | warehouse-interview |
| 后端 | AGENTS + OpenAPI rule | openapi.yaml | pre_change_guard | api-review |
| 前端 | AGENTS + token rule | Figma/组件库 | — | ui-implementation |
| Review | Skill 即可 | AGENTS 红线 | — | code-review |
| 决策 | 重 Skill | 证据清单 | — | yao-bayesian-skill |
| 教训 | lessons 模板 + KB | — | — | — |
| MCP | AGENTS §6 授权 | — | 查询 LIMIT | — |
| 老项目改造 | 分阶段 AGENTS | 现有 readme | 最高频 1 条 | 1 个 |
| 量化/交易 | AGENTS + grill + health | pipeline JSON | agent_gate_hook | — |

---

## 15. 场景 K：量化 / 交易策略（猎手模式）

> 案例全文：[`README-trading-hunter.md`](README-trading-hunter.md) · Loops 详解：[`docs/04-agent-loops-zero-ops.md`](docs/04-agent-loops-zero-ops.md)

与数仓场景的核心差异：**P0 证据由日频 pipeline 自动写盘**，用户零运维（不交 model_csv、不点 accept）。

```markdown
# 任务：为【量化/模拟交易】项目搭建 Agent 协作体系

## 项目画像

- 日频/盘中 pipeline 已存在或待建：【run_trading_pipeline.sh】
- 用户约定：零运维，仅 BLOCKER 汇报；Phase1 验证期冻结【门禁/阈值列表】
- 分轨要求：【如个股 vs ETF 不可混池】

## 必须产出

1. `AGENTS.md`（参考 AGENTS-trading-hunter.md）
   - P0：portfolio、routing、market_context、phase1_validation（pipeline 刷新）
   - 改后收尾：grill → health → journal → lessons

2. `.cursor/rules/evidence-priority.mdc` + `agent-loops-workflow.mdc`
   （模板见 examples/trading-hunter/）

3. `tools/strategy_grill.py`
   - WATCHED 关键路径 SHA256
   - --check / --force / --pipeline（通过自动 accept 基线）
   - verdict: PASS | WARN | FAIL

4. `tools/health_check.py`
   - severity: BLOCKER | SHOULD | INFO

5. `tools/signal_postmortem.py`（可选）
   - 分桶命中率；observe_only 直至 verified≥【N】

6. `memory/lessons.md`（draft → confirmed）

7. （推荐）`tools/agent_gate_hook.py` + `.cursor/hooks.json`
   - preToolUse 拦截 WATCHED 路径直至 grill PASS
   - 借鉴 SCALE Shield，不必引入完整 SCALE Engine

## Pipeline 末尾串联

signal_postmortem → strategy_grill --pipeline → health_check --soft
→ phase1_process_journal → deploy

## 验收

- 改 WATCHED 内文件 → Hook 拦截 → grill 通过后解除
- grill FAIL 时 Agent 自修，不向用户列 shell 命令
- 正常日频运行后 P0 JSON mtime 为当日
```

---

## 14. 维护建议

1. **AGENTS 保持索引化**：红线 >15 条时，把细则迁到 `on_demand.md`
2. **Skill description 写清触发词**
3. **教训升格**：draft → 业务/PR 确认 → confirmed → 再写入 AGENTS §3
4. **与低 token 模板配合**：搭骨架用本文；日常开发用 `low_token_prompt_templates.md`

---

*文档版本：2026-06-30 · 随 企业数据平台 + 猎手交易 实践更新*

---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。