---
source_url: "https://github.com/uuuuzz/UEBridgeMCP"
type: github-repository-summary
title: "UEBridgeMCP 仓库资料卡"
captured_at: 2026-05-11T21:29:58+08:00
repo: "uuuuzz/UEBridgeMCP"
default_branch: "main"
repo_language: "C++"
repo_license: "GPL-3.0"
topics:
  - Unreal Engine
  - Unreal Engine 5.6
  - Model Context Protocol
  - MCP Server
  - UE Editor Automation
  - AI Agent Tooling
  - Blueprint
  - UMG
  - Sequencer
  - PCG
  - Control Rig
  - C++
related_source:
  - "https://github.com/uuuuzz/UEBridgeMCP"
  - "https://raw.githubusercontent.com/uuuuzz/UEBridgeMCP/main/README.zh-CN.md"
  - "https://github.com/uuuuzz/UEBridgeMCP/blob/main/UEBridgeMCP.uplugin"
  - "https://github.com/uuuuzz/UEBridgeMCP/blob/main/CHANGELOG.md"
  - "https://github.com/uuuuzz/UEBridgeMCP/releases/tag/v1.19.0"
capture_scope: "GitHub 仓库元信息、README.zh-CN、UEBridgeMCP.uplugin、CHANGELOG、目录树与 Release API 核对；未克隆、未编译、未在 Unreal Editor 中运行"
---

# UEBridgeMCP 仓库资料卡

## 来源事实

- 用户提供 GitHub 仓库：`https://github.com/uuuuzz/UEBridgeMCP`。
- 使用 Codex Chrome 插件打开仓库页面时，浏览器标题显示为 `uuuuzz/UEBridgeMCP`。
- GitHub API 截取时，仓库 `uuuuzz/UEBridgeMCP` 为公开仓库，默认分支为 `main`，主语言为 `C++`，GitHub 识别许可证为 `GPL-3.0`。
- GitHub API 截取时，仓库创建时间为 `2026-04-19 19:48:14 +08:00`，最近 push 时间为 `2026-04-26 00:15:16 +08:00`，页面更新时间为 `2026-05-11 14:55:19 +08:00`。
- GitHub API 截取时，仓库约 `37` stars、`7` forks、`2` open issues；这些计数是动态数据，只代表本次采集时状态。
- GitHub Release API 显示当前公开 release 为 `v1.19.0`，名称为 `UEBridgeMCP v1.19.0`，发布时间为 `2026-04-19 19:55:48 +08:00`，包含 `UEBridgeMCP-v1.19.0.zip` 资产，大小约 `328712` bytes。

## 仓库定位

README.zh-CN 将 UEBridgeMCP 定位为面向 Unreal Engine 5.6+ 的原生 C++ MCP 插件。它把 MCP 服务嵌入 Unreal Editor 进程内部，通过 HTTP endpoint 暴露给兼容 MCP 的 AI 客户端，让客户端检查或编辑蓝图、关卡、资产、材质、Widget、StateTree、PIE 会话和构建流程。

README.zh-CN 明确说明，本项目基于 softdaddy-o 的 `yes-ue-mcp` 深度重构与扩充。当前仓库把自己描述为可迁移到 UE 5.6+ C++ 项目的 Editor 插件，而不是绑定某一个游戏项目的专用脚本。

## 目录结构事实

仓库根目录包含：

- `README.md` 与 `README.zh-CN.md`
- `UEBridgeMCP.uplugin`
- `Source/`
- `Config/`
- `Docs/`
- `Validation/`
- `Resources/`
- `CHANGELOG.md`
- `RELEASE_NOTES.md`
- `LICENSE`

GitHub 递归树 API 截取结果：

- blob 文件总数：`553`
- 目录总数：`140`
- `.cpp` 文件：`215`
- `.h` 文件：`218`
- Markdown 文件：`89`
- JSON 文件：`11`
- Python 文件：`0`
- 递归树未被 GitHub API 标记为 `truncated`

`Docs/` 目录同时提供英文与中文文档，包括架构说明、工具开发、工具参考、能力矩阵、故障排查和发布前预检。`Validation/` 目录包含 `Smoke/` 与 `Step6/`，README 与 CHANGELOG 都把这些内容作为发布验证和能力验证材料的一部分。

## 插件描述文件事实

`UEBridgeMCP.uplugin` 显示：

- `VersionName` 为 `1.19.0`
- `FriendlyName` 为 `UE Bridge MCP`
- 插件描述为面向 Unreal Engine 5.6+ 的原生 C++ MCP server，可让 AI assistant 通过 HTTP 与 UE Editor 交互
- `Category` 为 `Editor`
- `IsExperimentalVersion` 为 `true`
- `EnabledByDefault` 为 `false`
- `CanContainContent` 为 `false`

`.uplugin` 声明五个 Editor-only 模块：

- `UEBridgeMCP`
- `UEBridgeMCPEditor`
- `UEBridgeMCPControlRig`
- `UEBridgeMCPPCG`
- `UEBridgeMCPExternalAI`

`.uplugin` 声明的核心依赖包括 `EditorScriptingUtilities`、`GameplayAbilities`、`EnhancedInput`、`StateTree` 与 `GameplayStateTree`。它还声明 `PythonScriptPlugin`、`ControlRig`、`PCG`、`Niagara`、`Metasound` 等可选依赖；README.zh-CN 特别提醒，Python 工具链在源码构建中仍需要留意 Build.cs 的直接链接情况。

