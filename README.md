# AI 协作开发方法论

> **模板定义「需要什么证据」→ 用户按模板提供 → AI 按证据开发**

---

## 为什么需要这套方法论？

### 传统 AI 开发的痛点

```
用户：帮我写一个订单统计的 SQL
AI：好的，我来写...
（AI 不知道表结构、不知道业务口径、不知道红线规则）
→ 结果：写出来的代码大概率要返工
```

**问题根源**：AI 每次都要重新理解业务，而业务知识在人脑里。

### 本方法论的解决方案

```
模板说：你需要提供 model_csv（设计稿）、数据字典、红线规则
用户做：把文件放到指定路径
AI 做：读取文件，按优先级开发
```

**核心价值**：把「业务知识」从人脑转移到项目结构，AI 和开发者都能直接用。

---

## 这套方法论包含什么？

### 1. AGENTS.md — AI 的「工作手册」

AI 每次启动自动加载，告诉 AI：
- 该从哪里找证据（证据优先级）
- 哪些事情不能做（红线规则）
- 冲突时怎么办（处理流程）

### 2. 门卫脚本 — 自动纠错

改代码前自动跑，校验：
- 设计稿是否存在
- mtime 是否是当日版本
- 字段是否冲突

### 3. 教训库 — 经验传承

踩过的坑自动沉淀，下次自动避开。

### 4. 场景路由 — 任务分流

不同任务用不同工具，不是所有问题都走 RAG。

---

## 10 分钟上手

### 方式 1：新项目直接用

```bash
# 1. 克隆模板
git clone git@github.com:eninem123/agent-methodology.git
cd agent-methodology

# 2. 复制 AGENTS.md 到你的项目
cp agent_setup_prompts_generic_export.md /your-project/AGENTS.md

# 3. 打开 AGENTS.md，把【】替换成你的项目信息
# 4. 在 Cursor 里打开项目，AI 会自动加载 AGENTS.md
```

### 方式 2：老项目改造

```bash
# 1. 先复制 AGENTS.md
cp agent_setup_prompts_generic_export.md /your-project/AGENTS.md

# 2. 在 Cursor Agent 对话里粘贴这个提示词：
# "扫描当前仓库，帮我生成 .cursor/rules/ 目录和基本的 guard 脚本"
```

---

## 文件结构

```
agent-methodology/
├── README.md                              # 你正在看的文件
│
├── 📋 模板文件
│   ├── agent_setup_prompts_generic_export.md   # 通用 AGENTS.md 模板
│   ├── agent_setup_prompts_by_scenario.md      # 分场景搭建提示词手册
│   └── low_token_prompt_templates.md           # 日常低 token 提问模板
│
├── 📖 实战文档（来自 企业数据平台）
│   ├── AGENTS-企业数据平台.md                    # 数据中台 AI 协作规范
│   ├── warehouse_on_demand.md                 # 数仓扩展规范（按需查阅）
│   ├── SKILL-企业数据平台.md                     # Skills 总览
│   ├── knowledge_base_workflow.md             # 知识库主链路
│   └── README-企业数据平台.md                    # 项目结构说明
│
├── 📊 架构参考
│   └── docs/
│       ├── 01-routing-architecture.md         # 任务路由架构演进
│       └── 02-capability-mapping.md           # 能力对应表
│
└── 📝 变更记录
    └── optimization_changelog_20260105.md     # 代码库优化变更说明
```

---

## 核心概念

### 1. 分层架构（不让 AI 堆长文）

| 层级 | 放什么 | 不放什么 |
|------|--------|----------|
| **AGENTS.md** | 红线索引、证据优先级、入口路径 | 长篇语法细则 |
| **rules** | 开发前/中/后 checklist | 与 glob 无关的废话 |
| **on_demand** | StarRocks 细则、SAP 清洗 | 每轮都需要的红线 |
| **Skill** | 触发词、流程、输出契约 | 整个仓库规范 |
| **tools/hooks** | 可脚本化校验 | 需要人脑判断的业务口径 |

### 2. 证据优先级（让 AI 知道该读什么）

```
P0：用户 @ 的文件 / 磁盘当日设计稿（看 mtime）→ 必须读
P1：代码与 schema → 参考
P2：历史 KB / 教训 → 仅辅助，不替代设计
```

**用户按模板要求提供：**
- 设计稿路径 → AI 读取后开发
- 数据字典 → AI 按字段开发 SQL
- 教训库 → AI 自动避开踩过的坑

