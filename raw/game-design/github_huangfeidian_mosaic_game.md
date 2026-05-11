---
source_url: "https://github.com/huangfeidian/mosaic_game"
type: github-repository-summary
title: "mosaic_game 分布式大世界游戏服务端资料卡"
captured_at: 2026-05-11T21:34:52+08:00
repo: "huangfeidian/mosaic_game"
default_branch: "master"
repo_language: "C++"
repo_license: null
topics:
  - Game Server
  - Distributed Space
  - MMO Server
  - C++17
  - CMake
  - MongoDB
  - Redis
  - Code Generation
  - RecastNavigation
related_source:
  - "https://github.com/huangfeidian/mosaic_game"
  - "https://raw.githubusercontent.com/huangfeidian/mosaic_game/master/Readme.md"
  - "https://github.com/huangfeidian/mosaic_game/blob/master/CMakeLists.txt"
  - "https://github.com/huangfeidian/mosaic_game/tree/master/roles/server"
  - "https://github.com/huangfeidian/mosaic_game/tree/master/generator/script"
capture_scope: "GitHub 仓库元信息、Readme.md、CMakeLists.txt、目录树与关键目录 API 核对；未克隆、未编译、未运行服务端"
---

# mosaic_game 分布式大世界游戏服务端资料卡

## 来源事实

- 用户提供 GitHub 仓库：`https://github.com/huangfeidian/mosaic_game`。
- 使用 Codex Chrome 插件打开仓库页面时，浏览器标题显示为 `huangfeidian/mosaic_game: game server for distributed space`。
- GitHub API 截取时，仓库 `huangfeidian/mosaic_game` 为公开仓库，默认分支为 `master`，主语言为 `C++`，GitHub 未识别到许可证。
- GitHub API 截取时，仓库创建时间为 `2020-05-25 02:15:32 +08:00`，最近 push 时间为 `2026-03-19 01:45:01 +08:00`，页面更新时间为 `2026-05-11 13:55:40 +08:00`。
- GitHub API 截取时，仓库约 `190` stars、`49` forks、`0` open issues；这些计数是动态数据，只代表本次采集时状态。
- GitHub Release API 返回空列表，说明本次未发现 GitHub release。

## 仓库定位

Readme.md 将 mosaic_game 定位为基于 `C++17` 的游戏服务器示例项目，支持分布式大世界的无缝迁移功能，并注明有配套电子书《深入理解游戏服务端：从网络通信到无缝迁移》介绍完整实现细节。

仓库不是 Unreal、Unity 或 Godot 客户端项目，而是偏服务端架构与工程实践的代码库。README 明确列出服务端与客户端角色代码、数据导出与代码生成工具、MongoDB / Redis 示例配置，以及 Linux 下的批量启动脚本。

## 仓库结构事实

仓库根目录包含：

- `CMakeLists.txt`
- `Readme.md`
- `src/`
- `include/`
- `common/`
- `roles/`
- `tools/`
- `generator/`
- `data/`
- `deploy/`
- `deps/`
- `third_party/`
- `test/`

GitHub 递归树 API 截取结果：

- blob 文件总数：`708`
- 目录总数：`204`
- `.cpp` 文件：`223`
- `.h` / `.hpp` 文件：`279`
- Python 文件：`12`
- Markdown 文件：`1`
- shell 文件：`1`
- 递归树未被 GitHub API 标记为 `truncated`

`src/` 下分为 `anchor`、`encrypt`、`network`、`stub`、`tasks`、`utility`。`common/` 下包含 `db_logic`、`redis_logic`、`server_utility`、`entity_common`、`property`、`team_handler`、`group_handler`、`extend_detour` 等通用业务与基础组件。

## 服务角色事实

`roles/server/` 下包含这些服务进程目录：

- `mgr_server`：管理与协调中心
- `gate_server`：网关入口
- `service_server`：业务服务节点
- `space_server`：场景 / 游戏逻辑节点
- `map_server`：地图相关服务
- `db_server`：MongoDB 持久化相关服务
- `redis_server`：Redis 读写与缓存相关服务

`roles/client/` 下包含：

- `basic_client`
- `cli_client`

README 还说明默认本地测试拓扑包含 `mgr_server`、`redis_server_0`、`db_server_0`、`service_server_0`、`gate_server_0`、`gate_server_1`、`space_server_0`、`space_server_1` 和 `map_server_0`。

## 构建与依赖事实

顶层 `CMakeLists.txt` 要求 CMake `3.12+`，设置 `C++17`，开启 `CMAKE_UNITY_BUILD`，并把安装前缀固定为仓库内 `deploy/`。非 MSVC 编译器默认启用 `-Wall -Wextra -Wno-unused-parameter`；`WITH_ASAN` 默认开启时会加入 AddressSanitizer 参数。

CMake 直接查找或链接的依赖包括 `nlohmann_json`、`fmt`、`spdlog`、OpenSSL、Threads、`any_container`、`http_utils`、`cxxopts`、`typed_string`、`miniz`、`tinyxml2`、`xlsx_reader`、`typed_matrix`、Boost `system` / `date_time`、`magic_enum` 等。

