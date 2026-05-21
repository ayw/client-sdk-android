# livekit-android-camerax 模块分析

## 1. 模块定位

`livekit-android-camerax` 是 LiveKit Android SDK 的可选扩展模块，用 AndroidX **CameraX** 替代原生的 Camera2/Camera1 API 来实现视频采集。它作为上层插件注入到核心 SDK 的相机提供者体系中，对核心 SDK 零侵入。

## 2. 模块结构

只有 5 个核心类 + 1 个 UI 工具类，非常精简：

```
livekit-android-camerax/
└── src/main/java/
    ├── livekit/org/webrtc/          # 放在 WebRTC 包下，与 Camera1/Camera2 同级
    │   ├── CameraXProvider.kt       # CameraProvider 实现，注册入口
    │   ├── CameraXEnumerator.kt     # 设备枚举器
    │   ├── CameraXCapturer.kt       # 采集器
    │   ├── CameraXSession.kt        # 会话（最核心，~380 行）
    │   └── CameraXHelper.kt         # 对外工厂方法
    └── io/livekit/android/camerax/ui/
        └── ScaleZoomHelper.kt       # 捏合缩放工具（Compose + View 双支持）
```

## 3. 架构设计

### 3.1 整体架构图

```
┌──────────────────────────────────────────────────────────────────┐
│                         应用层 (App)                              │
│  CameraXHelper.createCameraProvider(lifecycleOwner, useCases)    │
│  CameraCapturerUtils.registerCameraProvider(provider)            │
│  ScaleZoomHelper(videoTrack)  ←  UI 缩放控制                     │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                    核心 SDK (livekit-android-sdk)                  │
│                                                                   │
│  CameraCapturerUtils                                              │
│  ├── cameraProviders: List<CameraProvider>  (优先级排序)          │
│  │   ├── Camera1Provider  (version=1)                            │
│  │   ├── Camera2Provider  (version=2)                            │
│  │   └── CameraXProvider  (version=3)  ← 最高优先级，自动选用    │
│  ├── registerCameraProvider() / unregisterCameraProvider()       │
│  └── getCameraProvider() → 按 version 降序，第一个 isSupported()  │
│                                                                   │
│  CameraProvider 接口：                                            │
│  ├── cameraVersion: Int              优先级                      │
│  ├── isSupported(context): Boolean   是否可用                     │
│  ├── provideEnumerator(context): CameraEnumerator                │
│  └── provideCapturer(context, options, handler): VideoCapturer   │
└──────────────────────────────┬───────────────────────────────────┘
                               │ 实现
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                  livekit-android-camerax                          │
│                                                                   │
│  CameraXProvider ──► CameraXEnumerator ──► CameraXCapturer       │
│       │                     │                    │                │
│       │                     │                    ▼                │
│       │                     │            CameraXSession           │
│       │                     │         (核心，管理整个采集生命周期) │
│       │                     │                                    │
│       │             继承 Camera2Enumerator                       │
│       │             (复用 Camera2 的设备枚举能力)                 │
└──────────────────────────────────────────────────────────────────┘
```

### 3.2 设计原理：策略模式 + 版本优先级

核心 SDK 的 `CameraCapturerUtils` 维护一个 `cameraProviders` 列表，每个 Provider 声明一个 `cameraVersion`。选择 Provider 时按 version **降序**排列，取第一个 `isSupported()` 返回 true 的：

```
优先级：CameraX (v3) > Camera2 (v2) > Camera1 (v1)
```

这意味着：**只要设备支持 CameraX，就自动选用 CameraX 采集**，开发者无需关心底层实现。

### 3.3 为什么放在 `livekit.org.webrtc` 包下？

CameraXCapturer 需要继承核心 SDK 中 `livekit.org.webrtc` 包下的 `CameraCapturer`（这是 WebRTC 内部类），且该构造函数是包级可见的。放在同一包下避免了反射或 public API 暴露。

## 4. 核心流程

### 4.1 相机启动时序图

