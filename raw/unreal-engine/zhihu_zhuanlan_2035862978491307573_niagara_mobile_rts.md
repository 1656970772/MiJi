---
source_url: "https://zhuanlan.zhihu.com/p/2035862978491307573"
type: webpage-summary
title: "UF2025(Orlando)——用 Niagara 驱动移动端 RTS：从粒子单位到距离场雾效"
captured_at: 2026-05-11T18:05:39+08:00
author: "wlxklyh"
column: "UF2025(Orlando)"
topics:
  - UE5
  - Niagara
  - RTS
  - GPU Simulation
  - Mass Framework
related_source:
  - "https://www.youtube.com/watch?v=n1TrBL-BX3I"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/overview-of-mass-gameplay-in-unreal-engine"
  - "https://dev.epicgames.com/documentation/unreal-engine/creating-visual-effects-in-niagara-for-unreal-engine"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/emitter-update-group-reference-for-niagara-effects-in-unreal-engine"
capture_scope: "摘要型资料卡；未全文转存"
---

# Niagara 驱动移动端 RTS 原型资料卡

## 来源事实

- 来源文章：知乎专栏《UF2025(Orlando)——用 Niagara 驱动移动端 RTS：从粒子单位到距离场雾效》。
- 作者与专栏：作者为 `wlxklyh`，收录于知乎专栏 `UF2025(Orlando)`。
- 编辑时间：知乎页面显示编辑于 `2026-05-08 01:26・新加坡`。
- 文章主题标签：`UE5`、`Niagara(特效）`、`即时战略游戏（RTS）`。
- 原始演讲：文章说明基于 `Chasing Waterfalls: Using Niagara to Power a Mobile RTS | Unreal Fest Orlando 2025` 整理。
- 视频链接：文章提供的源视频为 `https://www.youtube.com/watch?v=n1TrBL-BX3I`。
- 演讲信息：文章标注视频来源为 Unreal Engine，公开视频发布日期为 `2025-11-24`，视频时长为 `43 分 17 秒`，演讲者信息中出现 Epic 的 Matt Oztalay / Matt Ostel 表述。
- 官方文档佐证：Epic 官方文档说明 Niagara 是 UE 的视觉特效系统；Mass Entity 是 UE 中面向数据化计算的框架，Mass Gameplay 提供世界表示、生成、LOD、复制和 StateTree 等能力；Niagara 的 `Neighbor Grid 3D` 是可在 simulation stage 中读写 3D 数据数组的数据接口。

## 核心问题

这篇资料讨论的是一个边界实验：把移动端 RTS 中的大量单位重新表示为 Niagara 粒子，让 Niagara 不只负责渲染特效，也承担一部分 GPU 数据处理、局部交互、避让和战争迷雾生成。

它并不主张“用 Niagara 取代正式 RTS 架构”。文章开头就提醒：如果是真实项目做大量实体模拟，Unreal 的 `Mass Framework` 仍然是应优先评估的路线。Niagara 在这里更像一个 GPU 数据平台原型，用来探索“对象数量成为瓶颈时，能否把对象重写为连续数据并交给 GPU 批处理”。

## 一句话结论

我的判断：这篇资料最值得保留的不是“4096 单位”这个数字，而是建模方式的转换：单位从 Actor / Character 变成粒子数据，邻居关系变成网格查询，单位状态变成位打包，战争迷雾变成距离场。它给高密度单位表现和局部交互提供了很好的原型路线，但不应直接承担权威玩法、网络同步和复杂 AI。

## 技术路线拆解

### 1. 为什么不用传统 Actor 堆单位

RTS 单位需要可见、可选择、可移动、可避让、可携带状态。如果每个单位都是 Character，通常会带来 Skeletal Mesh、动画蓝图、碰撞胶囊、导航、组件生命周期和 Actor Tick 的成本。

文章强调，移动端高数量单位演示里，传统对象模型太重。因此演讲把单位塞进一个 Niagara Particle System：单位位置、目标点、生命值、选择状态和单位类型等信息都成为 Niagara 属性、用户参数、Render Target 数据或材质通道中的值。

