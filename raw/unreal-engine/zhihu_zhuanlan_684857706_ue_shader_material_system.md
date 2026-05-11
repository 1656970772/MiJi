---
source_url: "https://zhuanlan.zhihu.com/p/684857706"
type: webpage-summary
title: "Re：从零开始的UE渲染学习-4Shader与材质"
captured_at: 2026-05-11T20:47:50+08:00
author: "稻草人"
topics:
  - Unreal Engine
  - UE5
  - UE5.3
  - Rendering
  - Shader
  - Material
  - Vertex Factory
  - FGlobalShader
  - FMaterialShader
  - FMeshMaterialShader
  - ShaderMap
  - UMaterial
  - UMaterialExpression
  - Material Graph
  - Shader Compilation
related_source:
  - "https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/4Shader%E4%B8%8E%E6%9D%90%E8%B4%A8"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/shader-development-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/essential-unreal-engine-material-concepts"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-material-expressions-reference"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/FMaterial/GetShader"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterialExpression/Compile"
capture_scope: "摘要型资料卡；未全文转存"
---

# UE Shader 与材质系统资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《Re：从零开始的UE渲染学习-4Shader与材质》，链接为 `https://zhuanlan.zhihu.com/p/684857706`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `稻草人`，作者简介为 `路边的菜鸡TA...`。
- 文章收录于专栏 `UE学习笔记`，专栏链接为 `https://zhuanlan.zhihu.com/c_1747449492977352704`。
- 页面结构化数据中显示创建时间与更新时间均为 `2024-03-07 00:46:15 +08:00`。
- 截取时页面结构化数据中显示约 `29` 赞同、`1` 评论。
- 页面话题包括 `渲染`、`虚幻 5 （游戏引擎）`、`UE5`。
- 作者说明本文涉及源码版本为 `UE5.3`，主线是解释 UE 的 Shader、材质系统，以及二者在渲染管线中如何关联。

## 文章结构

- `1.前言`
- `2.顶点工厂`
- `3.Shader`
- `3.1Shader类`
- `3.2Shader的实现?`
- `4.材质系统`
- `4.1简单理解shader与材质`
- `4.2材质、shader、顶点工厂的关系`
- `4.3材质类`
- `4.4材质蓝图`
- `4.5材质的编译`
- `4.6材质蓝图的翻译`
- `5.总结`
- `参考`

## 作者附带图源

原文给出 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/知乎/Re从零开始的UE渲染学习/4Shader与材质`。

该目录包含 11 张配图：

- `P1.Shader类型.png`
- `P2.材质编辑器.png`
- `P3.获取shader.png`
- `P4.材质类.png`
- `P5.材质蓝图.png`
- `P6.Add节点Compile.png`
- `P7.Shader编译流程.png`
- `P8.材质蓝图翻译.png`
- `P9.FHLSLMaterialTranslator_Add.png`
- `P10.模板CalcPixelMaterialInputs.png`
- `P11.翻译后CalcPixelMaterialInputs.png`

我的判断：这组图是理解本文的关键补充材料，尤其是 `P3.获取shader.png`、`P7.Shader编译流程.png`、`P8.材质蓝图翻译.png`，适合在阅读 UE 源码时作为调用路径和概念关系的索引。依据是原文多处用这些图承接类关系、编译流程和材质翻译流程。

## 外部可核验来源

- Epic 官方 `Adding Global Shaders to Unreal Engine` 文档说明 Global Shader 不通过 Material Editor 创建，通常用于固定几何、后处理、compute shader 或清屏等场景，并说明 UE 使用 `.usf` 文件存储 shader 信息。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine`。
- Epic 官方 `Shader Development in Unreal Engine` 文档是原文引用的着色器开发入口文档。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/shader-development-in-unreal-engine`。
- Epic 官方材质概念文档说明 Material 用于定义场景对象的表面属性，并通过节点式 Material Graph 构建材质。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/essential-unreal-engine-material-concepts`。
- Epic 官方 `Material Expressions Reference` 文档说明 Material Expression 是在 Material Editor 中构建材质的节点基础。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-material-expressions-reference`。
- Epic 官方 `FMaterial::GetShader` API 文档说明该函数按 shader 类型与 vertex factory 查找匹配 shader，并且 `FMeshMaterialShaderType` 版本会接收 `FVertexFactoryType`。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/FMaterial/GetShader`。
- Epic 官方 `UMaterialExpression::Compile` API 文档说明 `Compile(FMaterialCompiler*, int32 OutputIndex)` 用于为表达式创建 shader code chunk。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterialExpression/Compile`。
- Epic 官方 `FMeshMaterialShader` API 文档显示其头文件路径为 `/Engine/Source/Runtime/Renderer/Public/MeshMaterialShader.h`。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Renderer/FMeshMaterialShader/__ctor`。

