# DownloadTask.swift

对应文件：`Sources/General/DownloadTask.swift`

## 文件职责

该文件实现单个下载任务的完整生命周期，包括创建 URLSession 下载任务、断点续传、暂停/取消/移除、下载完成处理、文件校验、速度统计和回调通知。

## 继承关系

`DownloadTask` 继承自 `Task<DownloadTask>`，复用通用状态、持久化和链式回调能力，并补充下载专属状态：

- `resumeData`：系统生成的断点续传数据。
- `response`：HTTP 响应。
- `tmpFileName`：断点续传临时文件名。
- `shouldValidateFile`：是否需要重新校验文件。

## 关键属性

- `sessionTask`：实际的 `URLSessionDownloadTask`。设置时会监听 `currentRequest`，并把自身关联到系统 task 的 `tiercelTask`。
- `filePath`：下载完成后的最终文件路径，由 `Cache.filePath(fileName:)` 生成。
- `pathExtension`：最终文件扩展名。
- `acceptableStatusCodes`：允许的 HTTP 状态码范围，固定为 `200..<300`。

## 下载启动流程

`download()` 根据当前状态决定行为：

- `waiting`、`suspended`、`failed`：检查目标文件是否已存在；存在则直接走本地完成，不存在则按并发限制启动或继续等待。
- `succeeded`：直接触发控制回调并登记成功。
- `running`：补发运行状态控制回调。

实际启动时，如果存在可用 `resumeData` 且临时文件可恢复，则调用 `downloadTask(withResumeData:)`；否则构造 `URLRequest` 并创建新下载任务。

## 控制操作

- `suspend`：运行中任务通过 `cancel(byProducingResumeData:)` 生成断点数据；等待/暂停/失败任务走本地完成分支。
- `cancel`：取消未成功任务，并移除未完成缓存。
- `remove`：移除任务，可选择是否删除已完成文件。
- `update`：更新 headers 和文件名，若文件已存在会通过 `Cache.updateFileName` 移动文件。

## URLSession 回调处理

- `didWriteData`：更新字节进度、保存 response、触发进度回调和 manager 进度更新。
- `didFinishDownloading`：状态码合格时将系统临时文件移动到最终文件路径，并清理断点临时文件。
- `didComplete`：根据本地完成或网络完成、错误、状态码和当前预操作状态，最终进入成功、失败、暂停、取消或移除状态。

## 文件校验

`validateFile(code:type:onMainQueue:handler:)` 保存校验参数，并在任务已成功时调用 `FileChecksumHelper.validateFile`。校验结果会更新 `validation`，写入持久化，并触发校验回调。

## 数组扩展

文件末尾为 `[DownloadTask]` 增加批量 `progress`、`success`、`failure`、`validateFile` 方法，便于 `multiDownload` 返回的任务数组统一设置回调。

## 注意点

iOS 环境下会监听 `UIApplication.didBecomeActiveNotification`，在应用回到前台后短暂 suspend/resume 当前 `sessionTask`，用于规避部分后台 URLSession delegate 回调异常。
