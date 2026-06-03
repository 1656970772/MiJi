---
source_url: "https://github.com/BDFFZI/Alife"
type: webpage-summary
title: "GitHub - BDFFZI/Alife: 一个基于 .NET / C# 的可扩展 AI 桌宠 / 赛博生命 Agent 框架"
captured_at: 2026-06-03T12:30:00+08:00
contributor: "Mavis"
topics:
  - Alife
  - AI 桌宠
  - 赛博生命
  - AI Agent
  - LLM 框架
  - .NET 9
  - C#
  - Live2D
  - WPF
  - Blazor Hybrid
  - Semantic Kernel
  - MCP
  - Skill
  - 函数调用
  - 长期记忆
  - QQ 机器人
  - OneBot v11
  - 自主活动
  - 个人项目
related_source:
  - "https://github.com/BDFFZI/Alife"
  - "https://github.com/BDFFZI/Alife/releases"
  - "https://img.shields.io/badge/Alife-AI_Assistant-blue"
  - "https://github.com/user-attachments/assets/02c2c56a-e805-43fc-a352-f866955a7aef"
capture_scope: "摘要型资料卡；本卡片为人工撰写，源材料来自仓库 README 与 GitHub API 元数据"
---

# BDFFZI/Alife 资料卡：基于 .NET / C# 的可扩展 AI 桌宠 Agent 框架

## 来源事实

- 仓库路径：`BDFFZI/Alife`，URL：<https://github.com/BDFFZI/Alife>。
- 仓库描述字段为 `null`，主语言：`C#`；语言占比（来自 GitHub API）：`C# 334,014` 字节、`HTML 145,492`、`Batchfile 5,818`、`CSS 5,267`、`Python 4,339`、`JavaScript 4,036`。
- Stars 22 / Forks 3 / Watchers 22 / Open issues 2 / Subscribers 0 / Size 65,970 KB。
- 许可证：MIT（`{"key":"mit","spdx_id":"MIT"}`）。
- 默认分支：`master`（非 `main`，是该项目自身的仓库习惯）。
- 时间线：`created_at 2026-03-15T16:55:09Z` → `pushed_at 2026-06-02T16:48:04Z`，最近 push 与最近 update（`2026-06-03T04:12:37Z`）间隔不到一天，是活跃维护中的项目。
- 仓库根目录结构（来自 GitHub `contents` API 调用）：
  - 解决方案：`Alife.slnx`、`Directory.Build.props`。
  - 三类源码目录：`sources/`、`Demos/`、`Tests/`。
  - 引导脚本：`点我启动.cmd`、`点我发布.cmd`（中文文件名）。
  - 配置文件：`.gemini/`（与 Gemini 相关的本地配置目录）。
- 仓库 `sources/` 下五个工程目录（README 描述与 API 返回一致）：`Alife.Basic`、`Alife.Function`、`Alife.Framework`、`Alife.Implement`、`Alife`。
- 仓库 `Demos/` 下十一个示例工程：`Alife.Demo.{DeskPet, Framework, Mcp, Memory, Plugin, Python, QChat, SkillIntegration, Speech, Surfing, Vision}`。
- 仓库 `Tests/` 下五个测试工程：`Alife.Test.{Browser, DeskPet, Interpreter, Python, QChat}`。
- README 标识的徽章：`.NET 9`、`Python 3.12`、`License MIT`，自描述为"AI Assistant"蓝色徽章。
- README 内置角色名：`真央`（Live2D 桌宠形象，自带自画像 `02c2c56a-…7aef`）。
- README 提供的作者联系方式：B 站 `BDFFZI`、QQ 交流群 `427674145`。
- README 中明确列出的依赖栈：
  - 编程语言：.NET 9、Python 3.12。
  - LLM 协议：Semantic Kernel。
  - 前端：WPF + Blazor Hybrid + AntDesign Blazor。
  - 模型管理：`modelscope`（默认下载到 C 盘，可通过环境变量改位置）。
  - 桌宠渲染：`pixi-live2d-display`（在 `DeskPetService` 描述中出现）。
  - 语音：`sherpa-onnx-sense-voice`（ASR）+ `silero-vad`（VAD）+ `Windows.AudioGraph`（回声消除 / 降噪）+ `edge-tts` / `vits`（TTS）。
  - 视觉：Qwen2.5-VL-3B 本地模型。
  - 记忆向量化：`bge-small-zh-v1.5` + `duckDb`。
  - QQ 平台：OneBot v11 协议。
  - 浏览器：`WebView2` 模拟人类操作。
- README 描述的五个内置插件分组（核心底座 / 环境搭建 / 对外表达 / 实用工具 / 扩展增强）共 14 个内置服务。

## 核心定位

> Alife 是一个"赛博生命"框架——不是简单的对话机器人，而是支持多模态、主动陪伴、永久记忆、可以生活在桌面的真正伙伴。

