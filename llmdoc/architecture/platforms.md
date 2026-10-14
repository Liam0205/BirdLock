# 双平台架构、构图与导出

## 两套实现，一条概念链路

| 职责 | macOS 原生版 | Windows / Python 版 |
|---|---|---|
| 应用入口与界面 | SwiftUI，`Sources/BirdLockApp/` | PySide6 / Qt，`windows/birdlock/app/` |
| 分析与轨迹 | `Sources/BirdLockCore/Analysis/` | `windows/birdlock/core/analysis.py`、`head.py`、`phasecorr.py`、`solver.py` |
| 模型推理 | Core ML | ONNX Runtime |
| 构图 | `Render/CompositionPlanner.swift` | `core/planner.py` |
| 图像渲染 | Core Image，优先 Metal | OpenCV，可尝试 OpenCL |
| 视频编码与音频 | AVFoundation、VideoToolbox | FFmpeg |
| CLI | `Sources/BirdLockCLI/main.swift` | `windows/birdlock/cli.py` |

Python 版可以在非 Windows 系统用于开发，但这不代表 macOS Swift 版可以在 Linux 构建。两套核心是独立实现，不共享源代码，也不保证缓存、配置或所有参数的语义完全一致。分析缓存虽然使用相同版本号，也不是跨平台工程格式；数据字段、时间基准和路径表达不同。Python 便携包仍保存照片、音乐等资源的绝对路径，迁移目录不保证自动重新链接。

## 从轨迹到构图

分析输出的是原图中的眼睛、头中心、头长和角度；构图设置才决定目标画面。基本变换可以写成：

```text
输出位置 = 输出锚点 + 缩放 × 旋转 × (原图位置 - 原图锁定点)
```

- 原图位置和锁定点用应用 EXIF 方向后的原图像素。
- 配置中的输出锚点按成片宽高归一化；规划后转成输出像素。
- 缩放由输出短边、取景倍数和参考头长决定。
- 锁定大小与固定取景决定鸟随远近变化时是否自动改变倍率。
- 柔和跟随是对轨迹/锁定点做平滑，允许鸟在画面中浮动。
- 自动方向留白调整目标锚点，不是把鸟的照片左右翻转。

Core Image 使用左下原点，Swift 渲染器在边界处做坐标转换；Python 图像主要按左上原点处理。改代理尺寸、EXIF 方向、裁剪区域或 UI 手势时，检查正反变换而不是只看正常横图。

权威入口：Swift `Composition.swift`、`CompositionPlanner.swift`、`FrameRenderer.swift` 和 App 原图手势；Python `settings.py`、`planner.py`、`render.py`、`app/preview.py`。

## 边缘策略是明确的取舍

| 策略 | 优先目标 | 代价与边界 |
|---|---|---|
| 平移取景 / shift | 尽量使用原图，避免鸟贴边 | 允许锁定点偏离；必要的原图适配放大与额外边缘放大是不同步骤 |
| 自动放大 / zoom | 尽量保持目标锚点，靠放大避免越界 | 会降低可用像素和场景范围；达到限制后仍可能平移或无法完全满足 |
| 模糊延展 / blur | 保持构图大小 | 使用已有边缘像素的模糊延展，不是生成真实场景 |
| 黑边 / black | 保持构图大小 | 原图之外无像素的位置留黑 |

这些几何决策由 planner 负责。渲染器应用计划，并不独立保证所有组合都不越界；不能把“最大放大”或“边缘最小距离”写成无条件承诺。构图统计中的处理前越界数量也不等于最终黑边数量。

## 渲染、去闪烁与叠加

去闪烁根据背景亮度/颜色统计形成逐帧校正，不是再次识别鸟眼。Swift 使用线性颜色工作空间，缩小倍率较大时会先做 Lanczos；Python 渲染采用自己的 OpenCV 重采样路径。

