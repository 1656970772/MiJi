---
source_url: "https://zhuanlan.zhihu.com/p/616999277"
type: webpage-summary
title: "【UE5 模拟交互篇】（一）可交互流体风场实现"
captured_at: 2026-05-11T20:02:50+08:00
author: "芝麻拌葱花"
topics:
  - Unreal Engine
  - UE5
  - Niagara
  - GPU Simulation
  - Fluid Simulation
  - Wind Field
  - Grid3D
  - Render Target
  - Stable Fluids
  - 交互风场
related_source:
  - "https://www.dgp.toronto.edu/public_user/stam/reality/Research/pdf/GDC03.pdf"
  - "https://www.josstam.com/publications"
  - "https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview?application_version=5.6"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/NiagaraDataInterfaceGrid3DCollection?application_version=4.27"
capture_scope: "摘要型资料卡；未全文转存"
---

# UE5 可交互流体风场资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《【UE5 模拟交互篇】（一）可交互流体风场实现》，链接为 `https://zhuanlan.zhihu.com/p/616999277`。
- 使用 Codex Chrome 插件读取页面结构化数据后，文章作者为 `芝麻拌葱花`，作者简介为 `在工地打灰的技术美术`。
- 文章收录于专栏 `模拟交互篇`，专栏链接为 `https://zhuanlan.zhihu.com/c_1779456485464461312`。
- 文章结构化数据中显示创建时间为 `2023-05-07 21:45:43 +08:00`，更新时间为 `2023-05-08 22:19:03 +08:00`。
- 截取时页面结构化数据中显示约 `260` 赞同、`23` 评论。
- 文章目录包括：`原理介绍`、`思路分析`、`解析论文`、`实现过程`、`基础搭建`、`AddSource(外力项)`、`Diffuse(扩散项)`、`Advect(平流项)`、`Project(投射项)`、`边界问题`、`采样风场`。
- 文章引用的主要理论来源是 Jos Stam 的 `Real-Time Fluid Dynamics for Games`，并在参考中列出 NVIDIA GPU Gems 的 `Chapter 38. Fast Fluid Dynamics Simulation on the GPU`。

## 外部可核验来源

- Jos Stam 的 GDC 2003 论文《Real-Time Fluid Dynamics for Games》说明：该文提供面向游戏引擎的简洁实时流体求解器，基于 Navier-Stokes 方程，但强调视觉质量、稳定性和速度，而不是严格物理精度。
- Jos Stam 个人出版物页面确认《Real-Time Fluid Dynamics for Games》属于 GDC 2003 Conference proceedings，并说明该文包含约 100 行可读 C 代码实现。
- NVIDIA GPU Gems 第 38 章说明：快速稳定的 GPU 流体模拟基于 Stam 1999 的 Stable Fluids 方法，流体速度场可表示在网格上，Navier-Stokes 可拆成 advection、pressure、diffusion、external forces 等项。
- Unreal Engine 官方 Fluid Simulation Overview 说明：UE 的流体模拟可使用网格表示气体，液体使用粒子与网格混合；`Grid 2D Collection` 与 `Grid 3D Collection` 是用于在 2D/3D 网格中存储命名属性的数据接口；Niagara Simulation Stages 用于让流体算法的每一步在进入下一步前处理完整网格。
- Unreal Python API 中 `NiagaraDataInterfaceGrid3DCollection` 页面列出了 `num_attributes`、`num_cells`、`world_b_box_size`、`render_target_user_parameter` 等属性，可佐证文章中用 Grid3D Collection 存速度属性、通过 RT 输出调试数据的实现方向。

## 核心问题

交互风场需要回答两个问题：

- 玩家或角色如何把运动、碰撞、挥动等外力写入风场。
- 植被、烟雾、布料、粒子等系统如何在任意位置采样这个风场，并把它转换成受力或方向。

