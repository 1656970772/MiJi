---
source_url: "https://zhuanlan.zhihu.com/p/2035865763496641970"
type: webpage-summary
title: "UF2025(Orlando)—UMG 动态界面架构：Viewmodel 在 Fortnite 中的工程化实践"
captured_at: 2026-05-11T18:01:34+08:00
author: "wlxklyh"
column: "UF2025(Orlando)"
topics:
  - UE5
  - UMG
  - MVVM
  - ViewModel
  - Fortnite UI
related_source:
  - "https://www.youtube.com/watch?v=xOTZ-DVNc9U"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/umg-viewmodel-for-unreal-engine"
capture_scope: "摘要型资料卡；未全文转存"
---

# UMG ViewModel / MVVM 动态界面架构资料卡

## 来源事实

- 来源文章：知乎专栏《UF2025(Orlando)—UMG 动态界面架构：Viewmodel 在 Fortnite 中的工程化实践》。
- 作者与专栏：作者为 `wlxklyh`，收录于知乎专栏 `UF2025(Orlando)`。
- 编辑时间：知乎页面显示编辑于 `2026-05-09 13:36・广东`。
- 文章主题标签：`UE5`。
- 原始演讲：文章说明基于 Unreal Fest Orlando 2025 演讲 `Powering Dynamic UI with Viewmodels` 整理。
- 演讲者：文章标注演讲者为 James Alman，来自 Epic Games，职位为 Fortnite Lead Technical UI Designer。
- 视频链接：文章提供的源视频为 `https://www.youtube.com/watch?v=xOTZ-DVNc9U`。
- 官方文档佐证：Epic 官方 `UMG Viewmodel` 文档说明 Viewmodel 用于把 UI 与数据连接起来，提供 Viewmodel asset 与 View Binding；文档也提示该功能为 Beta，需要谨慎用于出货项目，并需要启用 `UMG Viewmodel` 插件。

## 核心问题

这篇资料讨论的是：在 UE 的 UMG 中，如何用 MVVM / ViewModel 把 UI 表现层和数据层解耦，让长期运营项目可以频繁改 UI，而不让每次改版都牵动工程代码。

文章的主线不是“MVVM 概念科普”，而是 Fortnite 这类 UI 高频迭代项目里，Tech UI Designer 如何在 Blueprint 和 UMG 工作流内，把 ViewModel、View Binding、Conversion Function 和命名规范组合成可维护的生产流程。

## 一句话结论

我的判断：这篇资料的核心价值是说明 `ViewModel` 不是“给 Widget 取数据的工具”，而是“围绕业务语义组织 UI 数据的中间层”。一旦 VM 按语义拆得足够清楚，同一批数据可以驱动多个 View，Widget 也可以被重构、换皮、复用，而不必频繁改工程层。

## 工作流拆解

### 1. ViewModel 的定位：不要把代码绑死在 Widget 上

文章把 VM 的作用概括为：把屏幕视图和数据源解耦。过去常见做法是在 Widget 内部直接 Get 玩家状态、游戏状态或管理器数据，再绑定到 TextBlock、Image 等具体元素上。小项目这样够快，但大型项目里 Widget 改名、重组、拆分后，绑定容易断裂，工程侧需要频繁跟着 UI 改版。

官方文档也支持这个方向：Epic 的 UMG Viewmodel 文档说明，Viewmodel 可以让设计师把 UI 字段绑定到 Viewmodel 变量，而程序员负责构建 Viewmodel 和应用代码连接；Viewmodel 更新变量时，会通知绑定它的 Widget 更新。

我的判断：VM 要按“业务语义”设计，而不是按“某个 Widget 当前需要哪些字段”设计。否则短期看似省事，长期会把 VM 也变成另一种 Widget 脚本。

### 2. Fortnite 生产案例：同一份 VM 驱动多个 View

文章举了两个 Fortnite 案例：

- Locker 改版：从左侧导航 + 右侧内容的结构，改为右侧整页滚动浏览。文章强调这次重构几乎不需要工程师介入，原因是代码没有绑死在具体 Widget 结构上。
- Store 的 Jam Tracks：同一批曲库数据既能以商店主页的大瓷砖展示，也能进入列表视图展示。两个 View 布局不同，但共享同一批 VM 数据。

