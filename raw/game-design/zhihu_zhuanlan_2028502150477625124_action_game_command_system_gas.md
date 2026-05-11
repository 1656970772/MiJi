---
source_url: "https://zhuanlan.zhihu.com/p/2028502150477625124"
type: webpage-summary
title: "动作游戏的指令系统可以怎么做？"
captured_at: 2026-05-11T21:20:50+08:00
author: "魔流雲SOKIGA"
topics:
  - Action Game
  - Combat Design
  - Command System
  - Input Buffer
  - Combo
  - Cancel
  - Unreal Engine
  - Enhanced Input
  - Gameplay Ability System
  - Gameplay Tags
  - Gameplay Events
related_source:
  - "https://www.bilibili.com/video/BV1cjdVBLEoF"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine?application_version=5.6"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/input-overview-in-unreal-engine?application_version=5.6"
  - "https://dev.epicgames.com/documentation/unreal-engine/using-gameplay-abilities-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/using-gameplay-tags-in-unreal-engine?application_version=5.6"
capture_scope: "摘要型资料卡；未全文转存；基于知乎原文与 Epic 官方文档整理"
---

# 动作游戏指令系统资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《动作游戏的指令系统可以怎么做？》，链接为 `https://zhuanlan.zhihu.com/p/2028502150477625124`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `魔流雲SOKIGA`，作者简介为 `一名战斗策划，也学别的：我的身体有70%由游戏构成`。
- 页面结构化数据中显示文章创建时间为 `2026-04-17 22:09:18 +08:00`，更新时间为 `2026-04-17 22:24:22 +08:00`。
- 截取时页面结构化数据中显示约 `36` 赞同、`0` 评论。
- 页面话题包括 `动作游戏（ACT）`、`游戏开发`、`虚幻引擎`。
- 页面正文包含 `11` 张图片。
- 原文关联视频为 Bilibili《【UE5】用基于GAS的指令系统开发各类技能》，链接为 `https://www.bilibili.com/video/BV1cjdVBLEoF`。

## 文章结构

- `前言`
- `输入逻辑与技能逻辑的定义`
- `技能的激活链路`
- `输入`
- `指令`
- `激活技能`
- `技能结束`
- `最后`

## 外部可核验来源

- Epic 官方 Enhanced Input 文档说明 Input Mapping Context 描述触发一个或多个 Input Action 的规则，按键、轴、Input Triggers 和 Input Modifiers 共同决定 Input Action 如何被触发。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine?application_version=5.6`。
- Epic 官方 Input Overview 文档说明，要触发 Input Action，需要把它放入 Input Mapping Context，并把该 Mapping Context 加入本地玩家的 Enhanced Input Local Player Subsystem。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/input-overview-in-unreal-engine?application_version=5.6`。
- Epic 官方 Gameplay Ability 文档说明 `UGameplayAbility` 定义能力做什么、何时或在什么条件下可用，并提供 `CanActivateAbility`、`ActivateAbility` 等生命周期；文档还说明可通过 `Send Gameplay Event To Actor` 发送 Gameplay Event 触发 Ability。来源：`https://dev.epicgames.com/documentation/unreal-engine/using-gameplay-abilities-in-unreal-engine`。
- Epic 官方 Gameplay Tags 文档说明 Gameplay Tags 需要加入 tag dictionary 后 Unreal 才能识别，适合用来标记、分类和查询游戏概念。来源：`https://dev.epicgames.com/documentation/en-us/unreal-engine/using-gameplay-tags-in-unreal-engine?application_version=5.6`。

## 核心观点

原文认为，动作游戏中的单击、蓄力、节奏、组合输入，以及预输入、取消、派生等机制，需要一个完整清晰的指令系统。GAS 提供了较完整的技能方案，但在复杂指令需求和后期技能铺量下，如果直接用 GameplayTag 在 GameplayAbility 内组织连招，会让输入逻辑和技能逻辑边界不清，并且随着技能组扩大而变得复杂。

文章提出的核心分工是：输入逻辑负责“激活”，技能逻辑负责“条件”。节奏、输入组合、蓄力、技能过程中的指令缓存都属于输入逻辑，由指令系统处理；技能 A 需要技能 B 作为前置条件、某技能执行中禁止另一个技能激活等，属于技能逻辑，交给 GAS 处理。

我的判断：这篇资料最有价值的一句话就是“输入负责激活，技能负责条件”。它不是否定 GAS，而是给 GAS 前面加一层专门面向动作游戏手感的 command layer，让复杂连段、预输入、取消和派生先被整理成可维护的指令输出。依据是原文明确把输入逻辑和技能逻辑拆开，并把 GAS 放在条件、优先级、打断和前置条件一侧。

## 系统链路

### 1. 输入层

原文建议输入侧主要使用 UE 的 Enhanced Input System。项目创建一系列 `Input Action` 表示功能，在 `Input Mapping Context` 中配置按键；输入通过 Input Action 的 Trigger Event 绑定后续模块，最终输出指令。

我的判断：Enhanced Input 适合放在“物理输入归一化”层，而不是直接承载连招规则。按下、松开、蓄力、方向输入等可以在这里被转换成统一的输入事件，但连段优先级和缓存窗口应进入下一层处理。依据是原文把 Input Action/Mapping Context 放在输入侧，后续再交给 Command System Component。

### 2. 指令层

原文把指令侧拆成两个组件：

- `Command System Component`：负责指令定义、处理与输出。
- `Ability Activate Component`：负责接收指令并激活技能。

