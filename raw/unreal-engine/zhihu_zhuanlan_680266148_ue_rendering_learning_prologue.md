---
source_url: "https://zhuanlan.zhihu.com/p/680266148"
type: webpage-summary
title: "Re：从零开始的UE渲染学习-1开篇预览"
captured_at: 2026-05-11T20:34:13+08:00
author: "稻草人"
topics:
  - Unreal Engine
  - UE5
  - UE Rendering
  - Source Reading
  - RHI
  - RDG
  - PSO
  - DDC
  - Pass
  - Render Thread
  - Rendering Pipeline
related_source:
  - "https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/1%E5%BC%80%E7%AF%87%E9%A2%84%E8%A7%88"
  - "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/render-dependency-graph-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-dependency-graph?application_version=4.27"
  - "https://dev.epicgames.com/documentation/ja-jp/unreal-engine/graphics-programming-overview?application_version=4.27"
  - "https://dev.epicgames.com/documentation/ja-jp/unreal-engine/parallel-rendering-overview?application_version=4.27"
  - "https://learn.microsoft.com/en-us/windows/win32/direct3d12/managing-graphics-pipeline-state-in-direct3d-12"
capture_scope: "摘要型资料卡；未全文转存"
---

# UE 渲染学习开篇预览资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《Re：从零开始的UE渲染学习-1开篇预览》，链接为 `https://zhuanlan.zhihu.com/p/680266148`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `稻草人`，作者简介为 `路边的菜鸡TA...`。
- 文章收录于专栏 `UE学习笔记`，专栏链接为 `https://zhuanlan.zhihu.com/c_1747449492977352704`。
- 页面结构化数据中显示创建时间为 `2024-03-04 00:39:36 +08:00`，更新时间为 `2024-03-04 00:44:28 +08:00`。
- 截取时页面结构化数据中显示约 `101` 赞同、`3` 评论。
- 页面话题包括 `渲染`、`源码`、`UE5`。
- 作者说明该系列是自己的 UE 渲染学习笔记，定位为 UE 渲染模块的基础、通用、粗略导读，不包含渲染基础教学。

## 文章结构

- `1.前言`
- `2.渲染的开始`
- `2.1真正的开始`
- `2.2RHI`
- `2.3多线程`
- `2.4RDG`
- `3.其他概念解释`
- `3.1PSO`
- `3.2DDC`
- `3.3Pass`
- `4.总结`

## 作者附带图源

原文给出 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/知乎/Re从零开始的UE渲染学习/1开篇预览`。

该目录包含 4 张流程图：

- `P1.渲染基础网格流程.png`
- `P2.渲染基础网格流程.png`
- `P3.渲染基础网格流程.png`
- `P4.渲染基础网格流程.png`

我的判断：这篇文章是同一系列后续源码阅读文章的入口图。它先从“直接调用图形 API 画一个网格”讲起，再逐层引出 UE 对底层图形 API 的抽象、线程拆分、RDG 组织方式和常见术语。

## 外部可核验来源

- Epic 官方 RDG 文档说明，Rendering Dependency Graph 也称 RDG 或 Render Graph，是面向整帧渲染管线优化的图式调度系统；它会先收集 pass，再编译并按依赖顺序执行。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-dependency-graph?application_version=4.27`。
- Epic 官方图形编程概览文档中包含 `Render Hardware Interface (RHI)` 章节，用于解释 UE 渲染代码中的 RHI 层。来源：`https://dev.epicgames.com/documentation/ja-jp/unreal-engine/graphics-programming-overview?application_version=4.27`。
- Epic 官方 Parallel Rendering 概览说明，渲染命令会调用 RHI 层，RHI 在支持平台上作为不同图形 API 的跨平台接口。来源：`https://dev.epicgames.com/documentation/ja-jp/unreal-engine/parallel-rendering-overview?application_version=4.27`。
- Microsoft Learn 的 Direct3D 12 PSO 文档说明，图形管线状态包括光栅化、混合、深度模板、图元拓扑、shader 等设置；Direct3D 12 中大部分图形管线状态通过 Pipeline State Object 设置。来源：`https://learn.microsoft.com/en-us/windows/win32/direct3d12/managing-graphics-pipeline-state-in-direct3d-12`。

## 核心内容

这篇文章的核心是给 UE 渲染学习建立分层地图：

1. 从最原始的“画一个网格”开始理解：顶点/索引缓冲、shader、材质参数、渲染状态、draw call、驱动和 GPU。
2. 用 `RHI` 解释 UE 如何把平台差异和图形 API 差异封装起来。
3. 用多线程解释 Game Thread、Render Thread、RHI Thread 之间的大致职责分工。
4. 用 `RDG` 解释 UE 如何组织复杂 pass、资源依赖、生命周期和调度。
5. 用 `PSO`、`DDC`、`Pass` 解释阅读 UE 渲染源码时经常遇到的基础名词。

我的判断：它不是渲染入门课，也不是 UE 渲染底层实现剖析；更像一个“源码阅读前的地图和词汇表”。读完之后，读者会知道自己后续查源码时大概要往 RHI、RDG、Renderer、Pass、PSO、DDC 哪些方向找。

## 概念整理

### 直接绘制网格

作者先用直接调用 DX/OpenGL 这类图形 API 绘制网格作为起点：设置网格数据、shader、常量参数和渲染状态，再提交绘制命令。这个模型虽然简化，但适合帮助初学者理解所有上层渲染框架最后都会落到 GPU 可执行的命令与资源状态上。

