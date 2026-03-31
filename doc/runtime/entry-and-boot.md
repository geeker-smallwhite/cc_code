# 启动与装配链路

## 1. 文档目的

这份文档只关注一个问题：Claude Code 是怎样从一个 `claude` 命令，变成一个完整运行中的终端 agent 应用的。

这里重点覆盖根运行时文件：

- `src/entrypoints/cli.tsx`
- `src/main.tsx`
- `src/setup.ts`
- `src/interactiveHelpers.tsx`
- `src/replLauncher.tsx`
- `src/dialogLaunchers.tsx`
- `src/projectOnboardingState.ts`

## 2. 启动链总览

完整启动链可以抽象成六步：

1. 解析极早期 flag，命中 fast-path 就立即退出
2. 做全局环境修正和 profiler checkpoint
3. 进入 `main.tsx`，统一做 argv 重写和客户端类型识别
4. 通过 Commander 建立标准命令入口
5. 在 `preAction` 中执行初始化、迁移、插件和远端设置加载
6. 最终进入 REPL、print、doctor、plugin、mcp 或其他模式

## 3. `src/entrypoints/cli.tsx`

### 3.1 模块定位

这个文件是“真正的第一入口”。它的主要价值不是逻辑复杂，而是把大量无需加载全量系统的路径提前剪掉。

文件规模：

- 文件数：1
- 行数：303

### 3.2 关键职责

- 设置极早期环境变量
- 为 remote/CCR 场景调整 `NODE_OPTIONS`
- 在 feature gate 下快速处理内部/实验入口
- 对 `--version` 等请求做到近零成本返回

### 3.3 为什么它重要

如果没有这一层，所有命令都会：

- 导入整套 React/Ink/UI/runtime
- 拉起更多配置与状态逻辑
- 明显增加冷启动时间

这个文件事实上体现了仓库对 CLI 产品体验的一个核心判断：不是所有调用都值得加载完整系统。

### 3.4 已确认的 fast-path

从代码可以直接看出的路径包括：

- `--version`
- `--dump-system-prompt`
- `--claude-in-chrome-mcp`
- `--chrome-native-host`
- `--computer-use-mcp`
- `--daemon-worker`
- `remote-control` / `remote` / `sync` / `bridge`
- `daemon`
- `ps` / `logs` / `attach` / `kill`
- template jobs
- `environment-runner`
- `self-hosted-runner`
- `--worktree --tmux`

这些路径说明入口层不只是“转发到 main”，而是有明确的模式分流责任。

## 4. `src/main.tsx`

### 4.1 模块定位

`main.tsx` 是整个仓库的 assembly root。它负责把各种入口、配置、客户端类型、运行模式和后续 UI/runtime 拼成一个完整应用。

文件规模：

- 文件数：1
- 行数：4684

### 4.2 主要职责

- 初始化 warning handler、退出/中断行为
- 处理 direct-connect、deep link、assistant、ssh 等特殊 argv 改写
- 区分 interactive / non-interactive / sdk / remote / desktop 等客户端
- 建立 Commander 主命令与全局 option
- 在 `preAction` 中执行 `init()`、sink 初始化、plugin-dir 注入、迁移、remote managed settings、settings sync
- 把最终控制权交给 REPL、print 或各子命令

### 4.3 `main.tsx` 的典型模式

它不是把所有业务都写在一个函数里，而是不断做这三类事情：

- 早期“判别”：当前到底是什么入口
- 早期“重写”：把特殊输入归一化成主命令能消费的形式
- 后期“装配”：把各种能力注入到后续运行路径

### 4.4 典型早期重写场景

- `cc://` 和 `cc+unix://` 被改写为 direct-connect 主命令流
- `--handle-uri` 与 macOS URL handler 被提早处理
- `claude assistant [sessionId]` 会被剥离为主交互路径
- `claude ssh <host> [dir]` 会把参数拆出来交给后续 REPL 分支

这一设计很关键：很多“特殊模式”最终依然复用主交互壳，而不是重复实现一套 UI。

### 4.5 Commander 装配策略

`run()` 里做的不是少量命令注册，而是非常大的 CLI surface 装配：

- 默认 prompt 参数
- print 模式
- output format / input format
- structured output schema
- thinking、budget、model、settings、plugin-dir 等高级选项
- 一批子命令和控制面选项

它更接近一个“统一命令协议入口”，而不是薄薄一层 CLI wrapper。

## 5. `src/setup.ts`

### 5.1 模块定位

`setup.ts` 负责启动后的“环境就绪化”。它比 `main.tsx` 更接近系统初始化执行逻辑。

文件规模：

- 文件数：1
- 行数：477

### 5.2 关键职责

- Node 版本检查
- 会话切换
- UDS messaging server 初始化
- swarms teammate snapshot
- iTerm2 / Terminal.app 备份恢复
- `setCwd()`
- hooks 配置快照与 file watcher 初始化
- worktree 创建、tmux session 创建
- release notes、session memory、版本锁定、最近活动等启动侧逻辑

