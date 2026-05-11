---
source_url: "https://zhuanlan.zhihu.com/p/6353835934"
type: webpage-summary
title: "3D虚拟服装OOTD——纹理程序化上色技术"
captured_at: 2026-05-11T19:50:02+08:00
author: "Y.one"
topics:
  - 3D 虚拟服装
  - OOTD
  - 程序化上色
  - ReColor
  - Unity Albedo
  - BaseColor
  - CIE Lab
  - kmeans++
  - Davies-Bouldin 指数
  - GLSL
related_source:
  - "https://docs.unity.cn/Documentation/Manual/StandardShaderMaterialParameterAlbedoColor.html"
  - "https://docs.opencv.org/3.4/d5/d38/group__core__cluster.html"
  - "https://cie.co.at/publications/colorimetry-part-4-cie-1976-lab-colour-space-0"
  - "https://scikit-learn.org/1.5/modules/generated/sklearn.metrics.davies_bouldin_score.html"
capture_scope: "摘要型资料卡；未全文转存"
---

# 3D 虚拟服装程序化上色资料卡

## 来源事实

- 来源文章：知乎专栏《3D虚拟服装OOTD——纹理程序化上色技术》。
- 作者：`Y.one`，页面简介显示为 `AI技术美术`。
- 所属专栏：`YONE小栈`。
- 页面专栏模块显示：`所属专栏 · 2024-11-12 12:29 更新`。
- 页面底部显示：`编辑于 2025-06-24 17:29・广东`。
- 页面显示赞同数约 `199`，评论约 `11`。
- 文章主题标签：`技术美术`、`游戏开发`、`3D 渲染`。
- 文章核心问题：很多游戏用户有角色服装 OOTD / 二创需求，文章从技术层面讨论如何通过虚拟服装自定义染色产生新服装。
- 外部 Unity 佐证：Unity 2022.3 手册说明，`Albedo` 属性控制材质的 base color 和透明度，也可以给 Albedo 指定纹理；这可佐证文章把 Unity 中的 Albedo 贴图理解为服装基础颜色输入。
- 外部 OpenCV 佐证：OpenCV `kmeans` 文档说明其会寻找 K 个聚类中心，并把输入样本归入对应 cluster；`KMEANS_PP_CENTERS` 使用 kmeans++ 初始化。
- 外部 CIE 佐证：CIE 的 CIELAB 标准说明 CIE 1976 L*a*b* 是为更接近视觉均匀的颜色空间而提出，欧氏距离可用于描述近似色差大小。
- 外部 scikit-learn 佐证：`davies_bouldin_score` 文档说明，DB 分数衡量每个簇与最相似簇的相似度均值，分数越低表示聚类越好。

## 核心问题

如果直接用 AIGC 对服装 BaseColor 贴图做“图生图染色”，用户很难实时预览，也很难精确控制局部色值和渐变。文章提出的替代方案是：先离线从服装 BaseColor / Albedo 贴图里聚类出少量代表色块，作为用户可操作的调色盘；运行时根据用户替换色，在 shader 中对原始贴图做 Lab 空间偏移和权重混合，从而保留原图明暗细节并实现实时换色。

## 一句话结论

我的判断：这篇文章的价值在于把“服装染色”从不可控的 prompt 出图，转成一个可实时、可解释、可工程落地的纹理重映射问题：离线做色块聚类，在线做 Lab 偏移 ReColor，再用距离权重混合避免色块断层。

依据：文章明确比较了纯 AIGC 染色的实时性和可控性问题，并给出 kmeans++ 聚类、DB 指数选 K、Lab 空间换色和多标签色混合的完整流程。

## 方案拆解

### 1. 放弃纯 AI 染色出图

文章最初尝试通过 AIGC 纹理图生图：输入服装 BaseColor 纹理和染色 prompt，输出染色后的贴图。但实践后认为这种方式有明显局限：

- 实时性低，无法做交互式实时预览。
- 可控性不高，prompt 操作有门槛。
- 难以精确调整渐变色值。
- 容易得到整体颜色替换，而不是稳定的局部服装配色编辑。

我的判断：纯 AIGC 更适合生成新纹理风格和创意草稿，不适合作为游戏内用户实时换装染色的主链路。用户需要的是低延迟、可撤销、可局部控制、可预期的编辑工具。