```
App                  Core SDK              CameraXProvider    CameraXEnumerator   CameraXCapturer    CameraXSession    CameraX
 │                       │                       │                   │                  │                  │              │
 │  setCameraEnabled(true)                      │                   │                  │                  │              │
 │──────────────────────►│                       │                   │                  │                  │              │
 │                       │                       │                   │                  │                  │              │
 │                       │  getCameraProvider()  │                   │                  │                  │              │
 │                       │──────────────────────►│                   │                  │                  │              │
 │                       │                       │                   │                  │                  │              │
 │                       │  provideEnumerator()  │                   │                  │                  │              │
 │                       │──────────────────────►│──────────────────►│                  │                  │              │
 │                       │                       │                   │                  │                  │              │
 │                       │  provideCapturer()    │                   │                  │                  │              │
 │                       │──────────────────────►│  createCapturer() │                  │                  │              │
 │                       │                       │──────────────────►│                  │                  │              │
 │                       │                       │                   │  CameraXCapturer()                 │              │
 │                       │                       │                   │──────────────────►│                  │              │
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │  createCameraSession()        │
 │                       │                       │                   │                   │─────────────────►│              │
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │                  │  start()     │
 │                       │                       │                   │                   │                  │─────────────►│
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │                  │ openCamera() │
 │                       │                       │                   │                   │                  │─────────────►│
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │                  │  ┌───────────┐
 │                       │                       │                   │                   │                  │  │ 获取      │
 │                       │                       │                   │                   │                  │  │ Camera    │
 │                       │                       │                   │                   │                  │  │ Provider  │
 │                       │                       │                   │                   │                  │  └───────────┘
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │                  │  匹配       │
 │                       │                       │                   │                   │                  │  Capture    │
 │                       │                       │                   │                   │                  │  Format     │
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │                  │  Preview    │
 │                       │                       │                   │                   │                  │  + UseCases │
 │                       │                       │                   │                   │                  │  bindTo     │
 │                       │                       │                   │                   │                  │  Lifecycle  │
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │  onDone(session) │           │
 │                       │                       │                   │                   │◄─────────────────│              │
 │                       │                       │                   │                   │                  │              │
 │                       │                       │                   │                   │  currentCamera  │              │
 │                       │                       │                   │                   │  = camera       │              │
```

### 4.2 帧采集流程

```
CameraX 硬件
    │
    │ (原始帧)
    ▼
Preview UseCase
    │
    │ SurfaceProvider.provideSurface()
    ▼
SurfaceTextureHelper
    │
    │ surfaceTextureListener { frame -> ... }
    ▼
帧变换处理（在 CameraThread 上）:
    1. 去除系统自动添加的镜像效果 (isCameraFrontFacing)
    2. 撤销相机旋转，转为帧旋转角度 (cameraOrientation)
    3. 构建新的 VideoFrame (带修正后的变换矩阵)
    │
    ▼
events.onFrameCaptured(session, modifiedFrame)
    │
    ▼
核心 SDK → WebRTC 编码器 → 网络发送
```

### 4.3 相机关闭流程

```
stop()
    │
    ├── state = STOPPED
    ├── surfaceTextureHelper.stopListening()
    ├── surface.release()
    ├── cameraProvider.unbindAll()    ← 主线程执行
    └── events.onCameraClosed(this)   ← 回到 CameraThread
```

## 5. 设计要点深入分析

### 5.1 使用 Preview 而非 VideoCapture

CameraX 提供了 `Preview` 和 `VideoCapture` 两种 UseCase。本模块选择 `Preview` 而非 `VideoCapture` 的原因：

- `VideoCapture` 输出的是编码后的帧，LiveKit SDK 需要的是**原始帧**（由 WebRTC 自行编码）
- `Preview` 通过 `SurfaceProvider` 提供 `Surface`，`SurfaceTextureHelper` 从 Surface 中直接获取纹理帧作为 `VideoFrame`
- 这保持了与 Camera1/Camera2 一致的帧处理管线（都通过 SurfaceTexture → VideoFrame → WebRTC 编码器）

### 5.2 Lifecycle 绑定

CameraX 的核心机制是将 UseCase 绑定到 `LifecycleOwner`：

```kotlin
camera = cameraProvider.bindToLifecycle(lifecycleOwner, cameraSelector, preview, *useCases)
```

- 当 `lifecycleOwner` 进入 `ON_RESUME` 时自动打开相机
- 当 `lifecycleOwner` 进入 `ON_PAUSE` 时自动关闭相机
- 应用不需要手动管理相机开关，减少了生命周期 bug

在示例应用中，使用 `ProcessLifecycleOwner.get()` 将相机生命周期绑定到整个应用进程。

### 5.3 额外 UseCase 支持

构造函数接收 `useCases: Array<out UseCase>`，允许调用方附加 ImageAnalysis 等 UseCase：

```kotlin
CameraXHelper.createCameraProvider(
    lifecycleOwner,
    useCases = arrayOf(imageAnalysisUseCase),  // 可附加自定义 UseCase
)
```

这些 UseCase 会和 Preview 一起绑定到同一个 Camera，共享同一路视频流。

### 5.4 帧变换修正

由于 CameraX 和 Android 系统会自动对前置摄像头做镜像处理，模块在帧回调中手动还原：

```kotlin
val modifiedFrame = VideoFrame(
    CameraSession.createTextureBufferWithModifiedTransformMatrix(
        frame.buffer as TextureBufferImpl,
        isCameraFrontFacing,   // 是否前置（需要取消镜像）
        -cameraOrientation,     // 撤销系统旋转
    ),
    getFrameOrientation(),
    frame.timestampNs,
)
```

