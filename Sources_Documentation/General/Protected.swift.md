# Protected.swift

对应文件：`Sources/General/Protected.swift`

## 文件职责

该文件提供线程安全基础设施，包括锁封装、属性包装器、去抖器和节流器。它是 `SessionManager`、`Task`、`DownloadTask` 等状态安全读写的基础。

## 核心类型

- `UnfairLock`：基于 `os_unfair_lock` 的轻量锁封装，提供 `around` 方法执行临界区代码。
- `Protected<T>`：属性包装器，用锁保护泛型值，支持 `wrappedValue` 读写、`read` 只读闭包、`write` 可变闭包。
- `Debouncer`：去抖执行器，在指定时间内多次调用只保留最后一次任务。
- `Throttler`：节流执行器，在指定时间窗口内限制任务执行频率，并可选择是否使用最新任务替换旧任务。

## 使用位置

- `SessionManager.State` 由 `Protected<State>` 保护。
- `Task.State` 由 `Protected<State>` 保护。
- `DownloadTask.DownloadState` 由 `Protected<DownloadState>` 保护。
- `Cache.storeTasks(_:)` 使用 `Debouncer` 延迟写入 plist，减少频繁状态变更导致的磁盘写入。

## 关键流程

`Protected.write` 通过 `inout` 修改状态，保证同一时间只有一个线程进入写操作。`SessionManager` 和 `DownloadTask` 大量属性都通过计算属性转发到 `Protected`，避免外部并发回调导致状态竞争。

## 注意点

`wrappedValue` 读取返回的是值副本。对于结构体状态，推荐使用 `read` 或 `write` 执行成组操作，避免多次读取之间状态被其他线程改变。