文章选择欧拉视角：把角色周围空间切成规则网格，每个格子存速度向量。角色碰撞写入局部外力，随后用 Stable Fluids 的几个步骤让速度场扩散、平流、投射，再把结果导出到 Render Target，供其他 Niagara 系统采样。

我的判断：这篇文章不是在做高精度 CFD，而是在做“游戏交互风的局部速度场”。它的目标是让风与粒子、植被、烟雾、布料等效果产生可信联动，而不是模拟真实空气动力学。

依据：知乎原文明确把风场框限制在角色附近，并参考《战神》局部风场做法；Stam 论文也明确强调游戏场景中更看重视觉质量、速度和稳定性。

## 原理拆解

### 1. 拉格朗日与欧拉视角

文章先区分两种介质模拟视角：

- 拉格朗日视角：把介质看成粒子或微小网格集合，粒子随介质移动。
- 欧拉视角：把空间切成固定网格，记录每个格子的状态、输入输出和邻域影响。

对于游戏中的风力查询，欧拉视角更直接：任意位置只要映射到格子，就能读出平均速度。拉格朗日视角若要知道某点风力，需要统计单位时间经过该点的粒子速度，工程上不适合实时交互风查询。

我的判断：欧拉网格特别适合做“可被很多系统采样”的共享风场，因为采样接口稳定。粒子、植被、布料不需要知道谁制造了风，只需要按位置查询速度。

### 2. Stable Fluids 四项

文章把 Navier-Stokes/Stable Fluids 的速度更新拆成四个工程步骤：

- `AddSource`：把角色碰撞或外部输入作为速度源写入网格。
- `Diffuse`：让速度在邻域中扩散，模拟粘滞造成的平滑传播。
- `Advect`：让速度沿速度场自身移动，文章使用半拉格朗日回溯思想。
- `Project`：求压力场并减去压力梯度，使速度场满足近似无散条件，增强稳定性。

NVIDIA GPU Gems 第 38 章也把不可压 Navier-Stokes 的右侧项拆成 advection、pressure、diffusion、external forces，并说明速度场是流体模拟最重要的量。

我的判断：对技术美术来说，这四项的意义比公式更重要。`AddSource` 决定交互入口，`Diffuse` 决定风场软硬，`Advect` 决定风是否“带着自己走”，`Project` 决定场是否稳定、不明显膨胀或塌缩。

### 3. Niagara 基础搭建

文章实现路线：

- 创建空 Niagara 发射器，`Sim Target` 改为 `GPUCompute Sim`，并设置 `Fixed Bounds`。
- 在 User Exposed 中创建 `Texture Render Target` 变量 `WindFieldRT`，作为输出调试 RT。
- 创建 `int` 类型 `resolution`，控制 Render Target 分辨率。
- 在 Emitter Attribute 中创建 `Grid3D Collection` 变量 `VelocityGrid`，`Num Attribute` 设置为 3，用于保存速度的 XYZ 三个分量。
- 创建 `Render Target 2D` 变量 `RT_WindField`，格式设为 `RGBA 16f`，并绑定 `WindFieldRT` 用户参数。
- 通过 `NMS_SetResolution` 设置 `VelocityGrid` 网格数和 RT 分辨率。
- 通过 `NMS_ExportToRT` 把 Grid3D 写入 RT；文章采用 Z 轴切片后平铺到 Y 轴的方式，所以 RT 分辨率为 `(x, y*z)`。

文章还提醒：每一项计算需要放在独立 Simulation Stage 中，因为同一个 Stage 可视作并行计算，输入 Grid 是上一帧结果。

我的判断：这一步的关键不是 Niagara 节点名字，而是数据布局。VelocityGrid 是模拟状态，RT_WindField 是对外发布的采样资源；把 3D 网格切片打包进 2D RT 是为了让后续材质或 Niagara 系统更容易读。

## 实现步骤拆解

### 1. AddSource 外力项

文章把碰撞产生的作用力作为主要外力。流程是：

