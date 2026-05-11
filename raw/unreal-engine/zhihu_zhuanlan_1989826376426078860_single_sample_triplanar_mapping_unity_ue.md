---
source_url: "https://zhuanlan.zhihu.com/p/1989826376426078860"
type: webpage-summary
title: "TA实践分享：简单且只采样一次的三面映射(Unity+UE）"
captured_at: 2026-05-11T19:55:37+08:00
author: "彦青"
topics:
  - Unreal Engine
  - Unity
  - Triplanar Mapping
  - 三面映射
  - Shader
  - HLSL
  - 材质节点
  - UV 生成
  - 单次采样
related_source:
  - "https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-step"
  - "https://dev.epicgames.com/documentation/en-us/unreal-engine/texturing?application_version=4.27"
  - "https://docs.unity3d.com/2019.4/Documentation/Manual/SL-PropertiesInPrograms.html"
capture_scope: "摘要型资料卡；未全文转存"
---

# 单次采样三面映射资料卡

## 来源事实

- 用户提供来源：知乎专栏文章《TA实践分享：简单且只采样一次的三面映射(Unity+UE）》，链接为 `https://zhuanlan.zhihu.com/p/1989826376426078860`。
- 使用 Codex Chrome 插件读取页面后，页面标题显示为《TA实践分享：简单且只采样一次的三面映射(Unity+UE） - 知乎》。
- 页面作者显示为 `彦青`，作者简介片段为 `北京电影学院 艺术设计硕士`。
- 页面底部显示 `编辑于 2025-12-31 23:22・浙江`。
- 页面标签包含 `TA技术美术`、`UE引擎`。
- 文章正文说明：传统三面映射需要三次纹理采样，并把采样结果进行两次 `lerp` 混合；本文方案通过主导面选择把最终采样压到一次。
- 文章列出的三种 UV 模式为 `World Position UV`、`Object Position UV`、`Mesh UV0（支持模型缩放调整）`。
- 文章提醒：主导面遮罩需要在片元着色器中计算；如果在顶点阶段计算再插值到片元阶段，遮罩会被插值，导致 UV 混合错误。
- 文章提供的 Unity HLSL 核心逻辑使用 `step(normal.y, normal.x)`、`step(normal.z, normal.x)` 等表达式生成 one-hot 主导面遮罩，并最终只执行一次 `tex2D(_MainTex, finalUV)`。
- 文章 UE 部分以材质节点图示为主，文字小节包括 `材质节点总览`、`遮罩计算`、`UV计算与采样`。

## 外部可核验来源

- Microsoft Learn 的 HLSL `step(y, x)` 文档说明：该函数比较两个值，返回值等价于 `(x >= y) ? 1 : 0`。这支持文章中 `step(a, b)：当 b >= a 时返回 1，否则返回 0` 的解释。
- Unreal Engine 4.27 官方 Texturing 文档说明：`WorldAlignedTexture` 用于在世界空间把纹理平铺到物体表面，并提供世界单位缩放；`WorldCoordinate3Way` 会把纹理投影到 XY、YZ、XZ 平面，并允许控制边缘混合。这说明三向投影和世界对齐纹理本身是 UE 材质函数体系中的常见问题域。
- Unity 2019.4 官方手册说明：ShaderLab 的 2D texture 属性在 Cg/HLSL 中映射为 `sampler2D`，并且 Unity 会为纹理属性提供 `_ST` 等 tiling/offset 参数。这支持把文章中的 Unity 代码视为常规 Unity shader 属性与纹理采样写法。

## 核心问题

传统 triplanar mapping 的优势是减少模型 UV 拉伸和接缝依赖：它通常按表面法线权重，在 XY、YZ、XZ 三个方向各生成一组投影 UV，分别采样纹理，再进行混合。代价是纹理采样次数和混合计算增加。

本文方案的目标不是保留传统三面映射的软过渡，而是用“主导轴硬选择”替代“三方向加权混合”：先判断当前片元法线最接近 X/Y/Z 哪个轴，只选择对应平面的 UV，然后只采样一次纹理。

我的判断：这篇文章本质上是在 triplanar mapping 与 cube projection 思路之间取了一个性能优先的折中。它保留了“不依赖手工 UV 展开也能投影纹理”的工程价值，但牺牲了传统三面映射在转角处的软混合质量。

依据：知乎原文明确把采样次数从 3 次降到 1 次；UE 官方 `WorldCoordinate3Way` 描述的是三平面投影并在边缘混合；评论区也有人提到可用 dither 缓解断裂感。

## 算法拆解

### 1. 法线空间选择

文章给出两类处理方式：

- `World Position UV` 模式使用世界空间法线。
- `Object Position UV` 和 `Mesh UV0` 模式先把法线转换到物体空间。

我的判断：这个选择是合理的，因为 UV 生成所依据的位置空间和法线判断空间应保持一致。世界坐标投影用世界法线，物体坐标或模型 UV 缩放补偿用物体法线，能减少坐标系不一致带来的方向错判。

### 2. 取绝对法线并找主导轴

文章对归一化法线取绝对值，然后用 HLSL `step` 生成三通道遮罩：

```hlsl
face.x = step(normal.y, normal.x) * step(normal.z, normal.x);  // X轴主导
face.y = step(normal.z, normal.y) * (1 - face.x);              // Y轴主导
face.z = 1 - face.x - face.y;                                  // Z轴主导
```

这会让 `face.x`、`face.y`、`face.z` 中只有一个分量为 1。直观理解是：

- `face.x = 1`：X 轴主导，选择 YZ 平面投影。
- `face.y = 1`：Y 轴主导，选择 XZ 平面投影。
- `face.z = 1`：Z 轴主导，选择 XY 平面投影。

Microsoft Learn 对 `step(y, x)` 的定义可验证这个判断逻辑：当第二个参数大于或等于第一个参数时返回 1。

### 3. 只构造一组最终 UV

文章把三个候选 UV 用 one-hot mask 合成：

```hlsl
finalUV = i.worldPos.xy * face.z
        + i.worldPos.zy * face.x
        + i.worldPos.xz * face.y;
```

因为 `face` 是 one-hot，这里看似是三路相加，本质只会保留一个平面的坐标。随后只需要一次：

```hlsl
float4 color = tex2D(_MainTex, finalUV);
```

我的判断：这个写法的核心收益来自“先选 UV，再采样”，而不是“采样后混合”。一旦把选择推到采样之前，纹理带宽和采样器压力就明显下降。

### 4. Mesh UV0 的缩放补偿

文章从 `unity_ObjectToWorld` 矩阵的三个基向量长度中提取模型缩放：

```hlsl
float3 scale = float3(
    length(unity_ObjectToWorld._m00_m10_m20),
    length(unity_ObjectToWorld._m01_m11_m21),
    length(unity_ObjectToWorld._m02_m12_m22)
);
scale *= (1 - face);
```

再按当前主导面选择缩放组合，让 Mesh UV0 模式在模型缩放时仍能得到相对合理的纹理密度。

我的判断：这个补偿适合处理非均匀缩放下的视觉密度问题，但它不是完整的 UV 展开修复。若原始 UV 岛本身方向混乱、比例不统一或存在明显接缝，缩放补偿只能缓解局部尺度，不会自动解决 UV 拓扑问题。

### 5. 为什么要在片元阶段计算遮罩

文章明确提醒遮罩要在片元着色器里算。原因是主导轴遮罩是离散 one-hot 值；如果在顶点阶段算好，再经过三角形插值进入片元阶段，遮罩可能变成类似 `(0.3, 0.7, 0)` 的混合权重，从而让最终 UV 变成多平面混合坐标，产生错误。

我的判断：这是这篇文章里最值得单独记住的工程细节。主导面选择属于离散分类，不适合先在低频顶点上分类再插值；应当在片元阶段基于当前片元法线重新分类。

## 与传统三面映射的区别

| 维度 | 传统三面映射 | 本文方案 |
| --- | --- | --- |
| 投影方向 | XY / YZ / XZ 三方向都参与 | 只选主导轴对应平面 |
| 纹理采样 | 通常 3 次 | 1 次 |
| 边界处理 | 法线权重软混合 | 主导轴硬切换 |
| 视觉质量 | 转角过渡更平滑 | 可能出现方向跳变或接缝 |
| 性能取向 | 质量优先 | 带宽和采样次数优先 |

我的判断：这不是“传统 triplanar 的无损优化”，而是“用硬选择换取采样节省”。如果纹理是噪声、岩石、泥土、墙面污渍等方向性不强的材质，硬切边界较不显眼；如果纹理有文字、条纹、明显笔触方向或大块结构，边界跳变会更容易暴露。

## UE 落地方式

文章 UE 部分以节点截图为主。按其 Unity 逻辑迁移到 UE 材质图，可以拆成四段：

1. 取世界法线或对象空间法线，并做 `abs(normalize(N))`。
2. 用 `Step`、`Max`、`If` 或等价节点算出 X/Y/Z one-hot 主导面遮罩。
3. 从 `WorldPosition`、`ObjectPosition` 或 Mesh UV 中组合 XY / YZ / XZ 候选 UV。
4. 用遮罩选择最终 UV，接入单个 `Texture Sample` 的 UV 输入。

我的判断：UE 已有 `WorldAlignedTexture` 与 `WorldCoordinate3Way` 一类材质函数可作为语义参考，但本文方案并不是直接调用这些函数，而是手写/手搭一个“只采样一次”的硬选择版本。实现时应优先做 debug view：显示 `face.x/y/z` 三色遮罩和 `finalUV`，再接真实纹理。

## 适合场景

- 开放世界地形、岩石、崖壁、洞穴、废墟、泥土等大面积自然材质。
- 程序化生成关卡或动态生成网格，UV 不稳定或不值得手工展开的资产。
- 移动端、VR 或大量材质实例中，纹理采样带宽比转角软过渡更敏感的场景。
- 低频、无方向、可重复平铺的纹理。
- 需要在 Unity 和 UE 中复用同一类 shader 思路的技术美术实验。

## 不适合场景

- 有文字、图案、木纹长方向、布料织纹方向等强方向性的纹理。
- 镜头贴近转角、边界或法线临界区域的高质量材质。
- 需要传统 triplanar 平滑过渡的英雄资产。
- 法线不稳定、低模硬边密集或动态变形导致主导轴频繁变化的模型。
- 需要法线贴图、粗糙度、AO 等多张贴图完全一致投影时，仍要评估多贴图采样总成本。

我的判断：这套方案最像一个“性能模式”的 triplanar。它适合做工具箱里的可选分支，而不是直接替代项目中所有 `WorldAlignedTexture` 或传统三面映射材质。

## 可迁移清单

- 给材质函数暴露投影空间选项：World / Object / Mesh UV0。
- 给材质函数暴露 `Tile`、`Offset`、`Axis Flip`、`Rotation`，否则艺术侧很难调纹理方向。
- 给主导面遮罩做 debug 输出：X=红，Y=绿，Z=蓝。
- 对硬切边界敏感的材质，可尝试 dither、noise threshold 或保留传统三向混合分支。
- 在移动端测试时，不只看 ALU，还要看纹理采样、mipmap、anisotropic filtering 与目标 GPU 的实际 profiling。
- 若用于 UE，建议封装为 Material Function，输入 TextureObject、WorldPosition/ObjectPosition、Normal、Tiling、模式枚举，输出 sampled color 或 final UV。
- 若用于 Unity，建议把 fragment 阶段的 face mask 与 UV 计算保持在同一个函数中，避免未来维护者误移到 vertex 阶段。

## 我的判断

这篇资料适合归档到 `虚幻 / 材质 / 三面映射 / WorldAlignedTexture / Shader性能优化`，同时保留 `Unity HLSL` 标签。

它的价值在于把一个常见材质问题压缩成很清楚的工程选择：如果你愿意接受主导轴硬切，就能把三面映射的核心纹理采样从三次降到一次。对大面积自然材质和性能敏感平台，这个思路很实用；对近景高质量材质，它应作为可切换优化，而不是默认质量方案。

## 后续检索关键词

- 中文：`单次采样三面映射`、`三面映射 主导轴`、`UE WorldAlignedTexture 优化`、`材质节点 三向投影`、`Unity triplanar mapping shader`、`主导面遮罩`
- English: `single sample triplanar mapping`, `dominant axis triplanar`, `hard axis triplanar`, `WorldAlignedTexture optimization`, `WorldCoordinate3Way`, `triplanar mapping one sample`

## 来源

- 用户提供材料：知乎专栏《TA实践分享：简单且只采样一次的三面映射(Unity+UE）》，`https://zhuanlan.zhihu.com/p/1989826376426078860`。
- Microsoft Learn：HLSL `step` intrinsic，`https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-step`。
- Unreal Engine 官方文档：Texturing material functions，`https://dev.epicgames.com/documentation/en-us/unreal-engine/texturing?application_version=4.27`。
- Unity 官方手册：Accessing shader properties in Cg/HLSL，`https://docs.unity3d.com/2019.4/Documentation/Manual/SL-PropertiesInPrograms.html`。
