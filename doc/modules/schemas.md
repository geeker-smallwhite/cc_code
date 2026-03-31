# schemas 模块

## 模块定位

`src/schemas` 是仓库中显式的 schema 定义目录。当前快照里只看到一个文件：`schemas/hooks.ts`。

规模统计：

- 文件数：1
- 行数：222
- 核心文件：`schemas/hooks.ts`

## 模块职责

### 1. hooks 配置/协议校验

从文件名可以判断，这里承接 hooks 相关配置结构的 schema 定义。

### 2. 作为“可验证契约层”

相比把结构直接写成 TypeScript type，schema 更适合：

- 运行时校验
- 配置文件校验
- 更好的错误提示

## 为什么这个目录值得存在

Claude Code 的 hook 体系影响面很大：

- 启动 hook
- stop/failure hooks
- 文件变化相关 hook

因此把 hooks schema 独立出来，是在为扩展能力提供稳固契约。

## 与其他模块的关系

- `utils/hooks/*`
- `setup.ts`
- `query/stopHooks.ts`
- `services` 中依赖 hook 行为的部分

## 工程判断

当前快照里 `schemas` 很小，但这不代表 schema 使用少，更可能说明大量协议目前分散在别处。如果未来要提升可维护性，把更多运行时协议向 schema 层集中会是一个合理方向。
