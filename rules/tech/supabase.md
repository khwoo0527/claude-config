---
paths:
  - "**/supabase/**"
  - "**/*.sql"
---

# Supabase 개발 규칙

> 이 파일은 Supabase(PostgreSQL + Auth + Realtime + Storage)를 백엔드로 사용하는 프로젝트에 범용적으로 적용되는 시니어 수준 규칙입니다.
> TypeScript 기본 규칙은 `rules/tech/typescript.md`, React Native 규칙은 `rules/tech/react-native.md`를 참조합니다.
> 프로젝트별 설정, 테이블 스키마, RLS 정책은 `/CLAUDE.md`에 정의합니다.

## 핵심 원칙

- **RLS(Row Level Security) 필수**: 모든 테이블에 RLS를 활성화한다. RLS 없는 테이블은 전 세계에 공개된 것과 같다.
- **타입 안전성**: `supabase gen types`로 생성한 타입을 사용한다. 수동 타입 정의는 스키마와 불일치를 만든다.
- **서버 사이드 검증**: 클라이언트는 신뢰하지 않는다. 중요한 비즈니스 로직은 RLS 정책 또는 Edge Function에서 검증한다.
- **마이그레이션 기반 스키마 관리**: SQL을 직접 실행하지 않는다. 모든 스키마 변경은 마이그레이션 파일로 관리한다.
- **최소 권한 원칙**: 클라이언트에는 publishable 키만 노출한다. secret 키(구 service_role)는 절대 클라이언트에 포함하지 않는다.

---

## API 키 체계 (2025+ — 신규 프로젝트 기준)

| 신 키 | 대체 대상 (legacy) | 용도 | 특징 |
|------|-----------------|------|------|
| `sb_publishable_...` | `anon` JWT 키 | 브라우저/모바일 등 공개 클라이언트 | RLS 적용 대상 |
| `sb_secret_...` | `service_role` JWT 키 | 서버/Edge Function 전용 | **RLS 우회** — 복수 생성·개별 즉시 폐기 가능 |

- **신규 프로젝트에는 legacy `anon`/`service_role` 키가 제공되지 않는다** — 코드 생성 시 신 키 전제.
- ⚠️ **신 키는 JWT가 아니다**: `Authorization: Bearer`로 보내면 실패 — **`apikey` 헤더로 전송**. Edge Function을 신 키로 호출하려면 `verify_jwt = false` 필요.
- 기존 프로젝트는 legacy 키와 병행 동작 (최종 삭제일 미확정) — 마이그레이션 시 Webhook/pg_net 호출부의 헤더도 함께 이전.
- 본 문서에서 "publishable 키" = 구 anon, "secret 키" = 구 service_role 로 읽는다 (legacy 프로젝트).

---

## 프로젝트 구조

```
src/
├── services/
│   └── supabase/
│       ├── client.ts           # Supabase 클라이언트 초기화 (싱글톤)
│       ├── types.ts            # supabase gen types로 생성된 DB 타입
│       ├── auth.ts             # 인증 관련 함수
│       ├── database.ts         # DB 쿼리 함수 (테이블별 또는 기능별)
│       ├── realtime.ts         # Realtime 구독 함수
│       ├── storage.ts          # Storage 파일 업로드/다운로드
│       └── index.ts            # re-export
├── features/
│   ├── auth/
│   │   ├── hooks/
│   │   │   └── useAuth.ts      # 인증 훅 (supabase/auth.ts 사용)
│   │   └── ...
│   └── tasks/
│       ├── hooks/
│       │   └── useTasks.ts     # react-query + supabase/database.ts
│       └── ...
supabase/
├── migrations/                 # 스키마 마이그레이션 파일
│   ├── 20260326000000_create_teams.sql
│   ├── 20260326000001_create_members.sql
│   └── ...
├── seed.sql                    # 초기 데이터 (개발용)
├── config.toml                 # 로컬 개발 설정
└── functions/                  # Edge Functions (서버 사이드 로직)
    └── ...
```

### 구조 규칙

- Supabase 클라이언트는 **단일 파일**(`client.ts`)에서 초기화하고 export
- DB 쿼리 함수는 `services/supabase/`에 모아두고, 컴포넌트에서 직접 쿼리 작성 금지
- 컴포넌트 → 커스텀 훅 → DB 쿼리 함수 → Supabase 클라이언트 순서로 호출
- Edge Function은 `supabase/functions/`에 배치

---

## 네이밍 컨벤션

### 데이터베이스 (PostgreSQL)

| 종류 | 컨벤션 | 예시 |
|------|--------|------|
| 테이블명 | snake_case, 복수형 | `teams`, `team_members`, `tasks` |
| 컬럼명 | snake_case | `created_at`, `team_id`, `max_capacity` |
| 외래 키 | `{참조테이블_단수}_id` | `team_id`, `user_id`, `task_id` |
| 인덱스 | `idx_{테이블}_{컬럼}` | `idx_tasks_team_id`, `idx_members_user_id` |
| RLS 정책 | `{동작}_{대상}_{조건}` | `select_own_teams`, `insert_admin_only` |
| Enum 타입 | snake_case | `user_role`, `task_status` |
| 함수/트리거 | snake_case, 동사로 시작 | `handle_new_user`, `update_member_count` |

### TypeScript 코드

| 종류 | 컨벤션 | 예시 |
|------|--------|------|
| 쿼리 함수 | `fetch`, `create`, `update`, `delete` + 대상 | `fetchTeamMembers`, `createTask`, `updateStatus` |
| 구독 함수 | `subscribeTo` + 대상 | `subscribeToTaskChanges`, `subscribeToNotifications` |
| 훅 | `use` + 대상 | `useTeamMembers`, `useRealtimeTasks` |
| 타입 (DB 행) | PascalCase, `Row` 접미어 생략 | `Team`, `Member`, `Task` |
| 타입 (삽입) | `Create` + 대상 + `Input` | `CreateTeamInput`, `CreateTaskInput` |
| 타입 (수정) | `Update` + 대상 + `Input` | `UpdateTeamInput`, `UpdateTaskInput` |

---

## 클라이언트 초기화

```typescript
// services/supabase/client.ts
import { createClient } from '@supabase/supabase-js';
import { Platform } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import type { Database } from './types';

const supabaseUrl = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey, {
  auth: {
    // ⚠️ 웹에서는 undefined (localStorage 사용), 네이티브에서만 AsyncStorage
    storage: Platform.OS === 'web' ? undefined : AsyncStorage,
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: Platform.OS === 'web',  // 웹에서는 true (OAuth 콜백 URL 감지)
  },
});
```

