---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "tsconfig.json"
---

# TypeScript 개발 규칙

> 이 파일은 TypeScript 프로젝트에 범용적으로 적용되는 시니어 수준 규칙입니다.
> 프로젝트별 설정, 빌드 명령, 폴더 구조는 `/CLAUDE.md`에 정의합니다.

## 핵심 원칙

- **Type-First**: 타입이 문서다. 코드를 읽지 않아도 타입만 보면 의도를 알 수 있어야 한다.
- **Strict Mode 필수**: `tsconfig.json`에서 `"strict": true`는 절대 끄지 않는다. 타입 안전성의 기본이다.
- **any 금지**: `any`는 타입 시스템을 무력화한다. `unknown`을 쓰고 타입 가드로 좁혀라.
- **불변성 우선**: `const` 기본, `let`은 꼭 필요한 경우만. `readonly`와 `Readonly<T>`를 적극 활용한다.
- **명시적 > 암시적**: 타입 추론이 되더라도 함수 반환 타입, 공개 API는 명시적으로 작성한다.
- **단순함 추구**: 타입 체조(type gymnastics)는 지양한다. 복잡한 타입보다 명확한 타입이 낫다.

---

## 프로젝트 구조

- 기능(feature) 기반 폴더 구조를 권장한다 (기술 레이어 기반 X)
- 관련된 파일(컴포넌트, 훅, 타입, 유틸)은 같은 폴더에 배치한다 (co-location)
- `index.ts`는 re-export 용도로만 사용한다 — 로직을 넣지 않는다
- 순환 참조(circular dependency)를 만들지 않는다
- 공유 타입은 `types/` 또는 `shared/` 폴더에 분리한다

```
src/
├── features/           # 기능별 폴더
│   ├── auth/
│   │   ├── types.ts         # 이 기능의 타입 정의
│   │   ├── constants.ts     # 이 기능의 상수
│   │   ├── hooks/           # 커스텀 훅
│   │   ├── components/      # UI 컴포넌트
│   │   ├── utils.ts         # 유틸리티 함수
│   │   └── index.ts         # re-export만
│   └── tasks/
├── shared/             # 공유 모듈
│   ├── types/          # 전역 타입 정의
│   ├── utils/          # 공유 유틸리티
│   ├── hooks/          # 공유 훅
│   └── constants/      # 전역 상수
├── services/           # 외부 서비스 연동 (API 클라이언트 등)
├── config/             # 앱 설정
└── app/                # 앱 진입점
```

---

## 네이밍 컨벤션

### 파일/폴더

- **일반 파일**: kebab-case (`user-profile.ts`, `auth-provider.tsx`)
- **컴포넌트 파일**: PascalCase도 허용 (`UserProfile.tsx`) — 프로젝트에서 하나로 통일
- **테스트 파일**: `*.test.ts` 또는 `*.spec.ts` — 프로젝트에서 통일
- **타입 전용 파일**: `types.ts` (기능 폴더 내) 또는 `{feature}.types.ts`
- **상수 파일**: `constants.ts`
- **폴더명**: kebab-case (`user-management/`, `push-notification/`)

### 변수/함수/클래스

- **변수/함수**: camelCase (`getUserById`, `isAuthenticated`, `formatPrice`)
- **상수**: UPPER_SNAKE_CASE (`MAX_RETRY_COUNT`, `API_BASE_URL`, `DEFAULT_PAGE_SIZE`)
- **클래스**: PascalCase (`EventService`, `AuthProvider`)
- **타입/인터페이스**: PascalCase (`UserProfile`, `CalendarEvent`)
- **enum 은 사용하지 않는다** — `as const` 객체 + union 타입으로 대체 (아래 "타입 정의 패턴" 참조. 이유: erasable 문법이 아니라 Node 네이티브 실행 불가 + `erasableSyntaxOnly` 차단 대상)
- **제네릭 파라미터**: 단일 대문자 또는 T 접두어 (`T`, `TData`, `TError`, `K`, `V`)

### 의미 기반 접두어/접미어

| 종류 | 접두어/접미어 | 예시 |
|------|-------------|------|
| boolean 변수 | `is`, `has`, `can`, `should` | `isLoading`, `hasPermission`, `canEdit` |
| 이벤트 핸들러 | `handle` + 동사 | `handleSubmit`, `handlePress`, `handleChange` |
| 콜백 prop | `on` + 동사 | `onSubmit`, `onPress`, `onChange` |
| Props 타입 | `~Props` | `TaskCardProps`, `ProfileListProps` |
| State 타입 | `~State` | `AuthState`, `FormState` |
| API 응답 | `~Response` | `LoginResponse`, `EventListResponse` |
| API 요청 | `~Request` | `CreateEventRequest`, `UpdateUserRequest` |
| 훅 | `use` + 명사/동사 | `useAuth`, `useEvents`, `useFetchData` |
| 컨텍스트 | `~Context` | `AuthContext`, `ThemeContext` |
| Provider | `~Provider` | `AuthProvider`, `ThemeProvider` |
| 유틸 함수 | 동사로 시작 | `formatDate`, `parseQuery`, `validateEmail` |

---

## 코드 포맷팅

- **세미콜론**: 사용 (Prettier 기본값 준수)
- **따옴표**: 작은따옴표(`'`) — JSX 내부만 큰따옴표(`"`)
- **들여쓰기**: 스페이스 2칸
- **줄 길이**: 최대 100자 (Prettier `printWidth: 100`)
- **후행 쉼표**: 항상 사용 (`trailing comma: all`)
- **중괄호**: 한 줄이라도 생략하지 않음 (`if (x) { return y; }`)
- **빈 줄**: 논리 블록 사이에 1줄, 2줄 이상 금지
- **import 후**: 빈 줄 1줄
- **함수 사이**: 빈 줄 1줄

