---
AIGC:
 Label: "1"
 ContentProducer: 001191110102MACQD9K64018705
 ProduceID: coze_space/4496329707377712/favorite/low_token_prompt_templates-7d304d2198.md
 ReservedCode1: ""
 ContentPropagator: 001191110102MACQD9K64028705
 PropagateID: 4496329707377712#1782779553669
 ReservedCode2: ""
---
# 低 Token 提问模板（数据中台）

使用原则：每次只做 1 件事，限定路径，限定输出长度，默认不贴全量日志。

> 新项目搭 Agent 骨架、分场景可复制提示词见 [`agent_setup_prompts_by_scenario.md`](agent_setup_prompts_by_scenario.md)。

## 模板 1：SQL 开发

```text
任务：为 <目标表名> 开发/修改 SQL（仅此任务）。

范围：
- 只看这些文件：@model_project/src/prod/<layer>/<target>.sql @model_project/docs/model_csv/<target>.csv
- 不要全仓搜索，不要展示全量 git status。

已知口径：
- 粒度：<如 订单行>
- 主键：<字段1,字段2>
- 业务过滤：<条件>
- 目标产出：<字段清单或指标>

约束：
- StarRocks 3.3.19；禁止 SELECT *；SELECT/CASE 禁子查询。
- 若与 model_csv/数据字典冲突，先列冲突，不要直接实现。

输出要求（精简）：
1) 先给 3 条结论（每条 <= 20 字）
2) 再给改动文件列表
3) 最后给待我确认项（若无写"无"）
```

## 模板 2：SQL 排错

```text
任务：定位并修复 <报错/异常现象>。

范围：
- 仅排查：@model_project/src/prod/<layer>/<job>.sql
- 可参考：@model_project/docs/model_csv/<job>.csv
- 不做无关重构。

现象：
- 报错信息：<粘贴关键 5~20 行>
- 预期结果：<一句话>
- 实际结果：<一句话>

排查要求（低 token）：
- 先给"最可能原因 Top3"（按概率排序）
- 每个原因给"最小验证 SQL/步骤"1 条
- 确认后再改代码

输出要求：
- 总字数控制在 200 字内
- 不复述大段日志
```

## 模板 3：口径确认（先对齐再开发）

```text
任务：只做口径确认，不改代码。

主题：<指标名/报表名>
范围：
- 只对照：@model_project/docs/model_csv/<sheet>.csv @model_project/docs/dim_data_dictionary_df.csv
- 必要时补充：@model_project/src/prod/<layer>/<related>.sql

我要你输出（固定格式）：
1) 口径定义（<= 80 字）
2) 计算公式（1 行）
3) 过滤条件（最多 5 条）
4) 粒度与主键
5) 冲突清单（业务文档 vs 设计/字典）
6) 待确认问题（最多 3 条）

要求：
- 只给结论，不展开背景。
- 若证据不足，直接写"证据不足 + 缺失项"。
```

## 可复用短指令（建议追加在每次提问末尾）

```text
请用低 token 模式：限制在指定路径内工作；不主动输出 git status；先给结论后细节；总字数尽量 <= 200。
```

---

> 本内容由 Coze AI 生成，请遵循相关法律法规及《人工智能生成合成内容标识办法》使用与传播。