# Int64+TaskInfo.swift

对应文件：`Sources/Extensions/Int64+TaskInfo.swift`

## 文件职责

该文件为 `Int64` 增加下载信息展示相关的格式化方法，主要用于速度、剩余时间和字节大小展示。

## 核心 API

通过 `Int64.tr` 访问：

- `convertSpeedToString()`：将字节数格式化为文件大小字符串后追加 `/s`，例如 `1 MB/s`。
- `convertTimeToString()`：把秒数格式化为位置式时间字符串。
- `convertBytesToString()`：使用 `ByteCountFormatter` 把字节数格式化为文件大小。

## 使用位置

- `Task.speedString`
- `Task.timeRemainingString`
- `SessionManager.speedString`
- `SessionManager.timeRemainingString`

## 注意点

`convertTimeToString()` 使用 `DateComponentsFormatter`，实际输出会受系统格式化行为影响。`convertBytesToString()` 使用 `.file` 样式，适合展示文件大小。
