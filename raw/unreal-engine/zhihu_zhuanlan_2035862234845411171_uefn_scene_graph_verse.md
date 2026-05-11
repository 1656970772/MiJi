---
source_url: "https://zhuanlan.zhihu.com/p/2035862234845411171"
type: webpage-summary
title: "UF2025(Orlando)——看到了Unity的影子，UEFN 的Scene Graph 与 Verse 深度解析"
captured_at: 2026-05-11T19:04:35+08:00
author: "wlxklyh"
column: "UF2025(Orlando)"
topics:
  - UE5
  - UEFN
  - Scene Graph
  - Verse
  - Entity Component
related_source:
  - "https://www.youtube.com/watch?v=IW8HjoO-Uj0"
  - "https://dev.epicgames.com/documentation/uefn/scene-graph-in-unreal-editor-for-fortnite"
  - "https://dev.epicgames.com/documentation/uefn/getting-started-in-scene-graph-in-fortnite"
  - "https://dev.epicgames.com/documentation/en-us/uefn/components-in-unreal-editor-for-fortnite"
  - "https://dev.epicgames.com/documentation/en-us/fortnite/entity"
capture_scope: "摘要型资料卡；未全文转存"
---

# UEFN Scene Graph 与 Verse 资料卡

## 来源事实

- 来源文章：知乎专栏《UF2025(Orlando)——看到了Unity的影子，UEFN 的Scene Graph 与 Verse 深度解析》。
- 作者与专栏：作者为 `wlxklyh`，收录于知乎专栏 `UF2025(Orlando)`。
- 编辑时间：知乎页面显示编辑于 `2026-05-08 00:58・新加坡`。
- 文章主题标签：`UE5`。
- 原始演讲：文章说明基于 `Powering Up Your Projects With Scene Graph and Verse | Unreal Fest Orlando 2025` 整理。
- 演讲者：文章标注主讲为 Evan Brown，来自 Epic Games，职位为 Programmer Writer。
- 视频链接：文章提供的源视频为 `https://www.youtube.com/watch?v=IW8HjoO-Uj0`。
- 官方文档佐证：Epic 官方文档说明 Scene Graph 是 UEFN 中连接世界对象的统一结构，是 Verse native system；文档也提示这是 Beta 功能，出货时需要谨慎。官方文档还说明 Entity 是 Scene Graph 的基础对象，由 components 与 child entities 构成；components 为 entities 提供数据和行为。

## 核心问题

这篇资料讨论的是：UEFN 为什么要从传统 `Creative prop + Verse device` 的中心管理模式，转向 `Entity + Component + Verse` 的 Scene Graph 模型。

它不是普通 UE 主干项目的 Actor / Component 教程，而是 UEFN 生态里对象组织方式的转向：对象不再只是被外部 device 遥控的 prop，而是通过组件拥有自己的数据、行为和生命周期。

## 一句话结论

我的判断：Scene Graph 的核心价值不是“又一个层级面板”，而是把 UEFN 项目中的对象、组件、Prefab 和 Verse 行为放进同一个实体树。它把开发者的默认心智从“中心 device 管所有对象”推向“对象通过组件拥有能力”。

## 技术路线拆解

### 1. 为什么 UEFN 要引入 Scene Graph

文章先解释传统 UEFN 工作流的问题：Creative prop 负责被摆在世界中，Verse device 负责在游戏开始时启动逻辑并持续控制 prop。这个模式能工作，但对象数量、行为组合和运行时生成需求增加后，会出现结构性问题：

- 对象本身缺少行为归属，状态和逻辑常常落到外部 device 里。
- 跨对象协作依赖中心 device 调度。
- 复用粒度偏粗，新行为往往意味着新增 device 或继续膨胀基类。
- 动态生成成本高，需要生成 prop、目标点、管理类、配置项和手动 setup。

官方文档也把 Scene Graph 描述为统一连接世界对象的结构，可用于复制、迭代、Prefab 化和跨项目复用。

我的判断：Scene Graph 解决的不是“怎么摆对象”，而是“行为到底属于谁”。传统 device-driven 模式里答案常常是中心管理器；Scene Graph 试图把答案改成实体自己。

### 2. Entity：刻意保持空白的容器

