claude-config(범용 .claude 마스터) 와 특정 프로젝트의 `.claude/` 폴더를 양방향으로 동기화하는 반자동 커맨드.

> **왜 필요한가**: 정책/룰 변경은 양쪽에서 일어난다.
> - claude-config 가 최신: 정책 재정의 후 프로젝트로 전파
> - 프로젝트가 최신: 작업 중 발견한 노하우/룰을 claude-config 로 역반영
> - 일부는 양쪽에서 다른 시점에 갱신
>
> 단방향 가정은 위험. 매번 양방향 진단 + 항목별 결정이 필요.

## 인수

`$ARGUMENTS`: 동기화할 프로젝트의 경로 (예: `/c/Users/khwoo0527/Desktop/toy/CrewUp`).

호출 예시: `/SyncClaudeConfig CrewUp` 또는 절대 경로.

## 실행 절차

### 1단계: 양방향 진단

claude-config 와 프로젝트의 `.claude/` 양쪽을 diff 한다.

```bash
diff -rq "claude-config/" "<project>/.claude/" | grep -v "settings.local.json" | grep -v "\.git$"
```

**`settings.local.json` 은 항상 제외** — 로컬 환경 권한, 민감 정보 포함 가능.

### 2단계: 차이 항목 분류

진단 결과를 다음 카테고리로 분류:

| 카테고리 | 의미 | 일반 처리 방향 |
|---|---|---|
| **claude-config 에만 있음** (Only in claude-config) | 정책/룰/도구 신규 추가 — 프로젝트로 전파 | 프로젝트로 cp |
| **프로젝트에만 있음** (Only in project) | 작업 중 생성된 항목 — 종류별 분기 필요 | 분기 판단 (역반영 vs 유지 vs 삭제) |
| **양쪽 다른 내용** (Files differ) | 어느 쪽이 새것인지 / 변경 방향 | 시점/내용 판단 후 결정 |

### 3단계: 항목별 진실 원천 판단

각 차이 항목에 대해 [`verify_before_action`](../memory/feedback_verify_before_action.md) 룰에 따라 추측 금지, 검증 후 진행:

#### 시점 비교 (양쪽 다른 내용)
- `git log -1 --format="%ai" -- <file>` 으로 양쪽 마지막 commit 시점 확인
- commit 메시지에 의도/이유 명시되어 있는지 확인 (정책 변경 vs 옛 잔재 vs 프로젝트 종속 보강)

#### 분기 판단 (프로젝트에만 있음)

[`agent-memory.md`](../rules/workflow/agent-memory.md) 의 분기 룰 적용:

| 항목 성격 | 처리 |
|---|---|
| 다른 프로젝트에도 통하는 기술 노하우 | `rules/tech/{tech}.md` 로 역반영 (claude-config) |
| 다른 프로젝트에도 통하는 워크플로우 룰 | `rules/workflow/*.md` 로 역반영 |
| 다른 프로젝트에도 통하는 사용자 작업 스타일 | `memory/feedback_*.md` 로 역반영 |
| 이 프로젝트만의 진행 상태 | 프로젝트 진실 원천 (ROADMAP/sprint/deploy/CLAUDE.md) 으로 이관 후 삭제 |
| 이 프로젝트만의 코드베이스 컨텍스트 | `agent-memory/{name}/MEMORY.md` 또는 `CLAUDE.md` 에 이관 |
| 옛 정책의 잔재 (정책 변경으로 무의미해진 파일) | 진실 원천 검증 후 단순 삭제 |

#### 진실 원천 검증 (삭제/이관 전 필수)
- 삭제하려는 정보가 정말 다른 곳에 있는지 grep/Read 로 확인
- "있을 가능성 높음" 같은 추측 금지 — 실제 파일/라인 명시
- 검증 결과를 사용자에게 보고 (예: "deploy.md L40 에 있음" / "어디에도 없음")

### 4단계: 항목별 보고 + 사용자 컨펌

각 차이 항목에 대해 다음 형식으로 보고:

