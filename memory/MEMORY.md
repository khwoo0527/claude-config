# Memory Folder Guide

> 사용자 작업 스타일, 선호도, 핵심 원칙을 담는 **범용 메모리 폴더**입니다.
>
> 이 폴더는 프로젝트 무관 — 모든 프로젝트에서 그대로 재사용됩니다.
> 프로젝트 종속 메모리는 별도 위치에 둡니다 (참조: `feedback_document_guide.md`).

---

## 로딩 방식 (Loading Policy)

**이 폴더의 모든 `.md` 파일을 글롭(`.claude/memory/*.md`)으로 일괄 Read.**

- `/InitLoad` 커맨드가 이 정책을 그대로 수행함 — 정책 변경 시 이 섹션만 수정하면 됨.
- 새 메모리 파일은 **이 폴더에 두기만 하면 자동 로딩** — 인덱스 갱신 불필요.
- MEMORY.md 자신도 글롭에 포함됨 (가이드/추가 절차를 함께 인지).
- 누락 방지: 글롭 결과 파일 수 == 실제 Read 한 파일 수 확인.

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
5. 저장 — 다음 `/InitLoad` 부터 자동 로딩.

### 메모리 위치 결정 (요약)

| 종류 | 위치 | 이유 |
|------|------|------|
| 범용 (모든 프로젝트 적용) | `.claude/memory/` (이 폴더) | git 에 포함, 다른 프로젝트로 복사됨 |
| 프로젝트 종속 | 프로젝트 별도 위치 | 이 프로젝트에서만 유효 |

> 자세한 위치 결정 가이드는 [feedback_document_guide.md](feedback_document_guide.md) 참조.

### .claude/memory/ 종속성 금지
- 범용 메모리에 **프로젝트 종속 내용 (프로젝트명, 특정 기능명 등) 절대 금지**.
- 예시도 범용적으로 작성.
- `.claude` 폴더는 다른 프로젝트에 그대로 복사 가능해야 함.