- **形态**：C# / .NET 9 桌面端 AI 桌宠应用，内置 Live2D 角色"真央"，可做气泡文字、表情动作、鼠标交互、位置交互。
- **能力范围**：对话、长期记忆、视觉感知（Qwen2.5-VL-3B）、语音对话（流式 ASR+TTS）、QQ 机器人（OneBot v11）、Web 冲浪（WebView2 模拟浏览器）、Python 脚本执行、自主活动（基于主动事件与虚拟世界背景）、多开互联（角色之间可互聊）。
- **扩展机制**：全插件化架构，所有内置功能本身就是插件；标准插件只需 `继承 InteractivePlugin<T>` 并加 `[Plugin]` 属性即可注册；支持热重载（替换 dll 即可）。
- **关键自实现项**：
  - **函数调用**：自实现的"基于 XML 的流式函数调用"——支持开闭标签、嵌套链、注释转义、自动语法纠错；与 Semantic Kernel 兼容；不依赖 LLM 的"工具调用"格式，因此不挑模型。
  - **记忆系统**：自实现的"多级缓存 + 自然底数"压缩系统 + `bge-small-zh-v1.5 + duckDb` 向量检索；反对不可靠的 AI 自主记忆存储。
  - **模型下载**：用 `modelscope` 国内镜像 + 启动脚本"兜底崩溃 / 自适应环境"，对国内小白用户友好。
  - **协议接入**：支持 MCP（标准模型上下文协议）和 Skills（渐进式 SKILL.md 按需加载），可以接入业界标准工具生态。

## 架构层次

| 层 | 工程 | 职责（README 原文） |
| --- | --- | --- |
| 1 | `Alife.Basic` | 封装底层 OS 能力，统一软件环境变量 |
| 2 | `Alife.Function` | 针对各种特种功能的无依赖模块化实现 |
| 3 | `Alife.Framework` | 制定标准业务处理单元，实现基本功能系统，确定插件框架结构 |
| 4 | `Alife.Implement` | 以插件方式把功能模块层组装为内置业务功能 |
| 5 | `Alife` | 接入 UI 界面（WPF + Blazor Hybrid + AntDesign Blazor） |

> 来源：README「🏛️ 层次架构」一节（行 165–173）。
> 我的判断：这是一个"内核—插件—UI"三层分离的典型 Agent 框架结构，与 Unity 的 Package 系统 / Unreal 的 Module 系统思路高度同构；这意味着 `Alife.Framework` 是复用价值最高的一层，可独立抽出来给其他项目（甚至非桌宠形态的 AI 应用）做内核。

## 14 个内置服务（Plugin 视角）

| 分组 | 服务（README 中的 Service 名） | 简述 | 来源 |
| --- | --- | --- | --- |
| 核心底座 | `ChatService` | 兼容 OpenAI 协议的 LLM 接入，DeepSeek 调优 | README 行 181 |
| 核心底座 | `FunctionService` | 自实现 Xml 流式函数调用 + 自动语法纠错 | README 行 182 |
| 环境搭建 | `MessageProcessService` | 输入消息预处理（时间戳、输出风格等包装） | README 行 186 |
| 环境搭建 | `EventService` | 系统事件 + 阶梯性定时事件，驱动 AI 主动行为 | README 行 187 |
| 环境搭建 | `MemoryService` | 多级缓存压缩 + 向量化检索 | README 行 188 |
| 环境搭建 | `VirtualWorldService` | 虚拟世界背景 + 跨活动消息通讯 | README 行 189 |
| 对外表达 | `DeskPetService` | Live2D 桌宠（pixi-live2d-display） | README 行 193 |
| 对外表达 | `SpeechService` | 流式 ASR + 多声线 TTS + 回声消除 | README 行 194 |
| 对外表达 | `QChatService` | OneBot v11 QQ 群聊机制 | README 行 195 |
| 实用工具 | `SurfingService` | WebView2 模拟人类浏览 | README 行 199 |
| 实用工具 | `PythonService` | 自带 Python 环境执行脚本 | README 行 200 |
| 实用工具 | `VisionService` | Qwen2.5-VL-3B 视觉 + 窗口统计 OCR 兜底 | README 行 201 |
| 扩展增强 | `SkillService` | 渐进式 SKILL.md 按需加载 | README 行 205 |
| 扩展增强 | `McpService` | 标准 MCP 客户端，动态接入外部工具 | README 行 206 |

## 核心问题

做一个"不商业、不挑模型、不挑硬件、永久记忆、主动陪伴"的中文 AI 桌宠，最大的拦路虎是什么？