macOS 输出时先对两张照片分别构图与颜色校正，再补帧/混合，最后加文字、对焦框、图片水印和淡变。这使水印不被光流当成鸟体运动扭曲。Python 的插值与叠加顺序须按自己的 exporter/render 实现核实，不能照搬 Swift 假设。

“对焦框”是根据软件识别结果绘制的示意标记，不是相机真实对焦记录。眼睛不可见时是否画头框，与实际使用的锁定轨迹是不同的设置。

## 选帧与时间轴

排除单帧、剪起止、跳过模糊和无有效轨迹都属于输出选择。当前实现不因此自动删除该帧在分析里的测量、模板观测或配准 link；平滑也可能继续受到全组数据影响。

macOS 时间映射把选后的来源帧重新排为相邻序列，再按来源速度、播放速度与输出 FPS 采样。跳过帧会压紧时间，不保留原始间隔。正放、倒放、往返、重复与淡变属于这一步。

图片序列通常按被选来源帧逐张输出，与视频时间采样不同：不能推定它包含补帧、播放重复或所有视频淡变。视频、GIF、图片序列各自的具体路径见 exporter。

## 同名补帧选项不代表相同算法

| 路径 | 实际行为 |
|---|---|
| macOS 视频 AI 补帧 | 使用 VideoToolbox 帧率转换；需要合适的 macOS 版本、SDK、硬件与尺寸支持 |
| macOS 不支持或转换失败 | 返回空结果后降级为帧混合；当前 UI 不保证明确显示降级 |
| macOS 预览、GIF | 没有视频导出注入的 AI 转换回调，使用帧混合路径 |
| Python 光流补帧 | 视频导出使用 OpenCV DIS 双向稠密光流及重映射，不是 macOS VideoToolbox AI 模型；预览和 GIF 没有注入该插帧器，也走帧混合 |

权威入口：Swift `AIInterpolator.swift`、`VideoExporter.swift`、`RenderJob.swift`，Python `render.py` 和 `exporter.py`。不要只看到配置名 `ai` 就报告“AI 后端已启用”。

## 编码与加速要看实际可用性

Core ML 的 `.all` 允许系统选择执行设备，并不强制使用神经网络引擎。Metal 不可用时渲染也有其他上下文路径。VideoToolbox AI 支持须单独判断，与普通视频能否编码不是同一件事。

Python 版的推理、图像处理和视频编码是三个独立加速面。DirectML/其他 ONNX provider、OpenCL 与 FFmpeg 硬件编码器不应统称成“用了 GPU”。设备可用、provider 在 session 中实际启用、编码器能否成功启动，需要分别核实；见 `gpu.py`、`models.py` 和 `exporter.py`。编码器选型会先试小尺寸编码，不支持时可选择其他编码器；但选定硬件编码器在真实导出中失败，不会自动重试软件编码。

macOS 支持的视频/GIF/图片序列、音乐混入和编码选项由 Swift exporter 定义；Python 则由 FFmpeg 命令及包装资源定义。编码名不保证输入管线位深、HDR 或透明度；例如选择 ProRes 4444 不能据此宣称当前渲染管线保留高位深和 alpha。

## 状态、预览与持久化

状态层连接导入、分析、人工修正、构图设置、预览任务和导出。修改设置应重建计划，修改目标身份应重新分析。UI、后台结果和导出不能依赖旧任务仍属于当前序列的假设。

macOS 播放/拖动预览主要用代理，暂停后再补更清晰图像；暂停显示单张来源图，与视频补帧时刻未必一致。水印 UI 与最终烘焙也不是同一条路径。Python 预览应依照自己的 Qt 状态与渲染入口检查。

若用户报告“预览与导出不同”，先判断是清晰度、时间采样、AI 降级、水印、颜色、选帧还是陈旧后台任务结果，再决定修改哪一层。入口清单见 [源码导航](../reference/source-map.md)，运行与集成验证见 [开发与验证](../guides/development.md)。
