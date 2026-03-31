# services 模块

## 模块定位

`src/services` 是 Claude Code 的业务执行中台。它负责模型调用、MCP、紧凑化、分析埋点、插件安装、内存/提示建议、远端设置同步等核心执行语义。

规模统计：

- 文件数：130
- 行数：53683
- 代表文件：
  - `services/api/claude.ts`，3419 行
  - `services/mcp/client.ts`，3348 行
  - `services/mcp/auth.ts`，2465 行
  - `services/tools/toolExecution.ts`，1745 行
  - `services/compact/compact.ts`，1705 行

## 为什么 `services` 是关键中台

如果说：

- `components` 代表产品界面
- `tools` 代表模型能力面
- `state` 代表会话状态容器

那么 `services` 就代表真正把“能力变成可执行行为”的那层业务语义。

这里承担了大量不适合放进 UI、也不适合放进纯工具函数的逻辑。

## 子目录地图

### 模型与执行

- `api`，20 文件，10477 行
- `tools`，4 文件，3113 行
- `compact`，11 文件，3960 行

这是最接近 agent runtime 主链路的一组服务。

### 外部协议与扩展

- `mcp`，23 文件，12311 行
- `oauth`，5 文件，1051 行
- `plugins`，3 文件，1616 行
- `lsp`，7 文件，2460 行

这是系统与外部能力连接最紧密的一组。

### 体验与辅助能力

- `PromptSuggestion`，2 文件，1514 行
- `SessionMemory`，3 文件，1026 行
- `AgentSummary`，1 文件，179 行
- `MagicDocs`，2 文件，381 行
- `extractMemories`，2 文件，769 行
- `teamMemorySync`，5 文件，2167 行
- `tips`，3 文件，761 行
- `toolUseSummary`，1 文件，112 行
- `autoDream`，4 文件，550 行

### 控制面与同步

- `analytics`，9 文件，4040 行
- `policyLimits`，2 文件，690 行
- `remoteManagedSettings`，5 文件，951 行
- `settingsSync`，2 文件，648 行

## 核心子系统详解

### 1. `services/api`

这是模型 API 层。

其中最关键的是 `api/claude.ts`，从文件头可直接看出它要处理：

- message/tool schema 转换
- streaming / non-streaming
- beta headers
- structured outputs
- thinking / effort
- prompt caching
- task budget
- usage 统计
- retry / timeout / fallback

这个文件之所以是热点，不是因为“发 HTTP 很复杂”，而是 Claude Code 已经把模型调用做成一套产品化 runtime。

### 2. `services/mcp`

这是整个仓库最重要的扩展面之一。

目录体量甚至超过 `api`，说明 MCP 在这个产品里不是附属功能。它承担：

- transport 连接
- auth 处理
- tool/command/resource 抓取
- session 过期与重连
- elicitation hooks
- MCP tool 到 Claude Code tool 的适配

`mcp/client.ts` 与 `mcp/auth.ts` 两个大文件构成了完整的 MCP 集成骨架。

### 3. `services/tools`

这里不是实现每个 tool，而是定义“工具如何被运行”。

例如：

- `toolExecution.ts`
- `toolOrchestration.ts`
- `StreamingToolExecutor.ts`

其中 `toolOrchestration.ts` 的一个重要设计是：

- 并发安全工具批量并发执行
- 非并发安全工具串行执行

这说明工具执行被显式建模成编排问题，而不是简单 `for await` 调用。

### 4. `services/compact`

Claude Code 不把 compact 当“上下文不够了临时压缩一下”，而是正式的运行时能力。

这里负责：

- 普通 compact
- auto compact
- reactive compact
- snip / microcompact
- API context management

这个目录对长会话稳定性至关重要。

### 5. `services/analytics`

analytics 不是单点埋点，而是：

- event logging
- GrowthBook gate
- Datadog / first-party logging
- metadata 规范化

因为整个产品 heavily feature-gated，analytics/growthbook 已经深入主链路。

### 6. `services/PromptSuggestion`

提示建议在 Claude Code 里不是简单 UI 糖衣，而是实际的交互增强能力，因此有自己的服务目录。

### 7. `services/SessionMemory` 与 `teamMemorySync`

说明“记忆”不仅是 prompt 里的静态文件，还包括会话层与团队层的同步逻辑。

### 8. `services/plugins`

这个目录不大，但职责明确：更偏插件安装、后台 reconcile 和系统层插件管理，而不是 UI。

## 与其他模块的关系

- `QueryEngine` / `query.ts`：直接依赖 `services/api`、`services/tools`、`services/compact`
- `tools/*`：具体工具调用会落到 `services/tools`
- `commands/*`：很多 slash command 的业务实现在这里落地
- `components/*`：UI 会消费 analytics、PromptSuggestion、plugin 状态、MCP 状态
- `state`：许多 service 会读写 AppState 或其衍生数据

## 一个重要判断

Claude Code 的“业务中台”并没有完全统一到 `services`，因为很多复杂逻辑也在 `utils`。但 `services` 仍然是最接近“正式业务子系统”的目录。

## 维护风险

- `api/claude.ts`、`mcp/client.ts`、`mcp/auth.ts` 已经是典型超大中枢文件。
- `services` 与 `utils` 的边界不是绝对清晰，后续需要防止职责继续漂移。
- `compact`、`MCP`、`analytics/growthbook` 都是横切能力，改动容易影响主链路。

## 阅读建议

如果你从执行链路理解仓库，推荐顺序：

1. `services/api/claude.ts`
2. `services/tools/*`
3. `services/compact/*`
4. `services/mcp/*`
5. `services/plugins/*`
6. `services/analytics/*`