### 5.3 为什么它不能简单塞回 `main.tsx`

`setup.ts` 承担的是“有副作用的会话环境准备”。这部分和参数解析装配是不同关注点：

- `main.tsx` 负责决策
- `setup.ts` 负责执行准备动作

如果两者混在一起，`main.tsx` 会更快失控。

## 6. `src/interactiveHelpers.tsx`

### 6.1 模块定位

这是交互模式启动期的辅助层，主要解决“在 Ink root 已经创建后，怎么一致地渲染 setup/onboarding/dialog/退出消息”。

文件规模：

- 文件数：1
- 行数：366

### 6.2 关键职责

- `showDialog()`：统一 Promise 风格对话框挂载
- `exitWithError()` / `exitWithMessage()`：通过 Ink 渲染退出消息
- `showSetupDialog()`：统一包裹 `AppStateProvider` 与 `KeybindingSetup`
- `renderAndRun()`：挂载根元素、启动 deferred prefetch、等待退出、执行 graceful shutdown
- `showSetupScreens()`：串起 onboarding、trust dialog、API key 审批、危险模式确认、auto mode opt-in、grove 等一系列首次交互流程

### 6.3 这个文件的重要性

它把“启动期 UI 例外路径”从 `main.tsx` 里分离了出来。没有它，`main.tsx` 会混入大量 JSX 和 dialog wiring 细节。

## 7. `src/replLauncher.tsx`

### 7.1 模块定位

这是一个非常薄但非常有价值的装配函数，负责把 `App` 和 `REPL` 拼起来，并通过 `renderAndRun()` 统一启动。

文件规模：

- 文件数：1
- 行数：23

### 7.2 职责

- 动态导入 `components/App`
- 动态导入 `screens/REPL`
- 以统一方式渲染 `<App><REPL /></App>`

### 7.3 为什么专门抽出来

它不是因为逻辑复杂，而是因为大文件治理。该文件直接注释了自己是 `main.tsx` 提取工作的一部分，这反映出仓库已经在做“超大主文件拆分”。

## 8. `src/dialogLaunchers.tsx`

### 8.1 模块定位

这个文件专门承接 `main.tsx` 里那些“一次性 JSX 对话框挂载点”。

文件规模：

- 文件数：1
- 行数：133

### 8.2 关键职责

- 动态导入 dialog 组件
- 统一 `done` 回调和 Promise 化接口
- 避免 `main.tsx` 出现过多内联 JSX/回调 wiring

### 8.3 已覆盖的对话场景

- snapshot update dialog
- invalid settings dialog
- assistant session chooser
- assistant install wizard
- teleport resume wrapper
- teleport repo mismatch dialog
- resume conversation chooser

### 8.4 设计价值

它不是业务中枢，但对可维护性很关键。因为启动链路里最容易膨胀的就是“异常交互分支”，这个文件就是在削减这种膨胀。

## 9. `src/projectOnboardingState.ts`

### 9.1 模块定位

这个文件规模不大，但负责把项目 onboarding 是否已完成、是否应该再次提示之类的逻辑从大文件中抽离出来。

文件规模：

- 文件数：1
- 行数：83

### 9.2 作用判断

它属于“启动后第一次体验”的产品状态辅助模块，与 `interactiveHelpers.tsx`、`components/Onboarding` 一起构成 onboarding 流程的一部分。

## 10. 启动链路中的关键设计选择

### 10.1 入口层大量 dynamic import

好处：

- 冷启动成本低
- 少量命令不会触发全量模块评估
- feature gate 更容易做 dead code elimination

代价：

- 静态阅读跳转更多
- 调试时要理解哪些路径在什么时候才被加载

### 10.2 将“argv 解析”和“环境准备”分离

`main.tsx` 更像装配层，`setup.ts` 更像执行层。这个分离是对的，否则主入口会进一步成为不可维护大泥球。

### 10.3 在 Ink 根创建后统一走 helper

`interactiveHelpers.tsx` 的存在说明作者很清楚：一旦进入 Ink，普通 `console.error`、裸 Promise dialog 都会变得不可靠或不一致，需要统一包装。

## 11. 阅读这条链路时要特别注意什么

- 很多路径受 `feature('...')` 影响，代码存在不代表当前构建启用。
- 一些特殊模式会重写 `process.argv`，后续行为不能仅靠原始命令推断。
- `preAction` 做了大量真正重要的初始化；只看命令定义会漏掉初始化副作用。
- `main.tsx` 与 `interactiveHelpers.tsx`、`dialogLaunchers.tsx` 是一起看的，单看其中之一都不完整。

## 12. 小结

Claude Code 的启动层不是薄壳，而是一套为了支持多入口、多产品模式、多 feature gate 而演化出的装配系统。

理解这层的关键，不是记住每个 flag，而是记住三条原则：

- 先 fast-path 再全量加载
- 先统一入口形态再进入主运行时
- 把启动期 UI 和会话运行时明确分开
