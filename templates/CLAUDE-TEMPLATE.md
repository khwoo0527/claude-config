# CLAUDE.md 템플릿

> 새 프로젝트 시작 시 `project-init` 에이전트가 이 템플릿을 기반으로 프로젝트 루트의 `/CLAUDE.md` 를 생성합니다.
> 직접 작성하는 경우 아래 템플릿(코드 블록 안 내용)을 프로젝트 루트의 `CLAUDE.md` 로 복사한 뒤 채워 넣으세요.

---

````markdown
# {프로젝트명}

## 세션 시작 시 자동 로딩 (필수)

이 프로젝트는 `.claude/` 폴더에 다음을 포함합니다:
- `.claude/memory/` — 사용자 작업 스타일·원칙 (자동 로딩 대상 아님)
- `.claude/rules/workflow/session-init.md` — 세션 절차/작업 유형별 로딩 매트릭스
- `.claude/rules/tech/{tech}.md` — 기술 스택 전문 규칙
- `.claude/rules/workflow/*.md` — 워크플로우 규칙

세션 시작 시 사용자는 **`/InitLoad` 슬래시 커맨드를 호출**하여 메모리와 세션 절차를 일괄 로딩한 뒤 작업을 시작합니다.
`/InitLoad` 가 호출되지 않은 채로 작업 요청이 들어오면 — Claude 는 먼저 `/InitLoad` 절차를 수행하고 작업을 진행합니다.

> Claude Code 가 자동 로딩하는 것은 이 `CLAUDE.md` 와 사용자 홈 메모리뿐입니다.
> `.claude/memory/`, `.claude/rules/` 는 `/InitLoad` 또는 명시적 Read 가 필요합니다.

## 에이전트 진입 절차 (필수, 모든 에이전트 공통)

이 프로젝트의 어떤 에이전트로 호출되었든 (Task tool 통한 서브 에이전트 포함), **본 에이전트의 작업 절차에 진입하기 전에** 다음을 먼저 수행한다:

1. **메모리 로딩** — `.claude/memory/MEMORY.md` 의 로딩 방식 따라 메모리 일괄 Read (사용자 작업 스타일·원칙 인지).
2. **기술 룰 로딩** — 위 "기술 스택" 항목 보고 해당 `.claude/rules/tech/{tech}.md` Read.
3. **작업 유형별 추가 로딩** — `.claude/rules/workflow/session-init.md` 의 "작업 유형별 로딩 대상" 표 참조하여 필요한 것만.
4. (해당 시) **에이전트 자기 메모리** — `.claude/agent-memory/{에이전트명}/MEMORY.md` 가 있으면 Read.

자세한 절차/위반 감지/Good vs Bad 시나리오: [`.claude/rules/workflow/session-init.md`](./.claude/rules/workflow/session-init.md) 의 "에이전트 진입 절차" 섹션 참조.

> ⚠️ **이 절차 생략 시 발생하는 문제**: 사용자 룰 모르고 작업 시작 → 한꺼번에 여러 파일 수정, 컨벤션 위반, 같은 피드백 반복. 본 절차는 모든 에이전트에 강제된다.

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
- {API, 서비스 연동 정보}

## Notion 연동 (해당 시)
- 루트 페이지: {URL}
- Integration 토큰: 환경 변수 `NOTION_TOKEN` 으로 관리
- 하위 페이지 ID: {각 페이지별 ID}
````
