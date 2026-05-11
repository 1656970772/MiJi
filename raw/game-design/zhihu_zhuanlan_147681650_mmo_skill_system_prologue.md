---
source_url: "https://zhuanlan.zhihu.com/p/147681650"
type: webpage-summary
title: "如何实现一个强大的MMO技能系统——序章"
captured_at: 2026-05-11T20:15:29+08:00
author: "kasan"
topics:
  - MMO
  - 技能系统
  - Gameplay
  - Ability System
  - Buff
  - Projectile
  - AOI
  - 战斗系统
  - 游戏架构
  - Unity
  - Unreal Engine
related_source:
  - "https://zhuanlan.zhihu.com/p/148077453"
  - "https://zhuanlan.zhihu.com/p/149704315"
  - "https://zhuanlan.zhihu.com/p/150812545"
  - "https://zhuanlan.zhihu.com/p/155837535"
  - "https://zhuanlan.zhihu.com/p/158703076"
  - "https://zhuanlan.zhihu.com/p/161501648"
  - "https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools"
  - "https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools/Scripting"
  - "https://dev.epicgames.com/documentation/unreal-engine/using-gameplay-abilities-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine"
capture_scope: "摘要型资料卡；未全文转存"
---

# MMO技能系统序章资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《如何实现一个强大的MMO技能系统——序章》，链接为 `https://zhuanlan.zhihu.com/p/147681650`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `kasan`，作者简介为 `programmer`。
- 文章收录于专栏 `游戏开发中的GamePlay技术`，专栏链接为 `https://zhuanlan.zhihu.com/c_1253986063259426816`。
- 页面结构化数据中显示创建时间为 `2020-06-12 01:17:01 +08:00`，更新时间为 `2020-07-18 22:54:15 +08:00`。
- 截取时页面结构化数据中显示约 `484` 赞同、`8` 评论。
- 页面话题包括 `游戏开发`、`Unity（游戏引擎）`、`虚幻 4（游戏引擎）`。
- 文章定位为系列序章，作者明确将后续模块拆为：`AOI系统`、`技能（Ability）`、`Buff`、`子弹（Projectile）`、`特效`、`运动`、`动画`。

## 系列入口

- `AOI`：`https://zhuanlan.zhihu.com/p/148077453`
- `技能`：`https://zhuanlan.zhihu.com/p/149704315`
- `BUFF`：`https://zhuanlan.zhihu.com/p/150812545`
- `子弹`：`https://zhuanlan.zhihu.com/p/155837535`
- `特效`：`https://zhuanlan.zhihu.com/p/158703076`
- `运动`：`https://zhuanlan.zhihu.com/p/161501648`

## 外部可核验来源

- Valve Developer Community 的 `Dota 2 Workshop Tools` 页面说明，Dota 2 Workshop Tools 可用于制作物品、Steam Workshop 内容以及自定义游戏模式。来源：`https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools`。
- Valve Developer Community 的 `Dota 2 Workshop Tools/Scripting` 页面说明，脚本可控制游戏模式事件、游戏规则、技能、英雄互动、AI 等内容。来源：`https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools/Scripting`。
- Epic 官方 `Using Gameplay Abilities in Unreal Engine` 文档说明，Gameplay Ability 可表达技能做什么、消耗什么、何时可用，并支持多阶段任务、动画、特效音效、输入分支、网络复制、客户端预测等。来源：`https://dev.epicgames.com/documentation/unreal-engine/using-gameplay-abilities-in-unreal-engine`。
- Epic 官方 Gameplay Effects 文档说明，Gameplay Effect 可改变属性、承载临时 buff/debuff、持续效果、叠层，并通过 Gameplay Cue 管理粒子或声音等表现效果。来源：`https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine`。

## 核心问题

这篇序章的核心不是讲某个具体技能怎么写，而是提出一个工程边界：MMO 技能系统不是单个“释放技能函数”，而是战斗体验的中心系统。它要同时承载策划扩展、策略变化、目标选择、状态叠加、飞行物、表现反馈、运动控制、动画时序，以及客户端与服务器之间的同步和裁决。

我的判断：它更像一张技能系统系列的“模块地图”。单篇技术细节不多，但适合放在 `GamePlay / MMO / 技能系统 / 战斗架构` 分类下，作为后续深入 AOI、Ability、Buff、Projectile 等文章的入口。

依据：知乎原文列出了技能系统涉及的模块，并把 War3、Dota2、魔兽世界、守望先锋等案例作为技能系统复杂度与扩展性的背景；Valve 与 Epic 官方资料也能佐证“技能、效果、状态、脚本、表现、网络”这些概念在现代游戏工具链中确实是可独立建模的核心能力。

## 模块地图

### AOI系统

AOI 决定“谁能看到谁、谁能影响谁、服务器需要同步谁”。在 MMO 技能系统里，它不仅是视野裁剪，也会影响目标筛选、群体技能命中、仇恨范围、子弹同步和表现广播。

我的判断：如果没有清晰 AOI，技能系统很容易在小房间 Demo 中可用，但在多人同屏、野外战斗或大型 Boss 战中变成性能和一致性问题。

### 技能 Ability

Ability 层应负责释放条件、目标选择、消耗、冷却、阶段、打断、命中、失败原因，以及把伤害、Buff、Projectile、特效、动画等子模块串联起来。