```typescript
// ❌ Bad — 중괄호 생략 + 이중 따옴표 + 긴 체인 한 줄
if (isValid) return processData(items.filter(x => x.active).map(x => x.id).join(","))

// ✅ Good — 중괄호 필수, 체인은 단계별 줄바꿈
if (isValid) {
  return processData(
    items
      .filter((item) => item.active)
      .map((item) => item.id)
      .join(','),
  );
}
```

---

## import/export 규칙

### import 정렬 순서 (상단 → 하단)

```typescript
// 1. 외부 라이브러리 (node_modules)
import React, { useState, useCallback } from 'react';
import { View, Text } from 'react-native';
import { useQuery } from '@tanstack/react-query';

// 2. 내부 절대 경로 (@/ alias)
import { useAuth } from '@/features/auth';
import { ApiClient } from '@/services/api';

// 3. 상대 경로 (현재 기능 내)
import { EventCard } from './components/EventCard';
import { formatEventDate } from './utils';
import type { CalendarEvent } from './types';
```

### export 규칙

- **Named export 우선**: `export function`, `export const`, `export type`
- Default export는 **페이지/스크린 컴포넌트에만** 허용
- 이유: named export가 리팩토링에 안전하고, IDE 자동완성이 더 잘 됨
- **타입 전용 import/export**: `import type`, `export type` 사용 (번들에서 제거됨)

```typescript
// 타입만 import할 때
import type { User, UserRole } from './types';

// 값과 타입을 함께 import할 때
import { UserService } from './service';
import type { UserServiceConfig } from './types';

// re-export (index.ts)
export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/useAuth';
export type { AuthState, LoginCredentials } from './types';
```

---

## 파일 내부 코드 순서

```typescript
// 1. import 문 (위 정렬 순서에 따라)

// 2. 타입/인터페이스 정의 (이 파일 전용 — 공유 타입은 types.ts에)

// 3. 상수 정의

// 4. 헬퍼/유틸 함수 (private, 이 파일 내에서만 사용)

// 5. 메인 export (함수, 클래스, 컴포넌트)

// 6. 하위 컴포넌트 (있다면)
```

---

## 타입 정의 패턴

### 기본 타입 정의

```typescript
// interface: 객체 형태 정의 (확장 가능, 선언 병합 가능)
interface User {
  readonly id: string;        // 변경 불가 필드는 readonly
  name: string;
  email: string;
  role: UserRole;
  createdAt: Date;
  profileImage?: string;      // 옵셔널 필드
}

// type: 유니온, 인터섹션, 유틸리티 조합
type EventStatus = 'open' | 'closed' | 'cancelled';
type CreateUserInput = Omit<User, 'id' | 'createdAt'>;
type UserUpdate = Partial<Pick<User, 'name' | 'email' | 'profileImage'>>;
```

### 유틸리티 타입 실전 활용

```typescript
// Omit — 특정 필드 제거 (생성 DTO에 유용)
type CreateEventInput = Omit<Event, 'id' | 'createdAt' | 'updatedAt'>;

// Pick — 특정 필드만 선택 (요약/미리보기에 유용)
type EventSummary = Pick<Event, 'id' | 'title' | 'date' | 'status'>;

// Partial — 모든 필드 옵셔널 (업데이트 DTO에 유용)
type UpdateEventInput = Partial<CreateEventInput>;

// Required — 모든 필드 필수 (기본값이 있는 설정에 유용)
type RequiredConfig = Required<AppConfig>;

// Record — 키-값 매핑
type EventsByDate = Record<string, Event[]>;
type RolePermissions = Record<UserRole, Permission[]>;

// Extract / Exclude — 유니온에서 필터링
type ActiveStatus = Extract<EventStatus, 'open'>; // 'open'
type InactiveStatus = Exclude<EventStatus, 'open'>; // 'closed' | 'cancelled'
```

### enum 대체: as const 객체 + union

```typescript
// ❌ enum — 코드를 생성하는 문법 (Node 네이티브 실행 불가, erasableSyntaxOnly 차단)
enum UserRole { Admin = 'admin', Member = 'member' }

// ✅ as const 객체 + union — 런타임 코스트 0, 값과 타입 모두 제공
const UserRole = {
  Admin: 'admin',
  Member: 'member',
  Guest: 'guest',
} as const;
type UserRole = (typeof UserRole)[keyof typeof UserRole]; // 'admin' | 'member' | 'guest'

// 사용은 enum과 동일한 감각
function checkAccess(role: UserRole) { ... }
checkAccess(UserRole.Admin);
```

### satisfies — 검증하되 추론은 보존 (4.9+, 표준 관용구)

```typescript
// ❌ 타입 어노테이션 — 검증되지만 리터럴 추론을 잃음
const config: Record<string, { url: string }> = {
  dev: { url: 'http://localhost' },
  prod: { url: 'https://api.example.com' },
};
config.dev;        // 타입: { url: string } | undefined — 키 자동완성 상실

// ✅ satisfies — 형태는 검증하고, 추론된 구체 타입은 유지
const config = {
  dev: { url: 'http://localhost' },
  prod: { url: 'https://api.example.com' },
} satisfies Record<string, { url: string }>;
config.dev.url;    // 키 자동완성 + 오타 시 즉시 에러

// as const satisfies — 읽기 전용 리터럴 + 형태 검증 (설정 객체의 정석)
const ROUTES = ['/home', '/settings'] as const satisfies readonly `/${string}`[];
```

> **판단 기준**: 함수 파라미터/반환은 타입 어노테이션(`: T`), **객체/설정 리터럴 정의는 `satisfies`**, `as` 단언은 최후의 수단.

### 고급 타입 패턴