### 클라이언트 규칙

- 클라이언트 인스턴스는 **앱 전체에서 1개만** 생성 (싱글톤)
- `supabaseUrl`과 `supabaseAnonKey`는 **환경 변수**로 관리
- `service_role` 키는 클라이언트 코드에 **절대 포함하지 않는다**
- `Database` 제네릭 타입을 반드시 전달하여 타입 안전성 확보

### 🔴 플랫폼별 Auth 설정 (실전 함정)

| 설정 | 웹 | React Native |
|------|-----|-------------|
| `storage` | `undefined` (localStorage) | `AsyncStorage` |
| `detectSessionInUrl` | `true` | `false` |

**함정**: `storage: AsyncStorage`를 웹에서도 사용하면 **새로고침(F5) 시 세션이 유실**된다. AsyncStorage는 RN 전용이며 웹에서 정상 동작하지 않는다. 반드시 `Platform.OS`로 분기한다.

---

## 타입 생성 & 활용

```bash
# DB 타입 자동 생성 (스키마 변경 시마다 실행)
npx supabase gen types typescript --local > src/services/supabase/types.ts
```

```typescript
// 생성된 타입에서 테이블 타입 추출
import type { Database } from './types';

// Row 타입 (SELECT 결과)
type Team = Database['public']['Tables']['teams']['Row'];
type Member = Database['public']['Tables']['team_members']['Row'];
type Task = Database['public']['Tables']['tasks']['Row'];

// Insert 타입 (INSERT 입력)
type CreateTeamInput = Database['public']['Tables']['teams']['Insert'];

// Update 타입 (UPDATE 입력)
type UpdateTeamInput = Database['public']['Tables']['teams']['Update'];

// 커스텀 타입 (조인 결과 등)
type TaskWithAssignees = Task & {
  assignees: Member[];
  assignee_count: number;
};
```

### 타입 규칙

- DB 타입은 **항상 `supabase gen types`로 생성** — 수동 작성 금지
- 스키마 변경 후 타입 재생성을 잊지 않는다 (CI에서 자동화 권장)
- 조인/뷰 결과 등 생성 타입으로 커버 안 되는 경우만 커스텀 타입 작성
- 쿼리 함수의 반환 타입을 명시하여, 타입 변경 시 영향 범위를 즉시 파악

---

## 데이터베이스 쿼리

### 기본 CRUD

```typescript
// SELECT — 목록 조회
async function fetchTeamTasks(teamId: string): Promise<Task[]> {
  const { data, error } = await supabase
    .from('tasks')
    .select('*')
    .eq('team_id', teamId)
    .order('due_date', { ascending: true });

  if (error) throw new AppError(error.message, 'FETCH_TASKS', '목록을 불러올 수 없습니다.');
  return data;
}

// SELECT — 단건 조회
async function fetchTaskById(taskId: string): Promise<TaskWithAssignees> {
  const { data, error } = await supabase
    .from('tasks')
    .select(`
      *,
      assignees:task_assignees(
        *,
        member:team_members(*)
      )
    `)
    .eq('id', taskId)
    .single();

  if (error) throw new NotFoundError('항목', taskId);
  return data;
}

// INSERT
async function createTask(input: CreateTaskInput): Promise<Task> {
  const { data, error } = await supabase
    .from('tasks')
    .insert(input)
    .select()       // insert 후 생성된 행 반환
    .single();

  if (error) throw new AppError(error.message, 'CREATE_TASK', '생성에 실패했습니다.');
  return data;
}

// UPDATE
async function updateTask(taskId: string, input: UpdateTaskInput): Promise<Task> {
  const { data, error } = await supabase
    .from('tasks')
    .update(input)
    .eq('id', taskId)
    .select()
    .single();

  if (error) throw new AppError(error.message, 'UPDATE_TASK', '수정에 실패했습니다.');
  return data;
}

// DELETE
async function deleteTask(taskId: string): Promise<void> {
  const { error } = await supabase
    .from('tasks')
    .delete()
    .eq('id', taskId);

  if (error) throw new AppError(error.message, 'DELETE_TASK', '삭제에 실패했습니다.');
}
```

### 쿼리 규칙

- **`select('*')` 지양**: 필요한 컬럼만 명시하여 네트워크/메모리 절약 (단, 초기 MVP에서는 `*` 허용)
- **`.single()`**: 단건 조회 시 반드시 사용 — 없으면 배열이 반환됨
- **`.select()` after insert/update**: 생성/수정된 행을 반환받으려면 반드시 추가
- **에러 처리**: `{ data, error }` 구조분해 후 error 체크 필수 — error를 무시하지 않는다
- **필터 체이닝**: `.eq()`, `.in()`, `.gte()`, `.lte()`, `.like()` 등 타입 안전한 필터 사용
- **페이지네이션**: `.range(from, to)` 사용 — 전체 조회 금지 (대용량 테이블)

### 조인 & 관계 쿼리

```typescript
// 관계 데이터 함께 조회 (foreign key 기반)
const { data } = await supabase
  .from('teams')
  .select(`
    id,
    name,
    members:team_members(
      id,
      user_id,
      role,
      profile:profiles(name, avatar_url)
    )
  `)
  .eq('id', teamId)
  .single();

// 집계 (count)
const { count } = await supabase
  .from('team_members')
  .select('*', { count: 'exact', head: true })
  .eq('team_id', teamId);
```

---

## 인증 (Auth)

### 소셜 로그인 (카카오/네이버)

```typescript
// 소셜 로그인 시작
async function signInWithKakao(): Promise<void> {
  const { error } = await supabase.auth.signInWithOAuth({
    provider: 'kakao',
    options: {
      redirectTo: 'your-app-scheme://auth/callback',
      skipBrowserRedirect: true, // React Native에서 필수
    },
  });
  if (error) throw new AppError(error.message, 'AUTH_KAKAO', '카카오 로그인에 실패했습니다.');
}

// 세션 상태 감지 (앱 초기화 시)
supabase.auth.onAuthStateChange((event, session) => {
  switch (event) {
    case 'SIGNED_IN':
      // 로그인 완료 처리
      break;
    case 'SIGNED_OUT':
      // 로그아웃 처리 (화면 전환 등)
      break;
    case 'TOKEN_REFRESHED':
      // 토큰 갱신됨 (보통 자동 처리)
      break;
  }
});
```

### 인증 훅 패턴