文章强调 Entity 一开始几乎什么都没有：没有位置、网格、光照、交互性，也不天然具备 3D 空间存在感。要让它具备能力，必须显式添加组件：

- 位置：`transform component`
- 可见形态：`mesh component`
- 发光：`light component`
- 逻辑行为：自定义 `Verse component`

官方 Entity 文档也说明，Entity 是 Scene Graph 的基础对象，是轻量类，包含 components 与 child entities。

我的判断：Entity 的“空”是设计重点。它不是一个弱化版 Actor，而是一块白板；你用组件决定它是什么。

### 3. Component：能力的最小组合单位

文章把 Component 视为数据与行为的承载单元。一个实体可以通过不同组件组合成灯、平台、机关、生成器或其他对象。

官方 Components 文档也说明，Scene Graph 中的 component 是 focused 的，代表一种行为或特征；添加到 entity 后，就建立 entity 与该 component behavior 的关联，这被称为 composition oriented design。

文章提到自定义 Verse component 的关键变化：代码的运行入口从过去 Verse device 的 `OnBegin`，转为组件所在实体开始模拟时的生命周期，例如 `OnBeginSimulation`。

我的判断：这会逼迫开发者把“大管理器脚本”拆成更小的能力组件。能力组合优先于继承扩展，是这套模型的主要收益。

### 4. Prefab：结构、组件和行为一起复用

文章提到 Prefab 保存的不是简单 prop 列表，而是一组 Entity + Component 的结构。一个带 transform、mesh、movement component 和自定义 Verse component 的平台，可以作为完整可运行单元拖入关卡。

官方 Getting Started 文档也说明，Scene Graph 会把 entities 组织成 ancestor / descendent 关系并保存为 prefab；Prefab 可被实例化、嵌套和复用，修改 Prefab 可影响实例，同时实例也可以覆盖组件值。

我的判断：Prefab 真正的价值不是静态复制，而是“结构 + 行为”的复用。一个 Prefab 被运行时 spawn 后，组件生命周期能自动开始，这比手动 prop + device setup 更适合动态内容。

### 5. Verse：Scene Graph 稳定性的语言基础

文章把 Verse 放在 Scene Graph 的基础层，而不只是脚本语言层。它提到的关键属性包括：

- 强类型与向后兼容，用于支撑长期可用的公开内容；
- 全局命名空间，减少跨创作者代码冲突；
- 事务回滚等语言机制，用于减少用户代码破坏共享体验的风险；
- 对 Scene Graph 的深度访问，使实体和组件可由 Verse 查询、生成和修改。

官方 Getting Started 文档也说明 Scene Graph 是 Verse native system，Verse 可用于 spawn/remove entities、创建自定义 components 和 behaviors。

我的判断：UEFN 面向的是创作者生态与长期可运行内容，因此语言约束比传统“游戏脚本”更重。Verse 的稳定性承诺是 Scene Graph 能被开放给大量创作者的前提。

### 6. 核心示例：按组件查询场景实体

文章中的关键例子是：从 simulation entity 出发，查找整棵实体树中所有带 `light_component` 的实体，然后给这些实体添加粒子系统组件。

它展示了 Scene Graph 相比 device 模式的差异：

- 查询目标不是某类 prop，也不是手工维护数组，而是“所有含某组件的实体”。
- 查询范围可以覆盖统一实体树。
- 新能力作为组件加入目标实体，而不是外部粘附或中心遥控。
- 同一段逻辑可以挂在组件生命周期里运行，不依赖全局 device。

我的判断：按组件查询是 Entity-Component 模型的核心生产力。它让“给所有具备某种能力的对象加另一种能力”变得自然。

### 7. 样例关卡：移动平台从 device 管理变为实体自驱

文章复述了一个平台关卡例子：传统 device 版本需要中心管理器持有所有 animating prop、目标点和配置，并在 `OnBegin` 遍历 setup。新增平台时需要复制 prop、目标点、改名、回到 device 新增配置、手动绑定引用。

Scene Graph 版本则把移动能力做成 `animate_targets_component`，挂在平台实体上。平台实体持有自己的目标和 key frame movement，Prefab 拖入或运行时实例化后，组件自己开始模拟。

我的判断：这正是 Scene Graph 的体验差异。静态摆放时节省操作，动态生成时收益更大，因为对象一生成就是完整行为单元。

