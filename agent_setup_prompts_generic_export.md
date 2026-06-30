---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/agent_setup_prompts_generic_export-fbbebaa0ad.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779222697
 ReservedCode2: ""
---
# 智能体协作模板包（纯通用 Export）

> **用途**：复制到新项目即可用，不含 StarRocks / SAP / 数仓专有术语。
> 带 企业数据平台 实战案例的版本见 [`agent_setup_prompts_by_scenario.md`](agent_setup_prompts_by_scenario.md)。

---

## 使用方式

1. 将本文 **§A `AGENTS.md` 空白模板** 存为新仓库根目录 `AGENTS.md`，填 `【】`。
2. 按项目类型从 **§B 场景提示词** 选一段，粘贴到 Cursor Agent 对话执行。
3. 日常单任务用 **§C 低 token 短提示词**。
4. 可选：复制 **§D 目录骨架**、**§E 教训模板**、**§F Skill 模板**。

---

## §A `AGENTS.md` 空白模板（复制即用）

```markdown
# 【项目名称】AI 协作规范（索引版）

详细条款按需查看：`docs/ai_guides/on_demand.md`、`.cursor/rules/*.mdc`。

## 0) 自动化触发（人不必手敲）

| 条件 | 自动动作 |
|------|----------|
| Agent 将修改【守卫路径，如 src/**、migrations/**】 | `python tools/pre_change_guard.py --target <模块> --markdown`（Hook/CI 可自动调） |
| Agent 新增/修改 `ai_learning/lessons_learned/**` | Agent 执行【索引命令，如 `python scripts/build_kb.py --lessons-only`】 |
| 用户问【场景，如 review / 选型 / 面试】 | 加载对应 Skill（见 §7） |
| 本地 lint 且有关键文件变更 | 【`make lint` / `npm run lint`】best-effort |

## 1) 变更前流程（证据优先级）

**口径以「当前」为准，不以历史 KB 为准：**

| 优先级 | 来源 | 说明 |
|--------|------|------|
| P0 | **用户 @ 引用** | 对话里显式给的 PRD / 设计稿 / 配置 |
| P0 | **磁盘当日设计产物** | 【如 `docs/design/`、`api/openapi.yaml`】看 mtime |
| P1 | 代码与 schema | 【如 `src/`、`migrations/`、`schema/`】 |
| P2 | 历史 KB / lessons | 仅辅助查 **教训/规范**；**不能**替代 P0 设计稿 |

- **Agent**：改代码前须 **Read 当日设计稿全文**（路径 + mtime）；用户 @ 了文件则以 @ 为准。
- **守卫**：`python tools/pre_change_guard.py --target <模块> --markdown --write-cache`

## 2) 单一事实来源

- 设计：【`docs/design/`、`api/openapi.yaml`、Figma 导出等】
- 实现：【`src/`、`migrations/`】
- 本地 KB：可选，适合 lessons / 规范检索；**不要求**每日全量重建
- Agent 生成的 readme / 草稿文档 **不得**当作业务已确认依据
- 未经业务确认的兜底口径 **不得**自行扩展

## 3) 红线索引（细则链到 on_demand.md）

| 主题 | 要点 | 详情 |
|------|------|------|
| 【主题 1】 | 【一行摘要】 | `docs/ai_guides/on_demand.md` §【】 |
| 【主题 2】 | 【一行摘要】 | 同上 |
| 安全 | 禁止提交 secrets；禁止未授权访问生产 | §【】 |
| 破坏性变更 | migration / 对外 API 变更须人类确认 | §【】 |
| 【按需补充，建议 5–15 条】 | | |

## 4) 需求冲突

- 与 design / schema / 现有实现冲突 → 先列冲突待确认，禁止直接合并
- 需求歧义（范围、边界、默认值）→ 先问用户/业务
- 「空 / 无 / 未填」默认含义：【如 NULL 与空串均算空 / 仅 NULL】

## 5) 开发后

- 同步同目录说明文档（readme / CHANGELOG / ADR，与项目约定一致）
- 关键逻辑做可量化验证（测试、抽样、契约检查）
- 新教训用 `ai_learning/lessons_learned/_TEMPLATE.md`；**confirmed** 后才可写入 §3 红线表
- **教训入库**：Agent 保存后自行执行【`build_kb --lessons-only`】，勿让用户手敲

## 6) 环境与工具

- 测试：【命令、环境变量说明】
- 生产访问：【MCP / CLI；默认 test；prod 须用户明示】
- 禁止：【未授权工具、直连生产写库等】

## 7) 扩展入口

| 用途 | 路径 |
|------|------|
| 变更前守卫 | `tools/pre_change_guard.py` |
| Cursor Hook | `.cursor/hooks.json` |
| 扩展规范 | `.cursor/rules/` |
| 长规范 | `docs/ai_guides/on_demand.md` |
| 项目 Skills | `.cursor/skills/` |
| 教训模板 | `ai_learning/lessons_learned/_TEMPLATE.md` |

## Code Style

- 【语言、格式化、提交信息风格】
- 文档输出不用 emoji
```

---

## §B 分场景搭建提示词

### B0 任何新项目（搭骨架）

```markdown
# 任务：为本仓库搭建「项目专用 AI 智能体」协作体系

你是长期协作 Agent。扫描现有目录，复用约定，补齐缺失层。

## 项目画像

- 项目名称：【】
- 技术栈：【】
- Agent 主职责：【】
- 禁止 Agent 做：【】
- 人类必须确认：【】

## 必须产出

1. `AGENTS.md`（~120 行）：§0 自动化 ~ §7 扩展入口（结构见通用模板）
2. `.cursor/rules/<domain>.mdc`：`alwaysApply: false`，globs 绑定核心路径
3. `docs/ai_guides/on_demand.md`：长规范外链
4. `tools/pre_change_guard.py`：改【守卫路径】前校验设计稿存在、mtime、模块名
5. `ai_learning/lessons_learned/_TEMPLATE.md`：draft | confirmed
6. 至少 1 个 `.cursor/skills/<name>/SKILL.md`（description 写清触发词）

## 行为契约

- 证据：P0 @ / 当日设计 > P1 代码 schema > P2 历史教训
- 冲突：列待确认，禁止先改生产/先合并
- 改后：最小 diff；同步文档；可量化验证
- 自动化：能脚本的步骤 Agent 自行执行

## 验收

- [ ] 新人只读 AGENTS.md 可知证据来源与红线
- [ ] 至少 1 个 guard + 1 个 Skill
- [ ] AGENTS 不堆长文，细则在 on_demand

先列文件清单，再创建。未知业务用 `【TODO: 待确认】`，禁止编造。
```

---

### B1 数据管道 / SQL / ETL（通用，非厂商绑定）

```markdown
# 任务：搭建「数据管道」Agent 体系

技术栈：【PostgreSQL / BigQuery / Spark / 自研调度 …】
代码根目录：【如 pipelines/、sql/、etl/】
设计稿：【如 docs/models/、dbt schema、YAML 模型定义】
数据字典：【如 docs/data_dictionary.csv】

请创建：

1. AGENTS.md：P0 = 用户 @ + 磁盘当日模型设计；P2 KB 仅教训
2. `tools/pre_change_guard.py --entity <表/模型名> --markdown`
3. Hook：触及【`**/sql/**`、`**/models/**`】时自动 guard
4. rules globs：【管道代码路径】
5. 红线示例（按栈裁剪）：
 - 禁止 SELECT *（若团队有此规范）
 - 破坏性 DDL / 全量覆盖须确认
 - 字段含义以设计稿为准，禁止臆造
 - 先查来源类型再类型转换，禁止模板式强转
6. 教训写入后 Agent 跑【build_kb --lessons-only】
```

**日常：单模型开发**

```markdown
任务：开发/修改模型 <名称>（仅此任务）。

范围（禁止全仓搜索）：
- @【sql/模型文件路径】
- @【设计稿路径】

改前：Read 设计稿全文（含 mtime）；或 `pre_change_guard --entity <名称> --markdown`

口径：粒度【】；主键【】；过滤【】

约束：AGENTS §3；冲突先列待确认。

输出：① 三条结论（≤20 字/条）② 改动文件 ③ 待确认项
```

---

### B2 后端 API / 微服务

```markdown
# 任务：搭建「后端服务」Agent 体系

事实来源：
- P0：用户 @ 的 PRD / OpenAPI / 接口设计
- P0：`api/openapi.yaml` 或 `docs/api/`（当日）
- P1：`src/`、`migrations/`、`schema/`
- P2：`ai_learning/lessons_learned/`

产出：
1. AGENTS §3 红线：破坏性 migration 须确认；API 变更同步 OpenAPI + CHANGELOG；禁泄露 secrets/PII；禁 Agent 直连生产写库
2. `pre_change_guard --module <服务名>`：OpenAPI 与 handler 一致性
3. `.cursor/rules/backend-api.mdc` globs: `src/**`、`migrations/**`、`**/*_test.*`
4. Skill `api-review`：触发词 PR、review、breaking change
5. 开发后：`【test 命令】` + 关键接口 curl 示例
```

