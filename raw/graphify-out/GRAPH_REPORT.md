# Graph Report - raw

## 来源

- 文档 1：`raw/unreal-engine/slate/github_com_YawLighthouse_UMG-Slate-Compendium.md`
- 文档 2：`raw/tools/github_com_DayuanJiang_next-ai-draw-io_blob_main_docs_cn_README_CN_md.md`
- 文档 3：`raw/tools/github_com_winfunc_opcode.md`
- 文档 4：`raw/tools/github_com_zhukunpenglinyutong_jetbrains-cc-gui.md`
- 文档 5：`raw/unreal-engine/transcripts/yt_5a6fd1b18da4.txt`（YouTube 转写）
- 文档 6：`raw/ai-tools/github_com_Donchitos_Claude-Code-Game-Studios.md`
- 文档 7：`raw/ai-tools/github_com_Yuan-ManX_ai-game-devtools.md`
- 文档 8：`raw/ai-tools/github_com_pamirtuna_gamestudio-subagents.md`
- 文档 9：`raw/unity-engine/github_com_SaiTingHu_HTFramework.md`
- 文档 10：`raw/unity-engine/bilibili_BV1VKRPBTE6H_Unity_motion_interpolation.md`
- 文档 11：`raw/unity-engine/transcripts/yt_30c3774e422c.txt`（Bilibili 转写）
- 文档 12：`raw/game-design/bilibili_BV1f8dMBFEy2_game_narrative_text_reality_worldview.md`
- 文档 13：`raw/game-design/transcripts/yt_93de2d0474b6.txt`（Bilibili 转写）
- 文档 14：`raw/ai-tools/github_com_shitagaki-lab_see-through.md`
- 文档 15：`raw/ai-tools/github_com_shenminglinyi_PlotPilot.md`
- 文档 16：`raw/ai-tools/github_com_sparklecatta-lang_2D-Character-Starter.md`
- 文档 17：`raw/ai-tools/github_com_sparklecatta-lang_sprite-video-lab.md`
- 关系标注依据详见 `raw/graphify-out/MANUAL_ANALYSIS.md`。

## Corpus Check

- 20 个文件（17 docs + 3 audio artifacts）
- Bilibili 新增资料均已补充 `faster-whisper` 中文转写；转写存在少量近音词和繁简混用。

## Summary

- 170 nodes / 260 edges / 16 communities detected
- Extraction: 82% EXTRACTED / 17% INFERRED / 1% AMBIGUOUS
- Token cost: 0 input / 0 output（本轮为本地转写与手工图谱补充）

## God Nodes

1. `Sprite Video Lab` - 17 edges
2. `See-through` - 15 edges
3. `2D Character Starter` - 15 edges
4. `UMG-Slate Compendium` - 14 edges
5. `Next AI Draw.io` - 13 edges
6. `PlotPilot（墨枢）` - 12 edges
7. `Game Studio Sub-Agents` - 11 edges
8. `AI Game DevTools (AI-GDT)` - 11 edges
9. `opcode` - 10 edges
10. `Claude Code Game Studios` - 10 edges

## Surprising Connections

- `Unity Motion Interpolation` --conceptually_related_to--> `Unity Rapid Development Framework` [INFERRED]
  - 基于同属 Unity 角色控制 / 客户端开发实践层的判断；当前语料没有直接调用关系。
- `DevelopAUnityActionGameIn5Min` --conceptually_related_to--> `HTFramework` [INFERRED]
  - 一个是 Unity 动作游戏教程项目，一个是 Unity 客户端快速开发框架；这是生态层关联，不是源码依赖。
- `Display Layer Interpolation` --conceptually_related_to--> `Horizontal Camera Control` [EXTRACTED]
  - 转写明确说明插值显示物体的位置，并把相机跟随目标改为显示物体，用来缓解角色抖动。
- `Display Layer Interpolation` --conceptually_related_to--> `Character 3C` [INFERRED]
  - 视频标签含 `角色3C`，转写内容围绕角色运动与相机跟随展开，因此这是控制器实践层的推断关联。
- `Claude Code Game Studios` --semantically_similar_to--> `opcode` [INFERRED]
  - inferred connection - not explicitly stated in source; connects across different repos/directories; bridges separate communities; semantically similar concepts with no structural link
