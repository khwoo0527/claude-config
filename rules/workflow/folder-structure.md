---
paths:
  - ".claude/**"
---

# .claude 폴더 구조 정책 (단일 진실 원천)

> `.claude/` 안에 새 문서/폴더를 추가할 때 **"어디에 두지?"** 를 결정하는 기준.
>
> README 의 폴더 트리는 "현재 무엇이 있는가" 의 스냅샷. 이 문서는 "왜 거기에 두는가 + 새로 추가할 때 어디에 두는가" 의 결정 기준이다.

---

## 1. 폴더 매트릭스 (정체성 한눈에)

| 폴더 | 정체성 | 자동 로딩 | 새 프로젝트 시 | 누적 자산? |
|---|---|:-:|---|:-:|
| `/CLAUDE.md` (프로젝트 루트) | 이 프로젝트의 정의 | ✅ Claude Code 기본 | 새로 작성 | ❌ |
| `memory/` | 사용자 작업 스타일 + 핵심 원칙 | ✅ `/InitLoad` 강제 | 그대로 (범용) | ✅ |
| `rules/tech/` | 기술별 노하우 | ❌ 작업 시점 수동 | 그대로 (범용) | ✅ |
| `rules/workflow/` | 가이드/정책 (단일 진실 원천) | ❌ 작업 시점 수동 | 그대로 (범용) | ✅ |
| `agents/` | 에이전트 정의 | (호출 시) | 그대로 (범용) | ✅ |
| `commands/` | 슬래시 커맨드 | (호출 시) | 그대로 (범용) | ✅ |
| `agent-memory/{name}/` | 에이전트별 작업 컨텍스트 캐시 | ❌ 에이전트 진입 시 자기 캐시 Read | **리셋** | ❌ |
| `templates/` | 자동 복사용 빈 골격 + 카탈로그 | ❌ 수동 참조 | 그대로 (범용) | ✅ |
| `settings.json` | 권한/훅 (공유) | ✅ Claude Code | 검토 후 유지 | (범용) |
| `settings.local.json` | 로컬 권한/토큰 | ✅ Claude Code | **확인/리셋** | ❌ |

### 핵심 분류

- **누적 자산** = 다른 프로젝트에서도 그대로 통하는 가치
- **프로젝트 종속** = `agent-memory/`, `settings.local.json`, `/CLAUDE.md`

---

## 2. 새 문서 추가 결정 트리

