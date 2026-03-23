# Day 1 总结

## 今日目标回顾

Day 1 的目标不是深入源码，而是先建立对 `deepagents` 的整体认知：

1. 它到底是干什么的
2. 这个 monorepo 里每个模块分别负责什么
3. SDK、CLI、ACP、Evals、Partners 之间的关系是什么

---

## 项目结构图

```txt
deepagents/
├── libs/deepagents/
│   └── 核心 SDK
│      负责创建 Deep Agent，装配 model、tools、middleware、backend、subagents
│
├── libs/cli/
│   └── 终端产品层
│      把 SDK 包装成可直接使用的 CLI，提供 TUI、会话、审批、skills、搜索等能力
│
├── libs/acp/
│   └── 编辑器接入层
│      通过 ACP 协议把 Deep Agent 接入支持 ACP 的编辑器，例如 Zed
│
├── libs/evals/
│   └── 评测体系
│      用 Harbor、LangSmith、Terminal Bench 等方式评估 agent 质量
│
├── libs/partners/
│   └── 基础设施适配层
│      对接 Daytona、Modal、Runloop、QuickJS 等 sandbox/backend
│
├── examples/
│   └── 示例项目
│      展示 Deep Agent 的典型使用方式
│
└── README.md
   └── 项目总入口
      说明整体定位、默认能力、SDK 与 CLI 的关系
```

### 分层理解

```txt
核心框架层      -> libs/deepagents
产品层          -> libs/cli
协议接入层      -> libs/acp
质量评测层      -> libs/evals
基础设施适配层  -> libs/partners
```

---

## 我是怎么理解 deepagents 的

`deepagents` 不是一个普通的 LLM SDK，也不是只帮你封装模型调用的工具库。我对它的理解是，它是一个“深度 Agent 运行时框架”。它想解决的问题是：普通 tool-calling agent 在面对复杂任务时容易失控，比如不会规划、上下文很快爆掉、不会拆任务、不会稳定读写文件和执行命令。`deepagents` 把这些复杂任务真正需要的能力预装好了，包括 planning、filesystem、shell、subagents、memory、skills 和上下文管理。整个仓库是一个 monorepo：`libs/deepagents` 是核心 SDK，负责创建 agent；`libs/cli` 是终端产品层，类似 Claude Code 风格的可直接使用工具；`libs/acp` 是编辑器协议接入层；`libs/evals` 是评测和 benchmark 体系；`libs/partners` 是不同 sandbox 和 backend 的适配层。所以它不是单个库，而是一整套围绕 Deep Agent 的工程体系。

---

## 今日核心结论

- `deepagents` 的核心价值，不是单纯“让模型调用工具”，而是“让模型能稳定完成复杂任务”。
- 它的核心抽象不是聊天接口，而是一个可运行的 Deep Agent。
- 这个仓库不是单包项目，而是围绕 Agent 构建、交互、接入、评测、运行环境的完整 monorepo。
- `libs/deepagents` 是最核心的源码入口，后续学习要围绕它展开。
- `libs/cli` 是最适合你这种前端背景快速建立感性认知的模块，因为它更接近一个完整产品。

---

## 今天学到的关键词

- Deep Agent
- Agent harness
- Monorepo
- SDK
- CLI
- ACP
- Evals
- Sandbox
- Backend

---

## 明天要进入的内容

Day 2 开始进入最核心的源码装配入口：

- `libs/deepagents/deepagents/__init__.py`
- `libs/deepagents/deepagents/graph.py`

目标是搞清楚：

- `create_deep_agent()` 到底做了什么
- 一个 agent 是怎么被装配出来的
- model、tools、middleware、memory、subagents 是怎么接进去的
