---
source_url: "https://zhuanlan.zhihu.com/p/706117237"
type: webpage-summary
title: "【笔记】UE5 Shader 调试工具 - Microsoft PIX"
captured_at: 2026-05-11T20:11:27+08:00
author: "徐兮子美"
topics:
  - Unreal Engine
  - UE5
  - Shader Debugging
  - Microsoft PIX
  - DirectX 12
  - GPU Capture
  - RenderDoc
  - DXIL
  - Pipeline State
  - Debug Pixel
related_source:
  - "https://learn.microsoft.com/en-us/windows/win32/direct3dtools/pix/articles/general/pix-overview"
  - "https://devblogs.microsoft.com/pix/gpu-captures/"
  - "https://www.microsoft.com/en-us/download/details.aspx?id=106345"
  - "https://learn.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development"
  - "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/shader-debugging-workflows-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine?application_version=5.6"
capture_scope: "摘要型资料卡；未全文转存"
---

# UE5 Shader 调试工具 Microsoft PIX 资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《【笔记】UE5 Shader 调试工具 - Microsoft PIX》，链接为 `https://zhuanlan.zhihu.com/p/706117237`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `徐兮子美`，作者简介为 `很努力`。
- 文章收录于专栏 `开发技巧`，专栏链接为 `https://zhuanlan.zhihu.com/c_1370799232513830912`。
- 文章结构化数据中显示创建时间为 `2024-06-29 14:17:19 +08:00`，更新时间为 `2025-04-26 22:08:15 +08:00`。
- 截取时页面结构化数据中显示约 `144` 赞同、`1` 评论。
- 页面话题包括 `虚幻引擎`、`游戏开发`、`游戏引擎`。
- 文章问题背景：作者认为 RenderDoc 在调试 DX12 shader 时可能遇到渲染指令丢失或无法 replay、shader 符号表丢失导致不能 debug 源码、渲染时间 duration 不精确等问题，于是尝试 Microsoft PIX。
- 文章核心流程：UE5 项目切到 DX12，关闭 RenderDoc 插件，确认 PIX 插件开启，配置 shader 调试相关 cvar，Attach 到 Unreal 进程，取 GPU Capture，开启 Analysis，再在 PIX 中查看 Pipeline、Shader 资源、指令和 Debug Pixel。

## 外部可核验来源

- Microsoft Learn 的 PIX 概览说明：PIX 是面向使用 Direct3D 12 的游戏开发者的调试与性能分析工具，可用 GPU Captures 调试渲染问题和分析帧性能，也可用 Timing Captures 做传统 profiling；PIX 的 GPU 能力适用于 Direct3D 12 应用，也包括经 Direct3D 11 on 12 的 D3D11。
- Microsoft PIX 官方 GPU Captures 文档说明：GPU Capture 会记录游戏发出的 Direct3D 12 API 调用及其参数，之后可 replay，从而启用调试与分析功能；首次加载捕获后需要启动 Analysis 才能让功能完整可用。
- Microsoft PIX 官方 GPU Captures 文档还说明：Collect Timing Data 会多次 replay 捕获的 API 调用并测量 GPU 执行时间，结果会平均以降低噪声；这可佐证文章中“开启收集 TimingData”的入口意义。
- Microsoft Download Center 页面提供 `PIX on Windows 2408.09`，支持 Windows 10/11，并提供 x64 与 ARM64 安装包。
- Microsoft Learn 的 Windows Developer Mode 文档说明：Developer Mode 位于 Windows 设置中，面向构建、部署和测试应用；启用需要管理员权限。这可佐证文章中 Analysis 相关流程要求开启开发者模式的系统侧前置条件。
- Unreal 官方全局 Shader 文档说明：可在 `ConsoleVariables.ini` 中设置 `r.ShaderDevelopmentMode=1` 以获取 shader 编译详细日志。
- Unreal 官方 Shader Debugging Workflows 文档说明：UE5 中 `r.Shaders.KeepDebugInfo` 被拆分为 `r.Shaders.Symbols` 与 `r.Shaders.ExtraData`；其中 `r.Shaders.Symbols` 用于生成并写出 shader 调试符号，`r.Shaders.ExtraData` 用于生成 shader 名称和额外数据。

## 核心问题

UE5 在 DX12 渲染路径下调试 shader，常见痛点不是“能不能截图”，而是：

- 抓到的帧能否稳定 replay。
- 是否能看到 shader 源码、符号、资源绑定和管线状态。
- 是否能定位具体 draw call、像素、PS/VS 代码路径。
- 是否能获得更可靠的 GPU timing 数据。

文章的价值在于给出一条面向 UE5/DX12 的 PIX 实操路径：引擎侧先打开 shader 调试信息和 PIX attach，再用 PIX 捕获并分析帧，最后在 Pipeline 与 Debug Pixel 中定位 shader 问题。

