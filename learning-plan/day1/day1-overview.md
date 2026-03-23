# Day 1 学习计划：建立对 DeepAgents 的全局认知

## 今日目标

今天不深入源码实现，只做一件事：建立对 `deepagents` 项目的整体认知。

你今天学完后，应该能回答下面 3 个问题：

1. `deepagents` 到底是干什么的
2. 这个 monorepo 里每个包分别负责什么
3. SDK、CLI、ACP、Evals、Partners 之间是什么关系

---

## 适合你的学习方式

不要按 Python 初学者方式学，也不要一上来扎进实现细节。

今天只抓这 3 层：

1. 产品定位
2. 模块分层
3. 整体执行流程

你的优势要主动利用：

- 用前端框架分层思维理解 monorepo
- 用状态流思维理解 agent runtime
- 用产品工程思维理解 CLI、ACP、Evals 这些配套模块

---

## 今日阅读文件

按这个顺序读，不要跳：

1. `README.md`
2. `libs/README.md`
3. `libs/deepagents/README.md`
4. `libs/cli/README.md`
5. `libs/acp/README.md`
6. `libs/evals/README.md`

---

## 每个文件要看什么

### 1. `README.md`

重点看：

- 项目的主定位
- 默认包含哪些能力
- 为什么它叫 “batteries-included agent harness”
- SDK 和 CLI 在顶层是怎么被介绍的

你要得到的结论：

- `deepagents` 不是普通 LLM SDK
- 它是一个开箱即用的深度 Agent 框架
- 它想解决的是复杂任务下 agent 的规划、上下文、文件操作、任务拆分问题

### 2. `libs/README.md`

重点看：

- monorepo 里有哪些包
- 每个包在整体架构中的位置
- `partners/` 为什么存在

你要得到的结论：

- 这是一个多包协作仓库，不是单包项目
- 核心 SDK、CLI、ACP、Evals、Partners 是明确分层的

### 3. `libs/deepagents/README.md`

重点看：

- Deep Agent 的定义
- 为什么普通 tool-calling agent 会“浅”
- Deep Agent 的几个关键组成部分

你要得到的结论：

- 这个项目的核心抽象是“Deep Agent”
- 它强调 planning、subagents、filesystem、prompt orchestration

### 4. `libs/cli/README.md`

重点看：

- CLI 和 SDK 的关系
- CLI 额外增加了哪些能力
- 为什么说它类似 Claude Code / Cursor

你要得到的结论：

- CLI 是产品层，不是核心框架层
- 它是在 SDK 上面封装的终端交互产品

### 5. `libs/acp/README.md`

重点看：

- ACP 是什么
- 为什么要支持编辑器接入
- Deep Agent 怎么嵌入 Zed 这类编辑器

你要得到的结论：

- ACP 是协议接入层
- 它解决的是“把 Deep Agent 放进编辑器”这个场景

### 6. `libs/evals/README.md`

重点看：

- Harbor 和 LangSmith 是怎么配合的
- 为什么 agent 项目需要单独的评测体系
- benchmark 在这里扮演什么角色

你要得到的结论：

- agent 项目不能只靠单测
- 这个仓库重视 benchmark、trajectory、reward score 和回归分析

---

## 你今天必须记住的核心结论

### 一句话理解

`deepagents` 是一个让复杂 Agent 能开箱即用跑起来的运行时框架。

### 用前端视角类比

- `libs/deepagents` 像核心框架层
- `libs/cli` 像官方客户端产品
- `libs/acp` 像编辑器协议适配层
- `libs/evals` 像质量与 benchmark 平台
- `libs/partners` 像基础设施适配器

### 项目总图

```txt
deepagents/
├── libs/deepagents/   # 核心 SDK
├── libs/cli/          # 终端产品
├── libs/acp/          # 编辑器协议接入
├── libs/evals/        # benchmark / 评测
└── libs/partners/     # sandbox / backend 适配
```

### 你现在对项目的正确理解应该是

这个仓库不是“一个 Python AI 库”，而是一整套围绕 Deep Agent 的工程化体系：

- SDK 负责生成 agent
- CLI 负责直接使用
- ACP 负责编辑器集成
- Evals 负责持续评估
- Partners 负责接不同的运行环境

---

## 今日重点问题

读完后，你要自己回答这几个问题：

1. 为什么 `deepagents` 不等于普通 function-calling agent
2. 为什么这个项目要拆成这么多 `libs/*` 包
3. SDK 和 CLI 为什么不能混成一个包
4. ACP 和 MCP 这类外部协议接入，为什么值得单独存在
5. 为什么 agent 项目必须做 evals，而不是只写 unit test

---

## 今日输出物

今天必须产出两个东西。

### 输出物 1：项目结构图

手画也行，Markdown 也行。至少包含：

- `libs/deepagents`
- `libs/cli`
- `libs/acp`
- `libs/evals`
- `libs/partners`

并写出每个模块的一句话职责。

### 输出物 2：300 字总结

题目：

`我是怎么理解 deepagents 的`

建议结构：

- 它是什么
- 它和普通 Agent SDK 的区别
- 这个 monorepo 的几个核心模块分别做什么

---

## 推荐时间安排

### 第 1 段：20 分钟

读 `README.md` 和 `libs/README.md`

目标：
先建立“项目总图”

### 第 2 段：20 分钟

读 `libs/deepagents/README.md` 和 `libs/cli/README.md`

目标：
搞清楚 SDK 和 CLI 的边界

### 第 3 段：20 分钟

读 `libs/acp/README.md` 和 `libs/evals/README.md`

目标：
理解外部集成和评测体系

### 第 4 段：20 到 30 分钟

自己输出结构图和 300 字总结

---

## 今日验收标准

如果今天学完，你能够不看文档，直接说清楚下面这段话，就算 Day 1 合格：

`deepagents` 是一个开箱即用的深度 Agent 框架。它不是单纯封装模型调用，而是把复杂任务所需的规划、文件系统、Shell、子 Agent、记忆和上下文管理能力整合起来。整个仓库是一个 monorepo：`libs/deepagents` 是核心 SDK，`libs/cli` 是终端产品，`libs/acp` 是编辑器协议接入层，`libs/evals` 是评测体系，`libs/partners` 是不同执行环境和沙箱的适配层。

---

## 明天预告

Day 2 会进入项目最核心的装配入口：

- `libs/deepagents/deepagents/__init__.py`
- `libs/deepagents/deepagents/graph.py`

目标是搞清楚 `create_deep_agent()` 到底做了什么，它如何把 model、tools、middleware、subagents、memory 组装成一个真正能跑的 agent。
