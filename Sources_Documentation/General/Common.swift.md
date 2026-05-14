# Common.swift

对应文件：`Sources/General/Common.swift`

## 文件职责

该文件定义 Tiercel 的公共基础类型，包括日志类型、任务状态、`tr` 命名空间包装器，以及 `URLSessionTask` 到 `DownloadTask` 的关联对象。

## 核心类型

- `LogOption`：控制日志输出，`.default` 表示输出日志，`.none` 表示关闭日志。
- `LogType`：描述日志事件，分为 `sessionManager`、`downloadTask` 和 `error`。
- `Logable`：日志协议，要求提供 `identifier`、`option` 和 `log(_:)`。
- `Logger`：默认日志实现，通过 `print` 输出 manager、task 或 error 信息。
- `Status`：任务和管理器共用状态枚举，包括 `waiting`、`running`、`suspended`、`canceled`、`failed`、`removed`、`succeeded`，以及预操作状态 `willSuspend`、`willCancel`、`willRemove`。
- `TiercelWrapper<Base>` 与 `TiercelCompatible`：提供 `object.tr.xxx` 风格的扩展命名空间。

## 关键实现

`URLSessionTask.tiercelTask` 使用 Objective-C associated object 将系统 `URLSessionTask` 与业务层 `DownloadTask` 关联。`SessionDelegate` 收到 URLSession 回调时，会优先通过这个弱引用找到对应的下载任务；如果找不到，再通过 `SessionManager.mapTask(_:)` 兜底映射。

## 依赖关系

- 被 `SessionManager` 用于日志输出和状态管理。
- 被 `DownloadTask` 用于任务状态记录。
- 被多个 `Extensions` 文件用于实现 `tr` 命名空间扩展。
- 被 `SessionDelegate` 间接依赖，用来从 `URLSessionTask` 获取业务任务。

## 注意点

`tiercelTask` 使用 `.OBJC_ASSOCIATION_ASSIGN`，语义上是弱关联，避免循环引用；因此 URLSession 回调中仍然需要 manager 的映射表作为兜底。