我的判断：这是学习 UE 渲染最好的入口之一。不要一上来就读 Lumen、Nanite 或后处理；先把一笔 draw 需要什么资源和状态弄清楚，后面看 RHI、PSO、RenderDoc、RDG 才不会散。

### RHI

原文把 RHI 解释为 UE 的 Render Hardware Interface，用来把通用渲染命令和资源转换到目标平台/图形 API。Epic 官方图形编程与并行渲染文档也能佐证 RHI 是 UE 渲染代码中的重要抽象层，并承担跨图形 API 接口的角色。

我的判断：RHI 是“平台差异隔离层”，不是渲染算法本身。阅读 UE 渲染源码时，看到 RHI 调用应理解为进入更底层的资源创建、状态设置、命令提交和平台后端转换。

### 多线程

原文用 Game Thread、Render Thread、RHI Thread 的关系解释 UE 渲染为什么复杂：游戏线程处理一帧的游戏逻辑和世界状态，渲染线程组织场景渲染与渲染逻辑，RHI 线程再负责更底层的资源与命令执行。

我的判断：UE 渲染源码难读，很大一部分不是因为某个算法难，而是因为数据跨线程、跨帧、跨资源生命周期流动。读源码时要时刻问：这段逻辑在哪个线程运行，数据从哪里来，何时同步到渲染侧。

### RDG

原文引用 UE 官方文档解释 RDG：它把要编译和执行的渲染命令记录到图结构中，并负责资源依赖、生命周期、异步计算、并行记录、未使用 pass 剔除和验证等工作。Epic 官方英文文档也把 RDG 定位为整帧渲染管线优化的图式调度系统。

我的判断：RDG 是 UE5 渲染学习的关键门槛。想添加自定义渲染逻辑时，不能只知道“我要画一张 RT”，还要知道这个 pass 读写哪些资源、何时执行、生命周期如何交给 RDG 管理。

### PSO

原文把 PSO 理解为提前整理好的管线状态对象。Microsoft Direct3D 12 文档佐证：PSO 封装了 shader bytecode、输入顶点格式、图元拓扑类型、混合、光栅化、深度模板、RT 格式、多重采样、root signature 等状态中的大部分。

我的判断：PSO 适合理解为“渲染状态组合包”。在 UE 里排查 shader、材质、管线状态、平台编译和性能时，PSO 相关问题经常会出现；它不是只属于底层图形 API 的概念。

### DDC

原文把 DDC 解释为 Derived Data Cache，用来缓存纹理、模型、动画、已编译 shader 等派生数据，以减少重复处理成本，并提到团队开发中共享 DDC 可加速开发效率。

我的判断：DDC 是 UE 工程体验和团队效率里的基础设施。很多“第一次打开项目很慢”“shader 编译很久”“换机器重新生成资源”的问题，背后都和 DDC、缓存命中率、共享缓存配置有关。

### Pass

原文提醒：UE 语境里的 Pass 常常指渲染阶段，例如深度、ShadowMap、不透明、半透明、后处理等；这和 Unity 初学者常说的“同一网格多 pass 绘制”不完全等价。

我的判断：这个区分很实用。搜索“UE 添加 Pass”时，如果心里想的是“给一个 mesh 多画一遍”，但 UE 文档/源码讲的是“添加一个渲染阶段”，很容易搜错方向。

## 适合场景

- 作为 UE 渲染源码学习系列的第一篇入口。
- 给已有图形学基础但不熟 UE 架构的人建立 RHI、RDG、PSO、DDC、Pass 的概念索引。
- 在准备修改 UE 渲染管线、添加自定义 pass、阅读 Renderer 源码前做预热。
- 帮 Unity SRP 背景的开发者理解 UE 里“Pass”“管线”“源码修改”的语境差异。

## 不适合场景

- 不适合作为渲染基础教程；原文也说明不涉及渲染基础。
- 不适合作为 RHI、RDG、PSO 的底层实现细节文档；作者明确更关注应用侧和修改管线时的理解路径。
- 不适合作为 UE5 最新版本精确源码调用栈；它是学习导读，仍需结合当前 UE 版本源码验证。

## 后续检索关键词

- 中文：`UE5 渲染学习 RHI RDG`、`UE Render Hardware Interface`、`UE RDG Pass`、`UE PSO DDC`、`UE 添加自定义 Pass`、`UE 渲染线程 RHI线程`
- English: `Unreal Engine RHI RDG overview`, `Unreal Render Dependency Graph pass`, `Unreal Pipeline State Object`, `Unreal Derived Data Cache shader`, `Unreal render thread RHI thread`

## 来源

- 用户提供材料：知乎专栏文章《Re：从零开始的UE渲染学习-1开篇预览》，`https://zhuanlan.zhihu.com/p/680266148`。
- 作者 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/1%E5%BC%80%E7%AF%87%E9%A2%84%E8%A7%88`。
- Epic Developer Community：`Rendering Dependency Graph`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/rendering-dependency-graph?application_version=4.27`。
- Epic Developer Community：`Graphics Programming Overview`，`https://dev.epicgames.com/documentation/ja-jp/unreal-engine/graphics-programming-overview?application_version=4.27`。
- Epic Developer Community：`Parallel Rendering Overview`，`https://dev.epicgames.com/documentation/ja-jp/unreal-engine/parallel-rendering-overview?application_version=4.27`。
- Microsoft Learn：`Managing Graphics Pipeline State in Direct3D 12`，`https://learn.microsoft.com/en-us/windows/win32/direct3d12/managing-graphics-pipeline-state-in-direct3d-12`。
