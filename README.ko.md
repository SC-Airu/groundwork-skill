<p align="center">
  <h1 align="center">Groundwork</h1>
  <p align="center">
    만들기 전에 문제 공간을 리서치하세요.<br>
    <a href="https://docs.anthropic.com/en/docs/claude-code">Claude Code</a>용 랜드스케이프 스캔 스킬.
  </p>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href="SKILL.md"><img src="https://img.shields.io/badge/skill-v4-green.svg" alt="Skill v4"></a>
  <a href="https://docs.anthropic.com/en/docs/claude-code"><img src="https://img.shields.io/badge/Claude_Code-skill-blueviolet.svg" alt="Claude Code Skill"></a>
</p>

<p align="center">
  <a href="README.md">English</a>
</p>

---

**이미 있는 걸 또 만들고 있진 않나요?** Groundwork은 4개의 병렬 리서치 에이전트로 문제 공간을 스캔합니다 — 누가 이 문제를 겪는지, 어떻게 우회하는지, 어떤 솔루션이 있는지 — 코드 한 줄 쓰기 전에 충분한 정보를 갖추세요.

## 빠른 시작

```bash
# 설치
mkdir -p ~/.claude/skills/groundwork && curl -sL https://raw.githubusercontent.com/SC-Airu/groundwork-skill/main/SKILL.md -o ~/.claude/skills/groundwork/SKILL.md

# Claude Code에서 사용
/groundwork 게임 사운드 자동 배치 - AI가 영상 분석해 효과음 자동 삽입
```

## 하는 일

페인 포인트를 입력하면 ~2분 만에 3개의 구조화된 리서치 파일을 생성합니다:

```
.omc/groundwork/{slug}/
├── triage.md      # 문제 / 대상 / 이유
├── context.md     # 워크플로, 영향 대상, 우회 방법, 인접 문제, 사용자 목소리
└── solutions.md   # 솔루션 목록, 카테고리, 빈도 순위, 공백, 핵심 인사이트
```

### 출력 예시

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

## 작동 방식

```
Step 0: 트리아지
  입력 파싱 → 문제 / 대상 / 이유
  불명확하면 최대 2개 질문
  기존 결과 있으면 덮어쓸지 확인

Step 1: 탐색 (4개 에이전트 병렬)
  A: 컨텍스트    — 워크플로, 우회 방법, 사용자 목소리
  B: 솔루션      — 키워드 검색 + 큐레이션 리스트 (통합)
  C: 행동 패턴   — 사람들이 실제로 쓰는 것 (커뮤니티 리서치)
  D: JTBD        — 다른 도메인의 대안적 접근

  → 오케스트레이터: 갭 분석 + 중복 제거 + 모순 체크

Step 2: 저장 (기본 한글, 설정 가능)

Step 3: 요약
```

## 주요 기능

- **4개 병렬 리서치 에이전트** — Context, Solutions, Behavior, JTBD 동시 실행 (~2-3분)
- **갭 분석** — 기존 도구가 커버하지 못하는 영역 식별
- **모순 감지** — "마케팅 주장 vs 실사용자 경험" 불일치 포착
- **중복 체크** — 기존 리서치 결과가 있으면 덮어쓰기 전 확인
- **팩트만 제시** — 빌드/킬 추천 없음. 결정은 사용자 몫.
- **영어 검색, 한글 출력** — 영어로 검색해 넓은 커버리지, 한글로 저장 (변경 가능)

## 요구사항

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [oh-my-claudecode](https://github.com/nicholasgriffintn/oh-my-claudecode) (`document-specialist` 에이전트 라우팅용)

## 사용법

```bash
# 한글 입력
/groundwork 게임 사운드 자동 배치 - AI가 영상 분석해 효과음 자동 삽입

# 영어 입력
/groundwork auto SFX placement for game ad videos in After Effects

# 상세 입력 (트리아지 질문 스킵)
/groundwork Music Prompt Builder - 클릭 몇 번으로 Suno AI용 BGM 프롬프트 생성.
  기획자가 게임 배경, 음악 스타일, 분위기, 템포, 악기를 선택하면
  전문 음악 용어로 번역된 프롬프트를 자동 생성.
```

## 커스터마이즈

<details>
<summary><strong>출력 언어 변경</strong></summary>

`SKILL.md`의 `<Execution_Policy>` 섹션 수정:

```
- All saved files: written in Korean.
```

원하는 언어로 변경. 에이전트 검색은 항상 영어로 수행됩니다.

</details>

<details>
<summary><strong>검색 깊이 조절</strong></summary>

각 에이전트에 `Limit to N web searches max` 지시가 있음. 기본값: 대부분 10회, JTBD 8회.

- 깊은 리서치 → 증가
- 속도 우선 → 감소

</details>

<details>
<summary><strong>다른 스킬과 연계</strong></summary>

Groundwork 결과물은 후속 스킬에서 참조하도록 설계되었습니다:

| 스킬 | 연계 방식 |
|------|----------|
| `/plan` | `triage.md` 읽어서 문제 컨텍스트 파악 |
| `/discovery` | groundwork 결과 있으면 Step 0-1 스킵 |
| `CLAUDE.md` | groundwork 파일 경로 기록해서 팀 컨텍스트 공유 |

</details>

## 설계 결정

| 결정 | 이유 |
|------|------|
| **에이전트 4개 (6개 아님)** | Keyword + Curated 통합 (테스트 시 70% 중복). Behavior 분리 유지 — *마케팅* vs *실사용* 구분 필요. |
| **Gap Check 별도 에이전트 없음** | 오케스트레이터가 인라인 수행. 테스트 결과 품질 손실 없음. |
| **영어 검색** | 로컬 언어보다 넓은 커버리지. 출력 언어는 별도 설정. |
| **Depth 모드 없음** | 단일 모드. 4개 에이전트가 속도와 커버리지의 최적점. |

## 진화 과정

| 버전 | 에이전트 | 시간 | 변경 |
|------|---------|------|------|
| v1 | 6 | ~5분 | Discovery Step 1 포크. Keyword+Curated 70% 중복. |
| v2 | 2-4 | 가변 | Depth 모드 추가. 오버엔지니어링. |
| v3 | 3 | ~2분 | Depth 제거. Behavior를 Solutions에 통합 — 고유 발견 손실. |
| **v4** | **4** | **~2.3분** | Behavior 재분리. Keyword+Curated 통합. Gap Check 인라인. |

## 기여

이슈와 PR 환영합니다. 단일 파일 스킬(`SKILL.md`)이므로 변경은 집중적으로.

## 라이선스

[MIT](LICENSE)
