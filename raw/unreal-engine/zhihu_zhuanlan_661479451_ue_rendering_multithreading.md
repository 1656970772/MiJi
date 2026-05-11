---
source_url: "https://zhuanlan.zhihu.com/p/661479451"
type: webpage-summary
title: "Re：从零开始的UE渲染学习-2多线程"
captured_at: 2026-05-11T20:38:24+08:00
author: "稻草人"
topics:
  - Unreal Engine
  - UE5
  - UE5.3
  - Rendering
  - Multithreading
  - Game Thread
  - Render Thread
  - RHI Thread
  - Task Graph
  - ENQUEUE_RENDER_COMMAND
  - FRunnable
  - RHI Command
  - Thread Synchronization
related_source:
  - "https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/2%E5%A4%9A%E7%BA%BF%E7%A8%8B"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/parallel-rendering-overview?application_version=4.27"
  - "https://dev.epicgames.com/documentation/ja-jp/unreal-engine/parallel-rendering-overview?application_version=4.27"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/HAL/FRunnable"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/HAL/FRunnableThread"
capture_scope: "摘要型资料卡；未全文转存"
---

# UE 渲染多线程资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《Re：从零开始的UE渲染学习-2多线程》，链接为 `https://zhuanlan.zhihu.com/p/661479451`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `稻草人`，作者简介为 `路边的菜鸡TA...`。
- 文章收录于专栏 `UE学习笔记`，专栏链接为 `https://zhuanlan.zhihu.com/c_1747449492977352704`。
- 页面结构化数据中显示创建时间与更新时间均为 `2024-03-05 00:26:37 +08:00`。
- 截取时页面结构化数据中显示约 `41` 赞同、`2` 评论。
- 页面话题包括 `渲染`、`多线程`、`UE`。
- 作者说明本文涉及源码版本为 `UE5.3`，只讨论多线程渲染相关内容，并且不深入底层细节。

## 文章结构

- `1.前言`
- `2.多线程简单介绍`
- `2.1线程的分配`
- `2.2任务系统`
- `3.UE的多线程`
- `3.1线程的实现`
- `3.2任务系统的实现`
- `3.3游戏线程和渲染线程的通信`
- `3.4渲染线程和RHI线程通信`
- `3.5资源竞争`
- `3.6线程同步`
- `4.总结`
- `5.参考`

## 作者附带图源