我的判断：这是典型的数据化建模思路。不是“怎么创建更多对象”，而是“哪些对象特征可以压成连续数据”。

### 2. 架构主线：Niagara as Compute

文章把路线拆成四块：

- `Niagara as Compute`：用 Niagara 表示单位，并在 GPU 上更新部分单位逻辑。
- `Neighbor Grid 3D`：让粒子能高效访问附近粒子。
- `Bitwise Operations`：用位运算把更多状态压进有限通道。
- `Distance Fields`：用粒子位置生成战争迷雾等区域效果。

在这个架构里，CPU 更像指挥层，处理玩家输入、框选、目标点和蓝图交互；GPU 负责批量处理粒子位置更新、邻居查询、避让和部分视觉数据生成。

我的判断：如果正式落地，应把 Niagara 定位为“高密度显示与局部交互层”，而不是完整玩法模拟层。Mass、Gameplay 系统或自研 ECS 更适合保有权威状态。

### 3. CPU / GPU 数据交换：数组与 Render Target

文章提到 Niagara 系统使用用户参数、数组和 Texture Render Target 做数据交换：

- CPU 侧生成选择框、目标点、命令等输入；
- Niagara 读取输入，在模拟阶段更新粒子；
- Niagara 把位置或结果写回数组 / Render Target；
- Blueprint 或材质再消费这些结果。

演示中使用 RGBA 32F Render Target 保存粒子相关信息，并通过粒子索引把数组位置和 Render Target 像素位置关联起来。

我的判断：这是可行但危险的地方。GPU 到 CPU 的读回频率需要非常克制，少量交互状态可以读回，但大量同步读回可能抹掉 GPU 并行收益。

### 4. 选择与移动：让粒子可交互

文章中的框选思路是把粒子位置投影到屏幕空间，再和框选矩形做判断。被框中的粒子单位会被标记为选中，新目标点会写入对应单位的目标数据。

关键不在框选算法本身，而在数据所有权：如果位置在 Niagara 内更新，CPU 或输入系统必须有足够的位置数据，才能判断玩家选中了哪些单位。

我的判断：适合把“视觉反馈”和“局部交互”放在 GPU / Niagara 路径中；真正影响游戏规则的选中结果，仍应谨慎同步回权威逻辑层。

### 5. Neighbor Grid 3D：局部邻居查询

单位避让需要查找附近单位。如果每个粒子扫描所有粒子，复杂度很快失控。文章介绍的做法是使用 Niagara 的 `Neighbor Grid 3D`：先把粒子索引写入网格单元，再让粒子只访问附近格子的候选粒子。

官方 Niagara 模块参考也说明，`Neighbor Grid 3D` 是 Data Interface，可配合 simulation stages 读写 3D 数据数组，并在 simulation stage 中迭代访问。

我的判断：Neighbor Grid 的价值在于把全局搜索变成局部搜索。落地时要重点调网格范围、单元大小、每格最大粒子数和查询半径；演示参数不能直接当项目模板。

### 6. 位打包：用有限通道表达更多状态

文章提到，Niagara 粒子有很多属性，但当状态需要传给材质、实例数据或渲染通道时，通道数量有限。布尔值、小范围整数、枚举值、低精度状态可以通过位运算打包进一个数值通道。

概念示意：

```cpp
// [我的判断] 概念示意，不是原演讲完整代码。
packed =
  (unitType & 0xF)
  | ((isSelected ? 1 : 0) << 4)
  | ((healthBucket & 0xF) << 5);
```

我的判断：位打包适合高数量、低精度、视觉或局部交互状态；不适合权威游戏规则、网络同步、可回放战斗或需要强可读性的玩法字段。

### 7. 距离场战争迷雾

文章介绍的距离场思路是：把粒子世界位置写入位置 Render Target，再通过材质或蓝图绘制到另一个 Render Target，生成距离场；战争迷雾材质采样距离场，把单位附近区域处理成可见或过渡区域。

它特别提醒：`Draw Material to Render Target` 的上下文默认不知道你的世界空间，必须把 0 到 1 的 UV 空间转换到目标世界空间，才能让采样点、单位位置和场值对齐。