我的判断：这篇不是完整 PIX 教程，而是一张“UE5 项目接入 PIX 的踩坑清单”。它最适合放进渲染调试工具链资料库，作为 RenderDoc 遇到 DX12 shader 调试问题时的替代路线。

依据：知乎原文明确列出 RenderDoc 在 DX12 调试中的三个痛点；Microsoft Learn 也把 PIX 定位为 Direct3D 12 游戏调试与 profiling 工具。

## UE5 侧准备

文章给出的引擎侧前置条件：

1. 确保项目 `Default RHI` 为 DX12。
2. 关闭 RenderDoc 插件。
3. 确保 UE 的 PIX 插件开启。
4. 修改 `ConsoleVariables.ini`，写入 shader 调试相关 cvar。
5. 重启引擎，看到 PIX 相关提示后，用 `PrintScreen` 或自定义快捷键截帧。

文章给出的 cvar：

```ini
r.ShaderDevelopmentMode=1
r.Shaders.Optimize=0
r.Shaders.Symbols=1
r.Shaders.ExtraData=1
r.D3D12.AutoAttachPIX=1
```

我的判断：这组配置的目的可以拆成三层：`r.ShaderDevelopmentMode` 帮助 shader 编译调试；`r.Shaders.Symbols` 和 `r.Shaders.ExtraData` 让工具能关联 shader 符号、名称和额外数据；`r.Shaders.Optimize=0` 牺牲编译优化换取更易读的调试信息；`r.D3D12.AutoAttachPIX=1` 用于让 PIX 更顺利接入 D3D12 流程。

注意：这些设置适合开发调试环境，不应直接作为发行构建默认配置。关闭优化和写出调试信息可能改变编译输出、增加磁盘/编译成本，并影响性能观测。

## PIX 捕获流程

文章中的操作流程：

1. 打开 PIX，找到 Unreal 进程并 Attach。
2. 在 Unreal 中按 `PrintScreen` 或 `F12` 取帧。
3. 如果快捷键和系统冲突，在 PIX 中改额外 capture 快捷键。
4. 双击进入捕获文件后，启动 Analysis。
5. 若 Analysis 要求系统权限或开发设置，按 Windows 10/11 的 Developer Mode 路径开启。
6. 在打包游戏中可以边玩边截帧；文章提到 UnrealEditor 在 Analysis 播放时可能关闭进程。

Microsoft PIX 文档支持这里的关键点：GPU Capture 需要 replay 才能启用分析功能；首次加载捕获后需要 Start Analysis；GPU Capture 记录的是 Direct3D 12 API 调用与参数。

我的判断：如果目标是稳定调 shader，优先用 Development/DebugGame 打包包做 capture，通常比直接对 Editor 取帧更干净。Editor 进程里有大量编辑器 UI、工具窗口、额外 RHI 调用和插件状态，噪声更高。

## 常用入口

文章列出的 PIX 侧入口：

- `Collect Timing Data`：收集 GPU timing。
- Event/DrawCall 列表：定位需要 Debug 的 draw call。
- `Pipeline` 页签：查看具体 Shader 资源、指令和管线状态。
- 选中像素后 `Debug Pixel`：定位某个像素的 shader 执行。
- Compile Options 中可观察或设置 `/Od /Zi` 一类调试相关编译 flag。

我的判断：PIX 调 shader 时，最好按“DrawCall -> Pipeline State -> Shader -> Pixel/Vertex Debug”的顺序走。先确认自己看的是正确 pass 和 draw，再进入 shader 代码，否则很容易在后处理、阴影、Nanite/Lumen 等复杂 pass 里迷路。

## DX12 Pipeline 名词

文章对 DX12 Pipeline 的几个入口做了简述：

- `Pipeline State`：当前渲染管线参数和状态。
- `Root Signature`：shader 与 Pipeline State Object 之间的资源绑定接口，指定常量缓冲、纹理、采样器等资源如何被访问。
- `IA`：Input Assembler，读取顶点/索引并组装图元。
- `VS`：Vertex Shader。
- `PS`：Pixel Shader。
- `OM`：Output Merger，负责把 pixel shader 输出合并到 render target，包含 blending、depth/stencil test 等操作。

我的判断：对 UE 渲染调试来说，`Root Signature` 和资源绑定尤其关键。很多 shader bug 不在 HLSL 算法本身，而在“这个 draw call 实际绑定了哪个 SRV/UAV/CBV、哪个 uniform buffer、哪个 texture mip、哪个 sampler”。

## 与 RenderDoc 的关系

文章的出发点是 RenderDoc 在 DX12 shader 调试中遇到限制，于是改用 PIX。我的判断是：