```
새 문서를 추가하려고 한다 — 어디에 두지?

1️⃣ 다른 프로젝트에도 그대로 재사용 가능한가?
  ❌ NO (이 프로젝트만의 정보)
    → /CLAUDE.md (프로젝트 정의) 또는 agent-memory/{name}/ (작업 캐시)

  ✅ YES → 2️⃣

2️⃣ 어떤 종류인가?

  📌 사용자 작업 스타일 / 선호 / 원칙
    → memory/feedback_*.md 또는 user_*.md
    → MEMORY.md 인덱스 등록 필수

  📌 기술 스택 노하우 (안티패턴, 함정, 베스트 프랙티스)
    → rules/tech/{tech}.md
    → rules-guide.md 기준으로 작성

  📌 가이드 / 정책 (어떻게 작성하는가, 왜 그렇게 하는가)
    → rules/workflow/{name}.md
    → 단일 진실 원천 — 정책 변경 시 한 곳만 수정하면 전체 반영

  📌 에이전트 정의
    → agents/{name}.md
    → agent-guide.md 기준으로 작성

  📌 슬래시 커맨드 정의
    → commands/{name}.md

  📌 자동 복사용 빈 골격 (사용자가 수정해서 쓰는 시작점)
    → templates/{NAME}-TEMPLATE.md (CLAUDE-TEMPLATE.md 처럼)
    → 또는 templates/{folder}/ (designs/ 처럼 카탈로그)

3️⃣ 진입 경로 확보 (가이드/정책 문서 한정 — 누락하면 페이퍼워크)

  ⚠️ 핵심: `frontmatter paths` 는 자동 로딩 메커니즘이 아니다 (단순 메타데이터).
     Claude Code 가 자동 로딩하는 건 `/CLAUDE.md` + `~/.claude/projects/*/memory/MEMORY.md` 만.
     그 외는 모두 누군가 명시적으로 Read 해야 함.

  📌 신설 문서가 자동 진입 경로 있는가?
    ✅ session-init.md 매트릭스의 작업 유형에 매칭됨 → OK
       (예: 코드 구현 → sprint-workflow.md, 에이전트 작성 → agent-guide.md)
    ✅ 다른 가이드/에이전트 본문에서 명시적으로 참조됨 → OK
       (예: agent-memory.md 는 sprint-planner 등에서 참조)
    ❌ 어디서도 명시적으로 Read 안 됨 → 진입 경로 추가 필요!

  📌 진입 경로 추가 위치 (책임 분리)
    - 가이드/정책 (rules/workflow/*) → `session-init.md` 매트릭스에 작업 유형 + 로딩 대상 등록
    - 기술 노하우 (rules/tech/*) → 이미 매트릭스에 "코드 구현/리뷰/rules 작성" 행 있음, 별도 등록 불필요
    - 에이전트 (agents/*) → Task tool 호출 메커니즘 별도, CLAUDE.md "에이전트 진입 절차" 가 강제
```

---

## 3. rules/workflow vs templates 구분 (자주 헷갈림)

| 구분 | rules/workflow/ | templates/ |
|---|---|---|
| 본질 | **가이드 + 정책** (어떻게 작성하는가) | **빈 골격 + 카탈로그** (자동 복사용 시작점) |
| 사용 방식 | 작업 시점 Read 해서 따름 | 새 프로젝트 시작 시 복사 → 수정 |
| 예시 | `rules-guide.md` (rules/tech 작성법), `agent-guide.md` (에이전트 작성법), `prd-guide.md` (PRD 작성법) | `CLAUDE-TEMPLATE.md` (프로젝트 루트 CLAUDE.md 의 빈 골격), `designs/{brand}.md` (DESIGN.md 레퍼런스) |
| 변경 빈도 | 정책 진화에 따라 가끔 | 거의 안 바뀜 (골격이라서) |

### 판별 기준 (한 줄)

> **읽고 따르는가 (rules/workflow) vs 복사하고 수정하는가 (templates)**

### 실제 적용 사례

- **prd-guide.md** — "PRD 를 어떻게 작성할까" 의 가이드 → `rules/workflow/`
- **CLAUDE-TEMPLATE.md** — 프로젝트 루트 CLAUDE.md 의 빈 골격 → `templates/`
- **agent-guide.md** — "새 에이전트를 어떻게 작성할까" 의 가이드 → `rules/workflow/`
- **designs/{brand}.md** — DESIGN.md 작성 시 참고할 브랜드 카탈로그 → `templates/`

---

## 4. memory vs agent-memory 구분 (자주 헷갈림)

| 구분 | memory/ | agent-memory/{name}/ |
|---|---|---|
| 성격 | 사용자 작업 스타일 / 선호 / 원칙 | 에이전트별 작업 컨텍스트 캐시 |
| 범위 | 프로젝트 무관 (범용) | 프로젝트 종속 |
| 누적? | ✅ 누적 자산 | ❌ 새 프로젝트 시 리셋 |
| 자동 로딩 | ✅ `/InitLoad` 강제 | ❌ 에이전트 진입 시 자기 캐시만 |
| 인덱스 | MEMORY.md 필수 | MEMORY.md 단일 파일 |
| 다른 프로젝트로 복사? | ✅ 그대로 | ❌ 절대 X |

### 판별 기준 (한 줄)

> **새 프로젝트에서도 통하는가 (memory) vs 이 프로젝트만의 작업 캐시인가 (agent-memory)**

자세한 분기 룰: [`agent-memory.md`](agent-memory.md), [`tech-knowledge.md`](tech-knowledge.md) 참조.

---

## 5. 폴더별 추가 시 체크리스트

### memory/ 에 추가
- [ ] `feedback_*.md` 또는 `user_*.md` 명명 규칙 따름
- [ ] frontmatter `name`, `description`, `type` (user/feedback/project/reference) 명시
- [ ] `MEMORY.md` 인덱스에 한 줄 추가
- [ ] 다른 프로젝트에서도 그대로 통하는지 확인 (이 프로젝트 종속이면 X)

### rules/tech/ 에 추가
- [ ] [`tech-knowledge.md`](tech-knowledge.md) 의 누적 정책 따름
- [ ] [`rules-guide.md`](rules-guide.md) 의 4단계 품질 체크리스트 적용
- [ ] frontmatter `paths` 매칭 (자동 로딩 영역 정의)

### rules/workflow/ 에 추가
- [ ] 단일 진실 원천 위반 X (다른 곳에 같은 정책 있는지 확인)
- [ ] frontmatter `paths` 매칭 (메타데이터용 — 자동 로딩 X)
- [ ] 다른 문서에서 이 가이드 참조 가능하도록 명시적 제목
- [ ] **🔑 진입 경로 확보**: `session-init.md` 매트릭스에 작업 유형 + 로딩 대상 등록 (안 하면 페이퍼워크)
- [ ] 누락 시 발생 문제를 `session-init.md` "로딩하지 않았을 때 발생하는 문제" 표에 추가

### agents/ 에 추가
- [ ] [`agent-guide.md`](agent-guide.md) 의 표준 구조 따름
- [ ] description 에 한국어 자연 호출 `<example>` 2~3개
- [ ] 본문에 진입 절차 명시 X (CLAUDE.md 자동 로딩 강제)
- [ ] 자기 캐시 필요 여부 결정 → `agent-memory/{name}/MEMORY.md`

### commands/ 에 추가
- [ ] 슬래시 호출 형식 명확 (인수 / 옵션 명시)
- [ ] 에이전트 호출이면 위임 명시 (직접 구현 X)

### templates/ 에 추가
- [ ] **자동 복사용 빈 골격인가** 확인 (가이드면 rules/workflow/ 가 맞음)
- [ ] 또는 카탈로그/레퍼런스 모음인가 확인

### agent-memory/{name}/ 에 추가
- [ ] [`agent-memory.md`](agent-memory.md) 의 표준 형식 따름
- [ ] `MEMORY.md` 단일 파일 (다중 파일 모델 안티패턴)
- [ ] 다른 프로젝트에 통하는 노하우면 → `rules/tech/` 또는 `rules/workflow/` 로 이전

---

## 6. 안티패턴

### 폴더 정체성 위반
- ❌ `rules/workflow/` 에 빈 골격 (예: `*-TEMPLATE.md`) — `templates/` 가 맞음
- ❌ `templates/` 에 가이드/정책 — `rules/workflow/` 가 맞음
- ❌ `agent-memory/` 에 다른 프로젝트에 통할 일반 노하우 — `rules/tech/` 로
- ❌ `memory/` 에 이 프로젝트 종속 정보 — `/CLAUDE.md` 또는 `agent-memory/` 로

### 단일 진실 원천 위반
- ❌ 같은 정책이 여러 위치에 박힘 (예: 외부 서비스 위치 정의를 본문 여러 곳에)
- ❌ 가이드 본문이 다른 가이드와 중복 (참조로 대체)
- ❌ `agents/` 본문에 진입 절차 명시 (CLAUDE.md 자동 로딩과 중복)

### 진입 경로 / 메타데이터 위반
- ❌ `MEMORY.md` 인덱스 (글롭) 결과와 실제 파일 수 불일치 → `/InitLoad` 누락
- ❌ 폐지된 frontmatter 키 사용 (예: `memory: project`)
- ❌ **frontmatter `paths` 만 명시하고 진입 경로 등록 안 함 → 페이퍼워크 문서**
  - `paths` 는 자동 로딩이 아님 (단순 메타데이터)
  - rules/workflow/* 신설 시 `session-init.md` 매트릭스 등록 필수

### 다중 파일 모델 (옛 정책)
- ❌ `agent-memory/{name}/2026-04-25-task.md` 같은 날짜별 분리
- ✅ `agent-memory/{name}/MEMORY.md` 단일 파일 + 섹션별 누적

---

## 7. 변경 이력 (정책 진화)

### 2026-04-29: rules vs templates 통일
- **이전**: `templates/RULES-TEMPLATE.md` (rules 작성 가이드를 templates 에 둠 — 정체성 모호)
- **이후**: `rules/workflow/rules-guide.md` (가이드는 rules/workflow/ 가 맞음)
- **이유**: templates 는 "자동 복사용 빈 골격", rules/workflow/ 는 "가이드/정책" — 본질 기반 분리
- **연쇄**: 13개 파일 참조 일괄 갱신 + `agent-guide.md`, `folder-structure.md` 신설

### 2026-04-30: 진입 경로 자기 보호 메커니즘 추가
- **이전**: 신설 가이드 문서가 frontmatter `paths` 만 명시하고 진입 경로 없음 (페이퍼워크 리스크)
- **이후**: 결정 트리에 "3️⃣ 진입 경로 확보" 단계 추가 + rules/workflow/ 체크리스트에 매트릭스 등록 강제
- **이유**: `frontmatter paths` 는 자동 로딩 아님 (단순 메타데이터). 신설 4개 가이드(`rules-guide`, `tech-knowledge`, `agent-guide`, `folder-structure`) 가 진입 경로 없이 떠 있는 문제 발견 → `session-init.md` 매트릭스 3행 추가로 해결 + 같은 실수 재발 방지를 위해 본 가이드에 자기 보호 룰 박음
- **연쇄**: `session-init.md` 매트릭스 3행 + 누락 문제 표 3행 추가

### 2026-04-30: 자율 검토 사이클 iteration 1
- **자기 발견**: § 6 안티패턴 첫 항목 "frontmatter paths 누락 → 자동 로딩 영역 모호" 자기 모순 (paths 는 자동 로딩 아님이 본 정책) → 옛 인식 잔재 항목 제거
- **자기 발견**: tech-knowledge.md § 6 표의 `clubs` 테이블 — 프로젝트 종속 용어. rules-guide § 2-2 "프로젝트 종속 검증" 룰 위반 → `teams` 로 정정
- **자기 발견**: session-init.md 매트릭스 "검토 작업" 행 누락 → 추가 (`/ReviewClaudeConfig` 자연어 호출 매칭)
- **Sub-agent 발견**: README mermaid 다이어그램에 0단계 (`project-init`) 누락 vs 표는 0~7 명시 → mermaid 에 Z 노드 추가
- **Sub-agent 발견**: notion-writer.md frontmatter `color` 누락 (다른 8개는 모두 명시) → `color: purple` 추가
- **Sub-agent 발견 → 룰화**: README mermaid ↔ 표 정합 검증을 `/UpdateReadme` 2단계에 항목 추가 → 다음 갱신 시 자동 점검

### 2026-05-06: 자율 검토 사이클 iteration 2 + 권한 확장
- **인프라 변경**: settings.local.json 에 read-only Bash 권한 38개 추가 (`Bash(echo *)`, `Bash(for *)`, `Bash(grep *)` 등) — 자율 루프 진행 위해. git commit/push 는 별도 룰로 절대 X 유지
- **Sub-agent 발견**: 9개 에이전트 중 3개 (`hotfix-close`, `deploy-prod`, `project-init`) 가 "발견된 정보 분기 기록" 섹션 누락 → 박스 1줄 형태로 추가 (자기 캐시 없음 패턴)
- **Sub-agent 발견**: `notion-writer` 의 § 1 "디자인 철학" 이 핵심 원칙 도메인 변형 → agent-guide § 2 표준에 "도메인 변형 명칭 허용" 명시 (필수 섹션 표 + § 7 패턴별 변형 박스)
- **본질 통합**: agent-guide § 2 를 "필수 섹션" + "선택 섹션" + "본문 골격" 3 영역으로 재구성 — 도메인 변형 허용 명시로 일관성 + 변형 모두 수용

### 2026-05-06: 자율 루프 사후 검증 + 9건 통합 정정
- **자기 발견**: `prd-to-roadmap.md` 에 `## 에러 처리` 섹션 누락 (agent-guide § 2 필수 위반) → 5건 에러 케이스 추가
- **자기 발견**: `settings.local.json` 셸 키워드 5개 (`do/done/then/fi/if`) 불필요 (페이퍼워크) → 제거
- **Sub-agent 발견 (자기 매몰)**: 본 변경 이력의 "9개 중 4개" 카운트 모순 (실제 3개 나열) → "3개" 로 정정. 같은 안티패턴이 1축 점검 매몰의 사례 → `ReviewClaudeConfig § 1축` 점검 대상에 "변경 이력 본문 내 카운트 정합" 항목 추가 (자기 보호 룰화)
- **Sub-agent 발견**: `session-init.md:43` "6 사전 컨텍스트" → "6개 사전 컨텍스트" (한국어 정정)
- **Sub-agent 발견 (위험)**: 권한 38개 중 11개 (`xargs/awk/find/test/true/false/do/done/then/fi/if`) 가 destructive 합성 우회 가능 (예: `find -delete`, `awk system()`, `xargs rm`, `if X; then rm Y; fi`) → 11개 제거. 27개 → 26개 read-only 한정
- **Sub-agent 발견**: `project-init.md` 분기 박스 "사용자 작업 스타일은 memory/feedback_*.md 추가 검토" 주체 모호 → "사용자 스타일 발견 시만 별도 컨펌 후 제안" 으로 명확화 (부트스트랩 패턴 빈도 낮음 명시)
- **Sub-agent 발견**: `notion-writer.md` `color: purple` 의미 정책 부재 → agent-guide § 1 frontmatter 표준에 "color 는 단순 시각 구분 — 의미 정책 X" 명시 박스 추가
- **자기 발견**: `folder-structure § 6` "자동 로딩 메커니즘 무시" 섹션 제목이 paths 정책과 부분 모순 → "진입 경로 / 메타데이터 위반" 으로 정정
- **메타 학습**: 자기 단독 검토 시 발견 5건, sub-agent 호출 시 추가 5건 — 매번 자기 매몰 안티패턴 입증. ReviewClaudeConfig § 핵심 원칙 1번 (병행 필수) 의 가치 재확인.

---

## 8. 관련 문서

- [`agent-guide.md`](agent-guide.md) — 새 에이전트 작성 표준
- [`agent-memory.md`](agent-memory.md) — agent-memory 정책
- [`rules-guide.md`](rules-guide.md) — rules 작성 메타 가이드
- [`tech-knowledge.md`](tech-knowledge.md) — rules/tech 누적 정책
- [`session-init.md`](session-init.md) — 세션 시작 시 컨텍스트 로딩
- [`/README.md`](../../README.md) — 폴더 트리 스냅샷
- [`/commands/ReviewClaudeConfig.md`](../../commands/ReviewClaudeConfig.md) — `.claude/` 폴더 전체 검토 커맨드 (사용자 명시 호출형 — "검토해줘", "전체 점검" 등)
