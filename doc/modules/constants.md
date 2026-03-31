# constants 模块

## 模块定位

`src/constants` 提供系统范围内可复用的常量、长文本 prompt、输出样式枚举和配置字典，是整个仓库的“静态事实层”。

规模统计：

- 文件数：21
- 行数：2648
- 代表文件：
  - `constants/prompts.ts`，914 行
  - `constants/oauth.ts`，234 行
  - `constants/outputStyles.ts`，216 行
  - `constants/spinnerVerbs.ts`，204 行
  - `constants/files.ts`，156 行

## 模块职责

### 1. 系统 prompt 常量

`prompts.ts` 是最关键的文件之一。Claude Code 的很多行为边界并不只来自代码，也来自系统提示词配置。

### 2. 协议/枚举常量

例如：

- oauth 相关常量
- 输出样式名称
- 文件名/目录名约定
- spinner 文案

### 3. 降低魔法字符串扩散

大型仓库如果没有这一层，很多协议名和 prompt 片段会散落在各模块中，后续很难统一修改。

## 在架构中的位置

`constants` 本身不应该持有复杂业务逻辑，但它对这些模块有直接影响：

- `main.tsx`
- `query.ts`
- `services/api/*`
- `components/*`
- `tools/*`

尤其是 prompt 常量，会深刻影响模型行为。

## 特别关注点

### `prompts.ts`

这是“代码控制”和“模型控制”的交汇点。对 Claude Code 来说，prompt 并不是外围文本，而是产品逻辑的一部分。

### `outputStyles.ts`

这反映出系统支持多种输出风格/样式方案，而不是只有单一终端渲染模式。

## 维护建议

- 复杂逻辑不要继续塞进 `constants`。
- 常量文件应尽量保持“定义层”角色，而不是兼任生成层。
- 涉及 prompt 的改动应配合 runtime 文档一起理解，因为它们会改变模型侧行为，不只是文本替换。
