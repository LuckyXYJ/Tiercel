# SessionManager.swift

对应文件：`Sources/General/SessionManager.swift`

## 文件职责

该文件实现 Tiercel 的核心调度器 `SessionManager`。它负责创建后台 `URLSession`、维护下载任务集合、控制单个或全部任务、持久化任务状态、限制并发、统计整体速度和进度，并向调用方提供链式回调。

## 核心状态

`SessionManager.State` 被 `Protected` 保护，主要保存：

- `configuration`：会话配置。
- `session` 和 `shouldCreatSession`：后台 URLSession 及是否需要重建的标记。
- `tasks`：所有下载任务。
- `taskMapper`：原始 URL 字符串到 `DownloadTask` 的映射。
- `urlMapper`：当前请求 URL 到原始 URL 的映射，用于处理重定向和后台恢复。
- `runningTasks`：当前正在运行的任务。
- `restartTasks`：session 重建后需要重启的任务。
- `succeededTasks`：已成功任务集合。
- `speed`、`timeRemaining`：整体速度和剩余时间。
- 多个 `Executer<SessionManager>`：进度、成功、失败、完成和控制回调。

## 初始化流程

初始化时会：

1. 生成完整 background session identifier。
2. 创建或接收外部传入的 `Cache`。
3. 从缓存读取已持久化任务。
4. 恢复任务的 manager、operationQueue 和 URL 映射。
5. 创建后台 `URLSession`。
6. 调用 `restoreStatus()` 检查系统中仍在运行的下载任务。

## 下载 API

- `download(_:headers:fileName:onMainQueue:handler:)`：创建或复用单个任务并启动。
- `multiDownload(_:headersArray:fileNames:onMainQueue:handler:)`：批量创建任务，校验 headers/fileNames 数量，过滤无效和重复 URL，然后并发启动。
- `fetchTask(_:)`：按 URL 查找任务。
- `mapTask(_:)`：按当前请求 URL 查找任务，支持重定向后的 URL 映射。

## 控制 API

单任务控制：

- `start`
- `suspend`
- `cancel`
- `remove`
- `moveTask(at:to:)`

整体控制：

- `totalStart`
- `totalSuspend`
- `totalCancel`
- `totalRemove`
- `tasksSort(by:)`

这些 API 都通过 `operationQueue` 串行执行，避免并发修改任务状态。

## 状态维护

`maintainTasks(with:)` 统一维护任务数组、映射表、运行任务集合和成功任务集合。`determineStatus(fromRunningTask:)` 根据所有任务的状态判断 manager 是否进入 `succeeded`、`failed`、`suspended`、`canceled` 或 `removed`。

## 并发调度

`shouldRun` 判断当前运行任务数是否小于 `configuration.maxConcurrentTasksLimit`。当某个运行任务结束后，`startNextTask()` 会找到第一个 `waiting` 任务并启动。

## 进度统计

manager 创建 `DispatchSourceTimer`，每秒调用 `updateSpeedAndTimeRemaining()`。该方法汇总所有运行任务速度，并根据整体 `Progress` 估算剩余时间。

## 后台会话处理

`createSession(_:)` 使用 `URLSessionConfiguration.background(withIdentifier:)` 创建后台 session。`didBecomeInvalidation(withError:)` 会在 session 失效后重建，并启动 `restartTasks`。`didFinishEvents(forBackgroundURLSession:)` 会在主线程调用 `completionHandler`。

## 注意点

修改 `configuration` 会设置 `shouldCreatSession`，运行中修改还会触发 `totalSuspend()`。这意味着配置变化不是直接热更新当前 URLSession，而是通过暂停、失效、重建和重启任务完成。
