# query 子模块

## 模块定位

`src/query` 是 `src/query.ts` 的辅助子模块目录，承接 query loop 的拆分逻辑。

规模统计：

- 文件数：4
- 行数：652
- 代表文件：
  - `query/stopHooks.ts`，473 行
  - `query/tokenBudget.ts`，93 行
  - `query/config.ts`，46 行
  - `query/deps.ts`，40 行

## 模块职责

### 1. stop hooks 处理

`stopHooks.ts` 体量最大，说明“何时停止、停止后怎么触发 hook”在 Claude Code 里不是小逻辑，而是 query loop 的正式组成部分。

### 2. token budget 计算

`tokenBudget.ts` 负责 query loop 的预算控制辅助。

### 3. config 快照与依赖注入

`config.ts`、`deps.ts` 把 query loop 需要的稳定配置和可替换依赖显式化，这很有利于测试与运行时隔离。

## 为什么它值得单独成目录

`query.ts` 已经是 1700+ 行的大文件。如果不把 stop hooks、budget、deps 拆出来，这个文件会更快失控。

## 与其他模块的关系

- `QueryEngine.ts`：调用入口
- `services/compact/*`：压缩策略
- `services/tools/*`：工具执行
- `services/api/claude.ts`：模型调用
- `utils/hooks/*`：hook 执行

## 工程观察

这个子目录说明作者已经在对 query 主循环做“外围剥离”，但主循环本身仍然很重。未来如果继续演进，最可能进一步拆出的区域仍然是：

- tool-use 状态转移
- error recovery
- compact 相关状态机
