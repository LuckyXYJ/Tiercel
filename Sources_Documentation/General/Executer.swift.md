# Executer.swift

对应文件：`Sources/General/Executer.swift`

## 文件职责

该文件封装回调执行逻辑，统一处理回调是否派发到主线程。

## 核心类型

- `Handler<T>`：类型别名，表示接收对象 `T` 的闭包。
- `Executer<T>`：保存 `onMainQueue` 与 `handler`，通过 `execute(_:)` 执行回调。

## 关键行为

当 `onMainQueue == true` 时，`execute(_:)` 使用 `DispatchQueue.tr.executeOnMain` 确保回调在主线程执行；否则直接在当前线程执行。

## 使用位置

- `SessionManager` 的 `progress`、`success`、`failure`、`completion`、控制回调。
- `Task`/`DownloadTask` 的进度、成功、失败、完成、校验回调。
- `Cache.clearDiskCache` 的完成回调。

## 注意点

`Executer` 不持有业务对象，只在执行时传入对象。因此闭包内部如果捕获外部对象，需要调用方自行处理循环引用问题。
