# DeepAgents 学习计划

## 目标

这份计划的目标不是“看完源码”，而是让你以 8 年前端工程师的优势，快速建立对 `deepagents` 的完整心智模型：

1. 知道这个框架解决什么问题
2. 知道一次请求在项目里是怎么流动的
3. 知道 monorepo 每个包的职责边界
4. 知道 SDK、CLI、ACP、Evals、Partners 之间如何协作
5. 知道如果你要二开，应该先改哪里、怎么验证

---

## 先建立总图

### 一句话理解

`deepagents` 是一个开箱即用的深度 Agent 框架。它不是“包一层模型调用”，而是把复杂 Agent 真正需要的规划、文件系统、Shell、子 Agent、记忆、技能、总结、沙箱这些能力都预装好了。

### 你应该怎么理解它

用前端视角类比：

- `create_deep_agent()` 像应用启动入口
- `middleware` 像 Koa / Redux / Next middleware
- `tools` 像 agent 可以调用的外部能力接口
- `backend` 像运行时适配层 / 数据源适配层
- `subagents` 像 worker / 微流程 / 子任务执行器
- `cli` 像在 SDK 之上做好的产品壳
- `evals` 像回归测试 + benchmark 平台

### Monorepo 架构

```txt
deepagents/
├── libs/deepagents/   # 核心 SDK
├── libs/cli/          # 终端产品，类似 Claude Code 风格
├── libs/acp/          # 接入 ACP，让编辑器可用 Deep Agent
├── libs/evals/        # benchmark / 评测 / Harbor 集成
└── libs/partners/     # 各种沙箱和执行后端适配
```

### 整体执行流程

```txt
用户输入
  ↓
CLI main / app
  ↓
create_cli_agent()
  ↓
create_deep_agent()
  ↓
挂载 middleware
  - TodoList
  - Filesystem
  - SubAgent
  - Summarization
  - Skills
  - Memory
  - HITL
  ↓
LangGraph 编译后的 graph
  ↓
模型决策：输出文本 / 发起 tool call
  ↓
backend 执行文件 / shell / sandbox / memory 操作
  ↓
结果回填 state
  ↓
继续循环直到任务完成
```

### 第一批必须掌握的核心文件

- `README.md`
- `libs/README.md`
- `libs/deepagents/deepagents/__init__.py`
- `libs/deepagents/deepagents/graph.py`
- `libs/deepagents/deepagents/middleware/filesystem.py`
- `libs/deepagents/deepagents/middleware/subagents.py`
- `libs/deepagents/deepagents/middleware/memory.py`
- `libs/deepagents/deepagents/backends/protocol.py`
- `libs/deepagents/deepagents/backends/state.py`
- `libs/cli/deepagents_cli/main.py`
- `libs/cli/deepagents_cli/app.py`
- `libs/cli/deepagents_cli/agent.py`
- `libs/cli/deepagents_cli/server_graph.py`

---

## 学习策略

### 适合你的读法

你是 8 年前端，不要按“Python 语法教程”去学。按下面顺序最快：

1. 先看产品边界和模块边界
2. 再看真实执行链路
3. 再看中间件和后端协议
4. 最后看评测和扩展机制

### 你的优势要主动利用

- 你对组件分层敏感：直接拿来理解 monorepo 包分层
- 你对状态流敏感：直接拿来理解 LangGraph state 和 middleware 链
- 你对工程化敏感：重点盯 `Makefile`、`pyproject.toml`、tests、package 边界
- 你对产品化交互敏感：CLI 的 Textual UI 会比纯算法更容易上手

### 学习产出要求

每天都要产出，不要只读：

- 1 张你自己画的流程图
- 1 份 300-800 字当日总结
- 1 个“我如果来改，会从哪开始”的入口判断
- 1 个最小实验或验证动作

---

## 14 天学习计划

## Day 1: 建立全局认知

### 目标

搞清楚 `deepagents` 到底是什么，不把它误解成普通 LLM SDK。

### 阅读文件

- `README.md`
- `libs/README.md`
- `libs/deepagents/README.md`
- `libs/cli/README.md`
- `libs/acp/README.md`
- `libs/evals/README.md`

### 重点问题

- 项目为什么存在
- 和普通 tool-calling agent 的差异是什么
- monorepo 每个包分别服务谁
- SDK 和 CLI 的关系是什么