1. 从 Grid3D 索引还原单元格世界位置。
2. 用风场组件世界位置、模拟范围 `Size` 和分辨率 `res` 算出 `cellWorldPosition`。
3. 输入 `PhysicsAsset` 和 `cellWorldPosition` 到 `Get Closest Element`，取得最近距离和最近点速度。
4. 当 `Closest Distance` 为负时，认为位置在 PhysicsAsset 内部，只取相交部分的速度作为外力源。
5. 将该速度与上一帧速度叠加。

我的判断：这是一种“碰撞体把运动速度注入场”的做法。它适合角色身体、武器挥动、怪物冲撞等局部扰动，但对远场风、爆炸冲击波或扇形风压，需要额外的 source 形状和衰减函数。

### 2. Diffuse 扩散项

文章按每个格子和周围六邻域取值，用迭代形式逼近扩散求解。核心直觉是：扩散越强，速度场越平滑，局部尖锐外力会更快向周围传播。

我的判断：扩散参数是风场“粘”和“软”的美术旋钮。数值过低会让角色运动留下尖锐局部速度块；过高则会把方向性抹平，让交互反馈变钝。

### 3. Advect 平流项

文章在当前格子处读取速度 `(U,V,W)`，沿速度场回溯到上一时刻位置，再对周围八个采样点做三线性插值，得到当前格子的速度。

这对应 Stam 论文和 GPU Gems 中常见的半拉格朗日思想：问“当前格子的流体上一时刻来自哪里”，再从旧场中插值取值。

我的判断：平流项决定风场是否有“拖尾”和“流动记忆”。没有平流时，外力更像局部噪声或扩散贴图；有平流后，角色动作会留下可被带动的速度结构。

### 4. Project 投射项

文章把投射分为三步：

1. 计算当前速度场散度。
2. 根据亥姆霍兹-霍奇分解，用迭代方式求压力场。
3. 从速度场中减去压力梯度，得到更稳定的速度场。

NVIDIA GPU Gems 第 38 章说明，Helmholtz-Hodge Decomposition 可把向量场分解成无散向量场和标量场梯度，压力修正可以让速度场满足不可压缩约束。

我的判断：Project 是很多视觉向 Stable Fluids 实现里最容易被低估的一步。只做 AddSource/Diffuse/Advect 也能看到“动”，但场容易出现不稳定的膨胀、收缩或能量堆积；Project 的价值是让风场更像连续介质。

### 5. 边界与跟随角色

文章指出固定模拟框会遇到角色超出边界的问题。最简单但最贵的方案是让模拟框覆盖整张地图；文章采用更实际的局部方案：

- 风场模拟框跟随角色移动。
- 风场信息每帧按角色运动反方向偏移。
- 只保留角色附近范围内的交互风效果。
- 超出范围的区域不参与计算，必要时用噪声或默认速度补足。

我的判断：这是整篇最游戏工程化的一点。全世界 3D 风场很容易把显存和计算炸掉；跟随角色的局部场把问题从“全地图物理”改成“角色周围交互缓存”，更符合实时游戏需求。

### 6. 采样风场

文章额外定义一张 `1x3` 的 `RT_Data`，把风场组件位置、模拟框大小、分辨率写入三个像素中。后续其他 Niagara 系统读取这三个像素，解析出实时参数，再用 `RT_WindField` 采样速度。

文章实现了：

- `NFS_ParseWindFieldData`：读取 `RT_Data` 三个像素，解析位置、大小、分辨率。
- `NMS_WindForce`：在其他 Niagara 系统中采样风场 RT，把速度作为外力驱动粒子。
- Fountain 粒子和箭头矩阵：一个展示粒子受风影响，一个可视化风向。

我的判断：`RT_Data` 相当于一个非常小的 GPU 侧结构体。它把“风场在哪里、范围多大、分辨率是多少”发布出去，让其他 Niagara 系统只依赖 RT，不依赖蓝图里的一堆松散参数。

