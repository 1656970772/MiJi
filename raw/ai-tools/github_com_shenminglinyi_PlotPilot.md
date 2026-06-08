---
source_url: "https://github.com/shenminglinyi/PlotPilot"
type: github-repository-card
title: "PlotPilot（墨枢）：面向长篇 AI 创作的剧情引擎内核"
captured_at: 2026-06-09T00:43:20+08:00
contributor: "Codex"
license: "Apache-2.0 + Commons Clause"
language: "Python"
topics:
  - PlotPilot
  - 墨枢
  - Narrative Engine Kernel
  - AI Writing Infrastructure
  - Long-form AI Creation
  - Persistent Memory
  - Knowledge Graph
  - Story Bible
  - Storyline DAG
  - Quality Monitor
related_source:
  - "https://github.com/shenminglinyi/PlotPilot"
  - "https://github.com/shenminglinyi/PlotPilot/blob/master/README.md"
  - "https://github.com/shenminglinyi/PlotPilot/releases"
capture_scope: "摘要型资料卡；源材料来自仓库 README 与 GitHub API 元数据"
---

# shenminglinyi/PlotPilot 资料卡：长篇 AI 创作的剧情引擎内核

## 来源事实

- 仓库：`shenminglinyi/PlotPilot`，URL：<https://github.com/shenminglinyi/PlotPilot>。
- GitHub 描述字段：`【墨枢】作者的领航员`。
- 主语言：Python；默认分支：`master`。
- GitHub API 的 license 字段为 `NOASSERTION`；README 徽章与许可证章节说明项目采用 `Apache License 2.0` 并附加 `Commons Clause` 条件限制。
- README 将 PlotPilot 定义为 `剧情引擎内核（Narrative Engine Kernel）`，不是聊天式写作助手，也不是提示词模板集合。
- README 的一句话定位：面向长篇 AI 创作的基础设施，强调 `持久记忆`、`知识图谱`、`自动推进流水线`、`质量治理闭环`。
- README 提到技术栈：Python 3.14.x、FastAPI、Vue 3、TypeScript、Vite、Naive UI、ECharts、Tauri、SQLite、ChromaDB / FAISS、OpenAI 兼容协议、Anthropic Claude、火山方舟 Doubao。
- 仓库根目录包含 `domain/`、`application/`、`engine/`、`infrastructure/`、`interfaces/`、`frontend/`、`scripts/`、`tests/`、`tools/`、`docs/` 等目录。

## 核心定位

PlotPilot 要解决的问题不是“让 AI 生成一段文字”，而是让 AI 系统在数十万字叙事跨度中维持人物一致性、因果链完整性、伏笔闭合率，并能在无人值守条件下持续推进。

我的判断：PlotPilot 属于 `AI 创作基础设施 / 长篇叙事工程 / 写作自动化引擎`，而不是单纯的游戏设计理论资料。它应归入 `raw/ai-tools`，并在图谱中桥接到 `Game Narrative Text / 游戏叙事文本`，因为它把叙事方法论工程化为状态机、检索、管线和质量监控。

## 五个核心子系统

| 子系统 | README 中的职责 |
| --- | --- |
| 叙事状态机 | 持有完整叙事快照，包括 Story Bible、章级摘要链、叙事事件流、故事线 DAG、伏笔注册表 |
| 向量语义检索层 | 维护章节内容索引与三元组索引，支持结构化与语义混合召回 |
| 剧情引擎运行时 | 通过 `EngineDaemon`、`StoryPipelineRunner` 和 `BaseStoryPipeline` 执行章节规划、写作、审计和状态推进 |
| 提示词策略层 | 通过 `prompt_packages/` 的 YAML 配置暴露 20+ 独立提示接点，可切换短篇、长篇或游戏剧本任务 |
| 质量监控子系统 | 提供张力评分、文风相似度检测、漂移告警、定向修写、陈词滥调扫描 |

## 叙事状态管理

README 强调 PlotPilot 不依赖模型自身“记忆”，而是从结构化叙事快照中动态装配上下文窗口：

- `Story Bible`：人物档案、地点图、世界设定三元组。
- `章级摘要链`：每章生成后自动提炼压缩摘要，形成跨章上下文骨架。
- `叙事事件流`：登记关键事件时序，支持因果链追溯。
- `故事线 DAG`：可视化多故事线分支与汇合点。
- `伏笔注册表`：追踪钩子的开启、悬置和消费状态。

