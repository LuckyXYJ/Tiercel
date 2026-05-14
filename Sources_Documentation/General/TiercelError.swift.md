# TiercelError.swift

对应文件：`Sources/General/TiercelError.swift`

## 文件职责

该文件定义 Tiercel 统一错误类型，覆盖 URL 校验、任务查找、批量参数匹配、HTTP 状态码、缓存文件操作等错误场景。

## 核心错误

- `unknown`：未知错误。
- `invalidURL(url:)`：URL 转换失败。
- `duplicateURL(url:)`：批量下载中出现重复 URL。
- `indexOutOfRange`：移动任务时索引越界。
- `fetchDownloadTaskFailed(url:)`：无法从 manager 中找到指定任务。
- `headersMatchFailed`：批量下载 headers 数量与 URLs 数量不匹配。
- `fileNamesMatchFailed`：批量下载 fileNames 数量与 URLs 数量不匹配。
- `unacceptableStatusCode(code:)`：HTTP 响应状态码不在 200..<300。
- `cacheError(reason:)`：缓存层文件操作错误。

## CacheErrorReason

缓存错误细分为目录创建、文件删除、复制、移动、任务列表读取、任务列表编码、文件不存在和读取失败。

## 协议实现

- `LocalizedError`：提供可读的 `errorDescription`。
- `CustomNSError`：提供统一 error domain `com.Daniels.Tiercel.Error`，并为 `unacceptableStatusCode` 指定错误码 `1001`。

## 使用位置

`SessionManager`、`DownloadTask`、`Cache`、`URLConvertible` 和 `FileChecksumHelper` 都会使用或产生这些错误，用于日志输出和任务失败状态记录。
