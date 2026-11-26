# lanshu-create-ai-presenter-video

**English** · [简体中文](README.zh-CN.md)

[![Agent Skill][skill-badge]](SKILL.md) [![Release][release-badge]][latest-release] [![Validate Skill][ci-badge]][ci] [![License: MIT][license-badge]](LICENSE) [![GitHub stars][stars-badge]][stargazers]

Turn a topic or script into an explainer video with the coding agent you already use. Give it one authorized portrait for a digital presenter with synced captions and animated keywords, or choose one of twelve styles for an animated explainer.

## AI presenter demo

[![Cover of the original 30-second AI presenter demo](https://lanshuagent.com/media/2090436108115136643-yEcbdXq1vqOmVYK3.webp)](https://lanshuagent.com/works/presenter-video/)

[Watch the original 30-second demo](https://lanshuagent.com/media/2090436108115136643-yEcbdXq1vqOmVYK3.mp4) · [Launch post on X](https://x.com/LufzzLiz/status/2090436108115136643)

[Two routes](#two-routes) · [Installation](#installation) · [Quick start](#quick-start) · [Style gallery](#style-gallery) · [Reference guides](#reference-guides)

## Two routes

| Route | What you provide | What you get | Paid generation | Format |
| --- | --- | --- | --- | --- |
| **AI presenter** | A topic or script and an authorized portrait of one adult | A talking presenter with lip-sync, synced captions, and animated keyword graphics | Voice and presenter video, depending on the providers you use | Default 9:16, with other aspect ratios available |
| **Styled explainer** | A topic or script, with optional recorded narration | Animated scenes timed to the narration in one of twelve styles | MiniMax voice generation if used | 16:9 |

The agent can infer the route from the request and show the style gallery when you want to compare options. The [styled explainer guide](references/styled-explainer.md#optional-bringing-a-digital-human-into-a-styled-film) also shows how to feature a presenter in a styled film.

### Optional inputs

- **Recorded narration** can serve as the final voice track on either route. A voice sample is for cloning and requires explicit authorization.
- **Supporting media** can include screen recordings, images, B-roll, and brand assets.
- **Delivery preferences** can include platform, duration, aspect ratio, visual treatment, watermark, music, and call to action.

## Requirements

- An Agent Skills client such as Claude Code, Codex, Gemini CLI, Cursor, OpenCode, or GitHub Copilot. Any coding agent that can read files and run shell commands can also follow [SKILL.md](SKILL.md) directly.
- Python 3.9+, Bash, FFmpeg with `ffprobe`, `jq`, `awk`, and `sed`.
- **AI presenter** uses voice, presenter video, lip-sync, and speech recognition capabilities available in your environment. A cloud CLI, an API, or local models can provide them. Use an available compositor such as HyperFrames for editing and rendering.
- **Styled explainer** uses Node.js with `npx`, `rsync`, Python with `numpy` for sound, and network access for fonts. Set `PYTHON` to select the interpreter if needed. Set `MINIMAX_API_KEY` for narration with word timestamps, or use a recording with matching script text. With a recording, the tool estimates word timing.

## Installation

The repository is the skill. Clone it into your client's skills directory and keep the folder name `lanshu-create-ai-presenter-video` so it matches the name in `SKILL.md`.

| Client | Skills directory | Explicit invocation |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` or project-level `.claude/skills/` | `/lanshu-create-ai-presenter-video` |
| Codex | `~/.codex/skills/` | `$lanshu-create-ai-presenter-video` |
| Other clients | See the [Agent Skills client list](https://agentskills.io/clients) | Client-specific |

This command installs the skill for Codex.

```bash
git clone https://github.com/cclank/lanshu-create-ai-presenter-video.git \
  "$HOME/.codex/skills/lanshu-create-ai-presenter-video"
```

Run `git pull` inside the installed directory to update. For a tagged version or a ready-to-extract skill ZIP, see [GitHub Releases](https://github.com/cclank/lanshu-create-ai-presenter-video/releases) and the [changelog](CHANGELOG.md).

You can also clone the repository anywhere and ask your agent to read and follow `/path/to/lanshu-create-ai-presenter-video/SKILL.md`.

## Quick start

Give your agent a topic or script. Attach a portrait for the presenter route, or name a visual style for the styled route.

**AI presenter**

```text
Turn this script and authorized portrait into a 30-second 16:9 presenter video with synced captions and animated keywords.
```

**Styled explainer**

```text
Make a 16:9 animated explainer on KV cache in the chalkboard style. Show me the script before generating narration.
```

<details>
<summary>Run the scripts manually</summary>

Set `SKILL_DIR` to the directory containing `SKILL.md`.

```bash
SKILL_DIR="$HOME/.codex/skills/lanshu-create-ai-presenter-video"
```

**Create a presenter job.** Use the confirmation flags after checking the image rights and the presenter's age.

```bash
PRESENTER_JOB="$HOME/Videos/my-presenter-video"

python3 "$SKILL_DIR/scripts/init_job.py" \
  --job-dir "$PRESENTER_JOB" \
  --presenter-image "$HOME/Pictures/presenter.png" \
  --topic "Context engineering in one minute" \
  --duration 60 \
  --aspect 9:16 \
  --rights-confirmed \
  --adult-presenter-confirmed
```

Record the manual review and upload approvals in `job.json` when obtained. Run preflight to see which inputs and approvals are ready. Use the state checker after recording the artifacts from each stage. [Presenter production guide](references/generation.md)

```bash
python3 "$SKILL_DIR/scripts/preflight.py" "$PRESENTER_JOB/job.json"
python3 "$SKILL_DIR/scripts/check_state.py" "$PRESENTER_JOB/job.json" --write
```

**Preview a styled explainer.** This example uses the bundled KV cache script. It builds a draft with estimated word timing and no voice generation.

```bash
EXPLAINER_DIR="$SKILL_DIR/explainer"
STYLED_JOB="$HOME/Videos/kv-cache-explainer"

python3 "$SKILL_DIR/scripts/init_job.py" \
  --job-dir "$STYLED_JOB" \
  --route styled \
  --explainer-style v8-chalkboard \
  --topic "KV cache"

cd "$STYLED_JOB"
mkdir -p story
cp "$EXPLAINER_DIR/stories/kv-cache/script.md" story/script.md
python3 "$EXPLAINER_DIR/tools/story.py" story --dry
bash "$EXPLAINER_DIR/tools/new_film.sh" v8-chalkboard story film
(cd film && npx --yes hyperframes@0.8.81 preview)
```

For a voiced film, run `story.py story` with MiniMax or `story.py story --audio narration.wav` with your own recording. The guide covers writing scenes in `film/beats.js`, reviewing the storyboard, and running `qa.py` and `render.sh`. [Styled production guide](references/styled-explainer.md)

</details>

## Style gallery

Both previews use the same KV cache topic. Each style has a starter, a reusable kit, and an example project. All twelve include word-timed captions, chapter labels, a closing line, and a one-frame recap.

### Nine visual styles

Editorial, tech HUD, notebook, paper pop-up, pop comic, 3D one-take, drafting sheet, chalkboard, and clay town. The grid shows the same moment in all nine looks.

[![The same moment of the KV cache explainer in nine visual styles](explainer/gallery/nine-styles-kv-cache.jpg)](explainer/STYLES.md)

### Three narrative styles

**Derivation** builds the explanation on one canvas. **Casefile** follows the evidence to a cause. **Sitcom** uses an everyday scene to make the concept concrete.

[![The KV cache topic told as a derivation, a casefile, and a sitcom](explainer/gallery/three-narrative-styles-kv-cache.jpg)](explainer/STYLES.md)

[Full style guide](explainer/STYLES.md) · [KV cache example projects](explainer/examples/README.md)

## How it works

- **AI presenter.** Lock the script and narration, run a low-cost presenter pilot, then generate and edit the full video. For providers with duration caps, the segment planner splits narration at real pauses. The agent records the selected provider, model, parameters, and task IDs for each capability.
- **Styled explainer.** Pick a style before writing the script, time it to the narration, and build a playable draft from the starter. Use the storyboard and word timings to animate each spoken idea, then check and render the film.

The approved narration sets the timing for lip-sync, captions, graphics, and cuts. `check_state.py` reads job artifacts to determine which production stage has passed.

| Stage | Evidence |
| --- | --- |
| `intake` | Job manifest |
| `content_locked` | Passing preflight report, script, and beat sheet |
| `audio_locked` | Decodable narration and timing evidence |
| `visual_plan_locked` | Timeline, storyboard, and approved plan |
| `presenter_generated` | Selected presenter video or performed styled film, plus visual review |
| `composition_checked` | Composition report |
| `rendered` | Decodable render with video and audio |
| `verified` | Master, share copy, and delivery report with passing loudness |

On the styled route, audio lock requires narration and a matching `story.json` with word timing. Both encodes undergo full decode and loudness checks. The delivery report records black or frozen frames in the master, and the delivery includes a contact sheet. The styled workflow also exports SRT subtitles and a cover frame.

## Defaults

| Setting | Default |
| --- | --- |
| Format | Presenter 9:16 at 1080 × 1920. Styled 16:9 at 1920 × 1080. Both 30 fps. |
| Duration | 45–75 seconds from a topic. A supplied script follows its natural length. |
| Voice | Stock voice or an authorized sample for the presenter route. Styled uses MiniMax `Chinese (Mandarin)_Reliable_Executive` by default, with speed set per style. |
| Structure | A hook, 2–4 content beats, and a concise close. Music and calls to action are opt-in. |
| Loudness | −16 LUFS ±0.5 LU, with true peak ≤ −1 dBTP. Set `PROGRAM_LUFS` to change the target. |

## Cost and privacy

- The agent confirms image rights, adult status, upload approval, and any voice-cloning authorization before the relevant remote action.
- Before a paid call, the agent lists the assets to upload, requested units, known price, pilot size, and retry limit. Segment quotes use the total requested seconds. Interrupted requests are checked by task ID before resubmission, and the agent stops after three rejected paid candidates.
- Styled narration is cached per line. Editing one line regenerates just that line. Once narration is locked, comparing the nine visual styles requires no additional voice calls.
- Request records omit credentials and temporary URLs. Preflight and delivery reports use file names or job-relative paths. The styled route fetches font subsets from Google Fonts and jsDelivr, and keeps QA snapshots local with `--describe false`.

## Reference guides

| Guide | What it covers |
| --- | --- |
| [Generation](references/generation.md) | Intake, script, narration, provider selection, presenter prompts, billing, and segment planning |
| [Editing](references/editing.md) | Timeline, segment seams, captions and keyword graphics, cover, and export |
| [QA and recovery](references/qa-recovery.md) | Acceptance checks and fixes for lip-sync, identity, seams, loudness, and remote tasks |
| [Styled explainer](references/styled-explainer.md) | Story pipeline, scene animation, review, render, and adding a presenter to a styled film |
| [Style kits](explainer/KITS.md) · [HyperFrames notes](explainer/GOTCHAS.md) | Starter format, tooling, and rendering pitfalls |

### Main scripts

| Script | Purpose |
| --- | --- |
| [`init_job.py`](scripts/init_job.py) | Initializes a self-contained job directory and copies inputs |
| [`preflight.py`](scripts/preflight.py) | Checks inputs, manual review, and remote-generation approvals |
| [`plan_segments.py`](scripts/plan_segments.py) | Splits narration at suitable pauses when requests have a duration cap |
| [`check_state.py`](scripts/check_state.py) | Computes the current stage from recorded artifacts |
| [`finalize_delivery.sh`](scripts/finalize_delivery.sh) | Builds and checks the master and share copies, contact sheet, and delivery report |

The [styled tools](explainer/tools/) include `story.py` for narration and word timing, `new_film.sh` and `new_topic.sh` for drafts, `docs.py` for production documents, `qa.py` for composition and stillness checks, and `render.sh` for video and subtitles.

### Repository layout

```text
SKILL.md        Skill entry point
agents/         Codex display metadata
assets/         Job manifest template
references/     Production guides
scripts/        Job setup, verification, and delivery
explainer/      HyperFrames runtime, style kits, starters, stories, and examples
tests/          Offline smoke tests and narration timing checks
```

## Testing

Run both smoke tests using synthetic media, without calling generation providers. They require Python 3.9+, FFmpeg, and `jq`. The styled test also uses Node.js and `rsync`.

```bash
bash tests/smoke.sh
bash tests/explainer_smoke.sh
```

CI runs metadata, portability, and source checks alongside both smoke tests on pushes and pull requests.

## Contributing

Issues and pull requests are welcome, especially compatibility reports and improvements to production or QA. To add a style, review the [kit contract](explainer/KITS.md) and [starter brief](explainer/STARTER-BRIEF.md).

## Author

Created and maintained by **岚叔 (Lanshu)**. [@LufzzLiz on X](https://x.com/LufzzLiz)

## License

[MIT License](LICENSE) · [Third-party fonts, libraries, and services](explainer/THIRD_PARTY.md)

[skill-badge]: https://img.shields.io/badge/Agent_Skill-SKILL.md-111827
[release-badge]: https://img.shields.io/github/v/release/cclank/lanshu-create-ai-presenter-video
[latest-release]: https://github.com/cclank/lanshu-create-ai-presenter-video/releases/latest
[ci-badge]: https://github.com/cclank/lanshu-create-ai-presenter-video/actions/workflows/validate.yml/badge.svg
[ci]: https://github.com/cclank/lanshu-create-ai-presenter-video/actions/workflows/validate.yml
[license-badge]: https://img.shields.io/badge/License-MIT-F4C430.svg
[stars-badge]: https://img.shields.io/github/stars/cclank/lanshu-create-ai-presenter-video?style=flat
[stargazers]: https://github.com/cclank/lanshu-create-ai-presenter-video/stargazers
