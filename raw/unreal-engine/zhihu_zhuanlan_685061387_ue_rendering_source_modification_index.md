---
source_url: "https://zhuanlan.zhihu.com/p/685061387"
type: webpage-summary
title: "Re：从零开始的UE渲染学习-6章节小结(前路漫漫亦灿灿)"
captured_at: 2026-05-11T20:53:42+08:00
author: "稻草人"
topics:
  - Unreal Engine
  - UE5
  - Rendering
  - Rendering Source Modification
  - Custom Shading Model
  - Custom Pass
  - Overlay Material
  - Custom Texture
  - Material Editor Extension
  - Primitive Component
  - Buffer Visualization
  - GBuffer
related_source:
  - "https://zhuanlan.zhihu.com/p/36675543"
  - "https://zhuanlan.zhihu.com/p/203631693"
  - "https://www.cnblogs.com/timlly/p/13512787.html#参考文献"
  - "https://www.zhihu.com/column/c_1428323433248092160"
  - "https://zhuanlan.zhihu.com/p/565837897"
  - "https://zhuanlan.zhihu.com/p/677772284"
  - "https://zhuanlan.zhihu.com/p/552283835"
  - "https://zhuanlan.zhihu.com/p/609331592"
  - "https://github.com/EpicGames/UnrealEngine/commit/d7b8804119f53887f81a0da157c7fee85d2bd592"
  - "https://github.com/EpicGames/UnrealEngine/commit/91b03a9e649ce90129275d3ced25b50627a08c0a"
  - "https://zhuanlan.zhihu.com/p/90829551"
  - "https://zhuanlan.zhihu.com/p/664866122"
  - "https://zhuanlan.zhihu.com/p/422751694?utm_id=0"
  - "https://zhuanlan.zhihu.com/p/565776677"
  - "https://www.cnblogs.com/timlly/p/15109132.html#942-材质开发案例"
  - "https://zhuanlan.zhihu.com/p/441017278"
  - "https://blog.csdn.net/qq_29523119/article/details/120806384?spm=1001.2014.3001.5501"
  - "https://zhuanlan.zhihu.com/p/458915355"
  - "https://zhuanlan.zhihu.com/p/668782106"
  - "https://zhuanlan.zhihu.com/p/568775542"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/mesh-drawing-pipeline-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterialExpression"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/MaterialEditor/FMaterialEditorUtilities/CreateNewMaterialExpression/2"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/UPrimitiveComponent?application_version=5.5"
capture_scope: "导航型资料卡；未全文转存；未逐篇深度复核所有外链"
---

# UE 渲染源码修改资料导航卡

## 来源事实

- 用户提供来源：知乎专栏文章《Re：从零开始的UE渲染学习-6章节小结(前路漫漫亦灿灿)》，链接为 `https://zhuanlan.zhihu.com/p/685061387`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `稻草人`，作者简介为 `路边的菜鸡TA...`。
- 文章收录于专栏 `UE学习笔记`，专栏链接为 `https://zhuanlan.zhihu.com/c_1747449492977352704`。
- 页面结构化数据中显示创建时间与更新时间均为 `2024-03-09 16:58:20 +08:00`。
- 截取时页面结构化数据中显示约 `24` 赞同、`1` 评论。
- 页面话题包括 `UE`、`UE5`、`虚幻引擎`。
- 原文正文没有配图，主要由章节说明和外部资料链接组成。

## 文章结构

- `1.前言`
- `2.大佬们的专栏`
- `3.大佬们的分享-UE源码修改`
- `3.1自定义着色模型(Shading Model)`
- `3.2自定义Pass`
- `3.3覆盖材质(Overlay Material)`
- `3.4传入自定义纹理`
- `3.5材质编辑器拓展`
- `3.6自定义Primitive Component`
- `3.7自定义BufferVisualization(Debug)`
- `3.8自定义/拓展GBuffer`

## 原文链接清单

### 渲染编程基础专栏

- `YivanLee：虚幻4渲染编程专题概述及目录`：`https://zhuanlan.zhihu.com/p/36675543`
- `YivanLee：虚幻5渲染编程专栏概述及目录`：`https://zhuanlan.zhihu.com/p/203631693`
- `剖析虚幻渲染体系-开篇说明 - 0向往0 - 博客园`：`https://www.cnblogs.com/timlly/p/13512787.html#参考文献`
- `虚幻引擎游戏开发`：`https://www.zhihu.com/column/c_1428323433248092160`

