---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/SKILL-3a7cf4476a.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779285769
 ReservedCode2: ""
---
# 企业数据平台 Skills 总览

> 统一说明：在本项目中，"Skill" 指 **可以被 AI 或开发者复用的一段能力**，可以是：
> - Trae Skill（`trae_skills/` 目录下的脚本）
> - 工具脚本（`tools/` 目录下的 Python 脚本）
> - Agent Tool（LangChain `@tool` 函数）
> - RAG / SQL 模型等能力

---

## 1. Skill 类型说明

| 类型 | 说明 |
|------------|------|
| `trae_skill` | 位于 `trae_skills/`，基于 `BaseSkill` 的脚本，多用于设计校验、知识同步、自动修复建议等。 |
| `tool_script` | 位于 `tools/`，通用 Python 工具脚本，如预开发检查、Excel 解析、SQL 校验等。 |
| `agent_tool` | 位于 `ai_applications/` 下，通过 LangChain `@tool` 暴露给 Agent 的函数，在对话中被调用。 |
| `rag_service` | RAG 问答/归因分析服务，组合向量库 + LLM 的复杂技能。 |
| `sql_model` | 位于 `model_project/src/` 的 SQL 模型脚本，本身不是 Python Skill，但常作为数据能力底座。 |

---

## 2. Trae Skills 一览（`trae_skills/`）

| 技能名 | 类型 | 入口位置 | 主要作用 | 典型调用方式 |
|--------|------|----------|----------|--------------|
| 设计错误检测 | `trae_skill` | `trae_skills/skill_detect_errors.py` | 扫描模型设计、字段异常、版本不一致等问题，为 SQL/模型开发提供错误清单。 | `python trae_skills/skill_detect_errors.py --verbose` |
| 设计更新检查 | `trae_skill` | `trae_skills/skill_check_update.py` | 检查设计文档与现有实现是否有更新差异，提示需要同步的内容。 | `python trae_skills/skill_check_update.py` |
| 自动修复建议生成 | `trae_skill` | `trae_skills/skill_generate_fix.py` | 基于错误信息生成修复建议（如 SQL 修改方向、字段映射建议）。 | `python trae_skills/skill_generate_fix.py --input errors.json` |
| 知识同步 Skill | `trae_skill` | `trae_skills/skill_sync_knowledge.py` | 读取最新 Excel/CSV/Markdown，增量更新本地知识库。 | `python trae_skills/skill_sync_knowledge.py` |

---

## 3. 工具脚本 Skills（`tools/`）

| 技能名 | 类型 | 入口位置 | 主要作用 | 典型调用方式 |
|--------|------|----------|----------|--------------|
| 开发前置检查 | `tool_script` | `tools/pre_dev_check.py` | 开发/更新任意模型前加载历史错误、检查清单和经验教训，降低错误率。 | `python tools/pre_dev_check.py dws_sd_order_performance_df --update` |
| 设计文档解析 | `tool_script` | `tools/parse_design_document.py` | 全量解析《模型设计清单-技术开发.xlsx》，生成可用的字段/表结构信息。 | `python tools/parse_design_document.py` |
| DWS 代码生成 | `tool_script` | `tools/generate_dws_code.py` | 根据设计表或模板自动生成 DWS SQL 脚本。 | `python tools/generate_dws_code.py --model dws_sd_order_performance_df` |
| DWS 验证工具 | `tool_script` | `tools/verify_dws.py` | 对 DWS 表执行数据质量、主键、汇总结果的验证。 | `python tools/verify_dws.py --table dws_sd_order_performance_df` |
| Excel 读取工具 | `tool_script` | `tools/excel_reader.py` | 统一从 Excel 读取设计/样例数据并做基础校验。 | `python tools/excel_reader.py --file model_project/docs/模型设计清单-技术开发.xlsx` |
| StarRocks SQL 工具 | `tool_script` | `tools/hdap_sql.py` | 对 StarRocks 执行查询/写入，封装连接与安全规则。 | `python tools/hdap_sql.py --db dws --sql "SELECT 1"` |
| 表结构查看 | `tool_script` | `tools/show_table_ddl.py` | 拉取并展示 StarRocks 目标表的 DDL，辅助对比设计与实现。 | `python tools/show_table_ddl.py --db dws --table dws_sd_order_performance_df` |
| 字段验证 | `tool_script` | `tools/validate_table_fields.py` | 检查表字段是否与设计文档、数据字典一致。 | `python tools/validate_table_fields.py --db dws --table ...` |
| 主键检查 | `tool_script` | `tools/check_primary_keys.py` | 检查表主键定义与数据重复情况，辅助 PK 设计。 | `python tools/check_primary_keys.py --db dwd --table dwd_sd_order_detail_df` |
| 本地缓存读取 | `tool_script` | `tools/read_cache.py` | 从 `sr_cache/orders_analysis.db` 中读取缓存数据，避免频繁访问远程库。 | `python tools/read_cache.py --table ods_ods_sap_erp_vbrp_df` |
| 状态映射覆盖率检查 | `tool_script` | `tools/check_mes_mapping.py` | 检查状态映射配置与值域覆盖率，输出缺失值清单。 | `python tools/check_mes_mapping.py --system-code SAP-ERP --source-table ods_sap_erp_usr02_df --state-column UFLAG` |

