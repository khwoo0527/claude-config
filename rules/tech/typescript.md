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
│   └── schedule/
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
- **타입/인터페이스**: PascalCase (`UserProfile`, `ScheduleEvent`)
- **Enum**: PascalCase + 멤버도 PascalCase (`enum UserRole { Admin, Member, Guest }`)
- **제네릭 파라미터**: 단일 대문자 또는 T 접두어 (`T`, `TData`, `TError`, `K`, `V`)

### 의미 기반 접두어/접미어

| 종류 | 접두어/접미어 | 예시 |
|------|-------------|------|
| boolean 변수 | `is`, `has`, `can`, `should` | `isLoading`, `hasPermission`, `canEdit` |
| 이벤트 핸들러 | `handle` + 동사 | `handleSubmit`, `handlePress`, `handleChange` |
| 콜백 prop | `on` + 동사 | `onSubmit`, `onPress`, `onChange` |
| Props 타입 | `~Props` | `ScheduleCardProps`, `MemberListProps` |
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
import type { ScheduleEvent } from './types';
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
  court: string;
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
async function sendNotificationToMembers(clubId: string, message: string): Promise<void> {
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
// 옵셔널 체이닝 + nullish coalescing
const displayName = user?.profile?.name ?? '알 수 없음';
const pageSize = config.pagination?.pageSize ?? DEFAULT_PAGE_SIZE;

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
| 안티패턴 | 이유 | 대안 |
|---------|------|------|
| `any` 사용 | 타입 시스템 무력화 | `unknown` + 타입 가드 |
| `as` 타입 단언 남용 | 런타임 에러 숨김 | 타입 가드 또는 제네릭 |
| `// @ts-ignore` | 타입 에러 무시 | 근본 원인 해결 |
| `!` non-null assertion | null 런타임 에러 위험 | 옵셔널 체이닝 / 타입 가드 |
| 빈 인터페이스 `{}` | 모든 값을 허용 | `Record<string, never>` 또는 구체적 타입 |
| `object` 타입 | 너무 넓음 | 구체적 인터페이스 정의 |
| 숫자 enum | 직렬화 시 의미 상실 | 문자열 enum 또는 const object |
| `namespace` | 레거시 패턴 | ES 모듈 사용 |

### 코드 구조 관련
| 안티패턴 | 이유 | 대안 |
|---------|------|------|
| `index.ts`에 로직 작성 | 디버깅 어려움, 순환 참조 유발 | re-export만 |
| default export 남용 | 리팩토링 어려움, 자동완성 불리 | named export 우선 |
| 클래스 남용 | 불필요한 복잡성 | 함수 + 타입으로 충분한 경우가 대부분 |
| 콜백 지옥 | 가독성 최악 | async/await |
| 매직 넘버/문자열 | 의미 불명확 | 상수 또는 enum으로 정의 |
| 거대한 파일 (300줄+) | 유지보수 어려움 | 기능별로 분리 |

### AI가 흔히 생성하는 실수
| 실수 | 수정 방법 |
|------|----------|
| `console.log` 디버깅 코드 남기기 | 프로젝트 로거 사용 또는 제거 |
| `catch(e: any)` | `catch(e: unknown)` 후 타입 체크 |
| 불필요한 `async` (await 없는 async 함수) | `async` 제거, 그냥 Promise 반환 |
| `useEffect` 내 직접 async | 내부에 별도 함수 정의 후 호출 |
| 타입을 컴포넌트에 인라인 정의 | `types.ts`로 분리 |
| `==` 사용 | 항상 `===` 사용 |
| `var` 사용 | `const` (기본) 또는 `let` |
| 배열 `.forEach` 내 `await` | `for...of` 또는 `Promise.all` |
| `new Array()` | 배열 리터럴 `[]` 사용 |
| `String()`/`Number()` 남용 | 명확한 파싱: `parseInt`, `parseFloat`, 템플릿 리터럴 |

---

## 에러/예외 처리

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

---

## 성능 최적화

### 렌더링 (React/React Native)
- `React.memo`: props가 자주 바뀌지 않는 컴포넌트에 적용 — 단, **측정 후** 적용
- `useMemo`: 비용이 큰 계산에만 사용 (단순 객체/배열 생성에 과용 금지)
- `useCallback`: 자식 컴포넌트에 콜백을 전달할 때만 — memo와 함께 써야 의미 있음
- 렌더링 루프 내 인라인 객체/배열/함수 생성 지양 (`style={{}}`, `onPress={() => {}}`)
- 불필요한 상태(state) 만들지 않기 — 파생 가능한 값은 계산으로 처리

### 데이터
- 대용량 리스트: 가상화 적용 (`FlashList`, `react-window`)
- 이미지: 적절한 사이즈로 리사이즈 + lazy loading
- 페이지네이션/무한 스크롤: 전체 데이터를 한번에 로드하지 않기

### 번들
- tree-shaking 가능한 import: `import { debounce } from 'lodash-es'`
- `lodash` 전체 import 절대 금지 → 개별 import 또는 네이티브 대체
- 동적 import(`lazy`)로 코드 분할: 초기 로드에 필요 없는 화면/기능

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
 * 노쇼 패널티를 계산한다.
 * 3회 연속 노쇼 시 2배 패널티가 적용되며,
 * 이는 운영진 회의(2025.12)에서 결정된 정책이다.
 */
function calculateNoShowPenalty(user: User, consecutiveNoShows: number): number { ... }

// Bad: 코드가 이미 설명하는 것을 반복
/** 사용자의 이름을 가져온다 */
function getUserName(user: User): string { return user.name; }
```

---

## 자주 쓰는 라이브러리/도구

### 권장

| 용도 | 라이브러리 | 이유 |
|------|-----------|------|
| 스키마 검증 | **zod** | TypeScript 네이티브, 타입 추론 자동, 에러 메시지 커스터마이징 |
| 날짜 처리 | **date-fns** 또는 **dayjs** | 경량, tree-shaking 가능 |
| 상태 관리 | **zustand** | 간단, 보일러플레이트 최소, TypeScript 친화적 |
| 서버 상태 | **@tanstack/react-query** | 캐싱, 재시도, 무효화 자동 처리 |
| HTTP | **내장 fetch** 또는 **ky** | 대부분의 경우 fetch로 충분 |
| 린트 | **ESLint + typescript-eslint** | 타입 인식 린트 |
| 포맷터 | **Prettier** | ESLint와 포맷 역할 분리 |
| 테스트 | **Vitest** | 빠르고 ESM 친화적, Jest 호환 API |

### 피해야 할 것

| 라이브러리 | 이유 | 대안 |
|-----------|------|------|
| `moment.js` | deprecated, 번들 290KB+ | date-fns, dayjs |
| `lodash` (전체) | tree-shaking 불가, 번들 비대 | 개별 import 또는 네이티브 JS |
| `tslint` | deprecated | eslint + typescript-eslint |
| `axios` | 대부분 과도함 | 내장 fetch, ky |
| `class-validator` (단독) | 데코레이터 기반, TS 타입 추론 약함 | zod |

---

## tsconfig.json 권장 설정

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true
  }
}
```

- `strict: true` — 절대 끄지 않는다
- `noUncheckedIndexedAccess: true` — 배열/객체 접근 시 `undefined` 가능성 체크
- `noImplicitReturns: true` — 모든 분기에서 반환값 보장
- 나머지 프로젝트별 설정은 CLAUDE.md에서 정의

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
