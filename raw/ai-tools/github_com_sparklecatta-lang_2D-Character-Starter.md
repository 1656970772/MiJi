---
source_url: "https://github.com/sparklecatta-lang/2D-Character-Starter"
type: github-repository-card
title: "2D Character Starter：Codex 2D 可游玩角色图像生成与修正 Skill"
captured_at: 2026-06-16T11:06:01+08:00
contributor: "Codex"
license: "MIT"
language: null
topics:
  - 2D Character Starter
  - Codex Skill
  - 2D Playable Character
  - Chroma Green Screen
  - Character Simplification
  - Concept Target Style Transfer
  - Strict Pose Transfer
  - Single-frame Action
  - Concept Simplify
  - Subagent Sampling
related_source:
  - "https://github.com/sparklecatta-lang/2D-Character-Starter"
  - "https://github.com/sparklecatta-lang/2D-Character-Starter/blob/main/README.md"
  - "https://github.com/sparklecatta-lang/2D-Character-Starter/blob/main/SKILL.md"
capture_scope: "摘要型资料卡；源材料来自仓库 README、SKILL.md 与 GitHub API 元数据"
---

# sparklecatta-lang/2D-Character-Starter 资料卡：Codex 2D 可游玩角色图像生成与修正 Skill

## 来源事实

- 仓库：`sparklecatta-lang/2D-Character-Starter`，URL：<https://github.com/sparklecatta-lang/2D-Character-Starter>。
- GitHub API 显示仓库为公开仓库，默认分支为 `main`；仓库描述字段、主语言字段和仓库级 license 字段为空。
- GitHub API 抓取时显示 stars 为 3、forks 为 0，更新时间为 `2026-06-16T02:59:47Z`。
- README 将 `2D Character Starter` 定义为给 Codex 使用的 `2D 可游玩角色图像生成/修正 skill`。
- README 明确说明目标不是制作最终序列帧，而是快速获得便于继续测试、筛选和迭代的角色起始素材。
- README 推荐引用方式为 `$2DCS`，也兼容 `$2dcs`。
- README 说明所有产图模式默认输出 `3:2` 画幅、纯绿幕背景、单角色优先、不在图片里写文字标签，并按文件名和回复顺序排序。
- README 的 License 章节写明 `MIT License`。

## 核心定位

2D Character Starter 不是通用绘图提示词集合，而是一个面向 Codex 工作流的角色资产起步工具。它把常见的角色图像迭代需求拆成显式模式：简化、画风迁移、姿势迁移、动作方向探索和待机角色复杂度控制。

我的判断：它最适合放在 `AI 工具 / Codex Skill / 2D 角色资产起步流程` 分支。它和 See-through 的关系是前后阶段互补：See-through 偏“已有角色图拆层与 PSD / Live2D 前处理”，2D Character Starter 偏“从概念图或参考图快速生成可继续迭代的绿幕角色起始素材”。

## 主要模式

| 模式 | README / SKILL.md 中的职责 |
| --- | --- |
| `s` / `simplify` | 一张 2D 角色图生成轻度、中度、重度三档简化结果 |
| `ct` / concept target | 图 1 锁定角色设定，图 2 提供目标画风，输出一张概念图转目标画风结果 |
| `p` / pose | 图 1 锁定外观，后续图片作为独立姿势参考，每个姿势输出一张严格姿势迁移结果 |
| `sq` / sequence | 一张孤立单角色图生成 walk、run、jump、attack、dash 五张单帧动作 |
| `cs` / concept simplify | 一张设定图生成 3:2 绿幕、右朝向、站立待机角色的低/中/高复杂度版本 |
| `h` / help | 只显示用法，不启动图片处理或 subagent |

## 输出约束

- 所有产图模式默认使用 `3:2` 画幅和纯 chroma green screen 背景。
- 输出优先是单角色，不生成文字标签。
- 多图结果必须依赖文件名和回复顺序保持语义顺序。
- `p` 和 `sq` 不承诺像素级可无缝播放的动画序列；`sq` 用于快速探索动作方向，`p` 用于同一角色套用姿势。
- `s` 的轻度、中度、重度简化要求独立生成或编辑，不能用本地减色、海报化、边缘滤镜伪造。

## 数字后缀与 subagent 采样

README 与 SKILL.md 都说明，除 `h/help` 外，产图模式可以追加数字后缀，例如 `$2DCS s 5`、`$2DCS ct 5`、`$2DCS p 5`、`$2DCS sq 5`、`$2DCS cs 5`。这个数字不是强度、seed、复杂度等级、动作帧数量，而是让多个 subagent 用相同输入和相同模式独立运行，得到更多可选结果。

我的判断：这让它比普通单次生图提示词更接近“可编排的 Codex 资产生产 skill”。它强调模式显式、输入顺序、输出命名、绿幕背景和多 agent 独立采样，适合沉淀为游戏角色资产探索阶段的标准入口。

## 与现有知识库的连接

- 与 `Game Character Asset Pipeline`：2D Character Starter 解决角色资产起始图、绿幕待机图和单帧动作方向，适合进入角色资产生产线的早期探索环节。
- 与 `See-through`：两者都服务角色资产链路，但前者偏生成/修正起始素材，后者偏单图拆层、遮挡补全和 PSD / Live2D 前处理。
- 与 `AI Game DevTools`：它属于 AI 游戏工具景观中的图像、角色、动画素材探索工具。
- 与 `Game Studio Sub-Agents`：它的数字后缀支持多 subagent 独立采样，和游戏生产中的多 agent 组织方式存在工作流层面的桥接。
- 与 `PlotPilot` 不同：PlotPilot 解决叙事状态和长篇写作管线，2D Character Starter 解决角色视觉资产起步，不属于叙事引擎。

## 我的判断

2D Character Starter 的价值在于把“让 AI 帮我改角色图”这类模糊需求约束成可复用的模式协议。对游戏项目而言，它适合用于角色概念图进入可测试素材之前的快速探索：清理线条复杂度、测试目标画风、套姿势、生成动作方向单帧，以及生成绿幕待机角色。后续如果要进入可动画 PSD 或 Live2D，仍需要接 See-through 或人工拆层 / rigging 流程。

## 后续可搜索关键词

- 中文：2D Character Starter、2DCS、Codex Skill、2D 可游玩角色、绿幕角色、角色简化、概念图转画风、姿势迁移、单帧动作、待机角色、subagent 采样
- English：2D Character Starter, 2DCS, Codex skill, 2D playable character, chroma green screen, character simplification, concept target style transfer, strict pose transfer, single-frame action, concept simplify, subagent sampling

## Notable Signals For Graph Extraction

- `2D Character Starter -> Codex Skill Workflow`
- `2D Character Starter -> 2D Playable Character Image Generation`
- `2D Character Starter -> Chroma Green Screen Output`
- `2D Character Starter -> Character Simplification Mode`
- `2D Character Starter -> Concept Target Style Transfer`
- `2D Character Starter -> Strict Pose Transfer`
- `2D Character Starter -> Single-frame Action Generation`
- `2D Character Starter -> Concept Simplify Idle Generation`
- `2D Character Starter -> Numeric Subagent Sampling`
- `2D Character Starter -> Game Character Asset Pipeline`
- `2D Character Starter -> See-through`
- `2D Character Starter -> AI Game DevTools`
