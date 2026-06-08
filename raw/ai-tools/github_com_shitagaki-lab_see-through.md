---
source_url: "https://github.com/shitagaki-lab/see-through"
type: github-repository-card
title: "See-through: Single-image Layer Decomposition for Anime Characters"
captured_at: 2026-06-08T23:36:54+08:00
contributor: "Codex"
license: "Apache-2.0"
language: "Python"
topics:
  - See-through
  - Anime Character Layer Decomposition
  - 2.5D Model
  - LayerDiff 3D
  - Marigold Depth
  - PSD Export
  - Live2D Preparation
  - AI Art Tool
  - Computer Vision
related_source:
  - "https://github.com/shitagaki-lab/see-through"
  - "https://arxiv.org/abs/2602.03749"
  - "https://huggingface.co/spaces/24yearsold/see-through-demo"
  - "https://modelscope.cn/studios/ljsabc/See-Through"
capture_scope: "摘要型资料卡；源材料来自仓库 README 与 GitHub API 元数据"
---

# shitagaki-lab/see-through 资料卡：动漫角色单图图层分解工具

## 来源事实

- 仓库：`shitagaki-lab/see-through`，URL：<https://github.com/shitagaki-lab/see-through>。
- GitHub 描述字段：`"Single-image Layer Decomposition for Anime Characters" (SIGGRAPH 2026, Conditionally Accepted)`。
- 主语言：Python；许可证：Apache-2.0；默认分支：`main`。
- README 说明该项目是开源研究项目，作者没有设置任何付费服务；遇到收费网站应自行甄别。
- README 关联论文：`See-through: Single-image Layer Decomposition for Anime Characters`，arXiv：<https://arxiv.org/abs/2602.03749>。
- README 写明论文为 `ACM SIGGRAPH 2026 Conference Proceedings` conditional acceptance。
- README 提供两个在线体验入口：Hugging Face Space `24yearsold/see-through-demo` 与 ModelScope `ljsabc/See-Through`。
- 仓库根目录包含 `inference/`、`training/`、`ui/`、`common/`、`annotators/`、`requirements*.txt`、`README_datapipeline.md` 等目录或文件。

## 核心定位

See-through 面向静态动漫角色插画，目标是把单张 2D 图像自动拆成可操作的 2.5D 图层模型。README 的 TL;DR 描述为：它会把单图分解为已补全遮挡区域、语义上区分清楚、并带有推断绘制顺序的图层，最多包含 23 个语义层，例如头发、脸、眼睛、衣服、配饰等。

我的判断：这类工具最有价值的不是“把图切成透明 PNG”，而是把角色插画从最终成图转换回接近制作资产的中间层。它适合进入 AI 美术工具、角色动画预处理、Live2D 前处理、2.5D 展示和游戏立绘资产拆分工作流。

## 主要能力

- **图层分解**：从单张动漫角色图生成语义分层结果，并导出 PSD。
- **遮挡补全**：对被前景遮挡的区域做 inpainting，使分离出的图层可独立移动或编辑。
- **深度辅助**：使用 fine-tuned Marigold 进行 pseudo-depth estimation，再支持基于深度的进一步分层。
- **语义分割**：README 提到 SAM Body Parsing、SAM2、mmdet/detectron2 等可选 annotator 层级，用于身体部位或实例分割。
- **低显存推理**：README 给出 group offload、NF4 quantized pipeline、block swap pipeline 等低显存运行方案；默认 bf16 1280 分辨率约需 12-16 GB VRAM，8 GB GPU 可使用量化或 block swap 路径。
- **训练与数据管线**：`training/` 下提供 LayerDiff、Marigold depth、VAE、body part segmentation 的训练脚本、配置和数据管线说明。
- **UI 与社区集成**：README 记录了 ComfyUI-See-through、PachiPakuGen、StretchyStudio 等第三方集成或后续动画工具。

## 关键模型与脚本

| 项目 | 作用 |
| --- | --- |
| `LayerDiff 3D` | Diffusion-based transparent layer generation，负责透明图层生成 |
| `Marigold Depth` | 面向动漫图像微调的 pseudo-depth estimation |
| `SAM Body Parsing` | 语义身体部位分割 |
| `inference/scripts/inference_psd.py` | 主推理管线：layer decomposition 到 PSD 输出 |
| `inference/scripts/inference_psd_quantized.py` | NF4 量化推理路径，面向低显存 |
| `inference/scripts/inference_psd_blockswap.py` | block swap 低显存推理路径 |
| `inference/scripts/heuristic_partseg.py` | PSD 后处理，例如基于深度或左右方向继续拆分 |

## 使用路径

README 推荐创建 `conda` 环境，Python 版本为 3.12，并按 CUDA 12.8 安装 PyTorch，再安装 `requirements.txt`。主流程命令示例是运行 `inference/scripts/inference_psd.py`，指定单张图片或图片目录，并使用 `--save_to_psd` 输出分层 PSD；默认输出目录是 `workspace/layerdiff_output/`。

我的判断：这个项目的本地落地成本比普通 ComfyUI 节点高，因为它依赖 PyTorch、多个可选分割器、模型下载和显存策略；但它把“单张立绘 → PSD 图层资产”的关键路径做成了可重复命令，对游戏角色素材生产线有长期复用价值。

## 与现有知识库的连接

- 与 `AI Game DevTools` 的图像、动画、角色资产工具分支相关：See-through 属于 AI image / animation asset preparation 工具，而不是纯生成模型清单。
- 与 `Dan Milligan 分镜画风参考` 不同：后者是视觉风格参考，See-through 是角色资产处理工具。
- 与游戏叙事或引擎源码资料不同：See-through 更接近美术生产和角色动画前处理，可服务于 UI 立绘、Live2D、2.5D cutscene、角色表演素材拆分。
- 与 Live2D 工作流相关但不等于 Image-to-Live2D：README 明确说明完整 Live2D 仍需要更细艺术拆分、rigging、物理参数和动作曲线；See-through 主要解决自动分层和遮挡补全。

## 我的判断

See-through 适合收进本项目的 `AI 工具 / AI 美术 / 角色资产预处理` 分支。它的独特性在于：面向动漫角色图像，不只是 segmentation，而是把语义层、遮挡补全、深度关系和 PSD 导出连成一个可运行管线。后续如果项目需要“把已有角色概念图转成可动画资产”，它可以作为首选候选；如果目标是成品 Live2D，则应把它放在“自动预处理”位置，而不是替代人工 rigging。

## 后续可搜索关键词

- 中文：动漫角色图层分解、单图 PSD 拆层、AI 立绘拆分、2.5D 角色资产、Live2D 前处理、遮挡区域补全、动漫语义分割、角色深度分层
- English：See-through, single-image layer decomposition, anime character layer decomposition, 2.5D model, PSD export, LayerDiff 3D, Marigold depth, Live2D preprocessing, semantic layers, transparent layer generation

## Notable Signals For Graph Extraction

- `See-through -> Anime Character Layer Decomposition`
- `See-through -> 2.5D Model`
- `See-through -> PSD Export`
- `See-through -> LayerDiff 3D`
- `See-through -> Marigold Depth`
- `See-through -> Live2D Preprocessing`
- `See-through -> AI Art Tool`
- `See-through -> Game Character Asset Pipeline`
