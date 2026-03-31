# .claude/ — 범용 AI 개발 프레임워크

이 폴더는 Claude Code와 함께 사용하는 **범용 개발 프레임워크**입니다.
어떤 기술 스택의 프로젝트든 이 폴더를 복사하면 즉시 체계적인 AI 기반 개발 환경이 구축됩니다.

---

## 새 프로젝트 시작 가이드

### Step 1: .claude 폴더 복사

기존 프로젝트에서 `.claude/` 폴더 전체를 새 프로젝트 루트에 복사합니다.

```bash
cp -r {기존프로젝트}/.claude/ {새프로젝트}/.claude/
```

### Step 2: CLAUDE.md 작성 (필수, 최우선)

프로젝트 루트에 `/CLAUDE.md`를 작성합니다. **이 파일이 전체 시스템의 핵심**입니다.
모든 에이전트, 규칙, 워크플로우가 이 파일을 읽고 프로젝트에 맞게 동작합니다.

아래 템플릿을 복사하여 프로젝트에 맞게 수정하세요:

````markdown
# {프로젝트명}

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
{프로젝트의 주요 폴더/파일 구조를 트리로 작성}

```
{project-root}/
├── src/              # 소스 코드
│   ├── ...
├── tests/            # 테스트
├── docs/             # 문서
│   ├── sprint/       # 스프린트 명세서
│   └── phase/        # Phase 계획서
├── CLAUDE.md         # 이 파일
├── ROADMAP.md        # 로드맵
└── deploy.md         # 배포 기록
```

## 빌드 & 실행

### 빌드 명령
```bash
{프로젝트 빌드 명령}
```

### 개발 서버 실행
```bash
{개발 서버 실행 명령}
```

### 테스트 실행
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
- 작업 브랜치: `sprint{N}` / `feature/*` / `hotfix/*` → develop으로 PR

## CI/CD
{CI/CD 파이프라인 설명 — GitHub Actions, Jenkins 등}

## 배포
{배포 환경 및 절차 — Vercel, AWS, Docker 등}

## 코드 컨벤션
- {프로젝트 고유 네이밍 규칙}
- {폴더/파일 배치 규칙}
- {커밋 메시지 형식: conventional commits 등}

## 외부 연동 (해당 시)
- {API, 서비스 연동 정보}
- {인증 방식}

## Notion 연동 (해당 시)
- 루트 페이지: {URL}
- Integration 토큰: 환경 변수 `NOTION_TOKEN`으로 관리
- 하위 페이지 ID: {각 페이지별 ID}
````

### Step 3: 기술 스택 규칙 확인

`.claude/rules/` 폴더에서 프로젝트 기술 스택에 맞는 규칙 파일이 있는지 확인합니다.

| 기술 스택 | 규칙 파일 | 상태 |
|-----------|-----------|------|
| TypeScript | `rules/typescript.md` | 제공됨 |
| React Native + Expo | `rules/react-native.md` | 제공됨 |
| Supabase | `rules/supabase.md` | 제공됨 |
| C# | `rules/csharp.md` | 제공됨 |
| Notion 연동 | `rules/notion.md` | 제공됨 |

**규칙 파일이 없는 기술 스택이라면:**
프로젝트 진행하면서 Claude에게 `rules/{tech}.md`를 생성하도록 요청하세요.
한번 생성하면 다음 프로젝트에서도 재사용됩니다 — 이것이 **범용성 확장**의 핵심입니다.

### Step 4: 프로젝트 종속 데이터 초기화

**중요:** .claude 폴더는 범용 프레임워크이므로 **프로젝트 종속 데이터가 포함되어서는 안 됩니다.**
이전 프로젝트에서 복사해온 경우, 아래 항목을 반드시 초기화하세요.