```typescript
// Branded Type — 같은 원시 타입이지만 의미가 다른 값을 구분
type UserId = string & { readonly __brand: 'UserId' };
type EventId = string & { readonly __brand: 'EventId' };

function createUserId(id: string): UserId {
  return id as UserId;
}
// createUserId('abc')는 UserId, EventId 자리에 넣으면 타입 에러

// Discriminated Union — 상태별 처리에 최적
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// 사용
function renderState(state: AsyncState<User>) {
  switch (state.status) {
    case 'idle': return null;
    case 'loading': return <Spinner />;
    case 'success': return <UserCard user={state.data} />; // data 자동 타입 추론
    case 'error': return <ErrorView error={state.error} />; // error 자동 타입 추론
  }
}

// Result 패턴 — try-catch 대안
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function parseJSON<T>(json: string): Result<T> {
  try {
    return { ok: true, value: JSON.parse(json) as T };
  } catch (e) {
    return { ok: false, error: e instanceof Error ? e : new Error(String(e)) };
  }
}
```

---

## 함수 규칙

### 함수 선언

```typescript
// 공개 함수: function 키워드 (호이스팅 가능, 의도 명확)
export function calculateFee(amount: number, rate: number): number {
  return amount * rate;
}

// 콜백, 인라인: 화살표 함수
const formatPrice = (price: number): string => `${price.toLocaleString()}원`;

// 파라미터 3개 초과 → 객체로 묶기
interface CreateEventParams {
  title: string;
  date: Date;
  location: string;
  maxParticipants: number;
  fee: number;
}
export function createEvent(params: CreateEventParams): Promise<Event> { ... }

// 옵셔널 파라미터보다 기본값 우선
export function fetchEvents(page: number = 1, limit: number = 20): Promise<Event[]> { ... }
```

### 순수 함수 우선

- 같은 입력 → 항상 같은 출력
- 부수 효과(side effect) 없음
- 외부 상태를 변경하지 않음
- 부수 효과가 필요한 함수는 명확히 분리하고 이름으로 표시

```typescript
// 순수 함수 (Good)
function calculateTotalFee(events: readonly Event[]): number {
  return events.reduce((sum, event) => sum + event.fee, 0);
}

// 부수 효과 함수 — 이름에서 드러남 (Good)
async function sendNotificationToMembers(teamId: string, message: string): Promise<void> {
  // ...
}
```

### 불변 업데이트 패턴

```typescript
// 배열: spread 또는 불변 메서드
const addItem = <T>(arr: readonly T[], item: T): T[] => [...arr, item];
const removeItem = <T>(arr: readonly T[], index: number): T[] =>
  arr.filter((_, i) => i !== index);
const updateItem = <T>(arr: readonly T[], index: number, item: T): T[] =>
  arr.map((existing, i) => (i === index ? item : existing));

// 객체: spread
const updateUser = (user: User, updates: Partial<User>): User => ({
  ...user,
  ...updates,
});

// 중첩 객체: 깊은 spread
const updateNestedState = (state: AppState): AppState => ({
  ...state,
  user: {
    ...state.user,
    profile: {
      ...state.user.profile,
      name: 'New Name',
    },
  },
});
```

---

## 비동기 처리

- `async/await`를 기본으로 사용한다 (`.then().catch()` 체이닝 지양)
- 비동기 함수 반환 타입에 `Promise<T>`를 명시한다
- 병렬 실행 가능한 비동기 작업은 `Promise.all()` 또는 `Promise.allSettled()`를 사용한다
- floating promise 금지 (`await` 없이 Promise를 버리지 않는다)

```typescript
// ❌ Bad: forEach/map 안의 await — 기다려지지 않음 (실전 함정 #1)
items.forEach(async (item) => {
  await processItem(item);       // Promise가 버려짐 — 완료 보장 X, 에러 미포착
});
const results = items.map(async (item) => transform(item)); // Promise<T>[] — 값 배열 아님!

// ✅ Good: 순차가 필요하면 for...of
for (const item of items) {
  await processItem(item);
}

// ✅ Good: 병렬이면 Promise.all + map
const results = await Promise.all(items.map((item) => transform(item)));

// Good: 병렬 실행
const [users, events] = await Promise.all([
  fetchUsers(),
  fetchEvents(),
]);

// Good: 일부 실패해도 나머지 결과를 받기
const results = await Promise.allSettled([
  sendEmailNotification(userId),
  sendPushNotification(userId),
  sendSMSNotification(userId),
]);

// Good: 취소 가능한 요청
const controller = new AbortController();
const response = await fetch(url, { signal: controller.signal });
// 취소: controller.abort();

// Good: 타임아웃
const fetchWithTimeout = async <T>(
  fetcher: () => Promise<T>,
  timeoutMs: number,
): Promise<T> => {
  const timeout = new Promise<never>((_, reject) =>
    setTimeout(() => reject(new Error('Timeout')), timeoutMs),
  );
  return Promise.race([fetcher(), timeout]);
};
```

---

## Null/Undefined 처리

- `null`은 "값이 없음을 명시", `undefined`는 "값이 설정되지 않음"
- 함수 반환에서 `null | undefined` 혼용하지 않는다 — 하나로 통일
- `!` (non-null assertion)은 절대 사용하지 않는다 — 타입 가드로 좁혀라

```typescript
// ❌ Bad — || 는 falsy 전체를 기본값으로 치환 (0, '', false가 유효값인 경우 데이터 파괴)
const pageSize = config.pageSize || DEFAULT_PAGE_SIZE;   // pageSize: 0 → DEFAULT로 둔갑
const nickname = user.nickname || '익명';                 // '' (의도적 빈 닉네임) → '익명'으로 둔갑

// ✅ Good — ?? 는 null/undefined만 기본값 처리
const pageSize = config.pageSize ?? DEFAULT_PAGE_SIZE;
const nickname = user.nickname ?? '익명';

// ❌ Bad — ! 단언: 가정이 틀리면 크래시, 컴파일러 경고는 이미 꺼짐
const name = user!.profile!.name;

// ✅ Good — 옵셔널 체이닝 + nullish coalescing
const displayName = user?.profile?.name ?? '알 수 없음';

// 타입 가드로 좁히기 (non-null assertion 대신)
function processUser(user: User | null): void {
  if (!user) {
    return; // early return
  }
  // 여기서 user는 User 타입으로 자동 좁혀짐
  console.log(user.name);
}

// 커스텀 타입 가드
function isDefined<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

const validUsers = users.filter(isDefined); // (User | null)[] → User[]
```

