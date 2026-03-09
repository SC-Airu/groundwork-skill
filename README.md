<p align="center">
  <h1 align="center">Groundwork</h1>
  <p align="center">
    Research the problem space before you build.<br>
    A landscape scan skill for <a href="https://docs.anthropic.com/en/docs/claude-code">Claude Code</a>.
  </p>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href="SKILL.md"><img src="https://img.shields.io/badge/skill-v4-green.svg" alt="Skill v4"></a>
  <a href="https://docs.anthropic.com/en/docs/claude-code"><img src="https://img.shields.io/badge/Claude_Code-skill-blueviolet.svg" alt="Claude Code Skill"></a>
</p>

<p align="center">
  <a href="README.ko.md">한국어</a>
</p>

---

**Tired of building something that already exists?** Groundwork runs 4 parallel research agents to scan the landscape — who has this problem, how they work around it, and what solutions exist — so you can make informed decisions before writing a single line of code.

## Quick Start

```bash
# Install
mkdir -p ~/.claude/skills/groundwork && curl -sL https://raw.githubusercontent.com/SC-Airu/groundwork-skill/main/SKILL.md -o ~/.claude/skills/groundwork/SKILL.md

# Use in Claude Code
/groundwork auto SFX placement for game ad videos in After Effects
```

## What It Does

Give it a pain point. Get back 3 structured research files in ~2 minutes:

```
.omc/groundwork/{slug}/
├── triage.md      # Problem / Who / Why
├── context.md     # Workflow, affected roles, workarounds, adjacent problems, user voices
└── solutions.md   # Solution list, categories, frequency ranking, gaps, key insight
```

### Example Output

```
## Groundwork Complete: sound-auto-placement

### Context
- Game ad motion designers manually place title-specific SFX on every video
- Main workaround: MonkeySauce (marker→SFX automation, but markers are manual)

### Solution Landscape
- 24 solutions across 7 categories
- Key insight: No tool solves the 3-layer problem (detection/mapping/placement)
- Key gap: Many AI SFX tools exist but none support custom SFX libraries
```

## How It Works

```
Step 0: Triage
  Parse input → What / Who / Why
  If unclear, ask max 2 questions
  If results already exist, ask before overwriting

Step 1: Explore (4 agents in parallel)
  A: Context       — workflow, workarounds, user voices
  B: Solutions     — keyword search + curated lists (merged)
  C: Behavior      — what people actually use (community research)
  D: JTBD          — alternative approaches from other domains

  → Orchestrator: gap analysis + dedup + contradiction check

Step 2: Save (Korean by default, configurable)

Step 3: Summary
```

## Features

- **4 parallel research agents** — Context, Solutions, Behavior, JTBD run simultaneously (~2-3 min)
- **Gap analysis** — Finds what no existing tool covers
- **Contradiction detection** — Catches "marketed as X" vs "users say Y" discrepancies
- **Duplicate check** — Won't overwrite existing research without asking
- **Facts only** — No build/kill recommendations. You decide.
- **English search, localized output** — Searches in English for broad coverage, saves in Korean (configurable)

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [oh-my-claudecode](https://github.com/nicholasgriffintn/oh-my-claudecode) (for `document-specialist` agent routing)

## Usage

```bash
# Korean input
/groundwork 게임 사운드 자동 배치 - AI가 영상 분석해 효과음 자동 삽입

# English input
/groundwork auto SFX placement for game ad videos in After Effects

# Detailed input (skips triage questions)
/groundwork Music Prompt Builder - a tool that generates Suno AI BGM prompts
  through simple clicks. Planners select game background, style, mood, tempo,
  instruments and get translated professional music terminology prompts.
```

## Customization

<details>
<summary><strong>Change output language</strong></summary>

Edit the `<Execution_Policy>` section in `SKILL.md`:

```
- All saved files: written in Korean.
```

Change to your preferred language. Research agents always search in English.

</details>

<details>
<summary><strong>Adjust search depth</strong></summary>

Each agent has a `Limit to N web searches max` instruction. Defaults: 10 for most agents, 8 for JTBD.

- Increase for deeper research
- Decrease for speed

</details>

<details>
<summary><strong>Use with downstream skills</strong></summary>

Groundwork output is designed to feed into other skills:

| Skill | How |
|-------|-----|
| `/plan` | Reads `triage.md` for problem context |
| `/discovery` | Skips Steps 0-1 if groundwork exists |
| `CLAUDE.md` | Reference groundwork files for team context |

</details>

## Design Decisions

| Decision | Why |
|----------|-----|
| **4 agents, not 6** | Keyword + Curated merged (70% overlap in testing). Behavior kept separate — finds what people *use* vs what's *marketed*. |
| **No Gap Check agent** | Orchestrator handles dedup + contradiction inline. No quality loss in testing. |
| **English search** | Broader coverage than localized search. Output language is separate. |
| **No depth modes** | Single mode. 4 agents is the sweet spot between speed and coverage. |

## Evolution

| Version | Agents | Time | Change |
|---------|--------|------|--------|
| v1 | 6 | ~5min | Discovery Step 1 fork. Keyword+Curated had 70% overlap. |
| v2 | 2-4 | varies | Added depth modes. Over-engineered. |
| v3 | 3 | ~2min | Removed depth. Merged Behavior into Solutions — lost findings. |
| **v4** | **4** | **~2.3min** | Behavior separated. Keyword+Curated merged. Gap Check inline. |

## Contributing

Issues and PRs welcome. This is a single-file skill (`SKILL.md`) — keep changes focused.

## License

[MIT](LICENSE)