| 파일/폴더 | 종속성 | 새 프로젝트 시 조치 |
|-----------|--------|-------------------|
| `agent-memory/**` | 프로젝트 종속 | **내용 초기화** (에이전트 메모리 리셋) |
| `settings.local.json` | 환경 종속 | **확인 후 정리** (이전 프로젝트 권한 제거) |
| `rules/{tech}.md` | 범용 (재사용) | 유지 — 다른 프로젝트에서도 재사용 |
| `agents/**` | 범용 (재사용) | 유지 — 기술 스택 무관 |
| `commands/**` | 범용 (재사용) | 유지 — 기술 스택 무관 |
| `rules/sprint-workflow.md` | 범용 (재사용) | 유지 |
| `memory/feedback_workflow.md` | 사용자 선호 (재사용) | 유지 — 작업 스타일 공통 |
| `README.md` | 범용 (재사용) | 유지 |
| `settings.json` | 범용 (재사용) | 유지 |

> **원칙**: .claude 폴더 안에는 특정 프로젝트에서만 의미 있는 데이터를 넣지 않습니다.
> `agent-memory/`는 에이전트가 프로젝트 진행 중 자동으로 기록하는 영역이므로, 새 프로젝트 시작 시 리셋이 필요합니다.

> **팁**: Claude에게 "새 프로젝트 시작할 거야"라고 말하면, 프로젝트 종속 데이터를 확인하고 정리를 제안합니다.

### Step 5: 프로젝트 문서 작성 (필요 시)

| 문서 | 용도 | 작성 시점 |
|------|------|-----------|
| `ROADMAP.md` | Phase/Sprint 구조의 전체 로드맵 | PRD 작성 후 `prd-to-roadmap` 에이전트로 생성 |
| `docs/prd.md` | 요구사항 정의서 | 기능 개발 시작 전 |
| `docs/phase/phase{N}.md` | Phase 상세 계획 | `phase-planner` 에이전트로 생성 |
| `docs/sprint/sprint{N}.md` | Sprint 실행 명세 | `sprint-planner` 에이전트로 생성 |
| `deploy.md` | 배포 기록 및 검증 결과 | 첫 스프린트 마무리 시 자동 생성 |

### Step 5.5: settings.json 권한 설정 (선택)

기본 `settings.json`에는 `cd`와 `git` 명령만 허용되어 있습니다.
프로젝트 기술 스택에 맞는 빌드/테스트/린트 명령을 추가하면 매번 승인 없이 실행할 수 있습니다.

```json
{
  "permissions": {
    "allow": [
      "Bash(cd *:*)",
      "Bash(git *)",
      "Bash(npm *)",
      "Bash(npx *)",
      "Bash(python *)",
      "Bash(dotnet *)"
    ]
  }
}
```

> 프로젝트에 필요한 명령만 추가하세요. 불필요한 권한은 보안상 제거를 권장합니다.

### Step 6: 개발 시작

```
개발 사이클:

0. project-init 에이전트 → 기술 스택 결정 → rules 생성 → CLAUDE.md 생성
1. PRD 작성 → prd-to-roadmap 에이전트 → ROADMAP.md 생성
2. phase-planner 에이전트 → Phase 상세 계획 생성
3. sprint-planner 에이전트 → Sprint 실행 명세서 생성
4. /sprint-dev {N} → Sprint 구현
5. sprint-close 에이전트 → PR 생성 + 문서 정리
6. sprint-review 에이전트 → 코드 리뷰 + 검증
7. deploy-prod 에이전트 → 배포 준비
8. 반복
```

---

## 폴더 구조