---

## 안티패턴 (하지 말 것)

### 타입 관련
| 안티패턴 | 실패 결과 | 대안 |
|---------|----------|------|
| `any` 사용 | 오염이 전파되어 멀리 떨어진 코드에서 `Cannot read property of undefined` 크래시 — 원인 추적에 파일 여러 개 거슬러야 함 | `unknown` + 타입 가드 |
| `as` 타입 단언 남용 | 검증 없는 믿음 — 형태가 다른 데이터가 통과해 깊은 곳에서 런타임 에러, 타입 에러는 0개 | 타입 가드, `satisfies`, 제네릭 |
| `// @ts-ignore` | 그 줄의 **모든** 미래 에러까지 영구 침묵 — 리팩토링으로 새 버그가 생겨도 모름 | `@ts-expect-error` + 사유 (에러 사라지면 알려줌) |
| `!` non-null assertion | 가정이 틀리는 순간(빈 응답, 초기화 전 접근) 프로덕션 크래시 | 옵셔널 체이닝 / 타입 가드 |
| 빈 인터페이스 `{}` | 원시값 포함 거의 모든 값을 허용 — 타입 검사가 사실상 없음 | `Record<string, never>` 또는 구체적 타입 |
| `object` 타입 | 프로퍼티 접근 전부 에러 or 너무 넓음 — 실사용 불가 타입 | 구체적 인터페이스 정의 |
| `enum` (숫자/문자열 모두) | Node 네이티브 TS 실행에서 SyntaxError 크래시 + `erasableSyntaxOnly` 차단 대상 | `as const` 객체 + union (위 패턴 참조) |
| `namespace` | 코드 생성 문법 — enum과 동일한 문제 + ES 모듈과 이중 체계 | ES 모듈 사용 |

### 코드 구조 관련
| 안티패턴 | 실패 결과 | 대안 |
|---------|----------|------|
| `index.ts`에 로직 작성 | 순환 참조 유발 → 특정 import 순서에서만 undefined 나오는 하이젠버그 | re-export만 |
| default export 남용 | 이름 바꿔치기 import가 가능해 리팩토링 시 참조 추적 실패 | named export 우선 |
| 클래스 남용 | this 바인딩 버그 + 불필요한 상태 — 함수형으로 충분한 로직이 무거워짐 | 함수 + 타입으로 충분한 경우가 대부분 |
| 매직 넘버/문자열 | 같은 값이 3곳에 흩어져 1곳만 수정 → 데이터 불일치 버그 | 상수(`as const`)로 정의 |
| 거대한 파일 (300줄+) | 리뷰/수정 범위 파악 불가 — AI도 파일 일부만 보고 잘못 수정 | 기능별로 분리 |