### 练习

用你自己的话写一段 300 字说明：
“如果让我给一个资深前端介绍 deepagents，我会怎么说。”

### 当日产出

- 一张 monorepo 结构图
- 一段项目定位总结

---

## Day 2: 先看最核心入口 `create_deep_agent`

### 目标

理解这个框架到底是怎么把一个 Agent 组装出来的。

### 阅读文件

- `libs/deepagents/deepagents/__init__.py`
- `libs/deepagents/deepagents/graph.py`
- `libs/deepagents/deepagents/base_prompt.md`
- `libs/deepagents/deepagents/_models.py`

### 重点问题

- 默认模型怎么选
- 默认 tools 有哪些
- 默认 middleware 栈有哪些
- `create_deep_agent()` 最终返回的是什么
- `system_prompt`、`tools`、`subagents`、`memory`、`backend` 如何注入

### 前端类比

把 `create_deep_agent()` 当成应用启动文件，像 React 根组件 + Provider 装配层。

### 练习

画一张 `create_deep_agent()` 的参数到最终 graph 的装配图。

### 当日产出

- 一张装配链路图
- 一份 `create_deep_agent` 参数说明笔记

---

## Day 3: 吃透 middleware 思路

### 目标

理解为什么这个框架能“深”，核心就在 middleware 组合。

### 阅读文件

- `libs/deepagents/deepagents/middleware/__init__.py`
- `libs/deepagents/deepagents/middleware/filesystem.py`
- `libs/deepagents/deepagents/middleware/subagents.py`
- `libs/deepagents/deepagents/middleware/summarization.py`
- `libs/deepagents/deepagents/middleware/patch_tool_calls.py`
- `libs/deepagents/deepagents/middleware/_utils.py`

### 重点问题

- 每个 middleware 负责什么能力
- 为什么不是把所有逻辑写死在 agent 主函数里
- middleware 之间是怎么叠加生效的
- system prompt 是如何被动态增强的

### 前端类比

这部分最像 Koa middleware + 插件系统。你要重点抓“横切关注点”。

### 练习

列一张表，写清楚每个 middleware：

- 输入什么
- 改什么 state / prompt / tool
- 输出什么

### 当日产出

- 一张 middleware 责任表

---

## Day 4: 文件系统能力与 tool 设计

### 目标

理解 agent 为什么能像 coding agent 一样读写文件、grep、执行 shell。

### 阅读文件

- `libs/deepagents/deepagents/middleware/filesystem.py`
- `libs/deepagents/deepagents/backends/utils.py`
- `libs/deepagents/deepagents/backends/protocol.py`

### 重点问题

- `ls` / `read_file` / `edit_file` / `glob` / `grep` / `execute` 是怎么定义的
- 工具描述为什么写得这么长
- 为什么强调“先读再改”
- 文件内容为什么要带 line number / truncation / pagination

### 前端类比

这像你在做一个 IDE 插件平台时定义标准能力接口，关键不是功能多，而是“给模型一个稳定 API 契约”。

### 练习

写一张“tool contract”文档，说明这些工具的输入输出和设计约束。

### 当日产出

- 一份工具接口说明

---

## Day 5: backend 抽象层

### 目标

理解为什么 `deepagents` 能接不同存储和执行环境。

### 阅读文件

- `libs/deepagents/deepagents/backends/protocol.py`
- `libs/deepagents/deepagents/backends/state.py`
- `libs/deepagents/deepagents/backends/filesystem.py`
- `libs/deepagents/deepagents/backends/local_shell.py`
- `libs/deepagents/deepagents/backends/sandbox.py`
- `libs/deepagents/deepagents/backends/composite.py`

### 重点问题

- `BackendProtocol` 规定了哪些契约
- `StateBackend` 和真实文件系统 backend 有什么差异
- `CompositeBackend` 存在的意义是什么
- shell 和 filesystem 为什么要拆成 backend 而不是直接绑死

### 前端类比

类似前端的 repository / adapter pattern，一套上层逻辑，挂不同基础设施实现。

### 练习

自己画一个 backend 适配层类图。

### 当日产出

- 一张 backend 抽象图

---

## Day 6: SubAgent 机制

### 目标

理解 `deepagents` 和普通 Agent 拉开差距的关键能力之一：子 Agent。