```typescript
// features/auth/hooks/useAuth.ts
function useAuth() {
  const [session, setSession] = useState<Session | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // 현재 세션 가져오기
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session);
      setIsLoading(false);
    });

    // 인증 상태 변경 구독
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, session) => setSession(session),
    );

    return () => subscription.unsubscribe();
  }, []);

  return {
    session,
    user: session?.user ?? null,
    isLoading,
    isAuthenticated: !!session,
    signOut: () => supabase.auth.signOut(),
  };
}
```

### 인증 규칙

- 세션은 **Supabase가 자동 관리** — 직접 토큰을 저장/관리하지 않는다
- `onAuthStateChange`로 전역 인증 상태를 감지한다 — polling 하지 않는다
- 로그아웃 시 로컬 상태(react-query 캐시 등)를 초기화한다
- 인증이 필요한 화면은 `_layout.tsx`에서 가드 처리 (react-native.md 참조)
- `user.id`를 RLS 정책의 `auth.uid()`와 매칭하여 데이터 접근 제어

#### 서버측 사용자 확인 3형제 — 선택 기준 (혼용 금지)

| 함수 | 하는 일 | 언제 쓰나 |
|------|--------|----------|
| `getSession()` | 로컬 저장 세션 반환 — **검증 없음** | UI 표시용 힌트만. **서버측 인가 판단에 절대 사용 금지** (위조 가능) |
| `getClaims()` | JWT를 JWKS로 **로컬 검증** (네트워크 0회 — asymmetric 키 프로젝트) | 서버측 요청 인가의 기본 — 빠르고 검증됨 |
| `getUser()` | Auth 서버에 조회 — **최신 사용자 상태 보장** | 로그아웃/밴/세션 무효화까지 확인해야 하는 민감 작업 (결제, 권한 변경) |

> 신규 프로젝트는 asymmetric JWT + JWKS 가 기본 (2025-05+). SSR 은 `@supabase/ssr` 의 `getAll`/`setAll` 쿠키 인터페이스 사용 — 구 `get`/`set`/`remove` 는 deprecated.

```typescript
// ❌ Bad — 서버측 인가를 getSession으로 판단 (로컬 세션은 위조 가능 — 권한 상승 취약점)
const { data: { session } } = await supabase.auth.getSession();
if (session?.user) { await deleteResource(id); }

// ✅ Good — 검증된 클레임 기반 (일반 요청) / 민감 작업은 getUser로 최신 상태 확인
const { data: claims, error } = await supabase.auth.getClaims();
if (error || !claims) { return unauthorized(); }
await deleteResource(id, claims.sub);
```

---

## Row Level Security (RLS)

### RLS 정책 작성 패턴

```sql
-- 모든 테이블에 RLS 활성화 (필수!)
ALTER TABLE teams ENABLE ROW LEVEL SECURITY;

-- SELECT: 팀 멤버만 조회 가능
CREATE POLICY "select_team_members_only"
  ON tasks FOR SELECT
  USING (
    team_id IN (
      SELECT team_id FROM team_members
      WHERE user_id = auth.uid()
    )
  );

-- INSERT: 관리자만 생성 가능
CREATE POLICY "insert_admin_only"
  ON tasks FOR INSERT
  WITH CHECK (
    EXISTS (
      SELECT 1 FROM team_members
      WHERE team_id = NEW.team_id
        AND user_id = auth.uid()
        AND role IN ('admin', 'owner')
    )
  );

-- UPDATE: 본인이 생성한 것만 수정 가능
CREATE POLICY "update_own_tasks"
  ON tasks FOR UPDATE
  USING (created_by = auth.uid())
  WITH CHECK (created_by = auth.uid());

-- DELETE: 소유자만 삭제 가능
CREATE POLICY "delete_owner_only"
  ON tasks FOR DELETE
  USING (
    EXISTS (
      SELECT 1 FROM team_members
      WHERE team_id = tasks.team_id
        AND user_id = auth.uid()
        AND role = 'owner'
    )
  );
```

### RLS 규칙

- **모든 테이블**에 RLS를 활성화한다 — 예외 없음
- 정책은 **최소 권한 원칙**: 필요한 최소한의 데이터만 접근 허용
- `auth.uid()`로 현재 사용자를 식별한다
- `USING`은 기존 행 접근 (SELECT, UPDATE, DELETE), `WITH CHECK`는 새 행 검증 (INSERT, UPDATE)
- 복잡한 정책은 **PostgreSQL 함수**로 분리하여 재사용
- RLS 정책에 **성능 주의**: 정책이 참조하는 컬럼에 인덱스 필수
- **🔑 `auth.uid()` 는 반드시 `(select auth.uid())` 로 감싼다** — 공식 확정 패턴 (아래)

```sql
-- ❌ 행마다 auth.uid() 재평가 — 10만 행 스캔 시 10만 번 함수 호출
CREATE POLICY "own_rows" ON tasks USING (user_id = auth.uid());

-- ✅ (select ...) 랩핑 → 옵티마이저가 initPlan으로 문장당 1회만 평가 (수십 배 차이)
CREATE POLICY "own_rows" ON tasks USING (user_id = (select auth.uid()));
```
> Dashboard Advisors → Performance 의 `auth_rls_initplan` lint 가 미적용 정책을 자동 탐지한다. 단 행 데이터에 의존하지 않는 함수에만 적용 가능.

### 🔴 RLS 실전 함정 & 지뢰