---

## 4. Agent Tools（在线对话技能）

> 位置：`ai_applications/kb_qa_mvp/app/services/agent_tools.py`

| 技能名 | 类型 | 入口位置 | 主要作用 | 典型触发方式 |
|--------|------|----------|----------|--------------|
| 订单状态查询 | `agent_tool` | `agent_tools.get_order_status` | 从模拟 SAP 订单数据中查询指定订单状态。 | 自然语言："帮我查一下订单 ORD-9527 的状态" |
| 客户信息查询 | `agent_tool` | `agent_tools.find_customer_info` | 从模拟 CRM 数据中按名称关键字查询客户信息。 | 自然语言："查一下深圳大华的客户信息" |

---

## 5. RAG / 数据问答类 Skills

> 位置：`ai_applications/kb_qa_mvp/app/services/rag_service.py`

| 技能名 | 类型 | 入口位置 | 主要作用 | 数据来源 |
|--------|------|----------|----------|----------|
| 多源 RAG 问答 | `rag_service` | `RAGService._handle_rag_query` | 基于 FAISS 向量库做检索问答。 | `ai_applications/kb_qa_mvp/vector_store/faiss_index` |
| 经营归因分析 | `rag_service` | `RAGService._handle_attribution_analysis` | 读取模拟经营数据，进行结构化归因分析。 | `ai_applications/kb_qa_mvp/data/attribution_data.json` |
| SQLite 样例查询 | `rag_service + db_service` | `RAGService._handle_sql_query` | 用 LLM 生成 SQL 并查询，转成自然语言回答。 | `sr_cache/orders_analysis.db` |

---

## 6. SQL 模型能力（`model_project/src/`）

| 能力名 | 类型 | 入口位置 | 主要作用 |
|--------|------|----------|----------|
| 接单业绩模型 | `sql_model` | `model_project/src/dws/dws_sd_order_performance_df.sql` | 汇总订单维度的接单业绩结果。 |
| 接单业绩分配模型 | `sql_model` | `model_project/src/dws/dws_sd_order_performance_alloc_df.sql` | 按业务员/销售组织进行分配，支撑业绩考核。 |
| 验收+业绩分配模型 | `sql_model` | `model_project/src/dws/dws_fin_acceptance_with_perf_alloc_df.sql` | 将验收明细与业绩分配关联，用于验收维度的经营分析。 |
| 其他 DWD/DIM/ODS 模型 | `sql_model` | `model_project/src/*/*.sql` | 提供维度、明细和 ODS 层的数据基础。 |

---

## 7. 命名与扩展约定

1. **新增 Trae Skill** → `trae_skills/skill_xxx.py`，继承 `BaseSkill`
2. **新增工具脚本** → `tools/` 目录，命名如 `check_xxx.py`, `sync_xxx.py`
3. **新增 Agent Tool** → `ai_applications/.../agent_tools.py` 中新增 `@tool` 函数
4. **Skill 文档维护** → 新增/改造技能后，顺手在本文件补一行

---

> 本文档是 企业数据平台 中各类"技能 / 工具"的统一索引，作为 AI 上下文的重要组成部分。

---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。