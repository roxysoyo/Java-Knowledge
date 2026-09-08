# Agent八股

ai coding流程？怎么确保生成的东西一定是对的？

为什么 dsh 要把它做成插件化

dsh 插件化核心是怎么实现的

Cordis 有哪些思想可以借鉴

多 Agent 并发有什么性能问题？ 讲一下长时记忆和短时记忆的底层实现方式？ 讲一下 ReAct 的流程？

了解哪些 Agent 的架构模式？ 

Function Calling 的原理是什么？你怎么理解 Tool Use？ 如果大模型返回的 Function Call 参数格式不对怎么办？

意图识别准确率怎么算

怎么解决大量工具间互相干扰导致的路由问题

怎么设计一个To C的LLM as Judge的评分体系

Plan and Execute 和 ReAct这两种范式本质差异是什么

Agent工具输出很差，怎么定位是哪个环节的问题



## 记忆

不同生命周期的数据存储 + Retrieval。

### 短时记忆 Short-Term Memory

最简单的实现其实就是：

```
messages = [
    system_message,
    user_message,
    assistant_message,
    tool_result,
    ...
]
```

也就是当前模型的：

```
Context Window
```

所以最基础的短时记忆：

```
Short-term Memory
=
Conversation History
+
Current Tool Results
+
Current Plan
+
Current State
```

底层通常直接放：

```
RAM
Python object
Redis
Session Store
```



### 长时记忆 Long-Term Memory

长时记忆和短时记忆最大的区别：

```
短时：
当前 Session 内有效

长时：
跨 Session / 跨任务仍然存在
```

底层一般不是简单 messages，而是：

```
Persistent Storage
+
Index
+
Retriever
```

典型架构：

```
                     Long-Term Memory
                            │
             ┌──────────────┼──────────────┐
             │              │              │
          Metadata       Vector DB       SQL/KV
             │              │              │
             └──────────────┼──────────────┘
                            │
                         Retriever
                            │
                            ▼
                         Context
```

最常见的方法就是：

Embedding + Vector Database







## 工具调用 Tool Use

### 基础概念

Tool Use 是更大的概念：

> **让模型通过外部能力与真实世界或者外部环境交互。**

比如：

```
Tool Use
├── Function Calling
├── Bash
├── File System
├── Browser
├── Database
├── Search
├── Python
├── MCP
├── REST API
└── Computer Use
```

核心在于

```
参数校验
权限控制
超时
重试
幂等
并发
异常处理
Sandbox
日志
Tracing
结果截断
结果压缩
```

一个比较完整的 Tool Calling Runtime

```
                LLM
                 │
                 ▼
             Tool Call
                 │
                 ▼
          ┌──────────────┐
          │ Tool Registry │
          └───────┬──────┘
                  │
                  ▼
             JSON Parser
                  │
                  ▼
          Schema Validator
                  │
                  ▼
         Semantic Validator
                  │
                  ▼
         Permission / Policy
                  │
                  ▼
              Sandbox
                  │
                  ▼
            Tool Execute
                  │
          ┌───────┴────────┐
          │                │
        Result           Error
          │                │
          └───────┬────────┘
                  ▼
              Observation
                  │
                  ▼
                 LLM
```

同时需要区分

```
Parse Error // json解析
Schema Error // tool schema解析
Semantic Error // 参数语义错误，请求的是天气，输出的是城市
Execution Error // 执行错误，文件未找到等等
```



### Function Calling

#### 原理

假设给模型注册一个工具：

```
{
  "name": "get_weather",
  "description": "查询某个城市天气",
  "parameters": {
    "type": "object",
    "properties": {
      "city": {
        "type": "string"
      }
    },
    "required": ["city"]
  }
}
```

用户问：

```
北京今天天气怎么样？
```

模型不会直接执行：

```
get_weather("北京")
```

而是生成类似：

```
{
  "name": "get_weather",
  "arguments": {
    "city": "北京"
  }
}
```

然后 Harness / Runtime 解析：

