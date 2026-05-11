---
source_url: "https://zhuanlan.zhihu.com/p/1999455131893265237"
type: webpage-summary
title: "UF2025(Shanghai)：《洛克王国：世界》移动端渲染管线优化实战"
captured_at: 2026-05-11T21:16:13+08:00
author: "wlxklyh"
column: "UF2025(shanghai)"
topics:
  - Unreal Engine
  - UE4.26
  - Mobile Rendering
  - Forward Rendering
  - Tile-Based Rendering
  - RGB10A2
  - Custom Depth
  - Stencil
  - SSAO
  - Fog
  - SubPass
  - FrameBuffer Fetch
  - HDR Pipeline
  - RenderDoc
  - 洛克王国：世界
related_source:
  - "https://www.bilibili.com/video/BV1XK1YB4EyJ"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/mobile-rendering-and-shading-modes-for-unreal-engine?application_version=5.6"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-settings-in-the-unreal-engine-project-settings?application_version=5.6"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/post-process-effects-in-unreal-engine?application_version=5.6"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/UPrimitiveComponent/SetCustomDepthStencilValue"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterial/bEnableStencilTest"
  - "https://docs.vulkan.org/samples/latest/samples/performance/subpasses/README.html"
  - "https://github.khronos.org/Vulkan-Site/tutorial/latest/Building_a_Simple_Engine/Mobile_Development/04_rendering_approaches.html"
capture_scope: "摘要型资料卡；未全文转存；基于知乎文章与官方/可核验资料整理"
---

# 《洛克王国：世界》移动端渲染管线优化资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《UF2025(Shanghai)——移动端渲染管线优化实战：《洛克王国：世界》的技术突破之路》，链接为 `https://zhuanlan.zhihu.com/p/1999455131893265237`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `wlxklyh`，作者简介为 `UE5微信技术群加wlxklyh(备注UE5)`。
- 文章收录于专栏 `UF2025(shanghai)`，专栏链接为 `https://zhuanlan.zhihu.com/c_1989523339966973113`。
- 页面结构化数据中显示文章创建时间为 `2026-01-27 18:39:15 +08:00`，更新时间为 `2026-03-31 21:03:08 +08:00`。
- 截取时页面结构化数据中显示约 `81` 赞同、`2` 评论。
- 页面正文包含 `30` 张图片。
- 原文说明源视频为 Bilibili 技术演讲 `[UFSH2025]《洛克王国:世界》移动端管线设计与优化 | 朱谷才 腾讯游戏魔方工作室 客户端开发`，视频链接为 `https://www.bilibili.com/video/BV1XK1YB4EyJ`，时长约 `26分25秒`。
- 原文说明本文由 AI 技术基于视频字幕和关键帧截图整理，并做了技术解析和扩展。

## 文章结构

- `加入 UE5 技术交流群`
- `关于本文`
- `导读`
- `项目背景与技术挑战`
- `第一部分：Stencil 方案 - 角色与场景的像素级区分`
- `第二部分：后处理 Fog 方案 - 与 SSAO 的兼容性优化`
- `第三部分：SubPass 优化 - 单 Pass 多任务的极致优化`
- `第四部分：低配版HDR 管线 - 极致优化的终极方案`
- `实战总结与建议`
- `结语`

## 外部可核验来源

- Epic 官方 Mobile Rendering and Shading Modes 文档说明 UE 移动端有 Forward、Deferred 和 Desktop rendering paths，不同模式在性能和画质上有取舍；Mobile Forward 是移动端默认 shading mode，并具有较快 baseline performance 和较广硬件兼容性。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/mobile-rendering-and-shading-modes-for-unreal-engine?application_version=5.6`。
- Epic 官方 Rendering Settings 文档说明 `Custom Depth-Stencil Pass` 用于给 primitive 打标签以服务 post-processing pass，并可选择启用 stencil。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-settings-in-the-unreal-engine-project-settings?application_version=5.6`。
- Epic 官方 `UPrimitiveComponent::SetCustomDepthStencilValue` API 文档说明该函数设置 `CustomDepth` stencil value，范围为 `0-255`，并标记 render state dirty。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/UPrimitiveComponent/SetCustomDepthStencilValue`。
- Epic 官方 `UMaterial::bEnableStencilTest` API 文档说明 post process material 可以只对通过 Custom Depth/Stencil buffer stencil test 的像素执行。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterial/bEnableStencilTest`。
- Epic 官方 Post Process Effects 文档说明后处理效果用于定义场景整体 look and feel，包括 coloring、tonemapping、lighting 等。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/post-process-effects-in-unreal-engine?application_version=5.6`。
- Khronos Vulkan Subpasses 示例说明 subpasses 可以把一个 render pass 划分为多个逻辑阶段；tile-based renderer 可以利用 on-chip tile memory，减少外部内存带宽。来源：`https://docs.vulkan.org/samples/latest/samples/performance/subpasses/README.html`。
- Khronos Vulkan Mobile Rendering Approaches 文档说明现代移动 GPU 通常采用 tile-based rendering；优化建议包括避免外部 framebuffer read、使用 subpass/input attachment 做 tile-local read，以及正确使用 load/store 操作。来源：`https://github.khronos.org/Vulkan-Site/tutorial/latest/Building_a_Simple_Engine/Mobile_Development/04_rendering_approaches.html`。

