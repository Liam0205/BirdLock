# 模型契约与转换入口

## 权威信息分层

- 随附模型实际接口：Core ML 的 `Models/BirdDetector.mlmodelc/metadata.json`、`Models/BirdPose.mlmodelc/metadata.json`，以及对应 ONNX session 的输入输出描述。
- 关键点语义：`Models/BirdPose.json`、`windows/models/BirdPose.json`。
- 运行时解释：Swift `Vision/YOLOModel.swift`、`Vision/ModelStore.swift`；Python `core/models.py`。
- 训练与转换方法：`tools/train/` 的脚本。脚本提供一条制作路径，不证明当前二进制模型必然来自该路径。

模型输入尺寸、输出通道数、模型文件查找顺序和阈值优先查这些权威文件，替换模型时重新检查，不把文档内的摘要当作可执行配置。

## BirdDetector

随附 Core ML 元数据将它描述为 Ultralytics YOLO11m、COCO 鸟类检测，输出未内置 NMS。运行时先做方形 letterbox，再解释原始 YOLO 输出、筛选鸟类和进行 NMS，最后回映到输入图像坐标。

因此“支持 YOLO”不意味着可直接替换成任意布局的端到端/NMS 模型。类别编号、张量布局、通道语义、图像输入类型及 padding 回映必须保持匹配。

## BirdPose

两平台随附 JSON 都是六关键点布局：

```text
beak, left_eye, right_eye, head_top, left_shoulder, right_shoulder
```

Core ML 随附元数据描述 YOLO11n-pose，训练配置名为 `wildlife_bird.yaml`。该描述只能支持接口与配置标识判断，不能据此证明完整训练数据来源、训练过程或许可。

当前分析先在目标鸟周围建立正方形裁剪，裁剪扩展从 JSON 配置读取，再运行 pose。头部模板筛选语义点，不直接使用全部 pose 点：六点模型中的肩部不参与头部拟合；一侧眼睛、喙和头顶才是主要几何观测。详见 [处理链路](../architecture/pipeline.md)。

## JSON 不是可有可无的说明文件

关键点顺序影响输出张量如何解释。Swift 模型加载在 JSON 缺失或不可解析时回退到内置 CUB15 schema，而不是自动从模型 `kpt_names` 恢复。当前六点模型若误配 CUB15，可能出现鸟框正常、姿态解析失败且在分析入口表现为缺姿态的情况。

模型替换和打包必须把权重与对应 JSON 一起处理，验证关键点数量、顺序、名称与输出通道一致。Python 端也应独立检查加载/回退逻辑，不推定与 Swift 行为完全相同。

## 仓库中的训练与导出工具

| 文件 | 维护入口 |
|---|---|
| `tools/train/prepare_cub.py` | CUB 数据转为 pose 训练格式，核查划分、关键点顺序和可见性语义 |
| `tools/train/train_pose.py` | pose 训练参数、权重与输出位置 |
| `tools/train/eval_pose.py` | 关键点评估与模型比较；确认评估集、可见点和度量定义 |
| `tools/train/export_models.py` | Core ML 模型导出、编译、配套 schema |
| `tools/train/export_onnx.py` | ONNX 模型导出及 Python 版配套 JSON |

执行脚本前直接读命令行入口、参数和安装要求。训练依赖、原始数据和大权重与应用运行依赖是两回事，不要为了理解仓库自动下载数据或运行训练。

## README 与许可说明的边界

`README.md` 仍使用 CUB-200 描述 BirdPose；当前六点 schema 与随附 Core ML 元数据并不支持把它当成当前模型的准确接口说明。`docs/第三方许可.md` 还记录了模型来源与训练数据许可说明，替换或分发前应独立核查对应实际模型，不能把静态元数据当作许可审计。

本次文档把该差异记在维护入口，没有修改模型、训练工具或已有用户/许可文档，也没有验证二进制权重的训练血缘。

## 更换模型的最小检查面

1. 两平台模型文件都能被各自加载器找到，打包时 JSON 与权重一起进入产物。
2. 输入类型、分辨率、letterbox/padding 和裁剪扩大参数正确。
3. 原始输出布局、类别数量、鸟类索引、关键点数量及顺序正确。
4. 用真实照片验证鸟框、眼点和头部模板，并观察遮挡、双眼切换及多鸟场景。
5. 核查缓存版本/失效策略，避免拿旧分析验证新模型。
6. 分别验证 CPU、目标 GPU 后端和最终打包产物；源码推理通过不能替代包装验证。
7. 对应更新来源、许可证和模型契约文档；不要复制无法核实的精度或训练数据声明。
