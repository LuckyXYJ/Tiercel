# FileManager+AvailableCapacity.swift

对应文件：`Sources/Extensions/FileManager+AvailableCapacity.swift`

## 文件职责

该文件为 `FileManager` 增加可用磁盘空间读取能力。

## 核心 API

`FileManager.default.tr.freeDiskSpaceInBytes` 会读取用户 home 目录所在卷的 `volumeAvailableCapacityForImportantUsageKey`，返回可用于重要用途的剩余容量。

## 返回值

- 成功读取时返回字节数，类型为 `Int64`。
- 读取失败时返回 `0`。

## 使用关系

当前 `Sources` 中没有直接调用该属性，但它为使用方提供了下载前检查磁盘空间的能力。

## 注意点

该值由系统估算，可能受 iOS/macOS 存储清理策略影响，不一定等同于物理磁盘剩余空间。
