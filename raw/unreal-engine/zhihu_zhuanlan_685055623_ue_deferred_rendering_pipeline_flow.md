---
source_url: "https://zhuanlan.zhihu.com/p/685055623"
type: webpage-summary
title: "Re：从零开始的UE渲染学习-5延迟渲染管线流程"
captured_at: 2026-05-11T20:25:18+08:00
author: "稻草人"
topics:
  - Unreal Engine
  - UE5
  - UE5.3
  - Deferred Rendering
  - Rendering Pipeline
  - FSceneRenderer
  - FDeferredShadingSceneRenderer
  - FMobileSceneRenderer
  - GameViewport
  - RDG
related_source:
  - "https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/5%E5%BB%B6%E8%BF%9F%E6%B8%B2%E6%9F%93%E7%AE%A1%E7%BA%BF%E6%B5%81%E7%A8%8B"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/forward-shading-renderer-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-dependency-graph?application_version=4.27"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/mobile-rendering-and-shading-modes-for-unreal-engine?application_version=5.6"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-settings-in-the-unreal-engine-project-settings?application_version=5.6"
capture_scope: "摘要型资料卡；未全文转存"
---

# UE 延迟渲染管线流程资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《Re：从零开始的UE渲染学习-5延迟渲染管线流程》，链接为 `https://zhuanlan.zhihu.com/p/685055623`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `稻草人`，作者简介为 `路边的菜鸡TA...`。
- 文章收录于专栏 `UE学习笔记`，专栏链接为 `https://zhuanlan.zhihu.com/c_1747449492977352704`。
- 页面结构化数据中显示创建时间与更新时间均为 `2024-03-08 01:23:12 +08:00`。
- 截取时页面结构化数据中显示约 `63` 赞同、`2` 评论。
- 页面话题包括 `UE5`、`虚幻引擎`、`UE`。
- 作者说明本文涉及源码版本为 `UE5.3`，主要用于记录 UE 延迟渲染管线流程，不涉及具体实现。

## 文章结构

- `1.前言`
- `2.Tick()`
- `2.1.GameViewport->Viewport->Draw()`
- `2.2延迟渲染管线流程`
- `3.总结`
- `4.参考`

## 作者附带图源

