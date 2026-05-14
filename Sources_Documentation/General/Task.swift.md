# Task.swift

对应文件：`Sources/General/Task.swift`

## 文件职责

该文件定义下载任务的泛型基类 `Task<TaskType>`，负责保存任务通用状态、进度、持久化字段和链式回调接口。`DownloadTask` 继承该类并补充 URLSession 下载逻辑。

## 核心类型

- `Task.Validation`：文件校验状态，包含 `unkown`、`correct`、`incorrect`。
- `CompletionType`：区分本地完成与 URLSession 网络回调完成。
- `InterruptType`：描述任务中断原因，包括手动操作、错误、不可接受状态码。
- `State`：保存 session、headers、校验信息、状态、当前 URL、起止时间、速度、文件名、剩余时间、错误和各种回调执行器。

## 关键属性

- `url`：任务原始 URL。
- `currentURL`：实际请求 URL，可能因重定向变化。
- `fileName`：下载完成后的文件名，默认是 URL md5 加扩展名。
- `progress`：Foundation `Progress`，记录总字节和已完成字节。
- `status`：任务状态，状态变化时会输出下载任务日志。
- `speed`、`timeRemaining`：由下载进度定时计算。
- `verificationCode`、`verificationType`、`validation`：文件校验相关状态。

## Codable 持久化

`Task` 会编码 URL、当前 URL、文件名、headers、起止时间、字节数、状态、校验信息和错误。解码时通过 `decoder.userInfo[.cache]` 和 `decoder.userInfo[.operationQueue]` 恢复运行所需依赖。

## 回调 API

- `progress(onMainQueue:handler:)`
- `success(onMainQueue:handler:)`
- `failure(onMainQueue:handler:)`
- `completion(onMainQueue:handler:)`

这些方法返回 `Self`，支持链式调用。若设置回调时任务已经处于终态，会异步补发对应回调。

## 注意点

基类的 `execute(_:)` 是空实现，子类需要重写。`DownloadTask` 会重写该方法，使泛型回调能以 `DownloadTask` 类型执行。