### UE 源码修改方向

- 自定义 Shading Model：`https://zhuanlan.zhihu.com/p/565837897`
- 自定义 Pass / 自定义 Buffer：`https://zhuanlan.zhihu.com/p/677772284`
- 自定义 MeshDrawPass：`https://zhuanlan.zhihu.com/p/552283835`
- Overlay Material：`https://zhuanlan.zhihu.com/p/609331592`
- Epic Overlay Material 相关提交：`https://github.com/EpicGames/UnrealEngine/commit/d7b8804119f53887f81a0da157c7fee85d2bd592`
- Epic Overlay Material 相关提交：`https://github.com/EpicGames/UnrealEngine/commit/91b03a9e649ce90129275d3ced25b50627a08c0a`
- 向 MobileBasePass 传入 LUT：`https://zhuanlan.zhihu.com/p/90829551`
- 使用 ViewUniform 和 CurveAtlas 控制 Ramp：`https://zhuanlan.zhihu.com/p/664866122`
- 材质编辑器扩展：`https://zhuanlan.zhihu.com/p/422751694?utm_id=0`
- 材质中添加自定义变量：`https://zhuanlan.zhihu.com/p/565776677`
- 材质体系开发案例：`https://www.cnblogs.com/timlly/p/15109132.html#942-材质开发案例`
- 编辑器菜单栏扩展框架：`https://zhuanlan.zhihu.com/p/441017278`
- 自定义 `PrimitiveComponent`：`https://blog.csdn.net/qq_29523119/article/details/120806384?spm=1001.2014.3001.5501`
- 自定义 `BufferVisualization`：`https://zhuanlan.zhihu.com/p/458915355`
- 顶点色 debug view：`https://zhuanlan.zhihu.com/p/668782106`
- 自定义/扩展 `GBuffer`：`https://zhuanlan.zhihu.com/p/568775542`

## 外部可核验来源

