---
paths:
  - "**/*"
---

# 세션 시작 시 컨텍스트 로딩 규칙

> 이 규칙은 모든 작업(Sprint, ROADMAP, Phase, 코드 리뷰, 문서 작업 등)에 공통 적용된다.
> 어떤 작업이든 컨텍스트 없이 시작하면 품질이 떨어지고 같은 실수를 반복한다.

---

## 자동 로딩 (시스템이 처리)
- `/CLAUDE.md` — 프로젝트 기술 스택, 구조, 빌드 명령

## 수동 로딩 (에이전트가 직접 읽어야 함)

아래 파일이 로딩되지 않은 상태에서 작업을 시작하지 않는다.

| 순서 | 대상 | 목적 |
|------|------|------|
| 1 | `.claude/memory/MEMORY.md` | 사용자 프로필, 피드백, 프로젝트 상태 파악 |
| 2 | 작업에 관련된 `.claude/rules/*.md` | 워크플로우/기술 규칙 확인 |
| 3 | CLAUDE.md의 기술 스택에 해당하는 `rules/{tech}.md` | 기술 스택 전문가로 동작 |
| 4 | `ROADMAP.md` (있는 경우) | 현재 진행 상황 파악 |

### 작업 유형별 로딩 대상

모든 작업에 전부 읽을 필요는 없다. **작업 유형에 따라 필요한 것만** 읽는다:

| 작업 유형 | 필수 로딩 | 추가 로딩 (해당 시) |
|----------|----------|-------------------|
| **코드 구현** (Sprint 등) | MEMORY.md → rules/{tech}.md → sprint-workflow.md | ROADMAP.md, sprint{N}.md |
| **코드 리뷰** | MEMORY.md → rules/{tech}.md | sprint-workflow.md (품질 기준) |
| **문서 작성** (PRD, 정리 등) | MEMORY.md → prd-guide.md 또는 notion.md | ROADMAP.md |
| **Sprint 계획** | MEMORY.md → sprint-workflow.md → ROADMAP.md | rules/{tech}.md (Task 설계 시) |
| **단순 질문/탐색** | MEMORY.md | 질문 관련 rules (해당 시) |
| **새 프로젝트 시작** | MEMORY.md → README.md | session-init.md 정리 절차 |

---

## 로딩 확인 원칙

- 세션 시작 후 첫 작업 요청 시, 위 파일을 아직 읽지 않았으면 **먼저 읽고 나서 작업을 시작**한다
- 사용자가 "규칙 로딩해", ".claude 읽고 와" 등을 요청하면 즉시 전체 로딩을 수행한다
- memory에 저장된 피드백은 작업 중 **능동적으로 적용**한다 — 사용자가 같은 피드백을 반복하지 않도록
- memory 인덱스(MEMORY.md)를 읽고, 현재 작업에 관련된 개별 memory 파일을 추가로 읽는다

### Memory 로딩 깊이

```
1. MEMORY.md (인덱스) 읽기
   → 어떤 memory 파일이 있는지 목록 파악

2. 현재 작업과 관련된 memory 파일만 선별하여 읽기
   → user_*.md: 항상 (사용자 스타일 파악)
   → feedback_*.md: 항상 (같은 실수 반복 방지)
   → project_*.md: 해당 프로젝트 작업 시
   → reference_*.md: 외부 시스템 관련 작업 시
```

---

## 로딩하지 않았을 때 발생하는 문제

| 로딩 누락 | 실제 발생하는 문제 |
|----------|-----------------|
| MEMORY.md 미로딩 | 사용자 피드백을 모르고 같은 실수 반복 (예: 파일 한꺼번에 수정, 커밋 전 확인 생략) |
| rules/{tech}.md 미로딩 | 해당 기술의 안티패턴을 모르고 그대로 코드 생성 (예: 하드코딩, 잘못된 네이밍) |
| sprint-workflow.md 미로딩 | 품질 기준/보안 원칙 무시, 우회/땜빵 해결책 사용 |
| ROADMAP.md 미로딩 | 프로젝트 진행 상황을 모르고 엉뚱한 작업 시작, 이미 완료된 작업 재시도 |
| feedback_*.md 미로딩 | "한 스텝씩 진행" 원칙을 모르고 8개 파일 동시 수정 시도 |