## 核心内容

这篇文章围绕《洛克王国：世界》的移动端渲染管线优化展开。项目基于 `Unreal Engine 4.26`，以移动端 Forward Rendering 为基础，同时为了风格化美术、精灵/角色表现、昼夜天气、多人同屏和技能特效做了管线改造。文章把优化拆成四条主线：用 `RGB10A2` 替代传统 Custom Depth/Stencil 的角色场景区分、把 Fog 移到 SSAO 之后、用 FrameBuffer Fetch/SubPass 合并后处理与透明队列工作、为低配设备做更激进的一 pass HDR 管线。

我的判断：这篇的价值不在某一个 shader trick，而在“移动端 tile-based GPU 上如何把美术需求重新折叠回更少的 render pass 和更少的外部带宽写回”。依据是原文四个技术部分都围绕减少额外 pass、减少 draw call、减少显存读写和保留关键视觉效果展开；Khronos 文档也强调 tile memory、subpass 和 load/store 对移动端带宽的重要性。

## 技术点整理

### 1. RGB10A2 Stencil：角色与场景像素级区分

原文首先讨论一个需求：后续 pass 需要判断当前像素属于角色/精灵还是场景，用于压暗场景、突出角色，或在超分锐化时跳过角色。开箱即用方案是 UE 的 Custom Depth/Stencil，但原文指出它在移动端有额外 render pass、屏幕空间贴图带宽，以及角色/精灵多部件骨骼模型导致 draw call 增加的问题。

文章给出的项目方案是把 BasePass 输出格式改为 `RGB10A2`，在角色材质的 Alpha 通道写入标记，后续 pass 采样 BasePass 输出的 Alpha 来区分角色和场景，从而避免额外 Custom Depth pass、额外带宽和额外 draw call。

文章同时指出 `RGB10A2` 是 `UNORM` 格式，直接使用会把 HDR 高亮限制在 `0-1`，影响 Bloom 等高亮效果，因此需要做定点数编码；扩大颜色范围后又会带来低亮度小数精度损失，文章建议通过开根号编码保护接近零的精度。

我的判断：这是典型的“用主颜色 RT 的 spare bits/alpha 通道换掉额外 stencil RT”的移动端思路，适合角色/场景二值或少量类别标记，不适合需要完整 8-bit stencil、多层 mask 或复杂对象分类的场景。依据是原文把它用于角色/场景区分，并明确讨论了 `RGB10A2` 精度和 HDR 表示限制。

### 2. 后处理 Fog：避免 SSAO 压暗雾色

原文说明项目引入 SSAO 是为了补充场景暗部细节，但原移动 Forward 管线的顶点 Fog 与 SSAO 合入顺序冲突：如果 Fog 先算完再合 AO，AO 会错误压暗雾色。项目方案是 BasePass 不计算普通顶点 Fog，而是在 SSAO 合入之后通过场景深度做后处理 Fog。

文章还提到后处理 Fog 在顶点压力较高的场景中，可以减少 BasePass 顶点着色器时间；但并非所有对象都改用后处理 Fog，天空球、Fog 合入之后再渲染的物体、自定义材质 Fog 等仍保留顶点 Fog。

我的判断：这不是简单“后处理一定比顶点 Fog 好”，而是根据 AO/Fog 组合顺序、顶点压力和对象类别做混合策略。依据是原文同时保留天空球和后续物体的顶点 Fog，并把 SSAO 与 Fog 的颜色正确性作为迁移动机。

### 3. SubPass / FrameBuffer Fetch：合并后处理、透明与部分材质

原文第三部分的重点是减少 render pass 切换和中间显存写回。项目原有流程包含颜色解码、SSAO 合入、后处理 Fog、Color Grading、透明物体、Separate Translucency、Bloom、Distortion、Tone Mapping 等工作；其中后处理材质和 Distortion 会打断 pass 合并。

项目方案是利用移动端 FrameBuffer Fetch 获取当前像素颜色，把不需要邻域采样的后处理材质改造成 SubPass 后处理材质；对 Distortion，则把扭曲计算延迟到 Bloom/Tone Mapping 中，减少 pass 中断。

Khronos Vulkan Subpasses 示例与移动端渲染文档都支持这个方向：tile-based renderer 可以把 attachment data 保留在 on-chip tile memory，subpass/input attachment/tile-local read 能减少外部内存读写。

我的判断：这里的关键限制是“同像素本地读取”和“不依赖邻域采样”。只要后处理材质要采样周围像素，比如 blur、某些屏幕空间卷积、邻域边缘检测，就不能直接套这个 SubPass 化方案。依据是原文明确说可优化的后处理材质是不需要周围邻域像素的部分，Khronos 文档也把 local same-pixel read 与 tile-local read 作为优化重点。

### 4. 低配版 HDR 管线：从 PrePass 到 Tone Mapping 尽量留在一个 Render Pass