**日常：单接口**

```markdown
任务：实现【METHOD /path】（仅此接口）。

范围：@【openapi】 @【handler】 @【migrations】

先对齐契约；冲突列清单，不改代码。

约束：幂等【】；错误码对齐 OpenAPI；单测【正常/异常/未授权】。

输出：改动文件 + 待确认 + 一条验证命令
```

---

### B3 前端 / 全栈

```markdown
# 任务：搭建「前端」Agent 体系

事实来源：P0 设计稿/Figma/@标注；P1 `src/components/`、design token；P2 Storybook

红线：禁硬编码色值/间距；新组件对齐目录命名；a11y 基本要求；mock 不当生产契约

rules globs: `src/**/*.{tsx,vue,css,scss}`

Skill `ui-implementation`：触发词 页面、组件、样式、交互
```

**日常：单页面/组件**

```markdown
任务：实现 <名称>，对照 @设计稿 或 @同类组件。

范围：仅 @【页面/组件路径】

输出：组件树（≤5 行）→ 文件列表 → 差异/待确认
```

---

### B4 Code Review

```markdown
# 任务：创建 code-review Skill

路径：.cursor/skills/code-review/SKILL.md

触发词：review、PR、合并前检查

流程：
1. 只读 diff + 用户 @ 文件
2. 对照 AGENTS §3 + rules
3. 分级：BLOCKER / SHOULD / QUESTION
4. 禁止脱离仓库编造规范；readme 不当已确认业务依据

输出表：| 级别 | 文件:行 | 问题 | 依据路径 |
```