原文给出 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/知乎/Re从零开始的UE渲染学习/5延迟渲染管线流程`。

该目录包含 4 张流程图：

- `P1.Tick.png`
- `P2.GameViewport_Viewport_Draw.png`
- `P3.UE4延迟渲染管线.jpg`
- `P4.延迟渲染管线流程.png`

我的判断：这篇文章最有价值的内容在图，而不是正文。正文负责说明图的定位：从引擎 Tick 进入视口绘制，再定位到 `FDeferredShadingSceneRenderer` 这条延迟渲染路径。

## 外部可核验来源

- Epic 官方 Forward Shading Renderer 文档说明，Unreal Engine 默认使用 Deferred Renderer，因为它更通用并能访问更多渲染特性；Forward Rendering 则有不同性能与抗锯齿取舍。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/forward-shading-renderer-in-unreal-engine`。
- Epic 官方 Rendering Dependency Graph 文档说明，RDG 是基于图的调度系统，用于对整帧渲染管线做优化；它会先收集 pass，再编译并按依赖顺序执行。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-dependency-graph?application_version=4.27`。
- Epic 官方移动渲染文档说明，UE 为移动端提供 Forward 与 Deferred shading mode，并说明移动 Deferred 适合动态光照、复杂室外环境等场景。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/mobile-rendering-and-shading-modes-for-unreal-engine?application_version=5.6`。
- Epic 官方 Rendering Project Settings 文档中，`Clear Scene` 选项描述为定义 game mode 下 GBuffer 如何清除，并注明只影响 deferred shading。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-settings-in-the-unreal-engine-project-settings?application_version=5.6`。

## 核心内容

作者把一帧渲染相关流程拆成三层：

1. `Tick()`：游戏引擎启动后循环执行，每一次 Tick 代表一帧；这一帧内会处理逻辑、物理、渲染等系统。
2. `GameViewport->Viewport->Draw()`：进入视口绘制，负责创建和设置视口、配置矩阵、创建渲染器并进行渲染，同时处理一些额外绘制内容。
3. `FDeferredShadingSceneRenderer`：作者把它作为本篇关注的延迟渲染管线入口，并说明如果要自定义渲染管线，可以关注 `FSceneRenderer` 及其 `Render()` 路径。

我的判断：这篇不是“延迟渲染原理”文章，而是“UE 源码定位地图”。它适合在阅读 UE Renderer 源码时当索引，帮助快速知道从主循环到 Viewport Draw，再到 Deferred Renderer 的大致调用层级。

## 可迁移的阅读路线

### 先读帧入口

从 `Tick()` 开始看，是为了建立“一帧里哪些系统被推进”的整体感。渲染不是孤立执行，它依赖游戏线程上的世界状态、视图族构建、组件更新、渲染线程命令提交等上游数据。

我的判断：如果直接从 shader 或 pass 名字看起，很容易只看到局部；先从 Tick 路径看，能避免把“渲染管线”误解成纯 GPU pass 列表。

### 再读 Viewport Draw

`GameViewport->Viewport->Draw()` 是从视口进入渲染器的关键路径。它和项目运行方式、窗口、视图矩阵、ViewFamily、Canvas 或调试绘制等内容有关。

我的判断：排查“为什么这一帧没有画出来”“为什么 Editor/PIE/Standalone 表现不同”时，Viewport Draw 一层通常比底层 pass 更早暴露问题。

### 再读 Renderer

作者把 `FDeferredShadingSceneRenderer` 作为核心入口，是因为本篇关心桌面延迟渲染主路径。外部官方文档也说明 UE 默认使用 Deferred Renderer；移动端又有自己的 Forward/Deferred shading mode。

我的判断：做源码阅读时，最好把 `FDeferredShadingSceneRenderer`、`FMobileSceneRenderer`、Forward Shading、Mobile Deferred 分成不同路径，不要把所有 pass 混在一张图里理解。

### 最后读 RDG 与具体 pass

在 UE5 里，很多具体渲染 pass 会进入 RDG 组织。Epic 官方 RDG 文档说明，它会收集一帧的 pass，构建依赖图，再进行编译和依赖排序执行。

我的判断：如果目标是修改或插入渲染 pass，这篇文章只能提供入口地图；真正动手时还需要补读 RDG、RenderGraphBuilder、SceneTextures、GBuffer、Lighting、PostProcess 等资料。

## 适合场景

- 初学 UE 源码渲染流程，需要先知道一帧如何进入渲染。
- 想定位 `Tick -> Viewport Draw -> SceneRenderer -> Deferred Renderer` 这条调用链。
- 想把 UE4 时代延迟渲染管线图和 UE5.3 源码路径做粗粒度对照。
- 想快速判断某个渲染问题属于帧入口、视口绘制、Renderer 选择、RDG pass，还是具体 shader/lighting 阶段。

## 不适合场景

- 不适合作为延迟渲染原理教程；作者也明确说明没有解释延迟渲染如何工作。
- 不适合直接作为 UE5 最新源码精确调用栈；文章基于 UE5.3，并且部分图参考 UE4 时代流程。
- 不适合替代官方 RDG、Renderer、GBuffer、Lighting、PostProcess 文档或源码阅读。

## 后续检索关键词

- 中文：`UE5 延迟渲染管线`、`FDeferredShadingSceneRenderer Render`、`GameViewport Viewport Draw`、`UE5 RDG 渲染管线`、`UE5 GBuffer Lighting Pass`
- English: `Unreal Engine deferred renderer pipeline`, `FDeferredShadingSceneRenderer Render`, `GameViewport Viewport Draw`, `Unreal RDG render graph`, `UE5 GBuffer deferred shading`

## 来源

- 用户提供材料：知乎专栏文章《Re：从零开始的UE渲染学习-5延迟渲染管线流程》，`https://zhuanlan.zhihu.com/p/685055623`。
- 作者 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/5%E5%BB%B6%E8%BF%9F%E6%B8%B2%E6%9F%93%E7%AE%A1%E7%BA%BF%E6%B5%81%E7%A8%8B`。
- Epic Developer Community：`Forward Shading Renderer in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/forward-shading-renderer-in-unreal-engine`。
- Epic Developer Community：`Rendering Dependency Graph`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-dependency-graph?application_version=4.27`。
- Epic Developer Community：`Mobile Rendering and Shading Modes for Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/mobile-rendering-and-shading-modes-for-unreal-engine?application_version=5.6`。
- Epic Developer Community：`Rendering Settings in the Unreal Engine Project Settings`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-settings-in-the-unreal-engine-project-settings?application_version=5.6`。