### 阅读文件

- `libs/deepagents/deepagents/middleware/subagents.py`
- `libs/deepagents/deepagents/middleware/async_subagents.py`
- `libs/deepagents/deepagents/graph.py`

### 重点问题

- `task` 工具是怎么暴露出去的
- `SubAgent`、`CompiledSubAgent`、`AsyncSubAgent` 区别是什么
- 主 agent 为什么需要隔离上下文窗口
- 什么任务适合分给 subagent

### 前端类比

像主线程把重任务拆给 web worker 或独立服务，目的是隔离上下文和降低主流程复杂度。

### 练习

设计 3 个你自己的 subagent 场景：

- code review agent
- docs summarizer agent
- release note agent

### 当日产出

- 一份 subagent 使用策略笔记

---

## Day 7: Memory 与 Skills

### 目标

理解长期记忆和按需技能的差异。

### 阅读文件

- `libs/deepagents/deepagents/middleware/memory.py`
- `libs/deepagents/deepagents/middleware/skills.py`
- 当前仓库根目录 `AGENTS.md`
- `libs/cli/deepagents_cli/built_in_skills/remember/SKILL.md`

### 重点问题

- memory 为什么基于 `AGENTS.md`
- skills 和 memory 的边界是什么
- 什么信息应该进入长期记忆，什么不应该
- skills 是怎么影响 agent 行为的

### 前端类比

- memory 像全局持久化配置
- skills 像按需加载的工作流插件

### 练习

写一个对比表：

- memory 解决什么问题
- skills 解决什么问题
- 各自的触发时机和风险点

### 当日产出

- 一张 memory vs skills 对比表

---

## Day 8: CLI 入口和启动流程

### 目标

看懂 `deepagents-cli` 如何从命令行参数进入完整 agent 会话。

### 阅读文件

- `libs/cli/deepagents_cli/main.py`
- `libs/cli/deepagents_cli/ui.py`
- `libs/cli/deepagents_cli/non_interactive.py`
- `libs/cli/deepagents_cli/output.py`

### 重点问题

- 参数解析怎么做
- 为什么启动路径要避免重依赖导入
- interactive 和 non-interactive 两条路径怎么分流
- CLI 为什么强调启动性能

### 前端类比

这部分像应用入口、路由分流和 bundle 首屏优化。

### 练习

画一张 CLI 从 `main.py` 到进入会话的流程图。

### 当日产出

- 一张 CLI 启动流程图

---

## Day 9: CLI Agent 生成与 Server Graph

### 目标

理解 CLI 如何在 SDK 之上再包一层产品级能力。

### 阅读文件

- `libs/cli/deepagents_cli/agent.py`
- `libs/cli/deepagents_cli/server_graph.py`
- `libs/cli/deepagents_cli/tools.py`
- `libs/cli/deepagents_cli/subagents.py`

### 重点问题

- `create_cli_agent()` 相比 `create_deep_agent()` 额外加了什么
- web search / MCP / sandbox / ask-user 是怎么挂进去的
- 为什么 server graph 要独立
- CLI 和 SDK 的边界在哪里

### 练习

整理一份“CLI 在 SDK 之上新增能力清单”。

### 当日产出

- 一张 SDK vs CLI 分层图

---

## Day 10: Textual UI 层

### 目标

利用你的前端经验快速看懂 CLI UI。

### 阅读文件

- `libs/cli/deepagents_cli/app.py`
- `libs/cli/deepagents_cli/widgets/messages.py`
- `libs/cli/deepagents_cli/widgets/chat_input.py`
- `libs/cli/deepagents_cli/widgets/tool_widgets.py`
- `libs/cli/deepagents_cli/widgets/approval.py`
- `libs/cli/deepagents_cli/widgets/history.py`

### 重点问题

- Textual App 如何组织状态和组件
- 消息流、工具渲染、审批菜单如何协作
- 这个 UI 层如何映射 agent 的状态变化

### 前端类比

直接把它当“终端版 React 应用”来读：

- `App` 类似根容器
- `widgets` 类似组件目录
- message store 类似状态仓库

### 练习

画出 UI 组件树和核心事件流。

### 当日产出

- 一张 Textual UI 组件图

---

## Day 11: MCP / ACP / 外部编辑器集成

