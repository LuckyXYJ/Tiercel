# URLConvertible.swift

对应文件：`Sources/General/URLConvertible.swift`

## 文件职责

该文件定义 URL 输入的统一转换协议，使 `SessionManager` 的下载 API 可以接受 `String`、`URL` 和 `URLComponents`。

## 核心 API

- `URLConvertible.asURL() throws -> URL`：将调用方传入的对象转换为合法 `URL`。
- `String.asURL()`：通过 `URL(string:)` 构造 URL，失败时抛出 `TiercelError.invalidURL`。
- `URL.asURL()`：直接返回自身。
- `URLComponents.asURL()`：返回 `url` 属性，失败时抛出 `TiercelError.invalidURL`。

## 使用位置

- `SessionManager.download(_:)`
- `SessionManager.multiDownload(_:)`
- `SessionManager.fetchTask(_:)`
- `Cache.filePath(url:)`
- `Cache.fileURL(url:)`
- `Cache.fileExists(url:)`

## 注意点

协议只校验是否能构造 `URL`，不校验 scheme、host 或资源是否真实可访问。网络请求阶段的失败会由 `URLSession` 和 `DownloadTask` 状态处理逻辑接管。