### AI가 흔히 생성하는 실수
| 실수 | 실패 결과 | 수정 방법 |
|------|----------|----------|
| `console.log` 디버깅 코드 남기기 | 프로덕션 콘솔에 내부 데이터/토큰 노출 + 노이즈 | 프로젝트 로거 사용 또는 제거 |
| `catch(e: any)` | `e.message` 접근이 무검증 통과 → non-Error throw 시 2차 크래시 | `catch(e: unknown)` 후 `instanceof` 체크 |
| 불필요한 `async` (await 없는 async 함수) | 호출부가 불필요하게 await 체인화 — 에러 스택도 한 겹 늘어남 | `async` 제거, 그냥 Promise 반환 |
| `useEffect` 내 직접 async | effect가 Promise를 반환 → cleanup 함수 무시됨 (구독 해제 누락) | 내부에 별도 함수 정의 후 호출 |
| 타입을 컴포넌트에 인라인 정의 | 같은 형태가 파일마다 재정의 → 필드 변경 시 일부만 갱신되어 불일치 | `types.ts`로 분리 |
| `==` 사용 | `'' == 0` 이 true → 폼 검증/분기 오동작 | 항상 `===` 사용 |
| `var` 사용 | 함수 스코프 호이스팅 → 루프 클로저에서 마지막 값 공유 버그 | `const` (기본) 또는 `let` |
| 배열 `.forEach` 내 `await` | 순차 보장 없이 다음 로직 실행 — 완료 전 상태로 저장/응답 | `for...of` 또는 `Promise.all` (실전 함정 #1) |
| `.map(async ...)` 결과를 값 배열로 사용 | `Promise<T>[]`를 값으로 렌더링/저장 → `[object Promise]` 출력 | `await Promise.all(...)`로 감싸기 |
| 존재하지 않는 패키지/API import (환각) | 설치/빌드 실패, 최악은 이름만 같은 악성 패키지 설치 | package.json 실존 확인 + `npx tsc --noEmit` 즉시 검증 |
| 낡은 API로 생성 (deprecated 시그니처) | 경고 누적 → 다음 메이저에서 일괄 파손 | 공식 문서의 현행 시그니처 확인 — 특히 메이저 버전 경계 |
| 빈 catch 블록 (`catch {}`) | 실패가 무음 처리 → 데이터 누락인데 원인 추적 불가 | 최소 로깅 + 상위 전파 판단 |
| `new Array()` / `String()`/`Number()` 남용 | `new Array(3)`은 빈 슬롯 3개 (map 무시됨) — 의도와 다른 배열 | 리터럴 `[]`, 명확한 파싱(`parseInt` 등) |

### 실전에서 발견된 지뢰 (프로젝트 경험 누적분)

> 작업 중 발견한 함정을 여기에 누적한다 — 범용화(프로젝트 용어 제거) 후 기록. 형식: 발견일 + 증상 + 원인 + 해결.

- 2026-07-03: date-only 문자열 `new Date('YYYY-MM-DD')`가 UTC 자정으로 해석되어 KST에서 하루 밀림 — 실전 함정 #5로 승격 기록.

---

## 에러/예외 처리

### 경계에서만 catch — 내부는 전파 (판단 기준)

| 위치 | 처리 방식 | 이유 |
|------|----------|------|
| **UI 이벤트 핸들러 / API 라우트 / 잡 진입점** (경계) | **여기서만 try-catch** — 사용자 메시지 표시 or 에러 응답 변환 | 사용자/호출자에게 결과를 책임지는 마지막 지점 |
| **내부 함수 / 서비스 / 유틸** | catch 하지 않고 **그대로 전파** | 내부에서 삼키면 경계가 실패를 모른 채 성공 흐름 진행 |
| **부분 실패 허용 지점** (배치, 다건 처리) | 항목 단위 catch + 에러 목록 수집 | 1건 실패로 전체 중단 방지 — 단 수집한 에러는 반드시 보고 |
| **예상 가능한 실패가 잦은 로직** (파싱, 검증) | throw 대신 **Result 패턴** 반환 | 실패가 정상 흐름의 일부일 때 예외는 과잉 — 타입으로 분기 강제 |

> **Result vs throw 선택**: 호출자가 실패를 **매번 분기 처리해야 하는가?** Yes → Result (타입이 분기를 강제). 실패가 예외적 상황(버그, 인프라 장애)인가? Yes → throw (경계까지 전파).

### 기본 패턴

```typescript
// catch에서 unknown으로 받기 (TS 4.4+ 기본)
try {
  await someOperation();
} catch (error: unknown) {
  if (error instanceof AppError) {
    // 비즈니스 에러 — 사용자에게 표시
    showErrorToast(error.userMessage);
  } else if (error instanceof Error) {
    // 일반 에러 — 로깅 + 일반 메시지
    logger.error(error.message, { stack: error.stack });
    showErrorToast('문제가 발생했습니다. 다시 시도해주세요.');
  } else {
    // 예상치 못한 에러
    logger.error('Unknown error', { error: String(error) });
  }
}
```

### 커스텀 에러

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly userMessage: string,
    public readonly statusCode?: number,
  ) {
    super(message);
    this.name = 'AppError';
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(
      `${resource} not found: ${id}`,
      'NOT_FOUND',
      `${resource}을(를) 찾을 수 없습니다.`,
      404,
    );
  }
}
```

### 규칙

- 에러를 삼키지 않는다 — 최소한 로깅
- 사용자에게 보여줄 에러와 내부 에러를 구분한다
- 에러 메시지에 민감 정보 포함 금지
- `JSON.parse()`, 외부 데이터 파싱 시 반드시 try-catch + 스키마 검증
- API 에러는 일관된 형식으로 반환 (`{ code, message, details }`)

---

## 보안

- **입력 검증 필수**: 사용자 입력은 반드시 검증 (zod 등 스키마 검증 라이브러리 활용)
- **하드코딩 금지**: API 키, 토큰, 비밀번호 → 환경 변수 (`.env`)
- **`.env` 파일**: `.gitignore`에 반드시 포함
- **위험 함수 금지**: `eval()`, `new Function()`, `document.write()`
- **XSS 방지**: `dangerouslySetInnerHTML` 사용 시 반드시 sanitize, 직접 DOM 조작 지양
- **URL 검증**: 파라미터, 쿼리 스트링은 항상 검증/이스케이프
- **로그 보안**: 민감 정보(비밀번호, 토큰, 카드번호) 로그 출력 금지
- **의존성 보안**: 알려진 취약점이 있는 패키지 사용 금지 (`npm audit`)
- **타입으로 보안 강화**: Branded Type으로 검증된 값과 미검증 값을 구분

```typescript
// ❌ Bad — 외부 입력을 검증 없이 신뢰 (형태가 다르면 깊은 곳에서 크래시 + 인젝션 표면)
const body = await request.json();
await db.updateUser(body.userId, body.updates);

// ✅ Good — 신뢰 경계에서 스키마 검증 후에만 내부로 통과
const UpdateUserSchema = z.object({
  userId: z.string().uuid(),
  updates: z.object({ name: z.string().max(100) }).strict(),
});
const parsed = UpdateUserSchema.safeParse(await request.json());
if (!parsed.success) {
  return jsonError(400, parsed.error.flatten());
}
await db.updateUser(parsed.data.userId, parsed.data.updates);
```

```typescript
// ❌ Bad — 시크릿 하드코딩 (git 이력에 영구 잔존 → 유출 사고)
const client = createClient('https://api.example.com', 'sk_live_abc123...');

// ✅ Good — 환경 변수 + 시작 시점 존재 검증 (누락을 배포 직후가 아니라 기동 시 발견)
const API_KEY = process.env.API_KEY;
if (!API_KEY) {
  throw new Error('API_KEY 환경 변수가 설정되지 않았습니다');
}
const client = createClient('https://api.example.com', API_KEY);
```

---

## 성능 최적화

### 렌더링 (React/React Native)
- **React Compiler 활성 환경 (React 19 + Expo SDK 54+ 기본)**: 수동 `useMemo`/`useCallback`/`React.memo` **지양** — 컴파일러가 자동 최적화하며, 수동 메모이제이션은 노이즈+버그 표면만 추가. 컴파일러 lint가 지적하는 곳만 수정
- 컴파일러 비활성(레거시) 환경에서만: `React.memo`/`useMemo`/`useCallback`을 **측정 후** 선별 적용
- 렌더링 루프 내 인라인 객체/배열 생성은 여전히 지양 (`style={{}}` — 컴파일러와 무관한 리렌더 유발원)
- 불필요한 상태(state) 만들지 않기 — 파생 가능한 값은 계산으로 처리

### 데이터
- 대용량 리스트: 가상화 적용 (`FlashList`, `react-window`)
- 이미지: 적절한 사이즈로 리사이즈 + lazy loading
- 페이지네이션/무한 스크롤: 전체 데이터를 한번에 로드하지 않기

### 번들
- tree-shaking 가능한 import: `import { debounce } from 'lodash-es'`
- `lodash` 전체 import 절대 금지 → 개별 import 또는 네이티브 대체
- 동적 import(`lazy`)로 코드 분할: 초기 로드에 필요 없는 화면/기능

```typescript
// ❌ Bad — 전체 import: tree-shaking 불가, 번들에 라이브러리 통째로 포함
import _ from 'lodash';
_.debounce(handler, 300);

