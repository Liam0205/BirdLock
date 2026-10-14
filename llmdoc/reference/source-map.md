# 源码与既有文档导航

路径以仓库根目录为基准。此表是概念入口，不复制当前行号、阈值、默认值或全部文件清单。

## 分析链路

| 问题 | macOS / Swift | Python / Qt |
|---|---|---|
| 照片导入、EXIF、排序与分组 | `Sources/BirdLockCore/IO/SourceFrame.swift` | `windows/birdlock/core/frames.py` |
| 解码、代理与分析缓存 | `IO/ImageLoader.swift`、`IO/ProxyCache.swift` | `core/frames.py`、`core/analysis.py`、`core/paths.py` |
| 模型查找与 YOLO 张量解释 | `Vision/ModelStore.swift`、`Vision/YOLOModel.swift` | `core/models.py`、`core/gpu.py` |
| 分析总调度 | `Analysis/Analyzer.swift` | `core/analysis.py` |
| 目标鸟选择与连续追踪 | `Analysis/TargetTracker.swift` | `core/analysis.py` |
| 头部模板与遮眼推算 | `Analysis/HeadModel.swift` | `core/head.py` |
| 相位相关及置信度 | `Analysis/PhaseCorrelation.swift` | `core/phasecorr.py` |
| 观测融合与轨迹构建 | `Analysis/TrajectorySolver.swift`、`Analysis/TrackBuilder.swift` | `core/solver.py` |
| 分析数据表示 | `Analysis/AnalysisModel.swift` | `core/analysis.py`、`core/head.py`、`core/solver.py` |

表中 Swift 未写完整前缀的路径位于 `Sources/BirdLockCore/`；Python 未写完整前缀的路径位于 `windows/birdlock/`。

## 构图、画面与导出

| 问题 | macOS / Swift | Python / Qt |
|---|---|---|
| 构图设置、预设、比例、锁定点 | `Render/Composition.swift` | `core/settings.py` |
| 取景计划与越界处理 | `Render/CompositionPlanner.swift`、`Render/RenderJob.swift` | `core/planner.py` |
| 原图重采样、去闪烁与叠加 | `Render/FrameRenderer.swift` | `core/render.py` |
| 对焦框与文字 | `Render/FocusMark.swift`、`Render/TextOverlay.swift` | `core/render.py` |
| 编码、帧序列、GIF | `Export/VideoExporter.swift`、`Export/ExportSettings.swift` | `core/exporter.py` |
| 光流补帧 | `Export/AIInterpolator.swift` | `core/render.py`、`core/exporter.py` |
| 背景音乐 | `Export/AudioMixer.swift` | `core/exporter.py`、`app/timeline.py` |

## 用户交互与状态

| 问题 | macOS / Swift | Python / Qt |
|---|---|---|
| 应用入口 | `Sources/BirdLockApp/BirdLockApp.swift` | `windows/BirdLock.pyw`、`windows/birdlock/app/main.py` |
| 异步任务、选鸟、人工修正与规划刷新 | `Sources/BirdLockApp/AppState.swift` | `windows/birdlock/app/state.py` |
| 预览驱动 | `Sources/BirdLockApp/PreviewEngine.swift` | `windows/birdlock/app/preview.py` |
| 设置、时间轴和导出 UI | `Sources/BirdLockApp/Views/` | `windows/birdlock/app/main_window.py`、`inspector.py`、`timeline.py` |
| CLI 与自测 | `Sources/BirdLockCLI/main.swift` | `windows/birdlock/cli.py` |

## 模型、开发与打包

- Swift 产品与平台约束：`Package.swift`。
- Python 运行依赖：`windows/requirements.txt`。
- 随附 Core ML 描述与关键点语义：`Models/*/metadata.json`、`Models/BirdPose.json`。
- ONNX 与关键点语义：`windows/models/`。
- 训练、数据转换和模型导出：`tools/train/`，先读 [模型契约](models.md)。
- macOS App、DMG、ZIP 发布包装：`scripts/build_app.sh`、`scripts/make_dmg.sh`、`scripts/make_release.sh`。
- Windows 便携包装：`windows/build_windows.py`、`windows/BirdLock.spec`。
- Windows CI：`.github/workflows/windows.yml`；运行前读 [开发与验证](../guides/development.md) 的外部副作用说明。
- Markdown 使用说明转换：`scripts/md2html.py`；macOS PDF 转换：`scripts/html2pdf.swift`。

## 面向用户和许可的既有文档

- 项目概览与用户功能清单：`README.md`。
- macOS 使用说明：`docs/使用说明.md`。
- Windows 开发概述：`windows/README.md`。
- Windows 使用说明：`docs/windows/使用说明.md`。
- 用户协议：`docs/用户协议.md`。
- 第三方许可：`docs/第三方许可.md`、`docs/windows/第三方许可.md`。
- 仓库许可证原文：`LICENSE`。

这些文档保留各自面向用户或分发的职责。`llmdoc/` 聚焦维护定位与不显然的约束，避免复制一套完整手册。