| 증상 | 원인 | 해결 | 오답 (하지 말 것) |
|------|------|------|-------------------|
| 중첩 조인 쿼리 시 PostgREST 500 에러 | RLS 정책의 서브쿼리가 **같은 테이블을 참조**(재귀) | 정책에 `user_id = auth.uid()` 등 단순 조건 추가, 또는 쿼리를 2단계로 분리 | `SECURITY DEFINER` 함수로 RLS 우회 |
| 임베디드 집계(`table(count)`) + RLS = 500 | PostgREST가 RLS 정책을 적용하면서 집계 서브쿼리와 충돌 | 집계를 별도 쿼리로 분리 (FK 컬럼만 SELECT 후 코드에서 카운트) | count를 아예 제거하거나 RLS 비활성화 |
| INSERT 후 데이터가 조회 안 됨 | 상태 컬럼(status 등)에 **기본값이 없어** NULL 저장 → `.eq('status', 'active')` 필터에 안 걸림 | DB 컬럼에 `DEFAULT 'active'` 설정 + INSERT 시 명시적으로 값 지정 | 필터를 제거하거나 NULL 체크 추가 |
| 다른 테이블 조인 시 빈 배열 반환 | 참조하는 테이블의 RLS 정책이 현재 사용자 접근을 차단 | 참조 테이블의 RLS 정책 확인 및 필요한 SELECT 정책 추가 | 모든 정책을 `USING (true)`로 변경 |
| RLS 켰더니 모든 요청이 빈 결과/거부 | RLS 활성 + **정책 0개 = 전체 차단**이 기본 동작 | 활성화와 동시에 최소 SELECT 정책부터 작성 (테이블 생성 체크리스트 참조) | RLS를 다시 꺼버림 (anon 키로 전체 노출) |
| UPDATE가 에러 없이 0 rows — 조용히 실패 | UPDATE 는 대상 행을 먼저 **읽어야** 하는데 대응 SELECT 정책이 없음 | UPDATE 정책과 함께 같은 조건의 SELECT 정책 확인/추가 | UPDATE 정책의 USING 을 true 로 |
| Realtime(Postgres Changes) 이벤트가 안 옴 — 에러도 없음 | RLS 가 replication 컨텍스트에서 이벤트를 **무음 필터링** (auth 컨텍스트 부재) | 구독 대상 테이블의 RLS 정책을 realtime 관점에서 검증 + private 채널/브로드캐스트 전환 검토 | RLS 를 끄고 구독 (전체 노출) |
| 신 API 키로 요청 시 401 — 키는 분명 맞음 | `sb_publishable_`/`sb_secret_` 키는 **JWT가 아님** → `Authorization: Bearer` 로 보내면 실패 | **`apikey` 헤더로 전송**, Edge Function 은 `verify_jwt = false` | legacy anon 키로 되돌리기 (신규 프로젝트엔 없음) |
| 컴파일은 통과하는데 런타임에 필드가 undefined | 스키마 변경(마이그레이션) 후 **타입 미재생성** — 생성 타입과 실제 DB 불일치 | 마이그레이션 직후 `supabase gen types` 재실행 (체크리스트 항목) | 해당 필드에 `as any`/옵셔널 처리로 침묵 |
| Storage 업로드/다운로드만 403 — DB는 정상 | Storage 는 **`storage.objects` 테이블에 별도 RLS 정책** 필요 (버킷 만들었다고 끝 아님) | 버킷 생성 시 objects 정책(SELECT/INSERT 등)을 함께 작성 | 버킷을 public 으로 전환 (전체 파일 노출) |

#### 재귀 RLS 정책 상세 설명

```sql
-- ❌ 위험: 멤버십 테이블의 SELECT 정책에서 같은 테이블을 다시 참조 (재귀)
CREATE POLICY "select_same_group"
  ON group_members FOR SELECT
  USING (
    group_id IN (
      SELECT group_id FROM group_members  -- ← 같은 테이블! 재귀 서브쿼리!
      WHERE user_id = auth.uid()
    )
  );

-- ✅ 안전: 자기 자신의 행만 직접 조건으로 확인
CREATE POLICY "select_own_memberships"
  ON group_members FOR SELECT
  USING (user_id = auth.uid());

-- ✅ 다른 멤버도 보려면: 별도 정책 추가 (부모 테이블 참조, 같은 테이블 재참조 아님)
CREATE POLICY "select_public_group_members"
  ON group_members FOR SELECT
  USING (
    group_id IN (
      SELECT id FROM groups WHERE is_public = true
    )
    OR user_id = auth.uid()
  );
```

#### 2단계 쿼리 패턴 (중첩 조인 대안)

```typescript
// ❌ 위험: RLS가 있는 테이블을 통해 중첩 조인
const { data } = await supabase
  .from('group_members')
  .select('group:groups(*, category:categories(*))')
  .eq('user_id', userId);

// ✅ 안전: 2단계 쿼리로 분리
// Step 1: FK 목록만 가져오기
const { data: memberships } = await supabase
  .from('group_members')
  .select('group_id')
  .eq('user_id', userId)
  .eq('status', 'active');

const groupIds = memberships?.map(m => m.group_id) ?? [];

// Step 2: 부모 테이블에서 직접 조회 (RLS 충돌 없음)
const { data: groups } = await supabase
  .from('groups')
  .select('*, category:categories(*)')
  .in('id', groupIds);
```

---

## Realtime 구독

```typescript
// 테이블 변경 구독
function subscribeToTaskChanges(teamId: string, callback: (task: Task) => void) {
  const channel = supabase
    .channel(`tasks:${teamId}`)
    .on(
      'postgres_changes',
      {
        event: '*',           // INSERT, UPDATE, DELETE 모두
        schema: 'public',
        table: 'tasks',
        filter: `team_id=eq.${teamId}`,
      },
      (payload) => {
        callback(payload.new as Task);
      },
    )
    .subscribe();

  // cleanup 함수 반환
  return () => {
    supabase.removeChannel(channel);
  };
}

// React 훅으로 감싸기
function useRealtimeTasks(teamId: string) {
  const queryClient = useQueryClient();

  useEffect(() => {
    const unsubscribe = subscribeToTaskChanges(teamId, () => {
      // 변경 감지 시 react-query 캐시 무효화 → 자동 refetch
      queryClient.invalidateQueries({ queryKey: ['tasks', teamId] });
    });

    return unsubscribe;
  }, [teamId, queryClient]);
}
```

### Realtime 규칙

- 구독 시 반드시 **cleanup(unsubscribe)** 처리 — 메모리 릭 방지
- `filter`를 사용하여 **필요한 데이터만** 구독 — 전체 테이블 구독 금지
- 대량 업데이트가 예상되면 **디바운스** 적용
- Realtime은 **UI 갱신 트리거**로 사용 — 데이터 자체는 react-query가 관리
- 채널 이름에 고유 식별자 포함 (`tasks:${teamId}`)

---

## 마이그레이션

```bash
# 마이그레이션 생성
npx supabase migration new create_teams

# 마이그레이션 적용 (로컬 — 도커 사용 시)
npx supabase db reset

# 마이그레이션 적용 (Cloud — 운영 환경 또는 도커 미사용 시)
npx supabase db push

# 마이그레이션 상태 확인
npx supabase migration list
```

> **로컬 vs Cloud 분기**: `db reset` 은 로컬 도커 컨테이너 초기화 + 마이그레이션 + 시드 적용. `db push` 는 로컬 마이그레이션 파일을 Cloud 에 적용 (도커 불필요). 회사 환경 등 도커 사용 불가 시 또는 Cloud 직접 운영 시 `db push` 사용.