我的判断：这组设计把小说创作中的“人物一致性、伏笔闭合、世界观连续性”转成了可追踪数据结构。它和现有 `Game Narrative Text` 资料形成互补：后者偏文本表达方法，PlotPilot 偏生产系统和长篇状态治理。

## 生产运行时

README 指出当前生产入口收敛在 `engine/runtime/engine_daemon.py`，由 `EngineDaemon` 管理生命周期，委托 `StoryPipelineRunner` 运行章节规划、写作、审计与状态推进；章节写作默认走 `BaseStoryPipeline` 十步管线。

关键工程能力包括：

- 单一生产入口：`scripts/start_daemon.py`。
- 可回退写作路径：`PLOTPILOT_USE_STORY_PIPELINE=off/legacy`。
- 熔断保护：连续失败超过阈值自动暂停并附诊断信息。
- 单写者路由：SQLite 写操作统一串行调度，降低并发写冲突。
- SSE 实时推流：生成进度、Token 消耗、阶段和错误通过 Server-Sent Events 推送。
- 检查点快照：阶段推进前自动存档，支持恢复。

## 技术边界

- 后端是 FastAPI + uvicorn，提供 REST API 与 OpenAPI 文档。
- 前端工作台使用 Vue 3 + TypeScript + Vite + Naive UI + ECharts。
- 桌面客户端使用 Tauri。
- 向量存储支持 ChromaDB / FAISS。
- 嵌入服务支持 OpenAI 兼容 API 与本地 `bge-small-zh-v1.5`。
- 主数据库使用 SQLite，并通过 Write Dispatch 单写者调度器处理并发写。
- 生产部署可走 Windows 一键启动脚本、Tauri 桌面安装版或开发者模式。

## 与现有知识库的连接

- 与 `Game Narrative Text`：PlotPilot 把人物一致性、因果链、伏笔闭合和章节推进工程化，适合承接游戏叙事文本方法论。
- 与 `Game Studio Sub-Agents`：两者都面向游戏/创作生产流程自动化；PlotPilot 偏叙事生成内核，Game Studio Sub-Agents 偏团队代理组织。
- 与 `AI Game DevTools`：PlotPilot 属于 AI 创作工具景观中的 narrative / writing / knowledge graph / agentic pipeline 工具。
- 与 `See-through` 不同：See-through 是角色美术资产预处理，PlotPilot 是文本叙事与世界状态治理。

## 我的判断

PlotPilot 最适合放在 `AI 工具 / 叙事工程 / 长篇创作自动化` 分支。它对项目的价值不在于“生成文本效果”，而在于提供一套可参考的长篇叙事系统架构：状态机、检索、DAG、伏笔账本、提示包、质量监控、检查点与守护进程。如果后续项目需要做互动小说、游戏剧情生成、NPC 剧情线自动推进或 IP 设定库维护，它可以作为架构参考和工具候选。

## 后续可搜索关键词

- 中文：PlotPilot、墨枢、剧情引擎内核、长篇 AI 写作、叙事状态机、Story Bible、故事线 DAG、伏笔注册表、章级摘要链、张力心电图、质量治理闭环、自动推进流水线
- English：PlotPilot, Narrative Engine Kernel, AI writing infrastructure, long-form AI creation, narrative state machine, Story Bible, storyline DAG, foreshadowing registry, vector retrieval layer, prompt strategy layer, quality monitor

## Notable Signals For Graph Extraction

- `PlotPilot -> Narrative Engine Kernel`
- `PlotPilot -> Long-form AI Creation`
- `PlotPilot -> Narrative State Machine`
- `PlotPilot -> Vector Retrieval Layer`
- `PlotPilot -> Story Bible`
- `PlotPilot -> Storyline DAG`
- `PlotPilot -> Foreshadowing Registry`
- `PlotPilot -> StoryPipelineRunner`
- `PlotPilot -> Prompt Strategy Layer`
- `PlotPilot -> Quality Monitor`
- `PlotPilot -> Game Narrative Text`
- `PlotPilot -> AI Writing Infrastructure`
