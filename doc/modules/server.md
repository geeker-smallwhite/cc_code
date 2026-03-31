# server 模块

## 模块定位

`src/server` 是 direct-connect 相关的服务端侧支撑目录，当前主要承接会话创建和管理类型定义。

规模统计：

- 文件数：3
- 行数：358
- 代表文件：
  - `server/directConnectManager.ts`，213 行
  - `server/createDirectConnectSession.ts`，88 行
  - `server/types.ts`，57 行

## 模块职责

### 1. direct-connect session 创建

`createDirectConnectSession.ts` 从命名看就是 `cc://` / `cc+unix://` 这条链路的真正会话创建点。

### 2. direct connect 生命周期管理

`directConnectManager.ts` 承接会话级资源和生命周期。

### 3. 类型定义

`types.ts` 保持 direct-connect 服务端能力的结构清晰。

## 在系统中的位置

这个目录本身不大，但和启动层关系很近：

- `main.tsx` 会早期识别 `cc://`
- 后续会进入 `server/*` 来解析和创建连接会话

## 与其他模块的关系

- `remote`：会话与消息层适配
- `cli/transports/*`：实际传输
- `bridge`：都属于远程/连接能力，但 direct-connect 更偏点对点接入

## 工程价值

单独的 `server` 目录说明 direct-connect 被当作正式能力，而不是临时协议 hack。

## 维护点

- 该模块很靠近网络/认证/权限边界，文档和类型必须保持清晰。
- 如果 direct-connect 协议扩大，优先继续在这里扩张，而不是把逻辑散落到 `main.tsx`。
