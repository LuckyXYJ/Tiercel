# String+Hash.swift

对应文件：`Sources/Extensions/String+Hash.swift`

## 文件职责

该文件为 `String` 增加哈希计算能力，并让字符串遵循 `TiercelCompatible` 以使用 `tr` 命名空间。

## 核心 API

通过字符串的 `tr` 命名空间访问：

- `string.tr.md5`
- `string.tr.sha1`
- `string.tr.sha256`
- `string.tr.sha512`

每个属性都会先把字符串按 UTF-8 转为 `Data`，再复用 `Data+Hash.swift` 的哈希实现。

## 使用位置

`Cache` 中的 `URL.tr.fileName` 会对 URL 字符串计算 md5，生成默认下载文件名。

## 注意点

如果字符串无法按 UTF-8 转换为 Data，会直接返回原字符串。不过 Swift 字符串通常可以成功编码为 UTF-8，因此该分支主要是防御性处理。
