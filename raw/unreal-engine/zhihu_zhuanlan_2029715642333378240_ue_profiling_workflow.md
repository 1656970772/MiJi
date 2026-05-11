---
source_url: "https://zhuanlan.zhihu.com/p/2029715642333378240"
type: webpage-summary
title: "UF2025(Orlando)——古墓丽影开发商Crystal Dynamic技术美术的 UE 性能剖析工作流"
captured_at: 2026-05-11T17:52:27+08:00
author: "wlxklyh"
column: "UF2025(Orlando)"
topics:
  - UE5
  - UE引擎
  - 性能优化
related_source:
  - "https://www.unrealengine.com/en-US/events/unreal-fest-orlando-2025"
  - "https://youtu.be/mCZRHljYnZA"
capture_scope: "摘要型资料卡；未全文转存"
---

# UF2025(Orlando) UE 性能剖析工作流资料卡

## 来源事实

- 来源文章：知乎专栏《UF2025(Orlando)——古墓丽影开发商Crystal Dynamic技术美术的 UE 性能剖析工作流》。
- 作者与专栏：作者为 `wlxklyh`，收录于知乎专栏 `UF2025(Orlando)`。
- 发布时间：知乎页面显示发布于 `2026-04-21 00:42・广东`。
- 文章主题标签：`UE5`、`UE引擎`、`性能优化`。
- 原始演讲：文章说明基于 Unreal Fest Orlando 2025 演讲 `Technical Artist's Guide to Profiling in Unreal Engine` 整理；评论区作者补充的视频链接为 `https://youtu.be/mCZRHljYnZA`。
- 官方活动佐证：Unreal Engine 官方 Unreal Fest Orlando 2025 页面列出了 `Technical Artist's Guide to Profiling in Unreal Engine` 议程，时间为 2025-06-05 11:45 AM - 12:45 PM ET。

## 核心问题

这篇资料讨论的不是“引擎性能工具百科”，而是技术美术、偏技术的内容开发者、QA 或制作人如何在 UE 项目中快速判断性能问题来源，并把分析结果转化成内容团队能执行的优化清单。

它的中心问题可以概括为：

- 性能问题出现前，如何用资产准入和数据审计降低后期成本；
- 已经出现掉帧后，如何找到地图中最值得优化的位置；
- 进入具体场景后，如何判断瓶颈属于 Game Thread、Render Thread 还是 GPU；
- 当问题超出 TA 边界时，如何把证据完整交接给对应专家。

## 工作流拆解

### 1. 资产治理：让问题尽量不要进入主干

文章把 `Data Validation` 放在性能治理的第一层。可检查项包括碰撞设置、草体距离裁剪、材质数量、材质层数、网格密度、最大面数和 Map Check。

推荐触发路径是三层：

- 本地手动校验：内容制作者在 Content Browser 中主动 Validate。
- 提交时校验：通过 Perforce Trigger 在 submit 阶段拦截不合格资产，并把结果写入 changelist 描述。
- 夜间全量校验：构建机定期跑全项目资产检查，把失败清单发给负责人。

我的判断：这部分最适合做成项目的“性能门禁”。它不依赖某一次 profiling 的英雄操作，而是把低质量资产进入项目的概率系统性降下来。

### 2. 资产审计：把 Asset Audit 与 Statistics 合成行动表

文章提到两类数据源：

- `Asset Audit`：看单个资产本身的成本，例如面数、材质数量、包围盒尺寸、Nanite 状态等，并可导出 CSV。
- `Statistics`：看资产在真实关卡中的实例数量，也支持导出 CSV。

关键不是导出数据本身，而是把“资产成本 × 使用频次”加工为内容团队能理解的优先级列表。文章提到可用脚本派生指标、按团队拆分表格、用条件色标记超阈值行。

我的判断：如果团队没有现成性能平台，这一步可以先从最朴素的 CSV + Python + 表格条件色开始。它成本低，但能很快暴露“少数资产拖累大量场景”的问题。

### 3. 性能热点图：先回答“看哪里”

文章介绍了内部 `Performance Map` 思路：构建机周期采集性能快照，在地图上标记 POI，并显示 Game Thread、Render Thread、GPU Time 的对比。

它解决的是定位前的焦点问题：大地图里先优化哪里回报最大。文章也提醒不要被 View Mode 或 Debug Mode 这类可视化调试模式误导，因为视觉上的异常不一定对应当前帧瓶颈。

我的判断：热点图适合中大型 UE 项目。小项目可以先用固定巡检点 + 自动截图 + stat 数据记录替代，等问题规模变大后再升级成完整地图工具。

### 4. 可复现测试：锁位置、锁朝向、锁变量

文章强调性能排查必须可复现。标准做法包括：

- 先启动 Unreal Insights，再启动 development build；
- 到目标区域之前先 `trace.stop`，避免生成大量无效 trace；
- 到 POI 后慢慢转相机，因为同一位置不同朝向可能性能差异很大；
- 做 A/B 前锁定位置、朝向、时间、天气、风、湿度、ambient 动画；
- 如果隐藏玩家角色，需要反向确认被隐藏对象本身不是瓶颈。

