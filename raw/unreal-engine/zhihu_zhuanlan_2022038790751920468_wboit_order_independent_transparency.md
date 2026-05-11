---
source_url: "https://zhuanlan.zhihu.com/p/2022038790751920468"
type: webpage-summary
title: "顺序无关透明渲染WBOIT（如何跟AI一起学习图形学？）"
captured_at: 2026-05-11T19:23:46+08:00
author: "wlxklyh"
topics:
  - 渲染
  - 透明渲染
  - OIT
  - WBOIT
  - GPU Blending
  - 移动端渲染
related_source:
  - "https://jcgt.org/published/0002/02/09/"
  - "https://jcgt.org/published/0002/02/09/paper.pdf"
  - "https://registry.khronos.org/OpenGL-Refpages/gl4/html_test/glBlendFunc.html"
  - "https://docs.nvidia.com/gameworks/content/gameworkslibrary/graphicssamples/opengl_samples/weightedblendedoitsample.htm"
capture_scope: "摘要型资料卡；未全文转存"
---

# WBOIT 顺序无关透明渲染资料卡

## 来源事实

- 来源文章：知乎专栏《顺序无关透明渲染WBOIT（如何跟AI一起学习图形学？）》。
- 作者：`wlxklyh`。
- 所属专栏：`引擎渲染项移动端知识`。
- 页面显示编辑 / 更新时间：`2026-03-30 20:11`。
- 主题链接文本：`透明渲染`、`累积 Pass`、`Composite`。
- 文章说明：作者在前言中说明这是一次“用 AI 生成 Web Demo、截图 / GIF、再写文章”的图形学学习实验，正文解释部分主要由 AI 生成。
- 文章引用论文：McGuire, Bavoil. "Weighted Blended Order-Independent Transparency." JCGT Vol.2 No.2, 2013。
- 官方 / 可核验来源：
  - JCGT 原论文页面与 PDF 确认论文标题、作者、期刊和发布时间。
  - Khronos OpenGL `glBlendFunc` 参考页说明，常规透明度可使用 `GL_SRC_ALPHA`、`GL_ONE_MINUS_SRC_ALPHA` 并按远到近排序。
  - NVIDIA GameWorks WBOIT 示例说明，Weighted Blended OIT 可在不做深度排序的单个 geometry pass 中近似渲染 OIT，并使用两个 draw buffers 存储加权颜色/alpha 累积与 revealage。

## 核心问题

透明物体的标准 Alpha Blending 是顺序敏感的。两个半透明层只要绘制顺序不同，最终颜色就不同；当物体互相穿插、相机旋转或排序依据不稳定时，画面会出现颜色跳变、闪烁和层次错误。

WBOIT（Weighted Blended Order-Independent Transparency）要解决的是：不对透明片元做严格排序，也能得到稳定、近似合理的透明合成结果。

我的判断：WBOIT 的价值不是“透明度更物理正确”，而是“在排序不可解或成本过高时，用可并行、顺序无关的近似换稳定性”。这对粒子、烟雾、云雾、多层低 alpha 特效和移动端透明渲染尤其实用。

## 一句话结论

我的判断：WBOIT 把传统透明混合中的“有序叠加”替换为“加权累积 + revealage 透射率 + 最终合成”。它用加法和乘法这类交换律运算绕开排序问题，因此画面稳定；但因为它本质是近似平均，高不透明度重叠时会丢失前后遮挡层次。

依据：文章用红绿蓝透明面片展示了排序敏感问题；NVIDIA 示例明确说明 Weighted Blended OIT 是深度剥离结果的快速近似；JCGT 论文即该算法的一手来源。

## 原理拆解

### 1. 标准 Alpha Blending 为什么会错

文章给出的标准公式是：

```text
最终颜色 = 新物体颜色 * alpha + 画布已有颜色 * (1 - alpha)
```

这个公式天然依赖“画布已有颜色”。如果绿色先画、红色后画，结果偏红；红色先画、绿色后画，结果偏绿。

Khronos OpenGL 文档也说明，透明度通常用 `GL_SRC_ALPHA`、`GL_ONE_MINUS_SRC_ALPHA`，并需要 primitives 从远到近排序。

我的判断：标准 Alpha Blending 的问题不是公式错，而是它要求一个稳定、正确的排序。互相穿插的透明几何体通常不存在全局正确顺序，物体中心排序只能解决很有限的情况。

### 2. WBOIT 的加权平均

文章把 WBOIT 的核心写成：

```text
平均颜色 = sum(color_i * alpha_i * weight_i) / sum(alpha_i * weight_i)
```

因为分子和分母都是求和，加法不依赖顺序，所以红、绿、蓝谁先画都不会改变累积结果。

文章还给出一类权重函数思路：更靠近相机、更不透明的片元权重大。也就是说，WBOIT 并不是简单平均，而是用深度和 alpha 做启发式近似，让近处和更实的透明层更影响结果。

我的判断：权重函数是 WBOIT 的“审美旋钮”。它决定近处透明层有多强、远处层被压多少、不同深度范围是否稳定。项目里不能把论文常数当万能参数，应按相机深度范围和特效类型调。

### 3. 累积 Pass

文章解释 GPU 可通过加法混合来“攒”透明层：

```text
透明片元输出: vec4(color * alpha * weight, alpha * weight)
混合模式: ONE + ONE
```

画完后：

- RGB 存 `sum(color_i * alpha_i * weight_i)`；
- A 存 `sum(alpha_i * weight_i)`。

