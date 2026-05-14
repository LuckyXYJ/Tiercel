# Notifications.swift

对应文件：`Sources/General/Notifications.swift`

## 文件职责

该文件定义下载任务和会话管理器的通知名称、通知对象封装方式，以及从 `Notification` 中读取业务对象的便捷扩展。

## 通知名称

- `DownloadTask.runningNotification`：单个下载任务进度更新时发送。
- `DownloadTask.didCompleteNotification`：单个下载任务完成、失败、暂停、取消或移除后发送。
- `SessionManager.runningNotification`：会话整体进度更新时发送。
- `SessionManager.didCompleteNotification`：会话整体完成、失败、暂停、取消或移除后发送。

## 便捷读取

`Notification` 遵循 `TiercelCompatible`，可以通过：

- `notification.tr.downloadTask`
- `notification.tr.sessionManager`

读取通知携带的 `DownloadTask` 或 `SessionManager`。

## 发送方式

文件扩展了 `NotificationCenter`，提供：

- `postNotification(name:downloadTask:)`
- `postNotification(name:sessionManager:)`

这两个方法内部会把业务对象放入 `userInfo`。

## 使用位置

`DownloadTask.didWriteData`、`DownloadTask.executeCompletion`、`SessionManager.updateProgress` 和 `SessionManager.executeCompletion` 会触发这些通知。