### 2. 基于调色盘的实时 ReColor

文章改用类似修图软件 ReColor 的思路：输入服装 BaseColor / Albedo 贴图，先提取若干标签色作为调色盘，让用户对这些标签色快速换色。

这个流程既可以直接用于原始服装，也可以接入 AIGC 服装纹理流程作为后处理阶段。文章认为关键是如何提取合理色块，以平衡高可控和低门槛。

我的判断：这是一个很适合产品化的折中方案。AIGC 可负责“生成一件衣服”，ReColor 负责“让用户稳定调这件衣服的配色”。

### 3. 颜色聚类

文章把服装纹理划分成 K 个标签色，作为用户调色盘色块。该步骤可事先离线生成，运行时只使用聚类结果。

实现上，文章采用 kmeans++：输入 BaseColor 贴图像素色值，输出 K 个颜色标签。OpenCV 文档也说明 `cv::kmeans` 会找 K 个 cluster centers 并返回每个样本的 cluster index，而 `KMEANS_PP_CENTERS` 使用 kmeans++ 初始化。

我的判断：离线聚类是正确位置。把聚类放到运行时会增加成本，也会让用户每次打开服装时等待；离线生成标签色、运行时只调参数，才符合游戏内换色体验。

### 4. 色彩空间选择

文章比较 RGB、HSV/HSL、CIE Lab：

- HSV/HSL 对用户调色直观，但 Hue 是 360 度循环值，不适合直接用欧氏距离做 kmeans。
- RGB 是线性三通道加色空间，但欧氏距离不太符合人眼色差感知。
- Lab 把亮度 L 与色度 a/b 分开，且更接近人眼感知，可用欧氏距离近似色差。

文章最终选择 Lab，并对 L 与 a/b 施加不同权重，例如降低亮度权重、提高色度权重，以便聚类更偏“色调差异”而不是“明暗差异”。

外部来源：CIE 标准说明 CIELAB / CIELUV 是 CIE 在 1976 年推荐的更接近视觉均匀的颜色空间，欧氏距离可以用于描述近似颜色差异。

我的判断：对服装换色来说，强调色度比强调亮度更合理。因为用户调的是“这块布是什么颜色”，而不是把同一块布上的褶皱亮面和暗面拆成不同调色块。

### 5. 自动确定色块数 K

文章指出 K 太大会增加用户操作成本，K 太小会导致无法精准换色。因此需要用聚类评价指标自动选择合理 K。

文章比较了轮廓系数法、DB 指数法和手肘法，并最终选择 DB 指数法：对 K 从 1 到 5 分别运行 kmeans，计算 DB 指数，选择最低分对应的 K。

外部来源：scikit-learn 文档说明 DB score 是每个簇与其最相似簇的相似度均值，越低越好。

我的判断：把 K 限制在 1 到 5 是面向用户体验的好约束。服装编辑不是图像分割比赛，调色盘色块太多会让用户失去“简单换色”的感觉。

## ReColor Shader 思路

### 1. 单标签色换色

文章的单标签色 ReColor 方式是：

1. 把原始像素、标签色、替换色都转到 Lab 空间。
2. 计算 `lab_offset = change_lab - label_lab`。
3. 把偏移叠加到原始像素的 Lab 值上。
4. clamp 后转回 RGB。

我的判断：这个方案比直接把像素改成替换色更好，因为它保留了原始贴图的局部明暗、褶皱和细节。它本质上是“移动颜色分布”，不是“覆盖颜色”。

### 2. 多标签色混合

文章指出，若直接按色块硬切换，渐变纹理会产生颜色断层。为解决这个问题，文章使用距离权重混合：

1. 当前像素与每个标签色计算 Lab 距离。
2. 距离转权重，例如 `weight = pow(1 - dist, exponent)`，再归一化。
3. 对每个标签色分别得到一个 Lab 偏移染色结果。
4. 最终颜色为所有染色结果按权重线性叠加。

我的判断：这一步是工程落地关键。服装纹理通常有渐变、阴影和材质噪声，硬分区会显得像套索工具；距离权重混合能让换色结果保留连续性。

## 配色探索

文章最后讨论了降低用户操作门槛的配色方式：