- Epic 官方 Mesh Drawing Pipeline 文档可用于核对自定义 mesh pass、`FMeshPassProcessor`、`FMeshBatch`、`FMeshDrawCommand` 等路径。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/mesh-drawing-pipeline-in-unreal-engine`。
- Epic 官方 Global Shader 文档说明 Global Shader 不通过 Material Editor 创建，常用于固定几何、后处理、compute shader 或清屏等场景。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine`。
- Epic 官方 `UMaterialExpression` API 文档列出材质表达式类及 `Compile`、`GenerateHLSLExpression`、`GenerateHLSLStatements` 等函数，可用于核对材质节点扩展方向。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterialExpression`。
- Epic 官方 `FMaterialEditorUtilities::CreateNewMaterialExpression` API 文档说明可在材质图中创建指定 `UMaterialExpression` 子类节点。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/MaterialEditor/FMaterialEditorUtilities/CreateNewMaterialExpression/2`。
- Epic 官方 `UPrimitiveComponent` API 文档显示该类提供 `CreateSceneProxy()`，用于创建代表 primitive 进入渲染线程 scene manager 的 proxy。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/UPrimitiveComponent?application_version=5.5`。

## 核心内容

这篇文章是作者“从零开始的 UE 渲染学习”前几章之后的阶段性小结。它不是具体实现教程，而是把 UE 渲染源码修改常见方向做成资料索引：先给渲染编程专栏，再按自定义着色模型、自定义 pass、覆盖材质、自定义纹理输入、材质编辑器扩展、自定义 PrimitiveComponent、BufferVisualization 和 GBuffer 扩展分类列出参考文章。

我的判断：这张卡应当作为“路线索引”保存，不应当当成某个功能的唯一实现依据。依据是原文前言明确说作者没有提供实现内容，而是在汇总他人修改分享；同时这些链接横跨 UE4、UE5.1、UE5.3 和不同作者的引擎修改路径。

## 使用路线

我的判断：如果后续要按这篇索引真正动 UE 源码，可以按下面顺序查：

1. 先读基础专栏，补齐 UE renderer、材质、mesh drawing、shader 编译的整体地图。
2. 如果目标是“画一个新 pass”，优先读自定义 Pass、自定义 MeshDrawPass，并同时对照 Epic Mesh Drawing Pipeline 文档。
3. 如果目标是“材质面板多一个能力”，优先读材质编辑器扩展、自定义材质变量、材质体系开发案例，并对照 `UMaterialExpression` 与 `FMaterialEditorUtilities` API。
4. 如果目标是“组件自己提供渲染数据”，优先读自定义 PrimitiveComponent，并对照 `UPrimitiveComponent::CreateSceneProxy()`。
5. 如果目标是“调试视图或 GBuffer 数据”，优先读 BufferVisualization、顶点色 debug view、Customize GBuffer，并结合项目具体 UE 版本核对 shader、配置和 renderer 改动点。

依据：原文按这些方向分组；Epic 官方 API 和 Mesh Drawing Pipeline 文档分别覆盖了 pass、材质节点和 primitive proxy 的基础入口。

## 与已有资料卡的关系

我的判断：这篇可以作为当前 `raw/unreal-engine` 里几篇 UE 渲染资料的索引页：

- 先读 `zhihu_zhuanlan_680266148_ue_rendering_learning_prologue.md` 建立学习序言。
- 再读 `zhihu_zhuanlan_661479451_ue_rendering_multithreading.md` 理解线程与渲染侧数据边界。
- 再读 `zhihu_zhuanlan_683746986_ue_mesh_drawing_pipeline.md` 理解 mesh pass 与 draw command。
- 再读 `zhihu_zhuanlan_684857706_ue_shader_material_system.md` 理解材质、shader、vertex factory 和 ShaderMap。
- 最后用本卡选择具体源码修改方向。

依据：这些资料卡均来自同一作者或同一 UE 渲染学习链路，且主题顺序从概念、线程、mesh drawing、shader/material 逐步走向源码修改实践。

## 风险与注意

- 原文是导航汇总，外链内容并未在本次资料卡中逐篇深度复核。
- 部分资料基于 UE4 或具体 UE5 小版本，不能直接套到当前项目版本。
- 自定义 Shading Model、GBuffer、MeshDrawPass 往往涉及引擎源码修改、shader 编译缓存、平台兼容和后续合并成本。
- EpicGames/UnrealEngine 的 GitHub 提交链接可能需要已授权的 Epic/GitHub 账号才能访问。

我的判断：这类资料最适合在做技术预研时建立“应该搜什么、可能改哪里”的地图；正式实现前仍要回到目标 UE 版本源码、官方 API 文档和最小可复现工程验证。依据是原文链接跨版本、跨作者，且多个主题属于高耦合 renderer 修改。

## 后续检索关键词

- 中文：`UE 自定义 Shading Model`、`UE 自定义 MeshDrawPass`、`UE 自定义 Pass 写入 Buffer`、`UE Overlay Material`、`UE 材质编辑器扩展`、`UE 自定义 PrimitiveComponent`、`UE 自定义 BufferVisualization`、`UE 扩展 GBuffer`
- English: `Unreal Engine custom shading model`, `Unreal Engine custom mesh draw pass`, `FMeshPassProcessor custom pass`, `Overlay Material Unreal Engine`, `UMaterialExpression custom node`, `UPrimitiveComponent CreateSceneProxy`, `Unreal Engine Buffer Visualization`, `Customize GBuffer UE5`

## 来源

- 用户提供材料：知乎专栏文章《Re：从零开始的UE渲染学习-6章节小结(前路漫漫亦灿灿)》，`https://zhuanlan.zhihu.com/p/685061387`。
- 原文列出的外链：见本文“原文链接清单”。
- Epic Developer Community：`Mesh Drawing Pipeline in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/mesh-drawing-pipeline-in-unreal-engine`。
- Epic Developer Community：`Adding Global Shaders to Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine`。
- Epic Developer Community：`UMaterialExpression`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterialExpression`。
- Epic Developer Community：`FMaterialEditorUtilities::CreateNewMaterialExpression`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/MaterialEditor/FMaterialEditorUtilities/CreateNewMaterialExpression/2`。
- Epic Developer Community：`UPrimitiveComponent`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/UPrimitiveComponent?application_version=5.5`。