```
call = response.tool_call

if call.name == "get_weather":
    result = get_weather(**call.arguments)
```

函数真正执行完以后：

```
天气：晴，28℃
```

再作为 Tool Result 塞回模型：

```
User
 ↓
LLM
 ↓
Tool Call
 ↓
Runtime
 ↓
Function
 ↓
Tool Result
 ↓
LLM
 ↓
Final Answer
```

所以完整闭环是：

```
1. 把 Tool Schema 提供给 LLM
2. LLM 判断是否调用工具
3. LLM 选择工具
4. LLM 生成结构化参数
5. Runtime 校验参数
6. Runtime 调用真实函数
7. Tool Result 返回 Context
8. LLM 根据结果继续推理
```









### MCP

#### MCP 解决什么问题

传统方式下，每接一个工具，Agent 应用可能都要自己：

- 定义工具 Schema
- 编写调用逻辑
- 处理参数转换
- 管理鉴权
- 处理错误
- 维护不同工具的接口适配

工具一多，Agent 和工具之间就会形成大量定制化代码。

**MCP 是一个标准化的 Agent 与外部工具、数据和能力之间的连接协议，让模型应用可以用统一的方式发现和调用外部能力。**

#### MCP 到底是什么

MCP 全称是 **Model Context Protocol，模型上下文协议**。是一套 **标准化协议 + Client/Server 架构**，规定了 AI 应用如何与外部能力进行交互

**Host**

就是承载 Agent 的 AI 应用，比如一个 IDE、桌面 AI 应用或者你自己开发的 Agent。

**MCP Client**

Host 内部负责和 MCP Server 通信的客户端。

**MCP Server**

把具体的外部能力暴露出来。

**Host 管理 Agent，Client 负责通信，Server 提供能力。**

#### MCP 工作流程

```
用户
 ↓
Agent / LLM
 ↓
发现 MCP Server 提供的工具
 ↓
选择 execute_query
 ↓
MCP Client
 ↓
MCP Server
 ↓
MySQL
 ↓
查询结果
 ↓
MCP Server
 ↓
MCP Client
 ↓
Agent / LLM
 ↓
最终回答
```

#### MCP 与Function Calling

|            | Function Calling               | MCP                          |
| ---------- | ------------------------------ | ---------------------------- |
| 本质       | 模型调用工具的一种机制         | 工具/上下文连接协议          |
| 解决问题   | 模型如何表达“我要调用这个函数” | Agent 如何标准化连接外部能力 |
| 关注点     | Tool Call                      | Client ↔ Server              |
| 工具发现   | 通常由应用提供                 | MCP 支持标准化发现           |
| 跨应用复用 | 较弱                           | 更强                         |
| 标准化程度 | 模型/API层面的能力             | 整套连接协议                 |

**Function Calling 解决的是“模型怎么调用工具”，MCP 解决的是“模型应用怎么标准化地连接和使用外部工具与数据源”。**

#### MCP 的价值

**标准化**

* Agent → MCP → 各种 MCP Server

**解耦**

* Server负责：鉴权、参数转换、API调用、异常处理
* Agent负责：任务理解、工具选择、参数生成、结果分析

**复用**

* 一个 MCP Server 可以被不同的 AI Host 使用。

**生态化**



#### MCP 的传输协议

简单来说，MCP协议通过**传输层**来负责客户端与服务器之间的实际通信，目前有 `stdio` 和 `Streamable HTTP` 两种标准方式。

| 传输机制                 | 核心特点                                                     | 主要应用场景                                                 |
| :----------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Stdio 传输**           | 通过**标准输入/输出流**进行通信。性能最佳，无网络开销，**仅支持本地进程间通信**。 | 本地集成、命令行工具、与shell脚本协作等需要**最高性能**的本地场景。 |
| **Streamable HTTP 传输** | 基于**标准HTTP协议**，使用POST请求和（可选的）服务器发送事件（SSE）实现流式功能。**支持远程服务器通信**和标准HTTP认证（如OAuth）。 | 网络服务、需要远程访问或客户端与服务器解耦部署的场景。       |

