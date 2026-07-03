# Memory Folder Guide

> 사용자 작업 스타일, 선호도, 핵심 원칙을 담는 **범용 메모리 폴더**입니다.
>
> 이 폴더는 프로젝트 무관 — 모든 프로젝트에서 그대로 재사용됩니다.
> 프로젝트 종속 메모리는 별도 위치에 둡니다 (참조: `feedback_document_guide.md`).

---

## 로딩 방식 (Loading Policy)

**이 폴더는 `.claude/rules/` 하위이므로 Claude Code가 매 세션 자동 로딩한다** (`paths` frontmatter 없음 = 무조건 로딩).

- 수동 호출(`/InitLoad`) 불필요 — 세션 시작 시 시스템이 자동 주입.
- 새 메모리 파일은 **이 폴더에 두기만 하면 자동 로딩** — 인덱스 갱신 불필요.
- MEMORY.md 자신도 자동 로딩됨 (가이드/추가 절차를 함께 인지).
- ⚠️ **`paths` frontmatter를 넣지 말 것** — 넣으면 조건부 로딩으로 바뀌어 세션 시작 시 누락됨.
- 자동 로딩 예산: 상시 로딩 총량 관리는 `rules/workflow/session-init.md` 의 예산(~35KB)이 진실 원천 — 이 폴더가 그 예산을 압박하면 오래된/저빈도 항목 통합 검토.

> 변경 이력: 2026-07-03 `.claude/memory/` → `.claude/rules/memory/` 이동 — Claude Code의 rules 네이티브 자동 로딩 도입에 따라 수동 로딩(글롭 일괄 Read) 정책 폐기.

---

## 메모리 카테고리

파일명 prefix 로 카테고리 구분 (단순 명명 규약 — 로딩에는 영향 없음):

| Prefix | 용도 |
|--------|------|
| `user_*.md` | 사용자 프로필 (역할, 환경, 작업 스타일) |
| `feedback_*.md` | 작업 원칙/스타일 (반복 피드백 방지) |
| `project_*.md` | 특정 프로젝트 컨텍스트 (범용 폴더에는 두지 않음 — 프로젝트별 위치) |
| `reference_*.md` | 외부 시스템 포인터 (Linear, Slack 등) |

---

## 메모리 추가/수정 절차

### 새 메모리 추가
1. **type 결정**: `user` / `feedback` / `project` / `reference`
2. **파일명**: `{type}_{topic}.md` (예: `feedback_test_guide.md`)
3. **frontmatter**: `name`, `description`, `type` 필수
4. **본문**: 룰의 경우 **Why** + **How to apply** 포함
5. 저장 — 다음 세션부터 자동 로딩 (별도 절차 불필요).

### 메모리 위치 결정 (요약)

| 종류 | 위치 | 이유 |
|------|------|------|
| 범용 (모든 프로젝트 적용) | `.claude/rules/memory/` (이 폴더) | git 에 포함, 다른 프로젝트로 복사됨 |
| 프로젝트 종속 | 프로젝트 별도 위치 | 이 프로젝트에서만 유효 |

> 자세한 위치 결정 가이드는 [feedback_document_guide.md](feedback_document_guide.md) 참조.

### .claude/rules/memory/ 종속성 금지
- 범용 메모리에 **프로젝트 종속 내용 (프로젝트명, 특정 기능명 등) 절대 금지**.
- 예시도 범용적으로 작성.
- `.claude` 폴더는 다른 프로젝트에 그대로 복사 가능해야 함.
