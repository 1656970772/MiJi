---
source_url: "https://zhuanlan.zhihu.com/p/683746986"
type: webpage-summary
title: "Re：从零开始的UE渲染学习-3网格体绘制管线"
captured_at: 2026-05-11T20:42:08+08:00
author: "稻草人"
topics:
  - Unreal Engine
  - UE5
  - UE5.3
  - Rendering
  - Mesh Drawing Pipeline
  - FPrimitiveSceneProxy
  - FPrimitiveSceneInfo
  - FMeshBatch
  - FMeshDrawCommand
  - FMeshPassProcessor
  - FPassProcessorManager
  - Mesh Pass
  - Custom Pass
related_source:
  - "https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/3%E7%BD%91%E6%A0%BC%E4%BD%93%E7%BB%98%E5%88%B6%E7%AE%A1%E7%BA%BF"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/mesh-drawing-pipeline-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/ja-jp/unreal-engine/mesh-drawing-pipeline?application_version=4.27"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/FMeshBatch?application_version=5.6"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Renderer/FMeshDrawCommand/SubmitDraw"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Renderer/FMeshPassDrawListContext"
capture_scope: "摘要型资料卡；未全文转存"
---

# UE 网格体绘制管线资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《Re：从零开始的UE渲染学习-3网格体绘制管线》，链接为 `https://zhuanlan.zhihu.com/p/683746986`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `稻草人`，作者简介为 `路边的菜鸡TA...`。
- 文章收录于专栏 `UE学习笔记`，专栏链接为 `https://zhuanlan.zhihu.com/c_1747449492977352704`。
- 页面结构化数据中显示创建时间与更新时间均为 `2024-03-06 00:49:45 +08:00`。
- 截取时页面结构化数据中显示约 `30` 赞同、`1` 评论。
- 页面话题包括 `渲染`、`UE5`、`UE`。
- 作者说明本文涉及源码版本为 `UE5.3`，主题是网格在渲染时经历哪些步骤，目标是帮助理解自定义 pass 和 UE 绘制特性。

## 文章结构

- `1.前言`
- `2.虚拟世界的描述`
- `3.简单理解网格体绘制管线`
- `3.1FMeshDrawCommand`
- `3.2FMeshBatch`
- `3.3FMeshPassProcessor`
- `3.4FPassProcessorManager`
- `3.5计算网格相关性`
- `3.6提交绘制网格绘制命令`
- `4.UE的实现步骤`
- `5.缓存`
- `6.总结`
- `参考`

## 作者附带图源

原文给出 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/知乎/Re从零开始的UE渲染学习/3网格体绘制管线`。

该目录包含 8 张流程图：

- `P1.虚拟世界的描述.png`
- `P2.网格体绘制流程图.png`
- `P3.MeshProcessor.png`
- `P4.BuildMeshDrawCommands.png`
- `P5.FPassProcessorManager.png`
- `P6.MeshRelevance.png`
- `P7.SubmitDraw.png`
- `P8.CachedMeshDrawCommand.png`

我的判断：这篇是 UE 渲染学习系列里承上启下的一篇。第二篇讲线程边界，这篇讲“一个可渲染组件如何变成一个 mesh pass 里的 draw command”，对后续添加自定义 pass 或理解 BasePass/DepthPass 很关键。

## 外部可核验来源

- Epic 官方 Mesh Drawing Pipeline 文档说明：`FPrimitiveSceneProxy` 通过 `GetDynamicMeshElements` 和 `DrawStaticElements` 回调向 renderer 提交 `FMeshBatch`。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/mesh-drawing-pipeline-in-unreal-engine`。
- Epic 官方 Mesh Drawing Pipeline 文档说明：`FMeshBatch` 将 `FPrimitiveSceneProxy` 实现和 mesh pass 解耦，它包含 pass 决定最终 shader binding 和 render state 所需的信息，因此 proxy 不需要知道自己会在哪些 pass 渲染。来源同上。
- Epic 官方 Mesh Drawing Pipeline 文档说明：`FMeshDrawCommand` 是 `FMeshBatch` 和 RHI 之间的接口，是完全无状态的绘制描述，包含 RHI 对 mesh draw 需要知道的信息，例如 shader、resource binding 和 draw call 参数。来源同上。
- Epic 官方 Mesh Drawing Pipeline 文档说明：`FMeshDrawCommand` 由 mesh pass 特定的 `FMeshPassProcessor` 从 `FMeshBatch` 创建，最后通过 `SubmitMeshDrawCommands` 转换为设置到 `RHICommandList` 上的一系列 RHI command。来源同上。
- Epic 官方 `FMeshBatch` API 文档说明，`FMeshBatch` 是一批具有相同 material 和 vertex buffer 的 mesh element。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/FMeshBatch?application_version=5.6`。
- Epic 官方 `FMeshDrawCommand::SubmitDraw` API 文档说明，该函数向 RHI CommandList 提交 MeshDrawCommand 的绘制命令。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Renderer/FMeshDrawCommand/SubmitDraw`。

