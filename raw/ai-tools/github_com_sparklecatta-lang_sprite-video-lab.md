---
source_url: "https://github.com/sparklecatta-lang/sprite-video-lab"
type: github-repository-card
title: "Sprite Video Lab：本地 2D Sprite 视频与序列帧处理工具"
captured_at: 2026-06-16T11:18:02+08:00
contributor: "Codex"
license: "MIT"
language: "Python"
topics:
  - Sprite Video Lab
  - 2D Sprite Resource
  - Local Web Tool
  - Video Frame Extraction
  - Chroma Key
  - BiRefNet Matting
  - Luma Alpha
  - CorridorKey
  - Real-ESRGAN anime
  - Transparent WebM
related_source:
  - "https://github.com/sparklecatta-lang/sprite-video-lab"
  - "https://github.com/sparklecatta-lang/sprite-video-lab/blob/main/README.md"
  - "https://github.com/sparklecatta-lang/sprite-video-lab/blob/main/USAGE.zh-CN.md"
  - "https://github.com/sparklecatta-lang/sprite-video-lab/blob/main/AI_MATTING.md"
capture_scope: "摘要型资料卡；源材料来自仓库 README、GitHub 页面与 GitHub API 元数据"
---

# sparklecatta-lang/sprite-video-lab 资料卡：本地 2D Sprite 视频与序列帧处理工具

## 来源事实

- 仓库：`sparklecatta-lang/sprite-video-lab`，URL：<https://github.com/sparklecatta-lang/sprite-video-lab>。
- GitHub API 显示仓库为公开仓库，默认分支为 `main`，主语言为 Python，许可证为 MIT。
- GitHub API 抓取时显示 stars 为 70、forks 为 9，更新时间为 `2026-06-16T02:58:45Z`。
- README 将 Sprite Video Lab 定义为本地网页工具，用来把视频片段、单张图片或已有序列帧整理成干净的 2D Sprite 资源。
- README 说明项目优先服务 Windows 本地工作流，运行时主要依赖 Python、Pillow、ffmpeg，以及原生 HTML / CSS / JavaScript。
- README 记录默认本地服务地址为 `http://127.0.0.1:8894`，实验性线稿清理页为 `/app/line-cleaner-experiment.html`。
- README 的 License 章节链接到 MIT 许可证。

## 核心定位

Sprite Video Lab 解决的是“已有动态素材或生成图如何整理成可用 Sprite 资源”的问题。它不是角色生成工具，也不是游戏运行时动画系统，而是把视频、GIF、单图和序列帧放进一个本地处理流水线：截取、抽帧、抠图、统一画布、预览、二次缩放处理、导出透明帧和透明 WebM。

我的判断：它应放入 `AI 工具 / 2D Sprite 资源处理 / 本地视频帧处理管线` 分支。它和 2D Character Starter、See-through 的关系是相邻环节：2D Character Starter 偏角色起始素材生成，Sprite Video Lab 偏动态图 / 序列帧整理与透明化，See-through 偏单张角色图拆层和 PSD / Live2D 前处理。

## 主要工作流

| 工作流 | README 中的职责 |
| --- | --- |
| 本地素材导入 | 导入本地视频、GIF、单张图片或一次性多图序列帧 |
| 视频区间截取 | 预览视频区间，并支持按帧设置起止位置 |
| 固定间隔抽帧 | 从视频或动图中按固定间隔抽取帧 |
| 背景处理 | 去除纯色背景、绿幕 / 蓝幕背景或 AI 生成背景 |
| 亮部 VFX 保留 | 用 Luma 保留发光、火焰、闪电、粒子等亮部特效 |
| 统一画布 | 支持自动宽度画布或方形落地 / 居中画布 |
| MAGIC 二次处理 | Real-ESRGAN anime x4 超分后输出原尺寸 1/2、1/4、1/8 三档 |
| 导出 | 导出 frames 文件夹和自动时间命名的透明 WebM |

## 抠图与透明化能力

README 列出多种背景处理模式：

