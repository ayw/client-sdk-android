<!--BEGIN_BANNER_IMAGE-->

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="/.github/banner_dark.png">
  <source media="(prefers-color-scheme: light)" srcset="/.github/banner_light.png">
  <img style="width:100%;" alt="LiveKit 图标、仓库名称以及一些背景示例代码。" src="https://raw.githubusercontent.com/livekit/client-sdk-android/main/.github/banner_light.png">
</picture>

<!--END_BANNER_IMAGE-->

# LiveKit Android Kotlin SDK
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.livekit/livekit-android/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.livekit/livekit-android)
<!--BEGIN_DESCRIPTION-->
使用此 SDK 为你的 Android/Kotlin 应用添加实时视频、音频和数据功能。通过连接到 <a href="https://livekit.io/">LiveKit</a> Cloud 或自托管服务器，只需几行代码即可构建多模态 AI、直播或视频通话等应用。
<!--END_DESCRIPTION-->

# 目录

- [文档](#文档)
- [安装](#安装)
- [使用方法](#使用方法)
    - [权限](#权限)
    - [发布摄像头和麦克风](#发布摄像头和麦克风)
    - [屏幕共享](#屏幕共享)
    - [渲染订阅的轨道](#渲染订阅的轨道)
    - [音频模式](#音频模式)
    - [@FlowObservable](#flowobservable)
- [示例应用](#示例应用)
- [开发环境](#开发环境)
    - [可选（开发便利）](#可选开发便利)

## 文档

文档和指南请访问 [https://docs.livekit.io](https://docs.livekit.io)。

API 参考文档请访问 [https://docs.livekit.io/client-sdk-android/index.html](https://docs.livekit.io/client-sdk-android/index.html)。

> [!NOTE]
> 这是 Android SDK 的 v2 版本。从 v1.x 迁移到 v2.x 时可能会遇到少量破坏性变更。
> 请阅读[迁移指南](https://docs.livekit.io/recipes/migrate-from-v1/)了解变更的详细概述。

## 安装

LiveKit for Android 以 Maven 包的形式发布。

```groovy title="build.gradle"
...
dependencies {
  def livekit_version = "2.25.3"

  implementation "io.livekit:livekit-android:$livekit_version"

  // CameraX 支持，含捏合缩放、闪光灯控制等
  implementation "io.livekit:livekit-android-camerax:$livekit_version"

  // 轨道处理器，如虚拟背景
  implementation "io.livekit:livekit-android-track-processors:$livekit_version"

  // 最新开发版本的快照:
  // implementation "io.livekit:livekit-android:2.25.4-SNAPSHOT"
}
```

基于 Compose 的应用请查看 [Android Components SDK](https://github.com/livekit/components-android) 获取 Compose 组件支持。

此外还需要 JitPack 作为仓库之一。在 `settings.gradle` 中：

```groovy title="settings.gradle"
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        //...
        maven { url 'https://jitpack.io' }

        // 如需 SNAPSHOT 版本访问
        // maven { url 'https://central.sonatype.com/repository/maven-snapshots/' }
    }
}
```

## 使用方法

### 权限

LiveKit 依赖 `RECORD_AUDIO` 和 `CAMERA` 权限来使用麦克风和摄像头。这些权限必须在运行时请求。参考[示例应用](https://github.com/livekit/client-sdk-android/blob/4e76e36e0d9f895c718bd41809ab5ff6c57aabd4/sample-app-compose/src/main/java/io/livekit/android/composesample/MainActivity.kt#L134)了解示例。

### 发布摄像头和麦克风

```kt
room.localParticipant.setCameraEnabled(true)
room.localParticipant.setMicrophoneEnabled(true)
```

### 屏幕共享

```kt
// 创建一个用于屏幕采集的 intent launcher
// 必须在 onCreate() 之前注册，最好作为实例属性
val screenCaptureIntentLauncher = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    val resultCode = result.resultCode
    val data = result.data
    if (resultCode != Activity.RESULT_OK || data == null) {
        return@registerForActivityResult
    }
    lifecycleScope.launch {
        room.localParticipant.setScreenShareEnabled(true, data)
    }
}

// 需要开启屏幕共享时，执行以下操作
val mediaProjectionManager =
    getSystemService(MEDIA_PROJECTION_SERVICE) as MediaProjectionManager
screenCaptureIntentLauncher.launch(mediaProjectionManager.createScreenCaptureIntent())
```

### 渲染订阅的轨道

LiveKit 使用 `SurfaceViewRenderer` 来渲染视频轨道。同时也通过 `TextureViewRenderer` 提供了 `TextureView` 实现。订阅的音频轨道会自动播放。

```kt
class MainActivity : AppCompatActivity() {

    lateinit var room: Room

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        // 创建 Room 对象。
        room = LiveKit.create(applicationContext)

        // 设置视频渲染器
        room.initVideoRenderer(findViewById<SurfaceViewRenderer>(R.id.renderer))

        connectToRoom()
    }

    private fun connectToRoom() {

        val url = "wss://your_host"
        val token = "your_token"

        lifecycleScope.launch {

            // 设置事件处理。
            launch {
                room.events.collect { event ->
                    when (event) {
                        is RoomEvent.TrackSubscribed -> onTrackSubscribed(event)
                        else -> {}
                    }
                }
            }

            // 连接到服务器。
            room.connect(
                url,
                token,
            )

            // 打开音频/视频录制。
            val localParticipant = room.localParticipant
            localParticipant.setMicrophoneEnabled(true)
            localParticipant.setCameraEnabled(true)
        }
    }

    private fun onTrackSubscribed(event: RoomEvent.TrackSubscribed) {
        val track = event.track
        if (track is VideoTrack) {
            attachVideo(track)
        }
    }

    private fun attachVideo(videoTrack: VideoTrack) {
        videoTrack.addRenderer(findViewById<SurfaceViewRenderer>(R.id.renderer))
        findViewById<View>(R.id.progress).visibility = View.GONE
    }
}
```

完整实现请参阅[基础示例应用](https://github.com/livekit/client-sdk-android/blob/main/sample-app-basic/src/main/java/io/livekit/android/sample/basic/MainActivity.kt)。

### 音频模式

默认情况下，音频配置为双向通话模式。

如果你正在构建直播或媒体播放为主的应用，可以在创建 `Room` 对象时使用 `MediaAudioType` 预设以获得更好的音频质量。

```kt
val room = LiveKit.create(
    appContext = application,
    overrides = LiveKitOverrides(
        audioOptions = AudioOptions(
            audioOutputType = AudioType.MediaAudioType()
        )
    )
)
```

注意：音频路由将由系统自动处理，无法手动控制。

如需更精细地控制音频属性和模式，可以传入 `CustomAudioType`。

### `@FlowObservable`

标注 `@FlowObservable` 的属性可以作为 Kotlin Flow 来观察变更：

```kt
coroutineScope.launch {
    room::activeSpeakers.flow.collectLatest { speakersList ->
        /*...*/
    }
}
```

## 示例应用

**注意**：如果想直接从本仓库运行示例应用，请参考[开发环境说明](#开发环境)。

我们有一个基础快速入门示例应用（[链接](https://github.com/livekit/client-sdk-android/blob/main/sample-app-basic)），展示了如何连接到房间、发布设备音频/视频以及显示一个远程参与者的视频。

还有两个功能更完善的视频会议示例应用：

- [Compose 应用](https://github.com/livekit/client-sdk-android/tree/main/sample-app-compose/src/main/java/io/livekit/android/composesample)
- [传统 View 应用](https://github.com/livekit/client-sdk-android/tree/main/sample-app/src/main/java/io/livekit/android/sample)

两者都使用 [`CallViewModel`](https://github.com/livekit/client-sdk-android/blob/main/sample-app-common/src/main/java/io/livekit/android/sample/CallViewModel.kt)，它负责管理 `Room` 连接并暴露基础视频会议应用所需的数据。

每个应用中对应的 `ParticipantItem` 类负责各个参与者 UI 的展示：

- [Compose `ParticipantItem`](https://github.com/livekit/client-sdk-android/blob/main/sample-app-compose/src/main/java/io/livekit/android/composesample/ParticipantItem.kt)
- [View `ParticipantItem`](https://github.com/livekit/client-sdk-android/blob/main/sample-app/src/main/java/io/livekit/android/sample/ParticipantItem.kt)

## 开发环境

要开发 Android SDK 或直接从本仓库运行示例应用，你需要：

- 克隆仓库到本地
- 确保 protocol 子模块仓库已初始化并更新

```
git clone https://github.com/livekit/client-sdk-android.git
cd client-sdk-android
git submodule update --init
```

---

如果你在 Apple Silicon Mac（如 M1、M2 等）上进行开发，请在 `$HOME/.gradle/gradle.properties` 中添加以下内容：

```
protoc_platform=osx-x86_64
```

### 可选（开发便利）

1. 从 https://webrtc.googlesource.com/src 下载 WebRTC 源码
2. 在 Android Studio 中通过指向 `webrtc/sdk/android` 文件夹添加源码。

<!--BEGIN_REPO_NAV-->
<br/><table>
<thead><tr><th colspan="2">LiveKit 生态系统</th></tr></thead>
<tbody>
<tr><td>Agents SDK</td><td><a href="https://github.com/livekit/agents">Python</a> · <a href="https://github.com/livekit/agents-js">Node.js</a></td></tr><tr></tr>
<tr><td>LiveKit SDK</td><td><a href="https://github.com/livekit/client-sdk-js">浏览器</a> · <a href="https://github.com/livekit/client-sdk-swift">Swift</a> · <b>Android</b> · <a href="https://github.com/livekit/client-sdk-flutter">Flutter</a> · <a href="https://github.com/livekit/client-sdk-react-native">React Native</a> · <a href="https://github.com/livekit/rust-sdks">Rust</a> · <a href="https://github.com/livekit/node-sdks">Node.js</a> · <a href="https://github.com/livekit/python-sdks">Python</a> · <a href="https://github.com/livekit/client-sdk-unity">Unity</a> · <a href="https://github.com/livekit/client-sdk-unity-web">Unity (WebGL)</a> · <a href="https://github.com/livekit/client-sdk-esp32">ESP32</a> · <a href="https://github.com/livekit/client-sdk-cpp">C++</a></td></tr><tr></tr>
<tr><td>入门应用</td><td><a href="https://github.com/livekit-examples/agent-starter-python">Python Agent</a> · <a href="https://github.com/livekit-examples/agent-starter-node">TypeScript Agent</a> · <a href="https://github.com/livekit-examples/agent-starter-react">React App</a> · <a href="https://github.com/livekit-examples/agent-starter-swift">SwiftUI App</a> · <a href="https://github.com/livekit-examples/agent-starter-android">Android App</a> · <a href="https://github.com/livekit-examples/agent-starter-flutter">Flutter App</a> · <a href="https://github.com/livekit-examples/agent-starter-react-native">React Native App</a> · <a href="https://github.com/livekit-examples/agent-starter-embed">Web Embed</a></td></tr><tr></tr>
<tr><td>UI 组件</td><td><a href="https://github.com/livekit/components-js">React</a> · <a href="https://github.com/livekit/components-android">Android Compose</a> · <a href="https://github.com/livekit/components-swift">SwiftUI</a> · <a href="https://github.com/livekit/components-flutter">Flutter</a></td></tr><tr></tr>
<tr><td>服务端 API</td><td><a href="https://github.com/livekit/node-sdks">Node.js</a> · <a href="https://github.com/livekit/server-sdk-go">Golang</a> · <a href="https://github.com/livekit/server-sdk-ruby">Ruby</a> · <a href="https://github.com/livekit/server-sdk-kotlin">Java/Kotlin</a> · <a href="https://github.com/livekit/python-sdks">Python</a> · <a href="https://github.com/livekit/rust-sdks">Rust</a> · <a href="https://github.com/agence104/livekit-server-sdk-php">PHP (社区)</a> · <a href="https://github.com/pabloFuente/livekit-server-sdk-dotnet">.NET (社区)</a></td></tr><tr></tr>
<tr><td>资源</td><td><a href="https://docs.livekit.io">文档</a> · <a href="https://docs.livekit.io/mcp">Docs MCP Server</a> · <a href="https://github.com/livekit/livekit-cli">CLI</a> · <a href="https://cloud.livekit.io">LiveKit Cloud</a></td></tr><tr></tr>
<tr><td>LiveKit Server 开源</td><td><a href="https://github.com/livekit/livekit">LiveKit server</a> · <a href="https://github.com/livekit/egress">Egress</a> · <a href="https://github.com/livekit/ingress">Ingress</a> · <a href="https://github.com/livekit/sip">SIP</a></td></tr><tr></tr>
<tr><td>社区</td><td><a href="https://community.livekit.io">开发者社区</a> · <a href="https://livekit.io/join-slack">Slack</a> · <a href="https://x.com/livekit">X</a> · <a href="https://www.youtube.com/@livekit_io">YouTube</a></td></tr>
</tbody>
</table>
<!--END_REPO_NAV-->