原文给出 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/知乎/Re从零开始的UE渲染学习/2多线程`。

该目录包含 13 张流程图：

- `P1.UE多线程概览.png`
- `P2.任务系统概览.png`
- `P3.UE多线程构建.png`
- `P4.任务系统构建.png`
- `P5.游戏线程和渲染线程通信.png`
- `P6.ENQUEUE_RENDER_COMMAND宏.png`
- `P7.EnqueueUniqueRenderCommand函数.png`
- `P8.FRenderCommand.png`
- `P9.RHI命令.png`
- `P10.命令列表并行生成.png`
- `P11.FScene.BatchAddPrimitives.png`
- `P12.线程同步1.png`
- `P13.线程同步2.png`

我的判断：这篇的核心价值在图和调用链。正文把初学者最容易混在一起的 Game Thread、Render Thread、RHI Thread、Task Graph、RHI Command、Proxy/SceneInfo、Fence 同步分开讲，适合作为读 UE 渲染源码前的线程模型地图。

## 外部可核验来源

- Epic 官方 Parallel Rendering Overview 文档说明：早期 renderer 运行在 Render Thread，Game Thread 会把命令入队，稍后在帧中执行；这些命令会调用 RHI 层，RHI 是面向不同图形 API 的跨平台接口。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/parallel-rendering-overview?application_version=4.27`。
- 同一官方文档说明：为了提升效率，Render Thread 作为 frontend，把平台无关图形命令加入 renderer command list；RHI Thread 在 backend 通过合适的图形 API 转换/执行这些命令。来源同上。
- 同一官方文档还说明：UE 通常配置为 “single frame behind” renderer，即 Render Thread 处理 Frame N 时，Game Thread 处理 Frame N+1；RHI Thread 会进一步让 Render Thread 和 RHI Thread 的同步更复杂。来源同上。
- Epic API 文档提供 `FRunnable` 与 `FRunnableThread` 相关 API 页面，可用于核验原文中线程执行体和线程封装的概念入口。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/HAL/FRunnable`、`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/HAL/FRunnableThread`。

## 核心内容

这篇文章的主线是：UE 渲染不是单线程顺序执行，而是由多个线程和任务系统协同，把游戏状态、渲染逻辑、平台图形 API 命令和 GPU 提交拆开处理。

作者将渲染相关线程拆成三层：

- `Game Thread`：处理游戏逻辑、系统更新和场景状态。
- `Render Thread`：处理渲染逻辑，例如裁剪、渲染管线、光照、后处理，并生成平台无关的 RHI 命令。
- `RHI Thread`：处理 Render Hardware Interface 相关工作，把平台无关命令翻译成对应图形 API 后端命令。

我的判断：这篇是前一篇“开篇预览”的深入补章。上一篇解释为什么 UE 需要 RHI、RDG、Pass；这一篇解释这些渲染工作为什么不能只看单个函数调用，因为很多工作会跨线程排队、延迟执行和同步。

## 概念整理

### 线程分配

原文强调线程不是越多越好，而要看任务并行程度。游戏线程、渲染线程、RHI 线程之间需要减少无意义等待，让各自处理相对独立的工作。

我的判断：读 UE 渲染源码时，不能只问“这个函数在哪里调用”，还要问“这个函数在哪个线程执行”。很多看似直接的函数调用，实际是通过任务系统延后到渲染线程执行。

### 任务系统

原文把任务系统描述为每个线程的任务列表：一个线程想让另一个线程执行工作时，向任务系统添加任务并指定目标线程；目标线程按顺序执行自己的任务列表。

作者提到的核心类包括：

- `TGraphTask`：具体任务类，记录目标线程和任务内容。
- `FTaskGraphImplementation`：任务系统单例，管理线程任务表。
- `FWorkerThread`：线程任务表相关结构。
- `FTaskThreadBase`：任务表/可执行对象，派生出命名线程与非命名线程处理路径。

我的判断：这些类名适合作为源码搜索入口。真正理解任务系统，重点不是背类名，而是看任务如何排队、如何指定目标线程、何时被目标线程取出执行。

### FRunnable 与 FRunnableThread

原文把 `FRunnableThread` 类比为承载执行体的线程封装，把 `FRunnable` 类比为实际在线程上运行的对象。Epic API 文档可作为这两个名字的官方入口。

我的判断：这部分适合帮助读者区分“线程本体”和“线程要跑的工作”。做普通游戏逻辑异步时未必都要直接用 FRunnable，但读渲染线程/RHI 线程构造时，这两个概念很有用。

### Game Thread 到 Render Thread

原文用 `ENQUEUE_RENDER_COMMAND` 解释游戏线程向渲染线程添加任务。宏后面跟着 Lambda，这段 Lambda 就是渲染线程要执行的内容；任务会被加入对应线程队列。

我的判断：这是 UE 渲染源码阅读中非常关键的模式。看到 `ENQUEUE_RENDER_COMMAND` 时，应该马上意识到这不是“当前线程立即执行普通代码”，而是把工作投递到渲染线程的边界。

### Render Thread 到 RHI Thread

原文说明 UE 内部有许多 RHI 命令，并通过命令列表管理和执行，例如 `FRHICommandListImmediate`。当需要 RHI 线程执行转换时，会刷新命令并把工作分配给 RHI 线程和 GPU。

官方 Parallel Rendering 文档也说明，Render Thread 是 frontend，负责入队平台无关图形命令；RHI Thread 是 backend，负责通过图形 API 转换或执行。

我的判断：这一层是理解 UE 渲染性能与平台差异的入口。Render Thread 慢、RHI Thread 慢、GPU 慢，是三种不同问题；在 profiler 中看到瓶颈时要分开看。

### 资源竞争

原文用 `UPrimitiveComponent`、`FPrimitiveSceneProxy`、`FPrimitiveSceneInfo` 解释游戏线程和渲染线程之间的数据隔离：渲染线程维护自己的渲染侧表示，避免直接访问游戏线程对象造成竞争。

我的判断：Proxy 是理解 UE 渲染线程安全的关键词。很多自定义渲染 bug 来自跨线程直接读 UObject 或组件状态，而不是通过渲染侧 proxy/scene data 做同步。

### 线程同步

原文说明当渲染线程太慢时，游戏线程可能跑到后面多帧，造成画面与游戏状态不同步；UE 会通过游戏线程向渲染线程添加栅栏的方式同步，并允许 game thread 比 render thread 快一帧，以稳定帧率。

官方 Parallel Rendering 文档中 “single frame behind” 的描述也能佐证这个思路。

我的判断：这个点对排查卡顿很重要。某一帧 render/RHI 阻塞，会反过来让 game thread 等待同步；于是玩家看到的是整体帧时间尖峰，而不一定只是渲染线程单独慢。

## 适合场景

- 阅读 UE 渲染源码前理解线程模型。
- 排查 `Game`、`Draw`、`RHIT`、`GPU` 之间谁在等待谁。
- 理解 `ENQUEUE_RENDER_COMMAND`、RHI command list、render proxy、thread fence 的作用。
- 准备做自定义渲染 pass、渲染资源更新、组件渲染代理或跨线程数据传递。

## 不适合场景

- 不适合作为 UE 通用并发编程完整教程；它聚焦渲染线程体系。
- 不适合作为 UE5 最新 Task System 的完整 API 指南；文章基于 UE5.3 的阅读理解，且作者明确不深挖底层。
- 不适合作为性能结论来源；要判断实际瓶颈仍需配合 Unreal Insights、stat unit、stat rhi、RenderDoc/PIX 或平台 profiler。

## 后续检索关键词

- 中文：`UE 渲染多线程`、`UE Game Thread Render Thread RHI Thread`、`ENQUEUE_RENDER_COMMAND`、`FTaskGraphImplementation`、`FRunnableThread`、`FPrimitiveSceneProxy`、`UE 渲染线程同步`
- English: `Unreal Engine parallel rendering overview`, `Game Thread Render Thread RHI Thread`, `ENQUEUE_RENDER_COMMAND`, `FRHICommandListImmediate`, `FPrimitiveSceneProxy`, `Unreal single frame behind renderer`

## 来源

- 用户提供材料：知乎专栏文章《Re：从零开始的UE渲染学习-2多线程》，`https://zhuanlan.zhihu.com/p/661479451`。
- 作者 GitHub 高清图目录：`https://github.com/Straw1997/Document/tree/main/%E7%9F%A5%E4%B9%8E/Re%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84UE%E6%B8%B2%E6%9F%93%E5%AD%A6%E4%B9%A0/2%E5%A4%9A%E7%BA%BF%E7%A8%8B`。
- Epic Developer Community：`Parallel Rendering Overview`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/parallel-rendering-overview?application_version=4.27`。
- Epic Developer Community：`FRunnable`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/HAL/FRunnable`。
- Epic Developer Community：`FRunnableThread`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/HAL/FRunnableThread`。
