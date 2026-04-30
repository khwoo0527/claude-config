---
paths:
  - ".claude/agents/**"
---

# 에이전트 작성 가이드 (단일 진실 원천)

> 새 에이전트를 `.claude/agents/{name}.md` 에 작성할 때 따르는 표준 구조 + 정책.
>
> **원칙**: 본 가이드를 베이스로 작성하고, 정책 변경 시 이 가이드만 수정하면 모든 에이전트에 일괄 반영 가능하도록 한다 (단일 진실 원천).

---

## 1. frontmatter 표준

```yaml
---
name: {agent-name}              # kebab-case, 폴더의 다른 에이전트와 충돌 금지
description: "..."               # 한 문장 + <example> 블록 2~3개. 이 description 으로 사용자가 자연스러운 한국어 호출 시 매칭됨
model: opus | sonnet | haiku    # 작업 복잡도에 따라 선택
color: blue | green | red | ...  # 시각 구분
# skills: [...]                  # (선택) 사용할 Skill 명시. 없으면 생략
# maxTurns: 50                   # (선택) 긴 작업 한정. 기본값 사용 시 생략
---
```

### 폐지된 키 (사용 금지)

- `memory: project` — 옛 메타데이터. 메모리 사용 여부는 `agent-memory/{name}/` 폴더 존재로 판단 ([`agent-memory.md`](agent-memory.md) 참조).

---

## 2. 본문 구조 표준 (통일 패턴)

```markdown
당신은 **{역할}**입니다. {한 문단 — 무엇을 하는 에이전트인지}.

> **기술 스택 적응 / 전문가 모드** (해당 시): `/CLAUDE.md`에서 기술 스택을 확인하고, `.claude/rules/tech/{tech}.md`를 로드하여 ...

## 핵심 원칙

- {원칙 3~5개}

## 입력

{사용자/메인 Claude 가 제공해야 할 입력 명세}

## 작업 절차

### 1단계: 프로젝트 컨텍스트 파악
{필요 시 — 메모리/CLAUDE.md/rules 로딩}

### 2단계: ...
### 3단계: ...
...

### {N단계}: 발견된 정보 분기 기록

분기 룰: [`agent-memory.md`](../rules/workflow/agent-memory.md) + [`tech-knowledge.md`](../rules/workflow/tech-knowledge.md) 참조.
**핵심**: 발견 시 사용자 컨펌 후 분기 위치에 기록 (위임 신호 시 자동 + 사후 보고).

- 이 에이전트 캐시: `agent-memory/{name}/MEMORY.md` (또는 "자기 캐시 없음" + 다른 에이전트 캐시 사용)
- 주요 발견 영역: {도메인 특화 — 예시 1~2줄}
- 분기 1 변형 (해당 시): {특정 에이전트만 — 예: PRD/Notion 노하우는 rules/workflow/{...}.md 로}
- 관련 진실 원천: {ROADMAP / sprint{N} / deploy / phase{N} / CLAUDE.md 등 도메인 특화}

## 사용자 다음 단계 안내 (해당 시)

작업 완료 시 사용자에게 안내:
\`\`\`
📋 사용자 다음 단계:
1. ...
\`\`\`

## 에러 처리

- {에러 케이스별 대응}
```

---

## 3. 외부 서비스 위치 정의 패턴 (Notion/Slack/GitHub 등 사용 에이전트)

에이전트가 외부 서비스의 프로젝트 종속 정보 (페이지 ID, 토큰 등) 를 사용할 때, 본문 시작 부분에 **위치 정의 박스** 1곳 + 본문 짧은 참조로 작성:

```markdown
> **프로젝트 {서비스명} 설정 위치**: [`rules/workflow/{service}.md`](../rules/workflow/{service}.md) 정책 따름.
> 현재 정책 요약: 페이지 ID / 토큰 / ... 은 **`CLAUDE.md` 의 "{서비스명} 연동" 섹션**에 정의.
> 이하 본문의 모든 "{서비스명} 설정" 참조는 이 위치를 가리킨다.
```

**왜 이 패턴**:
- 위치 박힌 곳 1곳 (정책 변경 시 한 번만 수정)
- 다른 외부 서비스 에이전트 추가 시 같은 패턴 — 일관성 + 확장성
- `feedback_design_extensibility` 룰 부합

**참고**: `agents/notion-writer.md` 의 위치 정의 박스 (시작 부분) 가 표준 예시.

---

## 4. agent-memory 폴더 정책

### 폴더 생성 결정 트리

```
이 에이전트가 작업 중 캐시할 정보가 있는가?
  → YES: agent-memory/{name}/ 폴더 + 빈 MEMORY.md 생성
  → NO: 폴더 없음 (단순 에이전트)

자기 캐시 없이 다른 에이전트의 캐시에 기록하는가? (예: sprint-close → sprint-planner)
  → 본문 분기 기록 섹션에 명시 ("자기 캐시 없음 — 발견은 ... 에 기록")
```