### 5.5 线程模型

严格遵循核心 SDK 的 RTC 线程约束：

```
│  主线程 (Main)                 │  CameraThread (RTC 线程)        │
│                                │                                 │
│  bindToLifecycle()             │  start()                        │
│  unbindAll()                   │  openCamera()                   │
│                                │  surfaceTextureListener {}      │
│                                │  stopInternal()                 │
│                                │  checkIsOnCameraThread()        │
```

- 所有 CameraX API 调用通过 `helperExecutor` 转发到 CameraThread
- `cameraProvider.bindToLifecycle()` 和 `unbindAll()` 在主线程执行（CameraX 要求）
- 帧回调、start/stop 在 CameraThread 执行

### 5.6 物理摄像头支持

在 Android P+ 上，逻辑摄像头（如双摄）包含多个物理摄像头。模块在 `CameraXEnumerator.getDeviceNames()` 中枚举物理摄像头 ID，并支持通过 `setPhysicalCameraId()` 指定具体物理摄像头。

### 5.7 防抖稳定模式选择

优先级：**光学防抖 (OIS) > 电子防抖 (EIS) > 无防抖**

```kotlin
when {
    availableOpticalStabilization → StabilizationMode.OPTICAL   // 最佳
    availableVideoStabilization  → StabilizationMode.VIDEO     // 次选
    else                         → StabilizationMode.NONE
}
```

## 6. ScaleZoomHelper 设计

捏合缩放工具类，同时支持 View 体系和 Compose 体系：

**View 体系**（通过 `createGestureDetector`）：
```kotlin
val detector = ScaleZoomHelper.createGestureDetector(context, localVideoTrack)
view.setOnTouchListener { _, event -> detector.onTouchEvent(event); true }
```

**Compose 体系**（通过 `Modifier.pointerInput`）：
```kotlin
Modifier.pointerInput(scaleZoomHelper) {
    detectTransformGestures { _, _, zoom, _ -> scaleZoomHelper?.zoom(zoom) }
}
```

关键设计细节：

- **防止过时状态**：`targetZoom` 字段主动跟踪目标缩放值，因为 `CameraInfo.zoomState` 是基于 LiveData 定时更新的，存在延迟
- **Future 跟踪**：维护 `activeZoomFutures` 集合，只有当所有缩放操作完成时才清除 `targetZoom`，防止中间状态的 LiveData 更新覆盖正在进行的缩放
- **线程安全**：`@Synchronized` 注解保护缩放操作的原子性

## 7. 优缺点分析

### 优点

| 优点 | 说明 |
|---|---|
| **零侵入集成** | 通过 Provider 接口注入，核心 SDK 无需感知 CameraX 存在 |
| **生命周期安全** | 基于 LifecycleOwner 自动管理相机开关，减少 bug |
| **向后兼容** | 通过 isSupported() 降级，不支持的设备自动回退到 Camera2/Camera1 |
| **代码精简** | 仅 5 个核心类，~500 行有效代码，架构清晰 |
| **复用 Camera2 能力** | 枚举器继承 Camera2Enumerator，设备枚举、格式匹配等逻辑复用 |
| **双 UI 范式支持** | ScaleZoomHelper 同时提供 View 和 Compose API |
| **可扩展** | 支持注入额外 UseCase（如 ImageAnalysis）|

### 缺点 / 风险

| 缺点 | 说明 |
|---|---|
| **依赖 CameraX 库** | 引入 `camera-core`、`camera-camera2`、`camera-lifecycle` 三个库，增加 APK 体积 |
| **Preview 而非原生采集** | 用 Preview UseCase 模拟视频采集，不是 CameraX 的标准视频采集路径（VideoCapture），依赖内部实现细节 |
| **帧变换复杂度** | 需要在 CameraThread 上手动处理镜像反转和旋转补偿，容易出错 |
| **CameraX 内部 API 依赖** | 使用 `@ExperimentalCamera2Interop` API（Camera2Interop.Extender），可能在 CameraX 版本升级时变化 |
| **LiveData 延迟问题** | `CameraInfo.zoomState` 基于 LiveData 更新有延迟，需要 `targetZoom` 字段额外补偿（如 ScaleZoomHelper 中所做）|
| **测试覆盖不足** | 模块中的 robolectric 和 espresso 依赖已声明，但相机相关测试难以在纯单元测试中覆盖 |

## 8. 使用方式