- RenderDoc 仍然很适合跨 API 的帧分析，尤其 Vulkan、D3D11、OpenGL 等场景。
- PIX 在 Windows + D3D12 + Xbox/PC 游戏 profiling 语境下更贴近 Microsoft 官方工具链。
- UE5 DX12 shader 源码级调试、DXIL、Timing、Pipeline 状态分析，PIX 往往更顺手。
- 两者插件都可能介入 graphics API capture/loader 流程。文章提到需要关闭 RenderDoc 插件以避免 `dxgi.dll` 冲突，这一点可作为项目内实操经验记录，但具体是否复现取决于 UE 版本、插件状态、PIX/RenderDoc 版本和启动方式。

## 适合场景

- UE5 项目使用 DX12 RHI，并需要 shader 源码级调试。
- RenderDoc 捕获 DX12 帧时 replay 不稳定或 shader 符号缺失。
- 需要定位某个 draw call、某个像素的 shader 执行路径。
- 需要查看 PSO、Root Signature、资源绑定、Pipeline 阶段状态。
- 需要 GPU capture timing 与 GPU counter 辅助分析渲染性能。

## 不适合场景

- 项目主渲染 API 是 Vulkan 或非 Windows 平台。
- 只想快速看一帧资源和 draw call，不需要 DX12 shader debug；RenderDoc 可能更轻。
- 发行构建或性能基准环境，不适合长期保留 `r.Shaders.Optimize=0` 等调试配置。
- 需要解决 D3D12 API 使用错误时，PIX 不是唯一工具；Microsoft Learn 明确建议 D3D12 API 层问题使用 D3D12 Debug Layer 和 GPU-Based Validation。

## 可迁移清单

- 单独准备 `Config/ConsoleVariables.ini` 的 PIX 调试配置，不要和普通开发配置混在一起。
- 捕获前确认 `Default RHI = DX12`、PIX 插件启用、RenderDoc 插件关闭。
- 首次调试先用小场景或固定 camera 复现问题，降低 capture 噪声。
- 使用打包 Development/DebugGame 版本复现，减少 Editor 噪声。
- 把 PIX capture 快捷键改成项目内不冲突的组合。
- 抓帧后先 Start Analysis，再看 Timing、Pipeline、Debug Pixel。
- 保存 capture 时同时记录 UE 版本、显卡、驱动、PIX 版本、RHI、cvar 配置和复现步骤。
- 对 shader 符号路径、DDC、Cook 配置做项目内文档，否则换机器后很容易“能抓帧但不能看源码”。

## 我的判断

这篇资料适合归入 `Unreal Engine / 渲染调试 / Shader Debugging / Microsoft PIX / DX12`。

它的核心价值是把 UE5 + PIX 的最小可用路径写清楚：DX12 RHI、关闭 RenderDoc、开启 PIX、打开 shader debug cvar、Attach、Capture、Analysis、Pipeline、Debug Pixel。对做 UE5 渲染、材质、后处理、Nanite/Lumen 相关调试的人来说，这是一个很实用的起步 checklist。

边界也要明确：PIX 不是 RenderDoc 的全量替代；它更适合 Windows/D3D12 语境。文章中的 `r.Shaders.Optimize=0`、符号和 ExtraData 配置属于调试配置，应按需启用、用完关闭。

## 后续检索关键词

- 中文：`UE5 PIX Shader 调试`、`UE5 DX12 Debug Pixel`、`r.Shaders.Symbols`、`r.Shaders.ExtraData`、`r.D3D12.AutoAttachPIX`、`PIX GPU Capture Unreal`
- English: `Unreal Engine PIX shader debugging`, `PIX GPU Capture UE5`, `DXIL shader debugging PIX`, `Debug Pixel PIX`, `Unreal r.Shaders.Symbols ExtraData`

## 来源

- 用户提供材料：知乎专栏《【笔记】UE5 Shader 调试工具 - Microsoft PIX》，`https://zhuanlan.zhihu.com/p/706117237`。
- Microsoft Learn：`Get started with PIX`，`https://learn.microsoft.com/en-us/windows/win32/direct3dtools/pix/articles/general/pix-overview`。
- Microsoft PIX 官方博客：`GPU Captures`，`https://devblogs.microsoft.com/pix/gpu-captures/`。
- Microsoft Download Center：`PIX on Windows 2408.09`，`https://www.microsoft.com/en-us/download/details.aspx?id=106345`。
- Microsoft Learn：`Settings for developers / Developer Mode`，`https://learn.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development`。
- Unreal Engine 官方文档：`Shader Debugging Workflows`，`https://dev.epicgames.com/documentation/zh-cn/unreal-engine/shader-debugging-workflows-unreal-engine`。
- Unreal Engine 官方文档：`Adding Global Shaders to Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/adding-global-shaders-to-unreal-engine?application_version=5.6`。