```
.claude/
├── README.md                    # 이 파일 — 사용 가이드
├── settings.json                # Claude Code 권한 설정 (공유)
├── settings.local.json          # 로컬 전용 권한 설정 (환경별)
│
├── rules/                       # 규칙 라이브러리 (기술 스택별 + 워크플로우)
│   ├── TEMPLATE.md              #   rules 파일 작성 가이드 (메타 템플릿)
│   ├── session-init.md          #   세션 시작 시 컨텍스트 로딩 규칙
│   ├── prd-guide.md             #   PRD 작성 가이드 (품질 기준)
│   ├── sprint-workflow.md       #   스프린트/핫픽스 워크플로우 (범용)
│   ├── typescript.md            #   TypeScript 베스트 프랙티스
│   ├── react-native.md          #   React Native + Expo 베스트 프랙티스
│   ├── supabase.md              #   Supabase 베스트 프랙티스
│   ├── csharp.md                #   C# 베스트 프랙티스
│   ├── notion.md                #   Notion 연동 규칙
│   └── {tech}.md                #   (프로젝트 진행하며 확장)
│
├── agents/                      # 자동화 에이전트 (범용)
│   ├── project-init.md          #   새 프로젝트 초기 설정 (기술 스택 + CLAUDE.md)
│   ├── prd-to-roadmap.md        #   PRD → ROADMAP 변환
│   ├── phase-planner.md         #   Phase 상세 계획 수립
│   ├── sprint-planner.md        #   Sprint 실행 명세서 생성
│   ├── sprint-close.md          #   Sprint 마무리 (PR + 문서)
│   ├── sprint-review.md         #   코드 리뷰 + 검증
│   ├── hotfix-close.md          #   Hotfix 마무리
│   └── deploy-prod.md           #   배포 준비
│
├── commands/                    # 사용자 실행 커맨드
│   └── sprint-dev.md            #   /sprint-dev {N} — Sprint 구현 오케스트레이터
│
├── memory/                      # 사용자 선호 & 작업 스타일 (범용, 프로젝트 간 재사용)
│   ├── MEMORY.md                #   메모리 인덱스
│   └── feedback_workflow.md     #   작업 진행 방식 피드백
│
└── agent-memory/                # 에이전트 영구 메모리 (프로젝트 종속, 새 프로젝트 시 리셋)
    ├── phase-planner/MEMORY.md
    ├── prd-to-roadmap/MEMORY.md
    ├── sprint-planner/MEMORY.md
    └── sprint-review/MEMORY.md
```

**`memory/` vs `agent-memory/` 차이:**
- `memory/` — 사용자의 작업 스타일, 선호도 등 **프로젝트와 무관한 범용 피드백**. 새 프로젝트에서도 그대로 유지.
- `agent-memory/` — 각 에이전트가 **현재 프로젝트 진행 중** 기록하는 데이터 (스프린트 번호, 주의사항 등). 새 프로젝트 시작 시 반드시 리셋.

---

## 핵심 동작 원리

```
                    CLAUDE.md (프로젝트 정의)
                         │
                         ▼
              ┌─── 기술 스택 확인 ───┐
              │                      │
              ▼                      ▼
     rules/{tech}.md          rules/sprint-workflow.md
     (기술 전문 규칙)            (워크플로우 규칙)
              │                      │
              └──────────┬───────────┘
                         │
                         ▼
                   agents/ & commands/
                (기술 스택 전문가로 동작)
                         │
                         ▼
              베스트 프랙티스 품질의 코드
```

0. `project-init` 에이전트가 사용자와 함께 기술 스택을 결정하고 CLAUDE.md + rules 생성
1. 이후 에이전트들이 `/CLAUDE.md`를 읽어 프로젝트의 기술 스택, 빌드 명령, 구조를 파악
2. `rules/` 에서 해당 기술 스택의 규칙 파일을 로드하여 전문가 지식 확보
3. 기술 스택에 특화된 코드 리뷰, 빌드 검증, 코드 품질 기준으로 작업 수행

---

## 범용성 확장 가이드

이 프레임워크는 프로젝트를 진행할수록 강해집니다.

**새 기술 스택을 사용할 때:**
1. 해당 기술의 `rules/{tech}.md`가 없으면 Claude에게 생성 요청
2. 프로젝트 진행하면서 규칙 보완
3. 완성된 규칙 파일을 다른 프로젝트에도 적용

**에이전트/커맨드 개선:**
1. 워크플로우 개선점이 발견되면 agents/, commands/ 파일 수정
2. 수정 사항은 모든 프로젝트에 즉시 적용 가능

**새 에이전트 추가:**
1. 반복 작업이 발견되면 `agents/{name}.md`로 자동화
2. 범용적으로 작성하면 다른 프로젝트에서도 사용 가능

> .claude 폴더가 git에 포함되어 있으므로, 개선 사항은 가장 활발한 프로젝트에서 작업한 뒤
> 다른 프로젝트로 복사하여 동기화합니다.