我的判断：这两个案例说明 MVVM 在 UI 高频迭代里的真正收益不是“代码更优雅”，而是“设计师能重构界面”。这对运营型游戏、活动页、通行证、商城、奖励界面尤其有价值。

### 3. ViewModel 的获取方式：重点理解生命周期与所有权

文章提到 Unreal 当前有五种 VM 创建或获取方式：

- `Manual`：由 Blueprint 或代码手动创建、设置 VM，控制力最强。
- `Create Instance`：Widget 就地创建自己的 VM 实例，适合临时、只属于该 Widget 的数据。
- `Global View Model Collection`：把 VM 放到全局集合，通过 Identifier 获取。
- `Property Path`：通过属性路径或函数从已有对象上找到 VM。
- `Resolver`：通过 Resolver asset 自动创建或获取 VM，适合全局单例式数据。

文章说 Fortnite 实际主要使用 `Manual` 和 `Resolver`：有上下文的具体 VM 用 Manual，全局共享的单例 VM 用 Resolver。

我的判断：这背后是所有权问题。UI 架构最怕“谁都能拿、谁都能改、谁也说不清生命周期”的中间状态。Manual 和 Resolver 的边界最清楚：前者局部明确，后者全局明确。

### 4. 绑定方式：Direct Binding、Custom Function、Conversion Function

文章把绑定分成几个层次：

- `Direct Binding`：一个 Widget 属性直接绑定一个 VM 字段，适合单值、简单展示。
- `Custom Function`：当展示逻辑需要判断、拼接或格式化时，在 Widget 内写函数，再绑定到目标属性。
- `Conversion Function`：用于类型转换，例如 bool 转 Visibility、int 转 FText，减少为了小转换而写自定义函数。
- `Custom Conversion Function`：当一个输出依赖多个输入值时，把所有依赖都作为绑定输入传入函数，由函数纯计算并返回目标值。

官方文档也说明，View Bindings 中可以使用 Conversion Functions，把 Viewmodel 变量转换成目标 Widget 属性所需的数据类型；自定义 Conversion Function 需要满足可被 Blueprint 访问、输入与返回值约束等条件。

我的判断：Direct Binding 和 Conversion Function 是“表达简单事实”，Custom Conversion Function 是“表达多值决策”。真正要警惕的是把多值决策藏在单值绑定函数内部。

### 5. 最大陷阱：多值绑定只绑了一个值

文章最有工程价值的部分是一个 Fortnite 真实 bug：状态栏显示 `Locked Page`，但实际页面已经解锁，应显示 `5 to Claim`。

错误做法是：函数只绑定到某一个 VM 字段，函数内部再通过 Blueprint Get 读取其他 VM 字段。问题在于，绑定函数只会在“被绑定的那个字段”变化时重新求值，其他字段变化不会触发函数，于是 UI 卡在旧状态。

正确做法是：把所有参与决策的字段都作为 View Binding 输入传进 Custom Conversion Function。这样任意输入变化都会触发重新计算。

我的判断：这条可以直接写进团队规范：凡是绑定结果依赖多个 VM 字段，所有依赖字段必须显式出现在绑定输入里；不要在绑定函数内部偷偷 Get 其他 VM 字段。

### 6. 绑定方向与反向通信

文章提到常见绑定方向：

- `One Time to Widget`：每个 VM 实例设置一次。
- `One Way to Widget`：VM 到 Widget，Fortnite 最常用。
- `One Way to View Model` 和 `Two-Way`：适合输入框、复选框等用户编辑值需要回写 VM 的场景，但 Fortnite 很少用。

反向通信方面，文章总结三种方式：

- 在 Blueprint 中直接 Set VM 属性，这是 Fortnite 常用路径。
- 调用 VM 提供的函数，需要工程师提前提供接口。
- 使用 One Way to View Model 或 Two-Way 绑定，适合少数输入控件。

我的判断：大多数游戏 UI 更适合单向数据流。用户操作可以通过事件或 Set 写回 VM，但不要让双向绑定成为默认习惯，否则状态来源会变得难追。

### 7. VM 可以作为 UI 状态总线

文章的 Battle Pass 例子里，选中某个 Tile 后，上方标题和背景也需要变化。这些 Widget 不是父子关系，如果互相持引用，耦合会很重。

Fortnite 的做法是把“当前选中项”放到 VM。多个 Widget 监听 VM，不直接知道彼此存在。

