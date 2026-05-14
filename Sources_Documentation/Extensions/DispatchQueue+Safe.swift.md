# DispatchQueue+Safe.swift

对应文件：`Sources/Extensions/DispatchQueue+Safe.swift`

## 文件职责

该文件为 `DispatchQueue` 增加主线程安全执行工具。

## 核心 API

`DispatchQueue.tr.executeOnMain(_:)` 会：

1. 如果当前已经在主线程，立即执行闭包。
2. 如果当前不在主线程，异步派发到 `DispatchQueue.main`。

## 使用位置

- `Executer.execute(_:)` 用它保证 UI 回调在主线程。
- `SessionManager.updateProgress()` 用它更新 iOS 网络活动指示器。
- `SessionManager.didFinishEvents(forBackgroundURLSession:)` 用它回调后台 session 完成 handler。

## 注意点

该方法总是异步派发非主线程调用，不会阻塞当前线程等待主线程执行完成。
