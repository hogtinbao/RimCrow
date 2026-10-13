# RimCrow v0.24.8 — RimWorld 1.7 兼容修复构建

> 本仓库是基于上游 RimCrow v0.24.7 的**修复构建**，只解决一件事：让 RimCrow 在 RimWorld 1.7（Waylith 引擎）下重新识别游戏主程序、正常启动游戏、正常读取游戏日志。
> 功能、界面与打包方式全部来自上游，没有做其它改动，也没有新增功能。

## 为什么需要这个构建

RimWorld 1.7 把 Unity 换成了自研的 Waylith 引擎，随之改了两处磁盘布局：

| 项目 | 1.6（Unity） | 1.7（Waylith） |
| --- | --- | --- |
| Windows 主程序 | `RimWorldWin64.exe` | `RimWorld.exe` |
| 玩家日志 | `<用户数据>\Player.log` | `<用户数据>\Logs\Player.log` |

v0.24.7 的候选清单里只有旧名字，检测到的可执行文件为空 → 环境被判定不可用 → 日志出现「环境 default 路径失效，进入锁定模式」，于是既无法配置主程序，也无法通过 RimCrow 启动游戏。

## 这个构建改了什么

1. 主程序候选加入 `RimWorld.exe`（旧名保留，1.6 及更早照旧可用）：恢复安装路径检测，解除锁定模式。
2. 玩家日志候选优先 `<用户数据>\Logs\Player.log`（旧路径保留兜底）：游戏日志查看恢复正常。
3. 进程名候选加入 `RimWorld.exe`：经 Steam 启动后能正确识别「游戏运行中」，回到静默监测。

对应的源码修复与回归测试在本仓库分支 `fix/rimworld-1.7-waylith-detection`，并已向上游提交：
<https://github.com/Inky-Feather/RimCrow/pull/26>

## 下载与使用

> **下载渠道只有 GitHub**：本仓库的 Releases 页面，没有蓝奏云或其它网盘镜像。想自己构建/运行源码请看下一节。

1. 在 Releases 下载 `RimCrow-v0.24.8-windows.zip`。
2. 解压后运行 `RimCrow/RimCrow.exe`（Windows，需要 WebView2 Runtime）。
3. 首次打开后确认「设置 → 路径」里的游戏安装目录指向 Steam 的 RimWorld 目录。

### 压缩包内容

- `RimCrow/RimCrow.exe`：程序本体
- `RimCrow/tools/`：ripgrep、steamcmd、todds（贴图工具）
- `RimCrow/toolmods/RimCrowCompanion/`：随包带的管理器配套 Mod
- `RimCrow/data/`：公开社区数据库（SteamDB、替换规则、联机兼容表等）
- `RimCrow/README.md`、`RimCrow/LICENSE`、`RimCrow/doc/致谢.md`

## 自行构建（Windows / macOS / Linux 通用）

本构建的源码就在本仓库，任何系统都能按下面的步骤自己构建或直接跑源码。前置：Git、Node.js 18+（构建前端）、Python 3.11+（推荐 [uv](https://docs.astral.sh/uv/)，会自动准备 3.11 环境）；界面运行时依赖：Windows 需 WebView2 Runtime，macOS 用系统 WebView，Linux 需 WebKitGTK（如 `libwebkit2gtk-4.1` 及对应 gir 包）。

```bash
# 1. 克隆本仓库（含子模块）。分支 release/v0.24.8 = 含 RimWorld 1.7 兼容修复
#    本仓库地址：打开仓库页面用 Code 按钮复制，或直接用你的 fork 地址替换下面这行
git clone --recurse-submodules -b release/v0.24.8 <本仓库地址>
cd RimCrow
#    克隆时若漏了 --recurse-submodules，补执行：
git submodule update --init --recursive

# 2. Python 依赖
uv sync

# 3. 前端资源（必须，否则界面空白）
cd frontend && npm ci && npm run build && cd ..

# 4. Steamworks 运行库（Steam 相关功能需要；脚本按当前系统自动放置 dll/so/dylib）
uv run python scripts/setup_steamworks_runtime.py --sdk /path/to/steamworks_sdk_*.zip

# 5. 启动
uv run python main.py                          # 桌面窗口
uv run python main.py --browser                # 浏览器模式
uv run python main.py --reset-window-state     # 窗口跑到屏幕外时重置

# 6. 可选：自己打发布包（Windows/Linux 单文件，macOS .app），产物在 dist/
uv run python pack_pyinstaller.py

# 7. 可选：跑测试
uv run pytest -q tests
```

- 运行与打包都需要第 4 步产生的 `tools/steamworks/` 运行库；打包还会带上 `data/` 下的社区数据库与 `toolmods/RimCrowCompanion`，缺少时打包脚本会跳过、运行时会按需下载。
- 平台差异与上游一致：Windows/Linux 打包为单文件，macOS 打包为 `.app`；macOS/Linux 的构建流程未在本机构建验证，命令与上游一致。

## 使用须知

- **模组列表被重置**：RimWorld 1.7 在 `ModsConfig.xml` 版本号不匹配时会清空启用列表，这是游戏自身行为（与 RimCrow 无关）。首次以 1.7 启动后，在 RimCrow 里重新激活列表再进入游戏即可。
- 本包只包含程序与运行所需资源，不含任何个人配置、模组列表或日志。
- 上游发布正式版本后，建议换回上游版本。

## 致谢

- **原作者：[Inky Feather](https://github.com/Inky-Feather)** —— RimCrow（原名 RimModManager）的作者与维护者。本构建的全部功能与打包流程都来自上游仓库 <https://github.com/Inky-Feather/RimCrow>，版权与许可证归原作者所有。
- **[@1579486875](https://github.com/1579486875)** —— 报告并定位了 RimWorld 1.7 导致的游戏识别失败问题（issue #24 / PR #25，含 1.7 磁盘布局对照表与实机验证）。本构建的修复基于这份报告与参考补丁完成。

更多说明见 [doc/致谢.md](./doc/致谢.md)。

## 许可证

MIT，见 [LICENSE](./LICENSE)。上游仓库：<https://github.com/Inky-Feather/RimCrow>

---

### About this build (English)

This repository hosts a rebuilt package of RimCrow v0.24.7 with the RimWorld 1.7 (Waylith engine)
compatibility fix: the renamed Windows executable (`RimWorld.exe`), the relocated player log
(`<user data>/Logs/Player.log`) and the new process name used for game-running detection. Nothing
else was changed. Building from source works on Windows, macOS and Linux (clone branch
`release/v0.24.8` with `--recurse-submodules`, `uv sync`, build `frontend/`, then
`uv run python main.py`) - the build steps are listed above. Downloads are served from this
repository's Releases only (no third-party mirror). All credit goes to the upstream author
**Inky Feather** and to **[@1579486875](https://github.com/1579486875)**, who reported and
diagnosed the 1.7 breakage in issue #24 / PR #25. Licensed under MIT; see `LICENSE`. Prefer the
upstream release once it ships the fix: <https://github.com/Inky-Feather/RimCrow/releases>
