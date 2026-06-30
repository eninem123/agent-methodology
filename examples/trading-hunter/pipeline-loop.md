# Pipeline Loop 串联说明（模板）

将以下步骤挂在日频 pipeline **末尾**（交易逻辑完成之后）：

```bash
# 1. 信号复盘（只读，供 lessons / journal）
python3 tools/signal_postmortem.py

# 2. 策略烤机：有指纹变更则全量；通过则自动 accept 基线 + clear_gate
python3 tools/strategy_grill.py --pipeline

# 3. 集成健康检查
python3 tools/health_check.py --soft

# 4. 过程日记（含 grill / health / postmortem 摘要）
python3 tools/phase1_process_journal.py

# 5. 对外部署（可选）
# ./zhuli_deploy.sh
```

## strategy_grill WATCHED 示例

```
position_config_v4.json
tools/autonomous_gate.py
tools/signal_filter.py
tools/strategy_router.py
…
```

## agent_gate 状态

- 状态文件：`daily/agent_gate_state.json`
- `needs_grill: true` 时 preToolUse 拦截受保护路径
- grill pipeline PASS 后自动 `clear_gate`

## 严重级别与汇报

| grill | health | 对用户 |
|-------|--------|--------|
| FAIL | any | BLOCKER，Agent 自修后汇报 |
| PASS | critical | BLOCKER |
| PASS | warning | SHOULD，写 journal |
| PASS | pass | 沉默或一句正常 |