## 核心内容

这篇文章的主线是从 UE 的“两个世界”开始，解释网格体从游戏线程对象变成渲染线程绘制命令的过程：

1. 游戏线程用 `UWorld` 描述虚拟世界，可渲染对象通常通过 `UPrimitiveComponent` 提供渲染数据。
2. 渲染线程有对应的 `FScene`，保存要渲染的对象、灯光、特效等渲染侧数据。
3. `FPrimitiveSceneProxy` 作为游戏线程组件和渲染线程场景之间的代理桥梁。
4. `FPrimitiveSceneInfo` 存储 primitive 的渲染器内部状态。
5. 网格绘制管线把 `FPrimitiveSceneProxy` 转成 `FMeshBatch`，再由各个 `FMeshPassProcessor` 转成 `FMeshDrawCommand`。
6. 最终提交 `FMeshDrawCommand`，转成 RHI 命令并进入底层绘制。

我的判断：这篇资料适合用来理解“UE 里添加一个 mesh pass 到底要改哪些层”。它不是单纯讲 draw call，而是把组件、代理、pass、processor、draw command、RHI 提交串起来。

## 概念整理

### UWorld、FScene 与 Proxy

原文先解释游戏线程的 `UWorld` 和渲染线程的 `FScene`。`UPrimitiveComponent` 是游戏线程里可渲染组件的常见基类；渲染线程侧有 `FPrimitiveSceneProxy` 和 `FPrimitiveSceneInfo`。

我的判断：这延续了上一篇多线程的主题。UE 不会让渲染线程随意直接读游戏线程对象，而是通过 proxy 和 scene info 形成渲染侧表示，这既是线程安全设计，也是渲染架构解耦。

### FMeshDrawCommand

原文把 `FMeshDrawCommand` 定义为每个 `mesh pass` 下的专门绘制命令，存储 RHI 需要知道的 mesh draw 信息，包括 shader、资源绑定和 draw call 参数。官方文档也说明它是 `FMeshBatch` 和 RHI 之间的接口。

我的判断：`FMeshDrawCommand` 是“真正接近 draw call 的描述对象”。如果只看 `FPrimitiveSceneProxy`，还停留在“这个对象能画什么”；看到 `FMeshDrawCommand` 才进入“这个 pass 要用什么状态、什么 shader、怎样提交”的层面。

### FMeshBatch

原文解释 `FMeshBatch` 用于把 `FPrimitiveSceneProxy` 和具体 mesh pass 解耦。官方文档也说明，`FMeshBatch` 包含 pass 确定最终 shader binding 与 render state 所需的信息。

我的判断：`FMeshBatch` 是中间表示。它让组件 proxy 不需要知道自己会在 depth pass、base pass、shadow pass 还是其他 pass 里怎么画，这一点对自定义组件和自定义 pass 都很重要。

### FMeshPassProcessor

原文说明 `FMeshPassProcessor` 的 `AddMeshBatch()` 负责把 `FMeshBatch` 转成该 pass 对应的 `FMeshDrawCommand`。官方文档也说明具体 pass processor 需要从 `FMeshPassProcessor` 派生，负责最终过滤、shader 选择、shader binding 收集，并通常调用 `BuildMeshDrawCommands()`。

我的判断：想添加自定义 mesh pass，`FMeshPassProcessor` 是核心入口之一。它决定“这个 pass 接收哪些 mesh、用哪些 shader、用什么 pipeline state、如何生成 draw command”。

### FPassProcessorManager

原文把 `FPassProcessorManager` 解释为 pass processor 创建函数管理器，通过 `EShadingPath` 和 `EMeshPass::Type` 找到对应 pass 的创建函数。自定义 pass 通常还要添加自己的 `EMeshPass::Type`，并注册创建函数。

我的判断：这部分适合当自定义 pass 的检查清单：有 processor 类还不够，还要让系统能按 shading path 和 pass type 找到它。

