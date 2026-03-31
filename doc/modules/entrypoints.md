# entrypoints 模块

## 模块定位

`src/entrypoints` 是系统对外和对启动层暴露的“契约入口层”。它既包含 CLI 真入口，也包含 SDK 所需的 schema 和类型约束。

规模统计：

- 文件数：8
- 行数：4052
- 子目录：`sdk`
- 代表文件：
  - `entrypoints/sdk/coreSchemas.ts`，1889 行
  - `entrypoints/sdk/controlSchemas.ts`，663 行
  - `entrypoints/agentSdkTypes.ts`，443 行
  - `entrypoints/init.ts`，340 行
  - `entrypoints/cli.tsx`，303 行

## 模块职责

### 1. 真正的 CLI 启动入口

`cli.tsx` 是整个应用的第一入口，负责 fast-path 判断和最早期的 dynamic import 分流。

### 2. 初始化边界

`init.ts` 负责全局初始化中那些应被多个入口共享的逻辑，比如 telemetry trust 后初始化等。

### 3. SDK 契约定义

`sdk/coreSchemas.ts` 与 `sdk/controlSchemas.ts` 体量很大，说明 SDK 输出/控制协议不是随手拼 JSON，而是显式 schema 化的。

### 4. 类型桥接

`agentSdkTypes.ts` 把 SDK、runtime、events、status 之间的共享类型集中起来。

## 为什么这个模块不是简单入口目录

这个目录承担了两种非常不同的边界：

- 进程入口边界
- 外部程序集成边界

两者放在一起是合理的，因为它们都属于“系统从哪里进、外部如何看见它”的问题。

## 关键工程信号

### `sdk/coreSchemas.ts` 很大

说明 Claude Code 已经不是“只供终端手工使用”的工具，而是在认真维护一套可编程接口。

### `cli.tsx` 很轻

说明启动性能被强约束，入口层不能膨胀。

## 与其他模块的关系

- 对下连接 `main.tsx` 和整个应用装配层
- 对外连接 SDK/automation/desktop/remote 等调用方
- 与 `types`、`services/api`、`QueryEngine` 共同定义 headless 可编程语义

## 维护建议

- 启动逻辑应继续保持轻量，不要把业务细节塞进 `cli.tsx`
- SDK schema 变更必须被视为接口变更，而不是内部重构
- `entrypoints` 最适合承载协议和初始化边界，不适合承载产品细节
