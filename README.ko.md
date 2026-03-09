# Groundwork

[Claude Code](https://docs.anthropic.com/en/docs/claude-code)용 경량 랜드스케이프 스캔 스킬. 뭔가 만들기 전에 문제 공간을 리서치합니다 — 누가 이 문제를 겪는지, 어떻게 우회하는지, 어떤 솔루션이 이미 있는지.

## 하는 일

Groundwork은 페인 포인트를 받아 3개의 구조화된 리서치 파일을 생성합니다:

```
.omc/groundwork/{slug}/
├── triage.md      # 문제 / 대상 / 이유 (3줄 요약)
├── context.md     # 워크플로, 영향 대상, 우회 방법, 인접 문제, 사용자 목소리
└── solutions.md   # 솔루션 목록, 카테고리, 빈도 순위, 공백, 모순점, 핵심 인사이트
```

**하지 않는 것:**
- 빌드/킬 결정
- 기술적 실현 가능성 평가
- 로드맵이나 추천 생성

Groundwork은 팩트를 제시합니다. 결정은 사용자가 합니다.

## 작동 방식

```
Step 0: 트리아지
  입력 파싱 → 문제 / 대상 / 이유
  불명확하면 최대 2개 질문

Step 1: 탐색 (4개 에이전트 병렬)
  A: 컨텍스트      — 워크플로, 우회 방법, 사용자 목소리
  B: 솔루션        — 키워드 검색 + 큐레이션 리스트 (통합)
  C: 행동 패턴     — 사람들이 실제로 쓰는 것 (커뮤니티 리서치)
  D: JTBD          — 다른 도메인의 대안적 접근

  → 오케스트레이터: 인라인 갭 분석 + 중복 제거 + 모순 체크

Step 2: 파일 저장 (한글)

Step 3: 사용자에게 요약 제시
```

4개 리서치 에이전트가 병렬 실행 (총 ~2-3분). 갭 분석은 오케스트레이터가 인라인 수행 — 순차 병목 없음.

## 설계 결정

| 결정 | 근거 |
|------|------|
| **에이전트 4개 (6개 아님)** | Keyword + Curated 통합 (테스트 시 70% 중복). Behavior는 분리 유지 (마케팅 vs 실사용 구분 필요). |
| **Gap Check 별도 에이전트 없음** | 오케스트레이터가 중복 제거 + 모순 분석 인라인 수행. 테스트 결과: 전용 에이전트 대비 품질 손실 없음. |
| **리서치는 영어, 결과물은 한글** | 영어 검색이 더 넓은 커버리지. 한글 결과물은 팀 가독성. |
| **Depth 모드 없음** | 단일 모드로 단순화. 4개 에이전트가 속도와 정확도의 최적점. |

## 사용법

```bash
# Claude Code에서
/groundwork 게임 사운드 자동 배치 - AI가 영상 분석해 효과음 자동 삽입

# 영어 입력도 가능
/groundwork auto SFX placement for game ad videos in After Effects
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

## 설치

1. `SKILL.md`를 Claude Code 스킬 디렉토리에 복사:

```bash
mkdir -p ~/.claude/skills/groundwork
cp SKILL.md ~/.claude/skills/groundwork/
```

2. 끝. Claude Code에서 `/groundwork` 사용.

### 요구사항

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [oh-my-claudecode](https://github.com/nicholasgriffintn/oh-my-claudecode) (`document-specialist` 에이전트 라우팅용)

## 커스터마이즈

### 출력 언어 변경

`SKILL.md`의 `<Execution_Policy>` 섹션 수정:

```
- All saved files: written in Korean.
```

원하는 언어로 변경. 에이전트 검색은 언어와 무관하게 영어로 수행.

### 검색 횟수 조절

각 에이전트에 `Limit to N web searches max` 지시가 있음. 기본값: 대부분 10회, JTBD 8회. 깊은 리서치는 증가, 속도 우선은 감소.

### 다른 스킬과 연계

Groundwork 결과물은 후속 스킬에서 참조하도록 설계:

- `/plan` — `triage.md` 읽어서 문제 컨텍스트 파악 후 계획 수립
- `/discovery` — groundwork 결과 있으면 Step 0-1 스킵 가능
- 프로젝트 `CLAUDE.md` — groundwork 파일 경로 기록해서 팀 컨텍스트 공유

## 진화 과정

실제 테스트 기반 4회 이터레이션:

| 버전 | 에이전트 | 시간 | 주요 변경 |
|------|---------|------|----------|
| v1 | 5+1=6 | ~5분 | Discovery Step 1 복사. Keyword+Curated 70% 중복. |
| v2 | 2-4+1 | 가변 | Depth 모드 추가 (fast/standard/deep). 오버엔지니어링. |
| v3 | 3 | ~2분 | Depth 모드 제거. Behavior를 Solutions에 통합 — 고유 발견 손실. |
| **v4** | **4** | **~2.3분** | Behavior 재분리. Keyword+Curated 통합. Gap Check 인라인. |

## 라이선스

MIT
