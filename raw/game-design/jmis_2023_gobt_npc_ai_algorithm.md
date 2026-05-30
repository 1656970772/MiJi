---
source_url: "https://www.jmis.org/archive/view_article_pubreader?pid=jmis-10-4-321&utm_source=chatgpt.com"
canonical_url: "https://www.jmis.org/archive/view_article?pid=jmis-10-4-321"
doi: "https://doi.org/10.33851/JMIS.2023.10.4.321"
type: academic-paper-summary
title: "GOBT：面向 NPC 行为的行为树、GOAP 与 Utility 混合 AI 算法资料卡"
captured_at: 2026-05-30T20:21:49+08:00
paper_title: "GOBT: A Synergistic Approach to Game AI Using Goal-Oriented and Utility-Based Planning in Behavior Trees"
journal: "Journal of Multimedia Information System"
publication: "J Multimed Inf Syst 2023; 10(4):321-332"
published_online: "2023-12-31"
authors:
  - Yoosung Hong
  - Tianhao Yan
  - Jinseok Seo
topics:
  - NPC 行为算法
  - 游戏 AI
  - Behavior Tree
  - 行为树
  - GOAP
  - Utility System
  - Utility AI
  - Goal-Oriented Behavior Tree
  - Agent Decision Making
  - Unity
  - Behavior Designer
related_source:
  - "https://www.jmis.org/archive/view_article_pubreader?pid=jmis-10-4-321&utm_source=chatgpt.com"
  - "https://www.jmis.org/archive/view_article?pid=jmis-10-4-321"
  - "https://doi.org/10.33851/JMIS.2023.10.4.321"
capture_scope: "基于用户提供论文链接与 JMIS 经典视图/PubReader 页面整理；做资料卡、算法摘要与迁移判断；未下载 PDF、未复现实验、未实现代码。"
---

# GOBT：面向 NPC 行为的混合 AI 算法资料卡

## 来源事实

- 用户提供链接：`https://www.jmis.org/archive/view_article_pubreader?pid=jmis-10-4-321&utm_source=chatgpt.com`，并标注“这个是 NPC 行为算法，AI 算法”。
- JMIS 页面显示论文题名为 `GOBT: A Synergistic Approach to Game AI Using Goal-Oriented and Utility-Based Planning in Behavior Trees`。
- 论文发表于 `Journal of Multimedia Information System`，卷期页码为 `10(4):321-332`，DOI 为 `10.33851/JMIS.2023.10.4.321`，在线发表时间为 `2023-12-31`。
- 作者为 Yoosung Hong、Tianhao Yan、Jinseok Seo，作者单位均来自 Dong-eui University 相关院系。
- 论文关键词包括 `Behavior Tree`、`GOAP`、`Utility System`、`Artificial Intelligence`。
- 论文摘要说明，GOBT 全称是 `Goal-Oriented Behavior Tree`，在 Unity 中做模拟，把 GOAP、Utility Theory 与传统 Behavior Tree 组合，用于让游戏 Agent 能在巡逻、攻击、撤退等行为之间更灵活地响应环境变化。

## 一句话定位

这篇论文适合归入“NPC 行为算法 / 游戏 AI 决策架构”。它不是单独的寻路、感知或战斗技能算法，而是一个把 `行为树的可读结构`、`GOAP 的动态规划`、`Utility AI 的实时评分选项`合在一起的 NPC/Agent 高层决策框架。

## 论文要解决的问题

传统行为树适合策划和工程师阅读、调试和可视化，但在大型或动态场景里容易遇到两个问题：

- 行为分支越来越多时，树结构会膨胀，维护和扩展成本升高。
- 行为树通常预先写死结构；如果运行时出现未预设的状态变化，需要改上层节点或重组分支。

GOAP 能根据世界状态、目标和动作前后置条件动态生成行动序列，但完整行为流不容易直观看到；Utility System 能根据状态变量实时选择高效用动作，但效用函数设计、解释和计算成本会变成新负担。

GOBT 的核心尝试是：保留行为树作为外层逻辑骨架，只在复杂分支处插入 `Planner Node`，让这类节点内部处理目标选择、动作路径规划和效用评分。

## 算法结构

### 1. 外层：Behavior Tree

行为树仍负责主逻辑结构，例如巡逻、发现敌人、判断警报、进入战斗分支等。它提供直观、层级化、容易编辑的行为流程。

### 2. 中层：Planner Node

Planner Node 是论文提出的关键节点。它位于行为树内部，承担复杂分支的决策职责。论文中的实现方式是基于 Unity 的 Behavior Designer，把 Planner Node 作为一种节点放进行为树编辑流程。

一个 Planner Node 需要配置：

- 目标动作集合。
- 可执行动作集合。
- 状态变量集合。
- 每个动作的前置条件和后置效果。

运行时，Planner Node 会基于当前状态选择目标动作，再从满足当前状态的起始动作开始，沿动作前后置条件寻找通向目标动作的路径。

### 3. 内层：GOAP 式动作规划

GOBT 借用 GOAP 的动作因果链：动作拥有前置条件和后置效果。不同于传统 GOAP 一次性生成并执行成本最低的完整计划，GOBT 把规划范围限制在 Planner Node 所在的局部行为分支里，以降低搜索规模。

### 4. 实时选择：Utility System

GOBT 用 Utility 机制为目标动作和后续动作评分。状态变量会实时影响效用值；当一个动作完成后，Agent 会选择当前效用最高的后续动作继续执行。如果目标动作的效用排序在过程中变化，系统会重新设定目标并重新规划。

