# OperationQueue+DispatchQueue.swift

对应文件：`Sources/Extensions/OperationQueue+DispatchQueue.swift`

## 文件职责

该文件为 `OperationQueue` 增加便利初始化方法，方便一次性设置服务质量、最大并发数、底层 GCD 队列和名称。

## 核心 API

便利初始化参数包括：

- `qualityOfService`
- `maxConcurrentOperationCount`
- `underlyingQueue`
- `name`

初始化内部先调用 `self.init()`，再逐项设置属性。

## 使用位置

`SessionManager.createSession(_:)` 使用该初始化方法创建 URLSession delegate queue，并指定：

- `maxConcurrentOperationCount: 1`
- `underlyingQueue: operationQueue`
- `name: "com.Tiercel.SessionManager.delegateQueue"`

这样 URLSession delegate 回调会串行落到 manager 的操作队列上。

## 注意点

该扩展没有声明为 `public`，因此主要供模块内部使用。