```sql
-- supabase/migrations/20260326000000_create_teams.sql

-- 테이블 생성
CREATE TABLE teams (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  category TEXT NOT NULL DEFAULT 'general',
  owner_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  max_capacity INTEGER NOT NULL DEFAULT 50,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 인덱스
CREATE INDEX idx_teams_owner_id ON teams(owner_id);
CREATE INDEX idx_teams_category ON teams(category);

-- RLS 활성화
ALTER TABLE teams ENABLE ROW LEVEL SECURITY;

-- RLS 정책
CREATE POLICY "select_public_teams" ON teams
  FOR SELECT USING (true);  -- 공개 목록은 누구나 조회 가능

CREATE POLICY "insert_authenticated" ON teams
  FOR INSERT WITH CHECK (auth.uid() = owner_id);

-- updated_at 자동 갱신 트리거
CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON teams
  FOR EACH ROW
  EXECUTE FUNCTION moddatetime(updated_at);
```

### 마이그레이션 규칙

- 모든 스키마 변경은 **마이그레이션 파일**로 관리 — 대시보드에서 직접 SQL 실행 금지 (프로덕션)
- 파일명: `YYYYMMDDHHMMSS_{설명}.sql` — Supabase CLI가 자동 생성
- **되돌릴 수 없는 변경**(컬럼 삭제, 타입 변경)은 별도 마이그레이션으로 분리하고 주석으로 경고
- 새 테이블 생성 시 반드시 포함: RLS 활성화 + 기본 정책 + 인덱스
- `ON DELETE CASCADE` vs `SET NULL` 결정을 명확히 — 데이터 삭제 영향 분석 필수
- 시드 데이터(`seed.sql`)는 개발 환경 전용 — 프로덕션에 적용하지 않는다

---

## Storage (파일 업로드)

```typescript
// 프로필 이미지 업로드
async function uploadProfileImage(userId: string, file: Blob): Promise<string> {
  const filePath = `profiles/${userId}/${Date.now()}.jpg`;

  const { error } = await supabase.storage
    .from('avatars')
    .upload(filePath, file, {
      contentType: 'image/jpeg',
      upsert: true,
    });

  if (error) throw new AppError(error.message, 'UPLOAD', '이미지 업로드에 실패했습니다.');

  const { data } = supabase.storage.from('avatars').getPublicUrl(filePath);
  return data.publicUrl;
}
```

### Storage 규칙

- 버킷은 **용도별** 분리 (`avatars`, `uploads`, `documents`)
- 파일 경로에 **사용자 ID** 포함 — RLS 정책으로 본인 파일만 접근
- 업로드 시 **파일 크기, MIME 타입 검증** — Storage 정책에서 설정
- 이미지는 업로드 전 **리사이즈** (원본 4K 금지)
- Public 버킷과 Private 버킷을 구분 — 민감 파일은 Private + signed URL

---

## Edge Functions (서버 사이드 로직)

```typescript
// supabase/functions/send-notification/index.ts
// ❌ 구식 (2022 스타일): import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
// ✅ 현행: 내장 Deno.serve + npm: specifier (의존성은 supabase/functions/deno.json 에 선언)
import { createClient } from 'npm:@supabase/supabase-js@2';

Deno.serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SECRET_KEY')!, // 서버 전용 secret 키 (구 service_role) — 클라이언트 절대 금지
  );

  const { teamId, message } = await req.json();

  // 팀 멤버 조회
  const { data: members } = await supabase
    .from('team_members')
    .select('push_token')
    .eq('team_id', teamId)
    .not('push_token', 'is', null);

  // 푸시 알림 발송 로직
  // ...

  return new Response(JSON.stringify({ sent: members?.length ?? 0 }), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

### Edge Function 규칙

- **`Deno.serve()` 가 표준** — `deno.land/std` 의 `serve` import 는 폐기된 스타일 (구식 코드 생성 금지)
- 의존성: npm 패키지는 `npm:패키지명` / JSR 은 `jsr:@scope/패키지` specifier — 선언은 `supabase/functions/deno.json` 권장 (import map 보다 우선)
- secret 키(구 service_role)는 **Edge Function/서버에서만** 사용 (클라이언트 금지)
- 신 API 키(`sb_secret_...`)로 함수를 호출할 땐 함수 설정에 `verify_jwt = false` 필요 (신 키는 JWT가 아님 — 아래 "API 키 체계" 참조)
- 클라이언트에서 직접 처리하면 안 되는 로직: 알림 발송, 결제 처리, 관리자 작업
- 입력값 검증 필수 (zod 등 — `npm:zod`)
- 에러 응답은 일관된 형식: `{ error: { code, message } }`
- 타임아웃 고려 (Supabase Edge Function 기본 타임아웃: 60초)

---

## 안티패턴 (하지 말 것)

### 보안 관련

| 안티패턴 | 실패 결과 | 대안 |
|---------|----------|------|
| `service_role` 키를 클라이언트에 포함 | 모든 RLS 우회 → 전체 DB가 브라우저 DevTools에서 노출 | `anon` 키만 클라이언트에, `service_role`은 서버에서만 |
| RLS를 비활성화한 채 배포 | 누구든 URL 조작으로 모든 데이터 조회/수정/삭제 가능 | 모든 테이블에 RLS 필수 |
| `SECURITY DEFINER` 함수로 RLS 우회 | RLS 보안 모델 전체가 무력화, 어떤 사용자든 모든 데이터 접근 | RLS 정책 자체를 올바르게 수정 |
| 토큰을 AsyncStorage에 저장 | 루팅된 기기에서 토큰 탈취 가능 | `expo-secure-store` 사용 |

### 쿼리 관련

| 안티패턴 | 실패 결과 | 대안 |
|---------|----------|------|
| RLS 테이블에서 임베디드 집계 `related_table(count)` | PostgREST 500 에러 (RLS + 집계 서브쿼리 충돌) | 별도 count 쿼리로 분리 |
| RLS 테이블 통해 중첩 조인 | PostgREST 500 에러 (RLS 재귀 서브쿼리) | 2단계 쿼리로 분리 |
| status 컬럼에 DB 기본값 미설정 | INSERT 시 NULL → `.eq('status', 'active')` 필터에 안 걸림 → 데이터 "증발" | `DEFAULT 'active'` 설정 + INSERT 시 명시 |
| `.single()` 없이 단건 조회 | 배열이 반환되어 `data.name` 접근 시 undefined | 단건은 `.single()` 필수 |
| error를 체크하지 않음 | 조용한 실패 → 사용자에게 빈 화면 표시 → 디버깅 지옥 | `if (error) throw` 패턴 |
| 인덱스 없이 자주 쓰는 필터 컬럼 | 대용량 테이블에서 풀 스캔 → 타임아웃 | 자주 쿼리하는 컬럼에 인덱스 |
| 전체 테이블 조회 (페이지네이션 없음) | 수만 행 → 메모리 폭발, 네트워크 타임아웃 | `.range()` 또는 커서 페이지네이션 |

### AI가 흔히 생성하는 실수

| 실수 | 실패 결과 | 수정 방법 |
|------|----------|----------|
| `storage: AsyncStorage` 웹에서도 사용 | 웹 새로고침 시 세션 유실 → 로그인 반복 | `Platform.OS === 'web' ? undefined : AsyncStorage` |
| `detectSessionInUrl: false` 웹에서 설정 | OAuth 콜백 처리 안 됨 → 소셜 로그인 실패 | `Platform.OS === 'web'`으로 분기 |
| RLS 테이블을 통해 부모 테이블 중첩 조인 | RLS 재귀로 PostgREST 500 | 2단계 쿼리로 분리 |
| `createClient`를 컴포넌트마다 호출 | 다중 연결 → 세션 불일치 | 싱글톤 `client.ts`에서 한번만 생성 |
| 타입을 수동 정의 | 스키마 변경 시 타입 불일치 → 런타임 에러 | `supabase gen types`로 자동 생성 |
| RLS 정책 없이 테이블 생성 | 전체 데이터 노출 | 마이그레이션에 RLS 활성화 + 정책 포함 |
| insert 시 status 등 필수 필터 컬럼 누락 | NULL 저장 → 조회 누락 | INSERT에 모든 필터 대상 컬럼 명시 |
| `onAuthStateChange` cleanup 누락 | 메모리 릭, 이중 이벤트 처리 | `return () => subscription.unsubscribe()` |
| insert 후 `select()` 빠뜨림 | 생성된 행 데이터 없음 → UI 갱신 불가 | `.insert(data).select().single()` |

---

## 에러/예외 처리

```typescript
// Supabase 에러 래퍼
function handleSupabaseError(error: PostgrestError, context: string): never {
  // 일반적인 Supabase 에러 코드 매핑
  const errorMap: Record<string, string> = {
    '23505': '이미 존재하는 데이터입니다.',     // unique violation
    '23503': '참조하는 데이터가 존재하지 않습니다.', // foreign key violation
    '42501': '권한이 없습니다.',               // RLS policy violation
    'PGRST116': '데이터를 찾을 수 없습니다.',    // .single() no rows
  };

  const userMessage = errorMap[error.code] ?? '처리 중 문제가 발생했습니다.';

  throw new AppError(
    `[${context}] ${error.message} (code: ${error.code})`,
    error.code,
    userMessage,
  );
}