```kotlin
// 1. 创建 CameraX Provider（通常在 ViewModel init 中）
val cameraProvider = CameraXHelper.createCameraProvider(
    lifecycleOwner = ProcessLifecycleOwner.get(),
    useCases = arrayOf(),  // 可附加 ImageAnalysis 等
)

// 2. 注册到核心 SDK
if (cameraProvider.isSupported(context)) {
    CameraCapturerUtils.registerCameraProvider(cameraProvider)
}

// 3. 正常使用 Room API（SDK 会自动使用 CameraX）
room.localParticipant.setCameraEnabled(true)

// 4. （可选）获取 CameraX Camera 实例（用于缩放、闪光灯等高级控制）
val cameraFlow: StateFlow<Camera?>? = localVideoTrack.capturer.getCameraX()

// 5. （可选）启用捏合缩放
val scaleZoomHelper = ScaleZoomHelper(localVideoTrack)

// 6. 销毁时注销
CameraCapturerUtils.unregisterCameraProvider(cameraProvider)
```

## 9. 与 Camera1/Camera2 Provider 的对比

| 维度 | Camera1 | Camera2 | CameraX |
|---|---|---|---|
| cameraVersion | 1 | 2 | 3 |
| 摄像头 API | android.hardware.Camera | android.hardware.camera2 | CameraX (底层封装 Camera2) |
| 生命周期管理 | 手动 | 手动 | **自动** (LifecycleOwner 绑定) |
| 物理摄像头支持 | 无 | 有 (P+) | **有** (P+)，通过 Camera2Interop |
| 防抖模式 | 无 | 手动配置 | **自动检测** 并应用 |
| 额外 UseCase | 不支持 | 不支持 | **支持** (ImageAnalysis 等) |
| API 级别要求 | API 21+ | API 21+ | API 21+ |

## 10. CameraXSession 详细分析

`CameraXSession` 是整个模块最核心的类（~380 行），负责完整的相机采集生命周期。它继承自核心 SDK 的 `CameraSession`，将 CameraX 的 Preview API 适配为 WebRTC 兼容的 `VideoFrame` 输出。

### 10.1 类结构总览

```
CameraXSession (extends CameraSession)
│
├── 构造函数参数
│   ├── sessionCallback    — 通知父层会话创建成功/失败
│   ├── events             — 通知父层帧到达、相机事件
│   ├── context/lifecycleOwner — Android 上下文与生命周期
│   ├── surfaceTextureHelper — 共享纹理 + 线程句柄
│   ├── cameraId/width/height/frameRate — 采集规格
│   └── useCases           — 可选的附加 UseCase（ImageAnalysis 等）
│
├── 内部状态
│   ├── SessionState { RUNNING, STOPPED }
│   ├── StabilizationMode { OPTICAL, VIDEO, NONE }
│   └── CameraDeviceId(deviceId, physicalId?)
│
├── 核心成员
│   ├── cameraProvider: ProcessCameraProvider
│   ├── camera: Camera?                       ← 对外暴露（用于缩放等）
│   ├── surfaceProvider: SurfaceProvider      ← Preview 的 Surface 供给
│   ├── captureFormat: CaptureFormat?
│   ├── surfaceTextureListener                ← 帧回调核心
│   └── stabilizationMode: StabilizationMode
│
└── 关键方法
    ├── start() → stop()
    ├── openCamera()
    ├── obtainCameraConfiguration()
    ├── findCaptureFormat() / findStabilizationMode()
    ├── stopInternal()
    ├── reportError()
    └── applyCameraSettings()     ← Camera2Interop 扩展配置
```

### 10.2 完整状态流转图