- **商业味 + 隐私**：商用聊天 App 数据不安全，作者明确拒绝；本项目所有依赖走国内镜像、不需要 API Key、内置模型默认本地（`modelscope` 拉到 C 盘，可改环境变量）。
- **不挑模型**：刻意不用 LLM 的"Tool Use"格式，改用自实现的 XML 流式函数调用，作者声称 DeepSeek、mimo、kimi 均已测试可用。
- **永久记忆**：放弃"AI 自主写记忆"路线（不可靠），改用"多级缓存 + 自然底数"压缩 + 向量检索，做"伪常驻上下文"。
- **主动陪伴**：靠 `EventService` 给 LLM 阶梯性定时事件 + `VirtualWorldService` 给一个"虚拟世界"提示词环境，让 LLM 在空闲时自发产生好奇心、发起话题、做自主学习。
- **白盒 + 模块化**：所有功能以插件实现，单个功能可"拆离复用"；运行信息白盒化，对话和上下文均有专用 UI 完整显示（README「模块化白盒化软件结构」一行）。

## 一句话结论

`BDFFZI/Alife` 是一个 2026 年 3 月新立项、个人开发者主导的 .NET 9 AI 桌宠框架；它用 C# + 全插件化架构把"主动陪伴 + 永久记忆 + 多模态 + QQ 联网 + 本地化部署"打包成一键启动的开箱即用应用，并通过自实现的 XML 函数调用、bge/duckDb 向量记忆、MCP + Skills 扩展协议三个差异化设计来绕开商业 LLM 框架对工具调用格式的强约束。

## 我的判断

> 这部分是基于以上事实的个人分析/推断/方案设计，并非项目自述。

### 适合谁研究

1. **想做 AI 桌宠 / 数字伴侣 / AI 虚拟陪伴产品的人**：项目给出了完整的"个人 → 完整应用"参考实现，含 Live2D 集成、流式语音、本地视觉模型、QQ 机器人。
2. **正在评估"插件化 Agent 框架"架构的人**：`Alife.Framework` 是一份相对克制、文档齐全的参考实现（与 `Semantic Kernel` 的 `KernelFunction` 思路同构，但更轻量），值得作为"如何在 C# 里做插件系统"的学习样本。
3. **关心 MCP / Skills 生态整合的人**：项目同时支持 MCP 客户端（标准协议）和 Skills 系统（渐进式 SKILL.md），与本工作区 `mavis` 的 `skill` 机制在概念层同源——本资料卡也是基于"我自己的 agent 系统"视角产出的。
4. **想了解"国内个人项目怎么做 LLM 应用"的人**：作者明确写了"低配低开销、不要 key、国内镜像、DotNet 生态、小众个人没负担"——这是国内独立开发者的真实产品思路，与硅基叙事不同。

### 适合怎么用

- **不要直接 fork 当产品用**：README 自述"由于个人精力有限，Demos / Tests / UI 部分大量使用 AI 编程，这些子项目很可能存在冗余、低效问题"——明确说非核心代码质量不高。如果想拿来做产品，需要重写 Demo / Test / UI。
- **可借鉴的设计**：
  - XML 流式函数调用（避免锁定特定 LLM 的 Tool Use 格式）。
  - 多级缓存 + 向量检索的"伪常驻记忆"。
  - 主动事件 + 虚拟世界背景 → 主动陪伴。
  - 全插件 + 热重载（替换 dll 即可）。
  - `点我启动.cmd` 一键环境搭建思路（对国内小白友好）。
- **可借鉴的资源**：`Alife.Demo.Plugin` 提供了从数据类、插件类、UI 类到函数注册、配置注入、提示注入的完整示例（README 行 110–150 摘录的 C# 代码是教学级别的最小插件示例）。

### 与本工作区已有资源的连接

- `raw/ai-tools/github_com_Donchitos_Claude-Code-Game-Studios.md`（49 agents × 72 skills 的 Claude Code 游戏工作室）—— Alife 是从"游戏/桌宠"角度做的同构设计。
- `raw/ai-tools/github_com_pamirtuna_gamestudio-subagents.md`（12 agents × 多引擎的游戏工作室 sub-agents）—— 同上。
- `raw/ai-tools/zhihu_zhuanlan_2011851609727059384_renderdoc_ai_mcp_workflow.md`（RenderDoc + Unity MCP 工作流）—— Alife 的 `McpService` 是这一思路的"反过来"用法（让 AI 主动使用 MCP 工具，而不是让 AI 帮人类调 RenderDoc）。
- `raw/ai-tools/github_com_Yuan-ManX_ai-game-devtools.md`（AI 游戏开发工具汇总）—— Alife 属于"Agent / Code"分类下的具体落地。

### 我没确认的事

- 仓库根目录的 `.gemini/` 文件夹具体内容（GitHub API 仅返回空目录占位，未展开）。
- 角色"真央"的自画像 `02c2c56a-…7aef` 是否在项目中作为资源分发，还是仅 README 引用——未展开附件检查。
- Demos 11 个工程与 Tests 5 个工程的具体文件级内容（API 仅返回顶层目录）。
- 实际插件数 / 文件数（API 未提供 `tree` 数据）。
- 默认分支是 `master` 而非 `main`——是作者个人习惯还是项目早期遗留下来；不影响使用但下游 fork 时需注意。
