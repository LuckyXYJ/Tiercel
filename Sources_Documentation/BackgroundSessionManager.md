# Background SessionManager Usage

本文说明 Tiercel 在使用后台下载时，`SessionManager` 是否必须在 `AppDelegate` 中初始化，以及如何支持按用户或业务场景动态创建多个 `SessionManager`。

## 是否必须在 AppDelegate 初始化

不一定要在 `AppDelegate` 中提前初始化所有 `SessionManager`。

如果使用 `URLSessionConfiguration.background(withIdentifier:)`，则必须处理：

```swift
application(_:handleEventsForBackgroundURLSession:completionHandler:)
```

原因是后台传输完成或系统需要把后台 URLSession 事件交还给 app 时，iOS 会唤醒应用并调用该方法。应用需要根据系统传入的 background session identifier 找回或重建同一个后台 session，使系统任务能够重新关联到业务层对象。

因此，核心要求不是“必须提前创建所有 manager”，而是：

- background session identifier 必须稳定。
- background session identifier 必须唯一。
- 收到系统回调时，必须能根据 identifier 找回或重建对应的 `SessionManager`。
- 必须保存并在后台事件处理完成后调用系统传入的 `completionHandler`。

## 动态创建多个 SessionManager

可以根据用户、业务空间、下载分组等动态创建多个 `SessionManager`。例如：

- 每个用户一个 `SessionManager`。
- 每个业务模块一个 `SessionManager`。
- 每个隔离下载空间一个 `SessionManager`。

即使创建了很多 `SessionManager`，也可以通过统一注册表控制同一时间只允许一个 manager 下载，其他 manager 暂停。

推荐做法是增加一个全局注册表，例如 `SessionManagerRegistry` 或 `DownloadManagerCenter`。

## Registry 职责

注册表建议负责以下事情：

- 按业务 key 创建或复用 `SessionManager`。
- 维护 `identifier -> SessionManager` 映射。
- 维护 `identifier -> completionHandler` 映射。
- 在 `handleEventsForBackgroundURLSession` 中按 identifier 懒加载 manager。
- 启动某个 manager 前，暂停其他 manager。

## 示例结构

```swift
final class SessionManagerRegistry {
    static let shared = SessionManagerRegistry()

    private var managers: [String: SessionManager] = [:]
    private var completionHandlers: [String: () -> Void] = [:]

    func manager(for key: String) -> SessionManager {
        let shortIdentifier = "download.\(key)"

        let fullIdentifier = makeFullIdentifier(shortIdentifier)
        if let manager = managers[fullIdentifier] {
            return manager
        }

        let manager = SessionManager(shortIdentifier, configuration: SessionConfiguration())
        bindCompletionHandler(to: manager)
        managers[manager.identifier] = manager
        return manager
    }

    func handleBackgroundEvents(identifier: String, completionHandler: @escaping () -> Void) {
        completionHandlers[identifier] = completionHandler

        if let manager = managers[identifier] {
            bindCompletionHandler(to: manager)
            return
        }

        guard let shortIdentifier = parseShortIdentifier(from: identifier) else {
            completionHandlers.removeValue(forKey: identifier)?()
            return
        }

        let manager = SessionManager(shortIdentifier, configuration: SessionConfiguration())
        bindCompletionHandler(to: manager)
        managers[manager.identifier] = manager
    }

    func startOnly(_ active: SessionManager) {
        for manager in managers.values where manager !== active {
            manager.totalSuspend()
        }
        active.totalStart()
    }

    private func bindCompletionHandler(to manager: SessionManager) {
        manager.completionHandler = { [weak self, weak manager] in
            guard let identifier = manager?.identifier else { return }
            self?.completionHandlers.removeValue(forKey: identifier)?()
        }
    }

    private func makeFullIdentifier(_ shortIdentifier: String) -> String {
        let bundleIdentifier = Bundle.main.bundleIdentifier ?? "com.Daniels.Tiercel"
        return "\(bundleIdentifier).\(shortIdentifier)"
    }

    private func parseShortIdentifier(from fullIdentifier: String) -> String? {
        let bundleIdentifier = Bundle.main.bundleIdentifier ?? "com.Daniels.Tiercel"
        let prefix = "\(bundleIdentifier)."
        guard fullIdentifier.hasPrefix(prefix) else { return nil }
        return String(fullIdentifier.dropFirst(prefix.count))
    }
}
```

`AppDelegate` 中只需要转发系统回调：

```swift
func application(
    _ application: UIApplication,
    handleEventsForBackgroundURLSession identifier: String,
    completionHandler: @escaping () -> Void
) {
    SessionManagerRegistry.shared.handleBackgroundEvents(
        identifier: identifier,
        completionHandler: completionHandler
    )
}
```

## identifier 注意事项

当前 `SessionManager` 初始化时会把外部传入的 identifier 拼接为完整 background session identifier：

```swift
self.identifier = "\(bundleIdentifier).\(identifier)"
```

这意味着调用方传入的是短 identifier，例如：

```swift
SessionManager("download.userA", configuration: configuration)
```

实际注册到系统的 identifier 会是：

```text
<bundleIdentifier>.download.userA
```

而 `handleEventsForBackgroundURLSession` 收到的是完整 identifier。

如果要动态重建 manager，需要避免把完整 identifier 再传入 `SessionManager`，否则会被重复拼接，导致 identifier 不一致。

可选改造方向：

- registry 保存短 identifier 与完整 identifier 的映射。
- registry 从完整 identifier 中解析出短 identifier。
- 给 `SessionManager` 增加一个支持完整 identifier 的初始化入口。

## 同一时间只允许一个 Manager 下载

可以在注册表中统一控制下载互斥：

```swift
func startOnly(_ active: SessionManager) {
    for manager in managers.values where manager !== active {
        manager.totalSuspend()
    }
    active.totalStart()
}
```

这样可以支持“存在多个 manager，但同一时间只有一个 manager 处于下载状态”的业务需求。

## 结论

`SessionManager` 不必全部在 `AppDelegate` 里提前初始化。更适合动态场景的方式是使用一个全局 registry，根据业务 key 懒加载 manager，并在 `handleEventsForBackgroundURLSession` 中根据系统传入的 identifier 找回或重建对应 manager。

无论采用哪种方式，只要使用后台 URLSession，就必须集中处理系统后台 session 回调，并确保 completion handler 被正确保存和调用。