## 核心内容

本文承接前一篇 UE 网格体绘制管线：当自定义 pass 在 `BuildMeshDrawCommands()` 中真正要绘制网格时，需要拿到一个具体 shader。作者因此先解释顶点工厂，再解释 UE 的 shader 类体系和材质系统，最后串起 `材质 -> ShaderMap -> shader 类型 + vertex factory 类型 -> 具体 shader` 这条链路。

我的判断：这篇不是单纯讲“怎么写一个 shader”，而是在补齐 mesh drawing pipeline 里最容易断掉的一段：`FMeshBatch` 携带材质后，BasePass 或自定义 pass 如何从材质和顶点工厂组合中取到最终要绑定的 shader。依据是原文前言、`4.2材质、shader、顶点工厂的关系` 与 Epic `FMaterial::GetShader` API 文档相互印证。

## 概念整理

### 顶点工厂

原文把顶点工厂解释为 UE 向 shader 解释顶点数据的机制。网格顶点可能包含位置、法线、UV、切线等信息，一些特殊网格还会在顶点着色器阶段做额外处理，例如 GPU 蒙皮。顶点工厂让 shader 能以统一方式取得目标顶点数据。

我的判断：如果只是写全屏后处理，通常不需要先深入顶点工厂；如果要自定义 mesh 数据布局、自定义 vertex stream、或让 mesh pass 正确读取特殊顶点数据，就必须理解 vertex factory。依据是原文顶点工厂小节对“自定义网格信息并正确解释进 shader”的说明。

### Shader 类体系

原文主要关注三类 shader 基类：

- `FGlobalShader`：不依赖材质编辑器和顶点工厂，适合固定效果或全屏类效果。Epic Global Shader 文档也说明这类 shader 不通过 Material Editor 创建。
- `FMaterialShader`：依赖材质编辑器，但不依赖顶点工厂，常见于需要材质表达式但不绑定具体 mesh vertex factory 的效果。
- `FMeshMaterialShader`：同时依赖材质编辑器和顶点工厂，是常见 mesh 渲染 shader 类型；Epic API 文档也提供了 `FMeshMaterialShader` 的运行时 Renderer 模块入口。

我的判断：选择 shader 基类时，可以先问两个问题：是否需要 Material Graph 的材质属性？是否需要 Vertex Factory 的 mesh 顶点解释？两个答案基本决定是走 global、material 还是 mesh material shader。依据是原文对三类 shader 的划分与 Epic Global Shader/FMeshMaterialShader 官方文档。

### Shader 实现入口

原文提到 UE 通过一组宏和类型把 shader 文件与 C++ shader 类连接起来，例如 `DECLARE_SHADER_TYPE`、`SHADER_PARAMETER`、`IMPLEMENT_MATERIAL_SHADER_TYPE`。这些机制通常绑定 shader 文件路径、入口函数、参数结构和编译环境。

