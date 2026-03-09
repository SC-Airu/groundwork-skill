# Groundwork

A lightweight landscape scan skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Before building anything, Groundwork researches the problem space — who has this problem, how they work around it, and what solutions already exist.

## What It Does

Groundwork takes a pain point and produces 3 structured research files:

```
.omc/groundwork/{slug}/
├── triage.md      # Problem / Who / Why (3-line summary)
├── context.md     # Workflow, affected roles, workarounds, adjacent problems, user voices
└── solutions.md   # Solution list, categories, frequency ranking, gaps, contradictions, key insight
```

**It does NOT:**
- Decide whether to build or kill
- Assess technical feasibility
- Generate roadmaps or recommendations

Groundwork presents facts. You decide.

## How It Works

```
Step 0: Triage
  Parse input → What / Who / Why
  If unclear, ask max 2 questions

Step 1: Explore (4 agents in parallel)
  A: Context         — workflow, workarounds, user voices
  B: Solutions        — keyword search + curated lists (merged)
  C: Behavior         — what people actually use (community research)
  D: JTBD             — alternative approaches from other domains

  → Orchestrator: inline gap analysis + dedup + contradiction check

Step 2: Save files (Korean output)

Step 3: Present summary to user
```

4 research agents run in parallel (~2-3 minutes total). Gap analysis is done inline by the orchestrator — no sequential bottleneck.

## Architecture Decisions

| Decision | Rationale |
|----------|-----------|
| **4 agents, not 6** | Keyword + Curated merged (70% overlap in testing). Behavior kept separate (finds what people *actually use* vs what's *marketed*). |
| **No separate Gap Check agent** | Orchestrator handles dedup + contradiction inline. Tested: no quality loss vs dedicated agent. |
| **Research in English, output in Korean** | English searches have broader coverage. Korean output for the target team. |
| **No depth modes** | Single mode keeps it simple. 4 agents is the sweet spot between speed and accuracy. |

## Usage

```bash
# In Claude Code
/groundwork 게임 사운드 자동 배치 - AI가 영상 분석해 효과음 자동 삽입

# English input works too
/groundwork auto SFX placement for game ad videos in After Effects
```

### Output Example

```
## Groundwork 완료: sound-auto-placement

### 컨텍스트
- 게임 광고 모션 디자이너가 타이틀별 고정 SFX를 매 영상마다 수동 배치
- 주요 워크어라운드: MonkeySauce (마커→SFX 자동 할당, 단 마커는 수동)

### 솔루션 현황
- 24개 솔루션, 7개 카테고리
- 핵심 인사이트: 3-레이어 문제를 해결하는 도구 없음 (감지/매핑/배치)
- 핵심 공백: AI SFX 도구 다수 존재하나 커스텀 라이브러리 미지원이 치명적
```

## Installation

1. Copy `SKILL.md` to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/groundwork
cp SKILL.md ~/.claude/skills/groundwork/
```

2. That's it. Use `/groundwork` in Claude Code.

### Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [oh-my-claudecode](https://github.com/nicholasgriffintn/oh-my-claudecode) (for `document-specialist` agent routing)

## Customization

### Change output language

Edit the `<Execution_Policy>` section in `SKILL.md`:

```
- All saved files: written in Korean.
```

Change to your preferred language. Agent prompts search in English regardless.

### Adjust search limits

Each agent has a `Limit to N web searches max` instruction. Default is 10 for most agents, 8 for JTBD. Increase for deeper research, decrease for speed.

### Use with other skills

Groundwork output is designed to be referenced by downstream skills:

- `/plan` can read `triage.md` for problem context before planning
- `/discovery` can skip Steps 0-1 if groundwork output already exists
- Project `CLAUDE.md` can reference groundwork files for persistent team context

## Evolution

This skill went through 4 iterations based on real testing:

| Version | Agents | Time | Key Change |
|---------|--------|------|------------|
| v1 | 5+1=6 | ~5min | Direct copy of Discovery Step 1. Keyword+Curated 70% overlap. |
| v2 | 2-4+1 | varies | Added depth modes (fast/standard/deep). Over-engineered. |
| v3 | 3 | ~2min | Removed depth modes. Behavior merged into Solutions — lost unique findings. |
| **v4** | **4** | **~2.3min** | Behavior separated back. Keyword+Curated merged. Gap Check inline. |

## License

MIT