指令定义通过 `Input Config` 完成。`Input Config` 作为 Data Asset 给策划配置：某个 `Input Action` 要输出什么 `Input Tag`，对应 Trigger Event 绑定按下、松开、蓄力等输入形式。文章还提到可把移动输入处理成 Tag，存入玩家 `Ability System Component` 的 Owned Tag，用于方向派生技能激活。

`Command Update` 会做上下文采集和安全性筛查，并输出有效输入列表给 `Handlers` 决策。`Handlers` 以 Data Asset 管理，按顺序或 tick 激活；基础 handler 是 `Standard Priority Handler`，用于对 Input Tags 做优先级排序与缓存，实现复杂情况下的指令替换和共存。

我的判断：把 Handler 做成可配置、可扩展的决策链，是这套设计最适合量产动作技能的地方。轻重攻击、闪避派生、格挡派生、蓄力释放、输入覆盖和输入共存，都可以通过 handler 策略扩展，而不是把判断堆进每个 GameplayAbility。依据是原文把 Handlers、Standard Priority Handler、优先级排序与缓存列为指令层核心。

### 3. 激活层

原文说 `Command Output` 会把准备好的 `Input Tag` 发送到 `Ability Activate Component`，组件以事件形式接收 `In Tag`，并在蓝图中应用，从而实现技能激活逻辑落地。

在 `Ability Activate Component` 中，可以根据具体状态决定是否激活技能，例如闪避派生、格挡派生、连招数等；也可以在此处做开销检测，避免打断正在执行的动作。

原文还提到自封装函数 `Send Ability Activate Event to Actor`：它复用 `Send Gameplay Event`，由 Gameplay Tag 驱动激活 Gameplay Ability，并配合 `Command Buffer` 的动画通知状态，在 Tick 期间接收配置，在 End 时激活，从而实现预输入。文章说明连招窗口会略早于预输入窗口结束，以满足激活条件。

我的判断：这个设计把“按键发生了什么”和“技能能不能启动”中间隔出一层激活适配层，适合动作游戏。它允许同一个 Input Tag 在不同状态下触发不同技能，也允许预输入窗口和连招窗口按动画时序精细对齐。依据是原文对 Ability Activate Component、Send Ability Activate Event to Actor 和 Command Buffer 动画通知状态的描述。

### 4. 技能层与结束

文章最后把真正的技能逻辑交给 GAS：通过 Gameplay Event 和指定 Gameplay Tag 激活 Gameplay Ability，GA 内部处理优先级、打断、前置条件等技能逻辑。技能结束通常发生在 `EndAbility` 后，用于初始化、清理，避免下一个动作接入时出问题。

我的判断：技能结束清理不应只当作收尾，而应该是动作系统可靠性的关键节点。输入缓存、连段计数、窗口状态、消耗检测结果、临时 tags 和动画状态如果不在结束时清理，很容易造成下一招误触发或无法触发。依据是原文把技能结束定义为初始化和清理流程。

## 可迁移的设计模板

我的判断：可以把这篇文章抽象成一条动作游戏指令管线：

`物理输入 -> Enhanced Input Action/Trigger -> Input Config/Data Asset -> Input Tag -> Command Update -> Handler 决策/缓存/优先级 -> Command Output -> Ability Activate Component -> Gameplay Event/Gameplay Tag -> GameplayAbility -> EndAbility 清理`

这条链路的重点是把输入、指令、激活、技能条件分开。输入层负责采样和触发，指令层负责把输入整理成可缓存和可替换的意图，激活层负责在当前动作状态下选择技能，GAS 负责真正的技能生命周期、条件和打断。

## 适合场景

- UE/GAS 项目要实现动作游戏连招、派生、防御、闪避、蓄力和预输入。
- 技能数量会持续扩张，需要避免把输入判断散落到各个 GameplayAbility。
- 战斗策划需要通过 Data Asset 配置输入 tag、优先级、缓存和 handler 策略。
- 项目希望用 Enhanced Input 处理物理输入，用 GAS 处理技能生命周期，中间补一个动作游戏 command layer。

## 不适合场景

- 不适合非常简单的技能系统；如果只有少量按键直接触发技能，这套分层可能过重。
- 不适合把所有战斗判断都交给输入层；技能成本、打断、前置条件仍应回到 GAS 或技能状态机中。
- 不适合直接当作完整实现教程；原文是框架设计与示意，具体代码、网络同步、动画通知和多角色状态仍需项目落地验证。

## 后续检索关键词

- 中文：`动作游戏 指令系统`、`UE GAS 指令系统`、`Enhanced Input 连招`、`Gameplay Event 激活 Gameplay Ability`、`输入缓存 预输入 连招窗口`、`动作游戏 取消 派生`
- English: `action game command system`, `Unreal GAS combo input system`, `Enhanced Input combo buffer`, `Gameplay Event activate ability`, `input buffer cancel window`, `combat command layer`

## 来源

- 用户提供材料：知乎文章《动作游戏的指令系统可以怎么做？》，`https://zhuanlan.zhihu.com/p/2028502150477625124`。
- 原文关联视频：`https://www.bilibili.com/video/BV1cjdVBLEoF`。
- Epic Developer Community：`Enhanced Input in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine?application_version=5.6`。
- Epic Developer Community：`Input Overview in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/input-overview-in-unreal-engine?application_version=5.6`。
- Epic Developer Community：`Using Gameplay Abilities in Unreal Engine`，`https://dev.epicgames.com/documentation/unreal-engine/using-gameplay-abilities-in-unreal-engine`。
- Epic Developer Community：`Using Gameplay Tags in Unreal Engine`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/using-gameplay-tags-in-unreal-engine?application_version=5.6`。