文章最后提出低配版 HDR 管线：进一步去掉 SSAO、SSR、Bloom 等需要采样邻域或额外 buffer 的效果，保留画面影响大的后处理 Fog、Color Grading、Tone Mapping，并通过 FrameBuffer Fetch 与 Depth Fetch 尽量让计算在一个 render pass / tile memory 内完成。

原文给出的 RenderDoc 对比结果是：去掉 SSAO、SSR、Bloom 后，对比合并 pass 前后，真实帧优化约 `3%`，读带宽优化约 `15%`，写带宽优化约 `30%`。

我的判断：低配版 HDR 的本质是“为低端机定义一条画质保真度可接受、带宽写回更少的降级管线”，不是把全特效管线硬塞进一个 pass。依据是原文明确去掉了 SSAO、SSR、Bloom，同时保留 Fog、Color Grading、Tone Mapping 这些影响整体风格的后处理。

## 实战启发

我的判断：如果要把这篇经验迁移到其他 UE 移动项目，应按下面顺序评估：

1. 先明确美术需求是否真的需要角色/场景像素级 mask，而不是对象级或材质级开关。
2. 如果使用 Custom Depth/Stencil，先量化额外 pass、额外 RT、额外 draw call 和 720P/目标分辨率带宽成本。
3. 若考虑 `RGB10A2` 或 alpha 标记方案，必须先解决 HDR 范围、低亮度精度和 Bloom/Tonemapping 的正确性。
4. 对 Fog、SSAO、Color Grading 等后处理，先画出 pass 顺序和依赖图，再决定哪些可以后移、合并或保留顶点路径。
5. 对 SubPass/FrameBuffer Fetch，先筛出只需要当前像素、不需要邻域采样的材质和效果。
6. 低配管线应作为正式 quality tier，而不是临时开关；需要与美术一起确定哪些效果可以删、哪些风格必须保留。
7. 验证指标应至少包含 GPU 时间、render pass 数、读/写带宽、draw call、发热/降频和目标机实际帧率。

依据：原文每个方案都从具体需求、原方案成本、管线改造、兼容限制和性能收益展开；Khronos 文档也强调移动端 TBR 的带宽、tile memory 和 load/store 操作。

## 适合场景

- UE4/UE5 移动端 Forward Rendering 项目做风格化画面优化。
- 角色/场景需要在后处理中分离处理，且 Custom Depth/Stencil 成本过高。
- 移动端多人同屏、角色/精灵多、draw call 与带宽压力明显。
- 需要为中低端设备做单独渲染质量档位。
- 希望理解 FrameBuffer Fetch、SubPass、tile memory 对移动端管线合并的价值。

## 不适合场景

- 不适合作为通用 UE 渲染管线教程；它是具体项目案例。
- 不适合直接复制到 PC deferred 管线；文章主要针对移动端 Forward 与 tile-based GPU。
- 不适合需要复杂多类别 stencil 或邻域采样后处理的场景直接套用。
- 不适合作为未验证的性能承诺；原文中的耗时和带宽收益来自特定项目、特定设备和特定管线。

## 后续检索关键词

- 中文：`洛克王国 世界 移动端管线优化`、`UE4.26 移动端 Forward Rendering RGB10A2`、`UE 移动端 Custom Depth Stencil 替代方案`、`FrameBuffer Fetch SubPass UE 移动端`、`后处理 Fog SSAO 兼容`、`低配 HDR 管线`
- English: `Unreal Engine mobile forward rendering RGB10A2 stencil`, `UE mobile framebuffer fetch subpass`, `tile based rendering render pass bandwidth`, `mobile HDR pipeline low end`, `Custom Depth Stencil mobile cost`, `post process fog SSAO mobile`

## 来源

- 用户提供材料：知乎文章《UF2025(Shanghai)——移动端渲染管线优化实战：《洛克王国：世界》的技术突破之路》，`https://zhuanlan.zhihu.com/p/1999455131893265237`。
- 原文关联视频：`https://www.bilibili.com/video/BV1XK1YB4EyJ`。
- Epic Developer Community：`Mobile Rendering and Shading Modes for Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/mobile-rendering-and-shading-modes-for-unreal-engine?application_version=5.6`。
- Epic Developer Community：`Rendering Settings in the Unreal Engine Project Settings`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-settings-in-the-unreal-engine-project-settings?application_version=5.6`。
- Epic Developer Community：`UPrimitiveComponent::SetCustomDepthStencilValue`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/UPrimitiveComponent/SetCustomDepthStencilValue`。
- Epic Developer Community：`UMaterial::bEnableStencilTest`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Materials/UMaterial/bEnableStencilTest`。
- Epic Developer Community：`Post Process Effects in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/post-process-effects-in-unreal-engine?application_version=5.6`。
- Khronos Vulkan Samples：`Subpasses`，`https://docs.vulkan.org/samples/latest/samples/performance/subpasses/README.html`。
- Khronos Vulkan Tutorial：`Mobile Development: Rendering Approaches`，`https://github.khronos.org/Vulkan-Site/tutorial/latest/Building_a_Simple_Engine/Mobile_Development/04_rendering_approaches.html`。
