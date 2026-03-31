# remote 模块

## 模块定位

`src/remote` 负责“本地客户端如何管理远程 Claude 会话”的那一层逻辑，尤其是会话订阅、消息转发和权限桥接。

规模统计：

- 文件数：4
- 行数：1127
- 代表文件：
  - `remote/SessionsWebSocket.ts`，404 行
  - `remote/RemoteSessionManager.ts`，343 行
  - `remote/sdkMessageAdapter.ts`，302 行
  - `remote/remotePermissionBridge.ts`，78 行

## 模块职责

### 1. 远程会话连接管理

`RemoteSessionManager.ts` 明确负责：

- 建立 WebSocket 订阅
- 发送用户消息
- 处理 control request / response
- 管理 permission request 生命周期

### 2. WebSocket 封装

`SessionsWebSocket.ts` 负责底层远程消息通道。

### 3. SDK 消息适配

`sdkMessageAdapter.ts` 负责把远端事件适配成本地可消费的 SDKMessage 语义。

### 4. 权限桥接

`remotePermissionBridge.ts` 把远端会话里的权限请求和本地审批机制接起来。

## 与 `bridge` 的区别

- `remote`：本地如何连接并管理一个远端 session
- `bridge`：本地如何充当远端可控环境/可桥接环境

前者偏“客户端视角”，后者偏“环境/控制面视角”。

## 与其他模块的关系

- `cli/transports/*`
- `bridge/*`
- `services/mcp/*` 和 `tools`：远端会话最终仍会回到同一 runtime 协议
- `screens/REPL.tsx`：远程模式会影响状态展示和交互能力

## 工程判断

这个目录体量不大，但边界清晰，是一个比较健康的“远程会话客户端”切面。

## 维护点

- 需要谨慎处理 control message 和普通 SDKMessage 的区分。
- viewer-only 场景和可交互场景行为不同，不能混为一谈。