概念关系：

```cpp
// [我的判断] 概念示意，不是原演讲完整代码。
distanceValue = distance(sampleWorldXY, unitWorldXY) - radius;
field = min(field, distanceValue);
```

我的判断：距离场是这套方案里最需要单独做性能预算的一块。它可能比单位粒子模拟更贵，尤其是 Render Target 分辨率过高、每帧全量绘制或采样范围过大时。

### 8. 性能结果与证据边界

文章记录的结果是：演示在 Galaxy S10 上与约 4096 个单位交互，主要成本来自 Distance Field Rendering，而不是单位本身的粒子表示。

这个结论只能说明该演示配置、设备和实现方式下的表现，不能外推成“Niagara RTS 在移动端可稳定支撑 4096 单位”。

我的判断：真正要带走的是瓶颈定位：当单位粒子化之后，原本的 Actor 成本下降了，但 Render Target、距离场、读回同步、邻居网格和材质解码会变成新的成本中心。

## 适合与不适合

### 适合

- 单位数量高，但单位行为相对简单。
- 单位视觉可以接受精灵、材质或低成本实例化表达。
- 交互集中在选择、移动、局部避让和状态显示。
- 游戏规则不要求每个单位都是独立 Actor。
- 目标平台 CPU 对象数量敏感，尤其是移动端。

### 不适合

- 每个单位需要复杂 AI、复杂动画蓝图、独立物理或精确碰撞。
- 网络同步要求每个单位状态权威、可预测、可回放。
- 玩法依赖 Gameplay Ability、组件化状态机、Actor 生命周期回调。
- 团队缺少 Niagara、材质、Render Target 和 GPU 调试经验。

## 可迁移清单

- 先做最小闭环：目标点输入 → 粒子移动 → 位置读回 / 显示 → 框选反馈。
- 明确数据方向：CPU 到 Niagara、Niagara 到 CPU、GPU 内部消费三类数据分开设计。
- 把 Render Target 可视化、Neighbor Grid 参数可视化、粒子密度统计和状态解码调试做在原型早期。
- 不要过早位打包；先确认通道数量真的是瓶颈。
- 把距离场作为单独性能项目处理：控制绘制范围、分辨率、更新频率和采样次数。
- 正式项目优先评估 Mass / Gameplay / ECS 承担权威模拟，Niagara 承担高密度视觉与局部反馈。

## 我的判断

这篇资料最适合归入 `虚幻 / Niagara / GPU 数据平台 / 移动端 RTS 原型`。

对原型和技术验证，它非常有启发：可以用 Niagara 快速探索“海量单位表现 + 简单交互 + 战争迷雾”的视觉路线。

对正式项目，我会把它拆成分层架构：Mass 或自研 ECS 管权威状态，Gameplay 层处理规则，Niagara 负责高密度显示、局部避让、框选反馈和雾效可视化。这样既保留 GPU 批处理优势，又不把核心玩法压进一个难调试的特效系统里。

## 后续检索关键词

- 中文：虚幻 Niagara RTS、Niagara GPU 计算、Niagara Neighbor Grid 3D、UE 移动端 RTS、Niagara Render Target、战争迷雾 距离场、UE Mass Framework、粒子单位
- English：Unreal Niagara mobile RTS, Niagara as compute, Neighbor Grid 3D, Niagara Render Target, GPU particles RTS, distance field fog of war, Mass Framework Unreal, Chasing Waterfalls Niagara

## 来源

- 知乎原文：https://zhuanlan.zhihu.com/p/2035862978491307573
- 源视频：https://www.youtube.com/watch?v=n1TrBL-BX3I
- Epic 官方 Mass Gameplay 文档：https://dev.epicgames.com/documentation/en-us/unreal-engine/overview-of-mass-gameplay-in-unreal-engine
- Epic 官方 Niagara 文档入口：https://dev.epicgames.com/documentation/unreal-engine/creating-visual-effects-in-niagara-for-unreal-engine
- Epic 官方 Niagara 模块参考：https://dev.epicgames.com/documentation/en-us/unreal-engine/emitter-update-group-reference-for-niagara-effects-in-unreal-engine
