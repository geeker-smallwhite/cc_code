# voice 模块

## 模块定位

`src/voice` 是语音模式的顶层辅助目录。当前快照只可见一个文件：`voiceModeEnabled.ts`。

规模统计：

- 文件数：1
- 行数：54
- 核心文件：`voice/voiceModeEnabled.ts`

## 模块职责

当前能够明确确认的职责是：

- 统一判断语音模式是否启用

这类模块通常承担 feature gate / settings / environment 的聚合判断，而不是具体语音实现。

## 与其他模块的关系

- `hooks/useVoice.ts`
- `context/voice.tsx`
- `commands/voice/*`
- `main.tsx` 或 setup 流程中对语音模式的启停判断

## 工程判断

这说明语音功能在当前快照里被很轻地挂接进主系统，具体实现大概率分散在 hook、component 或内部 feature 路径里，而顶层 `voice` 目录只保留了启用判断。

## 维护建议

- 如果未来语音功能继续增强，建议把“启用判断”和“运行逻辑”分离得更明确。
- 当前这种单文件目录适合承载 feature 开关，不适合逐渐塞入大块逻辑。
