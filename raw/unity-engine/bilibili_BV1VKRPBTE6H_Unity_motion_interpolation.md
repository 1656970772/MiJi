---
source_url: "https://www.bilibili.com/video/BV1VKRPBTE6H/"
type: video_metadata
title: "5分钟完全了解Unity-运动插值"
bvid: "BV1VKRPBTE6H"
uploader: "数据传输小助手"
upload_date: "2026-05-04"
duration: "7:24"
project_repo: "https://github.com/Kurassier/DevelopAUnityActionGameIn5Min.git"
transcript: "transcripts/yt_30c3774e422c.txt"
audio_cache: "transcripts/downloads/yt_30c3774e422c.m4a"
captured_at: "2026-05-04T06:30:35Z"
contributor: "unknown"
---

# 5分钟完全了解Unity-运动插值

Source: https://www.bilibili.com/video/BV1VKRPBTE6H/

## 来源

- 来源类型：Bilibili 视频元数据
- 采集工具：yt-dlp 2026.03.17
- 采集时间：2026-05-04 14:30:35（Asia/Shanghai）
- 视频作者：数据传输小助手
- BVID：BV1VKRPBTE6H
- 发布时间：2026-05-04
- 时长：7:24
- 转写文件：`transcripts/yt_30c3774e422c.txt`
- 音频缓存：`transcripts/downloads/yt_30c3774e422c.m4a`

## 视频简介

这期是五一特别加更，感谢大家一直以来对我频道的支持。

项目 Github 仓库：https://github.com/Kurassier/DevelopAUnityActionGameIn5Min.git

《铅雨2》游戏试玩群(QQ)：599208349

## 标签

- 游戏开发
- 独立游戏
- 教程
- 操作手感
- UNITY开发教程
- 横版相机控制
- 运动插值
- 动作游戏
- unity
- 角色3C

## 转写摘要

- 主题：Unity 运动插值，用于缓解角色相对屏幕位置抖动。
- 关键问题：`Update` 间隔不固定，以及 `Update` / `FixedUpdate` 执行节奏不匹配。
- 解决方向：把相机跟随放到更合适的更新阶段，并在两次 `FixedUpdate` 之间只插值“显示层”位置。
- 实现要点：保留真实物理位置、`Collider`、`Rigidbody` 不变，把可显示内容挂到单独物体下，通过该显示物体做插值；相机跟随目标改为显示物体。
- 适用范围：视频以 2D 横版游戏为例，但作者说明这套插值系统也可用于第一/第三人称 3D 游戏和相机运动平滑。

## 我的判断

- 归类到 `raw/unity-engine`：标题包含 Unity，标签包含 `UNITY开发教程`、`运动插值`、`横版相机控制`、`角色3C`，视频简介还给出了 Unity 动作游戏项目仓库。
- 已补充语音转写：使用本地 `graphify.transcribe` 调用 `faster-whisper` 生成中文转写；转写结果存在少量近音词和繁简混用，但足以抽取教程主题。
