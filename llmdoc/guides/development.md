# 开发、验证与打包

## 先选平台，再选验证层级

macOS Swift 版直接依赖 Apple 框架，需要 Mac 与合适的 Xcode/SDK；Python/Qt 版是单独实现，主要发布为 Windows 便携包。不要用 Python 跑通推断 Swift 也通过，或用数值自测推断模型、界面和编码都通过。

本页命令以仓库根目录为起点，除 Windows 段明确要求的工作目录。依赖版本和构建参数以 `Package.swift`、`windows/requirements.txt` 及构建脚本为准。

## macOS：核心 CLI 与应用

```zsh
# 在 Mac 的仓库根目录
swift build -c release --product birdlock-cli
B=.build/release/birdlock-cli

$B selftest
BIRDLOCK_MODELS="$PWD/Models" $B models
BIRDLOCK_MODELS="$PWD/Models" $B detect /绝对路径/bird.jpg

# 真实照片集成验证；会写代理与分析缓存
BIRDLOCK_MODELS="$PWD/Models" $B analyze /绝对路径/连拍文件夹 \
  --fresh --debug tmp/debug --json tmp/analysis.json --csv tmp/track.csv

# 会真正导出视频；--dump-plan 不是 dry-run
BIRDLOCK_MODELS="$PWD/Models" $B export /绝对路径/连拍文件夹 \
  --fresh --preset lead --edge shift --aspect 9:16 --short 1080 \
  --fps 30 --speed 0.5 --interp blend --dump-plan --out tmp/check.mp4
```

`BIRDLOCK_MODELS` 影响模型查找，核实日志或 `models` 的实际加载信息。模型与 JSON 必须配套，参见 [模型契约](../reference/models.md)。

### 自测能证明什么

- `selftest` 检查合成纹理的相位相关，以及含异常观测的合成轨迹融合。
- 使用随机数据；不是固定真实飞鸟样本，也不覆盖识别、几何边缘、UI、编码、音乐和 AI 补帧。
- **当前 Swift selftest 打印 FAIL 时不主动返回非零退出码**，不能只看进程 exit code。
- `Package.swift` 没有 `testTarget`，不要报告 `swift test` 已覆盖核心回归。
- `detect` 是单图模型烟测，不等于正式分析中的完整高分辨率裁剪与追踪流程。
- CLI 分析整个输入文件夹；GUI 会组织连拍组并默认选最长组，比较结果前保证输入范围一致。
- `--fresh` 跳过分析缓存，但仍可能复用已有代理；`--noreg` 当前未接到真正关闭配准的路径，不要用它作无配准对照。

### App 与分发包装

```zsh
zsh scripts/build_app.sh
open dist/BirdLock.app

# 以下是分发包装操作，执行前确认授权和覆盖范围
VERSION=1.1 zsh scripts/make_dmg.sh
VERSION=1.1 LABEL=测试版 zsh scripts/make_release.sh
```

`build_app.sh` 重建 `dist/BirdLock.app`，复制模型、JSON、协议与手册，生成 HTML/PDF 并进行临时签名。通用构建由 `UNIVERSAL` 控制。DMG 和 ZIP 脚本会调用构建并覆盖对应输出，不要在里面保留唯一副本。

临时签名不等于 Developer ID 签名和 Apple 公证。`make_release.sh` 会从已跟踪与未忽略文件收集源代码，分发前检查临时文件与许可要求；不要把“脚本生成产物”和“已上传 Release”混为一谈。

## Windows / Python：开发

在 Windows 的 PowerShell 中：

```powershell
Set-Location windows
py -3.12 -m venv .venv
.venv\Scripts\pip install -r requirements.txt pyinstaller

.venv\Scripts\python -m birdlock.cli devices
.venv\Scripts\python -m birdlock.cli selftest
.venv\Scripts\python BirdLock.pyw
```

`devices` 用于查看运行环境与后端；自测成功不能单独证明真实照片推理使用了目标 GPU。Python 的图像重采样、模型推理、视频编码分别使用不同加速路径，见 [双平台架构](../architecture/platforms.md)。

Windows `selftest` 对相位相关、合成轨迹、模型构造、空图鸟检测和小尺寸 H.264 编码器试运行做检查，累计结果后以 0/1 退出；它不覆盖 pose 的真实推理精度、实际连拍分析、DIS 插帧质量、音乐合成或 GUI 操作。现有 Python CLI 也未提供插帧方式参数，默认的视频导出不能用于证明 DIS 已启用。

在其他操作系统做 Python 开发时，依赖清单使用平台条件选择 ONNX Runtime 包；必须分别验证目标 Windows 包装，不能把非 Windows 运行等同于 DirectML 成功。

### 打包与 CI

```powershell
# 当前工作目录 windows；会下载/收集 FFmpeg 并重建产物
.venv\Scripts\python build_windows.py --version 1.0
```

入口是 `windows/build_windows.py` 与 `windows/BirdLock.spec`。输出到 `windows/dist/`，包含 Python/Qt 运行时、ONNX 模型、FFmpeg、图标与说明文档。

`.github/workflows/windows.yml` 的实际行为：

1. 在 Windows runner 上安装 Python 依赖并打包。
2. 运行源码的 `devices`、`selftest`。
3. 运行包装后 EXE 的 `--selftest`，读取日志并检查退出码。
4. 运行 `--smoketest`，检查是否生成界面截图；截图存在不等于所有 UI 交互通过。
5. 上传 ZIP 和烟测截图为 workflow artifacts。
6. **仅 tag 构建**执行 `gh release upload --clobber`，向对应 Release 上传/替换资产。

普通文档分支推送不会触发这个以标签/手动为入口的工作流。不要为了验证文档创建 `v*` 标签或触发发布，也不要把手动 workflow 运行说成必定上传 Release。

## 推荐的最小集成样例

准备得到授权的照片与输出位置，分别检查：

- 横图与 EXIF 旋转图；不同解码尺度下眼点、裁剪和手动坐标一致。
- 单鸟、多鸟交叉、遮眼、头部转动、失焦、长段漏检。
- 手动修正的保存/撤销；重新识别同一只鸟与重新选择另一只鸟的区别。
- 排除/剪切、倒放/往返、速度与帧率、单帧输入和全部帧被过滤。
- 四种边缘策略、旋转取景、极宽/竖屏比例、鸟靠近原图边缘。
- 原图与代理预览、视频/GIF/图片序列差异、补帧降级、音频和水印。
- 最终包装内实际找到的模型、schema、FFmpeg/编码器和说明文档。

## 纯文档变更的检查

- `git diff --check`。
- 验证 `llmdoc/` 内 Markdown 相对链接能解析，索引列出新增文档，源码路径确实存在。
- 不为纯文档任务自动安装运行依赖、下载模型或启动训练。
- 若生成本地 HTML，检查结构与外部资源依赖；没有浏览器截图时，不宣称页面已完成视觉验证。

## 本次初始化的验证边界

本套文档创建时进行了源码与随附模型元数据的静态核对。当前环境是 Linux，缺少 Swift 工具链及 Python 的 NumPy/OpenCV/ONNX Runtime 等运行依赖：未运行应用、数值自测、模型推理、训练、GPU 验证或打包。已执行 Python AST 语法检查；它只证明语法可解析，不证明运行兼容性。真实运行状态应由后续目标平台验证重新确认，而不是沿用这段记录当成现状。