我的判断：这里最值得迁移的是“固定相机位姿 + 环境变量”的纪律。没有控制变量，后续任何优化收益都很容易变成错觉。

### 5. 第一层瓶颈判断：stat unit 决定下一把工具

文章用 `stat unit` 做第一层诊断：

- `Game` 接近 `Frame`：偏 Game Thread CPU 瓶颈，下一步看 Unreal Insights。
- `Draw` 接近 `Frame`：偏 Render Thread CPU 瓶颈，下一步仍看 Unreal Insights。
- `GPU` 接近 `Frame`：偏 GPU 瓶颈，下一步看 `ProfileGPU`，必要时进入 PIX、Razor 等平台工具。

我的判断：`stat unit` 的价值在于快速分流。它不需要解释完整原因，但能防止一上来就打开错误工具。

### 6. CPU 深挖：Unreal Insights 先看连续低帧

文章建议 CPU 问题进入 Unreal Insights 后，优先从 Frames 面板查看：

- `Hitch`：单帧尖刺，可能涉及加载、GC、流送等，通常更难。
- `Consistent Low FPS`：连续低帧，更容易和内容问题对应，也更适合作为 TA 的切入点。

抓 trace 前建议打开命名事件，关键时刻用 `trace.bookmark` 或 `trace.screenshot` 标记。定位时结合 GPU 轨道空闲、Game Thread 忙碌等现象，判断是否 CPU-bound。

我的判断：TA 最有效的边界不是“完全解释底层调用”，而是找到超预算子系统、可疑内容源、复现步骤和证据截图，然后把问题交给正确团队。

### 7. GPU 深挖：ProfileGPU 顺着最大色块下钻

GPU 方向的基本路径是：

- 开启材质或 Nanite 相关 draw event；
- 必要时关闭异步计算，让时间线更易读；
- 执行 `ProfileGPU`；
- 从最大 pass 或 event 开始逐层展开，直到定位到具体 material、draw 或渲染阶段。

文章案例中提到玻璃材质可能成为累计成本很高的瓶颈。

我的判断：这部分适合形成“GPU 初筛 SOP”。TA 不一定要深入驱动或渲染器内部，但应该能把问题缩小到 pass、material 或某类内容特征。

### 8. 跨团队交接：证据完整比结论绝对正确更重要

文章推荐把性能问题集中到 Slack 频道或同类沟通区，并使用标准工单模板。工单应包含：

- 简洁问题摘要；
- Jira 链接；
- 基础分析；
- 平台与 GPU 型号；
- 精确复现步骤；
- changelist 号；
- trace 帧号或构建内帧号；
- 截图、视频、标注过的 trace 截图；
- trace 文件共享路径。

我的判断：这份清单特别适合直接变成团队模板。性能优化最怕“我这里看到了但别人复现不了”，模板化能减少大量沟通损耗。

## 可迁移清单

- 立刻可做：建立 `stat unit`、`trace.bookmark`、`trace.screenshot`、`slomo` 快捷键配置，并提交到版本库。
- 低成本可做：导出 Asset Audit 与 Statistics CSV，用脚本合并出按团队分发的优化表。
- 中等投入：在 CI 或夜间构建中加入资产校验和性能巡检点。
- 高投入：搭建 Performance Map，把 POI、帧时间和构建版本关联起来。
- 流程建设：建立统一性能工单模板，要求每个问题附复现步骤、平台信息、trace 标记和截图。

## 我的判断

这篇资料最适合归入 `虚幻 / 性能优化 / 技术美术工作流`。它的价值不在某个单独命令，而在把 UE 性能剖析拆成“预防、聚焦、诊断、交接”四段流程。

对于独立开发者，它可以简化为：先用 `stat unit` 分流，再用 Insights 或 ProfileGPU 深挖，最后用固定复现点验证优化。

对于中大型团队，它更像一套性能生产线设计：资产门禁、数据审计、热点地图、标准 trace、工单模板、仪表盘。真正值得学的是“让性能分析从个人经验变成团队协议”。

## 后续检索关键词

- 中文：虚幻 性能剖析、UE 技术美术 Profiling、Unreal Insights 教程、UE stat unit、UE ProfileGPU、UE 资产校验、UE Asset Audit、UE Performance Map
- English：Unreal Engine profiling workflow, Technical Artist profiling, Unreal Insights, stat unit, ProfileGPU, Data Validation plugin, Asset Audit, Performance Map, Unreal Fest Orlando 2025

## 来源

- 知乎原文：https://zhuanlan.zhihu.com/p/2029715642333378240
- Unreal Fest Orlando 2025 官方活动页：https://www.unrealengine.com/en-US/events/unreal-fest-orlando-2025
- 演讲视频链接：https://youtu.be/mCZRHljYnZA