NVIDIA WBOIT 示例也说明，在第一个 draw buffer 中，RGB 存 `Sum[FragColor.rgb * FragColor.a * Weight(FragDepth)]`，Alpha 存 `Sum[FragColor.a * Weight(FragDepth)]`。

我的判断：这个 Pass 的关键工程意义是“透明物体不必排序也能并行画”。这对 GPU 友好，因为每个片元只要把自己的贡献加进去。

### 4. Revealage

文章把 revealage 解释为“背景透过了多少”：

```text
revealage = (1 - alpha_1) * (1 - alpha_2) * ... * (1 - alpha_n)
```

乘法同样不依赖顺序。文章提到可用类似：

```text
blendFunc(ZERO, ONE_MINUS_SRC_ALPHA)
```

从 1 开始，把每层透明度剩余透射率乘进去。

NVIDIA 示例也说明第二个 draw buffer 的 R 通道存 `Product[1.0 - FragColor.a]`。

我的判断：revealage 是 WBOIT 不只是“算平均色”的关键。没有 revealage，算法不知道背景该露出多少；有了 revealage，最终合成才有“透明层挡住多少背景”的概念。

### 5. Composite 合成

文章给出的合成公式：

```text
最终颜色 = 平均色 * (1 - revealage) + 背景 * revealage
平均色 = 累积纹理 RGB / 累积纹理 A
```

我的判断：可以把整个 WBOIT 管线理解成三张图：

- Accum：透明层贡献的加权颜色和权重；
- Revealage：背景还能透过多少；
- Opaque/Background：不透明场景颜色。

最后 Composite 把透明平均色按遮挡量覆盖到背景上。

## 适用场景

### 适合

- 烟雾、云层、雾效、魔法特效；
- 多层低 alpha 粒子；
- 毛玻璃感、能量罩、半透明 UI/世界特效；
- 透明几何体相互穿插，传统排序容易闪烁的场景；
- 移动端或实时项目中，希望用较少 pass 获得稳定 OIT 近似的场景。

### 不适合

- `alpha` 接近 1 的高不透明度透明面；
- 需要明确前后层次和锐利遮挡边界的玻璃 / 硬透明物体；
- 需要物理精确的折射、吸收、有色透射；
- 透明物体数量少且可稳定排序的简单场景；
- 对颜色严格准确有要求的离线渲染或产品可视化。

## 已知局限

文章总结的局限包括：

- 高不透明度重叠会变混浊；
- 权重函数需要针对深度范围和场景调参；
- 算法是近似，不是物理精确光学；
- 不处理折射；
- 不处理有色透射。

我的判断：这些局限都来自同一个根因：WBOIT 用加权平均替代了真实的“前层遮挡后层”。因此只要视觉目标接近“很多薄层混合”，它表现就好；视觉目标接近“前景硬遮挡后景”，它就容易显脏。

## 可迁移清单

- 先判断透明内容类型：粒子/烟雾优先考虑 WBOIT，玻璃/水面/厚透明物体谨慎。
- 保留不透明 Pass，WBOIT 只处理透明层，最后 Composite 到不透明场景上。
- Accum buffer 至少需要能承载 HDR / 浮点累积，避免多层透明时精度不足。
- Revealage buffer 可单通道存储，但要确认混合模式能实现连续乘积。
- 权重函数应暴露调参项：深度衰减、alpha 权重、最大/最小 clamp。
- 对高 alpha 透明材质设置阈值或改用排序透明、Depth Peeling、Stochastic Transparency 等其他方案。
- 在移动端重点测试 MRT 支持、带宽、浮点 RT 格式、tile GPU 的带宽压力。
- 做 debug view：Accum RGB、Accum A、Revealage、Composite 四宫格能显著降低排查成本。
- 把 WBOIT 归类为“稳定近似透明方案”，不要作为所有透明材质的默认答案。

## 我的判断

这篇资料最适合归入 `虚幻 / 渲染 / 透明渲染 / OIT / WBOIT / 移动端渲染`。

它的价值有两层：第一层是把 WBOIT 的数学和 GPU 实现拆得很直观，适合给技术美术或初级图形程序建立透明渲染直觉；第二层是展示“用 AI 生成交互 Demo 来学习图形学”的资料生产方式，这对于复杂渲染概念很有启发。

对项目来说，我会把 WBOIT 当作粒子、烟雾、多层低 alpha 透明的候选方案，而不是通用透明渲染替代品。它解决的是排序稳定性和 pass 成本问题，不能替代真实折射、吸收或严格前后遮挡。

## 后续检索关键词

- 中文：顺序无关透明、WBOIT、Weighted Blended OIT、透明渲染排序、Revealage、Alpha Blending、GPU 混合、移动端透明渲染
- English: Weighted Blended Order-Independent Transparency, WBOIT, OIT, revealage, alpha blending, additive blending, transparent sorting, depth peeling, GPU transparency

## 来源

- 知乎原文：https://zhuanlan.zhihu.com/p/2022038790751920468
- JCGT 论文页：https://jcgt.org/published/0002/02/09/
- JCGT 论文 PDF：https://jcgt.org/published/0002/02/09/paper.pdf
- Khronos OpenGL `glBlendFunc` 参考：https://registry.khronos.org/OpenGL-Refpages/gl4/html_test/glBlendFunc.html
- NVIDIA GameWorks WBOIT 示例：https://docs.nvidia.com/gameworks/content/gameworkslibrary/graphicssamples/opengl_samples/weightedblendedoitsample.htm
