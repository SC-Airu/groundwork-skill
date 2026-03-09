---
name: groundwork
description: Lightweight landscape scan — context understanding + solution survey for any pain point or idea. Produces reusable project research in Korean.
argument-hint: "<pain point or idea, e.g. '게임 사운드 자동 배치'>"
---

<Purpose>
Quick research scan before building. Takes a pain point and produces 3 files:
1. **triage.md** — 문제/대상/이유 (3줄 요약)
2. **context.md** — 워크플로, 영향 대상, 우회 방법, 인접 문제, 사용자 목소리
3. **solutions.md** — 솔루션 목록, 카테고리, 빈도 순위, 핵심 공백/인사이트

Output is written in **Korean** and persists across sessions at `.omc/groundwork/{slug}/`.
Research agents search in **English** for broader coverage.

Does NOT decide whether to build. Does NOT assess technical feasibility. Those are separate decisions.
</Purpose>

<Use_When>
- Before starting any new tool or feature
- "What's out there for this?" or "Is there something like..."
- Need to understand the landscape before planning
- Keywords: "groundwork", "scan", "landscape", "what exists", "research this"
</Use_When>

<Do_Not_Use_When>
- You need a full discovery pipeline with tech assessment and strategy
- You already know what to build and need a plan (use /plan)
- It's a bug fix or small change (just do it)
</Do_Not_Use_When>

<Execution_Policy>
- NEVER write code. This skill produces documents only.
- Save all output to `.omc/groundwork/{slug}/` before proceeding.
- All research agent prompts: search in English for broader results.
- All saved files: written in Korean.
- Launch all 4 agents in parallel — maximize speed.
- Gap analysis done by orchestrator inline after agents complete (no separate agent).
- Present a brief summary to the user at the end. Do NOT make build/kill recommendations.
- If input is sufficiently detailed, skip triage questions.
- **Agent fallback**: Use `subagent_type="oh-my-claudecode:document-specialist"` when oh-my-claudecode is available. If unavailable (agent type not recognized or errors), fall back to `subagent_type="general-purpose"` with the same prompt. The general-purpose agent has web search access and will produce comparable results.
</Execution_Policy>

<Steps>

## Step 0: Triage

**Duplicate check**: Before parsing, check if `.omc/groundwork/{slug}/triage.md` already exists. If it does, ask the user: "이미 groundwork 결과가 있습니다. 덮어쓸까요, 아니면 기존 결과를 사용할까요?" If the user says keep existing, skip to Step 3 (present existing summary). If the user says overwrite, proceed normally.

Parse `{{ARGUMENTS}}` and check three dimensions:
- **What**: What is the pain point or idea?
- **Who**: Who experiences this?
- **Why**: Why is it a problem?

If any dimension is unclear, ask the user using `AskUserQuestion` — one focused question at a time, maximum 2 questions. If all are clear, skip to Step 1.

Save to `.omc/groundwork/{slug}/triage.md`:
```markdown
# 트리아지
- 문제: {pain point}
- 대상: {who experiences it}
- 이유: {why it matters}
```

## Step 1: Explore (4 Agents in Parallel)

Launch ALL 4 agents simultaneously:

### A. Context Understanding

```
Task(subagent_type="oh-my-claudecode:document-specialist", model="sonnet", prompt="
[GROUNDWORK:CONTEXT]

Research the following pain point. Search in ENGLISH for broader coverage.
Return your analysis in ENGLISH (it will be translated when saved).

Pain point: {what}
Who experiences it: {who}
Why it matters: {why}

Investigate:
1. In what situations/workflows does this problem occur?
2. Who typically deals with this? (roles, skill levels)
3. How are people currently working around it?
4. What adjacent problems are connected to this?

Search communities (Reddit, HN, forums, Stack Overflow) for real user voices.
Include direct quotes where possible. Limit to 10 web searches max.

Format your response as markdown with these exact sections:
## Usage Context
## Who Is Affected
## Current Workarounds
## Adjacent Problems
## User Voices (with source links)
")
```

### B. Solutions Survey (keyword + curated merged)

```
Task(subagent_type="oh-my-claudecode:document-specialist", model="sonnet", prompt="
[GROUNDWORK:SOLUTIONS]

Find ALL existing solutions for this problem. Search in ENGLISH.
Combine direct keyword search AND curated lists.

Problem: {what}
Who: {who}

Search strategy:
1. Direct keyword search — tools, plugins, libraries, SaaS products, GitHub projects
2. Curated lists — 'best X alternatives' articles, Product Hunt, 'awesome-*' GitHub lists, G2/Capterra/AlternativeTo

Limit to 10 web searches max.

For each solution found, note:
- Name and URL
- How it solves the problem
- Pricing/availability
- Known limitations (if found)

Also note:
- Which solutions appear most frequently across sources

Format as markdown with:
## Solutions (table: name | URL | approach | pricing | limitations)
## Frequency Ranking (most mentioned solutions)
## Curated Lists Found (with links)
")
```

### C. User Behavior Search

```
Task(subagent_type="oh-my-claudecode:document-specialist", model="sonnet", prompt="
[GROUNDWORK:BEHAVIOR]

Find how people ACTUALLY solve this problem in practice. Search in ENGLISH.

Problem: {what}
Who: {who}

Search Reddit, forums, Stack Overflow, HN for:
- 'how do you handle...' questions
- 'what do you use for...' discussions
- Workaround threads and tips
- Frustration posts about existing tools

Focus on what real users actually USE, not what's marketed to them.
Include links to discussions. Limit to 10 web searches max.

Format as markdown with:
## What People Actually Use
## Common Workarounds
## Pain Points with Current Solutions
## Sources (with links)
")
```

