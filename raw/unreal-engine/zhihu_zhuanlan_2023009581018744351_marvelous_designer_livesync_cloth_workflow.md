---
source_url: "https://zhuanlan.zhihu.com/p/2023009581018744351"
type: webpage-summary
title: "UF2025(shanghai)—Marvelous Designer 布料动画解算工作流：从建模到虚幻引擎的完整实践"
captured_at: 2026-05-11T19:17:09+08:00
author: "wlxklyh"
topics:
  - UE5
  - Marvelous Designer
  - LiveSync
  - Chaos Cloth
  - Geometry Cache
  - MetaHuman
related_source:
  - "https://support-connect.clo-set.com/hc/en-us/articles/45304300058137-What-is-LiveSync"
  - "https://support-connect.clo-set.com/hc/en-us/articles/45304309404441-Opening-CLO-MD-LiveSync-Mode"
  - "https://support.marvelousdesigner.com/hc/en-us/articles/47358232630937-Simulation"
  - "https://support.marvelousdesigner.com/hc/en-us/articles/47358347315481-Remeshing"
  - "https://support.marvelousdesigner.com/hc/en-us/articles/51752244831897-MetaHuman-DNA-Importer"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/alembic-file-importer-in-unreal-engine?application_version=5.6"
capture_scope: "摘要型资料卡；未全文转存"
---

# Marvelous Designer 到虚幻引擎布料动画工作流资料卡

## 来源事实

- 来源文章：知乎专栏《UF2025(shanghai)—Marvelous Designer 布料动画解算工作流：从建模到虚幻引擎的完整实践》。
- 作者：`wlxklyh`。
- 页面显示编辑 / 更新时间：`2026-04-07 13:54`。
- 主题标签：页面链接文本包含 `虚幻引擎`、`Chaos Cloth`。
- 源视频：`[UFSH2025]高效布料动画解算流程解析 | Sherry Yao 姚夏清 Marvelous Designer 资深3D设计师`，时长 `25分40秒`。页面里的 Bilibili 视频链接显示异常。
- 文章说明：该文由 AI 辅助生成，结合视频内容与技术实践经验整理。
- 官方佐证：
  - CONNECT 官方帮助中心说明，LiveSync 是连接 CLO / Marvelous Designer 与 Unreal Editor 的桥接工具，用于缩短需要在 Unreal Engine 中渲染的工作流。
  - CONNECT 的 LiveSync 2 用户指南说明，LiveSync 面板支持 `Update`、`Save`、`AsCloth`；服装可保存为 Chaos Cloth Asset，但需要启用 `Include Garment Simulation Data`；如果 CLO/MD 侧存在动画，可启用 `Include Cache Animation`，动画服装数据可保存为 Geometry Cache Asset。
  - Marvelous Designer 官方 Simulation 手册说明，`Animation (Stable)` 预设适合录制动画缓存；GPU 模拟可根据机器规格提升速度，并在 2024.2 版本改进了碰撞处理一致性。
  - Marvelous Designer 官方 Remeshing 手册说明，Remeshing 可生成对齐的自动重拓扑结果。
  - Marvelous Designer 官方 MetaHuman DNA Importer 手册说明，MD 可直接导入 MetaHuman DNA 数据，并将 Body / Head DNA 组合为 Avatar。
  - Epic 官方 Alembic Importer 文档说明，导入为 Geometry Cache 会创建可播放顶点变化序列的动画资产，性能会随网格复杂度变化。

## 核心问题

这篇资料讨论的是：如何用 Marvelous Designer 制作高质量服装，并根据项目需求选择两条进入虚幻引擎的路线：

- 静态模型流程：服装作为可实时交互资产进入 UE，再由 Chaos Cloth / Panel Cloth 等引擎侧解算。
- 动画缓存流程：在 MD 中完成高质量布料解算和录制，再以缓存动画 / Geometry Cache 的形式进入 UE，用于影视、过场和高精度展示。