- `UMG-Slate Compendium` --conceptually_related_to--> `Next AI Draw.io` [AMBIGUOUS]
  - ambiguous connection - not explicitly stated in source; connects across different repos/directories
- `HTFramework` --conceptually_related_to--> `Unity AI Tools` [INFERRED]
  - 同属 Unity 生态资料，但当前语料没有显式工作流示例。
- `Game Narrative Text` --conceptually_related_to--> `Game Studio Sub-Agents` [INFERRED]
  - 我的判断：新增视频偏“游戏内容创作方法”，而 Game Studio Sub-Agents 偏“生产组织工作流”；两者可在游戏开发流程中衔接，但当前语料没有显式协作关系。
- `Dialogue as Character Reveal` --conceptually_related_to--> `Inner Speech Writing Method` [INFERRED]
  - 我的判断：视频分别讨论台词如何暴露人物内心，以及写作前用内部语言跑通逻辑；二者同属叙事文本创作方法，但不是源文直接定义的上下位关系。
- `See-through` --conceptually_related_to--> `AI Game DevTools (AI-GDT)` [INFERRED]
  - 我的判断：See-through 属于 AI 图像 / 动画资产预处理工具，可接入 AI Game DevTools 的 Image、Animation、Avatar、3D Model 等工具景观分支。
- `Game Character Asset Pipeline` --conceptually_related_to--> `Display Layer Interpolation` [INFERRED]
  - 我的判断：See-through 解决角色图层资产拆分，Display Layer Interpolation 解决 Unity 中显示层运动平滑；二者都服务于角色表现资产进入可动展示或游戏运行时的链路。
- `PlotPilot（墨枢）` --conceptually_related_to--> `Game Narrative Text` [INFERRED]
  - 我的判断：PlotPilot 是长篇叙事生产系统，Game Narrative Text 是文本表达方法论；二者可形成“叙事方法 -> 自动化生产内核”的桥接。
- `Narrative State Machine` --structures--> `Game Narrative Text` [INFERRED]
  - 我的判断：PlotPilot 用 Story Bible、故事线 DAG、伏笔注册表和事件流约束长篇文本，这相当于把游戏叙事文本中的人物、因果、伏笔与世界观连续性工程化。
- `2D Character Starter` --conceptually_related_to--> `Game Character Asset Pipeline` [INFERRED]
  - 我的判断：2D Character Starter 生成 / 修正绿幕可游玩角色起始素材，适合放在角色资产管线的早期探索环节；当前现有资料没有直接把该 skill 纳入 See-through 管线，因此标为推断。
- `2D Character Starter` --conceptually_related_to--> `See-through` [INFERRED]
  - 我的判断：两者都服务 2D 角色资产链路，但前者偏生成、简化、姿势与动作方向探索，后者偏单图拆层、遮挡补全和 PSD / Live2D 前处理。
- `Numeric Subagent Sampling` --conceptually_related_to--> `Game Studio Sub-Agents` [INFERRED]
  - 我的判断：2D Character Starter 的数字后缀是同模式多 subagent 独立采样，Game Studio Sub-Agents 是游戏生产组织层的多 agent 体系；二者同属多 agent 生产工作流，但不是同一项目。
- `Sprite Video Lab` --feeds--> `Game Character Asset Pipeline` [INFERRED]
  - 我的判断：Sprite Video Lab 把视频、GIF、单图或序列帧整理为透明 Sprite 资源，处于“素材进入游戏前”的视频 / 帧处理和透明化阶段。
- `Sprite Video Lab` --conceptually_related_to--> `2D Character Starter` [INFERRED]
  - 我的判断：2D Character Starter 生成或修正绿幕角色起始素材，Sprite Video Lab 可继续处理动态图、抽帧、alpha、透明 WebM 和 frames 导出；两者是相邻资产工作流，不是直接依赖。
- `Sprite Video Lab` --conceptually_related_to--> `Display Layer Interpolation` [INFERRED]
  - 我的判断：Sprite Video Lab 产出的帧序列或透明 WebM 可服务运行时角色 / 特效表现，而 Display Layer Interpolation 代表 Unity 运行时显示层平滑问题；这是资产准备到运行时表现的跨阶段联系。