可迁移到 UE 时，可对照 Gameplay Ability：技能对象不仅表达行为，还要处理成本、CD、动画、输入、复制和预测。来源：Epic `Using Gameplay Abilities in Unreal Engine`。

### Buff

Buff 不只是“身上挂一个状态”。它通常还需要处理持续时间、层数、刷新规则、免疫规则、属性修改、触发器、来源归属、驱散、死亡清理、表现提示等。

可迁移到 UE 时，可对照 Gameplay Effect：临时 buff/debuff、持续效果、属性修改和叠层都是 Gameplay Effect 的核心职责。来源：Epic Gameplay Effects 文档。

### 子弹 Projectile

Projectile 层处理飞行物、弹道、速度、追踪、碰撞、生命周期、命中回调和表现同步。它和 Ability 的关系是：技能决定“何时产生什么子弹”，Projectile 负责“这个实体如何飞、如何判定、如何结束”。

我的判断：在 MMO 中，Projectile 尤其要注意服务器权威、延迟补偿和表现预测，否则玩家看到的命中与服务器判定容易不一致。

### 特效

特效层承载视觉、音频、受击反馈、相机震动、地面提示、范围预警和 UI 提示。它不应和核心判定硬绑定，否则会让服务端逻辑、客户端表现和策划配置互相污染。

可迁移到 UE 时，可对照 Gameplay Cue：它是 GAS 中管理粒子或声音等装饰效果的一种方式。来源：Epic Gameplay Effects 文档。

### 运动

运动层包含冲刺、击退、拉扯、位移、浮空、定身、根运动、路径约束和服务器校正。它既影响技能表现，也直接影响命中判定与反作弊。

我的判断：位移技能是技能系统和角色控制器最容易纠缠的部分，应尽早定义谁拥有运动权威、谁负责表现预测、谁负责回滚或校正。

### 动画

动画层处理前摇、后摇、取消窗口、动作段、蒙太奇、受击、硬直、连招和判定帧。技能释放不是瞬间函数，很多战斗手感来自动画时间轴和判定时间轴的对齐。

我的判断：动画不能只当表现资源，它实际上是 Ability 执行状态机的一部分。好的技能系统需要能让策划清晰表达“第几帧扣资源、第几帧出手、第几帧允许取消、第几帧生成特效或子弹”。

## 为什么作者提 War3 和 Dota2

原文把 War3 与 Dota2 放在前言里，是为了强调“编辑器 + 脚本 + 数据驱动”的技能生态价值。一个强技能系统的真正魅力，不只是引擎程序能写复杂逻辑，而是非核心引擎代码也能安全地组合、验证和扩展玩法。

Valve 官方资料能佐证这个背景：Dota 2 Workshop Tools 面向自定义游戏模式；其脚本文档也说明脚本可以控制技能、英雄互动、游戏规则和 AI。来源：Valve Developer Community。

我的判断：这也是本系列对 MMO 项目最有价值的地方。MMO 技能需求变化极快，如果所有技能都靠程序逐个硬编码，系统会很快失去可维护性；如果 Ability、Buff、Projectile、Effect、Motion、Animation 都有清晰数据边界，策划扩展才有空间。

## 与 UE 和 Unity 的迁移

UE 项目可以把本篇模块地图和 GAS 做概念对照：`Ability` 对应 Gameplay Ability，`Buff/状态/属性修改` 对应 Gameplay Effect 与 Attribute，`特效提示` 对应 Gameplay Cue。但 MMO 项目仍需自己认真处理服务器权威、AOI、跨服或分线、延迟补偿、技能日志和反作弊。

Unity 项目可以采用数据驱动方案：用 ScriptableObject 或外部表描述技能配置，用运行时执行器处理阶段和事件，用 Buff/Effect pipeline 处理状态与属性修改，用 Projectile 系统处理飞行物，用表现层订阅事件播放特效与动画。

我的判断：不要把这篇只归到 UE 或 Unity。它讲的是技能系统架构边界，属于通用 GamePlay 资料；引擎只是落地工具。

## 后续检索关键词

- 中文：`MMO 技能系统`、`技能系统 架构`、`Ability Buff Projectile AOI`、`Dota2 技能脚本`、`War3 技能编辑器`、`GAS 技能系统`、`Gameplay Cue`
- English: `MMO ability system architecture`, `gameplay ability system buff projectile`, `Dota 2 Workshop Tools scripting abilities`, `Unreal Gameplay Ability System`, `ability buff projectile combat system`

## 来源

- 用户提供材料：知乎专栏文章《如何实现一个强大的MMO技能系统——序章》，`https://zhuanlan.zhihu.com/p/147681650`。
- Valve Developer Community：`Dota 2 Workshop Tools`，`https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools`。
- Valve Developer Community：`Dota 2 Workshop Tools/Scripting`，`https://developer.valvesoftware.com/wiki/Dota_2_Workshop_Tools/Scripting`。
- Epic Developer Community：`Using Gameplay Abilities in Unreal Engine`，`https://dev.epicgames.com/documentation/unreal-engine/using-gameplay-abilities-in-unreal-engine`。
- Epic Developer Community：`Gameplay Effects for the Gameplay Ability System in Unreal Engine`，`https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine`。