### Mesh Relevance

原文把计算网格相关性解释为预筛选：通过可见性、LOD、光照等条件筛选和某个 pass 相关的 mesh。自定义 pass 也需要在相关性计算里加入自己的逻辑，例如涉及动态和静态 mesh 的不同路径。

我的判断：很多自定义 pass “写了 processor 但没有东西画出来”，问题可能不在 shader，而在 mesh relevance、pass flag、view relevance 或静态/动态路径没有接上。

### Submit

原文说明最终要设置参数并提交绘制命令。官方 `FMeshDrawCommand::SubmitDraw` API 也说明它向 RHI CommandList 提交 MeshDrawCommand 的绘制命令。

我的判断：这一步是从 renderer 层进入 RHI command list 的边界。排查 draw 是否真的提交时，可以沿着 `FMeshDrawCommand`、visible draw command、submit path 和 RHI command list 去追。

### 缓存

原文提到动态网格和静态网格路径，以及缓存 `FMeshDrawCommand` 的意义。官方文档也区分 cached mesh batch 和 dynamic mesh batch：静态路径可复用构建结果，动态路径更灵活但每帧重建。

我的判断：缓存是理解 UE mesh drawing 性能的关键。静态 mesh、动态 mesh、材质变化、view relevance、pass relevance 会影响 draw command 是否能复用，以及能否减少每帧 CPU 开销。

## 自定义 Pass 迁移清单

我的判断：基于原文和官方 Mesh Drawing Pipeline 文档，如果要添加一个自定义 mesh pass，最少要检查这些点：

- 是否需要新的 `EMeshPass::Type`。
- 是否实现了自己的 `FMeshPassProcessor`。
- 是否重写 `AddMeshBatch()` 做过滤、shader 选择、pipeline state 和 draw command 构建。
- 是否通过 `FRegisterPassProcessorCreateFunction` 或相应机制注册 processor 创建函数。
- 是否在 mesh relevance / view relevance / static mesh relevance 里让目标 mesh 进入该 pass。
- 是否处理动态 mesh 与静态 mesh 的不同路径。
- 是否在 RDG pass 或对应 dispatch 阶段提交 draw command。
- 是否考虑缓存路径、排序、PSO、资源绑定和 RHI 线程提交成本。

## 适合场景

- 阅读 UE Mesh Drawing Pipeline 源码前建立整体图。
- 理解 `FPrimitiveSceneProxy`、`FMeshBatch`、`FMeshDrawCommand`、`FMeshPassProcessor` 的关系。
- 准备添加自定义 mesh pass 或自定义 buffer。
- 排查 mesh 没有进入某个 pass、draw command 没生成、静态/动态路径没接上的问题。

## 不适合场景

- 不适合作为完整 UE renderer 源码剖析；原文是入门导读。
- 不适合作为 Nanite、Lumen、Ray Tracing 专门管线说明；这些路径会有额外机制。
- 不适合作为所有 UE 版本的精确实现说明；文章基于 UE5.3，具体类名和调用路径仍需结合当前项目源码核验。

## 后续检索关键词

- 中文：`UE 网格体绘制管线`、`FMeshDrawCommand`、`FMeshBatch`、`FMeshPassProcessor`、`FPassProcessorManager`、`Mesh Relevance`、`UE 自定义 Mesh Pass`
- English: `Unreal Engine Mesh Drawing Pipeline`, `FPrimitiveSceneProxy FMeshBatch FMeshDrawCommand`, `FMeshPassProcessor AddMeshBatch`, `FPassProcessorManager`, `SubmitMeshDrawCommands`, `custom mesh pass Unreal`

## 来源

- 用户提供材料：知乎专栏文章《Re：从零开始的UE渲染学习-3网格体绘制管线》，`https://zhuanlan.zhihu.com/p/683746986`。
- 作者 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/3%E7%BD%91%E6%A0%BC%E4%BD%93%E7%BB%98%E5%88%B6%E7%AE%A1%E7%BA%BF`。
- Epic Developer Community：`Mesh Drawing Pipeline in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/mesh-drawing-pipeline-in-unreal-engine`。
- Epic Developer Community：`FMeshBatch`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/FMeshBatch?application_version=5.6`。
- Epic Developer Community：`FMeshDrawCommand::SubmitDraw`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Renderer/FMeshDrawCommand/SubmitDraw`。
- Epic Developer Community：`FMeshPassDrawListContext`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Renderer/FMeshPassDrawListContext`。
