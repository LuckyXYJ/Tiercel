# SessionConfiguration.swift

对应文件：`Sources/General/SessionConfiguration.swift`

## 文件职责

该文件定义 `SessionManager` 创建后台 `URLSessionConfiguration` 时使用的业务配置。

## 核心属性

- `timeoutIntervalForRequest`：请求超时时间，默认 `60.0` 秒。
- `maxConcurrentTasksLimit`：最大并发下载任务数。外部可设置，但内部限制在 `1...6`。
- `allowsExpensiveNetworkAccess`：iOS 13/macOS 10.15 之后用于控制是否允许昂贵网络，默认 `true`。
- `allowsConstrainedNetworkAccess`：iOS 13/macOS 10.15 之后用于控制是否允许受限网络，默认 `true`。
- `allowsCellularAccess`：是否允许蜂窝网络下载，默认 `false`。

## 与 SessionManager 的关系

`SessionManager.createSession(_:)` 会把这些字段转换到系统的 `URLSessionConfiguration.background(withIdentifier:)` 上。修改 `SessionManager.configuration` 时，如果 manager 正在运行，会触发整体暂停并准备重建 session。

## 注意点

并发数被硬限制为最多 6，设置超过 6 会被压回 6；设置小于 1 会被提升到 1。这个限制影响 `SessionManager.shouldRun` 和等待任务调度。
