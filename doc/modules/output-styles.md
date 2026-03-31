# outputStyles 模块

## 模块定位

`src/outputStyles` 负责输出样式目录的发现与加载，是输出呈现能力的扩展点。

规模统计：

- 文件数：1
- 行数：98
- 核心文件：`outputStyles/loadOutputStylesDir.ts`

## 模块职责

### 1. 扫描输出样式目录

把用户或系统定义的 output style 载入运行时。

### 2. 让输出样式成为可扩展资源

这说明 Claude Code 的输出外观不是纯写死的，至少在某些路径上支持目录化扩展。

## 与其他模块的关系

- `constants/outputStyles.ts`：定义样式名和枚举
- `commands/output-style/*`：用户入口
- `cli/print.ts` 与 UI 输出层：实际消费样式

## 工程价值

虽然这个模块极小，但它体现出一个重要设计方向：输出不仅是字符串，也是一种可以配置和装配的产品能力。
