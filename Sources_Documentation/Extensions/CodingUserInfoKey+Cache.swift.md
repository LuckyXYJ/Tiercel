# CodingUserInfoKey+Cache.swift

对应文件：`Sources/Extensions/CodingUserInfoKey+Cache.swift`

## 文件职责

该文件定义 Codable 解码时传递运行时依赖的 `CodingUserInfoKey`。

## 核心键

- `.cache`：用于在解码 `DownloadTask`/`Task` 时注入当前 `Cache`。
- `.operationQueue`：用于在解码任务时注入当前 `DispatchQueue`。

## 使用位置

- `Cache.init` 会设置 `decoder.userInfo[.cache] = self`。
- `Cache.invalidate()` 会清空 `.cache`。
- `Task.init(from:)` 会从 `decoder.userInfo` 读取 `.cache` 和 `.operationQueue`，没有读取到时使用默认值。

## 注意点

这些 key 是 `internal`，只服务于 Tiercel 内部持久化恢复流程。外部使用方不需要直接操作。