我的判断：这篇文章的核心不是单独介绍 MD 或 UE，而是在讲“布料到底应该在哪一端解算”。实时游戏重交互和预算，影视渲染重确定性和细节，两个目标会导向完全不同的资产形态。

## 一句话结论

我的判断：Marvelous Designer 到 Unreal 的布料工作流应先按交付目标分流。需要实时交互、换装和可运行时响应时，优先走静态模型 + Chaos Cloth；需要稳定、高精度、可控镜头表现时，优先走 MD 动画缓存 + Geometry Cache。LiveSync 的价值在于把这两条路线之间的导出、材质、物理属性和缓存传递变成更少手工步骤的桥接流程。

依据：文章将静态模型和动画缓存并列为两种主工作流；CONNECT 官方文档确认 LiveSync 可保存 Chaos Cloth Asset 和 Geometry Cache Asset；Epic 官方文档确认 Geometry Cache 适合播放顶点变化序列，但性能与网格复杂度相关。

## 工作流拆解

### 1. Marvelous Designer 的角色

文章把 MD 定位为专业布料制作和解算工具：

- 使用 2D 版片、缝合线和 3D 模拟构建立体服装；
- 用真实面料属性、重力、摩擦、固定针等控制布料行为；
- 支持 FBX、USD、缓存动画等数据流；
- 可在录制前调高模拟品质，换取更稳定的离线布料结果。

我的判断：MD 更像“服装物理创作端”，UE 更像“渲染和运行时集成端”。如果项目把所有褶皱、碰撞和多层衣物都推给 UE 实时解算，制作负担会从美术工具转移到运行时性能预算上。

### 2. 静态模型流程

文章认为静态模型流程适合游戏开发、移动端、实时互动和动态换装。核心步骤是：

1. 在 MD 中完成服装设计。
2. 做网格优化、四边面化 / 重拓扑、配件减面。
3. 处理 PBR 材质、法线、UV 和必要的纹理烘焙。
4. 需要绑定时用 AvatarMode 或相关绑定流程生成可跟随角色骨骼的网格。
5. 导入 UE 后使用 Chaos Cloth / Panel Cloth 等实时布料解算。

官方 Remeshing 手册可佐证 MD 提供对齐自动重拓扑；官方 LiveSync 文档也说明衣装可通过 LiveSync 导入 Unreal，并在满足条件时保存为 Chaos Cloth Asset。

我的判断：静态模型流程的重点是“低成本可运行”。它牺牲一部分离线级细节，换取运行时交互、LOD、换装、碰撞响应和可调参能力。

### 3. 动画缓存流程

文章认为动画缓存流程适合影视、过场动画、高精度展示和复杂裙摆 / 多层衣物。核心步骤是：

1. 导入带动画的角色或可调姿势的模型。
2. 在 MD 中布置服装，调整固定针、摩擦力、模拟品质和时间采样。
3. 使用 CPU / GPU 模拟录制布料动画。
4. 可以对布料物理属性、风场、固定针、重力等做关键帧。
5. 将结果以缓存动画形式导入 UE，用 Sequencer 或 Geometry Cache 播放。

Epic 官方 Alembic Importer 文档说明，Geometry Cache 可播放顶点变化序列；CONNECT LiveSync 文档说明 `Include Cache Animation` 可把 CLO/MD 侧动画服装数据带入 Unreal，并保存为 Geometry Cache Asset。

我的判断：动画缓存流程的核心优势是确定性。布料结果在 MD 里已经解算完成，UE 端主要负责播放和渲染，因此更适合镜头制作；代价是缓存体积、不可实时交互和版本管理压力。

## 制作要点

### 1. 网格优化

文章强调导入 UE 前要做网格优化：

- 自动四边面化，生成更干净拓扑；
- 复杂服装可手动重拓扑；
- 对纽扣、拉链等配件做减面；
- 游戏角色建议将面数控制在合理范围，文章给出 `5-10 万面` 的经验值。