**单次 Review**

```markdown
任务：Review 本次变更，不改代码。

范围：branch diff + @【关键文件】

输出：BLOCKER / SHOULD / QUESTION，每类 ≤5 条，每条引依据路径。
```

---

### B5 不确定决策（选型 / 上不上线）

```markdown
# 任务：接入「决策报告」Skill

场景：选型、环境顺序、本期是否上线、A 还是 B

要求：
- 输入：假设、证据分级、先验、成功指标、行动阈值
- 信息不足：弱先验 + 最少追问（≤3）
- 输出：结论在前；数字标 observed / estimated / assumed
- 禁止：假装确定；替代专业医疗/法律/投资建议

触发词：是否应、选型、A还是B、上不上线

可自建 `.cursor/skills/decision-report/SKILL.md` + `references/intake.md`
```

**单次决策**

```markdown
任务：对「【问题】」出决策报告，非头脑风暴。

背景：【】可选行动：A【】B【】已有证据：【路径/无】
时间窗口：【】成功指标：【】

证据不足时：弱先验 + ≤3 追问，勿假装确定。
```

---

### B6 面试 / 答辩 / 知识教练

```markdown
# 任务：搭建 interview-coach Skill

路径：.cursor/skills/<project>-interview/SKILL.md
题库：docs/interview/<project>/bank.md

原则：
1. 答案指向仓库真实路径（AGENTS、src、lessons）
2. 区分规范 vs 业务口径；无证据标明勿编造
3. 结构：结论 → 项目怎么做 → 为什么 → 可追问点

触发词：面试、答辩、STAR、简历深挖
```

**模拟面试**

