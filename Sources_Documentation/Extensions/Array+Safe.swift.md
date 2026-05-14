# Array+Safe.swift

对应文件：`Sources/Extensions/Array+Safe.swift`

## 文件职责

该文件为数组提供安全下标读取方法，避免索引越界导致运行时崩溃。

## 核心 API

`safeObject(at:)` 会检查传入 index 是否位于 `0..<count`，合法时返回元素，否则返回 `nil`。

## 使用位置

- `SessionManager.multiDownload` 中读取与 URL 对应的 headers 和 fileNames。
- `[DownloadTask].validateFile(codes:type:onMainQueue:handler:)` 中读取与任务对应的校验码。

## 注意点

该方法只提供读取保护，不支持写入。调用方需要处理返回 `nil` 的情况。