#### MCP特别多怎么办

- 部署统一MCP网关，网关提供服务发现、负载均衡、访问控制
- 按场景分组
- 权限分级管理
- 动态激活



## 场景问题

### 有用量限制的AI服务

> 做一个有用量限制的AI服务该怎么做？ 什么时间去截断用户的请求？如果请求结束后才知道token超限该怎么办？

如果要设计一个有用量限制的 AI 服务，不会简单地在 API 层做 token 截断，而是会把它设计成一个完整的**资源预算管理系统（Resource Budget Management System）**。

因为在实际的 AI 服务中，资源消耗不仅仅是 token，还包括请求次数、模型调用次数、工具调用次数、上下文长度以及最终的计算成本。所以核心需要解决几个问题：

1. 如何准确统计用户消耗了多少资源；
2. 如何提前控制用户是否还能继续使用；
3. 在 Agent 场景下，如何控制一次复杂任务的资源预算；
4. 当资源超限时，如何保证系统能够优雅降级，而不是直接失败。

整体架构上，我会拆成几个模块：

- Usage Metering：负责资源计量；
- Quota System：负责用户额度管理；
- Budget Manager：负责单次任务预算控制；
- Context Manager：负责上下文管理和压缩；
- Runtime Monitor：负责运行过程中的实时监控；
- Billing 和 Settlement：负责最终结算。

**Usage Metering**

每一次 AI 请求，包括普通 Chat 请求或者 Agent 多步骤任务，都需要记录完整的资源消耗。

例如：用户 ID；请求 ID；使用的模型；输入 token 数；输出 token 数；总 token 数；工具调用次数；执行时间；估算成本。

比如一次请求：

用户输入消耗 20k token，模型输出 5k token，调用了两个工具，那么这些信息都会记录下来。

这样系统可以知道：用户今天已经用了多少 token；当前任务消耗了多少成本；是否接近限制。

**Quota System**

Quota 主要解决的问题是：

> 这个用户有没有权限继续消耗资源。

```
用户等级：
free

资源类型：
token

限制：
100000

周期：
daily
```

当用户发送请求时，首先不会直接调用模型，而是进入 Quota Check。除了总额度限制，还需要考虑 Rate Limit。因为即使用户每天额度很高，也可能在短时间内大量请求，造成系统压力。

**Budget Manager**

一个任务开始时分配：

```
max_total_tokens:
100k

max_steps:
20

max_tool_calls:
10

max_cost:
1美元
```

Agent 每执行一步之前，都检查：当前剩余预算是否足够

例如：当前已经消耗：

```
token:
80k

steps:
15
```

下一步预计需要：

```
30k token
```

那么系统判断：继续执行会超过预算。

这时候可以：停止任务；压缩上下文；降低模型；返回部分结果。这样可以避免 Agent 无限循环导致成本失控。



**Context Manager**

第一阶段：请求进入时进行预估。

也就是在真正调用模型之前。例如用户上传一个 100 页 PDF，让模型总结。系统首先估算：输入可能达到 500k token。但是：用户剩余额度只有 100k。那么没有必要继续执行。可以：拒绝请求；引导用户拆分文件；先做摘要；分阶段处理。这个阶段主要是避免明显超限。

第二阶段：Prompt 构建阶段进行 Context Management。

这是 Agent 场景最重要的一层。因为最终输入给模型的不只是用户问题，还包括：System Prompt；历史对话；Memory；RAG 检索结果；Tool 返回结果；Agent 中间状态。 按重要程度进行上下文压缩。

第三阶段：生成过程中实时控制。

因为模型输出长度无法完全预测。

所以 Streaming 输出时，需要实时统计 output token。





你怎样理解 AI Coding？它能替代开发者完成哪些工作，不能替代哪些工作？ 使用 Coding Agent 开发复杂功能时，如果上下文过长或模型产生幻觉，怎样把任务拉回正确方向？