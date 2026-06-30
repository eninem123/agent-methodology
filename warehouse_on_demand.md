---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/warehouse_on_demand-081c0780b3.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779405174
 ReservedCode2: ""
---
# 数仓扩展规范（按需查阅）

本文档用于承接不需要每轮注入的长规范。默认先遵循 `AGENTS.md` 核心版，命中以下场景再查本文件。

## 1. SAP `ods_sap_erp_*` 空格清洗（命中即查）

- 先跑：`python tools/ensure_sap_erp_space_profile.py --from-model <模型名> --prod`。
- 报告以"字段详情"页签逐列判定，不只看摘要。
- 列按 T0~T4 定型后再选模板：
 - `T0`：无专项清洗要求。
 - `T1`：可用 `NULLIF(col, ' ')`。
 - `T2`：在确认无 T3/T4 时可用 `NULLIF(TRIM(col), '')`。
 - `T3`：保留原值，不做整列 `TRIM` 后回写。
 - `T4`：必须按值 `CASE`，禁止整列一刀切。
- 业务键/JOIN 键两侧必须同式，避免失配。
- **ETL 脚本默认**：关联键、输出字段与模型 CSV **嵌入参考 SQL** 一致，**不做 blanket `TRIM`**。SAP 同步到 StarRocks 后，键字段常见为定长右补空格，`mseg.charg` 与 `zcotab.zcharg` 等**两侧同填充**时直连等值即可，勿先 TRIM 一侧。
- **「全是空格」**：过滤/关联条件用 `IS NOT NULL`、`<> ''`、`<> ' '` 等定点判断；不要把「统计非空」的 `TRIM` 写法套进全表 SELECT/JOIN。
- **上线前是否加 TRIM**：用 gpustack 查生产 `SUM(CASE WHEN col <> TRIM(col) THEN 1 END)`；若为 0，则不加（复盘案例：`dwd_co_mat_transfer_df` 的 `awkey`/`mblnr`/`belnr` 等；`dwd_fin_cost_variance_analysis_month_di` 2026-05 收入键 `need_trim_pairs=0`，见 `ai_learning/lessons_learned/20260515_新开发禁止blanket_CAST与TRIM_先查来源表与SAP空格.md`）。

## 6. 类型 CAST（新开发默认）

- 写 `CAST` 前必查 ODS/DWD **来源列类型**（生产 `information_schema`），禁止「模板式」`CAST(源字段 AS VARCHAR(n))`。
- 通常保留：`CAST(... AS DECIMAL(...))`（金额/数量结果）、`STR_TO_DATE(LPAD(日期字段,8,'0'), ...)`（SAP 日期字符）、与目标 DDL 不一致时的**少量**显式转换。
- 与参考 SQL 一致时，源已是 `varchar` 则**原样输出/关联**。

## 2. StarRocks 语法与写入细则（复杂 SQL 时查）

- 不支持 `QUALIFY`，用子查询 + `ROW_NUMBER()`。
- `INSERT OVERWRITE ... WITH ... SELECT ...` 时，`INSERT` 在前，`WITH` 在后。
- 禁止 `SELECT` 列表内子查询与 `CASE` 内子查询。
- 新建日快照表优先按 `dt` 日分区口径；避免无分区条件的整表覆盖写入历史数据。

## 3. 需求冲突处理（口径争议时查）

- 业务文档与 `model_csv` / 设计 Excel / 数据字典 / 源表样本冲突时，先列冲突项并暂停开发。
- 未确认前不得把任一口径写死到生产 SQL。
- "空/无/未填"默认兼指 `NULL` 与空串；数值 `0` 是否算空需单独确认。

## 4. Python ETL 细节（写 Python 时查）

- 递归父子匹配必须包含完整业务键，禁止单字段匹配。
- `Decimal` 参与浮点运算前先转型；递归累加防 `NaN`。
- 批量入库前将空值统一转 `None`：`df.astype(object).where(pd.notnull(df), None)`。

## 5. 收尾与质量

- 修改 SQL 后同步同目录 `readme.md`。
- 关键报表上线前做行数/金额/明细抽样核对并留痕。
- 更新本地知识库：`lessons_learned`/`errors` 用 `--lessons-only`；设计 Excel/model_csv 用 `--incremental`（详见 `docs/knowledge_base/README.md`）。

---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。