我的判断：这很适合迁移到复杂 UI：选中项、焦点项、当前 Tab、当前 Offer、当前 Reward 等 UI 状态都可以放进 VM，让 Widget 通过共享状态协作，而不是通过引用彼此调用。

### 8. 命名与整洁性规范

文章提到 Fortnite 的命名约定：

- `VM_`：绑定用的自定义函数前缀。
- `CF_`：自定义 Conversion Function 前缀。
- 通过 Category 分组，让 View Binding 面板和详情面板更容易检索。
- 跨 Widget 复用的 Conversion Function 放进 Blueprint Function Library。

我的判断：这些规范看似小，但 UMG 绑定列表一长，命名就是生产力。建议把前缀、Category、函数纯度、跨 Widget 复用位置都写进项目 UI 规范。

### 9. UE 5.6 QoL 改进

文章提到 UE 5.6 对 ViewModel 工作流做了几项体验改进：

- 绑定支持复制、粘贴和 Duplicate；
- 在 Hierarchy 面板右键 Widget 可以创建 Widget Binding；
- View Bindings 面板支持展开或折叠 categories / bindings。

我的判断：这类 QoL 对 Tech UI Designer 的影响大于看上去。绑定工作本质上是高频手工工程，减少点击路径会明显降低出错率。

### 10. Reward Track 案例：小而专的 VM 粒度

文章最后用 Fortnite 的 Reward Track 屏幕说明 VM 粒度：

- 同一个 Tile Widget 可以被不同 Pass、不同赛季的 VM 数据“染色”，形成不同视觉风格。
- 一个 Screen Widget 可以服务 Battle Pass、OG Pass、Music Pass、Lego Pass 等不同界面。
- 屏幕内多个 Widget 可以共享同一组 VM，而不是一 VM 一 Widget。
- `Season Pass Category VM` 放分类相关数据，`Item Details VM` 放通用物品信息，`Season Pass Selection VM` 追踪选中项。

我的判断：这是“语义内聚”的最佳例子。VM 不应该围绕 Widget 拆，也不应该做成巨型全能对象；它应该围绕可复用的数据语义拆。

## 可迁移清单

- 建立项目级 UI 架构原则：Widget 只负责表现与轻量交互，业务数据放进 VM。
- 给 VM 拆分设规则：按语义内聚拆，不按屏幕或 Widget 临时需要拆。
- 绑定规范：单值展示用 Direct Binding；类型转换优先 Conversion Function；多值决策必须用 Custom Conversion Function，并显式绑定全部输入。
- 命名规范：`VM_`、`CF_`、Category 分组，跨 Widget 复用函数进入 Blueprint Function Library。
- 状态规范：选中项、焦点项、当前页签等跨 Widget 状态优先放 VM，不让 Widget 互持引用。
- 风险提示：UMG Viewmodel 在官方文档中仍标为 Beta，正式项目使用前要评估版本、平台和团队维护能力。

## 我的判断

这篇资料最适合归入 `虚幻 / UMG / MVVM / ViewModel / UI 架构`。它不是讲“怎么画一个 Widget”，而是讲怎样让大型 UI 在长期运营中保持可重构。

对独立开发者，它的实用版本是：先不要把所有 UI 逻辑都塞进 Widget 蓝图，至少把“数据状态”和“表现 Widget”拆开。

对团队项目，它的实用版本是：把 VM 粒度、绑定方式、Conversion Function、命名前缀、反向通信和 UI 状态归属全部写成规范。这样 UI 改版时，设计师可以在 UMG 内完成更多工作，工程师只需要维护稳定的数据边界。

## 后续检索关键词

- 中文：虚幻 UMG ViewModel、UE MVVM、UMG View Binding、UMG Conversion Function、Fortnite UI 架构、UE 动态界面、UMG 绑定陷阱
- English：Unreal Engine UMG Viewmodel, UE MVVM, View Bindings, Conversion Function, Custom Conversion Function, Fortnite UI, Dynamic UI with Viewmodels, Unreal Fest Orlando 2025

## 来源

- 知乎原文：https://zhuanlan.zhihu.com/p/2035865763496641970
- 源视频：https://www.youtube.com/watch?v=xOTZ-DVNc9U
- Epic 官方 UMG Viewmodel 文档：https://dev.epicgames.com/documentation/en-us/unreal-engine/umg-viewmodel-for-unreal-engine
