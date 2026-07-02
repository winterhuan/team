# Clowder AI 参考分析文档

本目录整理自对 `references/` 中 **clowder-ai** 源码与 **cat-cafe-tutorials** 教程的深入分析，供团队快速理解平台架构与核心机制。

## 文档索引

| 文档 | 内容 |
|------|------|
| [clowder-ai-architecture-analysis.md](./clowder-ai-architecture-analysis.md) | **完整架构分析**（v2 全量补全，45 章 + 附录） |
| [acp-agent-client-protocol-research.md](./acp-agent-client-protocol-research.md) | **ACP 协议调研**（Agent Client Protocol 生态、协议机制、与 Clowder 关系） |
| [acpr-project-analysis.md](./acpr-project-analysis.md) | **acpr 项目分析**（Registry 启动器、Meta Agent、与 Clowder F161 对比） |
| [agent-team-build-plan.md](./agent-team-build-plan.md) | **Agent Team 建设方案**（ACP+Clowder 融合、团队角色、6 阶段路线图） |
| [cat-cafe-tutorials-to-clowder-ai-mapping.md](./cat-cafe-tutorials-to-clowder-ai-mapping.md) | **教程→实现对照**（16 课 + DEMO 27 项 + 作业 + Feature 索引） |

### 架构文档章节速查

| 范围 | 章节 | 主题 |
|------|------|------|
| **核心链路** | §1–§7 | Monorepo、CLI 调用、U2A/A2A、Worklist、Session、MCP 回调 |
| **治理与 Hub** | §8–§10 | 记忆系统、Skills/SOP/Governance、Mission Hub |
| **前端与模式** | §11–§14 | 前端架构、设计模式、教程对照、设计决策 |
| **平台基础设施** | §15–§23 | 连接器、排队、认证、球权、调度、MCP 工具、Desktop、审批、插件 |
| **协作与引导** | §24–§26 | 协同全景/TeamAct、Guides/Bootcamp、Concierge |
| **能力扩展** | §27–§34 | Signals、World、Limb、Terminal、Preview、Community、Pack、Projects |
| **运维与治理** | §35–§42 | Governance Bootstrap、External Runtime、Services、Workspace、Harness、Agent Key、Action Plane、Code Intelligence |
| **索引与存储** | §43–§45 + 附录 | 体验层、数据存储、ADR/Cells 索引、术语表 |

## 参考源码位置

```
references/
├── clowder-ai/          # 主源码（pnpm monorepo）
└── cat-cafe-tutorials/  # 配套教程（文档，无应用源码）
```

- 开源仓库：[github.com/zts212653/clowder-ai](https://github.com/zts212653/clowder-ai)
- 教程仓库：[github.com/zts212653/cat-cafe-tutorials](https://github.com/zts212653/cat-cafe-tutorials)

## 关键源码入口（clowder-ai）

| 主题 | 路径 |
|------|------|
| API 入口 | `packages/api/src/index.ts` |
| Agent 路由 | `packages/api/src/domains/cats/services/agents/routing/AgentRouter.ts` |
| A2A Worklist | `packages/api/src/domains/cats/services/agents/routing/WorklistRegistry.ts` |
| 排队续跑 | `packages/api/src/domains/cats/services/agents/invocation/QueueProcessor.ts` |
| IM 连接器 | `packages/api/src/infrastructure/connectors/ConnectorRouter.ts` |
| 球权引擎 | `packages/api/src/domains/ball-custody/` |
| 统一调度 | `packages/api/src/infrastructure/scheduler/TaskRunnerV2.ts` |
| MCP 工具策略 | `packages/mcp-server/src/server-toolsets.ts` |
| CLI 子进程 | `packages/api/src/utils/cli-spawn.ts` |
| MCP 回调 | `packages/mcp-server/src/tools/callback-tools.ts` |
| SOP 定义 | `sop-definitions/development.yaml` |
| Skills 内容 | `cat-cafe-skills/` |
| Mission Hub UI | `packages/web/src/components/mission-control/` |
| Desktop 壳 | `desktop/main.js` |
| 协同全景（上游） | `docs/architecture/collaboration-landscape.md` |
| 记忆全景（上游） | `docs/architecture/memory-system-overview.md` |
| Ownership Cells | `docs/architecture/ownership/README.md` |
| ADR-001 | `docs/decisions/001-agent-invocation-approach.md` |
| F073 SOP 告示牌 | `docs/features/F073-sop-auto-guardian.md` |