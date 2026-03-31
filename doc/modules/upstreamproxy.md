# upstreamproxy 模块

## 模块定位

`src/upstreamproxy` 承担上游代理/转发相关逻辑，是网络接入层的一个窄而关键的切面。

规模统计：

- 文件数：2
- 行数：740
- 代表文件：
  - `upstreamproxy/relay.ts`，455 行
  - `upstreamproxy/upstreamproxy.ts`，285 行

## 模块职责

从命名判断，这个目录主要负责：

- 对上游服务做 relay/forward
- 作为某些 remote/transport 场景的代理层

## 为什么单独成目录

代理逻辑通常有自己的关注点：

- 请求转发
- 头信息处理
- 连接维护
- 错误转译

把这类逻辑混进 `services/api` 或 `cli/transports` 会让边界变得模糊。

## 与其他模块的关系

- `cli/transports/*`
- `remote`
- `bridge`
- 可能的 self-hosted / environment-runner 路径

## 工程判断

当前目录不大，但属于典型的“基础设施关键点”。这类代码通常不是经常改，但一旦出错影响面会很广。