// 사용
async function fetchTeam(teamId: string): Promise<Team> {
  const { data, error } = await supabase
    .from('teams')
    .select('*')
    .eq('id', teamId)
    .single();

  if (error) handleSupabaseError(error, 'fetchTeam');
  return data;
}
```

### 에러 처리 규칙

- Supabase의 `{ data, error }` 패턴을 항상 처리 — error 무시 금지
- PostgreSQL 에러 코드를 사용자 친화적 메시지로 변환
- 네트워크 에러 (오프라인)와 서버 에러를 구분
- react-query의 `onError`에서 일괄 에러 토스트 처리

---

## 보안

- **publishable 키만 클라이언트에**: secret 키(구 service_role)는 Edge Function/서버에서만
- **환경 변수**: `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY` (Expo 공개 접두어 — 신 키 프로젝트는 PUBLISHABLE_KEY)
- **RLS 정책 테스트**: 마이그레이션 작성 후 다른 사용자로 접근 시도하여 검증
- **SQL Injection**: Supabase JS 클라이언트가 자동 파라미터화 — `.rpc()` 사용 시에도 안전
- **민감 데이터 암호화**: 비밀번호, 결제 정보 등은 DB에 평문 저장 금지
- **API Rate Limiting**: Supabase 대시보드에서 설정 가능 — 남용 방지

```typescript
// ❌ Bad — private 버킷 파일을 public URL로 노출 시도 (혹은 버킷을 public으로 전환)
const { data } = supabase.storage.from('private-docs').getPublicUrl(path);
// private 버킷이면 접근 불가, "해결"한답시고 버킷 public 전환 = 전체 파일 노출

// ✅ Good — 만료 시간 있는 signed URL (접근 권한은 RLS/정책이 판단)
const { data, error } = await supabase.storage
  .from('private-docs')
  .createSignedUrl(path, 60 * 10); // 10분 유효
```

```sql
-- ✅ RLS 침투 테스트 — 다른 사용자 시점으로 정책을 직접 검증 (배포 전 필수)
BEGIN;
SET LOCAL role = 'authenticated';
SET LOCAL request.jwt.claims = '{"sub": "다른-사용자-uuid"}';
SELECT * FROM tasks;  -- ❌ 이 결과에 남의 행이 보이면 정책 결함
ROLLBACK;
```

---

## 성능 최적화

### 쿼리 성능
- 자주 필터/정렬하는 컬럼에 **인덱스** 생성 — RLS 정책이 참조하는 컬럼 포함
- `select('*')` 대신 **필요한 컬럼만** 선택
- 대용량 조회는 **페이지네이션** 필수 (`.range()`)
- 관계 쿼리에서 **N+1 문제** 주의 — Supabase 관계 쿼리(`select('*, related(*)')`)는 자동으로 조인하므로 보통 안전

```sql
-- ✅ 인덱스: 필터 컬럼 + RLS 참조 컬럼. 효과는 추측 말고 EXPLAIN으로 측정
CREATE INDEX idx_tasks_user_id ON tasks (user_id);          -- RLS의 user_id = (select auth.uid())
CREATE INDEX idx_tasks_status_created ON tasks (status, created_at DESC); -- 목록 필터+정렬

EXPLAIN ANALYZE SELECT * FROM tasks WHERE user_id = '...' AND status = 'active';
-- Seq Scan이 나오면 인덱스 미사용 — 정책/쿼리 조건 재점검
```

```typescript
// ❌ Bad — 전체 로드 후 클라이언트에서 자르기 (10만 행이면 그대로 10만 행 전송)
const { data } = await supabase.from('tasks').select('*');
const page = data?.slice(0, 20);

// ✅ Good — 서버에서 페이지 단위로 (count는 필요할 때만)
const { data, count } = await supabase
  .from('tasks')
  .select('id, title, status, created_at', { count: 'estimated' })
  .order('created_at', { ascending: false })
  .range(0, 19);
