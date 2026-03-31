# Claude Code 仓库文档导航

这套文档的目标不是做“目录罗列”，而是把这个仓库拆成可理解、可维护、可继续研究的模块地图。阅读时建议先看总览，再按运行时主链路进入各子模块。

## 建议阅读顺序

1. [仓库总报告](./repository-report.md)
2. [总体架构总览](./architecture-overview.md)
3. [启动与装配链路](./runtime/entry-and-boot.md)
4. [对话与执行运行时](./runtime/conversation-runtime.md)
5. 各模块详细文档

## 文档结构

- `repository-report.md`
  - 仓库定位、规模、核心判断、主要风险。
- `architecture-overview.md`
  - 整体架构分层、主调用链、模块关系、扩展机制。
- `runtime/`
  - 根运行时文件和主链路说明。
- `modules/`
  - 按 `src` 顶层模块拆分的详细文档。

## 模块文档索引

### 核心运行时与入口

- [entrypoints](./modules/entrypoints.md)
- [bootstrap](./modules/bootstrap.md)
- [screens](./modules/screens.md)
- [state](./modules/state.md)
- [query](./modules/query-subsystem.md)
- [tools](./modules/tools.md)
- [services](./modules/services.md)
- [tasks](./modules/tasks.md)

### 交互层与终端 UI

- [cli](./modules/cli.md)
- [components](./modules/components.md)
- [hooks](./modules/hooks.md)
- [ink](./modules/ink.md)
- [context](./modules/context.md)
- [keybindings](./modules/keybindings.md)
- [vim](./modules/vim.md)
- [buddy](./modules/buddy.md)
- [voice](./modules/voice.md)

### 扩展与能力边界

- [commands](./modules/commands.md)
- [skills](./modules/skills.md)
- [plugins](./modules/plugins.md)
- [remote](./modules/remote.md)
- [bridge](./modules/bridge.md)
- [server](./modules/server.md)
- [upstreamproxy](./modules/upstreamproxy.md)
- [schemas](./modules/schemas.md)

### 基础设施与公共能力

- [constants](./modules/constants.md)
- [assistant](./modules/assistant.md)
- [coordinator](./modules/coordinator.md)
- [memdir](./modules/memdir.md)
- [migrations](./modules/migrations.md)
- [native-ts](./modules/native-ts.md)
- [outputStyles](./modules/output-styles.md)
- [types](./modules/types.md)
- [utils](./modules/utils.md)
- [moreright](./modules/moreright.md)

## 阅读建议

- 如果你关注“Claude Code 是怎么跑起来的”，先读 `entrypoints`、`bootstrap`、`screens`、`state`、`services`。
- 如果你关注“模型如何调工具”，先读 `runtime/conversation-runtime.md`、`tools`、`services`、`tasks`、`query`。
- 如果你关注“如何扩展”，先读 `skills`、`plugins`、`commands`、`services/mcp` 对应的 `services` 章节。
- 如果你关注“为什么这个仓库复杂”，优先看 `utils`、`services`、`components`、`commands` 四个模块。