我的判断：这里的 `5-10 万面` 应视为文章经验建议，不是统一性能标准。真正标准要按平台、角色同屏数量、布料是否实时解算、LOD 策略和目标帧率来定。

### 2. 材质与纹理

文章提到 PBR 材质生成器、法线强度调整、UV 自动填充、动态 UV 更新和纹理烘焙。

我的判断：布料进 UE 前最好把“几何细节”和“贴图细节”分清。能烘焙到法线 / AO 的褶皱和缝线，不一定要保留成高面数几何；这对实时项目尤其关键。

### 3. 绑定与姿势测试

文章描述 AvatarMode 可用于自动绑定服装网格，并支持调整权重、姿势测试和雕刻修复穿模。

我的判断：绑定不是导出前的机械步骤，而是“布料是否能跟角色动画稳定合作”的检查点。腋窝、肘部、膝盖、腰带和袖口应配专门测试动作。

### 4. 录制前稳定性

文章建议录制前检查固定针、摩擦力、模拟品质和场景时间变换；动作帧间变化大时，通过更细的时间采样提升稳定性。

官方 Simulation 手册也说明 `Animation (Stable)` 适合录制动画缓存，GPU 模拟可提高速度但仍受计算机规格和功能限制影响。

我的判断：离线布料也不是“点录制就完事”。录制前的稳定性设置决定缓存结果是否干净，后期在 UE 里很难修复已经烘进缓存的穿模和抖动。

## LiveSync 2.0 / 2.x 对接

### 1. 文章描述的能力

文章总结 LiveSync 2.0 的价值：

- 集成 USD 工作流程；
- 一键导入服装、模特、材质、动画缓存；
- 将带动画的骨骼网格从 UE 导回 MD 完成后续解算；
- 勾选 `Include Garment Simulation Data` 后，将 MD 物理属性传给 UE 侧 Chaos Cloth 节点；
- 碰撞厚度可使用 MD 导出的布料厚度数值。

### 2. 官方文档能确认的部分

CONNECT 官方文档确认：

- LiveSync 连接 CLO / MD 和 Unreal Editor；
- LiveSync 面板的 `Update` 会按 Garment / Avatar / Scene 等导入选项拉取对象；
- `AsCloth` 可保存 Chaos Cloth Asset，并要求启用 `Include Garment Simulation Data`；
- `Include Cache Animation` 会把 CLO/MD 侧动画服装数据带入 UE，并保存为 Geometry Cache Asset；
- LiveSync 2.1.0 引入 Combine Pose，LiveSync 2.2.0 引入若干动画选项。

我的判断：文章对 LiveSync 2.0 的说法整体方向可信，但具体功能名、版本号和兼容性应以当前 CONNECT / Fab 文档和项目实际插件版本为准。尤其是 LiveSync 与 UE 版本、MD 版本的兼容关系，不能只看视频内容。

## MetaHuman 集成

文章提到 MD 提供 MetaHuman 导入器，可导入 MetaHuman 网格、骨骼关节和纹理，并基于尺寸表复用衣物资产、自动适配版片。

官方 MetaHuman DNA Importer 手册确认：MD 可导入 MetaHuman 的 Body / Head DNA 文件，并将组合后的 MetaHuman 加载为 Avatar；导入选项包括单位、自动安排点、自动创建 Fitting Suit、轴转换等。

我的判断：MetaHuman 服装工作流的价值在于复用和标准化。只要角色尺寸、姿势、DNA、骨骼 / Geometry Cache 路径统一，服装资产就能更接近“可批量适配”的生产资料，而不是每个角色手工试穿一次。

## 适合与不适合

### 适合

- 角色服装资产生产、换装系统、MetaHuman 服装适配；
- 影视和实时渲染中过场布料动画；
- 主角高质量服装、裙摆、披风、多层衣物；
- 需要把 MD 的服装物理属性和材质较完整地带入 UE 的流程；
- 技术美术能维护网格、材质、缓存、Chaos Cloth 和 LiveSync 版本的团队。

### 不适合

