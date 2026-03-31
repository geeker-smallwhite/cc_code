# context 模块

## 模块定位

`src/context` 是 React context 层，负责把一些跨组件共享但又不适合沉到全局 AppState 的 UI 语义分发出去。

规模统计：

- 文件数：9
- 行数：1013
- 代表文件：
  - `context/notifications.tsx`，240 行
  - `context/stats.tsx`，220 行
  - `context/overlayContext.tsx`，151 行
  - `context/promptOverlayContext.tsx`，125 行
  - `context/voice.tsx`，88 行

## 模块职责

### 1. 通知上下文

把通知 current/queue 相关语义提供给组件树，而不需要层层传 props。

### 2. 统计上下文

让运行期统计数据能在界面多处消费。

### 3. overlay / prompt overlay

管理提示层、悬浮层、弹出层这类 UI 状态。

### 4. voice 上下文

让语音模式相关状态可以在交互组件中共享。

## 和 `state` 模块的关系

这两层常常会被混淆：

- `state` 负责全局应用数据模型
- `context` 负责特定横切 UI 语义的分发

也就是说，`context` 更像消费层 wiring，`state` 更像权威状态源。

## 为什么不把所有东西都放进 AppState

因为有些内容：

- 生命周期很局部
- 只服务特定 UI 区域
- 需要 provider 组合能力
- 不值得进入统一状态树

这类内容用 React context 更合适。

## 工程观察

这个目录虽然不大，但很说明问题：Claude Code 的终端 UI 已经复杂到需要多层 provider 才能保持组件可维护。

## 维护要点

- context 应维持“分发层”而不是“业务中心”。
- 如果某个 context 开始承担大量副作用或复杂计算，通常说明它应该下沉到 `services` 或上收进 `state`。