```
                   ┌──────────┐
                   │ 构造     │
                   └────┬─────┘
                        │ init { cameraThreadHandler.post { start() } }
                        ▼
                   ┌──────────┐
                   │ start()  │
                   └────┬─────┘
                        │ surfaceTextureHelper.startListening(listener)
                        │ → 开始监听帧数据
                        │ openCamera()
                        ▼
              ┌────────────────────┐
              │ openCamera()       │
              │                    │
              │ ProcessCamera      │
              │ Provider           │
              │ .getInstance()     │
              │ + addListener()    │
              └────────┬───────────┘
                       │
              ┌────────▼───────────┐
              │ obtainCamera       │  ← 匹配 Camera2CameraInfo
              │ Configuration()    │    获取方向、焦距、
              │                    │    计算最佳 CaptureFormat
              │ • 获取相机朝向     │    选择防抖模式
              │ • 判断前置/后置   │
              │ • 匹配分辨率+FPS  │
              │ • 选择防抖模式     │
              └────────┬───────────┘
                       │
              ┌────────▼───────────┐
              │ Preview.Builder    │
              │ .setResolution     │  ← 用 ResolutionSelector
              │   Selector()       │    指定最佳分辨率策略
              │ .applyCamera       │    FALLBACK_CLOSEST_HIGHER_
              │   Settings()       │    THEN_LOWER
              │ .build()           │
              │                    │
              │ + Camera2Interop   │  ← 设置物理摄像头ID、
              │   .Extender:       │    AE_TARGET_FPS_RANGE、
              │   • 物理摄像头ID   │    AE_MODE=ON、
              │   • FPS 范围       │    防抖模式
              │   • AE 参数        │
              │   • 防抖参数       │
              └────────┬───────────┘
                       │
              ┌────────▼───────────┐
              │ 主线程:            │
              │ cameraProvider     │
              │ .unbindAll()       │  ← 解绑所有旧 UseCase
              │ cameraProvider     │
              │ .bindToLifecycle(  │  ← 绑定 Preview + 附加 UseCase
              │   lifecycleOwner,  │    到 lifecycleOwner
              │   cameraSelector,  │
              │   preview,         │    绑定成功 → camera 实例
              │   *useCases        │    绑定失败 → 抛异常 → reportError
              │ )                  │
              └────────┬───────────┘
                       │ 成功
                       ▼
            cameraThreadHandler.post {
                sessionCallback.onDone(this)
                // CameraXCapturer 收到回调后设置 currentCamera
            }
                       │
                       ▼
           ┌───────────────────────┐
           │  RUNNING 状态         │
           │                       │
           │  surfaceTextureListener 被持续回调:
           │  ┌─────────────────┐  │
           │  │ 每一帧到达:     │  │
           │  │ 1. 检查 RUNNING │  │
           │  │ 2. 首帧记录启动 │  │
           │  │    耗时 (直方图)│  │
           │  │ 3. 去除镜像     │  │
           │  │ 4. 校正旋转     │  │
           │  │ 5. 构建新帧     │  │
           │  │ 6. events.      │  │
           │  │    onFrame      │  │
           │  │    Captured()   │  │
           │  └─────────────────┘  │
           └───────────┬───────────┘
                       │
                  stop() 被调用  或  cameraProvider 异常
                       │
              ┌────────▼───────────┐
              │ stop()             │
              │ state = STOPPED    │
              │ stopInternal()     │
              └────────┬───────────┘
                       │
              ┌────────▼───────────┐
              │ stopInternal()     │
              │ 1. stopListening() │ ← 停止帧监听
              │ 2. surface.release │ ← 释放 Surface
              │ 3. 主线程:         │
              │    unbindAll()     │ ← 解绑所有 UseCase
              │ 4. CameraThread:   │
              │    events.         │
              │    onCameraClosed  │ ← 通知已关闭
              └────────────────────┘
```

### 10.3 构造函数：参数注入与线程启动

```kotlin
internal constructor(
    sessionCallback, events,      // 父层回调
    context, lifecycleOwner,      // Android 环境
    surfaceTextureHelper,         // 共享纹理（提供 handler、SurfaceTexture）
    cameraId,                     // 逻辑或物理摄像头 ID
    width, height, frameRate,     // 目标格式
    useCases = emptyArray(),      // 附加 UseCase
)
```

关键点：

- **`surfaceTextureHelper` 的双重用途**：既提供输出 Surface 的 `SurfaceTexture`，又提供 `Handler`（`cameraThreadHandler`）。所有相机操作必须在这个 Handler 对应的线程上执行。
- **构造函数直接 post `start()`**：`init { cameraThreadHandler.post { start() } }` — 这是异步启动，构造函数不阻塞。

### 10.4 帧采集核心：`surfaceTextureListener`

这是整个模块最关键的代码段，每一帧都经过这里：

```kotlin
private val surfaceTextureListener = { frame: VideoFrame ->
    checkIsOnCameraThread()

    if (state != SessionState.RUNNING) {
        Logging.d(TAG, "Texture frame captured but camera is no longer running.")
        return  // ← 已停止，丢弃帧
    }

    if (!firstFrameReported) {
        firstFrameReported = true
        val startTimeMs = TimeUnit.NANOSECONDS.toMillis(
            System.nanoTime() - constructionTimeNs  // ← 统计启动耗时
        ).toInt()
        cameraXStartTimeMsHistogram.addSample(startTimeMs)
    }

    // 核心：变换矩阵修正
    //   1. 撤销系统对前置摄像头的自动镜像
    //   2. 撤销系统添加的 display orientation（改为帧 rotation 表达）
    val modifiedFrame = VideoFrame(
        CameraSession.createTextureBufferWithModifiedTransformMatrix(
            frame.buffer as TextureBufferImpl,
            isCameraFrontFacing,      // true → 水平翻转取消镜像
            -cameraOrientation,        // 反向旋转
        ),
        getFrameOrientation(),        // 当前设备物理方向
        frame.timestampNs,
    )

    events.onFrameCaptured(this, modifiedFrame)
    modifiedFrame.release()  // ← 必须释放，防止纹理泄漏
}
```

**帧变换详解**：

