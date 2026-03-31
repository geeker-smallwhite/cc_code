# hooks 模块

## 模块定位

`src/hooks` 是 Claude Code 的行为胶水层，用 React hook 的方式把 UI、状态、权限、bridge、voice、suggestion 等能力组织起来。

规模统计：

- 文件数：104
- 行数：19232
- 子目录：
  - `notifs`，16 文件，1355 行
  - `toolPermission`，5 文件，1386 行
- 代表文件：
  - `hooks/useTypeahead.tsx`，1385 行
  - `hooks/useVoice.ts`，1144 行
  - `hooks/useInboxPoller.ts`，969 行
  - `hooks/fileSuggestions.ts`，811 行
  - `hooks/useReplBridge.tsx`，723 行

## 模块职责

### 1. 输入增强与建议

`useTypeahead.tsx`、`fileSuggestions.ts` 负责输入建议、补全和文件候选。

### 2. 声音与外部输入

`useVoice.ts` 说明语音模式在 UI 层需要持续状态机式 hook 支撑。

### 3. 通知与 inbox

`useInboxPoller.ts`、`notifs/*` 把后台消息、通知和轮询接到 REPL 中。

### 4. bridge / remote 连接行为

`useReplBridge.tsx` 把 REPL bridge 的连接状态、事件和 UI 反馈接起来。

### 5. 权限决策入口

`useCanUseTool.tsx` 非常关键。它把：

- 静态权限判断
- auto mode / classifier
- swarm/coordinator 特殊路径
- interactive dialog
- bridge/channel callback

统一整合成一个可在 runtime 中调用的 hook 接口。

## 为什么这个目录重要

Claude Code 的很多复杂性并不在组件 JSX，而在组件和 runtime 之间的行为协调。`hooks` 目录就是这种协调层。

## 与其他模块的关系

- `components`：主要消费者
- `state`：读取和更新状态
- `services`：调用执行逻辑
- `utils`：依赖大量底层工具函数
- `bridge` / `remote`：实时状态接入

## 工程观察

当一个终端应用开始大量使用 hooks 来承接复杂行为时，通常意味着：

- UI 已经足够复杂
- 需要把事件/副作用从组件树剥离出来
- 很多功能既不是纯 service，也不是纯 view

Claude Code 显然处于这个阶段。

## 维护风险

- hooks 很容易变成“隐形业务层”，需要防止单个 hook 继续无限膨胀。
- `useCanUseTool.tsx`、`useTypeahead.tsx`、`useVoice.ts` 是尤其需要重点治理的大文件。