// ✅ Good — 개별 import (또는 네이티브로 대체 가능한지 먼저 검토)
import { debounce } from 'lodash-es';
debounce(handler, 300);
```

```typescript
// ❌ Bad — 렌더링마다 새 객체/배열 생성 → 자식이 매번 리렌더
<List style={{ padding: 16 }} items={items.filter((i) => i.active)} />

// ✅ Good — 스타일은 상수로, 파생 데이터는 렌더 밖(또는 컴파일러 최적화 대상 코드)에서
const listStyle = { padding: 16 } as const;
const activeItems = items.filter((i) => i.active);
<List style={listStyle} items={activeItems} />
```

### TypeScript 컴파일
- 깊은 제네릭 중첩 지양 (3단계 이내)
- 조건부 타입(Conditional Type) 과용 주의 — IDE 성능 저하
- `type` vs `interface`: 성능 차이 미미하지만, `interface`가 에러 메시지가 더 읽기 좋음

---

## 테스트

- **테스트 파일 위치**: 소스 파일 옆에 배치 (`UserCard.test.tsx`) 또는 `__tests__/` 폴더
- **네이밍**: `describe`에 대상, `it`/`test`에 기대 동작
- **유틸/헬퍼**: 유닛 테스트 필수
- **API 연동**: MSW(Mock Service Worker)로 API 모킹
- **커버리지**: 비즈니스 로직 80%+, UI 컴포넌트는 주요 인터랙션 위주

```typescript
describe('calculateFee', () => {
  it('금액과 비율을 곱한 값을 반환한다', () => {
    expect(calculateFee(10000, 0.1)).toBe(1000);
  });

  it('금액이 0이면 0을 반환한다', () => {
    expect(calculateFee(0, 0.1)).toBe(0);
  });

  it('음수 금액은 음수를 반환한다', () => {
    expect(calculateFee(-1000, 0.1)).toBe(-100);
  });
});
```

---

## JSDoc/TSDoc 규칙

- **모든 함수에 주석을 달지 않는다** — 이름과 타입이 충분히 설명하면 불필요
- **JSDoc이 필요한 경우**: 복잡한 비즈니스 로직, 비직관적 파라미터, 사용 예시가 필요한 유틸
- **주석은 "Why"만** — "What"은 코드가, "How"는 타입이 설명한다

```typescript
// Good: 비직관적 로직에 Why 설명
/**
 * 연체 수수료를 계산한다.
 * 3회 연속 연체 시 2배 수수료가 적용되며,
 * 이는 정책 결정 회의(2025.12)에서 확정된 비즈니스 규칙이다.
 */
function calculateLateFee(user: User, consecutiveLateCount: number): number { ... }

// Bad: 코드가 이미 설명하는 것을 반복
/** 사용자의 이름을 가져온다 */
function getUserName(user: User): string { return user.name; }
```

---

## 자주 쓰는 라이브러리/도구

### 권장

| 용도 | 라이브러리 | 이유 |
|------|-----------|------|
| 스키마 검증 | **zod 4** | TypeScript 네이티브, v4에서 파싱 7~14배 고속화, `zod/mini`(~2KB) 제공. 번들이 극도로 민감하면(엣지 함수) valibot 대안 |
| 날짜 처리 | **date-fns** 또는 **dayjs** | 경량, tree-shaking 가능 |
| 상태 관리 | **zustand** | 간단, 보일러플레이트 최소, TypeScript 친화적 |
| 서버 상태 | **@tanstack/react-query** | 캐싱, 재시도, 무효화 자동 처리 |
| HTTP | **내장 fetch** 또는 **ky** | 대부분의 경우 fetch로 충분 |
| 린트 | **ESLint + typescript-eslint** | 타입 인식 린트 |
| 포맷터 | **Prettier** | ESLint와 포맷 역할 분리 |
| 테스트 | **Vitest 4** | 빠르고 ESM 친화적, Jest 호환 API. 단 **React Native는 Jest가 여전히 표준** |
| TS 스크립트 실행 | **Node 24+ 네이티브** (type stripping, stable) | tsx/ts-node 불필요 — 단 erasable 문법 전제 (`erasableSyntaxOnly`) |

### 피해야 할 것

| 라이브러리 | 이유 | 대안 |
|-----------|------|------|
| `moment.js` | deprecated, 번들 290KB+ | date-fns, dayjs |
| `lodash` (전체) | tree-shaking 불가, 번들 비대 | 개별 import 또는 네이티브 JS |
| `tslint` | deprecated | eslint + typescript-eslint |
| `axios` | 대부분 과도함 | 내장 fetch, ky |
| `class-validator` (단독) | 데코레이터 기반, TS 타입 추론 약함 | zod |

---

## tsconfig.json 권장 설정 (TS 6.x 기준)

> TS 6.0 (2026-03) 부터 `strict`, `moduleResolution: "bundler"`, `target: "es2025"`, `esModuleInterop` 등이 **기본값**이 됐다.
> 따라서 6.x에서는 "기본값 위에 안전 옵션을 추가"하는 구조로 작성한다. (5.x 프로젝트는 아래 옵션을 전부 명시)

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "verbatimModuleSyntax": true,
    "erasableSyntaxOnly": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true,
    "resolveJsonModule": true
  }
}
```