- 没有明确区分实时解算与缓存播放的项目；
- 希望一套高面数衣服同时满足移动端实时、影视特写和动态换装的项目；
- 缺少缓存资产管理、LOD 和版本兼容验证的团队；
- 对运行时交互要求高，但选择了纯 Geometry Cache 的场景；
- 对镜头确定性要求高，却把最终布料表现全部交给实时解算的场景。

## 可迁移清单

- 先按目标拆分资产路线：实时交互走静态模型 + Chaos Cloth，镜头确定性走动画缓存 + Geometry Cache。
- 为服装建立四类资产目录：MD 源文件、优化后静态网格、UE Cloth / Dataflow、缓存动画 / Geometry Cache。
- 对每件衣服记录：面数、LOD、材质贴图、布料厚度、物理属性、是否包含动画缓存。
- 在导入 UE 前完成重拓扑、配件减面、UV、法线 / AO 烘焙和关键动作姿势测试。
- 使用 LiveSync 前确认 MD 没有处于模拟或录制状态，并确认 LiveSync 连接为可用状态。
- 保存 Chaos Cloth Asset 时确认 `Include Garment Simulation Data` 已启用。
- 使用动画缓存时确认 `Include Cache Animation`、FPS、起止帧、Geometry Cache 播放范围和 Sequencer 对齐。
- 对 MetaHuman 项目建立统一的 DNA、A-Pose / 起始姿势、动画导出和服装试穿规范。
- 把 LiveSync、MD、UE 的版本兼容性写入项目检查表，不跨项目照抄插件版本。

## 我的判断

这篇资料最适合归入 `虚幻 / DCC / 布料 / Marvelous Designer / LiveSync / Chaos Cloth / Geometry Cache`。

它的价值在于把“服装生产”从单点工具技巧变成路线选择：静态模型路线服务实时交互，动画缓存路线服务镜头质量，LiveSync 服务两个软件之间的数据桥接。对项目管理来说，真正关键的是一开始就决定哪些布料在运行时算、哪些布料提前烘、哪些材质和褶皱要转成贴图、哪些区域需要保留几何复杂度。

我会把它作为 UE 角色服装管线的决策资料，而不是单纯的软件教程。项目越早把实时 / 缓存、面数 / 贴图、绑定 / 缓存、MetaHuman / 自定义角色这些边界定清楚，后期返工越少。

## 后续检索关键词

- 中文：Marvelous Designer 虚幻引擎、MD LiveSync、布料动画缓存、Chaos Cloth 物理属性、Geometry Cache 布料、MetaHuman DNA 导入、MD GPU 模拟、服装重拓扑
- English: Marvelous Designer LiveSync Unreal Engine, MD LiveSync Chaos Cloth, Include Garment Simulation Data, Include Cache Animation, Geometry Cache cloth, MetaHuman DNA Importer, Marvelous Designer GPU simulation, garment remeshing

## 来源

- 知乎原文：https://zhuanlan.zhihu.com/p/2023009581018744351
- CONNECT 官方 LiveSync 说明：https://support-connect.clo-set.com/hc/en-us/articles/45304300058137-What-is-LiveSync
- CONNECT 官方 LiveSync 2 面板说明：https://support-connect.clo-set.com/hc/en-us/articles/45304309404441-Opening-CLO-MD-LiveSync-Mode
- Marvelous Designer 官方 Simulation 手册：https://support.marvelousdesigner.com/hc/en-us/articles/47358232630937-Simulation
- Marvelous Designer 官方 Remeshing 手册：https://support.marvelousdesigner.com/hc/en-us/articles/47358347315481-Remeshing
- Marvelous Designer 官方 MetaHuman DNA Importer 手册：https://support.marvelousdesigner.com/hc/en-us/articles/51752244831897-MetaHuman-DNA-Importer
- Epic 官方 Alembic Importer / Geometry Cache 文档：https://dev.epicgames.com/documentation/en-us/unreal-engine/alembic-file-importer-in-unreal-engine?application_version=5.6
