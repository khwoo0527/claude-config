# CLAUDE.md 템플릿

> 새 프로젝트 시작 시 `project-init` 에이전트가 이 템플릿을 기반으로 프로젝트 루트의 `/CLAUDE.md` 를 생성합니다.
> 직접 작성하는 경우 아래 템플릿(코드 블록 안 내용)을 프로젝트 루트의 `CLAUDE.md` 로 복사한 뒤 채워 넣으세요.

---

````markdown
# {프로젝트명}

## 세션 컨텍스트 로딩 (자동)

이 프로젝트의 `.claude/rules/` 는 Claude Code 가 **네이티브 자동 로딩**합니다:
- `.claude/rules/memory/` — 사용자 작업 스타일·원칙 (`paths` 없음 → 매 세션 자동)
- `.claude/rules/workflow/session-init.md` — 세션 절차/작업 유형별 매트릭스 (매 세션 자동)
- `.claude/rules/tech/{tech}.md` — 기술 스택 전문 규칙 (`paths` 매칭 파일 작업 시 조건부 자동)
- `.claude/rules/workflow/*.md` — 워크플로우 규칙 (paths 유무 따라 상시/조건부)

**계획·리뷰 등 코드 파일을 안 건드리는 작업**은 조건부 로딩이 발동하지 않으므로, `session-init.md` 의 작업 유형별 매트릭스에 따라 해당 rules 를 명시적으로 Read 합니다.
메모리 자동 주입이 의심될 때만 `/InitLoad` (로딩 상태 점검) 를 호출합니다.

## 에이전트 진입 절차 (필수, 모든 에이전트 공통)

이 프로젝트의 어떤 에이전트로 호출되었든 (Task tool 통한 서브 에이전트 포함), **본 에이전트의 작업 절차에 진입하기 전에** 진입 절차를 먼저 수행한다.

**진입 절차 (핵심)**: 메모리 → 기술 룰 → 작업 유형별 추가 로딩 → (해당 시) 자기 agent-memory.

**자세한 단계 (Read 위치, 매트릭스, 위반 감지, Good/Bad 시나리오)**: [`.claude/rules/workflow/session-init.md`](./.claude/rules/workflow/session-init.md) 의 "에이전트 진입 절차" 섹션 참조 — 이 명령은 모든 에이전트에 강제 적용된다.

> ⚠️ **이 절차 생략 시 발생하는 문제**: 사용자 룰 모르고 작업 시작 → 한꺼번에 여러 파일 수정, 컨벤션 위반, 같은 피드백 반복.

## 프로젝트 개요
- 목적: {한 줄 설명}
- 유형: {웹 앱 / 데스크톱 앱 / API 서버 / 라이브러리 / CLI 도구 등}

## 기술 스택
- 언어: {예: TypeScript, Python, C# 등}
- 프레임워크: {예: React, FastAPI, .NET 8 등}
- 런타임: {예: Node.js 20, Python 3.12, .NET 8 등}
- 패키지 관리: {예: npm, pip, NuGet 등}
- 데이터베이스: {예: PostgreSQL, MongoDB, SQLite 등} (해당 시)
- 주요 라이브러리: {프로젝트 핵심 의존성 목록}

## 프로젝트 구조
```
{project-root}/
├── src/              # 소스 코드
├── tests/            # 테스트
├── docs/             # 문서
│   ├── sprint/       # 스프린트 명세서
│   └── phase/        # Phase 계획서
├── CLAUDE.md         # 이 파일
├── ROADMAP.md        # 로드맵
└── deploy.md         # 배포 기록
```

## 빌드 & 실행

### 빌드
```bash
{프로젝트 빌드 명령}
```

### 개발 서버
```bash
{개발 서버 실행 명령}
```

### 테스트
```bash
{테스트 실행 명령}
```

### 린트/포맷
```bash
{린트, 포맷팅 명령}
```

## 브랜치 전략
- 메인 브랜치: `main` (또는 `master`)
- 개발 브랜치: `develop`
- 작업 브랜치: `sprint{N}` / `feature/*` / `hotfix/*` → develop 으로 PR

## CI/CD
{CI/CD 파이프라인 설명}

## 배포
{배포 환경 및 절차}

## 코드 컨벤션
- {프로젝트 고유 네이밍 규칙}
- {커밋 메시지 형식}

## 외부 연동 (해당 시)
- {API/서비스명}: 인증 방식 / 환경변수명 / 문서 링크

> **토큰/시크릿 관리**: 정석은 `.env` / 환경변수 분리.
> 임시 평문 보관 시 ⚠️ 위험 표시 + 분리 일정 명시 필수 (public 전환 전 반드시 .env 분리).
> 자세한 정책: [`rules/memory/feedback_principles.md`](./.claude/rules/memory/feedback_principles.md) "토큰/시크릿 관리" 섹션 참조.

## Notion 연동 (해당 시)
- 루트 페이지: {URL}
- 하위 페이지 ID: {각 페이지별 ID}
- Integration 토큰: **권장** — 환경 변수 `NOTION_TOKEN` (.env / .env.local)
  - 임시 평문 (private repo 한정): `토큰 값` ⚠️ public 전환 전 .env 분리 필수 (분리 일정: {예: Phase 2})
````
