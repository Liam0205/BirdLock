# 启动阅读顺序

## 必读

1. [index.md](index.md)：定位这次任务需要的文档。
2. [must/invariants.md](must/invariants.md)：先核实坐标、分析/构图职责边界、人工修正和缓存约束。
3. 按下面的任务提示继续，不必通读所有文档。

## 按任务加读

| 任务 | 先读 | 再定位源码 |
|---|---|---|
| 初次认识仓库 | [architecture/pipeline.md](architecture/pipeline.md)、[architecture/platforms.md](architecture/platforms.md) | [reference/source-map.md](reference/source-map.md) |
| 改检测、追踪、鸟眼估计、配准或轨迹融合 | [architecture/pipeline.md](architecture/pipeline.md) | [reference/models.md](reference/models.md)、[reference/source-map.md](reference/source-map.md) |
| 换模型、重新训练、改关键点语义 | [reference/models.md](reference/models.md) | 模型 JSON、两平台模型解析、头部模型与缓存版本 |
| 改裁切、锁定、旋转、画面边缘 | [architecture/platforms.md](architecture/platforms.md) | 两平台 planner、renderer 及预览交互 |
| 改人工鸟眼修正、目标选择或重新识别 | [architecture/pipeline.md](architecture/pipeline.md) | 两平台状态层、TrackBuilder/solver 及分析缓存 |
| 改导出、音乐、补帧、视频编码 | [architecture/platforms.md](architecture/platforms.md) | exporter、AIInterpolator 或 Python 插值入口 |
| 改构建、依赖、打包或 CI | [guides/development.md](guides/development.md) | Package.swift、requirements、构建脚本与 Windows workflow |
| 更新用户说明 | [reference/source-map.md](reference/source-map.md) 的既有文档入口 | 对应平台实际能力，避免扩大为算法保证 |

## 开始修改前

- 查阅文档后，以源码、实际环境和可运行验证核实这次要改的具体事实。
- 用简短说明明确修改范围和做法；需求、授权和重要取舍清楚时直接推进。
- 双平台代码是独立实现：改动一边后，检查另一边是否需要同步；能力名称相同不等于实现相同。
- 文档若提到测试数据，区分合成自测、真实连拍测试和硬件集成验证，不用一种覆盖替代另一种。

## 完成后

仅同步确实受影响的稳定约束、设计或操作方式；更新索引和必要的任务路由。报告实际执行的验证，以及因平台、依赖或数据不足而未执行的验证。