### 3. 门卫机制（自动纠错）

改代码前自动跑守卫脚本，校验：
- 设计稿是否存在
- mtime 是否是当日版本
- 字段是否冲突

### 4. 验证模式（从竞赛到工程）

灵感来源：IMO 2025 Gemini 金牌验证提示词。核心思路：**生成（Solver）和核验（Verifier）分离**。

```
┌─────────────────────────────────────────────────────────────┐
│                    验证闭环 Loop                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
│   │  生成   │───→│ 独立核验 │───→│ 修正复核 │───→│ 交叉验证 │ │
│   │ (Solver)│    │(Verifier)│   │ (Fix)   │    │(Cross)  │ │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘ │
│        ↑                              │                    │
│        └──────────────────────────────┘                    │
│              不通过则循环                                   │
└─────────────────────────────────────────────────────────────┘
```

**5 个核心模式**：
1. **角色分离**：AI 生成代码 → 门卫脚本独立校验（不依赖 AI 自己的判断）
2. **分类标注**：BLOCKER（致命错误）/ SHOULD（建议改）/ QUESTION（待确认）
3. **逐步核验**：逐字段检查设计稿 mtime + 字段清单
4. **不修改只审计**：门卫只报错，不自动修代码
5. **交叉验证**：行数/金额/明细抽样核对

详见 `docs/03-verification-pattern.md`

### 5. 教训闭环 Loop（经验传承）

这是本方法论的**持续进化机制**——不是单向输出，而是越用越聪明。

```
┌─────────────────────────────────────────────────────────────┐
│                    教训闭环 Loop                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │
│   │  踩坑   │───→│ 写教训  │───→│ 业务确认 │───→│ 写入红线 │ │
│   │         │    │ (draft) │    │(confirmed)│   │ AGENTS  │ │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘ │
│        ↑                                              │     │
│        │              下次自动避开                    │     │
│        └──────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**为什么这是闭环？**
- 教训写入 AGENTS 后，AI 每次启动自动加载
- 下次遇到类似场景，AI 自动避开
- 避开后如果又踩新坑，继续循环
- **越用越聪明，不是一次性文档**

---

## 适用场景

| 场景 | 优先建什么 | 参考文档 |
|------|------------|----------|
| 数据中台 / 数仓 | AGENTS + guard + hook | `AGENTS-企业数据平台.md` |
| 后端 API / 微服务 | AGENTS + OpenAPI rule | `agent_setup_prompts_by_scenario.md` §4 |
| 前端 / 全栈 | AGENTS + token rule | `agent_setup_prompts_by_scenario.md` §5 |
| Code Review | Skill 即可 | `agent_setup_prompts_by_scenario.md` §6 |
| 老项目改造 | 分阶段 AGENTS | `agent_setup_prompts_by_scenario.md` §12 |

---

## 真实案例

### 企业数据平台（企业数据平台）

**改造前**：
- AI 每次重新理解业务
- 改 SQL 经常出错
- 踩过的坑重复踩

**改造后**：
- AGENTS.md 写清红线和证据来源
- 门卫脚本自动校验 model_csv
- 教训库沉淀 50+ 条经验
- AI 开发效率提升 3-5x

详见 `README-企业数据平台.md`

---

## 进阶用法

### 配合 Cursor 使用

```bash
# 1. 在项目根目录创建 .cursor/rules/
mkdir -p .cursor/rules

# 2. 从 agent_setup_prompts_by_scenario.md 复制对应场景的 rules
# 3. 在 Cursor Settings → Rules 里启用
```

### 配合其他 AI 工具

这套方法论不绑定 Cursor，适用于：
- Cursor Rules
- Claude Code
- GitHub Copilot
- 任何支持 System Prompt 的 AI 编码工具

---

## 相关资源

| 资源 | 说明 |
|------|------|
| [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) | 3900+ 个 Cursor 规则文件 |
| [Cursor 官方文档](https://docs.cursor.com/context/rules) | Project Rules 说明 |
| [企业数据平台](https://github.com/eninem123/企业数据平台) | 本方法论的实战来源 |

### 工程化落地

本方法论解决「AI 应该怎么做」，[SCALE Engine](https://gitee.com/hongmaple/scale-engine) 解决「AI 不遵守怎么办」——它把治理规则变成 CLI 门禁、退出码拦截和证据文件，让约束不再只靠提示词自律。两者配合使用效果最佳。

---

## 许可

MIT License

---

> 基于企业级 AI 协作实践沉淀 · 2026-06-30