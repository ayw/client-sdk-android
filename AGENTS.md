# LiveKit Android SDK

## 命令

支持的平台：Android（最低 API 级别 21）

- 构建：`./gradlew assemble`
- 运行测试：`./gradlew test`
- 运行 detekt（静态分析）：`./gradlew detektDebug`
- 运行 detekt 带基线：`./gradlew detektRelease`
- 安装示例应用：`./gradlew sample-app-compose:installDebug`

## 架构

SDK 通过 `livekit-android-sdk` Gradle 模块提供（包根路径为 `io.livekit.android`）。

```
livekit-android-sdk/src/main/java/io/livekit/android
├── annotations/           # API 标记注解（如 @Beta）
├── audio/                 # AudioHandler, AudioProcessingController
├── coroutines/            # 协程相关工具方法
├── dagger/                # LiveKit SDK 内部依赖注入
├── e2ee/                  # 端到端加密
├── events/                # RoomEvent, TrackEvent, ParticipantEvent
├── memory/                # 资源生命周期辅助工具
├── renderer/              # 视频渲染视图
├── room/                  # Room, SignalClient, RTCEngine, tracks
│   ├── datastream/        # 输入/输出数据流 IO
│   ├── metrics/           # RTC 指标
│   ├── network/           # 重连和网络回调
│   ├── participant/       # LocalParticipant, RemoteParticipant
│   ├── provisions/        # 内部供应辅助工具
│   ├── rpc/               # 房间级 RPC
│   ├── track/             # 音频/视频/屏幕采集轨道及其发布
│   ├── types/             # 共享的房间相关类型
│   └── util/              # 房间内部工具
├── rpc/                   # RPC 错误类型（包级别）
├── stats/                 # 客户端/网络统计辅助工具
├── token/                 # TokenSource 认证实现
├── util/                  # 通用工具方法、日志、FlowDelegate
└── webrtc/                # WebRTC 辅助类和扩展
```

入口类（如 `LiveKit` 和 `ConnectOptions`）与上述目录一起放在 `io.livekit.android` 包中。

核心组件：

- `LiveKit` - 主入口点；创建 `Room` 对象。
- `Room` - 用户交互的主要类；管理连接状态、参与者和轨道。
- `Participant` - `LocalParticipant`/`RemoteParticipant` 的基类；持有轨道发布。
- `SignalClient` - 与 LiveKit 服务器的 WebSocket 连接。
- `FlowDelegate` - 将标记为 `@FlowObservable` 的类成员通过 `flow` 扩展属性提供为 `Flow`。

## WebRTC

WebRTC 负责参与方之间的实际媒体传输（音频/视频/数据）。SDK 通过 `Room`、`Participant` 和 `Track` API 封装了 WebRTC 的复杂性，而 LiveKit 服务器负责协调信令。

核心类：

- `PeerConnectionTransport` - 封装 `PeerConnection`；处理 ICE 候选、SDP 提议/应答。
- `RTCEngine` - 将 SignalClient 和 PeerConnectionTransport 整合为统一的连接。
- `io.livekit.android.webrtc` 包 - WebRTC 类型的便捷扩展。

线程：

- 所有 WebRTC API 调用必须使用 `executeOnRTCThread`、`executeBlockingOnRTCThread` 或 `launchBlockingOnRTCThread` 以确保线程安全。
- 每个调用都需要一个管理线程执行请求的 `RTCThreadToken`。

## 依赖注入

此库大量使用 Dagger 来提供整个代码库的依赖注入。

- 依赖需求应通过注入到 `@Inject` 或 `@AssistedInject` 注解的构造函数中来满足。
- 可变依赖（如 ID、不同的实现）可以通过 `@AssistedFactory` 注解来提供。

## FlowObservable

SDK 大量依赖 `@FlowObservable` 修饰的类成员，使它们可以像普通变量一样使用，同时也可以作为 `Flow` 被观察。这对 Android Compose 项目尤其有用，允许将其转换为 `State` 对象以正确更新 UI。