- `strict: true` — 절대 끄지 않는다 (6.x 기본이지만 명시 유지 — 다운그레이드 방지)
- `noUncheckedIndexedAccess: true` — strict에 **포함 안 됨**. 인덱스 접근의 `undefined` 가능성 체크 (실전 함정 #6)
- `verbatimModuleSyntax: true` — `import type` 강제. esbuild/SWC/Vite 파일 단위 트랜스파일 + Node 네이티브 실행 호환 조건
- `erasableSyntaxOnly: true` — enum/namespace 등 코드 생성 문법 차단 (Node 24+ 네이티브 실행 대비, 실전 함정 #2)
- `moduleResolution`: 번들러 프로젝트는 `bundler`(6.x 기본) / **Node로 직접 실행·npm 퍼블리시하는 패키지는 `nodenext`**
- TS 7 (Go 네이티브, ~10배 고속) 전환 임박 — erasable 문법 유지가 최선의 대비책
- 나머지 프로젝트별 설정은 CLAUDE.md에서 정의

---

## 실전 함정 & 지뢰

> 함정 = "정상으로 보이지만 특정 조건에서 깨지는 것". 안티패턴보다 위험 — 경험자도 당한다.

| # | 증상 | 원인 | 해결 | 오답 (하지 말 것) |
|---|------|------|------|------------------|
| 1 | `forEach`/`map` 안의 `await`가 안 기다려짐 — 완료 전에 다음 로직 실행, 에러도 안 잡힘 | `forEach`는 반환된 Promise를 버림. `map`은 `Promise<T>[]`를 반환하는데 값 배열로 착각 | 순차: `for...of` / 병렬: `await Promise.all(items.map(async ...))` | `setTimeout`으로 대기 시간 벌기 |
| 2 | Node 24+에서 `.ts` 직접 실행 시 enum에서 SyntaxError 크래시 | Node 네이티브 실행(type stripping)은 erasable 문법만 지원 — enum/namespace는 JS 코드를 생성하는 문법 | `as const` 객체 + union 타입으로 대체, tsconfig에 `erasableSyntaxOnly: true` | 빌드 스텝을 되살려서 우회 |
| 3 | 오타 속성이 타입 에러 없이 통과 → 런타임에서 조용히 무시됨 | 초과 속성 검사는 **객체 리터럴을 직접 대입할 때만** 동작 — 변수를 거치면 구조적 타이핑으로 통과 | 객체 정의 시점에 `satisfies T` 로 검증 (리터럴 추론도 보존) | `as T` 단언 (검사 자체를 꺼버림) |
| 4 | API 응답 필드 접근에서 런타임 `undefined` 크래시 — 타입 에러는 0개였음 | `JSON.parse`/`response.json()` 반환이 `any` → 이후 코드 전체에 `any` 오염 전파 | `unknown`으로 받고 경계에서 zod 스키마 `safeParse`로 검증 후 사용 | `as ApiResponse` 캐스팅 (검증 없는 믿음) |
| 5 | 날짜가 하루 밀림 (한국 시간대에서 7/3이 7/2로) | `new Date('2026-07-03')` 같은 date-only 문자열은 **UTC 자정**으로 해석 — 로컬 변환 시 전날이 됨 | date-fns `parseISO` + 타임존 정책을 프로젝트 1곳에서 정의 | `getDate() + 1` 보정 (다른 타임존에서 또 깨짐) |
| 6 | `arr[0].name` 런타임 크래시 — 빈 배열 케이스 | 기본 설정에서 인덱스 접근이 `T`로 추론됨 (`undefined` 가능성 은폐) | `noUncheckedIndexedAccess: true` + `arr[0]?.name ?? 기본값` | `arr[0]!.name` (`!`로 경고 침묵) |
| 7 | 외부 입력 검증에서 서버 500/앱 크래시 | zod `parse()`는 실패 시 throw — 경계에서 예외가 그대로 터짐 | 신뢰 경계(API 핸들러, 폼)에서는 `safeParse()` + `success` 분기 | try-catch로 감싸고 빈 객체 반환 (실패가 묻힘) |
| 8 | 번들러에선 되는데 Node 직접 실행에서 "X is not a function" | CJS 패키지의 default export 처리(interop)가 도구마다 다름 | `verbatimModuleSyntax: true`로 import 의도 명시 + 패키지 문서의 권장 import 형태 준수 | `require` 혼용으로 임시 회피 |
| 9 | 존재하지 않는 패키지/API를 import한 코드가 그럴듯하게 생성됨 (AI 협업 시) | LLM 환각 — JS 생태계 환각 import 비율 ~21.7% (연구 기준) | import 추가 시 `package.json` 실존 확인 + `npx tsc --noEmit`으로 즉시 검증 | "그럴듯하니까" 그대로 커밋 |
| 10 | `Object.keys(obj)` 순회에서 `obj[key]` 접근이 타입 에러 — 또는 `as` 지옥 | `Object.keys`는 의도적으로 `string[]` 반환 (구조적 타이핑상 초과 키 가능성 때문) | 키가 닫혀 있음을 보장할 수 있으면 커스텀 `typedKeys<T>()` 헬퍼 1곳 정의, 아니면 `Object.entries` + 값 중심 처리 | 매 사용처에서 `as (keyof T)[]` 단언 반복 |
| 11 | 상태 업데이트 후 다른 화면의 데이터도 같이 바뀜 (원본 오염) | spread(`{...obj}`)는 **얕은 복사** — 중첩 객체/배열은 참조 공유 | 중첩 갱신은 경로마다 spread (불변 업데이트 패턴 참조) 또는 `structuredClone()` | 깊은 곳만 직접 mutate ("한 곳인데 뭐") |

---

## 검증 체크리스트

### 새 기능/모듈 작성 후
- [ ] `npx tsc --noEmit` — 타입 에러 0 (LLM 생성 코드 컴파일 에러의 94%가 타입 체크 실패 — 생성 직후 반드시 실행)
- [ ] `any` / `as ` / `@ts-ignore` / `!.` 신규 추가분 grep — 0건 (있다면 각각 사유 설명 가능해야)
- [ ] 새 import 패키지가 `package.json`에 실존 (환각 import 방지)
- [ ] 외부 입력(API 응답, 폼, URL 파라미터)에 스키마 검증 존재
- [ ] ESLint 통과 (`npx eslint . --max-warnings 0`)

### 타입 에러 디버깅 흐름
1. 에러 메시지의 **마지막 줄**부터 읽기 (핵심 불일치가 거기 있음)
2. 어느 쪽이 진실인지 판정: 타입 정의가 낡았나, 코드가 틀렸나 — **코드를 타입에 맞추는 게 기본**
3. IDE hover로 실제 추론 타입 확인 (추측 금지)
4. 최소 재현 만들기 (해당 타입만 분리)
5. 그래도 안 풀리면 설계 재검토 — **단언(`as`)으로 봉합하지 않는다**

### 리팩토링 후
- [ ] `npx tsc --noEmit` — 기존 타입 에러 0 유지 (리팩토링이 타입 계약을 깨지 않았는지)
- [ ] 기존 테스트 전원 통과 — 동작 보존 확인
- [ ] 삭제한 export를 참조하는 곳 없는지 grep
- [ ] `@ts-expect-error`가 리팩토링으로 불필요해졌으면 제거 (에러 알림이 왔을 것)

### 새 패키지 도입 시
- [ ] 최근 1년 내 릴리즈 존재 + 주간 다운로드/이슈 응답 확인 (유기 프로젝트 회피)
- [ ] `npm audit` 통과 + 라이선스 확인
- [ ] TypeScript 타입 제공 여부 (내장 또는 @types) — 없으면 도입 재고
- [ ] 기존 의존성으로 대체 가능한지 먼저 검토 (번들 비대 방지)

### API 연동 추가 시
- [ ] 응답 스키마(zod) 정의 + 경계에서 `safeParse` (실전 함정 #4, #7)
- [ ] 에러 응답(4xx/5xx) 처리 분기 존재 — happy path만 구현 금지
- [ ] 타임아웃/취소(AbortController) 정책 적용
- [ ] 민감 정보가 로그/에러 메시지에 노출되지 않는지 확인

### 배포 전
- [ ] `strict: true` 등 안전 옵션이 꺼진 곳 없는지 tsconfig diff 확인
- [ ] `npm audit` — 알려진 취약점 0 (high 이상)
- [ ] `.env`가 `.gitignore`에 포함 + 번들에 시크릿 미포함
- [ ] 프로덕션 빌드 실제 실행 확인 (개발 서버와 번들 동작 차이 존재)

---

## 우회/핵 금지 원칙

| 우회 패턴 (금지) | 왜 위험한가 | 정석 해결 |
|-----------------|-----------|----------|
| `as any` / 이중 단언 (`as unknown as T`) | 타입 시스템 전체 무력화 — 이후 모든 사용처가 무검증 | 타입 가드/`satisfies`로 좁히기, 진짜 모르면 `unknown` 유지 |
| `@ts-ignore` | 다음 줄의 **모든** 에러를 영구히 숨김 (다른 에러가 생겨도 침묵) | `@ts-expect-error` + 사유 주석 — 에러가 사라지면 알려줌 |
| `!` non-null assertion 남발 | 가정이 틀리는 순간 프로덕션 크래시 — 컴파일러 경고는 이미 껐음 | 옵셔널 체이닝 + 기본값, early return 타입 가드 |
| 타입 에러를 타입 넓혀서 해결 (`string`으로 도피) | 에러는 사라지지만 잘못된 값이 깊숙이 흘러들어감 | 좁은 타입 유지 + 값을 타입에 맞게 수정 |
| `strict`/`noUncheckedIndexedAccess` 끄기 | 파일 전체의 안전망 제거 — 한 곳 편하자고 전부 포기 | 해당 지점만 정석으로 해결 (옵션은 절대 불변) |
| 테스트 기대값을 구현에 맞춰 수정 | 검증 무력화 — 버그가 스펙이 됨 | 구현을 고치거나, 스펙 변경이면 사유를 커밋에 명시 |

---

## 프로젝트별 확장 포인트

> 아래 항목들은 프로젝트마다 다르므로 `/CLAUDE.md`에서 정의합니다:
> - 프로젝트 구조 상세 및 path alias 설정
> - 빌드/실행 명령
> - tsconfig.json 세부 설정 (target, module, jsx 등)
> - CI/CD 설정
> - 외부 서비스 연동
> - 환경별 설정 (dev/staging/prod)
> - 린트/포맷 규칙 커스터마이징
> - 테스트 프레임워크 설정

<!-- 개선 이력
- 2026-07-03: /ScoreRules 독립 채점 55(상한)→1차 50→보강→**76점 Level 3** (게이트 전 카테고리 통과).
  함정 11개(4요소)/체크리스트 5종+디버깅 흐름/우회금지 6행/에러 경계 기준/|| vs ?? 쌍 등 보강.
- 2026-07-03: 전면 보강 (55점 상한 해소) — 실전 함정 & 지뢰 9개(4요소)/검증 체크리스트 3종+디버깅 흐름/우회금지 6행 신설.
  TS 6.0 기준 현대화: tsconfig 재작성(verbatimModuleSyntax·erasableSyntaxOnly), enum 지양 확정(as const 대체 패턴),
  satisfies 관용구, React Compiler 시대 메모이제이션 지침 전환, Zod 4/Vitest 4/Node 네이티브 실행 반영.
  안티패턴 표 "이유"→"실패 결과"(실패 체인) 전환 + AI 실수 4행 추가(환각 import 등). 근거: 2026-07 웹 리서치 (출처 확보).
-->