## Communities

### Community 0 - Unreal UMG / Slate Topics

- Nodes (12): `UMG-Slate Compendium`, `Unreal Engine UI Framework`, `UMG`, `Slate`, `Performance & Design Considerations`, `Widget Components`, `Input Framework of Unreal Engine`, `Unreal Focusing System` (+4 more)

### Community 1 - AI Diagram Generation

- Nodes (11): `Next AI Draw.io`, `Natural Language Diagramming`, `draw.io Integration`, `Image and PDF Uploads`, `Cloud Architecture Diagrams`, `Multi-provider Support`, `Next.js`, `Vercel AI SDK` (+3 more)

### Community 2 - Unreal Character Movement

- Nodes (10): `Blueprint`, `C++`, `Unreal Engine | Character Movement Component: In-Depth`, `Character Movement Component (CMC)`, `Custom Character Movement Component`, `Multiplayer Movement`, `Client-side Prediction`, `Server Authority` (+2 more)

### Community 3 - Claude Code Desktop Tooling

- Nodes (6): `opcode`, `Project and Session Management`, `Custom Agents`, `Background Execution`, `Usage Analytics Dashboard`, `Tauri 2`

### Community 4 - Tool Integration and MCP

- Nodes (4): `MCP Server`, `Claude Code CLI`, `MCP Server Management`, `CLAUDE.md Management`

### Community 5 - JetBrains Plugin Workflow

- Nodes (4): `JetBrains CC GUI`, `IntelliJ IDEA Plugin`, `Permission and Security Controls`, `Session Management`

### Community 6 - JetBrains Agent Features

- Nodes (3): `Built-in Agent System`, `Slash Command System`, `MCP Support`

### Community 7 - JetBrains AI Support

- Nodes (3): `Dual AI Engine Support`, `Claude Code Support`, `OpenAI Codex Support`

### Community 8 - Claude Code Game Studio

- Nodes (13): `Claude Code Game Studios`, `49 AI Agents`, `72 Workflow Skills`, `Studio Hierarchy`, `Unreal Engine 5`, `Unreal Specialist`, `Game Studio Sub-Agents`, `12 Specialized Agents` (+5 more)

### Community 9 - AI Game Dev Tool Landscape

- Nodes (7): `AI Game DevTools (AI-GDT)`, `LLM Tools`, `World Model & Agents`, `AI Code Tools`, `Multimodal Asset Tools`, `Unity AI Tools`, `Unreal Engine AI Tools`

### Community 10 - Unity Client and Motion Control

- Nodes (22): `HTFramework`, `Unity Rapid Development Framework`, `Hotfix Module`, `ECS Module`, `FSM Module`, `Network Client Module`, `Debugger Module`, `5分钟完全了解Unity-运动插值` (+14 more)

### Community 11 - Game Narrative and Text Writing

- Nodes (8): `闲聊：游戏的【叙事文本】怎么做？（现实世界观篇）`, `Game Narrative Text`, `Reality Worldview Writing`, `Minimal Background Description`, `Dialogue as Character Reveal`, `Inner Speech Writing Method`, `Source Accuracy for Worldbuilding`, `Text Adventure Narrative`

### Community 12 - AI Character Asset Pipeline

- Nodes (14): `See-through`, `Anime Character Layer Decomposition`, `2.5D Models`, `23 Semantic Layers`, `PSD Export`, `LayerDiff 3D`, `Marigold Depth`, `Live2D Preprocessing` (+6 more)

### Community 13 - AI Narrative Engine

- Nodes (20): `PlotPilot（墨枢）`, `Narrative Engine Kernel`, `Long-form AI Creation`, `Narrative State Machine`, `Story Bible`, `Chapter Summary Chain`, `Narrative Event Stream`, `Storyline DAG` (+12 more)

### Community 14 - AI Character Skill Workflow

- Nodes (13): `2D Character Starter`, `Codex Skill Workflow`, `2D Playable Character Image Generation`, `Chroma Green Screen Output`, `3:2 Character Canvas`, `Character Simplification Mode`, `Concept Target Style Transfer`, `Strict Pose Transfer` (+5 more)

