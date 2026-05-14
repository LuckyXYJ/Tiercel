# SessionDelegate.swift

对应文件：`Sources/General/SessionDelegate.swift`

## 文件职责

该文件实现 `URLSessionDownloadDelegate`，将系统 URLSession 回调转发给 `SessionManager` 和对应的 `DownloadTask`。

## 核心属性

- `manager`：弱引用 `SessionManager`，避免 delegate 与 manager 形成循环引用。

## 回调处理

- `urlSession(_:didBecomeInvalidWithError:)`：通知 manager 当前 session 失效，由 manager 负责重建 session 和恢复任务。
- `urlSessionDidFinishEvents(forBackgroundURLSession:)`：通知 manager 后台事件处理完毕，用于调用外部 `completionHandler`。
- `urlSession(_:downloadTask:didWriteData:...)`：找到业务 `DownloadTask` 后转发进度更新。
- `urlSession(_:downloadTask:didFinishDownloadingTo:)`：找到业务 `DownloadTask` 后转发文件完成位置。
- `urlSession(_:task:didCompleteWithError:)`：找到业务 `DownloadTask` 后转发最终完成或错误。

## 任务查找策略

回调中优先读取系统 task 的 `tiercelTask` 关联对象。如果不存在，则使用 `downloadTask.currentRequest?.url` 或错误中的 failing URL，通过 `manager.mapTask(_:)` 查找业务任务。找到后会重新写入 `tiercelTask`，减少后续查找成本。

## 注意点

后台 URLSession 恢复时，系统可能重新创建 `URLSessionDownloadTask`，关联对象会丢失。因此该文件必须保留 URL 映射兜底逻辑。