论文实现里，状态变量通过继承 `AgentConsideration` 并作为 Unity `ScriptableObject` 资产管理；动作类继承 `GAction`，并实现 `UpdateUtility` 来计算效用值。

## 决策流程抽象

```text
Behavior Tree tick
  -> 进入 Planner Node
    -> 读取当前状态变量
    -> 对目标动作计算 Utility
    -> 选择当前最合适的目标动作
    -> 找到满足当前状态的起始动作
    -> 基于动作前置条件 / 后置效果生成通向目标的路径
    -> 每一步执行后重新计算后续动作 Utility
    -> 选择效用最高的下一步
    -> 若目标效用变化，重新设定目标并规划
  -> 返回行为树继续执行
```

## 论文实验与对比

论文用敌人威胁、警报、支援、武器攻击力变化等场景比较行为树、GOAP、Utility System 和 GOBT。

关键对比结果可以这样理解：

- 传统行为树在敌人能力变化后，可能继续执行已经进入的攻击序列，除非额外改树或加新分支。
- GOAP 能根据状态变化切换目标并重新规划，但如果目标没变，已生成的动作序列可能继续沿旧路径走。
- Utility System 能实时根据效用切换行为，但可解释性和计算成本会随状态变量与公式复杂度上升。
- GOBT 在扩展场景中能根据枪和火箭的攻击效用变化选择不同攻击手段，并在攻击/报警效用变化后切换目标。

论文给出的复杂度说明是：动态规划部分最坏时间复杂度与动作分支因子和搜索深度相关；Utility 选择部分按候选动作数线性计算；额外内存主要随动作和状态变量数量线性增长。

## 适合收录标签

- `NPC 行为算法`
- `游戏 AI`
- `行为树`
- `GOAP`
- `Utility AI`
- `Agent 决策`
- `AI Planner`
- `Unity AI`
- `Behavior Designer`

## 适合使用场景

- NPC 需要在巡逻、追击、撤退、求援、换武器、支援队友等行为间动态切换。
- 项目已有行为树，但树越来越大，复杂决策节点难维护。
- 需要策划能看懂外层行为结构，同时希望局部行为具备动态规划能力。
- 需要根据生命值、敌我强度、距离、弹药、警报状态、队友状态等变量实时改变 NPC 决策。
- 动作集合有明确前后置条件，适合建成“动作因果链”。

## 不适合直接套用的场景

- NPC 行为非常简单，只需要少量固定状态机或简单行为树。
- 行为动作没有清晰前置条件/后置效果，难以构成 GOAP 式因果链。
- 项目缺乏对 Utility 函数的调试工具，策划无法理解为什么 NPC 选了某个行为。
- 需要大量实时 Agent 同时决策，但没有做分帧、缓存、LOD 或事件触发更新，Utility 与路径搜索可能带来性能压力。

## 可迁移设计模板

我的判断：如果把 GOBT 迁移到 Unreal、Godot 或自研引擎，可以不照搬 Unity/Behavior Designer 实现，而是迁移这条架构原则：

`行为树负责可读流程 -> Planner Node 负责复杂局部决策 -> Action 定义前置条件/后置效果 -> Consideration 定义状态评分 -> Utility 决定目标和下一步 -> GOAP 式因果链连接动作`

落地时可以拆成这些数据结构：

- `Blackboard / World State`：敌人、血量、距离、警报、队友、资源等上下文。
- `Action`：名称、前置条件、后置效果、执行函数、冷却、成本或风险。
- `Goal`：想达成的高层结果，例如消灭敌人、进入安全区、拉响警报、支援队友。
- `Consideration`：把某些状态变量映射成效用值，例如血量越低撤退效用越高。
- `Planner Node`：挂在行为树中，只负责一个局部分支的目标选择和动作链规划。

## 我的判断

这篇论文最有价值的点，是把“策划可读性”和“运行时适应性”分层处理：行为树继续做清晰外壳，复杂节点内部再用 GOAP 和 Utility 做动态决策。依据是论文明确把 GOBT 定义为 BT、GOAP、Utility System 的组合，并把 Planner Node 作为连接三者的核心。

我的判断：对 NPC 行为系统来说，GOBT 更像一套“复杂行为节点设计模式”，而不是必须全项目统一替换行为树。更稳妥的做法是只在高复杂度分支使用，例如战斗策略、求援策略、撤退策略、武器选择和协作行为。依据是论文也把搜索与效用计算限定在 Planner Node 的局部分支中，以控制复杂度。

我的判断：真正落地时，最难的不是 Planner，而是 Utility 调参和可解释性。需要工具显示每个候选动作的效用来源、状态变量值、前后置条件命中情况和最终选择原因。依据是论文对 Utility System 的比较中指出，效用公式和资源分配会增加理解与维护成本。

## 后续检索关键词

- 中文：`GOBT NPC 行为算法`、`Goal-Oriented Behavior Tree`、`行为树 GOAP Utility AI`、`NPC 决策 Planner Node`、`游戏 AI 动态规划 效用系统`、`行为树 目标导向 动作规划`
- English: `Goal-Oriented Behavior Tree`, `GOBT game AI`, `Behavior Tree GOAP Utility AI`, `NPC behavior planning`, `planner node behavior tree`, `utility-based action selection`

## 来源

- 用户提供材料：`https://www.jmis.org/archive/view_article_pubreader?pid=jmis-10-4-321&utm_source=chatgpt.com`
- JMIS 经典视图：`https://www.jmis.org/archive/view_article?pid=jmis-10-4-321`
- DOI：`https://doi.org/10.33851/JMIS.2023.10.4.321`
