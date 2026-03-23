# Day 2 学习计划：读懂 `create_deep_agent()` 的装配过程

## 今日目标

Day 2 的目标是进入 `deepagents` 最核心的源码入口，搞清楚一个 Deep Agent 到底是怎么被创建出来的。

今天学完后，你应该能回答下面 3 个问题：

1. `create_deep_agent()` 最终返回的是什么
2. model、tools、middleware、subagents、memory 是怎么被装进去的
3. 为什么 `graph.py` 是整个 SDK 最重要的入口文件

---

## 适合你的学习方式

你现在已经把项目跑起来了，所以今天不要再停留在 README 层，也不要一上来抠实现细节。

今天重点是“装配视角”：

1. 先看入口导出
2. 再看 agent 构建过程
3. 再看默认能力是怎么挂载的

从前端经验类比：

- `create_deep_agent()` 像应用启动函数
- `graph.py` 像根级装配文件
- 各类 middleware 像一层层 provider / plugin / middleware

---

## 今日阅读文件

按下面顺序读：

1. `libs/deepagents/deepagents/__init__.py`
2. `libs/deepagents/deepagents/graph.py`
3. `libs/deepagents/deepagents/_models.py`
4. `libs/deepagents/deepagents/base_prompt.md`

---

## 每个文件要看什么

### 1. `libs/deepagents/deepagents/__init__.py`

重点看：

- 对外导出了哪些能力
- 哪些类/函数被认为是 SDK 的公开入口

你要得到的结论：

- `create_deep_agent` 是核心公开入口
- `FilesystemMiddleware`、`MemoryMiddleware`、`SubAgentMiddleware` 这些是公开能力模块
- 这个包不是扁平 API，而是围绕 agent 装配能力展开

### 2. `libs/deepagents/deepagents/graph.py`

这是今天的核心文件。

重点看：

- `BASE_AGENT_PROMPT`
- `get_default_model()`
- `create_deep_agent()`
- 默认 middleware 栈的构造逻辑
- general-purpose subagent 的自动注入逻辑

你要得到的结论：

- `create_deep_agent()` 会返回一个编译好的 LangGraph graph
- Deep Agent 的默认能力不是运行时拼凑，而是创建时一次性装配好
- 这个文件是 SDK 的“应用组装总入口”

### 3. `libs/deepagents/deepagents/_models.py`

重点看：

- model string 是如何被解析的
- 为什么 `model` 参数既支持字符串，也支持 `BaseChatModel`

你要得到的结论：

- SDK 支持 provider:model 形式
- 模型层被抽象成可替换依赖，而不是写死在框架里

### 4. `libs/deepagents/deepagents/base_prompt.md`

重点看：

- 默认 prompt 在强调什么
- 它和 middleware 的关系是什么

你要得到的结论：

- prompt 不是附属品，而是 agent 运行时的一部分
- `deepagents` 很强调对模型行为的引导，而不是只靠工具接口

---

## 今天必须抓住的主线

今天你不要平均用力，重点抓下面这条装配链：

```txt
create_deep_agent()
  ↓
解析 model
  ↓
设置 backend
  ↓
构建默认 middleware 栈
  - TodoListMiddleware
  - FilesystemMiddleware
  - SummarizationMiddleware
  - PatchToolCallsMiddleware
  - SkillsMiddleware（可选）
  - MemoryMiddleware（可选）
  - HumanInTheLoopMiddleware（可选）
  ↓
处理 subagents
  ↓
拼 system prompt
  ↓
调用 LangChain / LangGraph create_agent
  ↓
返回 CompiledStateGraph
```

---

## 今日重点问题

读完以后，你要自己回答这几个问题：

1. `create_deep_agent()` 为什么不是简单返回一个 model wrapper
2. 为什么默认能力要通过 middleware 装配
3. 为什么会自动添加 `general-purpose` subagent
4. `deepagents` 为什么强调 prompt + tools + middleware 一起设计
5. 返回 `CompiledStateGraph` 这件事说明了什么

---

## 前端视角理解

今天这部分你可以直接这样类比：

- `create_deep_agent()` 像 `createApp()` / `bootstrapApp()`
- `graph.py` 像应用的根装配文件
- middleware 栈像框架插件和全局中间件
- `CompiledStateGraph` 像“应用编译后的运行时实例”

一句话理解今天的内容：

`deepagents` 不是在“创建一个模型”，而是在“创建一个可运行的 agent 应用”。

---

## 今日输出物

今天要产出两个东西。

### 输出物 1：装配流程图

至少画出：

- `create_deep_agent`
- model
- backend
- middleware
- subagents
- prompt
- `CompiledStateGraph`

### 输出物 2：300-500 字总结

题目建议：

`create_deep_agent() 是怎么把 Deep Agent 装起来的`

建议结构：

- 它的职责是什么
- 为什么它是 SDK 核心入口
- 它返回的是什么
- 为什么说它更像“应用装配器”而不是“工具函数”

---

## 推荐时间安排

### 第 1 段：15 分钟

读 `__init__.py`

目标：
搞清楚公开 API 面

### 第 2 段：45-60 分钟

精读 `graph.py`

目标：
把 agent 装配流程读顺

### 第 3 段：15-20 分钟

读 `_models.py` 和 `base_prompt.md`

目标：
理解模型注入和 prompt 设计

### 第 4 段：20-30 分钟

自己输出流程图和总结

---

## 今日验收标准

如果你今天学完，能不看源码，直接说清楚下面这段话，就算 Day 2 合格：

`create_deep_agent()` 是 `deepagents` SDK 的核心入口。它会解析模型、设置 backend、装配 planning/filesystem/subagent/summarization 等 middleware，并把 prompt、tools、memory、subagents 等能力统一接入，最后返回一个编译好的 `CompiledStateGraph`。所以它不是简单地创建一个模型实例，而是在创建一个完整可运行的 Deep Agent。

---

## 明天预告

Day 3 会继续往下走，重点进入 middleware 机制：

- `filesystem.py`
- `subagents.py`
- `summarization.py`
- `patch_tool_calls.py`

目标是彻底看懂：

- 为什么 `deepagents` 的能力是通过 middleware 叠加出来的
- 这些 middleware 各自负责什么
- 为什么这套设计让 agent 更“深”
