# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用命令

- 整体构建：`./gradlew assemble`
- 运行所有单元测试：`./gradlew test`
- 运行单个测试：`./gradlew :livekit-android-test:test --tests "io.livekit.android.test.ClassName.methodName"`
- 运行指定模块的测试：`./gradlew :livekit-android-test:test`
- 静态分析（detekt）：`./gradlew detektDebug`（带基线：`detektRelease`）
- 代码格式检查：`./gradlew spotless`
- 自动格式化代码：`./gradlew spotlessApply`
- 安装 Compose 示例应用：`./gradlew sample-app-compose:installDebug`
- 安装基础示例应用：`./gradlew sample-app-basic:installDebug`
- 构建指定模块：`./gradlew :livekit-android-sdk:assemble`

## 首次搭建

```bash
git submodule update --init  # 拉取 protocol protobuf 定义
```

在 Apple Silicon Mac 上，需要在 `~/.gradle/gradle.properties` 中添加：
```
protoc_platform=osx-x86_64
```

## 架构

SDK 以三个 Maven 构件发布：
- `livekit-android-sdk` — 核心 SDK（WebRTC 信令、房间管理、轨道、参与者）
- `livekit-android-camerax` — CameraX 集成（捏合缩放、闪光灯控制等）
- `livekit-android-track-processors` — 视频轨道处理器（如虚拟背景）

### 模块概览

| 模块 | 用途 |
|---|---|
| `livekit-android-sdk` | 核心 SDK — 入口、房间、参与者、轨道、信令 |
| `livekit-android-camerax` | CameraX 视频采集扩展 |
| `livekit-android-track-processors` | 采集后视频处理（虚拟背景等） |
| `livekit-android-test` | 测试工具、mock、`MockE2ETest` 基类。SDK 的友元模块（可访问 `internal`） |
| `livekit-detekt-rules` | 自定义 detekt 规则 |
| `livekit-lint` | 自定义 Android lint 检查 |
| `sample-app-common` | 示例应用共享的 `CallViewModel` |
| `sample-app-compose` / `sample-app` / `sample-app-basic` | 示例应用 |

### 核心 SDK 包结构（`io.livekit.android`）

- `LiveKit` — 主入口；`LiveKit.create(context)` 创建 `Room`
- `Room` — 用户交互的主要类；管理连接、参与者、轨道。持有协程作用域
- `ConnectOptions` / `RoomOptions` / `LiveKitOverrides` — 配置类
- `room/` — `SignalClient`（WebSocket 信令）、`RTCEngine`、`PeerConnectionTransport`，以及参与者/轨道子包
- `room/participant/` — `LocalParticipant`、`RemoteParticipant`，继承自 `Participant`
- `room/track/` — `LocalAudioTrack`、`LocalVideoTrack`、`RemoteAudioTrack`、`RemoteVideoTrack` 及其 `*Publication` 包装类
- `events/` — `RoomEvent`、`TrackEvent`、`ParticipantEvent` 密封类层次结构
- `audio/` — `AudioHandler`、音频路由/处理
- `renderer/` — `SurfaceViewRenderer`、`TextureViewRenderer`
- `e2ee/` — 端到端加密支持
- `token/` — `TokenSource` 接口及认证实现
- `dagger/` — Dagger 依赖注入组件和模块（内部）
- `webrtc/` — WebRTC 辅助扩展和 PeerConnection 封装
- `util/` — `FlowDelegate`（`@FlowObservable` 机制）、`LKLog` 及通用工具

### 核心设计模式

**FlowObservable / FlowDelegate** — 标注 `@FlowObservable` 的属性既可以作为普通值读取，也可以通过 `.flow` 扩展属性作为 Kotlin `Flow` 进行观察。这使得 Compose 的 `collectAsState()` 集成成为可能。使用 `var x by flowDelegate(initial)` 创建。

**Dagger 依赖注入** — 整个 SDK 大量使用。通过 `@Inject` 或 `@AssistedInject` 构造函数注入依赖。可变依赖使用 `@AssistedFactory`。

**线程模型** — 所有 WebRTC API 调用必须通过 `executeOnRTCThread`、`executeBlockingOnRTCThread` 或 `launchBlockingOnRTCThread` 并传入 `RTCThreadToken`。SDK 类拥有自己的协程作用域。

**SignalClient** — 与 LiveKit 服务器的 WebSocket 连接，负责信令。与 `RTCEngine` 配合工作，后者整合信令和 WebRTC 媒体传输。

### 测试

测试位于 `livekit-android-test` 模块，作为 SDK 的"友元"模块（可访问 `internal`）。基类：
- `MockE2ETest` — 使用 mock 依赖测试 `Room` 行为的基类
- `BaseTest` — 通用测试基类

### 发布与 CI

- 版本号在 `gradle.properties` 中（`VERSION_NAME`）。开发阶段使用 SNAPSHOT 版本，发布时去掉 `-SNAPSHOT`。
- CI 通过 `publish` GitHub Action 发布到 Maven Central，由带 `v` 前缀的 tag 触发。
- 发布流程：更新 `VERSION_NAME`，打 tag `v[版本号]`，推送，确认 publish action 成功，然后升版本并加回 `-SNAPSHOT`。

## PR 工作流

PR 需要 changeset 文件：
```bash
pnpm install
pnpm changeset
```
提交前格式化代码：`./gradlew spotlessApply`

## 编码规范

- Kotlin 1.9.25，Java 8 目标版本，最低 API 21，编译/目标 SDK 35
- 遵循 Kotlin 官方代码风格；Kotlin/Java 使用 4 空格缩进，配置文件使用 2 空格
- **不允许**通过未检查异常导致消费者代码崩溃。优先使用 `Result` 而非抛出异常。可能抛出异常的方法必须用 `@throws` 注解。
- 避免使用 `assert()` / `Preconditions`。在编译时通过密封类、类型状态等预判无效状态。
- 每个公开 API 必须有 Kdoc。在入口类中添加简短代码示例。
- 使用 `LKLog` 记录日志，不要使用 `Log` 或 `println`。
- 一致性优先于最新语法糖。
- Ratchet 格式化：`spotless` 配置了 `ratchetFrom 'origin/main'`——仅检查变更的文件。
- 公开 API 中的废弃警告是允许的，不要修复它们。
