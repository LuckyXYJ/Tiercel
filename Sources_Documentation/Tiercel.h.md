# Tiercel.h

对应文件：`Sources/Tiercel.h`

## 文件职责

该文件是 Tiercel framework 的 Objective-C umbrella header，用于暴露 framework 版本符号，并作为 Objective-C/C 侧公共头文件的统一入口。

## 核心内容

- 导入 `Foundation/Foundation.h`。
- 声明 `TiercelVersionNumber`，表示 framework 的数值版本。
- 声明 `TiercelVersionString`，表示 framework 的字符串版本。
- 预留公共头文件导入位置。

## 与 Swift 源码的关系

Tiercel 的主要实现位于 Swift 文件中。该头文件主要用于 framework 打包与 Objective-C 兼容，不承载下载、缓存、任务管理等业务逻辑。

## 注意点

当前没有额外导入公共 Objective-C 头文件。如果未来增加 Objective-C API，需要在该 umbrella header 中统一导入，便于 framework 使用方通过 `#import <Tiercel/Tiercel.h>` 获取公共接口。