```markdown
角色：面试官（【方向】）。基于本仓库真实内容，禁止臆造。

流程：1 题 → 等我答 → 点评 → 评分维度。

题库：docs/interview/【项目】/bank.md 第【】章
```

---

### B7 知识库与教训闭环

```markdown
# 任务：搭建教训 → 检索闭环

目录：
- ai_learning/lessons_learned/
- ai_learning/errors/
- 【可选】scripts/build_kb.py 或任意本地检索索引

规则：
1. 模板：现象、根因、禁止项、检测步骤、status draft|confirmed
2. Agent 保存 lessons 后自动【build_kb --lessons-only】
3. KB 不替代 P0 设计稿
4. 仅 confirmed 升格进 AGENTS §3

AGENTS §0 写明自动化行；验收：检索能命中新教训标题
```

**沉淀教训**

```markdown
任务：沉淀踩坑为教训，不改代码。

现象：【】根因：【附证据路径】
结论：禁止/必须各 ≤3 条

按 _TEMPLATE.md 起草 status=draft；保存后你执行 build_kb --lessons-only。
```

---

### B8 MCP / 外部工具

```markdown
# 任务：配置 MCP 工具边界

写入 AGENTS §6：
- 允许：【工具名 + 只读/读写 + test/prod】
- 禁止：【未授权资源、生产写操作】
- 查询：说明目的；默认 LIMIT；敏感列脱敏；结果摘要不回贴全量

默认 test；用户明示才 prod。
```

---

### B9 Hook + 守卫

```markdown
# 任务：为【glob】添加 Cursor Hook

触发：preToolUse / postToolUse on Write|StrReplace|EditNotebook
匹配：【如 **/src/**/*.py】

逻辑：
1. 解析目标路径；不匹配则 exit 0
2. 调 tools/pre_change_guard.py --target <推断> --markdown --write-cache
3. pre：摘要注入 agent_message；post：刷新 .cursor/cache/
4. 超时 120s；失败标注「守卫失败」不静默

更新 AGENTS §0
```

---

### B10 老项目 Agent 化（分阶段）

```markdown
# 任务：现有仓库 Agent 化（不大重构）

阶段 1：AGENTS.md + 1 个 rules + lessons 模板
阶段 2：最高频路径 1 个 pre_change_guard
阶段 3：1 个 Skill + hooks
阶段 4：KB 只索引 lessons + AGENTS

先扫描：已有规范、核心 glob、可脚本化重复步骤。
输出分阶段清单 + 验收标准。不要一次 20 个 Skill。
```

---

## §C 日常低 token 短提示词（通用）

### C1 单任务开发

```markdown
任务：【一件事】。

范围：只 @【文件 1】@【文件 2】；不全仓搜索；不贴 git status。

约束：AGENTS §3；【栈特定 1–2 条】。

输出：① 三条结论（≤20 字）② 改动文件 ③ 待确认（无则「无」）

低 token：总字数尽量 ≤200。
```

### C2 排错

```markdown
任务：定位【现象/报错】。

范围：仅 @【相关文件】

现象：报错【关键 5–20 行】；预期【】；实际【】

要求：Top3 原因 + 每条 1 个最小验证步骤；确认后再改代码。≤200 字。
```

### C3 口径/契约确认（不改代码）

```markdown
任务：只做确认，不改代码。

主题：【】
对照：@【设计稿】@【实现/字典】

输出固定：
1) 定义（≤80 字） 2) 规则/公式（1 行） 3) 边界条件（≤5 条）
4) 冲突清单 5) 待确认（≤3 条）

证据不足写「证据不足 + 缺失项」。
```

### C4 可追加短指令

```markdown
低 token 模式：限定路径；不主动 git status；先结论后细节；≤200 字。
```

---

## §D 推荐目录骨架

```text
your-project/
├── AGENTS.md
├── .cursor/
│ ├── rules/
│ │ └── domain.mdc
│ ├── skills/
│ │ └── code-review/
│ │ └── SKILL.md
│ ├── hooks.json # 可选
│ └── cache/ # guard 输出，可 gitignore
├── docs/
│ ├── design/ # P0 设计稿
│ └── ai_guides/
│ └── on_demand.md
├── tools/
│ └── pre_change_guard.py
├── ai_learning/
│ └── lessons_learned/
│ └── _TEMPLATE.md
├── scripts/
│ └── build_kb.py # 可选
└── src/ # 或 api/、apps/ 等
```

