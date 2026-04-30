---
paths:
  - ".claude/rules/tech/**"
---

# Tech Knowledge 누적 정책 (단일 진실 원천)

> `.claude/rules/tech/{tech}.md` 에 **범용 기술 노하우를 어떻게 누적할지** 정의한다.
> rules/tech 는 정적 규칙뿐 아니라 작업 중 발견한 노하우의 누적 자산이기도 하다.

---

## 1. 왜 rules/tech 에 누적하나

- `rules/tech/` 는 모든 프로젝트에서 그대로 재사용된다 (범용 — 누적 가치).
- agent-memory 는 새 프로젝트 시 리셋되므로 거기에 노하우를 누적하면 다음 프로젝트에서 무의미.
- 따라서 **"다른 프로젝트에도 통하는 기술 노하우" 는 rules/tech 가 유일한 누적 위치**.

---

## 2. 어떤 정보가 rules/tech 에 들어가나

### ✅ 들어가는 것 (범용 기술 노하우)
- 안티패턴 (예: "Supabase 의 update_updated_at() 함수 중복 정의 금지 — 기본 제공됨")
- 함정/지뢰 (예: "RLS 정책 누락 시 익명 접근 가능")
- 베스트 프랙티스 (예: "환경변수는 env.ts 한 곳에서 관리")
- 보안/성능/안정성 패턴
- 버전 호환성 메모

### ❌ 들어가지 않는 것
- **이 프로젝트만의 특정 모듈/스키마** → `agent-memory` 또는 `CLAUDE.md`
- **사용자 작업 스타일** → `memory/feedback_*.md`
- **공식 진행 상태** → `ROADMAP.md` / `sprint{N}.md`

---

## 3. 누적 시점 — 사용자 컨펌

작업 중 다른 프로젝트에도 통하는 기술 노하우 발견 시:

```
💡 발견 (범용 노하우 후보): {요약}
{tech}.md 의 {섹션} 에 추가할까요? (Y/N)
```

- 사용자 Y → 추가 후 기록 보고
- 사용자 N → 무시 (또는 agent-memory 로 격하)
- 위임 신호 받은 경우 자동 추가 + 사후 보고

---

## 4. 누적 형식

기존 `rules/tech/{tech}.md` 의 섹션 구조에 맞춰 추가:

| 발견 종류 | 추가 위치 (대표 예시) |
|---|---|
| 안티패턴 | "안티패턴" 섹션 |
| 함정/지뢰 | "실전 함정" 섹션 |
| 베스트 프랙티스 | "패턴" 또는 "권장 사항" 섹션 |
| 보안 이슈 | "보안" 섹션 |

> 적합한 섹션이 없으면 새로 만든다. 단 [`RULES-TEMPLATE.md`](../../templates/RULES-TEMPLATE.md) 의 구조 원칙을 따른다.

### 항목 형식
- 한 항목당 1~3줄 권장
- "안티패턴 — 왜 — 어떻게 (예시)" 의 흐름

---

## 5. 점수/품질 기준

[`RULES-TEMPLATE.md`](../../templates/RULES-TEMPLATE.md) 의 자가 채점 기준을 따른다. 누적 후 점수 변화는 별도 보고 X (외부 문서 README 갱신은 [`/UpdateReadme`](../../commands/UpdateReadme.md) 시점에).

---

## 6. 분기 헷갈릴 때

다음 표로 판단:

| 발견 정보 | 다른 프로젝트에도 통하나? | 위치 |
|---|:-:|---|
| Supabase trigger 중복 정의 함정 | ✅ Yes | `rules/tech/supabase.md` |
| 이 프로젝트의 `clubs` 테이블 스키마 | ❌ No | `agent-memory` 또는 `CLAUDE.md` |
| 사용자가 한꺼번에 작업 싫어함 | (사용자 스타일) | `memory/feedback_work_style.md` |
| Sprint 3 다음 진입 | (진행 상태) | `ROADMAP.md` |

판단 애매하면 사용자에게 컨펌.

---

## 7. 위반 감지

- agent-memory 에 다른 프로젝트에도 통할 일반 기술 노하우가 있음 → `rules/tech/{tech}.md` 로 이전 제안
- 같은 노하우가 여러 위치에 중복 → rules/tech 가 진실 원천으로 통합
