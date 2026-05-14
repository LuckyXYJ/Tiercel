
# Double+TaskInfo.swift

对应文件：`Sources/Extensions/Double+TaskInfo.swift`

## 文件职责

该文件为 `Double` 增加时间戳到日期字符串的转换能力，用于展示任务开始和结束时间。

## 核心 API

`Double.tr.convertTimeToDateString()` 会把 `Double` 视为 Unix 时间戳，转换为 `Date` 后使用固定格式 `yyyy-MM-dd HH:mm:ss` 输出。

## 使用位置

- `Task.startDateString`
- `Task.endDateString`

## 注意点

格式化器没有显式设置 locale 或 time zone，因此输出使用当前运行环境的默认区域与时区。