---

## §E 教训模板（通用 `_TEMPLATE.md`）

```markdown
# [简短标题：现象或禁止项]

**日期**: YYYY-MM-DD
**关联模块**: 【服务名 / 模型名 / 「通用」】
**状态**: draft | confirmed（须 PR/业务确认后才标 confirmed）

## 现象

（1–3 句）

## 根因

（附证据：设计稿路径、PR、日志、复现步骤）

## 结论 / 禁止项

- **禁止**：…
- **必须**：…

## 推荐写法

（可选：代码/SQL/配置示例）

## 检测方法（上线前建议）

（测试命令、断言、抽样步骤）

## 备选方案（若曾考虑）

| 方案 | 未采用原因 |
|------|------------|
| A | … |

## 关联文档

- 设计：【路径】
- 相关教训：【路径】
- 升格：confirmed 后可写入 AGENTS §3

## 元数据（可选）

```yaml
type: lesson
modules: [example-service]
tags: [api, migration, security]
confidence: draft
```
```

---

## §F Skill 模板（`SKILL.md`）

```markdown
---
name: 【skill-name】
description: 【一句话 + 何时自动启用，含触发关键词】
---

# 【Skill 名称】

## 何时启用

- 【场景 1】
- 关键词：【】

## 作答原则（强制）

1. **证据优先**：指向仓库路径，禁止臆造
2. **单一事实来源**：设计稿 > 代码 > 历史教训
3. **冲突**：列待确认，禁止先改

## 推荐流程

1. Read 【必要文件】
2. 【步骤 2】
3. 输出：【格式契约】

## 禁止

- 【】

## 关联文档

- `AGENTS.md` §【】
- `docs/ai_guides/on_demand.md`
```

---

## §G `pre_change_guard.py` 行为契约（实现提示）

给 Agent 或开发者实现守卫脚本时的通用要求（语言不限）：

```markdown
# 任务：实现 tools/pre_change_guard.py

CLI：
 --target <模块/实体名> 必填
 --markdown 人类可读摘要（Agent Read）
 --write-cache 写入 .cursor/cache/pre_change_guard.json
 --with-kb 可选，仅查历史教训

输出须含：
- 设计稿路径 + mtime（不存在则 ERROR）
- 与代码/schema 的字段/契约差异摘要
- 相关 lessons 标题（仅 --with-kb）

退出码：0=通过或警告；非 0=缺设计稿等硬错误

禁止：用 KB 内嵌设计替代磁盘当日文件
```

---

## §H 场景速查（无厂商绑定）

| 场景 | 优先建什么 | 设计稿等价物 | guard | Skill |
|------|------------|--------------|-------|-------|
| 数据管道 | AGENTS + guard | 模型 YAML / 设计目录 | pre_change_guard | pipeline-review |
| 后端 | AGENTS + OpenAPI rule | openapi.yaml | pre_change_guard | api-review |
| 前端 | AGENTS + token rule | Figma / 组件库 | — | ui-implementation |
| Review | Skill | AGENTS 红线 | — | code-review |
| 决策 | 决策 Skill | 证据清单 | — | decision-report |
| 教训 | 模板 + KB 脚本 | — | — | — |
| MCP | AGENTS §6 | — | 查询 LIMIT | — |
| 老项目 | 分阶段 AGENTS | 现有文档 | 最高频 1 条 | 1 个 |

---

## §I 维护 checklist

- [ ] AGENTS 保持 ~120 行；红线 >15 条则迁到 `on_demand.md`
- [ ] Skill `description` 含触发词
- [ ] 教训 draft → confirmed → 再进 AGENTS §3
- [ ] §0 自动化表与 Hook/脚本实际行为一致
- [ ] 新项目复制本文 §A + §D 即可开工

---

*Export 版本：2026-06-11 · 纯通用，可整文件复制到新仓库*

---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。