### 目标

理解这个项目如何被接到编辑器、外部工具和 MCP 生态里。

### 阅读文件

- `libs/acp/README.md`
- `libs/cli/deepagents_cli/mcp_tools.py`
- `libs/cli/deepagents_cli/mcp_trust.py`
- `libs/cli/deepagents_cli/server.py`

### 重点问题

- ACP 是解决什么问题
- MCP tool 是怎么被发现和接入的
- 为什么需要 trust / 配置 / server 进程

### 练习

写一份“本地 Deep Agent 接入编辑器”的流程说明。

### 当日产出

- 一张外部集成流程图

---

## Day 12: Evals 与工程质量

### 目标

理解这个仓库怎么评估 Agent，不只看功能。

### 阅读文件

- `libs/evals/README.md`
- `libs/deepagents/tests/README.md`
- `libs/cli/tests/README.md`
- `libs/evals/pyproject.toml`

### 重点问题

- Harbor / LangSmith / terminal-bench 在这里扮演什么角色
- agent 项目为什么需要单独的 eval 系统
- 单测、集成测试、benchmark 各测什么

### 练习

整理一张测试金字塔：

- unit tests
- integration tests
- evals / benchmark

### 当日产出

- 一份项目验证体系说明

---

## Day 13: Partners 与沙箱生态

### 目标

理解 `deepagents` 为什么要有这么多 partners 包。

### 阅读文件

- `libs/partners/daytona/README.md`
- `libs/partners/modal/README.md`
- `libs/partners/runloop/README.md`
- `libs/partners/quickjs/README.md`

### 重点问题

- 这些 partners 为什么独立成包
- 它们如何接入 backend / sandbox 能力
- 本地执行和远程沙箱执行在产品上有什么差异

### 练习

做一张对比表：

- provider
- 支持能力
- 适合场景
- 风险点

### 当日产出

- 一张 sandbox provider 对比表

---

## Day 14: 输出你的最终项目地图

### 目标

把前 13 天碎片知识整成你自己的可复用认知资产。

### 你要输出的最终文档

1. 项目总览
2. 一次请求的完整调用链
3. monorepo 包职责图
4. 核心 middleware 和 backend 图
5. CLI 分层图
6. 如果要二开，我会优先改哪些点
7. 哪些模块最值得继续深入

### 建议输出形式

- 一份 1500-3000 字总结
- 3-5 张图
- 1 张“源码阅读导航图”

### 最终检验标准

到第 14 天，你应该能回答这些问题：

- `create_deep_agent()` 做了什么
- `deepagents-cli` 比 SDK 多了什么
- `task` 子 agent 机制为什么重要
- backend 为什么要协议化
- memory 和 skills 的区别是什么
- 评测体系为什么对 agent 项目很关键

---

## 每周节奏建议

### 第 1 周

目标是搞懂 SDK 核心：定位、graph、middleware、backend、subagents、memory。

### 第 2 周

目标是搞懂产品化层：CLI、Textual UI、MCP/ACP、evals、partners、扩展点。

---

## 每天建议时长

- 30 分钟：快速通读
- 60 分钟：精读核心文件
- 30 分钟：画图和写总结
- 20 分钟：做一个最小验证

总计建议：每天 2 到 2.5 小时。

---

## 最小实践路线

如果你想更快进入“能改代码”的状态，而不是只读文档，建议在第 3 天后开始穿插这 4 个小实验：

1. 用最小代码跑通一个 `create_deep_agent()` 示例
2. 给 agent 加一个自定义 tool
3. 加一个简单 subagent
4. 跑一次 CLI，观察消息、工具、审批和执行流

---

## 对你最重要的阅读原则

- 不要平均用力，优先吃透 `graph.py`、middleware、backend、CLI agent 装配
- 不要被 Python 语法细节拖住，先抓职责和流程
- 不要只看 README，一定要画调用链
- 不要只读不产出，每天必须写“我已经搞懂了什么，还没搞懂什么”

---

## 你学完后的最佳下一步

完成这份计划后，下一阶段最值得做的是三选一：

1. 给 `deepagents` 增加一个你自己的自定义工具链
2. 给 CLI 增加一个你真正会用的 slash command 或 workflow
3. 给项目画一份对外分享的架构图，逼自己把它讲清楚
