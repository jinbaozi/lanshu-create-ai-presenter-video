# lanshu-create-ai-presenter-video

[English](README.md) · **简体中文**

[![Agent Skill][skill-badge]](SKILL.md) [![Release][release-badge]][latest-release] [![Validate Skill][ci-badge]][ci] [![License: MIT][license-badge]](LICENSE) [![GitHub stars][stars-badge]][stargazers]

把一个主题或一份文案交给你正在用的 AI agent，就能做成讲解视频。提供一张授权人物图，可以做带字幕和关键词动效的数字人口播。也可以从十二种风格里挑一种，把内容做成动画讲解。

## 数字人口播演示

[![原版 30 秒数字人口播演示封面](https://lanshuagent.com/media/2090436108115136643-yEcbdXq1vqOmVYK3.webp)](https://lanshuagent.com/works/presenter-video/)

[播放 30 秒原片](https://lanshuagent.com/media/2090436108115136643-yEcbdXq1vqOmVYK3.mp4) · [X 原始发布](https://x.com/LufzzLiz/status/2090436108115136643)

[两条制作路线](#两条制作路线) · [安装](#安装) · [快速开始](#快速开始) · [风格预览](#风格预览) · [参考文档](#参考文档)

## 两条制作路线

| 路线 | 你需要提供 | 成片效果 | 付费生成 | 画幅 |
| --- | --- | --- | --- | --- |
| **数字人口播** | 主题或文案，以及一张经授权的成年人物图 | 对口型的数字人、同步字幕、关键词动效 | 配音与数字人视频，视所用服务而定 | 默认 9:16，可选其他画幅 |
| **风格化讲解** | 主题或文案，可附已录好的旁白 | 十二种风格之一，按口播节奏逐句演绎 | 使用 MiniMax 配音时计费 | 固定 16:9 |

Agent 会根据需求选择路线，需要比较时可以先看风格预览。也可以把数字人放进风格化成片，具体方法见[风格化讲解指南](references/styled-explainer.md#optional-bringing-a-digital-human-into-a-styled-film)。

### 可选素材

- **旁白和声音样本。** 自己录好的旁白可以直接用作成片配音。声音样本用于克隆，需要明确授权。
- **辅助素材。** 可以提供屏幕录制、图片、B-roll 或品牌素材。
- **交付偏好。** 可以指定平台、时长、画幅、视觉风格、水印、音乐和结尾引导。

## 运行环境

- Claude Code、Codex、Gemini CLI、Cursor、OpenCode、GitHub Copilot 等支持 Agent Skills 的工具。能读文件和执行 shell 命令的 AI agent 也可以直接按 [SKILL.md](SKILL.md) 执行。
- Python 3.9+、Bash、FFmpeg 及 `ffprobe`、`jq`、`awk`、`sed`。
- **数字人口播**需要配音、数字人视频生成、口型同步和语音识别能力，云服务 CLI、API 或本地模型均可。字幕、动效和渲染可用 HyperFrames 等合成工具。
- **风格化讲解**需要带 `npx` 的 Node.js、`rsync`、装有 `numpy` 的 Python，以及下载字体的网络连接。可用 `PYTHON` 指定解释器。配置 `MINIMAX_API_KEY` 可生成配音并取得逐字时间，也可以提供音频和对应文案，由工具估算逐字时间。

## 安装

仓库目录本身就是这个 Skill。克隆到工具的 Skills 目录时，使用 `lanshu-create-ai-presenter-video` 作为目录名，与 `SKILL.md` 中的名称一致。

| 工具 | Skills 目录 | 显式调用 |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` 或项目级 `.claude/skills/` | `/lanshu-create-ai-presenter-video` |
| Codex | `~/.codex/skills/` | `$lanshu-create-ai-presenter-video` |
| 其他工具 | 参见 [Agent Skills 客户端列表](https://agentskills.io/clients) | 以客户端为准 |

下面的命令会将 Skill 安装到 Codex。

```bash
git clone https://github.com/cclank/lanshu-create-ai-presenter-video.git \
  "$HOME/.codex/skills/lanshu-create-ai-presenter-video"
```

更新时，在已安装的目录里运行 `git pull`。也可以下载指定版本的 Skill ZIP，解压到对应的 Skills 目录。[GitHub Releases](https://github.com/cclank/lanshu-create-ai-presenter-video/releases) · [更新记录](CHANGELOG.md)

你也可以把仓库克隆到任意目录，再让 agent 读取并执行 `/path/to/lanshu-create-ai-presenter-video/SKILL.md`。

## 快速开始

给 agent 一个主题或一份文案。做数字人口播时附上人物图，做动画讲解时可以指定想用的风格。

**数字人口播**

```text
把这份文案和授权人物图做成一条 30 秒的 16:9 数字人口播视频，加同步字幕和关键词动效。
```

**风格化讲解**

```text
用黑板报风格做一条 16:9 动画视频，讲清楚 KV cache。先给我看讲稿，再开始配音。
```

<details>
<summary>手动运行脚本</summary>

把 `SKILL_DIR` 指向包含 `SKILL.md` 的安装目录。

```bash
SKILL_DIR="$HOME/.codex/skills/lanshu-create-ai-presenter-video"
```

**创建数字人任务。** 确认图片使用权和出镜人物已成年后，再使用示例中的确认参数。

```bash
PRESENTER_JOB="$HOME/Videos/my-presenter-video"

python3 "$SKILL_DIR/scripts/init_job.py" \
  --job-dir "$PRESENTER_JOB" \
  --presenter-image "$HOME/Pictures/presenter.png" \
  --topic "用一分钟讲清楚上下文工程" \
  --duration 60 \
  --aspect 9:16 \
  --rights-confirmed \
  --adult-presenter-confirmed
```

把已完成的人工素材审核和上传审批记录到 `job.json`，再运行预检。每个制作阶段完成后登记产物，用状态检查脚本确认进度。[数字人制作指南](references/generation.md)

```bash
python3 "$SKILL_DIR/scripts/preflight.py" "$PRESENTER_JOB/job.json"
python3 "$SKILL_DIR/scripts/check_state.py" "$PRESENTER_JOB/job.json" --write
```

**预览风格化讲解。** 以下用仓库现成的 KV cache 讲稿生成草稿，逐字时间用估算值，无需调用配音服务。

```bash
EXPLAINER_DIR="$SKILL_DIR/explainer"
STYLED_JOB="$HOME/Videos/kv-cache-explainer"

python3 "$SKILL_DIR/scripts/init_job.py" \
  --job-dir "$STYLED_JOB" \
  --route styled \
  --explainer-style v8-chalkboard \
  --topic "讲清楚 KV cache"

cd "$STYLED_JOB"
mkdir -p story
cp "$EXPLAINER_DIR/stories/kv-cache/script.md" story/script.md
python3 "$EXPLAINER_DIR/tools/story.py" story --dry
bash "$EXPLAINER_DIR/tools/new_film.sh" v8-chalkboard story film
(cd film && npx --yes hyperframes@0.8.81 preview)
```

制作有声版时，可以运行 `story.py story` 调用 MiniMax 配音，或用 `story.py story --audio narration.wav` 读取自己的旁白。分镜、`film/beats.js` 场景演绎、验收和渲染步骤见[风格化制作指南](references/styled-explainer.md)。

</details>

## 风格预览

下面两组预览都讲 KV cache。每种风格都有自己的风格包、起步模板和示范工程，并支持逐字字幕、章节标记、结尾句和一帧复习图。

### 九种画风

设计师留白、科技信号、手写手帐、纸艺立体书、波普漫画、3D 一镜到底、图纸与注脚、黑板报、黏土小城。下图是同一句讲解在九种画风里的画面。

[![KV cache 讲解同一时刻的九种画风](explainer/gallery/nine-styles-kv-cache.jpg)](explainer/STYLES.md)

### 三种讲法

**推演**在一张画布上逐步推出结论。**案卷**沿着证据找原因。**小剧场**用一个生活场景讲清抽象概念。

[![KV cache 的推演、案卷和小剧场三种讲法](explainer/gallery/three-narrative-styles-kv-cache.jpg)](explainer/STYLES.md)

[完整风格指南](explainer/STYLES.md) · [KV cache 示范工程](explainer/examples/README.md)

## 工作方式

- **数字人口播。** 锁定文案和配音，先做低成本人物试片，再生成和剪辑全片。服务商有时长上限时，按旁白的真实停顿分段。Agent 按能力选用当前环境里的工具，并记录实际使用的服务商、模型、参数和任务 ID。
- **风格化讲解。** 先选风格，再按这个风格写稿、配音和对时。从起步模板生成可播放的草稿，按分镜与逐字时间做场景动效，检查后渲染。

审定后的配音决定口型、字幕、动效和剪辑点。`check_state.py` 根据任务目录里的产物判断完成了哪个阶段。

| 阶段 | 所需证据 |
| --- | --- |
| `intake` | 任务清单 |
| `content_locked` | 通过的预检报告、文案和节拍表 |
| `audio_locked` | 可解码的最终配音与对时记录 |
| `visual_plan_locked` | 时间轴、分镜和已审批的计划 |
| `presenter_generated` | 选定的数字人视频或完成演绎的风格化工程，以及画面审核 |
| `composition_checked` | 合成检查报告 |
| `rendered` | 同时含音频和画面、可解码的渲染文件 |
| `verified` | 母版、分享版，以及响度达标的交付报告 |

风格化路线需要最终旁白和记录逐字时间的 `story.json` 才能锁定配音。母版和分享版都经过完整解码和响度检查。交付时附缩略图总览和报告，报告记录母版检测到的黑帧及静止画面。风格化成片还会导出 SRT 字幕和封面帧。

## 默认设置

| 设置 | 默认值 |
| --- | --- |
| 画幅 | 数字人 9:16（1080 × 1920），风格化 16:9（1920 × 1080）。均为 30 fps。 |
| 时长 | 只有主题时写成 45–75 秒，自带文案时按自然时长。 |
| 配音 | 数字人路线用预设声音或已授权样本。风格化路线默认使用 MiniMax `Chinese (Mandarin)_Reliable_Executive`，语速按风格而定。 |
| 结构 | 开场切入，2–4 个内容段落，简洁结尾。音乐与结尾引导按需添加。 |
| 响度 | −16 LUFS ±0.5 LU，真峰值 ≤ −1 dBTP。可以用 `PROGRAM_LUFS` 调整目标响度。 |

## 费用与隐私

- 涉及远程调用时，agent 会先确认图片使用权、人物已成年、素材上传许可，以及适用的声音克隆授权。
- 首次付费前说明上传素材、请求量、已知价格、试片规模和重试上限。分段报价按总请求时长计算。远程任务中断时先凭任务 ID 查询，连续三个付费候选未通过验收后停止生成。
- 风格化配音按句缓存，改一句只重配那一句。配音锁定后，比较九种画风无需额外调用配音服务。
- 请求记录会去除凭据和临时 URL，预检和交付报告只记录文件名或任务内相对路径。风格化路线从 Google Fonts 和 jsDelivr 下载用到的字体子集，验收截图通过 `--describe false` 留在本机。

## 参考文档

| 指南 | 内容 |
| --- | --- |
| [素材生成](references/generation.md) | 素材检查、文案、配音、服务商选择、人物提示词、报价和分段 |
| [剪辑与合成](references/editing.md) | 时间轴、分段接缝、字幕和关键词动效、封面和导出 |
| [验收与故障恢复](references/qa-recovery.md) | 验收标准和口型、人物一致性、接缝、响度、远程任务的修复方法 |
| [风格化讲解](references/styled-explainer.md) | 讲稿处理、场景演绎、审片、渲染，以及如何在风格化成片中加入数字人 |
| [风格包](explainer/KITS.md) · [HyperFrames 制作笔记](explainer/GOTCHAS.md) | 起步模板、工具和渲染注意事项 |

### 主要脚本

| 脚本 | 用途 |
| --- | --- |
| [`init_job.py`](scripts/init_job.py) | 创建任务目录并复制输入素材 |
| [`preflight.py`](scripts/preflight.py) | 检查输入、素材审核和远程生成审批 |
| [`plan_segments.py`](scripts/plan_segments.py) | 有请求时长上限时，按合适的口播停顿分段 |
| [`check_state.py`](scripts/check_state.py) | 根据已经登记的产物计算制作阶段 |
| [`finalize_delivery.sh`](scripts/finalize_delivery.sh) | 生成并检查母版、分享版，输出缩略图总览和交付报告 |

[风格化工具](explainer/tools/)包括配音与对时的 `story.py`、生成草稿的 `new_film.sh` 和 `new_topic.sh`、生成制作记录的 `docs.py`、检查合成和静止画面的 `qa.py`，以及导出视频与字幕的 `render.sh`。

### 目录结构

```text
SKILL.md        Skill 入口
agents/         Codex 界面显示信息
assets/         任务清单模板
references/     制作指南
scripts/        任务初始化、验收和交付
explainer/      HyperFrames 运行时、风格包、模板、讲稿和示范工程
tests/          离线冒烟测试与配音时序测试
```

## 测试

两组冒烟测试使用合成素材，无需调用生成服务。需要 Python 3.9+、FFmpeg 和 `jq`。风格化测试还会用到 Node.js 和 `rsync`。

```bash
bash tests/smoke.sh
bash tests/explainer_smoke.sh
```

推送和 Pull Request 会触发 CI 中的元数据、可移植性、源码检查和两组冒烟测试。

## 参与贡献

欢迎提交 Issue 和 Pull Request，分享在不同工具和服务商上的使用情况，也欢迎改进制作和验收流程。新增风格可参考[风格包规范](explainer/KITS.md)和[起步模板说明](explainer/STARTER-BRIEF.md)。

## 作者

由 **岚叔（Lanshu）** 创建并维护。[@LufzzLiz on X](https://x.com/LufzzLiz)

## 许可证

[MIT License](LICENSE) · [第三方字体、库和服务](explainer/THIRD_PARTY.md)

[skill-badge]: https://img.shields.io/badge/Agent_Skill-SKILL.md-111827
[release-badge]: https://img.shields.io/github/v/release/cclank/lanshu-create-ai-presenter-video
[latest-release]: https://github.com/cclank/lanshu-create-ai-presenter-video/releases/latest
[ci-badge]: https://github.com/cclank/lanshu-create-ai-presenter-video/actions/workflows/validate.yml/badge.svg
[ci]: https://github.com/cclank/lanshu-create-ai-presenter-video/actions/workflows/validate.yml
[license-badge]: https://img.shields.io/badge/License-MIT-F4C430.svg
[stars-badge]: https://img.shields.io/github/stars/cclank/lanshu-create-ai-presenter-video?style=flat
[stargazers]: https://github.com/cclank/lanshu-create-ai-presenter-video/stargazers