```

### Realtime 성능
- 구독은 **필요한 테이블/필터만** — 전체 구독 금지
- 대량 업데이트 시 **디바운스** 적용
- 컴포넌트 언마운트 시 **반드시 구독 해제**

```typescript
// ❌ Bad — 이벤트마다 즉시 재조회 (연속 업데이트 100건 = 재조회 100번)
channel.on('postgres_changes', { event: '*', schema: 'public', table: 'tasks' }, () => {
  refetch();
});

// ✅ Good — 디바운스로 마지막 이벤트 후 1회만
const debouncedRefetch = debounce(refetch, 500);
channel.on('postgres_changes', { event: '*', schema: 'public', table: 'tasks' }, () => {
  debouncedRefetch();
});
```

### 연결 관리
- Supabase 클라이언트는 내부적으로 커넥션 풀 관리 — 직접 관리 불필요
- 앱이 백그라운드에 갈 때 Realtime 채널 정리 고려
- 네트워크 복구 시 자동 재연결 (Supabase 내장)

---

## 테스트

```typescript
// Supabase 클라이언트 모킹
jest.mock('@/services/supabase/client', () => ({
  supabase: {
    from: jest.fn().mockReturnThis(),
    select: jest.fn().mockReturnThis(),
    eq: jest.fn().mockReturnThis(),
    single: jest.fn().mockResolvedValue({
      data: mockTeam,
      error: null,
    }),
    auth: {
      getSession: jest.fn().mockResolvedValue({
        data: { session: mockSession },
        error: null,
      }),
    },
  },
}));

// 쿼리 함수 테스트
describe('fetchTeam', () => {
  it('팀 정보를 정상적으로 반환한다', async () => {
    const team = await fetchTeam('team-123');
    expect(team.name).toBe('Test Team');
  });

  it('존재하지 않는 항목은 에러를 던진다', async () => {
    mockSupabaseError('PGRST116');
    await expect(fetchTeam('invalid')).rejects.toThrow(AppError);
  });
});
```

### 테스트 규칙

- Supabase 클라이언트는 **모킹** 처리 — 실제 DB 호출 금지 (유닛 테스트)
- RLS 정책 테스트는 **로컬 Supabase**에서 다른 사용자로 접근하여 검증
- 마이그레이션 테스트: `supabase db reset`으로 처음부터 적용 확인
- Edge Function 테스트: Deno 테스트 러너 사용

---

## 문서화 규칙

> TypeScript 기본 문서화(`typescript.md`)를 따른다.

### 마이그레이션 파일

```sql
-- Good: 마이그레이션에 목적과 주의사항 명시
-- 팀 작업 테이블 생성
-- 참고: task_assignees와 1:N 관계, cascade 삭제
CREATE TABLE tasks ( ... );

-- Bad: 주석 없는 마이그레이션
CREATE TABLE tasks ( ... );
```

### 쿼리 함수

```typescript
// Good: 복잡한 쿼리에 비즈니스 규칙 설명
/**
 * 사용자가 참여 가능한 작업 목록 조회.
 * 이미 마감된 항목은 제외하고, 정원 미달인 항목만 반환한다.
 */
async function fetchAvailableTasks(teamId: string): Promise<Task[]> { ... }

// Bad: 단순 CRUD에 불필요한 주석
/** 팀 조회 */
async function fetchTeam(id: string): Promise<Team> { ... }
```

---

## 빌드 & 설정

### 환경 변수

```
# .env.local (개발)
EXPO_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
EXPO_PUBLIC_SUPABASE_ANON_KEY=eyJ...local-anon-key

# .env.production (프로덕션)
EXPO_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=eyJ...production-anon-key
```

- `EXPO_PUBLIC_` 접두어: 클라이언트에 노출되는 환경 변수 (Expo 규칙)
- `service_role` 키는 환경 변수에도 `EXPO_PUBLIC_` 접두어를 쓰지 않는다 — 서버 전용
- `.env.local`은 `.gitignore`에 포함

### 로컬 개발

```bash
# Supabase 로컬 시작
npx supabase start

# 로컬 Studio 접속
# http://127.0.0.1:54323

# 마이그레이션 적용
npx supabase db reset