### Community 15 - Sprite Video Processing Pipeline

- Nodes (20): `Sprite Video Lab`, `Local Web Sprite Tool`, `2D Sprite Resource`, `Video/GIF/Image/Sequence Import`, `Frame Range Trimming`, `Fixed Interval Frame Sampling`, `Background Removal Pipeline`, `Chroma Key Background Removal` (+12 more)

## Suggested Questions

- **现实世界观的游戏叙事文本，如何用台词和行动替代空泛背景描写？**
  - 新增视频给出了一条从“少写背景”到“让台词暴露人物内心”的写法链条。
- **游戏文本创作者如何用 Inner Speech 先跑通人物动机和事件逻辑？**
  - 转写明确把内部语言作为避免前后矛盾、积累关键表达和再查资料补细节的方法。
- **Unity Motion Interpolation 与 HTFramework / Unity 客户端框架资料之间能形成什么学习路径？**
  - 新增视频让 Unity 社区从“框架模块”延伸到“角色运动手感 / 动作游戏控制”。
- **Unity 的 Update / FixedUpdate / LateUpdate 节奏如何影响相机跟随抖动？**
  - Bilibili 转写已经提供了一个从问题成因到显示层插值实现的教程链条。
- **DevelopAUnityActionGameIn5Min 项目仓库是否值得单独抓取进图谱？**
  - 视频简介显式给出 GitHub 仓库；当前已添加视频资料卡与转写，但尚未抓取仓库源码或 README。
- **HTFramework -> Unity AI Tools -> Multi-Engine Support -> Game Studio Sub-Agents 这条跨引擎路径是否仍成立？**
  - 新增 Unity 动作游戏教程后，Unity 实战资料更多，值得重新审视这条桥接。
- **Game Studio Sub-Agents 与 Claude Code Game Studios 在实践中应该如何区分？**
  - 它们仍是当前 AI 游戏工作流社区中的核心对照。
- **See-through 能否作为“单张角色概念图 -> 可动画 PSD 资产”的预处理入口？**
  - 新增资料卡记录了它的 23 语义层、PSD 输出、遮挡补全和 Live2D 前处理边界。
- **See-through 与 Unity Motion Interpolation / Display Layer Interpolation 能否组合成角色立绘动效链路？**
  - 前者负责拆层和 PSD 资产准备，后者代表运行时显示层插值与相机跟随问题，两者可作为“资产生成 -> 运行时表现”的跨阶段对照。
- **PlotPilot 的 Narrative State Machine 能否作为游戏剧情线 / NPC 叙事线的状态治理参考？**
  - 新增资料卡记录了 Story Bible、故事线 DAG、伏笔注册表、事件流和章级摘要链，可对照现有游戏叙事文本方法论。
- **PlotPilot 与 Game Studio Sub-Agents 如何分工？**
  - PlotPilot 偏叙事生产内核和质量治理，Game Studio Sub-Agents 偏多角色协作组织；两者可能形成“创作引擎 + 生产团队代理”的组合路径。
- **2D Character Starter 能否作为“角色概念图 -> 可测试绿幕角色素材”的入口？**
  - 新增资料卡记录了 `s`、`ct`、`p`、`sq`、`cs` 五类产图模式，可用于角色简化、画风迁移、姿势迁移、单帧动作方向探索和绿幕待机角色生成。
- **2D Character Starter 与 See-through 应如何分工？**
  - 2D Character Starter 偏生成和修正角色起始素材，See-through 偏把已有角色图拆成可编辑 PSD / Live2D 前处理层，两者可组成“起始素材生成 -> 拆层与动画预处理”的链路。
- **Sprite Video Lab 能否补上“动态图 / 视频素材 -> 游戏 Sprite”的处理段？**
  - 新增资料卡记录了视频区间截取、固定间隔抽帧、绿幕 / AI 抠图、Luma 亮部 VFX 保留、MAGIC 二次处理和透明 WebM 导出。
- **Sprite Video Lab、2D Character Starter、See-through 如何串成角色资产链路？**
  - 2D Character Starter 负责起始角色素材生成和姿势探索，Sprite Video Lab 负责动态图 / 序列帧透明化与导出，See-through 负责单图拆层和 PSD / Live2D 前处理。
