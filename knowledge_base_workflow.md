---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/knowledge_base_workflow-50470f120b.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779242902
 ReservedCode2: ""
---
# 数据中台知识库：主链路与更新闭环

## 1) 目标与边界

- 目标：把"规范/设计/字典/口径/复盘"等知识做成可检索、可复用、可追溯的资产。
- 边界：知识库只解决"找得到/用得上/可核对"，不替代业务确认与生产抽样验证。

## 2) 当前推荐架构（按你现状收敛）

- 主链路（研发协作检索，默认使用）
 - 内容源：`ai_learning/`、`AGENTS.md`、`model_project/docs/`、`docs/`、`业务需求/` 等
 - 索引实现：`local_knowledge_model`（TF-IDF）
 - 索引产物：`local_knowledge_model/embeddings/simple_db/knowledge.pkl`、`vectorizer.pkl`
- 可选链路（对外问答应用）
 - 项目：`ai_applications/kb_qa_mvp`
 - 索引实现：FAISS（RAG 用）
 - 索引产物：`ai_applications/kb_qa_mvp/vector_store/faiss_index/`

## 3) 安全与忽略规则（强烈建议保留）

- 忽略规则文件：`config/kb_ignore_patterns.txt`
- 用途：避免把索引自身/缓存/二进制大文件/明显敏感信息纳入索引。
- 需要纳入 PDF/图片等时：从忽略规则中移除对应后缀即可（建议先在小目录试跑）。

## 4) 日常更新闭环（推荐命令）

- 查看同步状态：
 - `python tools/kb_status.py`
- 构建主链路（默认）：
 - `python tools/kb_build_all.py`
 - 增量：`python tools/kb_build_all.py --incremental`
 - 文档优先（更快）：`python tools/kb_build_all.py --profile docs`
- 同时构建问答链路（可选）：
 - `python tools/kb_build_all.py --with-qa`
 - 使用 kb_qa_mvp 的默认目录（更大、更慢）：`python tools/kb_build_all.py --with-qa --qa-default-dirs`
 - 自定义目录：`python tools/kb_build_all.py --with-qa --qa-dir d:\企业数据平台\业务需求 --qa-dir d:\企业数据平台\docs`

## 5) 最小验收标准（建议固定）

- 同步：`kb_status.py` 显示 TF-IDF 索引不落后于关键来源文件。
- 可检索：能用整句问题在本地命中表名/字段/规范（来源路径与更新时间正确）。
- 不泄露：敏感文件不会被纳入索引（通过忽略规则保障）。


---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。