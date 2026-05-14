# Info.plist

对应文件：`Sources/Info.plist`

## 文件职责

该文件是 Tiercel framework target 的 Info.plist 模板，负责向构建系统提供 bundle 元数据。文件中的值大多使用 Xcode 构建变量，在编译时由 target 的 Build Settings 注入。

## 主要配置项

- `CFBundleDevelopmentRegion`：使用 `$(DEVELOPMENT_LANGUAGE)`，表示本地化开发语言。
- `CFBundleExecutable`：使用 `$(EXECUTABLE_NAME)`，表示最终产物的可执行文件名。
- `CFBundleIdentifier`：使用 `$(PRODUCT_BUNDLE_IDENTIFIER)`，表示 framework 的 bundle id。
- `CFBundleInfoDictionaryVersion`：固定为 `6.0`，是 Info.plist 格式版本。
- `CFBundleName`：使用 `$(PRODUCT_NAME)`，表示产品名称。
- `CFBundlePackageType`：使用 `$(PRODUCT_BUNDLE_PACKAGE_TYPE)`，表示 bundle 类型。
- `CFBundleShortVersionString`：使用 `$(MARKETING_VERSION)`，表示对外展示版本号。
- `CFBundleVersion`：使用 `$(CURRENT_PROJECT_VERSION)`，表示构建版本号。

## 与代码的关系

源码中没有直接读取该文件的逻辑；它主要服务于 Apple 平台 framework 打包、版本展示和 bundle 识别。

## 注意点

修改该文件通常会影响打包产物元信息，而不会改变下载逻辑本身。版本相关字段应优先在 Xcode Build Settings 中维护，避免模板值与构建设置不一致。