我的判断：这里适合配合 Epic 官方 shader 文档一起读，不宜只记宏名。真正排错时要确认 shader 文件虚拟路径、入口函数名、参数结构、编译环境宏和模块加载时机。依据是原文 `3.2Shader的实现?` 与 Epic `Adding Global Shaders` 文档关于 `.usf` 文件和 shader class 声明的说明。

### 材质与 Shader

原文把 shader 解释为计算光照和输出渲染结果的程序，把材质解释为物体表面质感与属性集合，例如基色、金属度、粗糙度、半透明、着色模型、双面等设置。UE 的 Material Editor 让艺术家通过节点计算材质属性，再翻译成 HLSL 函数供 shader 调用。

我的判断：把“材质”和“shader”混成一个概念，会导致调试时定位错误。材质更像“属性与表达式的资产/配置”，shader 更像“使用这些属性完成某个 pass 绘制的程序”。依据是原文 `4.1简单理解shader与材质` 和 Epic 材质概念文档。

### 材质、Shader、顶点工厂的关系

原文说明 UE 会把材质使用到的 shader 类型与顶点工厂类型按组合编译，结果存储在材质的 `ShaderMap` 中。实际渲染时，可从 `FMeshBatch` 找到材质，再从材质 `ShaderMap` 中按 shader 类型与 vertex factory 类型取出具体 shader。Epic `FMaterial::GetShader` API 文档也说明按 shader 类型与 vertex factory 查找匹配 shader。

我的判断：这是本文最值得保存的主结论。自定义 pass 中“拿不到 shader”或“材质没有对应 permutation”时，排查重点通常不只是 shader 文件是否存在，还包括材质是否触发该 shader 类型、vertex factory 是否参与编译、permutation 和宏环境是否一致。依据是原文 `4.2材质、shader、顶点工厂的关系` 和 Epic `FMaterial::GetShader` API 文档。

### 材质相关类

原文整理了几个常见类：

- `UMaterial`：编辑器侧材质资产，包含材质属性设置和材质蓝图/图节点。
- `UMaterialInstance`：材质实例，以父材质为基础覆盖参数。
- `FMaterial`：偏渲染线程侧的材质表示，提供编译、材质信息访问和 `ShaderMap` 管理等能力。

我的判断：扩展材质编辑器设置时应关注 `UMaterial`；运行时渲染和 shader 查找应关注 `FMaterial`/`FMaterialRenderProxy` 这条渲染侧表示。依据是原文 `4.3材质类` 和 Epic `FMaterial` API 文档族。

### 材质蓝图与材质节点

原文说明 `UMaterialGraph` 管理材质节点和参数，材质节点使用图与数据分离思路，常见组合是 `UMaterialGraphNode` 与 `UMaterialExpression`。扩展自定义材质节点通常继承 `UMaterialExpression` 并实现编译相关函数，尤其是 `Compile()`。Epic `UMaterialExpression::Compile` API 文档也说明该函数用于创建表达式对应的 shader code chunk。

我的判断：自定义材质节点的核心不是 UI 节点长什么样，而是这个节点如何通过 `FMaterialCompiler` 生成可拼进 MaterialTemplate 的表达式代码。依据是原文 `4.4材质蓝图` 和 Epic `UMaterialExpression::Compile` API 文档。

### 材质编译与翻译

原文把材质编译入口概括为 `FMaterial::BeginCompileShaderMap()` 附近：先把 Material Graph 翻译成 HLSL，再结合 shader 类型和 vertex factory 类型的排列组合编译，结果进入 `ShaderMap`。作者还提醒编译环境宏会影响翻译和最终 shader 效果。

原文进一步指出，UE5.3 中材质蓝图到 HLSL 有新旧两套路径；旧路径以 `FHLSLMaterialTranslator` 为主，新路径也要注意 `MaterialHLSLEmitter.cpp` 中的 `GetMaterialEnvironment()`。材质翻译最终会填入 `/Engine/Private/MaterialTemplate.ush` 之类模板中的占位位置，生成 shader 可调用的函数。