```
原始帧（来自 CameraX Preview）
  │  Android 系统自动做了两件事:
  │  1. 前置摄像头 → 水平镜像 (isFrontFacing=true)
  │  2. 根据 display orientation 旋转到显示方向
  │
  ▼
createTextureBufferWithModifiedTransformMatrix()
  │
  │  参数: isCameraFrontFacing, -cameraOrientation
  │
  │  操作:
  │  • isCameraFrontFacing=true → 水平翻转矩阵 (撤销镜像)
  │  • -cameraOrientation → 反向旋转矩阵 (撤销系统旋转)
  │
  │  WebRTC 编码器期望的是:
  │  - 未镜像的原始帧
  │  - 帧本身不旋转，通过 getFrameOrientation() 单独告知旋转角度
  │
  ▼
修正后的帧 (符合 WebRTC 预期)
  │
  ▼
events.onFrameCaptured()  → 核心 SDK → WebRTC 编码 → 网络
```

### 10.5 启动流程：`openCamera()` 详解

```kotlin
private fun openCamera() {
    checkIsOnCameraThread()
    events.onCameraOpening()  // ← 通知上层"相机正在打开"

    // ── 步骤 1: 异步获取 ProcessCameraProvider ──
    val cameraProviderFuture = ProcessCameraProvider.getInstance(context)

    // ── 步骤 2: 构造 Executor 将回调路由到 CameraThread ──
    val helperExecutor = Executor { command ->
        surfaceTextureHelper.handler.let {
            if (it.looper.thread.isAlive) {
                it.post(command)      // ← 转发到 CameraThread
            }
        }
    }

    cameraProviderFuture.addListener({
        cameraProvider = cameraProviderFuture.get()

        // ── 步骤 3: 获取相机参数 (方向、焦距) + 匹配最佳格式 ──
        obtainCameraConfiguration()

        // ── 步骤 4: 设置输出 Surface 尺寸 ──
        surfaceTextureHelper.setTextureSize(
            captureFormat?.width  ?: width,
            captureFormat?.height ?: height
        )

        // ── 步骤 5: 创建输出 Surface ──
        surface = Surface(surfaceTextureHelper.surfaceTexture)

        // ── 步骤 6: 创建 SurfaceProvider ──
        surfaceProvider = SurfaceProvider { request ->
            surface?.let {
                request.provideSurface(it, helperExecutor) { }
            } ?: request.willNotProvideSurface()  // ← surface 为空时拒绝
        }

        // ── 步骤 7: 按摄像头 ID 精确选择 ──
        val cameraSelector = CameraSelector.Builder()
            .addCameraFilter { cameraInfo ->
                cameraInfo.filter {
                    val id = Camera2CameraInfo.from(it).cameraId
                    id == cameraDevice.deviceId || id == cameraDevice.physicalId
                }
            }
            .build()

        // ── 步骤 8: 主线程构建 Preview + 绑定 ──
        ContextCompat.getMainExecutor(context).execute {
            val preview = Preview.Builder()
                .setResolutionSelector(
                    ResolutionSelector.Builder()
                        .setResolutionStrategy(
                            ResolutionStrategy(
                                Size(captureFormat?.width ?: width,
                                     captureFormat?.height ?: height),
                                FALLBACK_RULE_CLOSEST_HIGHER_THEN_LOWER
                            )
                        )
                        .build()
                )
                .applyCameraSettings()   // ← Camera2Interop 扩展 (FPS、防抖、物理摄像头)
                .build()
                .also { it.setSurfaceProvider(surfaceProvider) }

            cameraProvider.unbindAll()
            camera = cameraProvider.bindToLifecycle(
                lifecycleOwner, cameraSelector, preview, *useCases
            )

            cameraThreadHandler.post {
                sessionCallback.onDone(this)
            }
        }
    }, helperExecutor)
}
```

**执行流程图**：

```
CameraThread                          helperExecutor                    MainThread
    │                                       │                              │
    │──── ProcessCameraProvider ────►        │                              │
    │          .getInstance()               │                              │
    │                                       │                              │
    │           addListener(callback, helperExecutor)                      │
    │                                       │                              │
    │                                       │◄── callback 就绪 ────────────│
    │                                       │                              │
    │                                       │── obtainCameraConfiguration()│
    │                                       │── setTextureSize()           │
    │                                       │── SurfaceProvider {}         │
    │                                       │── CameraSelector {}          │
    │                                       │                              │
    │                                       │── getMainExecutor() ────────►│
    │                                       │                              │
    │                                       │                ┌─────────────┤
    │                                       │                │ Preview     │
    │                                       │                │ .Builder()  │
    │                                       │                │ .apply      │
    │                                       │                │  Settings() │
    │                                       │                │ .build()    │
    │                                       │                │             │
    │                                       │                │ unbindAll() │
    │                                       │                │ bindTo      │
    │                                       │                │ Lifecycle() │
    │                                       │                └─────────────┤
    │                                       │                              │
    │◄── sessionCallback.onDone() ──────────│                              │
    │                                       │                              │
```

