# Cache.swift

对应文件：`Sources/General/Cache.swift`

## 文件职责

该文件实现下载缓存与任务持久化。它负责创建下载目录、保存完成文件、备份断点临时文件、读取/写入任务列表、清理缓存，并为 URL 生成默认文件名。

## 路径结构

`Cache` 根据 `identifier` 生成默认缓存目录：

- `downloadPath`：下载模块根目录，默认位于系统 caches 目录下。
- `downloadTmpPath`：断点续传临时文件备份目录。
- `downloadFilePath`：下载完成文件目录。

调用方也可以在初始化时传入自定义路径。

## 任务持久化

- `retrieveAllTasks()`：从 `downloadPath/<identifier>_Tasks.plist` 读取 `[DownloadTask]`，并恢复 cache 引用。读取到 `waiting` 状态会转为 `suspended`。
- `storeTasks(_:)`：使用 `PropertyListEncoder` 编码任务数组，并通过 `Debouncer` 延迟写入，降低磁盘写频率。
- 解码时会通过 `decoder.userInfo[.cache]` 把当前 cache 注入到任务中。

## 文件操作

- `createDirectory()`：确保根目录、临时目录和完成文件目录存在。
- `filePath(fileName:)`、`fileURL(fileName:)`、`fileExists(fileName:)`：按文件名访问完成文件。
- `filePath(url:)`、`fileURL(url:)`、`fileExists(url:)`：按 URL 默认文件名访问完成文件。
- `storeFile(at:to:)`：把 URLSession 下载完成的临时文件移动到最终位置。
- `storeTmpFile(_:)`：把系统临时目录中的断点文件复制到 `downloadTmpPath`。
- `retrieveTmpFile(_:)`：恢复断点文件到系统临时目录。
- `remove(_:,completely:)`：删除断点临时文件，必要时删除已完成文件。
- `clearDiskCache`：删除整个下载根目录后重新创建目录结构。

## URL 文件名扩展

文件末尾让 `URL` 支持 `tr.fileName`，规则是 URL 字符串 md5 加原始 path extension。该规则用于默认下载文件名。

## 注意点

任务列表使用二进制 plist 保存。`storeTasks` 是去抖异步写入，因此频繁状态变更不会立即落盘；如果外部需要强一致持久化，需要留意这一延迟行为。
