# ResumeDataHelper.swift

对应文件：`Sources/Utility/ResumeDataHelper.swift`

## 文件职责

该文件用于解析 `URLSessionDownloadTask` 生成的 resumeData，从中提取断点续传临时文件名，并包含一段修复旧格式 resumeData 的辅助逻辑。

## 关键常量

文件定义了一组系统 resumeData 字典键，例如：

- `NSURLSessionResumeInfoVersion`
- `NSURLSessionResumeCurrentRequest`
- `NSURLSessionResumeOriginalRequest`
- `NSURLSessionResumeInfoTempFileName`
- `NSURLSessionResumeInfoLocalPath`
- `NSURLSessionResumeBytesReceived`

这些键用于兼容不同系统版本的 resumeData 结构。

## 核心 API

- `getResumeDictionary(_:)`：优先按 keyed archive 解码 resumeData；失败后按 property list 解码；最终返回可变字典。
- `getTmpFileName(_:)`：根据 resumeData 版本提取临时文件名。版本大于 1 时读取 `NSURLSessionResumeInfoTempFileName`；旧版本读取本地路径并取最后一个 path component。

## 私有修复逻辑

`correct(with:)` 会修正某些 resumeData 中 request archive key 不兼容的问题，例如把 `__nsurlrequest_proto_prop_obj_x` 转换为 `$x` 形式，并修复 `$top` 里的 root object key。

## 使用位置

`DownloadTask.DownloadState.resumeData` 的 `didSet` 会调用 `ResumeDataHelper.getTmpFileName`，从 resumeData 中解析 `tmpFileName`。之后 `Cache.storeTmpFile` 和 `Cache.retrieveTmpFile` 会基于这个文件名备份或恢复断点文件。

## 注意点

`correct(with:)` 当前是私有方法，源码中没有调用。主要有效路径是解析 resumeData 和提取临时文件名。