### Good vs Bad 시나리오

```
❌ Bad — 컨텍스트 없이 바로 코드 작성
사용자: "이 함수 리팩토링해줘"
AI: (rules 안 읽고) 바로 코드 수정 → 네이밍 컨벤션 위반, 공용화 원칙 무시, 파일 3개 동시 수정
사용자: "왜 컨벤션 안 지켜?" / "한 파일씩 하라고 했잖아"
```

```
✅ Good — 컨텍스트 로딩 후 작업
사용자: "이 함수 리팩토링해줘"
AI: MEMORY.md → feedback_work_style.md (한 스텝씩) → rules/{tech}.md (네이밍, 구조) 읽기
    → 파일 1개씩, 컨벤션 준수하여 수정
사용자: "좋아, 다음 파일"
```

---

## .claude 폴더를 새 프로젝트에 복사했을 때 정리 절차

> .claude 폴더는 범용 문서(rules, feedback memory, agents, commands)로 구성되어 있어
> 다른 프로젝트에 그대로 복사해서 재사용할 수 있다.
> 단, **이전 프로젝트에서 자동 생성된 프로젝트 종속 파일**이 남아있을 수 있으므로
> 새 프로젝트 시작 시 아래 정리를 먼저 수행한다.

### 왜 정리해야 하는가
- `memory/`와 `agent-memory/`에 이전 프로젝트의 인프라 정보, Sprint 상태 등이 남아있으면
  에이전트가 현재 프로젝트가 아닌 **이전 프로젝트 맥락으로 동작**할 수 있다
- 예: 이전 프로젝트의 Sprint 2 상태가 남아있으면 sprint-planner가 Sprint 3부터 시작하려 함

### 정리 대상 (삭제 또는 초기화)

| 위치 | 대상 | 판단 기준 |
|------|------|----------|
| `memory/` | `project_*.md` 파일 | type: project인 파일은 프로젝트 종속 — 삭제 |
| `agent-memory/*/` | 각 에이전트의 project 관련 `.md` | 에이전트가 작업 중 생성한 프로젝트 상태 파일 — 삭제 |
| `memory/MEMORY.md` | 삭제한 파일의 참조 라인 | 인덱스에서 해당 항목 제거 |
| `settings.local.json` | `permissions.allow` 배열 | 이전 프로젝트에서 허용한 명령어 이력 — **비우기** (`[]`로 초기화) |

> **settings.local.json 주의**: 이전 프로젝트 작업 중 허용한 명령어에 **API 토큰, 비밀 키, 프로젝트 경로** 등이
> 포함되어 있을 수 있다. 새 프로젝트 시작 시 반드시 `allow` 배열을 비워야 민감 정보가 남지 않는다.

### 정리하면 안 되는 것 (범용 — 유지)

| 위치 | 유지 대상 |
|------|----------|
| `memory/` | `user_*.md`, `feedback_*.md`, `reference_*.md` |
| `memory/MEMORY.md` | 위 파일들의 참조 라인 |
| `rules/` | 모든 파일 (기술 스택별 rules는 paths 매칭으로 자동 필터링) |
| `agents/`, `commands/` | 모든 파일 (범용 워크플로우) |
| `agent-memory/*/MEMORY.md` | 인덱스 파일 자체는 유지 (내용만 비움) |

### 정리 순서
1. `memory/`에서 `project_*.md` 파일 삭제
2. `agent-memory/*/`에서 프로젝트 종속 `.md` 파일 삭제
3. `memory/MEMORY.md`에서 삭제된 파일 참조 제거
4. 각 `agent-memory/*/MEMORY.md`에서 삭제된 파일 참조 제거
5. `settings.local.json`의 `allow` 배열 확인 및 정리
6. 정리 완료 후 프로젝트 작업 시작

### 정리 완료 확인 체크리스트
- [ ] `memory/`에 `project_*.md` 파일이 0개인가?
- [ ] `agent-memory/*/`에 프로젝트 종속 파일이 0개인가?
- [ ] `MEMORY.md` 인덱스에 삭제된 파일 참조가 없는가?
- [ ] `settings.local.json`에 이전 프로젝트의 토큰/경로가 남아있지 않은가?
- [ ] `/CLAUDE.md`가 현재 프로젝트에 맞게 작성/수정되었는가?