- 基于原有标签色做 HSV 色调偏移，生成新的替换色，类似 Adobe Illustrator 色轮。
- 根据配色准则输出替换色组合。
- 根据标签色数量确定更合适的配色方案。

我的判断：这说明系统可以分两层：底层是 Lab ReColor 算法，上层是配色推荐 UI。普通用户不一定想逐个色块调 RGB，系统应提供“整体换风格”“互补色”“邻近色”“同色系”等一键方案。

## 可迁移清单

- 离线读取服装 BaseColor / Albedo 贴图。
- 把像素转 Lab，并给 L 与 a/b 设置可调权重。
- 用 kmeans++ 聚类得到候选标签色。
- 用 DB 指数或其他聚类指标在小范围 K 内选择色块数。
- 按像素数量排序标签色，过滤面积太小的噪声色块。
- 运行时暴露调色盘 UI，让用户为每个标签色选择替换色。
- 在 shader 中把原像素、标签色、替换色转 Lab，做偏移 ReColor。
- 对多个标签色用距离权重混合，避免渐变断层。
- 保留 alpha，并避免把金属、法线、粗糙度等非颜色贴图错误参与 ReColor。
- 给用户提供 HSV 色相偏移、配色准则和随机配色等高级入口。

## 风险与边界

- Lab 偏移可能超出可显示色域，需要 clamp 或更细致的 gamut mapping。
- DB 指数对异常点敏感，文章也提到需要平滑降噪。
- 贴图里的光照烘焙、污渍、图案、刺绣可能被聚类误判为色块。
- 如果 BaseColor 不是干净的材质底色，而包含大量阴影或后期调色，换色会更难稳定。
- 多标签混合的 exponent 参数会影响边界软硬，需要给不同材质调默认值。
- 仅改 BaseColor 无法处理“材质也变了”的需求，例如丝绸变皮革、棉布变金属。

## 适合与不适合

适合：

- 游戏内服装染色、换装 OOTD、玩家二创。
- 已有服装贴图，需要低成本派生配色版本。
- 需要实时预览、可撤销、低门槛操作的编辑器。
- AIGC 服装生成后的后处理配色。

不适合：

- 需要完全重新生成图案、刺绣、纹理结构的场景。
- 想用一句 prompt 完成复杂风格迁移的场景。
- BaseColor 贴图质量很差、光照烘焙严重、材质信息混杂的资产。
- 需要同时改变颜色、材质、法线、粗糙度和服装版型的完整重设计。

## 我的判断

这篇资料应归入：`Unity / 技术美术 / 虚拟服装 / 程序化染色 / ReColor / 贴图工具链`。

它的核心价值是给出一条能落地到游戏内编辑器的染色链路：AIGC 负责创意生成，ReColor 负责实时可控编辑。对于换装游戏、虚拟偶像、UGC 角色装扮和 3D 电商试衣，这种“离线聚类 + 在线 Lab 偏移”的方案比纯 AI 出图更适合产品化。

## 后续检索关键词

- 中文：`虚拟服装 程序化染色`、`服装 ReColor Lab`、`BaseColor 聚类 调色盘`、`kmeans++ 贴图聚类`、`Davies-Bouldin 指数 色块数`、`游戏内服装染色`
- English: `virtual clothing recolor`、`texture recoloring CIELAB`、`palette based recoloring`、`kmeans texture color clustering`、`Davies-Bouldin index color clustering`、`game clothing dye system`

## 来源

- 用户提供材料：知乎专栏《3D虚拟服装OOTD——纹理程序化上色技术》，`https://zhuanlan.zhihu.com/p/6353835934`。
- Unity 官方手册：`Albedo`，`https://docs.unity.cn/Documentation/Manual/StandardShaderMaterialParameterAlbedoColor.html`。
- OpenCV 官方文档：`Clustering / kmeans`，`https://docs.opencv.org/3.4/d5/d38/group__core__cluster.html`。
- CIE 标准页面：`Colorimetry - Part 4: CIE 1976 L*a*b* Colour space`，`https://cie.co.at/publications/colorimetry-part-4-cie-1976-lab-colour-space-0`。
- scikit-learn 文档：`davies_bouldin_score`，`https://scikit-learn.org/1.5/modules/generated/sklearn.metrics.davies_bouldin_score.html`。