# 타입 재생성
npx supabase gen types typescript --local > src/services/supabase/types.ts
```

---

## 자주 쓰는 라이브러리/도구

### 권장

| 용도 | 라이브러리 | 이유 |
|------|-----------|------|
| Supabase 클라이언트 | **@supabase/supabase-js** | 공식 클라이언트, 타입 안전 |
| 서버 상태 관리 | **@tanstack/react-query** | Supabase 쿼리 결과 캐싱/갱신 자동화 |
| 세션 저장 (RN) | **@react-native-async-storage/async-storage** | Supabase Auth 세션 영속화 |
| 보안 저장 | **expo-secure-store** | 민감 토큰 저장 (AsyncStorage 대신) |
| 스키마 검증 | **zod** | Edge Function 입력 검증, 폼 검증 |
| 로컬 개발 | **supabase CLI** | 로컬 DB, 마이그레이션, 타입 생성 |
| 실시간 동기화 | **Supabase Realtime** (내장) | 별도 라이브러리 불필요 |

### 피해야 할 것

| 라이브러리/방식 | 이유 | 대안 |
|---------------|------|------|
| 직접 PostgreSQL 드라이버 | Supabase 클라이언트가 RLS/Auth 자동 처리 | `@supabase/supabase-js` |
| Firebase와 혼용 | 인증/DB 이중 관리 혼란 | Supabase로 통일 |
| ORM (Prisma 등) 클라이언트에서 | 클라이언트에서 DB 직접 접근은 보안 위험 | Supabase JS 클라이언트 사용 |
| 수동 JWT 관리 | Supabase Auth가 자동 관리 | `supabase.auth` 사용 |

---

## 검증 체크리스트

### 새 테이블 생성 시

```
- [ ] RLS 활성화 (`ALTER TABLE ... ENABLE ROW LEVEL SECURITY`)
- [ ] SELECT/INSERT/UPDATE/DELETE 정책 작성 (최소 SELECT 필수)
- [ ] RLS 정책에 재귀 서브쿼리 없는지 확인 (같은 테이블 참조 금지)
- [ ] 다른 유저로 접근 테스트 (RLS 검증)
- [ ] status 등 필터 대상 컬럼에 DEFAULT 값 설정
- [ ] 외래 키에 인덱스 생성
- [ ] 정책의 auth.uid() 가 (select auth.uid()) 로 랩핑됐는지 확인 (auth_rls_initplan lint)
- [ ] 마이그레이션 파일 생성
- [ ] 타입 재생성 (`supabase gen types`)
```

### 새 쿼리 함수 작성 시

```
- [ ] { data, error } 구조분해 후 error 체크
- [ ] 조인 쿼리에서 RLS 충돌 가능성 확인 (중첩 조인 → 2단계 쿼리 고려)
- [ ] 단건 조회 시 .single() 사용
- [ ] INSERT 시 필터 대상 컬럼 (status 등) 명시
- [ ] 반환 타입 명시
```

### Edge Function 배포 시

```
- [ ] `Deno.serve()` 사용 (deno.land/std serve 아님) + 의존성 deno.json 선언
- [ ] 입력 스키마 검증 (zod safeParse) + 에러 응답 형식 통일
- [ ] secret 키가 코드/로그에 노출되지 않음 (Deno.env 만)
- [ ] 신 API 키 호출 경로면 verify_jwt = false 설정 확인
- [ ] 로컬 테스트: npx supabase functions serve → curl 로 정상/에러 케이스 확인
```

### 배포 전 (프로덕션)

```
- [ ] Dashboard Advisors (Security/Performance) 경고 0 — 특히 auth_rls_initplan, RLS 미활성 테이블
- [ ] 신 키 체계 확인: 클라이언트 번들에 secret 키 미포함 (grep sb_secret_)
- [ ] 마이그레이션이 로컬 db reset 으로 재현 가능 (드리프트 없음 — 회사/집 2환경 동기화 확인)
- [ ] Realtime 구독의 unsubscribe 경로 존재 (화면 이탈 테스트)
- [ ] 민감 작업(결제/권한)의 서버측 확인이 getUser() 기반인지 확인
```

### RLS 정책 수정 시

```
- [ ] 수정 전 현재 정책 백업 (pg_policies 조회 결과 기록)
- [ ] auth.uid() 가 (select auth.uid()) 로 랩핑되어 있는지 확인
- [ ] 같은 테이블 재참조(재귀) 없는지 확인
- [ ] UPDATE 정책 수정 시 대응 SELECT 정책과 조건 일치 확인 (무음 실패 방지)
- [ ] 침투 테스트: 다른 사용자 JWT 로 SET LOCAL 후 접근 시도 (보안 섹션 SQL)
- [ ] 해당 테이블을 구독하는 Realtime 이 있으면 이벤트 수신 재확인 (무음 필터링)
```

### Supabase 500 에러 디버깅 흐름

```
1. 브라우저 콘솔/네트워크 탭에서 실패한 PostgREST URL 확인
2. URL의 select 파라미터에서 중첩 조인 확인 (테이블A → 테이블B → 테이블C)
3. 관련 테이블의 RLS 정책 점검:
   - pg_policies 뷰에서 확인: SELECT * FROM pg_policies WHERE tablename = '테이블명'
   - 정책의 USING 절에 같은 테이블 참조(재귀) 있는지 확인
4. 재귀 발견 시: 정책을 user_id = auth.uid() 등 단순 조건으로 교체
5. 재귀 아니면: 쿼리를 2단계로 분리 (ID 목록 조회 → 데이터 조회)
6. 수정 후 동일 시나리오 재테스트
```

---

## 우회/핵 금지 원칙

| 우회 패턴 (금지) | 위험성 | 정석 해결 |
|-----------------|--------|----------|
| `SECURITY DEFINER` 함수로 RLS 우회 | 전체 보안 모델 무력화 — 모든 사용자가 모든 데이터 접근 | RLS 정책 자체를 올바르게 수정 |
| secret 키(구 service_role)를 클라이언트에서 사용 | RLS 완전 무시, 전체 DB 노출 | publishable 키 + 올바른 RLS 정책 |
| `getSession()` 결과로 서버측 인가 판단 | 로컬 세션은 위조 가능 — 권한 상승 취약점 | `getClaims()`(기본) / `getUser()`(민감 작업) |
| RLS 비활성화하여 에러 해결 | 보안 없음 상태로 배포 | 정책 점검 후 올바른 정책 설정 |
| 쿼리가 500이면 해당 기능을 제거 | 근본 원인(RLS) 미해결, 다른 쿼리에서 동일 문제 재발 | RLS 정책 수정 또는 쿼리 분리 |

**원칙**: RLS/Auth/CORS가 에러를 일으키면, **보안 메커니즘을 우회하지 말고 올바르게 설정**한다. "일단 동작하게" 하는 것은 보안 시한폭탄이다.

---

## 프로젝트별 확장 포인트

> 아래 항목들은 프로젝트마다 다르므로 `/CLAUDE.md`에서 정의합니다:
> - Supabase 프로젝트 URL, anon key (환경 변수)
> - 테이블 스키마 설계
> - RLS 정책 상세
> - Edge Function 목록과 역할
> - Storage 버킷 구성
> - Realtime 구독 대상 테이블
> - 소셜 로그인 프로바이더 설정 (카카오, 네이버 등)
> - 로컬/스테이징/프로덕션 환경 설정

<!-- 개선 이력
- 2026-07-03: 2026 최신화 (웹 리서치 기반, 출처 확보) — 신 API 키 체계(sb_publishable_/sb_secret_, apikey 헤더 함정),
  Deno.serve + npm: specifier 전환, (select auth.uid()) initplan 패턴, getClaims/getUser/getSession 선택 기준,
  RLS 함정 4→10행 (정책0 전체차단/UPDATE-SELECT 무음실패/Realtime 무음필터링/gen types 미재생성/Storage 별도 정책 등),
  체크리스트 3→6종(RLS 정책 수정 시 포함), 보안(signed URL·RLS 침투 테스트 SQL)/성능(인덱스+EXPLAIN·페이지네이션·디바운스) 코드 예시 신설.
  /ScoreRules 독립 채점: 60(상한)→74→**75점 Level 3** 확정.
- 2026-03-27: 플랫폼별 Auth 설정 추가 — 웹에서 AsyncStorage 사용 시 세션 유실 발견
- 2026-03-27: RLS 실전 함정 섹션 추가 — 중첩 조인/재귀 서브쿼리/임베디드 집계 + RLS = 500 발견
- 2026-03-27: DB 컬럼 기본값 함정 추가 — status NULL로 INSERT 후 조회 누락 발견
- 2026-03-27: 검증 체크리스트 추가 — 새 테이블/쿼리/500 디버깅 체크리스트
- 2026-03-27: 우회/핵 금지 원칙 추가 — SECURITY DEFINER 우회 시도 경험에서 도출
-->
