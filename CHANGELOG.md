# client-sdk-android

## 2.25.3

### 补丁变更

- docs(audio): 阐明 `AudioSwitchHandler.selectDevice()` 是粘性的，会覆盖 `preferredDeviceList`。文档中说明：仅需调整优先级顺序的调用者应改为设置 `preferredDeviceList`，而调用 `selectDevice(null)` 则可以清除粘性选择。 - [#941](https://github.com/livekit/client-sdk-android/pull/941) ([@daxiondi](https://github.com/daxiondi))

- 将 libwebrtc 更新至 144.7559.05 - [#936](https://github.com/livekit/client-sdk-android/pull/936) ([@davidliu](https://github.com/davidliu))

- fix: 在重连握手期间收到 LEAVE 时恢复 joinContinuation，以避免重连循环挂起的问题 - [#934](https://github.com/livekit/client-sdk-android/pull/934) ([@YashJainSC](https://github.com/YashJainSC))

- 修复了当 DataChannel.send 返回 false 以及缓冲项在多次恢复中重放时，可靠数据静默丢失的问题。 - [#921](https://github.com/livekit/client-sdk-android/pull/921) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复字节流未发送字节流名称的问题 - [#939](https://github.com/livekit/client-sdk-android/pull/939) ([@davidliu](https://github.com/davidliu))

- 更新 AudioSwitch 以处理注销音频设备监听器时可能出现的异常 - [#944](https://github.com/livekit/client-sdk-android/pull/944) ([@davidliu](https://github.com/davidliu))

## 2.25.2

### 补丁变更

- 修复 Room.connect 在 Room.join() 过程中遇到 WebSocket 连接失败时未正确抛出 ConnectException 的问题 - [#926](https://github.com/livekit/client-sdk-android/pull/926) ([@davidliu](https://github.com/davidliu))

- 修复重连可能因 WebSocket 失败而被取消的问题 - [#926](https://github.com/livekit/client-sdk-android/pull/926) ([@davidliu](https://github.com/davidliu))

- 修复了 RTCEngine.addTrack 在超时或调用方取消时泄漏 `pendingTrackResolvers` 条目的问题，此前这会导致同一轨道的后续发布因 `DuplicateTrackException` 而失败，直到连接被断开。 - [#920](https://github.com/livekit/client-sdk-android/pull/920) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复恢复后重新发送数据通道消息时出现的异常 - [#923](https://github.com/livekit/client-sdk-android/pull/923) ([@davidliu](https://github.com/davidliu))

## 2.25.1

### 补丁变更

- 为共享密钥加密向 E2EEOptions 添加便捷构造函数 - [#917](https://github.com/livekit/client-sdk-android/pull/917) ([@davidliu](https://github.com/davidliu))

## 2.25.0

### 次要变更

- AudioOptions: 添加 `disableAudioPrewarming` 标志 - [#912](https://github.com/livekit/client-sdk-android/pull/912) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 修复 StreamSender 因异常导致的潜在泄漏 - [#913](https://github.com/livekit/client-sdk-android/pull/913) ([@davidliu](https://github.com/davidliu))

- 更新音频处理，在 Android S 及以上版本使用 AudioManager 通信设备 API - [#910](https://github.com/livekit/client-sdk-android/pull/910) ([@davidliu](https://github.com/davidliu))

- 对协程重新抛出取消异常 - [#913](https://github.com/livekit/client-sdk-android/pull/913) ([@davidliu](https://github.com/davidliu))

- 实现通话中动态更改 `AudioSwitchHandler` 的首选音频设备列表 - [#910](https://github.com/livekit/client-sdk-android/pull/910) ([@davidliu](https://github.com/davidliu))

## 2.24.1

### 补丁变更

- 修复 LocalParticipant.publishData 在发送超过 15KB 的数据包时抛出异常的问题 - [#902](https://github.com/livekit/client-sdk-android/pull/902) ([@xianshijing-lk](https://github.com/xianshijing-lk))

## 2.24.0

### 次要变更

- 添加设置自定义重连策略的功能 - [#894](https://github.com/livekit/client-sdk-android/pull/894) ([@davidliu](https://github.com/davidliu))

- 将 libwebrtc 更新至 m144 - [#886](https://github.com/livekit/client-sdk-android/pull/886) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 修复连接断开/恢复后有时重连不生效的问题 - [#894](https://github.com/livekit/client-sdk-android/pull/894) ([@davidliu](https://github.com/davidliu))

- 修复转录属性未正确转换的问题 - [#889](https://github.com/livekit/client-sdk-android/pull/889) ([@davidliu](https://github.com/davidliu))

- 专门保留原生 libwebrtc 方法不被混淆 - [#893](https://github.com/livekit/client-sdk-android/pull/893) ([@davidliu](https://github.com/davidliu))

- 正确处理在 DataChannel 低缓冲时等待的 job，在 dispose 时取消而不是完成 - [#897](https://github.com/livekit/client-sdk-android/pull/897) ([@davidliu](https://github.com/davidliu))

- 修复使用 LocalParticipant.publishData 时未捕获的异常 - [#897](https://github.com/livekit/client-sdk-android/pull/897) ([@davidliu](https://github.com/davidliu))

## 2.23.5

### 补丁变更

- 修复未正确检测服务端支持的视频编解码器的问题 - [#876](https://github.com/livekit/client-sdk-android/pull/876) ([@davidliu](https://github.com/davidliu))

- 从依赖中移除 Timber - [#874](https://github.com/livekit/client-sdk-android/pull/874) ([@davidliu](https://github.com/davidliu))

## 2.23.4

### 补丁变更

- 改善自拍分割器效果 - [#861](https://github.com/livekit/client-sdk-android/pull/861) ([@Deneath](https://github.com/Deneath))

- 确保 ICE 重连超时时清理子 job - [#870](https://github.com/livekit/client-sdk-android/pull/870) ([@davidliu](https://github.com/davidliu))

- 当 join 协程被取消时取消 WebSocket - [#871](https://github.com/livekit/client-sdk-android/pull/871) ([@davidliu](https://github.com/davidliu))

- SignalClient 连接的并发修复 - [#871](https://github.com/livekit/client-sdk-android/pull/871) ([@davidliu](https://github.com/davidliu))

## 2.23.3

### 补丁变更

- 优化连接参数构建 - [#852](https://github.com/livekit/client-sdk-android/pull/852) ([@pulakdp](https://github.com/pulakdp))

- 修复了 ScreenCaptureConnection 在 bindService 失败时无限挂起以及在恢复已取消的 continuation 时崩溃的问题。 - [#838](https://github.com/livekit/client-sdk-android/pull/838) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 将 AgentAttribute 反序列化从 Klaxon 迁移至 kotlinx-serialization - [#851](https://github.com/livekit/client-sdk-android/pull/851) ([@davidliu](https://github.com/davidliu))

- perf: 对空的 agent attribute 映射跳过 Klaxon 解析 - [#849](https://github.com/livekit/client-sdk-android/pull/849) ([@YashJainSC](https://github.com/YashJainSC))

## 2.23.2

### 补丁变更

- 修复解析 AgentAttribute 的 inputs 和 outputs 时出现的异常 - [#847](https://github.com/livekit/client-sdk-android/pull/847) ([@davidliu](https://github.com/davidliu))

- 断连后正确重置网络回调管理器 - [#841](https://github.com/livekit/client-sdk-android/pull/841) ([@davidliu](https://github.com/davidliu))

- 修复了 pre-O 设备上因监听器实例不匹配导致音频焦点未被释放的问题。 - [#837](https://github.com/livekit/client-sdk-android/pull/837) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复了 ByteStreamSender 中读取后未关闭 Source 导致的文件描述符泄漏。 - [#839](https://github.com/livekit/client-sdk-android/pull/839) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 通过使用 `CopyOnWriteArraySet` 实现线程安全迭代，修复了 `CameraEventsDispatchHandler` 中的数据竞争问题。 - [#822](https://github.com/livekit/client-sdk-android/pull/822) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复了 Room 在连接尝试失败后陷入 CONNECTING 状态的问题。 - [#836](https://github.com/livekit/client-sdk-android/pull/836) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复 SignalClient 中可能导致消息静默丢弃的线程可见性问题。 - [#819](https://github.com/livekit/client-sdk-android/pull/819) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复轨道订阅之前发送同步状态的竞态条件 - [#831](https://github.com/livekit/client-sdk-android/pull/831) ([@davidliu](https://github.com/davidliu))

- 对于无效的 Agent 枚举类型值不再抛出异常 - [#847](https://github.com/livekit/client-sdk-android/pull/847) ([@davidliu](https://github.com/davidliu))

- 为 TokenSource.fromEndpoint 添加接受字符串 URL 的重载 - [#844](https://github.com/livekit/client-sdk-android/pull/844) ([@davidliu](https://github.com/davidliu))

- 修复了使用 VP9/AV1 编解码器时的屏幕共享问题 - [#828](https://github.com/livekit/client-sdk-android/pull/828) ([@pblazej](https://github.com/pblazej))

- ProGuard 规则优化 - [#803](https://github.com/livekit/client-sdk-android/pull/803) ([@davidliu](https://github.com/davidliu))

## 2.23.1

### 补丁变更

- 修复了 `onConnectionQuality` 中的非局部返回，当一个参与者在列表中未找到时，会导致其他参与者的连接质量更新丢失。 - [#817](https://github.com/livekit/client-sdk-android/pull/817) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复了 Room.onAddTrack 中不可能出现的空流检查，当 WebRTC 以空流数组调用 `onAddTrack` 时可能崩溃。 - [#816](https://github.com/livekit/client-sdk-android/pull/816) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 改进 `hasPublished` 标志的线程安全性和重连可靠性。 - [#814](https://github.com/livekit/client-sdk-android/pull/814) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复了当有 sink 注册时，`LocalAudioTrack.dispose()` 中的 `ConcurrentModificationException`。 - [#820](https://github.com/livekit/client-sdk-android/pull/820) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复取消发布轨道时 LocalParticipant 的 jobs map 清理问题。 - [#807](https://github.com/livekit/client-sdk-android/pull/807) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 修复了 PeerConnectionTransport 协程作用域在关闭时未被取消的问题。 - [#818](https://github.com/livekit/client-sdk-android/pull/818) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 通过加固状态标志和同步数据通道接收状态来提高重连可靠性。 - [#806](https://github.com/livekit/client-sdk-android/pull/806) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 移除对 OkHttp 内部方法的引用 - [#811](https://github.com/livekit/client-sdk-android/pull/811) ([@davidliu](https://github.com/davidliu))

- 通过丢弃过期的 SDP 应答和转发 offer ID 来提高 RTC 协商可靠性 - [#813](https://github.com/livekit/client-sdk-android/pull/813) ([@davidliu](https://github.com/davidliu))

- 修复了在发布构建中使用 AgentAttributes 时的 ProGuard/R8 混淆崩溃。为基于 Klaxon 的数据类添加了 `@Keep` 注解，并更新 ProGuard 规则以防止混淆基于反射的 JSON 解析类。#808 - [#809](https://github.com/livekit/client-sdk-android/pull/809) ([@adrian-niculescu](https://github.com/adrian-niculescu))

- 将 WebRTC-SDK 更新至 137.7151.05。 - [#824](https://github.com/livekit/client-sdk-android/pull/824) ([@davidliu](https://github.com/davidliu))

  - 修复回声消除和噪声抑制无法启用的问题。
  - 修复静音时麦克风未关闭的问题。

## 2.23.0

### 次要变更

- 将 TokenSource.fetch 方法改为返回 `Result<TokenSourceResponse>` 以显式处理异常 - [#802](https://github.com/livekit/client-sdk-android/pull/802) ([@davidliu](https://github.com/davidliu))

- 添加 AudioSwitchHandler 的多监听器支持 - [#802](https://github.com/livekit/client-sdk-android/pull/802) ([@davidliu](https://github.com/davidliu))

- 将 AgentState 重命名为 AgentSdkState - [#802](https://github.com/livekit/client-sdk-android/pull/802) ([@davidliu](https://github.com/davidliu))

- 废弃 `Room.withPreconnectAudio` 方法。 - [#802](https://github.com/livekit/client-sdk-android/pull/802) ([@davidliu](https://github.com/davidliu))

  - 改为在 RoomOptions 中设置 `AudioTrackPublishDefaults.preconnect = true` 来使用预连接缓冲。

- 将 `agentAttributes` 作为 Participant 的值暴露 - [#802](https://github.com/livekit/client-sdk-android/pull/802) ([@davidliu](https://github.com/davidliu))

- 在 Room 上暴露当前连接服务器的服务端信息 - [#802](https://github.com/livekit/client-sdk-android/pull/802) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 修复清理本地参与者时的崩溃 - [#802](https://github.com/livekit/client-sdk-android/pull/802) ([@davidliu](https://github.com/davidliu))

- 修复为通信模式变通方案创建音频轨道时的崩溃 - [#805](https://github.com/livekit/client-sdk-android/pull/805) ([@davidliu](https://github.com/davidliu))

## 2.22.1

### 补丁变更

- 修复网络断开后摄像头指示灯仍然保持开启的问题，通过释放重连失败尝试产生的孤立轨道（#296） - [#798](https://github.com/livekit/client-sdk-android/pull/798) ([@adrian-niculescu](https://github.com/adrian-niculescu))

## 2.22.0

### 次要变更

- 提取 CameraXProvider 并对外暴露。 - [#754](https://github.com/livekit/client-sdk-android/pull/754) ([@KasemJaffer](https://github.com/KasemJaffer))

### 补丁变更

- 修复摄像头查找需要检查 physicalId 的问题 - [#792](https://github.com/livekit/client-sdk-android/pull/792) ([@KasemJaffer](https://github.com/KasemJaffer))

## 2.21.1

### 补丁变更

- #721 修复发布者协商竞态条件导致 ICE 超时的问题。 - [#789](https://github.com/livekit/client-sdk-android/pull/789) ([@adrian-niculescu](https://github.com/adrian-niculescu))

## 2.21.0

### 次要变更

- 数据通道端到端加密选项 - [#762](https://github.com/livekit/client-sdk-android/pull/762) ([@davidliu](https://github.com/davidliu))

  - 向 DataReceived 事件和 StreamInfo 对象添加了 EncryptionType 字段，用于指示加密状态。

- 添加 TokenSource 实现，用于与 Token 服务器配合使用 - [#769](https://github.com/livekit/client-sdk-android/pull/769) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 将 RPC 方法最大往返时间提高到 7 秒 - [#775](https://github.com/livekit/client-sdk-android/pull/775) ([@davidliu](https://github.com/davidliu))

## 2.20.3

### 补丁变更

- 切换到基于 Header 的 Bearer Token 认证 - [#770](https://github.com/livekit/client-sdk-android/pull/770) ([@davidliu](https://github.com/davidliu))

## 2.20.2

### 补丁变更

- 移除 registerRpcMethod 上不必要的 suspend 修饰符 - [#757](https://github.com/livekit/client-sdk-android/pull/757) ([@davidliu](https://github.com/davidliu))

- 修复发布已释放轨道时的崩溃 - [#758](https://github.com/livekit/client-sdk-android/pull/758) ([@davidliu](https://github.com/davidliu))

- 修复释放 Room 对象时的竞态条件 - [#756](https://github.com/livekit/client-sdk-android/pull/756) ([@davidliu](https://github.com/davidliu))

- 修复 VirtualBackgroundVideoProcessor 未响应 backgroundImage 变化 - [#752](https://github.com/livekit/client-sdk-android/pull/752) ([@davidliu](https://github.com/davidliu))

- 确保 Room 在释放资源前已断开连接 - [#756](https://github.com/livekit/client-sdk-android/pull/756) ([@davidliu](https://github.com/davidliu))

## 2.20.1

### 补丁变更

- 修复额外 simulcast 层分辨率与原始分辨率相同时导致的崩溃 - [#749](https://github.com/livekit/client-sdk-android/pull/749) ([@davidliu](https://github.com/davidliu))

- 将 sendText 和 sendFile 中抛出的异常包装为 Result - [#749](https://github.com/livekit/client-sdk-android/pull/749) ([@davidliu](https://github.com/davidliu))

## 2.20.0

### 次要变更

- 将 libwebrtc 更新至 137.7151.03 - [#742](https://github.com/livekit/client-sdk-android/pull/742) ([@davidliu](https://github.com/davidliu))

- DataStream 发送辅助方法返回 streamInfo - [#741](https://github.com/livekit/client-sdk-android/pull/741) ([@davidliu](https://github.com/davidliu))

- 向 VideoTrackPublishOptions 添加 `simulcastLayers`，用于直接指定要使用的分辨率 - [#746](https://github.com/livekit/client-sdk-android/pull/746) ([@davidliu](https://github.com/davidliu))

- 添加 H265 作为支持的编解码器 - [#742](https://github.com/livekit/client-sdk-android/pull/742) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 数据通道的端到端可靠性，支持重连后重新发送 - [#738](https://github.com/livekit/client-sdk-android/pull/738) ([@davidliu](https://github.com/davidliu))

- 修复默认 simulcast 层使用低于预期分辨率的问题 - [#746](https://github.com/livekit/client-sdk-android/pull/746) ([@davidliu](https://github.com/davidliu))

- 手动发布屏幕共享轨道时正确使用 screenShareTrackPublishDefaults - [#746](https://github.com/livekit/client-sdk-android/pull/746) ([@davidliu](https://github.com/davidliu))

## 2.19.1

### 补丁变更

- 修复 ProGuard 规则以保留 JniInit 类用于 WebRTC 原生注册 (#735) - [#736](https://github.com/livekit/client-sdk-android/pull/736) ([@adrian-niculescu](https://github.com/adrian-niculescu))

## 2.19.0

### 次要变更

- 添加 Agent 和转录属性的类型 - [#733](https://github.com/livekit/client-sdk-android/pull/733) ([@davidliu](https://github.com/davidliu))

- 将 libwebrtc 更新至 137.7151.01 - [#727](https://github.com/livekit/client-sdk-android/pull/727) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 修复 localVideoTrack 切换摄像头时的 IllegalStateException - [#725](https://github.com/livekit/client-sdk-android/pull/725) ([@binkos](https://github.com/binkos))

- 使 VirtualBackgroundTransformer 中的 blurRadius 可动态调整。 - [#731](https://github.com/livekit/client-sdk-android/pull/731) ([@binkos](https://github.com/binkos))

- 为 LocalParticipant 添加 sendText 和 sendFile 辅助方法以方便使用 - [#732](https://github.com/livekit/client-sdk-android/pull/732) ([@davidliu](https://github.com/davidliu))

- 为 StreamByteOptions 添加默认名称 "unknown" 以支持无参构造 - [#732](https://github.com/livekit/client-sdk-android/pull/732) ([@davidliu](https://github.com/davidliu))

## 2.18.3

### 补丁变更

- 将音频处理移至后台线程以避免 UI 冻结。 - [#715](https://github.com/livekit/client-sdk-android/pull/715) ([@davidliu](https://github.com/davidliu))

## 2.18.2

### 补丁变更

- 正确处理 ByteStreamSender 便捷方法的 Result 返回 - [#709](https://github.com/livekit/client-sdk-android/pull/709) ([@davidliu](https://github.com/davidliu))

## 2.18.1

### 补丁变更

- 修复连接后无法立即发布的问题 - [#706](https://github.com/livekit/client-sdk-android/pull/706) ([@davidliu](https://github.com/davidliu))

## 2.18.0

### 次要变更

- 重构部分内部数据消息发送方法，使用 Result 替代抛出异常，以修复崩溃。 - [#703](https://github.com/livekit/client-sdk-android/pull/703) ([@davidliu](https://github.com/davidliu))

## 2.17.1

### 补丁变更

- 将 CameraX 依赖更新至 1.4.2 - [#694](https://github.com/livekit/client-sdk-android/pull/694) ([@davidliu](https://github.com/davidliu))

- 避免当轨道已被释放时重连导致的崩溃 - [#691](https://github.com/livekit/client-sdk-android/pull/691) ([@jeankruger](https://github.com/jeankruger))

## 2.17.0

### 次要变更

- 将 `isMicrophoneEnabled`、`isCameraEnabled`、`isScreenshareEnabled` 改为 FlowObservable 变量 - [#685](https://github.com/livekit/client-sdk-android/pull/685) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 修复当摄像头 ID 为物理 ID 时 switchCamera 不生效的问题 - [#676](https://github.com/livekit/client-sdk-android/pull/676) ([@KasemJaffer](https://github.com/KasemJaffer))

- 修复当 ByteBuffer 有 backing array 时发送预连接音频数据的问题 - [#678](https://github.com/livekit/client-sdk-android/pull/678) ([@davidliu](https://github.com/davidliu))

- 为 StreamTextOptions 和 streamText 指定默认值 - [#688](https://github.com/livekit/client-sdk-android/pull/688) ([@davidliu](https://github.com/davidliu))

## 2.16.0

### 次要变更

- 将 lossy 数据通道设为无序 - [#665](https://github.com/livekit/client-sdk-android/pull/665) ([@bcherry](https://github.com/bcherry))

- 添加预连接音频功能，用于 Agent 场景 - [#666](https://github.com/livekit/client-sdk-android/pull/666) ([@davidliu](https://github.com/davidliu))

  详见 `Room.withPreconnectAudio`。

- CameraX: 支持通过物理 ID 选择摄像头 - [#668](https://github.com/livekit/client-sdk-android/pull/668) ([@KasemJaffer](https://github.com/KasemJaffer))

- 添加 Participant.State 及相关事件 - [#666](https://github.com/livekit/client-sdk-android/pull/666) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 修复流式传输文本时的 NPE - [#670](https://github.com/livekit/client-sdk-android/pull/670) ([@davidliu](https://github.com/davidliu))

- 在 Room 类上添加 RPC handler 方法以方便使用。 - [#663](https://github.com/livekit/client-sdk-android/pull/663) ([@davidliu](https://github.com/davidliu))

- 修复传出数据流错误填充数据的问题 - [#666](https://github.com/livekit/client-sdk-android/pull/666) ([@davidliu](https://github.com/davidliu))

## 2.15.0

### 次要变更

- 添加 VirtualBackgroundVideoProcessor 和 track-processors 包 - [#660](https://github.com/livekit/client-sdk-android/pull/660) ([@davidliu](https://github.com/davidliu))

## 2.14.3

### 补丁变更

- 修复断连问题 - [#656](https://github.com/livekit/client-sdk-android/pull/656) ([@jeankruger](https://github.com/jeankruger))

## 2.14.2

### 补丁变更

- 修复 CameraXSession 未正确设置目标采集格式的问题 - [#652](https://github.com/livekit/client-sdk-android/pull/652) ([@davidliu](https://github.com/davidliu))

- 改进轨道发布失败的处理：引入新的 TrackPublicationFailed 事件，修复了轨道保持活跃但不可访问的错误状态问题（该问题导致麦克风或摄像头保持开启却没有已发布的轨道，并导致重新发布不可靠）。 - [#637](https://github.com/livekit/client-sdk-android/pull/637) ([@jeankruger](https://github.com/jeankruger))

## 2.14.1

### 补丁变更

- 修复 IncomingDataStreamManager 中的 ConcurrentModificationException - [#642](https://github.com/livekit/client-sdk-android/pull/642) ([@davidliu](https://github.com/davidliu))

- 去重支持的编解码器以提供有效的 SDP - [#643](https://github.com/livekit/client-sdk-android/pull/643) ([@davidliu](https://github.com/davidliu))

## 2.14.0

### 次要变更

- 实现数据流功能 - [#625](https://github.com/livekit/client-sdk-android/pull/625) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 停止时取消发布屏幕共享轨道，并引入 `ScreenCaptureParams`，可配置通知和设置 `onStop` 回调 - [#626](https://github.com/livekit/client-sdk-android/pull/626) ([@jeankruger](https://github.com/jeankruger))

## 2.13.0

### 次要变更

- 预热音频以加速麦克风发布 - [#623](https://github.com/livekit/client-sdk-android/pull/623) ([@davidliu](https://github.com/davidliu))

- 快速轨道发布支持 - [#612](https://github.com/livekit/client-sdk-android/pull/612) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 修复服务端无响应时的发布死锁问题 - [#618](https://github.com/livekit/client-sdk-android/pull/618) ([@davidliu](https://github.com/davidliu))

- 添加 SCREEN_SHARE_AUDIO 作为 Track.Source.Type - [#610](https://github.com/livekit/client-sdk-android/pull/610) ([@davidliu](https://github.com/davidliu))

- 在 ParticipantPermission 上暴露 canPublishSources、canUpdateMetadata 和 canSubscribeMetrics - [#610](https://github.com/livekit/client-sdk-android/pull/610) ([@davidliu](https://github.com/davidliu))

- 快速失败：无权限的发布尝试立即拒绝 - [#618](https://github.com/livekit/client-sdk-android/pull/618) ([@davidliu](https://github.com/davidliu))

## 2.12.3

### 补丁变更

- 修复发布轨道时的死锁问题 - [#604](https://github.com/livekit/client-sdk-android/pull/604) ([@jeankruger](https://github.com/jeankruger))

## 2.12.2

### 补丁变更

- 向 RPC 请求添加版本号 - [#605](https://github.com/livekit/client-sdk-android/pull/605) ([@davidliu](https://github.com/davidliu))

## 2.12.1

### 补丁变更

- 修正 AudioSwitchHandler 中 preferredDeviceList 的文档默认值 - [#584](https://github.com/livekit/client-sdk-android/pull/584) ([@davidliu](https://github.com/davidliu))

- 允许在 ParticipantAttributesChanged 事件中访问 participant 字段 - [#591](https://github.com/livekit/client-sdk-android/pull/591) ([@binkos](https://github.com/binkos))

## 2.12.0

### 次要变更

- 默认优先选择扬声器而非听筒 - [#579](https://github.com/livekit/client-sdk-android/pull/579) ([@davidliu](https://github.com/davidliu))

- 实现 RPC 功能 - [#578](https://github.com/livekit/client-sdk-android/pull/578) ([@davidliu](https://github.com/davidliu))

- 从 Room 显式暴露 AudioSwitchHandler 以方便音频处理 - [#579](https://github.com/livekit/client-sdk-android/pull/579) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 添加 publishDTMF 方法，用于向 SIP 参与者发送 DTMF 信号 - [#576](https://github.com/livekit/client-sdk-android/pull/576) ([@dipak140](https://github.com/dipak140))

## 2.11.1

### 补丁变更

- 修复 maxFps 在极低帧率下不生效的问题 - [#573](https://github.com/livekit/client-sdk-android/pull/573) ([@davidliu](https://github.com/davidliu))

## 2.11.0

### 次要变更

- 向 CameraX createCameraProvider 添加 UseCase 参数 - [#536](https://github.com/livekit/client-sdk-android/pull/536) ([@KasemJaffer](https://github.com/KasemJaffer))

- 为屏幕共享轨道检测旋转 - [#552](https://github.com/livekit/client-sdk-android/pull/552) ([@davidliu](https://github.com/davidliu))

- 默认对摄像头输出进行缩放裁剪以适应目标尺寸 - [#558](https://github.com/livekit/client-sdk-android/pull/558) ([@davidliu](https://github.com/davidliu))

  - 可通过 `VideoCaptureParams.adaptOutputToDimensions` 关闭此行为。

- 为屏幕共享轨道添加独立的默认采集/发布选项 - [#552](https://github.com/livekit/client-sdk-android/pull/552) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 添加 AudioPresets 并将默认音频最大比特率提高到 48kbps - [#551](https://github.com/livekit/client-sdk-android/pull/551) ([@davidliu](https://github.com/davidliu))

- 修复设置发布层时的崩溃 - [#559](https://github.com/livekit/client-sdk-android/pull/559) ([@davidliu](https://github.com/davidliu))

- 添加 VideoFrameCapturer，支持直接推送视频帧 - [#538](https://github.com/livekit/client-sdk-android/pull/538) ([@davidliu](https://github.com/davidliu))

- 修复部分设备上 surface 导致空指针异常的问题 - [#544](https://github.com/livekit/client-sdk-android/pull/544) ([@KasemJaffer](https://github.com/KasemJaffer))

- 将 Kotlin 依赖更新至 1.9.25 - [#552](https://github.com/livekit/client-sdk-android/pull/552) ([@davidliu](https://github.com/davidliu))

## 2.10.0

### 次要变更

- 实现自定义音频混音到音频轨道中 - [#528](https://github.com/livekit/client-sdk-android/pull/528) ([@davidliu](https://github.com/davidliu))

- 更新 webrtc-sdk 到 125.6422.06.1 - [#528](https://github.com/livekit/client-sdk-android/pull/528) ([@davidliu](https://github.com/davidliu))

- 实现屏幕共享音频采集器 - [#528](https://github.com/livekit/client-sdk-android/pull/528) ([@davidliu](https://github.com/davidliu))

## 2.9.0

### 次要变更

- 实现 `LocalAudioTrack.addSink` 以接收本地麦克风的音频数据 - [#516](https://github.com/livekit/client-sdk-android/pull/516) ([@davidliu](https://github.com/davidliu))

- 实现客户端指标 - [#511](https://github.com/livekit/client-sdk-android/pull/511) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 在 RTC 线程上正确释放 Peer Connection - [#506](https://github.com/livekit/client-sdk-android/pull/506) ([@davidliu](https://github.com/davidliu))

- LocalParticipant 方法的文档更新 - [#510](https://github.com/livekit/client-sdk-android/pull/510) ([@davidliu](https://github.com/davidliu))

- 仅初始化 WebRTC 库一次 - [#508](https://github.com/livekit/client-sdk-android/pull/508) ([@davidliu](https://github.com/davidliu))

## 2.8.1

### 补丁变更

- 修复本地视频轨道不渲染处理后帧的问题 - [#495](https://github.com/livekit/client-sdk-android/pull/495) ([@davidliu](https://github.com/davidliu))

- 添加 NoDropVideoProcessor 工具类，用于在未连接时强制进行视频处理 - [#495](https://github.com/livekit/client-sdk-android/pull/495) ([@davidliu](https://github.com/davidliu))

- 进一步修复使用已释放轨道导致的崩溃 - [#497](https://github.com/livekit/client-sdk-android/pull/497) ([@davidliu](https://github.com/davidliu))

## 2.8.0

### 次要变更

- 实现 LocalTrackSubscribed 事件 - [#489](https://github.com/livekit/client-sdk-android/pull/489) ([@davidliu](https://github.com/davidliu))

- 向 TranscriptionSegment 添加首次和最后接收时间 - [#485](https://github.com/livekit/client-sdk-android/pull/485) ([@davidliu](https://github.com/davidliu))

### 补丁变更

- 进一步保护 RTC API 调用以防止崩溃 - [#488](https://github.com/livekit/client-sdk-android/pull/488) ([@davidliu](https://github.com/davidliu))

## 2.7.1

### 补丁变更

- 当 VideoRenderer 在未初始化的情况下使用时，以明显级别记录日志 - [#482](https://github.com/livekit/client-sdk-android/pull/482) ([@davidliu](https://github.com/davidliu))

- 修复 RegionProvider 中无法确定 host 时的 NPE - [#482](https://github.com/livekit/client-sdk-android/pull/482) ([@davidliu](https://github.com/davidliu))
