# Data+Hash.swift

对应文件：`Sources/Extensions/Data+Hash.swift`

## 文件职责

该文件为 `Data` 增加哈希计算能力，支持 MD5、SHA1、SHA256 和 SHA512。

## 核心 API

通过 `Data` 的 `tr` 命名空间访问：

- `data.tr.md5`
- `data.tr.sha1`
- `data.tr.sha256`
- `data.tr.sha512`

每个属性都会使用 CommonCrypto 计算摘要，并把字节结果格式化为小写十六进制字符串。

## 使用位置

- `String+Hash.swift` 会把字符串转为 UTF-8 Data 后复用这些实现。
- `FileChecksumHelper.validateFile` 使用这些哈希值校验下载文件。
- `Cache` 中 URL 默认文件名最终依赖 `String.tr.md5`，间接使用 Data 哈希。

## 注意点

该文件依赖 `CommonCrypto`。返回值固定为十六进制小写字符串，校验时与用户传入 code 会做大小写无关比较。