## 适合场景

- 角色经过草丛、烟雾、尘土、雪、叶片时产生局部风扰动。
- 武器挥动、冲刺、技能释放带动粒子方向。
- 需要把同一个交互风源共享给多个 Niagara 系统。
- 想用 GPU 侧局部网格承载交互状态，避免大量 CPU 粒子或全地图风场。
- 技术美术原型验证：先做局部速度场，再逐步接植被、布料、烟雾。

## 不适合场景

- 需要严格空气动力学、压力传播或工程级 CFD 的模拟。
- 需要全地图全时段持续高精度风场的开放世界。
- 对 GPU Niagara、Simulation Stage、Grid 数据接口不熟悉，且缺少调试视图的项目。
- 低端平台上同时叠加大量 3D Grid、RT 读写和多套粒子系统的场景。
- 需要网络同步确定性的多人玩法逻辑。视觉风场可以本地表现，玩法判定不宜直接依赖它。

## 可迁移清单

- 先做 `VelocityGrid` debug view，再接粒子或植被，避免在黑箱里调风。
- 把 AddSource、Diffuse、Advect、Project 拆成独立 Simulation Stage，便于调试和开关。
- 为扩散强度、粘度、投射迭代次数、风场衰减、最大速度做用户参数。
- 风场框跟随角色时，必须处理上一帧位置和当前帧位置的相对偏移。
- 对外采样协议要稳定：RT 数据布局、切片方式、坐标归一化、边界 clamp 都应写成注释或文档。
- 为粒子、植被、烟雾、布料分别做采样强度和响应曲线，不要共用一套硬参数。
- 如果用于生产，需要补 profiling：Grid 分辨率、Simulation Stage 次数、RT 格式、GPU 带宽和平台差异都要测。

## 我的判断

这篇资料适合归入 `Unreal Engine / Niagara / GPU Simulation / 交互风场 / Stable Fluids`。

它最值得保存的不是某一段节点截图，而是完整的数据流：角色运动写入局部 Grid3D 速度场，Stable Fluids 步骤维护场的连续性和稳定性，Render Target 把风场发布给其他 Niagara 系统采样。这个结构很适合扩展到风与植被、烟雾、布料、粒子特效的统一交互入口。

边界也要明确：它是视觉特效风场，不是严格物理风场；是局部角色邻域模拟，不是全世界风模拟；是 GPU 侧表现系统，不应未经处理就承担关键玩法逻辑。

## 后续检索关键词

- 中文：`UE5 Niagara 交互风场`、`Niagara Simulation Stage Grid3D`、`UE5 Stable Fluids`、`Niagara Render Target 风场`、`交互风 植被 烟雾 布料`
- English: `UE5 Niagara interactive wind field`, `Niagara Grid3D Simulation Stage`, `Stable Fluids Unreal Engine`, `GPU fluid simulation render target`, `wind field sampling Niagara`

## 来源

- 用户提供材料：知乎专栏《【UE5 模拟交互篇】（一）可交互流体风场实现》，`https://zhuanlan.zhihu.com/p/616999277`。
- Jos Stam：`Real-Time Fluid Dynamics for Games` PDF，`https://www.dgp.toronto.edu/public_user/stam/reality/Research/pdf/GDC03.pdf`。
- Jos Stam 个人出版物页面，`https://www.josstam.com/publications`。
- NVIDIA GPU Gems：`Chapter 38. Fast Fluid Dynamics Simulation on the GPU`，`https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-38-fast-fluid-dynamics-simulation-gpu`。
- Unreal Engine 官方文档：`Fluid Simulation in Unreal Engine - Overview`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/fluid-simulation-in-unreal-engine---overview?application_version=5.6`。
- Unreal Engine Python API：`NiagaraDataInterfaceGrid3DCollection`，`https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/NiagaraDataInterfaceGrid3DCollection?application_version=4.27`。
