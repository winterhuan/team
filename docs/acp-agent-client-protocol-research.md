# Agent Client Protocol (ACP) 调研

> 调研日期：2026-07-02  
> 来源：[github.com/agentclientprotocol](https://github.com/agentclientprotocol)、[agentclientprotocol.com](https://agentclientprotocol.com/)

---

## 1. 是什么

**Agent Client Protocol (ACP)** 是标准化 **代码编辑器/IDE（Client）** 与 **编程 Agent（Agent）** 之间通信的开放协议，类比 [Language Server Protocol (LSP)](https://microsoft.github.io/language-server-protocol/) 之于语言服务器。

| 角色 | 定义 | 典型实现 |
|------|------|----------|
| **Client** | 用户交互界面，管理环境、权限、资源 | Zed、JetBrains、VS Code ACP 扩展、Neovim 插件 |
| **Agent** | 使用生成式 AI 自主修改代码的程序，通常作为 Client 子进程运行 | Claude Code、Codex CLI、Gemini CLI、Cursor CLI |

核心假设：**用户主要在编辑器里工作**，需要按需唤起 Agent 协助编码任务。

---

## 2. 解决什么问题

| 痛点 | ACP 的解法 |
|------|-----------|
| 每个编辑器要为每个 Agent 写定制集成 | 统一协议，Agent 实现一次、多 Client 可用 |
| Agent 只能服务少数编辑器 | Client 支持 ACP 即接入整个 Agent 生态 |
| 开发者被绑定在特定 Agent 的 UI | Client 与 Agent 解耦，可自由组合 |

与 MCP 的关系：**互补而非替代**。ACP 管「编辑器 ↔ Agent」会话层；MCP 管「Agent ↔ 工具/数据源」。ACP 在 prompt 阶段把用户配置的 MCP server 信息传给 Agent，由 Agent 直连 MCP。

---

## 3. GitHub 组织与仓库

组织主页：[https://github.com/agentclientprotocol](https://github.com/agentclientprotocol)

### 3.1 核心仓库

| 仓库 | Stars（约） | 说明 |
|------|------------|------|
| [agent-client-protocol](https://github.com/agentclientprotocol/agent-client-protocol) | 3.6k | 协议规范、JSON Schema、Rust schema crate |
| [claude-agent-acp](https://github.com/agentclientprotocol/claude-agent-acp) | 2.2k | Claude Agent SDK 的 ACP 适配器（`@agentclientprotocol/claude-agent-acp`） |
| [registry](https://github.com/agentclientprotocol/registry) | 269 | ACP 兼容 Agent 注册表（已稳定） |
| [codex-acp](https://github.com/agentclientprotocol/codex-acp) | 82 | OpenAI Codex CLI 的 ACP server |
| [typescript-sdk](https://github.com/agentclientprotocol/typescript-sdk) | 206 | `@agentclientprotocol/sdk` |
| [python-sdk](https://github.com/agentclientprotocol/python-sdk) | 279 | Python SDK |
| [rust-sdk](https://github.com/agentclientprotocol/rust-sdk) | 164 | `agent-client-protocol` crate |
| [kotlin-sdk](https://github.com/agentclientprotocol/kotlin-sdk) | 79 | Kotlin SDK（JVM） |
| [java-sdk](https://github.com/agentclientprotocol/java-sdk) | 55 | Java SDK |
| [acpr](https://github.com/agentclientprotocol/acpr) | 7 | 实验性 meta ACP agent → 详见 [acpr 项目分析](./acpr-project-analysis.md) |
| [meetings](https://github.com/agentclientprotocol/meetings) | 3 | 工作组会议纪要 |

许可：**Apache-2.0**，无需 CLA。

### 3.2 官方 SDK

| 语言 | 包名 | 用途 |
|------|------|------|
| TypeScript | `@agentclientprotocol/sdk` | Client / Agent 双向实现 |
| Python | `python-sdk` | Client / Agent |
| Rust | `agent-client-protocol`（运行时）、`agent-client-protocol-schema`（类型） | 生产集成优先用运行时 crate |
| Kotlin | `acp-kotlin` | JVM 目标 |
| Java | `java-sdk` | JVM |

社区库索引：[agentclientprotocol.com/libraries/community](https://agentclientprotocol.com/libraries/community)

---

## 4. 协议架构

### 4.1 设计哲学

1. **MCP-friendly** — 基于 JSON-RPC，尽可能复用 MCP 类型，减少重复建模
2. **UX-first** — 为 Agent 协作 UX 设计（diff 展示、plan、tool call 状态、权限请求）
3. **Trusted** — 面向「用户信任模型」场景；Client 控制本地文件、终端、MCP 访问

### 4.2 通信模型

遵循 [JSON-RPC 2.0](https://www.jsonrpc.org/specification)：

| 类型 | 方向 | 特点 |
|------|------|------|
| **Methods** | 双向 | 请求-响应，有 `id` |
| **Notifications** | 单向 | 无 `id`，不期待响应 |

命名约定：对象属性 `camelCase`；discriminator 字符串值 `snake_case`。

用户可读文本默认格式：**Markdown**（非 HTML）。

### 4.3 传输层（Transports）

| 传输 | 状态 | 说明 |
|------|------|------|
| **stdio** | 稳定、推荐 | Client 启动 Agent 子进程；NDJSON 按行分隔；`stderr` 仅日志 |
| **Streamable HTTP** | 草案中 | Transports Working Group 推进 |
| **WebSocket** | RFD 草案 | 远程 Agent 场景 |
| **Custom** | 允许 | 须保持 JSON-RPC 消息格式 |

stdio 约束：

- Agent **禁止**向 `stdout` 写非 ACP 消息
- Client **禁止**向 Agent `stdin` 写非 ACP 消息
- 消息以 `\n` 分隔，**不得**含内嵌换行

本地与远程：

- **本地 Agent**：子进程 + stdio（当前主路径）
- **远程 Agent**：HTTP/WebSocket（进行中，与 agentic 平台协作完善）

### 4.4 版本

| 概念 | 说明 |
|------|------|
| **Wire protocol version** | `initialize` 时 `protocolVersion` 协商；当前稳定版 **1** |
| **Schema release version** | 如 `schema-v1.17.0`（2026-06-29 最新）；描述 JSON Schema 制品，不等同于 wire 破坏性变更 |
| **v2 提案** | RFD 进行中：message chunks、tool call updates、prompt lifecycle 等 |

JSON Schema 产物：

- 源码：`schema/v1/`、`schema/v2/`
- 发布：GitHub Releases 附件（`schema-v*` tags）

---

## 5. 协议生命周期

### 5.1 连接建立

```
Client 启动 Agent 子进程
  → Client → Agent: initialize（protocolVersion + clientCapabilities）
  → Agent → Client: initialize response（agentCapabilities + authMethods）
  → 可选: authenticate
```

### 5.2 Session 管理

| 方法 | 方向 | 说明 |
|------|------|------|
| `session/new` | C→A | 创建新会话 |
| `session/load` | C→A | 恢复已有会话（需 `loadSession` capability） |
| `session/list` | 已稳定 | 发现历史会话 |
| `session/resume` | 已稳定 | 恢复会话 |
| `session/close` | 已稳定 | 关闭活跃会话 |
| `session/delete` | 可选 | 从历史删除 |
| `session/set_mode` | C→A | 切换 Agent 运行模式 |
| `session/set_config_option` | 已稳定 | 模型选择等配置项 |

一个连接可支持**多个并发 Session**（多线程思考）。

### 5.3 Prompt Turn（核心对话流）

```
Client → Agent: session/prompt（用户消息 ContentBlock[]）
Agent → Client: session/update（流式通知）
  ├── plan（执行计划条目）
  ├── agent_message_chunk（文本块）
  ├── tool_call / tool_call_update（工具调用及状态）
  ├── usage_update（token / cost）
  └── mode_change 等
Agent → Client: session/request_permission（敏感操作需人类批准）
Client → Agent: permission response
Agent → Client: session/prompt response（stopReason）
```

**StopReason**：`end_turn` | `max_tokens` | `max_turn_requests` | `refusal` | `cancelled`

**取消**：Client 发 `session/cancel` notification；Agent 须以 `cancelled` stopReason 响应（非 error）。

### 5.4 Agent 侧方法（基线 + 可选）

| 方法 | 类型 |
|------|------|
| `initialize` | 基线 |
| `authenticate` | 基线（按需） |
| `logout` | 可选（已稳定） |
| `session/new` | 基线 |
| `session/prompt` | 基线 |
| `session/load` | 可选 |
| `session/set_mode` | 可选 |

### 5.5 Client 侧方法（Agent 回调 Client）

| 方法 | Capability | 说明 |
|------|------------|------|
| `session/request_permission` | 基线 | 工具调用权限请求 |
| `fs/read_text_file` | `fs.readTextFile` | 读文件 |
| `fs/write_text_file` | `fs.writeTextFile` | 写文件 |
| `terminal/create` | `terminal` | 创建终端 |
| `terminal/output` | `terminal` | 读输出 |
| `terminal/wait_for_exit` | `terminal` | 等待退出 |
| `terminal/kill` | `terminal` | 终止命令 |
| `terminal/release` | `terminal` | 释放终端 |

文件路径**必须绝对路径**；行号 1-based。

### 5.6 内容与 UX 元素

| 元素 | 文档 |
|------|------|
| ContentBlock（text/image/audio/resource） | `protocol/v1/content` |
| Agent Plan | `protocol/v1/agent-plan` |
| Tool Calls | `protocol/v1/tool-calls` |
| Slash Commands | `protocol/v1/slash-commands` |
| Diff 展示 | 自定义类型，支持 deleted file 等 RFD |
| Extensibility | `_meta` 字段、`_` 前缀自定义方法 |

---

## 6. MCP 集成架构

```
用户 → Client（编辑器）
         ├─ 配置 MCP servers
         ├─ session/prompt 时把 MCP 配置传给 Agent
         └─ 可选：Client 自身 MCP 能力通过 proxy 暴露给 Agent

Agent 子进程
  ├─ 接收 prompt + MCP server 列表
  ├─ 直连用户 MCP servers（HTTP/SSE/stdio）
  └─ 需要 Client 资源时回调 fs/terminal/permission
```

Client 若需把自身能力当 MCP 工具暴露：通过 **MCP proxy** 隧道回 Client，避免 MCP 与 ACP 共用同一 socket。

相关 RFD：[MCP-over-ACP](https://agentclientprotocol.com/rfds/mcp-over-acp.md)

---

## 7. 生态概览

### 7.1 主要 Agent（部分）

来源：[agentclientprotocol.com/overview/agents](https://agentclientprotocol.com/overview/agents)

| Agent | 备注 |
|-------|------|
| Claude Agent | 经 `claude-agent-acp` 适配 |
| Codex CLI | 经 `codex-acp` 适配 |
| Gemini CLI | 原生 `--acp` |
| Cursor CLI | 官方 ACP 支持 |
| GitHub Copilot CLI | 2026-01 public preview |
| OpenCode / OpenHands / Goose / Kimi CLI / Qwen Code | 各原生或适配 |
| Cline / Factory Droid / Augment / Mistral Vibe | 商业/开源 Agent |

### 7.2 主要 Client（部分）

来源：[agentclientprotocol.com/overview/clients](https://agentclientprotocol.com/overview/clients)

| 类别 | 代表 |
|------|------|
| **编辑器/IDE** | Zed、JetBrains、VS Code ACP 扩展、Neovim（多插件）、Emacs |
| **CLI/TUI** | acpx、pool、Toad |
| **Desktop/Web** | ACP UI、Codeg、Jockey、Braide、Devin Desktop |
| **消息桥接** | Telegram/Discord/Slack/飞书/微信 ACP 桥 |
| **框架** | LangChain Deep Agents、Mastra、Koog、fast-agent、ACP Kit |

### 7.3 ACP Registry

- 仓库：[github.com/agentclientprotocol/registry](https://github.com/agentclientprotocol/registry)
- 状态：**已稳定**（2026 公告）
- 作用：策展可分发、支持认证的 ACP Agent 列表，便于 Client 发现安装

---

## 8. 治理与演进

### 8.1 治理结构

由 **Zed** 与 **JetBrains** 联合治理，目标过渡到独立基金会。

| 层级 | 职责 |
|------|------|
| Contributors | 提交 PR/Issue |
| Maintainers | 分管 SDK、文档等子项目 |
| Core Maintainers | 规范演进、任命 maintainer |
| Lead Maintainers (BDFL) | Ben Brandt (Zed)、Sergey Ignatov (JetBrains) |

变更流程：**RFD (Request for Dialog)** — 见 [rfds/about](https://agentclientprotocol.com/rfds/about.md)

### 8.2 近期稳定化能力（2026）

| 公告 | 内容 |
|------|------|
| Session List / Resume / Close | 会话发现与生命周期 |
| Session Config Options | 模型选择等配置 |
| Session Info Update | 会话元数据更新 |
| Logout Method | 认证登出 |
| ACP Registry | Agent 注册表稳定 |
| Transports WG | 新传输格式工作组 |

### 8.3 v2 方向（RFD）

- Message updates and chunks
- Tool call updates 细化
- v2 prompt lifecycle
- v2 client filesystem/terminal surface
- Plan variants

---

## 9. 与团队参考项目（Clowder AI）的关系

### 9.1 Clowder 中的 ACP 用法

Clowder AI（`references/clowder-ai`）将 ACP 作为 **CLI 传输协议之一**，角色是 **ACP Client**（平台扮演编辑器），Agent 是外部 CLI 子进程。

| 组件 | 路径 | 说明 |
|------|------|------|
| `AcpClient` | `packages/api/.../providers/acp/AcpClient.ts` | NDJSON-over-stdio，管理 spawn → initialize → session → prompt |
| `AcpAgentService` | `packages/api/.../providers/acp/AcpAgentService.ts` | 通用 `AgentService` 实现（F161 由 `GeminiAcpAdapter` 泛化） |
| `AcpProcessPool` | 同目录 | 进程池、session lease、idle TTL（F149） |
| `acp-event-transformer` | 同目录 | ACP `session/update` → 平台 `AgentMessage` 流 |

当前 ACP 载体：

| CLI | 启动方式 |
|-----|----------|
| Gemini CLI | `gemini --acp`（catalog 配了 `acp` section 时优先） |
| OpenCode | `opencode acp`（F161 规划接入） |

非 ACP 默认路径：Claude `stream-json`、Codex `exec --json`、Antigravity `agy --print`。

### 9.2 架构对比

| 维度 | ACP 标准场景 | Clowder AI |
|------|-------------|------------|
| **Client** | Zed/JetBrains 等编辑器 | Cat Cafe Web/API 平台 |
| **Agent** | Claude/Codex/Gemini CLI | 同上，通过 `AcpAgentService` 调用 |
| **传输** | stdio JSON-RPC | stdio NDJSON（`AcpClient`） |
| **权限** | Client `session/request_permission` | 平台 `AuthorizationManager` + MCP `request_permission` |
| **文件/终端** | Client 提供 `fs/*`、`terminal/*` | Agent 自带工具；平台通过 MCP callback 回写 |
| **MCP** | Client 传配置，Agent 直连 | 平台注入 `cat_cafe_*` MCP + whitelist（`acp-mcp-resolver`） |
| **多 Agent 协作** | 单 Client ↔ 单 Agent Session | Thread + A2A + Worklist（平台层，ACP 未覆盖） |

### 9.3 关键差异与启示

1. **ACP 是单用户单 Agent 会话协议**；Clowder 在 ACP 之上叠加了多猫路由、A2A worklist、SOP、记忆等平台能力。
2. **Clowder 是 ACP Client，不是 ACP Agent** — 不会把猫暴露给外部 Zed；方向与 F126「Limb」里「IDE 伸手进来用猫」相反。
3. **F161 ACP Carrier Generalization** 正在把 ACP 从 Gemini 硬编码提升为通用 `protocol: acp` 传输，模板化 env 映射。
4. **F210**：Google consumer Gemini CLI 将停服，非 ACP 路线默认切 Antigravity；有 ACP 配置仍走 `gemini --acp`。
5. 若未来要让 **外部 IDE 直接接入 Clowder 的猫**，需实现 **ACP Agent 端**（反向），属 F126 Phase C / F050 远期方向，与当前「平台作 Client 调外部 CLI」不同。

---

## 10. ACP vs 相关协议

| 协议 | 连接对象 | 粒度 | 类比 |
|------|----------|------|------|
| **LSP** | Editor ↔ Language Server | 符号/补全/诊断 | 语言智能 |
| **MCP** | Host ↔ Tools/Data | 工具调用/资源 | 能力供给 |
| **ACP** | Editor ↔ Coding Agent | 会话/计划/工具/权限/diff | Agent 协作 UX |
| **Cat Cafe MCP callback** | Agent CLI ↔ 平台 API | 消息/工作流/记忆写回 | 平台私有集成 |

ACP 不取代 MCP：Prompt 阶段把 MCP 配置交给 Agent，Agent 执行中同时使用 ACP（找 Client 要权限/文件）和 MCP（调工具）。

---

## 11. 调研结论

### 11.1 成熟度评估

| 维度 | 评估 |
|------|------|
| 规范完整度 | v1 核心流（init → session → prompt → tool → permission）已稳定；会话管理系列 2026 年陆续固化 |
| 生态活跃度 | 高 — 3.6k stars、60+ Agent、多编辑器/桥接/框架；Registry 已稳定 |
| 传输 | stdio 生产可用；远程 HTTP/WS 仍在 RFD |
| SDK | 5 语言官方 SDK + 社区库 |
| 治理 | Zed + JetBrains 双 BDFL，RFD 流程清晰 |

### 11.2 对团队的价值

| 场景 | 建议 |
|------|------|
| **继续用 ACP 调 Gemini/OpenCode** | 跟进 F161 通用 ACP carrier；复用 `AcpProcessPool` |
| **评估新 Agent 接入** | 查 [Registry](https://github.com/agentclientprotocol/registry) + 是否原生 `--acp` |
| **外部 IDE 接入猫** | 需另做 ACP Agent adapter（非当前主路径） |
| **协议升级** | 关注 v2 RFD 与 `schema-v*` releases；`protocolVersion` 协商 |

### 11.3 推荐跟进资源

| 资源 | URL |
|------|-----|
| 官方文档 | https://agentclientprotocol.com |
| 文档索引（LLM 友好） | https://agentclientprotocol.com/llms.txt |
| 协议 Schema v1 | https://github.com/agentclientprotocol/agent-client-protocol/tree/main/schema/v1 |
| 协议概览 | https://agentclientprotocol.com/protocol/v1/overview |
| Prompt Turn 详解 | https://agentclientprotocol.com/protocol/v1/prompt-turn |
| Agent 列表 | https://agentclientprotocol.com/overview/agents |
| Client 列表 | https://agentclientprotocol.com/overview/clients |
| RFD 流程 | https://agentclientprotocol.com/rfds/about |
| Clowder ACP 实现 | `references/clowder-ai/packages/api/src/domains/cats/services/agents/providers/acp/` |
| Clowder F161 规格 | `references/clowder-ai/docs/features/F161-acp-carrier-generalization.md` |

---

## 附录 A：协议方法速查

### Agent 实现（被 Client 调用）

```
initialize, authenticate, logout
session/new, session/load, session/list, session/resume, session/close, session/delete
session/prompt, session/cancel (notification), session/set_mode, session/set_config_option
```

### Client 实现（被 Agent 回调）

```
session/request_permission
fs/read_text_file, fs/write_text_file
terminal/create, terminal/output, terminal/wait_for_exit, terminal/kill, terminal/release
```

### Agent 通知 Client

```
session/update（plan | agent_message_chunk | tool_call | tool_call_update | usage_update | ...）
```

---

## 附录 B：Clowder `AcpClient` 生命周期（实现对照）

```
spawn(command, args=['--acp'], cwd, env)
  → initialize → 协商 protocolVersion + capabilities
  → newSession() / reuseSession(sessionId)
  → prompt(contentBlocks) 
      ← 流式 session/update notifications
      ← 可能触发 permissionHandler
  → stopReason
  → release lease → pool idle / eviction
```

失败分类（`AcpAgentService`）：`init_failure` | `prompt_failure` | `model_capacity` | `mcp_pollution` | `stream_idle_stall` | `turn_budget_exceeded`