---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/AGENTS-d5d8611073.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779292666
 ReservedCode2: ""
---
# 数据中台 AI 协作规范（索引版 / 低 token）

详细条款按需查看：`docs/ai_guides/warehouse_on_demand.md`、`.cursor/rules/warehouse-workflow-extended.mdc`。

## 0) 自动化触发（人不必手敲）

| 条件 | 自动动作 |
|------|----------|
| Agent 将写入 `model_project/src/**` 下 `*.sql` | `.cursor/hooks.json` → `warehouse_pre_edit_guard`（见 `pre_edit_guard.json`） |
| 本地 `scripts/ci/lint.*` 且有未暂存 SQL 变更 | `scripts/ci/lint.ps1` / `lint.sh`（best-effort） |
| 用户问面试/答辩 | Skill `warehouse-interview` + `docs/interview/warehouse_interview/` |
| 用户问「是否应/选型/先 test 还是先 prod」 | `.agents/skills/yao-bayesian-skill`（须带证据） |
| Agent 新增/修改 `ai_learning/lessons_learned/**` 或 `errors/**` | Agent 自行执行 `python local_knowledge_model/scripts/build_knowledge_enhanced.py --lessons-only`（**勿**要求用户手敲；禁止默认全量/`--skip-sql` 代替） |

## 1) 改表前流程（证据优先级）

**口径以「当前」为准，不以历史 KB 为准：**

| 优先级 | 来源 | 说明 |
|--------|------|------|
| P0 | **用户 @ 引用** | 对话里显式给的 CSV/Excel/SQL |
| P0 | **磁盘当日 model_csv** | 你每日手动下载后放入 `model_project/docs/model_csv/` |
| P1 | 数据字典 + 当前 SQL/DDL | `dim_data_dictionary_df.csv`、`model_project/src/prod/...` |
| P2 | 历史 KB 索引 | 仅辅助查 **教训/规范**；**不能**替代当日 model_csv |

新模型通常流程：**先更新 model_csv（下载）→ 再写脚本**。守卫默认只扫磁盘 CSV（`warehouse_pre_edit_guard`），默认**不**用 KB 里的 model_csv 块。

```text
python tools/warehouse_pre_edit_guard.py --tables <表名> --markdown --write-cache
# 仅想查历史教训时加：--with-kb
```

- **Agent**：改 SQL 前须 **Read 当日 model_csv 全文**（守卫输出的路径 + mtime）；用户 @ 了设计稿则以 @ 为准。
- **CI/Hook**：自动跑守卫（定位 CSV 路径）；不代替你打开文件核对。
- SAP `ods_sap_erp_*`：`ensure_sap_erp_space_profile.py --from-model <模型> --prod`

## 2) 单一事实来源

- 设计：**当日** `model_project/docs/model_csv/`（手动更新）+ 《模型设计清单-技术开发.xlsx》
- 字段：`model_project/docs/dim_data_dictionary_df.csv` 与源表结构
- 本地 KB（`build_knowledge_enhanced.py`）：可选，适合 **lessons_learned / 规范** 检索；**不要求**每日为 model_csv 全量重建索引
- `readme.md` 仅辅证；AI 生成文档**不得**当作业务已确认依据
- 兜底/默认补齐口径：未经业务确认不得扩展（见 `warehouse_on_demand.md` §5）

## 3) SQL 红线索引（细则见链接）

| 主题 | 要点 | 详情 |
|------|------|------|
| TRIM/CAST | 先查来源类型；禁止 blanket TRIM/成片 VARCHAR CAST | `warehouse_on_demand.md` §1、§6；`20260515_新开发禁止blanket_CAST与TRIM_…md` |
| 语法 | 禁 `SELECT *`；`SELECT/CASE` 禁子查询；无 `QUALIFY` | `warehouse-workflow-extended.mdc` |
| NULL 相等 | 无 `IS NOT DISTINCT FROM`；用 `(a=b OR (a IS NULL AND b IS NULL))` | 同上 |
| 写入 | 全量默认 `INSERT OVERWRITE`；明细表禁 DML `UPDATE` | `StarRocks_3.3.19_开发规范.md` |
| DDL COMMENT | 列/表注释**单引号** | `20260515_StarRocks_DDL_COMMENT须单引号.md` |
| 汇总维度 | CSV「直接获取」→ `GROUP BY` 直出；禁滥用 `MAX`/`MIN` 揉维度 | `20260519_DM汇总禁止滥用MAX带出维度.md` |

## 4) 需求冲突

- 与 design/字典/源表冲突 → 先列冲突待确认，禁止落库
- 清单歧义（粒度、兜底、输出范围）→ 先问用户/业务
- 「空/无/未填」默认含 `NULL` 与 `''`

## 5) 开发后

- 同步同目录 `readme.md`；关键结果行数/金额/明细抽样
- 新增教训用 `ai_learning/lessons_learned/_TEMPLATE.md`；**confirmed** 后才可写入本文件红线表
- **教训入库后（Agent 做，用户不必）**：写入 `lessons_learned`/`errors` 后由 Agent 跑 `--lessons-only`；Excel/model_csv 大变更才用 `--incremental`（细则见 `docs/knowledge_base/README.md`）

## 6) 环境与查询

- 生产查数：gpustack MCP（`platform_tools.gpustack_safe_query`），禁未授权的 `starrocks*`/`hone_meta*`/`bi_mysql*`
- 用户指定 prod 时只改 `model_project/src/prod/`
- 调度参数：`${g_biz_date}` 及派生占位符 → `tools/hone_parm.md`

## 7) 扩展入口

| 用途 | 路径 |
|------|------|
| 分场景搭 Agent 提示词 | `docs/ai_guides/agent_setup_prompts_by_scenario.md` |
| 通用 Export 模板包 | `docs/ai_guides/agent_setup_prompts_generic_export.md` |
| 变更前守卫 | `tools/warehouse_pre_edit_guard.py` |
| Cursor Hook | `.cursor/hooks.json` |
| 面试题库 | `docs/interview/warehouse_interview/interview-bank.md` |
| Yao Skills | `.agents/skills/` |
| 扩展规范 | `.cursor/rules/warehouse-workflow-extended.mdc` |
| 知识库 | `docs/knowledge_base/README.md` |
| 教训模板 | `ai_learning/lessons_learned/_TEMPLATE.md` |
| StarRocks | `aianswer/2025-12-19/StarRocks_3.3.19_开发规范.md` |

## Code Style

- 文档与输出不用 emoji；可用 →、⚠ 等符号
- CLI 彩色输出见 `cli/src/color.rs`，遵守 `NO_COLOR`

---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。