### 10.6 流控策略：`ResolutionStrategy`

使用 `ResolutionStrategy.FALLBACK_RULE_CLOSEST_HIGHER_THEN_LOWER`：

```
请求分辨率: 1280x720
    │
    ▼
相机支持的输出分辨率列表: [640x480, 1280x720, 1920x1080, 2560x1440, ...]
    │
    │ 策略:
    │  1. 精确匹配 → 1280x720 ✓ → 使用
    │  2. 没有精确 → 取比它大的最接近值 (CLOSEST_HIGHER)
    │  3. 没有更大的 → 取比它小的最接近值 (THEN_LOWER)
    │
    ▼
selectedSize = 1280x720  (或最接近的值)
```

这比 Camera1/Camera2 的纯 `getClosestSupportedSize()` 更加灵活，优先选择不低于目标的分辨率。

### 10.7 线程模型深入

三类线程分工严格：

| 线程 | 角色 | 典型操作 |
|---|---|---|
| **CameraThread** | 相机操作线程 | start(), stop(), openCamera() 的配置部分, 帧回调, 首帧统计 |
| **MainThread** | UI 线程 | `bindToLifecycle()`, `unbindAll()`（均为 CameraX 强制要求）|
| **helperExecutor** | 桥接线程 | `ProcessCameraProvider.addListener()` 的回调在此执行，再通过 `handler.post()` 转发到 CameraThread |

**线程安全检查**：

```kotlin
private fun checkIsOnCameraThread() {
    check(Thread.currentThread() === cameraThreadHandler.looper.thread) {
        "Wrong thread"
    }
}
```

在所有关键方法入口调用 `checkIsOnCameraThread()`，确保操作线程正确。线程错误的后果是 WebRTC 内部状态可能被并发破坏。

### 10.8 `CameraDeviceId`：逻辑/物理摄像头双 ID 模型

```kotlin
private data class CameraDeviceId(
    val deviceId: String,        // 逻辑摄像头 ID
    val physicalId: String?      // 物理摄像头 ID (API 28+)
)
```

在 Android P+ 多摄设备上：
- `deviceId` = 逻辑摄像头组 ID（如 `"0"` 表示后置双摄组）
- `physicalId` = 具体物理摄像头 ID（如 `"2"` 表示广角镜头）

`findCamera()` 方法通过 `cameraManager.cameraIdList` 遍历，匹配逻辑 ID 或物理 ID：

```kotlin
// 先检查逻辑 ID 直接匹配
if (id == deviceId) return CameraDeviceId(id, null)

// 再检查作为物理 ID 是否存在
for (physicalId in characteristic.physicalCameraIds) {
    if (deviceId == physicalId) {
        return CameraDeviceId(id, physicalId)  // 找到！父逻辑 ID + 物理 ID
    }
}
```

找到物理 ID 后，在 `applyCameraSettings()` 中通过 `Camera2Interop.Extender.setPhysicalCameraId()` 设置，CameraX 会将请求路由到特定物理摄像头。

### 10.9 `applyCameraSettings()`：低层 Camera2 参数注入

此方法利用 `Camera2Interop.Extender` 绕过 CameraX 的抽象层，直接设置 Camera2 原生参数：

```kotlin
private fun <T> ExtendableBuilder<T>.applyCameraSettings(): ExtendableBuilder<T> {
    val cameraExtender = Camera2Interop.Extender(this)

    // ① 指定物理摄像头 (API 28+)
    cameraDevice.physicalId?.let { physicalId ->
        cameraExtender.setPhysicalCameraId(physicalId)
    }

    // ② 设置自动曝光目标 FPS 范围
    //    fpsUnitFactor: Camera1 时代遗留的兼容系数
    //    min 和 max 除以 fpsUnitFactor 因为 CameraX 内部会再乘回去
    cameraExtender.setCaptureRequestOption(
        CaptureRequest.CONTROL_AE_TARGET_FPS_RANGE,
        Range(
            captureFormat.framerate.min / fpsUnitFactor,
            captureFormat.framerate.max / fpsUnitFactor
        )
    )
    cameraExtender.setCaptureRequestOption(
        CaptureRequest.CONTROL_AE_MODE,
        CaptureRequest.CONTROL_AE_MODE_ON
    )
    cameraExtender.setCaptureRequestOption(
        CaptureRequest.CONTROL_AE_LOCK,
        false
    )

    // ③ 防抖模式互斥配置
    when (stabilizationMode) {
        OPTICAL → {
            // 打开光学防抖，关闭电子防抖
            LENS_OPTICAL_STABILIZATION_MODE → ON
            CONTROL_VIDEO_STABILIZATION_MODE → OFF
        }
        VIDEO → {
            // 打开电子防抖，关闭光学防抖
            CONTROL_VIDEO_STABILIZATION_MODE → ON
            LENS_OPTICAL_STABILIZATION_MODE → OFF
        }
        NONE → {}  // 不做设置
    }
}
```