```kotlin
val identity = participant.identity // 普通访问
val identityFlow = participant::identity.flow // 作为 flow
val identityState = participant::identity.flow.collectAsState() // 作为 state
```

`@FlowObservable` 类成员可以通过 `flowDelegate` 属性委托创建：

```kotlin
@FlowObservable
@get:FlowObservable
var identity: Identity? by flowDelegate(identity)
```

## 测试

单元测试通过 `livekit-android-test` 模块提供。

- `io.livekit.android.test.mock` 包 - mock 和 fake。
- `MockE2ETest` - 测试 `Room` 行为时的基类。

`livekit-android-test` 被设置为 `livekit-android-sdk` 的友元模块，可以测试内部类。但测试中应尽量避免直接调用内部方法。

## 使用 Kotlin

### 并发与状态

- SDK 使用协程进行后台线程处理。
- 使用协程的类应创建并拥有自己的协程作用域。

### 错误处理

- **不允许**通过未检查的异常导致消费者代码崩溃。
- 应避免使用 `assert()`/`Preconditions`。
- 优先返回 `Result` 而非抛出异常。
- 可能抛出异常的方法必须在文档中使用 `@throws` 标注。
- 对于可恢复的错误，优先采取防御性编程（重试、退避、优雅降级）。
- 使用代数数据类型、类型状态等在编译时预判无效状态。

### 编码风格

- 遵循 Kotlin 官方代码风格。
- 各功能之间的一致性比最新的语法糖更重要。
- 运行 `./gradlew spotless` 检查代码风格；**不要**引入新的警告。
- 公开 API 中的废弃警告是允许的，不要修复它们。
- `// 代码注释`应尽量少用；优先通过更好的命名/结构来表达。
- 不要添加无意义的"是什么"注释，如 `// 这里是修改`。
- **每个**公开 API 都需要 KDoc 注释。
- 在入口类（如 `Room`）中为新 API 添加简短的代码示例。
- 使用 `LKLog` 进行日志记录。

<skills_system priority="1">

## 可用技能

<!-- SKILLS_TABLE_START -->
<usage>
当用户要求你执行任务时，检查以下可用技能是否可以帮助更高效地完成任务。技能提供专门的能力和领域知识。

如何使用技能：

- 调用：`npx openskills read <skill-name>`（在 shell 中运行）
    - 多个：`npx openskills read skill-one,skill-two`
- 技能内容将加载详细的任务完成说明
- 输出中提供的基目录用于解析捆绑的资源（references/、scripts/、assets/）

使用说明：

- 仅使用下面 <available_skills> 中列出的技能
- 不要调用已经加载到上下文中的技能
- 每次技能调用都是无状态的
  </usage>

<available_skills>

<skill>
<name>android-coroutines</name>
<description>面向 Android 的生产级 Kotlin 协程的权威规则和模式。涵盖结构化并发、生命周期集成和响应式流。</description>
<location>project</location>
</skill>

<skill>
<name>android-gradle-logic</name>
<description>使用 Convention Plugins 和 Version Catalogs 搭建可扩展 Gradle 构建逻辑的专家指导。</description>
<location>project</location>
</skill>

<skill>
<name>android-testing</name>
<description>涉及单元测试、集成测试、Hilt 测试和截图测试的综合测试策略。</description>
<location>project</location>
</skill>

<skill>
<name>gradle-build-performance</name>
<description>调试和优化 Android/Gradle 构建性能。适用于构建速度慢、分析 CI/CD 性能、分析构建扫描或识别编译瓶颈时。</description>
<location>project</location>
</skill>

<skill>
<name>kotlin-concurrency-expert</name>
<description>面向 Android 的 Kotlin 协程审查和修复。适用于审查协程使用、修复协程相关错误、改进线程安全或解决 Kotlin/Android 代码中的生命周期问题。</description>
<location>project</location>
</skill>

</available_skills>
<!-- SKILLS_TABLE_END -->

</skills_system>