### 8. 组件化思维与中心管理思维的边界

文章特别提醒：不要形式上用了组件，实际上又写一个 God Component / Manager 去远程操纵所有实体。

但文章也没有否定 Creative device。它提到 device 仍适合 mutator zones、全局规则、回合制、分数管理和与现有 Creative 设备紧密集成的逻辑。

我的判断：现阶段最现实的是混合模式：Scene Graph 管对象自身状态和行为，Verse device / Creative device 管全局协调和已有设备生态。

### 9. 当前限制与路线图边界

文章记录的限制包括：

- Scene Graph 当前仍有底层组件未完全暴露；
- physics 仍在推进，尤其是与角色互操作；
- Fortnite characters 未来会逐步以 entity + component 形态存在；
- Fab 对 Scene Graph 的支持仍在开发；
- Itemization 在路线图中。

官方 Scene Graph 文档也明确提示这是 Beta 功能，出货时需要谨慎。

我的判断：不要把路线图当既成能力。生产项目应以当前 UEFN 版本实际暴露 API 为准，尤其是 physics、characters、Fab 和 itemization。

## 适合与不适合

### 适合

- 自带行为的对象：移动平台、机关、灯具、NPC 生成器、可复用交互物。
- 高频 Prefab 化内容：需要被多次放置、变体化、实例覆盖的内容。
- 运行时动态实例化：spawn 后应自动具备完整行为。
- 需要按组件查询和批量操作的场景。

### 仍适合 device-driven 或混合模式

- 全局规则入口、回合系统、计分系统。
- Mutator zones 等现有 Creative 设备强相关逻辑。
- 与旧设备生态紧密集成的关卡机制。
- 需要项目级协调，而不是某个实体自身行为的系统。

## 可迁移清单

- 实体命名体现职责，而不只是外观，例如 `MovingPlatform_Prefab` 优于 `BlueBlock_01`。
- 组件命名描述能力，例如 `AnimateToTargetsComponent`、`DamageOnTouchComponent`、`SpawnGuardsOnOverlapComponent`。
- 一个组件尽量只解决一件事，避免多能力巨型组件。
- 把能独立复用的对象做成 Prefab，把全局规则放在更高层系统。
- 动态生成内容时，优先验证 Prefab 实例化后组件生命周期是否自动运行。
- 跨实体通信优先使用组件查询、实体层级和明确接口，不要回退到隐式全局状态。
- 使用 Scene Graph 前确认当前 UEFN 版本的 Beta 状态、可用组件和限制。

## 我的判断

这篇资料最适合归入 `虚幻 / UEFN / Scene Graph / Verse / Entity-Component 架构`。

它的长期价值在于解释 UEFN 为什么要把对象世界推向组合式建模：Entity 是容器，Component 是能力，Prefab 是可复用结构，Verse 是稳定运行和迁移的语言基础。

对现阶段项目，我会采用混合策略：用 Scene Graph 管实体自身行为和可复用对象，用 Verse device / Creative device 管全局规则与现有设备集成。这样能吃到组件化与 Prefab 复用收益，又不把 Beta 能力当成完整替代品。

## 后续检索关键词

- 中文：UEFN Scene Graph、UEFN Verse、UEFN Entity Component、Scene Graph Prefab、Verse component、UEFN 组件化、Creative device、UEFN Prefab
- English：UEFN Scene Graph, Verse Scene Graph, Entity Component UEFN, Verse component, Scene Graph prefab, Powering Up Your Projects With Scene Graph and Verse, Unreal Fest Orlando 2025

## 来源

- 知乎原文：https://zhuanlan.zhihu.com/p/2035862234845411171
- 源视频：https://www.youtube.com/watch?v=IW8HjoO-Uj0
- Epic 官方 Scene Graph 文档：https://dev.epicgames.com/documentation/uefn/scene-graph-in-unreal-editor-for-fortnite
- Epic 官方 Getting Started in Scene Graph 文档：https://dev.epicgames.com/documentation/uefn/getting-started-in-scene-graph-in-fortnite
- Epic 官方 Components 文档：https://dev.epicgames.com/documentation/en-us/uefn/components-in-unreal-editor-for-fortnite
- Epic 官方 Entity 文档：https://dev.epicgames.com/documentation/en-us/fortnite/entity