README 说明 `deps/install_deps.py` 会把缺失依赖拉到 `deps/sources/`，在 `deps/builds/` 下构建，并安装到 `deps/packages/`。它处理的依赖包括 `mongo-c-driver`、`mongo-cxx-driver`、`hiredis`、`xlsx_reader`、`distributed_space`、`behavior_tree`、`formula_tree`、`property_sync`、`http_utils` 等。

## 代码生成与运行事实

README 说明项目包含两类生成流程：

- C++ 工具：`export_xlsx`、`generate_rpc`、`generate_property_sync`
- Python 封装：`generator/script/xlsx_gen.py`、`rpc_gen.py`、`property_gen.py`、`debug_gen.py`

README 强调，首次 CMake 初始化后需要先编译并安装 `generate_rpc`，再执行 `python ./rpc_gen.py all` 和 `python ./property_gen.py all` 生成 RPC 与属性同步相关代码，然后重新配置并全量编译。

运行配置主要位于 `data/config/`，包括服务拓扑、MongoDB 连接、Redis 连接和 gate server 列表。README 说明 Linux 下可通过 `deploy/scripts/run_servers.py`、`run_clients.py`、`stop_servers.py` 批量启动或停止；这些脚本当前显式限制为 Linux，Windows 下更适合单独启动某个服务进程调试。

## 适合用途

- 研究分布式大世界 / 无缝迁移游戏服务端架构。
- 学习多进程服务端角色拆分，例如 gateway、manager、space、map、db、redis、service。
- 参考 C++17 + CMake + MongoDB + Redis 的游戏后端工程组织方式。
- 参考游戏服务端的代码生成流程，包括 RPC、属性同步、导表和调试命令生成。
- 作为 MMO / 大世界后端资料库中的实现样本，而不是只读概念文章。

## 不适合用途

- 不适合直接当作可商用模板复用；GitHub 未识别许可证，本次也未发现根目录 LICENSE 文件。
- 不适合直接在 Windows 上按 README 的一键脚本运行；README 明确说明批量启动脚本当前限制为 Linux。
- 不适合跳过代码生成步骤直接全量编译；README 明确要求先生成 RPC 与 property 相关代码。
- 不适合当作客户端玩法、渲染或引擎资料；仓库重心是游戏服务器与分布式空间服务。
- 不适合在未配置 MongoDB、Redis、依赖包与数据路径前直接启动完整拓扑。

## 我的判断

这个仓库适合加入资料库，但应该归到“游戏服务端 / 游戏架构”，而不是“虚幻相关”。依据是仓库 README、GitHub topics 和目录结构都指向 C++ 游戏服务器、分布式空间、MongoDB、Redis、服务拓扑和代码生成，并没有 Unreal Engine 相关目录或插件描述。

我的判断是，它的价值不在于“可直接拿来跑一个游戏”，而在于展示一个中大型游戏服务端的工程骨架：多角色进程、通用业务组件、数据导出、RPC 生成、属性同步、Mongo/Redis 配置和 Linux 部署脚本都放在同一个仓库里。依据是 README 对目录职责、服务角色、生成流程和默认拓扑的详细说明。

我的判断是，如果后续要深入研究，优先阅读路径应是 `Readme.md` -> `roles/server/` -> `common/` -> `src/network` / `src/stub` -> `generator/script` -> `data/config`。依据是 README 的构建与运行流程显示，理解服务角色和生成代码的关系比先看单个业务文件更重要。

我的判断是，许可证缺失是复用时的主要风险点。GitHub API 返回 `license: null`，根目录列表也没有 `LICENSE` 文件；因此它可以作为学习材料索引，但在复制代码、改造成商业项目或再分发前，需要先获得作者授权或确认许可证。

## 后续检索关键词

- 中文：`mosaic_game`、`分布式大世界 游戏服务器`、`游戏服务端 无缝迁移`、`C++17 游戏服务端`、`游戏服务端 RPC 生成`、`MongoDB Redis 游戏后端`
- English: `mosaic_game`, `distributed space game server`, `C++17 game server`, `MMO server architecture`, `game server code generation`, `MongoDB Redis game backend`

## 来源

- 用户提供材料：GitHub 仓库 `https://github.com/huangfeidian/mosaic_game`
- 仓库 README：`https://raw.githubusercontent.com/huangfeidian/mosaic_game/master/Readme.md`
- 顶层 CMake：`https://github.com/huangfeidian/mosaic_game/blob/master/CMakeLists.txt`
- 服务端角色目录：`https://github.com/huangfeidian/mosaic_game/tree/master/roles/server`
- 代码生成脚本目录：`https://github.com/huangfeidian/mosaic_game/tree/master/generator/script`
- 部署脚本目录：`https://github.com/huangfeidian/mosaic_game/tree/master/deploy/scripts`