## 工具面与工作流要点

README.zh-CN 与 CHANGELOG 都强调：UEBridgeMCP 的工具清单是动态的，应以运行时 `tools/list` 为准。always-on 编辑器工具由 `RegisterBuiltInTools()` 注册，可选模块会带来 Sequencer、Landscape、Foliage、World Partition、Niagara、MetaSound、Control Rig、PCG、External AI 等条件工具或扩展工具。

CHANGELOG 把公开工具面描述为围绕“摘要查询 -> 定向详情 -> 事务编辑 -> 编译/保存 -> 断言”的 agent-first 工作流组织。公开 release 的早期工具范围包括 Blueprint 查询与编辑、World/Level 查询、Actor 详情、Material 查询、Asset 搜索、Python 脚本执行、项目设置读取、类层级检查和 Data Asset 检查等。

README.zh-CN 的快速开始要求包括 Unreal Engine 5.6+、C++ Unreal 项目、复制插件到项目 `Plugins/UEBridgeMCP/`、启用插件、重新生成项目文件并编译编辑器目标。默认服务配置位于 `Config/DefaultUEBridgeMCP.ini`，默认端口为 `8080`，默认绑定地址为 `127.0.0.1`。README 特别提醒 Windows 上优先使用 `127.0.0.1`，以避免 `localhost` 被解析到 IPv6 后造成连接问题。

## 适合用途

- 作为 Unreal Editor 与 AI Agent 之间的本地 MCP 桥接插件候选。
- 作为研究“UE 编辑器自动化 + MCP + C++ 插件化工具面”的源码样本。
- 作为蓝图、关卡、资产、材质、Widget、Sequencer、Niagara、PCG、Control Rig 等 UE 编辑器工作流的 agent 化参考。
- 作为对比本地 UE UMG MCP、yes-ue-mcp、Python bridge、remote control 等方案的资料入口。
- 作为后续整理“AI 驱动 UE Editor 自动化”专题时的核心仓库之一。

## 不适合用途

- 不适合直接视为已在当前项目可用的工具；本次只做资料采集，未克隆、未编译、未在 UE 5.6 项目中加载。
- 不适合直接当作 UE 5.4 或 UE 5.5 插件使用；README 与 badge 都把目标版本标成 UE 5.6+。
- 不适合忽略许可证后直接并入闭源商业工具链分发；GitHub API 与 LICENSE 均指向 GPL-3.0，需要在复用和分发前单独审查合规边界。
- 不适合把 README 的工具清单当作固定清单；仓库自己强调运行时 `tools/list` 才是权威状态。

## 我的判断

这个仓库值得收进“虚幻相关”资料库，而且优先级比较高。依据是：它不是单篇教程或资料索引，而是一个真实 UE C++ 插件仓库；目录中有 215 个 `.cpp` 与 218 个 `.h` 文件，文档和验证材料也比较完整，说明作者目标是把 MCP server 作为 UE Editor 内部能力来工程化。

我的判断是，它的最大价值在于“把 UE 编辑器能力变成 AI Agent 可调用的工具面”。这和普通 UE 教程不同：普通教程解决“人怎么操作编辑器”，UEBridgeMCP 解决“agent 如何用结构化请求查询、编辑、保存、验证 UE 项目状态”。依据是 README.zh-CN 的项目概览、CHANGELOG 中的 agent-first 工作流描述，以及 `.uplugin` 中的 Editor 模块设计。

我的判断是，后续如果要实际试用，应先单独建 UE 5.6 C++ 测试项目，而不是直接放进生产项目。依据是 `.uplugin` 标记 `IsExperimentalVersion: true`，README 要求 UE 5.6+ 与 C++ 项目，且工具面会触达蓝图、关卡、资产、材质、Widget、PIE 和构建流程，风险面比只读资料库更大。

我的判断是，许可证是这份资料最需要额外注意的地方。GPL-3.0 对复制、修改、再分发和衍生作品有强约束；如果只是本地研究和资料索引，风险较低，但如果要把源码或修改版合入自己的工具并分发，就必须先做许可证合规审查。依据是 GitHub API 返回的 `license: GPL-3.0` 与仓库根目录 `LICENSE`。

## 后续检索关键词

- 中文：`UEBridgeMCP`、`UE MCP 插件`、`Unreal Editor MCP`、`虚幻编辑器 AI Agent`、`UE 5.6 MCP Server`、`AI 驱动 UE 编辑器自动化`
- English: `UEBridgeMCP`, `Unreal Engine MCP server`, `Unreal Editor AI agent`, `Model Context Protocol Unreal`, `UE 5.6 editor automation`, `AI assistant Unreal Engine plugin`

## 来源

- 用户提供材料：GitHub 仓库 `https://github.com/uuuuzz/UEBridgeMCP`
- 仓库 README.zh-CN：`https://raw.githubusercontent.com/uuuuzz/UEBridgeMCP/main/README.zh-CN.md`
- 插件描述文件：`https://github.com/uuuuzz/UEBridgeMCP/blob/main/UEBridgeMCP.uplugin`
- 更新日志：`https://github.com/uuuuzz/UEBridgeMCP/blob/main/CHANGELOG.md`
- Release 页面：`https://github.com/uuuuzz/UEBridgeMCP/releases/tag/v1.19.0`
- LICENSE：`https://github.com/uuuuzz/UEBridgeMCP/blob/main/LICENSE`