```
📋 동기화 검토 결과

🟢 단순 동기화 (claude-config → 프로젝트):
  - {파일1} (정책/룰 신규)
  - {파일2} (옛 정책 갱신)

🔵 역반영 (프로젝트 → claude-config):
  - {파일/내용}: 다른 프로젝트에도 통하는 노하우 → rules/tech/{tech}.md 보강
    추천: 보강 / 이유: {이유}

🟠 이관 후 삭제:
  - {프로젝트만 있는 파일}:
    - {정보 A} → {진실 원천} 에 이미 있음 ✅
    - {정보 B} → {이관 위치} 로 이동 필요
    - 이관 후 삭제 추천 / 이유: {이유}

🟡 보류 (결정 어려움 — 사용자 판단 필요):
  - {파일/항목}: {이유}

진행하시겠습니까? (Y/N + 항목별 조정 가능)
```

[`recommend_with_reason`](../memory/feedback_recommend_with_reason.md) 룰 적용 — 각 항목에 추천 + 이유 명시.

[`blanket_authorization`](../memory/feedback_blanket_authorization.md) 룰 적용 — 위임 신호 받은 경우 항목별 컨펌 생략, 합리적 기본값으로 자동 진행 + 사후 보고.

### 5단계: 실행

사용자 컨펌 결과 따라:
- 단순 동기화: `cp -f claude-config/<path> <project>/.claude/<path>` 또는 역방향
- 신규 폴더 동기화: `cp -rf` 폴더 단위
- 이관: 사용자 컨펌받은 위치에 Edit/Write
- 삭제: `rm` (이관 검증 통과한 경우만)

**`settings.local.json` 절대 덮어쓰지 않는다** (각 머신 로컬 권한 보존).

### 6단계: 양쪽 commit + push

두 repo 각각:
1. `git status` 확인
2. `git add -A`
3. `git commit -m "{변경 요약}"` — 의도와 이유 명시
4. `git push origin <branch>`

> claude-config 가 변경되면 claude-config 도 push 필요. 프로젝트가 변경되면 프로젝트 도 push 필요. **양쪽 모두 누락 없도록 확인**.

### 7단계: 최종 검증

```bash
diff -rq "claude-config/" "<project>/.claude/" | grep -v "settings.local.json" | grep -v "\.git$"
```

차이 0 (또는 의도된 보류 항목만 남음) 확인 후 보고:

```
✅ 동기화 완료
- claude-config commit: {hash}
- 프로젝트 commit: {hash}
- 보류 항목: {목록 — 다음 동기화 시 다룸}
- 다음 권장 시점: {추천}
```

## 동작 원칙

- **단방향 가정 금지** — 매번 양방향 진단부터 시작.
- **추측 금지** — 모든 삭제/이관 결정 전 진실 원천 검증.
- **항목별 결정** — 일괄 처리 X, 각 차이 항목마다 분기 판단.
- **이관 우선 → 삭제** — 정보 손실 위험 0 확인 후 삭제.
- **양쪽 push 누락 금지** — claude-config 변경되면 그쪽도 commit + push.
- **보류 항목 명시** — 결정 어려우면 일단 제외하고 다른 거 먼저, 사용자에게 명확히 보고.

## 주의사항

- **`settings.local.json` 절대 동기화하지 않는다** — 민감 정보(PGPASSWORD, 토큰, 경로) 포함 가능. 양방향 cp 대상에서 명시적 제외.
- **공식 진행 상태 (ROADMAP/sprint/deploy)** 는 agent-memory 가 아닌 진실 원천에 있어야 — agent-memory 에서 발견되면 진실 원천 위치 갱신 후 agent-memory 에선 제거.
- **frontmatter 의 `memory:` 같은 폐지된 메타데이터** 발견 시 제거 권고 ([`agent-memory.md`](../rules/workflow/agent-memory.md) 참조).
- **사용자 컨펌 없이 자동 기록 금지** — 위임 신호 외에는 항목마다 컨펌. 단 명시적 위임 시 자동 진행 + 사후 보고.
- **양쪽이 모두 변경된 항목** (둘 다 다른 방향 변경) 발견 시: 자동 판단 금지, 사용자에게 양쪽 내용 보여주고 결정 받기.

## 호출 시점 권장

다음 시점에 사용자가 명시 호출:
- claude-config 의 정책/룰 변경 후 (다른 프로젝트로 전파 필요할 때)
- 프로젝트 작업 중 범용 노하우/룰 발견 후 (claude-config 에 누적할 때)
- 새 프로젝트로 .claude 복사 후 처음 동기화할 때
- 한동안 동기화 안 한 프로젝트가 다시 활성화될 때