我的判断：扩展材质编译或排查材质宏时，不能只看某一个 translator。UE5.3 既可能走旧路径，也可能走新 HLSL generator 相关路径，因此环境宏、MaterialTemplate、节点 `Compile()`/新生成接口都要一起核对。依据是原文 `4.5材质的编译`、`4.6材质蓝图的翻译` 和 Epic `UMaterialExpression` API 中同时存在 `Compile` 与 `GenerateHLSLExpression`/`GenerateHLSLStatements` 等函数。

## 自定义渲染开发检查清单

我的判断：基于原文和官方文档，如果要做自定义 mesh pass、材质节点或 shader，至少要检查这些点：

- 是否需要 `FGlobalShader`、`FMaterialShader` 还是 `FMeshMaterialShader`。
- shader 文件是 `.usf` 还是 `.ush`，虚拟路径、入口函数和模块加载时机是否正确。
- 参数结构是否通过 `SHADER_PARAMETER` 等机制和 HLSL 侧一致。
- 自定义 pass 是否能从 `FMeshBatch` 找到目标材质。
- 目标材质是否会编译该 shader 类型与对应 vertex factory 的组合。
- `ShaderMap` 中是否存在目标 permutation。
- 自定义顶点数据是否需要新的或扩展过的 vertex factory。
- 自定义材质节点是否正确实现 `UMaterialExpression` 的编译/生成逻辑。
- 修改材质宏或环境时，是否同时检查旧 translator 与新 HLSL generator 路径。

## 适合场景

- 阅读 UE5.3 材质系统和 shader 编译源码前建立地图。
- 理解 `BuildMeshDrawCommands()` 中 shader 从哪里来。
- 做自定义 mesh pass、自定义材质节点、自定义 shader permutation 前梳理依赖。
- 排查材质编译、ShaderMap 缺失、vertex factory 组合缺失、材质节点翻译异常等问题。

## 不适合场景

- 不适合作为完整 UE shader 编译系统源码剖析；原文是入门导读和关系梳理。
- 不适合作为所有 UE 版本的精确实现说明；原文明确基于 UE5.3，后续版本中材质翻译路径和 API 可能变化。
- 不适合作为 Shader 语言教程；它关注 UE 内部把材质、shader、vertex factory 串起来的方式。

## 后续检索关键词

- 中文：`UE Shader 材质系统`、`UE 顶点工厂`、`UE ShaderMap`、`FMaterial GetShader`、`FMeshMaterialShader`、`UMaterialExpression Compile`、`FHLSLMaterialTranslator`、`MaterialTemplate.ush`
- English: `Unreal Engine FMeshMaterialShader`, `Unreal Engine ShaderMap material vertex factory`, `FMaterial GetShader VertexFactoryType`, `UMaterialExpression Compile FMaterialCompiler`, `FHLSLMaterialTranslator MaterialTemplate.ush`, `custom mesh material shader Unreal Engine`

## 来源

- 用户提供材料：知乎专栏文章《Re：从零开始的UE渲染学习-4Shader与材质》，`https://zhuanlan.zhihu.com/p/684857706`。
- 作者 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/4Shader%E4%B8%8E%E6%9D%90%E8%B4%A8`。
- Epic Developer Community：`Adding Global Shaders to Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine`。
- Epic Developer Community：`Shader Development in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/shader-development-in-unreal-engine`。
- Epic Developer Community：`Essential Unreal Engine Material Concepts`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/essential-unreal-engine-material-concepts`。
- Epic Developer Community：`Unreal Engine Material Expressions Reference`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-material-expressions-reference`。
- Epic Developer Community：`FMaterial::GetShader`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/FMaterial/GetShader`。
- Epic Developer Community：`UMaterialExpression::Compile`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterialExpression/Compile`。
- Epic Developer Community：`FMeshMaterialShader`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Renderer/FMeshMaterialShader/__ctor`。