- `chroma key`：处理受控纯色背景，适合绿幕、蓝幕、白底、灰底等素材。
- `BiRefNet`：AI 主体抠图，适合非纯色背景或生成图背景。
- `Luma`：基于亮度生成 alpha，适合亮部特效、火焰、闪电、粒子等素材。
- `CorridorKey`：用于绿幕 / 蓝幕边缘精修和前景颜色重建。
- BiRefNet 与 Luma / CorridorKey 可以组合成“补亮部 / 精修边缘”或“收紧抠图”两类策略。

我的判断：这组模式把普通 chroma key、AI 主体抠图、VFX 亮度 alpha 和绿幕边缘重建放到同一个交互式工具里，适合处理 AI 生成动图、技能特效素材、角色动作视频和横版连招条。

## MAGIC 二次处理

README 说明，MAGIC 会对当前选中的处理后帧执行 Real-ESRGAN anime x4 超分，再派生出三套透明 PNG：

- `MAGIC 1/2`
- `MAGIC 1/4`
- `MAGIC 1/8`

MAGIC 支持 `硬` / `软` 两种缩小方式：`硬` 使用 nearest-neighbor 保留像素硬边缘，更适合 Sprite 动画；`软` 使用 BOX 缩小，适合需要更柔和抗锯齿的素材。

## 与现有知识库的连接

- 与 `2D Character Starter`：2D Character Starter 生成绿幕角色起始图、姿势图或单帧动作方向；Sprite Video Lab 可继续整理动态图、序列帧、透明帧和 WebM 导出。
- 与 `Game Character Asset Pipeline`：Sprite Video Lab 位于“素材进入游戏前”的视频 / 帧处理和透明化阶段。
- 与 `See-through`：See-through 偏单张角色插画拆层，Sprite Video Lab 偏视频、GIF 和序列帧处理；二者同属角色视觉资产前处理，但目标产物不同。
- 与 `AI Game DevTools`：它属于 AI 游戏工具景观中的图像、Sprite、动画素材处理工具。
- 与 `Display Layer Interpolation`：Sprite Video Lab 产出的帧序列或透明 WebM 可服务运行时角色 / 特效表现；Display Layer Interpolation 代表 Unity 运行时显示层平滑问题，二者处于资产准备和运行时表现的相邻阶段。

## 我的判断

Sprite Video Lab 的价值在于把“动态图或生成视频素材变成可用游戏 Sprite”这段容易散落在 ffmpeg、PS、抠图模型、超分工具和手工预览里的流程收进一个本地网页工具。对 MiJi 知识库而言，它补上了 2D Character Starter 与 See-through 之间的动态素材整理环节：前者更像起始素材生成，后者更像拆层，Sprite Video Lab 则负责帧、alpha、画布、缩放和导出。

## 后续可搜索关键词

- 中文：Sprite Video Lab、2D Sprite、视频抽帧、序列帧导出、透明 WebM、绿幕抠图、BiRefNet 抠图、Luma 亮度抠图、CorridorKey 边缘精修、Real-ESRGAN anime、MAGIC 二次处理、AI 动图素材
- English：Sprite Video Lab, 2D sprite resource, video frame extraction, transparent WebM, chroma key, BiRefNet matting, Luma alpha, CorridorKey, Real-ESRGAN anime, sprite animation frames

## Notable Signals For Graph Extraction

- `Sprite Video Lab -> Local Web Sprite Tool`
- `Sprite Video Lab -> Video/GIF/Image/Sequence Import`
- `Sprite Video Lab -> Frame Range Trimming`
- `Sprite Video Lab -> Fixed Interval Frame Sampling`
- `Sprite Video Lab -> Background Removal Pipeline`
- `Background Removal Pipeline -> Chroma Key`
- `Background Removal Pipeline -> BiRefNet AI Matting`
- `Background Removal Pipeline -> Luma Bright VFX Alpha`
- `Background Removal Pipeline -> CorridorKey Edge Refinement`
- `Sprite Video Lab -> MAGIC Sprite Processing`
- `MAGIC Sprite Processing -> Real-ESRGAN anime x4`
- `MAGIC Sprite Processing -> Transparent PNG Scales`
- `Sprite Video Lab -> Transparent WebM Export`
- `Sprite Video Lab -> 2D Character Starter`
- `Sprite Video Lab -> Game Character Asset Pipeline`
- `Sprite Video Lab -> See-through`
