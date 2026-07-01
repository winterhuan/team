# Clowder AI 参考分析文档

本目录整理自对 `references/` 中 **clowder-ai** 源码与 **cat-cafe-tutorials** 教程的深入分析，供团队快速理解平台架构与核心机制。

## 文档索引

| 文档 | 内容 |
|------|------|
| [clowder-ai-architecture-analysis.md](./clowder-ai-architecture-analysis.md) | 完整架构分析：Monorepo、Agent 调用、A2A、Session、记忆、Skills/SOP/Governance、Mission Hub |

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
| 串行执行 | `packages/api/src/domains/cats/services/agents/routing/route-serial.ts` |
| CLI 子进程 | `packages/api/src/utils/cli-spawn.ts` |
| MCP 回调 | `packages/mcp-server/src/tools/callback-tools.ts` |
| SOP 定义 | `sop-definitions/development.yaml` |
| Skills 内容 | `cat-cafe-skills/` |
| Mission Hub UI | `packages/web/src/components/mission-control/` |
| ADR-001 | `docs/decisions/001-agent-invocation-approach.md` |
| F073 SOP 告示牌 | `docs/features/F073-sop-auto-guardian.md` |