### MEMORY.md 형식

[`agent-memory.md`](agent-memory.md) 의 "5. agent-memory 기록 방식" 참조. 표준 형식:

```markdown
# {Agent} — 작업 컨텍스트 캐시

> 이 파일의 용도/기록 시점/형식은 [`agent-memory 정책`](../../rules/workflow/agent-memory.md) 을 따른다.
>
> **담는 것**: {도메인 특화 컨텍스트}
> **담지 않는 것**:
> - 다른 프로젝트에 통하는 노하우 → `rules/tech/{tech}.md` 또는 `rules/workflow/{...}.md`
> - 사용자 작업 스타일 → `memory/feedback_*.md`
> - 공식 진행 상태 → ROADMAP/sprint/deploy/phase 가 진실 원천

---

## {섹션 1 — 도메인 특화}
(아직 캐시된 컨텍스트 없음)

## {섹션 2}
...
```

---

## 5. 진입 절차 (자동 적용)

새 에이전트는 본문에 진입 절차를 명시할 필요 없음. **`CLAUDE.md` 의 "에이전트 진입 절차" 섹션이 자동 로딩되어 모든 에이전트에 강제 적용**됨 ([`session-init.md`](session-init.md) 의 "에이전트 진입 절차" 섹션 참조).

본 작성 시 진입 절차 명시 금지 — 중복.

---

## 6. 작성 후 체크리스트

- [ ] frontmatter `memory: project` 같은 폐지된 키 없음
- [ ] description 에 한국어 자연 호출 예시 (`<example>`) 2~3개
- [ ] 본문에 "발견된 정보 분기 기록" 섹션 존재 (메모리 사용 시)
- [ ] 분기 룰: `agent-memory.md` + `tech-knowledge.md` 참조 명시
- [ ] 외부 서비스 사용 시 — 시작 부분에 위치 정의 박스 1곳 + 본문 짧은 참조
- [ ] agent-memory 폴더 생성 여부 결정 (필요 시 폴더 + 빈 MEMORY.md)
- [ ] 본문에 진입 절차 명시 X (CLAUDE.md 자동 로딩 강제)
- [ ] 사용자 다음 단계 안내 (해당 시)
- [ ] 에러 처리 케이스 명시

---

## 7. 살아있는 표준 — 패턴 분류 카탈로그

새 에이전트 작성 시 다음 분류 중 자기 에이전트가 어느 패턴에 속하는지 매핑하고, 대표 예시를 참고한다. 새 에이전트가 추가되어도 이 표는 갱신 불필요 — 패턴 자체가 카탈로그.

| 패턴 분류 | 특징 | 대표 예시 | 신규 에이전트 매핑 시 |
|---|---|---|---|
| **자기 캐시 보유** | 작업 컨텍스트 캐시 (`agent-memory/{name}/MEMORY.md`) 보유 — 다음 호출 시 복원 | `phase-planner`, `sprint-planner` | 반복 호출되며 컨텍스트 누적 가치 있는 경우 |
| **자기 캐시 없음** | 외부 진실 원천(deploy.md / PR / 다른 에이전트 캐시) 에 기록 | `sprint-close`, `hotfix-close`, `deploy-prod` | 일회성 마무리/검증/배포 작업 |
| **분기 누적 변형** | 작업 중 발견 노하우를 `rules/tech/*` 또는 `rules/workflow/*` 로 누적 | `sprint-review` (rules/tech), `prd-to-roadmap` (rules/workflow/prd-guide) | 도메인 노하우가 다른 프로젝트에도 통하는 경우 |
| **외부 시스템 연동** | 외부 서비스 위치 정의 박스 1곳 + 본문 짧은 참조 (§ 3 패턴) | `notion-writer` | Notion/Slack/GitHub/Jira 등 외부 서비스 사용 |
| **부트스트랩** | 다른 에이전트의 진입점 — `CLAUDE.md` / `rules/tech` 자동 생성 | `project-init` | 초기 세팅/스캐폴딩 — 워크플로우 시작점 |

> 한 에이전트가 여러 분류에 속할 수 있다 (예: `notion-writer` = 외부 시스템 연동 + 자기 캐시 보유). 가장 핵심인 것 1~2개에 매핑.

---

## 8. 안티패턴

- frontmatter 의 폐지된 키 (`memory: project`) 사용
- 본문에 진입 절차 명시 (CLAUDE.md 자동 로딩과 중복)
- 외부 서비스 위치를 본문 여러 곳에 박음 (페이퍼워크)
- 자기 캐시 형식을 "학습/패턴 누적" 으로 (옛 정책)
- agent-memory 폴더만 만들고 MEMORY.md 없음
- description 이 영어만 / `<example>` 누락 (한국어 자연 호출 매칭 실패)
