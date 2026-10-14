# BirdLock 文档索引

这套文档帮助维护者先理解处理链路、约束和修改影响，再定位源码。它不替代用户手册，也不作为当前运行状态、参数默认值或性能数据的权威来源。

## 阅读入口

先读 [startup.md](startup.md)，再按任务定位。文档中的源码路径以仓库根目录为基准；具体实现、阈值和默认值须回到源码核实。

## must/：每次维护都要知道的约束

- [invariants.md](must/invariants.md)：坐标体系、分析与构图的职责边界、人工修正、缓存及平台能力边界。

## architecture/：方法与结构

- [pipeline.md](architecture/pipeline.md)：从连拍照片到稳定轨迹的处理链路；头部模板、相位相关与鲁棒融合为什么要一起用。
- [platforms.md](architecture/platforms.md)：macOS 与 Python/Qt 两条实现、构图与导出边界、平台功能差异。

## guides/：开发与验证

- [development.md](guides/development.md)：本地运行、自测、打包、验证覆盖范围与发布注意事项。

## reference/：精确定位的入口

- [models.md](reference/models.md)：模型输入输出、鸟类筛选、关键点语义以及训练和转换入口。
- [source-map.md](reference/source-map.md)：按问题导航到权威源码与现有文档。

## 使用与维护

- 面向用户的使用方法保留在 `docs/使用说明.md` 和 `docs/windows/使用说明.md`；构建脚本、依赖清单和模型元数据继续由对应源码文件维护。
- 新增、移动或删除本目录文档时，同时更新本索引；影响任务阅读入口时也更新 `startup.md`。
- 临时调查记录放在 `.llmdoc-tmp/`，不要混入稳定文档。只把经过核实、对后续维护有用的结论沉淀进来。
- 数值、参数与缓存版本优先链接权威源码，不复制一份容易漂移的配置。

## 找不到信息时

- 想知道项目在做什么 → `architecture/pipeline.md`。
- 鸟眼跳动、遮挡后漂移、跟错鸟 → `architecture/pipeline.md`，再查 `reference/source-map.md` 的分析入口。
- 裁切、边缘、预览与导出不一致 → `architecture/platforms.md`。
- Windows 与 macOS 的效果或选项不同 → `architecture/platforms.md`，不要只依据 README 的“功能一致”。
- 想找模型的某个数值或关键点编号 → `reference/models.md` 指向的元数据及加载代码。
- 想运行、验证或打包 → `guides/development.md`。
- 想看用户操作、协议或许可证 → `reference/source-map.md` 的既有文档入口。
