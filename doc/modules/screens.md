# screens 模块

## 模块定位

`src/screens` 是 Claude Code 的场景级终端页面层。它和 `components` 的区别在于：这里的文件直接对应顶层用户场景，而不是可复用部件。

规模统计：

- 文件数：3
- 行数：5980
- 代表文件：
  - `screens/REPL.tsx`，5006 行
  - `screens/Doctor.tsx`，575 行
  - `screens/ResumeConversation.tsx`，399 行

## 三个场景的职责

### `REPL.tsx`

这是全仓库最关键的场景文件之一，也是整个产品的主交互壳。

它负责：

- 消息区
- Prompt 输入区
- footer/panel/overlay
- task/agent 视图
- 通知
- 权限请求 UI
- bridge/remote/voice 等多种状态联动

严格说，它已经不是单纯 screen，而是一个完整的终端应用外壳。

### `Doctor.tsx`

承接诊断/健康检查相关的专门界面。

### `ResumeConversation.tsx`

承接会话恢复/选择的交互界面，通常会在启动期被 `dialogLaunchers.tsx` 或 `main.tsx` 调用。

## 为什么 `REPL.tsx` 这么重要

Claude Code 的用户价值大部分直接暴露在 REPL 中，因此很多能力最终都会汇聚到这里：

- 输入
- 输出
- 状态
- 任务
- 权限
- 远程连接

这让 `REPL.tsx` 成为业务集成度极高的文件。

## 与其他模块的关系

- `components`：组成 REPL 视图
- `hooks`：驱动行为与副作用
- `state`：提供会话状态
- `QueryEngine` / `services`：提供执行结果和 runtime 事件

## 维护风险

- `REPL.tsx` 已经 5000+ 行，是最需要继续抽分的大文件之一。
- 任何新功能如果都优先接进 REPL 根文件，复杂度会继续上升。