### D. JTBD Search

```
Task(subagent_type="oh-my-claudecode:document-specialist", model="sonnet", prompt="
[GROUNDWORK:JTBD]

Find alternative solutions from a Jobs-to-be-Done perspective. Search in ENGLISH.

The job: {who} needs to {what_as_job}

Search for:
- Completely different approaches to the same job
- Solutions from other industries/domains
- Non-obvious competitors (different category, same job)

Example: If the job is 'not be bored during commute', competitors include
podcasts, audiobooks, mobile games — not just music apps.

Limit to 8 web searches max.

Format as markdown with:
## The Job
## Direct Solutions
## Indirect/Alternative Solutions
## Cross-Industry Approaches
")
```

### Gap Analysis (orchestrator inline, after A/B/C/D complete)

After all 4 agents return, the orchestrator:
1. Deduplicates solutions from B, C, D
2. Cross-references with workarounds from A
3. Identifies structural gaps (what no existing tool covers)
4. Notes contradictions between reports (e.g., "marketed as X" vs "users say Y")
5. Writes the merged `solutions.md` in Korean

## Step 2: Save Files (Korean)

### context.md format:
```markdown
# 컨텍스트: {project name}

## 워크플로 현황
{when/where the problem occurs}

## 영향 대상
| 역할 | 책임 | 기술 수준 |
|------|------|----------|
{who is affected}

## 현재 우회 방법
{numbered list with tool names and limitations}

## 인접 문제
{lettered list of adjacent problems}

## 사용자 목소리
{user quotes with source links}
```

### solutions.md format:
```markdown
# 솔루션 현황: {project name}

## 솔루션 목록
| 이름 | 접근 방식 | 강점 | 약점 | 비고 |
|------|----------|------|------|------|
{deduplicated master list from B + C + D}

## 카테고리 분류
{solutions grouped by category}

## 실제 사용 현황
{what people actually use from C — separate from what's marketed}

## 빈도 순위
{most frequently mentioned solutions}

## 핵심 공백
{what's missing — the structural gap no tool covers}

## 모순점
{contradictions between reports — e.g., marketed capability vs actual user experience}

## 핵심 인사이트
{one paragraph: what the solution landscape tells us}
```

## Step 3: Summary

Present a brief Korean summary to the user:

```
## Groundwork 완료: {slug}

### 컨텍스트
- {1-2 sentence summary}
- 주요 워크어라운드: {most common workaround}

### 솔루션 현황
- {N}개 솔루션, {M}개 카테고리
- 핵심 인사이트: {one sentence}
- 핵심 공백: {what's missing}

### 파일
- `.omc/groundwork/{slug}/triage.md`
- `.omc/groundwork/{slug}/context.md`
- `.omc/groundwork/{slug}/solutions.md`
```

Do NOT recommend build/kill/adopt. Just present the facts.

</Steps>

<Tool_Usage>
- `AskUserQuestion` — for triage questions only (max 2)
- `Agent(subagent_type="oh-my-claudecode:document-specialist")` x 4 — web research (English). Falls back to `Agent(subagent_type="general-purpose")` if OMC is unavailable.
- `Write` — save outputs to `.omc/groundwork/{slug}/` (Korean)
</Tool_Usage>

<Examples>

<Good>
All 4 agents launched in parallel:
```
User: "groundwork 게임 사운드 자동 배치"

→ Launch simultaneously:
  A: Context (워크플로 + 우회 방법 + 사용자 목소리)
  B: Solutions (키워드 + 큐레이션 통합)
  C: Behavior (실제 사용 패턴 — 커뮤니티 기반)
  D: JTBD (다른 관점의 솔루션)

→ All 4 complete
→ Orchestrator: dedup + gap analysis + contradiction check
→ Save triage.md + context.md + solutions.md (all Korean)
→ Present summary (Korean, no recommendation)
```
</Good>

<Good>
Vague input — ask one question then proceed:
```
User: "groundwork 빌드가 느려요"

Triage:
- 문제: Build is slow ✓
- 대상: ? → Ask: "누가 겪는 문제인가요? 본인만? 팀 전체?"
- 이유: Slow build = productivity loss ✓

After 1 question, proceed to Step 1.
```
</Good>

<Bad>
Making a recommendation:
```
"리서치 결과 Turborepo를 도입하는 것을 추천합니다."
```
Why bad: Groundwork presents facts, not recommendations.
</Bad>

<Bad>
Output in English:
```
"## Key Insight: The solution landscape shows..."
```
Why bad: All saved files and summaries must be in Korean.
</Bad>

<Bad>
Running agents sequentially:
```
Context done → then Solutions → then Behavior → then JTBD
```
Why bad: All 4 are independent. Must run in parallel.
</Bad>

</Examples>

<Final_Checklist>
- [ ] Triage complete (문제/대상/이유 clear)
- [ ] 4 agents launched in parallel (Context + Solutions + Behavior + JTBD)
- [ ] Research conducted in English
- [ ] Gap analysis + contradiction check done inline by orchestrator
- [ ] All files saved in Korean
- [ ] `triage.md` saved
- [ ] `context.md` saved
- [ ] `solutions.md` saved (deduplicated, with gap/contradiction analysis)
- [ ] Summary presented to user (Korean, facts only)
- [ ] All files in `.omc/groundwork/{slug}/`
</Final_Checklist>

Task: {{ARGUMENTS}}
