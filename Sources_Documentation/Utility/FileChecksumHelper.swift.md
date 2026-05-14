# FileChecksumHelper.swift

对应文件：`Sources/Utility/FileChecksumHelper.swift`

## 文件职责

该文件提供下载完成文件的校验能力，支持 MD5、SHA1、SHA256 和 SHA512。

## 核心类型

- `VerificationType`：校验算法枚举，包含 `md5`、`sha1`、`sha256`、`sha512`。
- `FileVerificationError`：校验错误，包含验证码为空、验证码不匹配、文件不存在、读取数据失败。

## 核心 API

`validateFile(_:code:type:completion:)` 会：

1. 检查传入的校验码是否为空。
2. 在并发 `ioQueue` 中检查文件是否存在。
3. 使用 `Data(contentsOf:options: .mappedIfSafe)` 读取文件。
4. 根据 `VerificationType` 调用 `Data.tr.md5`、`sha1`、`sha256` 或 `sha512`。
5. 忽略大小写比较计算结果与传入校验码。
6. 通过 `Result<Bool, FileVerificationError>` 返回结果。

## 使用位置

`DownloadTask.validateFile(code:type:onMainQueue:handler:)` 会调用该工具，并根据结果更新任务的 `validation` 状态。

## 注意点

该实现会一次性把文件映射/读取为 `Data`，对超大文件可能带来内存或 I/O 压力。当前代码适合常规下载文件校验。