注意防抖模式的互斥性：光学和电子防抖不能同时开启，这在 Camera2 规范中有明确规定。

### 10.10 `fpsUnitFactor`：Camera1 遗留兼容值

```kotlin
fpsUnitFactor = Camera2Enumerator.getFpsUnitFactor(fpsRanges)
```

某些旧设备的 FPS 范围值需要除以 1000 还原为真实值（Camera1 时代使用 `×1000` 定点数表示）。这个因子的计算在 `Camera2Enumerator` 中：

- 正常设备：`fpsUnitFactor = 1`
- Legacy 设备：`fpsUnitFactor = 1000`

CameraX 内部会自动 ×1000 处理，所以需要预先除以 `fpsUnitFactor` 来抵消。

### 10.11 指标埋点

```kotlin
companion object {
    // 启动耗时直方图 (ms): [1ms, 10000ms], 50 buckets
    private val cameraXStartTimeMsHistogram =
        Histogram.createCounts("WebRTC.Android.CameraX.StartTimeMs", 1, 10000, 50)

    // 停止耗时直方图 (ms)
    private val cameraXStopTimeMsHistogram =
        Histogram.createCounts("WebRTC.Android.CameraX.StopTimeMs", 1, 10000, 50)

    // 分辨率枚举 (与 Camera2 共享 COMMON_RESOLUTIONS)
    private val cameraXResolutionHistogram =
        Histogram.createEnumeration("WebRTC.Android.CameraX.Resolution", ...)
}
```

首帧时间从 `constructionTimeNs`（构造函数调用时）开始计时，覆盖完整启动路径。这些数据用于 WebRTC 内部性能监控。

### 10.12 错误处理

```kotlin
private fun reportError(error: String) {
    checkIsOnCameraThread()
    Logging.e(TAG, "Error: $error")

    val startFailure = camera == null && state != SessionState.STOPPED

    state = SessionState.STOPPED
    stopInternal()  // ← 清理资源

    if (startFailure) {
        // 启动阶段失败 → 通知父层创建失败
        sessionCallback.onFailure(CameraSession.FailureType.ERROR, error)
    } else {
        // 运行阶段故障 → 通知事件层出错
        events.onCameraError(this, error)
    }
}
```

区分两种错误场景：
- **启动失败**（`camera == null`）：此时 session 从未成功创建，需要通知 `CreateSessionCallback.onFailure()`
- **运行时出错**（`camera != null`）：session 曾经正常工作，通知 `Events.onCameraError()`

两种情况下都会执行 `stopInternal()` 清理资源（停止帧监听、释放 Surface、解绑 UseCase）。

### 10.13 设计亮点总结

| 设计点 | 说明 |
|---|---|
| **Camera2Interop 桥接** | 通过 `Camera2Interop.Extender` 在 CameraX 抽象层上精准控制底层的 FPS、防抖、物理摄像头，兼顾易用性和灵活性 |
| **精确摄像头选择** | 用 `addCameraFilter` 按 deviceId/物理ID 精确选择，而非使用 CameraX 默认的 LensFacing 选择逻辑，确保与核心 SDK 的摄像头选择一致 |
| **帧变换修正** | 正确撤销 Android 系统对前置摄像头镜像和旋转的自动处理，输出 WebRTC 期望的裸帧格式 |
| **线程精确控制** | 三类线程分工明确，`checkIsOnCameraThread()` 硬断言保证线程安全 |
| **异步启动不阻塞** | 构造函数立即返回，`post { start() }` 异步启动，不阻塞调用方 |
| **内部指标** | 首帧时间、停止时间、分辨率分布，可观测采集性能 |
| **启动/运行时错误分离** | 根据 `camera` 是否为 null 判断错误阶段，走不同的回调路径 |

## 11. 总结

`livekit-android-camerax` 是一个设计精巧的扩展模块：通过在核心 SDK 的 Provider 体系中注册一个更高优先级的 CameraProvider，在不修改核心 SDK 任何代码的前提下，实现了用 CameraX 替代底层 Camera2 API。其 5 个核心类各司其职，代码量极少但功能完整。

其中 `CameraXSession` 作为核心中的核心，承担了：异步启动编排、摄像头精确匹配、Camera2 底层参数注入、帧变换修正、线程安全保障、错误恢复和性能监控七项关键职责，是理解整个模块设计思想的最佳切入点。它的主要权衡在于用 CameraX Preview UseCase 作为视频帧采集通道，而非 CameraX 官方的 VideoCapture——这一选择使得帧管线与 Camera1/Camera2 保持完全一致，但也引入了一定的 CameraX 版本升级维护风险。
