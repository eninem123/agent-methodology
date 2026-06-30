---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/README-a170648ec3.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779328131
 ReservedCode2: ""
---
# 企业数据平台

这是一个企业数据平台，包含数据模型、数据管道和各种工具。

**工作交接**：见 [项目移交说明_企业数据平台.md](项目移交说明_企业数据平台.md)（目录清单、AI 协作上手、验收勾选表）。

## 项目文件结构

### 核心配置文件
- `.clinerules` - AI 辅助开发与核心规范
- `AGENTS.md` - 数据平台 AI 交互上下文整合版
- `load_project_context.py` - AI 上下文加载脚本
- `requirements.txt` - 项目依赖清单
- `requirements-dev.txt` - Ruff / SqlFluff / sqlglot 等开发检查依赖
- `.gitlab-ci.yml` - GitLab CI（lint 阶段）
- `scripts/ci/lint.sh` / `scripts/ci/lint.ps1` - 本地与 CI 共用的检查入口（Windows 用 `.ps1`）
- `.cursor/rules/` - Cursor 分场景规则（SQL / Python / 流程）
- `README.md` - 项目说明文档

### 数仓模型核心
- `model_project/docs/` - 设计文档：模型设计清单-技术开发.xlsx、dim_data_dictionary_df.csv等
- `model_project/src/` - SQL 源码，按以下分层存储：
 - `ods/` - 贴源层 SQL 脚本
 - `dwd/` - 明细层 SQL 脚本
 - `dws/` - 轻度汇总层 SQL 脚本
 - `dim/` - 维度层 SQL 脚本
 - `dm/` - 数据集市层 SQL 脚本
 - `ddl/` - 数据定义语言脚本
 - `dml/` - 数据操纵语言脚本
 - `sync/` - 数据同步配置文件
 - `数据分发/` - 数据分发脚本
 - `test/` - 测试脚本
 - `demo/` - 演示脚本
 - `get/` - 数据获取脚本
 - `index_choose/` - 索引选择脚本
 - `sql/` - 通用 SQL 脚本

更多说明见 `AGENTS.md` **第九节**（本地检查、CI、凭证、血缘）。

### 工具与脚本
- `tools/` - 运维/校验脚本，包含：
 - 前置检查工具：`pre_dev_check.py`
 - Excel 解析工具：`excel_reader.py`
 - 数据库操作工具：`hdap_sql.py`
 - 表结构检查工具：`check_primary_keys.py`
 - 缓存读取工具：`read_cache.py`
 - 数据验证工具：`verify_data.py`
 - SQL 生成工具：`generate_dws_code.py`
 - 分区管理工具：`generate_partitions_dm_sd_weekly.py`
 - SAP 字段分析工具：`sap_erp_field_analyzer.py`
 - 其他各种校验和调试工具
- `trae_skills/` - 设计校验、知识同步、SQL 修复工具

### 知识库系统
- `local_knowledge_model/` - 本地语义检索系统（TF-IDF 驱动），包含知识库构建和查询脚本
- 知识库统一入口文档：`docs/knowledge_base/README.md`
- 维护流程文档：`docs/ai_guides/knowledge_base_workflow.md`

### AI 学习系统
- `ai_learning/` - 错误案例库、分层开发规范、经验教训

### 输出目录
- `audits/` - 审计/验证结果（*.json，按模块/日期分目录）
- `config/` - 配置文件（*.json）
- `queries/` - 临时 SQL 查询脚本

### 业务文档与项目管理
- `业务文档/` - 业务需求文档
- `业务需求/` - 业务需求相关文档
- `项目管理/` - 项目管理相关文档
- `帆软报表自动化项目/` - 帆软报表自动化相关文档

## 上层AI应用 (AI Applications)

### 知识库问答MVP (kb_qa_mvp)

这是一个基于RAG（检索增强生成）技术的智能问答系统MVP，旨在通过整合企业内部的结构化数据（如CRM客户信息、SAP订单数据），结合大语言模型（LLM），为用户提供一个智能、高效的自然语言问答界面。该系统包含FastAPI后端API和Streamlit网页界面。

新增能力概览：

- 多格式文档知识库学习：PDF/Word/Excel/PPT/图片/Markdown/Txt
- OCR：图片与扫描 PDF 入库（本地 tesseract）
- Excel 深度解析：按 Sheet 转 Markdown 表格，保留行列关系
- PPT 备注提取：Slide Notes 作为重要知识来源

**详细信息请参阅：** [ai_applications/kb_qa_mvp/README.md](ai_applications/kb_qa_mvp/README.md)

---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。