---
paths:
  - "**/*.tsx"
  - "app.json"
  - "app.config.ts"
  - "babel.config.js"
  - "metro.config.js"
---

# React Native + Expo 개발 규칙

> 이 파일은 React Native (Expo) 프로젝트에 범용적으로 적용되는 시니어 수준 규칙입니다.
> TypeScript 기본 규칙은 `rules/tech/typescript.md`를 참조합니다. 여기서는 React Native/Expo 특화 규칙만 다룹니다.
> 프로젝트별 설정, 빌드 명령, 폴더 구조는 `/CLAUDE.md`에 정의합니다.

---

## 1. 핵심 원칙

- **모바일 퍼스트**: 모바일 경험이 최우선. 웹은 보조 — 터치 영역(44x44pt), 스크롤 성능, 오프라인 대응을 항상 고려한다.
- **네이티브 느낌**: 웹을 모바일에 옮긴 것이 아니라, 네이티브 앱처럼 느껴져야 한다 (애니메이션, 제스처, 햅틱).
- **성능 = UX**: 60fps 유지가 기본. 렌더링 병목, 메모리 릭, 과도한 리렌더를 적극적으로 방지한다.
- **플랫폼 차이 인식**: iOS/Android/Web은 동작이 다르다. `Platform.OS`로 분기가 필요한 지점을 미리 파악한다.
- **Expo 생태계 우선**: 가능하면 Expo SDK 모듈을 사용한다. 네이티브 모듈이 꼭 필요한 경우만 개발 빌드(dev client)로 전환한다.
- **4가지 상태 필수**: 모든 화면은 로딩/에러/빈 데이터/정상 상태를 반드시 처리한다.
- **New Architecture 전제**: 신규 코드는 New Architecture(Fabric + TurboModules) 위에서 동작한다 — 구 아키텍처 전용 API/패턴을 생성하지 않는다 (다음 섹션).

---

## 2. New Architecture (신규 프로젝트 기본 전제)

> RN 0.76+에서 기본 활성, **0.82+부터는 옵트아웃 설정 자체가 무시**되며 Expo SDK 55에서 레거시 아키텍처가 완전히 제거됐다.
> 신규 코드는 New Architecture를 전제로 작성한다 — `newArchEnabled: false`로 돌아갈 길은 없다.

### 달라지는 동작: state 배칭

레거시에서는 `setTimeout`/네이티브 이벤트 콜백 안의 setState가 각각 렌더를 유발했지만,
New Architecture에서는 **모든 setState가 항상 배칭**된다. "중간 렌더"에 의존하던 코드가 조용히 깨진다.

```typescript
// ❌ Bad: 연속 setState 사이의 "중간 렌더"에 의존 — 레거시에서 우연히 동작하던 코드
setTimeout(() => {
  setHighlight(true);   // 레거시: 렌더 1회 발생 → 하이라이트가 잠깐 보임
  setHighlight(false);  // New Arch: 두 호출이 배칭 → 하이라이트가 아예 안 보임
}, 0);

// ✅ Good: 시간차 UI는 명시적으로 스케줄링 (짧은 강조 효과는 애니메이션 API가 정석)
setHighlight(true);
setTimeout(() => setHighlight(false), 300);
```

### Interop layer는 마이그레이션 유예 장치일 뿐

- 레거시 네이티브 모듈은 interop layer로 당분간 동작하지만 **한시적이며 concurrent 기능 미지원**. "interop으로 돌아가니까 OK"는 라이브러리 선택 기준이 될 수 없다.
- 서드파티 호환이 New Arch의 최대 난관 — 새 라이브러리 도입 전 반드시 호환 확인.

### 라이브러리 호환 확인법

- [ ] reactnative.directory 에서 "Supports New Architecture" 필터로 확인
- [ ] `npx expo-doctor` — 의존성/설정 문제 일괄 진단
- [ ] README/최근 릴리즈 노트에서 New Architecture 지원 명시 확인 — 장기 미갱신(1년+) 라이브러리는 도입 재고

### 구 아키텍처 전용 API 회피 목록

| 구 API (생성 금지) | 대체 |
|---|---|
| `SafeAreaView` (react-native 코어 — 0.81 deprecated) | `react-native-safe-area-context` |
| `UIManager.setLayoutAnimationEnabledExperimental` + `LayoutAnimation` | Reanimated entering/exiting/layout 애니메이션 |
| `findNodeHandle` + `UIManager` 직접 조작 | ref 직접 메서드 호출 (`ref.current?.measure(...)`) |
| `Clipboard`, `PushNotificationIOS` (코어에서 추출됨) | `expo-clipboard`, `expo-notifications` |

---

## 3. 프로젝트 구조 (Expo Router 기반)

```
{project-root}/
├── app/                        # Expo Router — 파일 기반 라우팅
│   ├── _layout.tsx             #   루트 레이아웃 (Provider, 글로벌 설정)
│   ├── index.tsx               #   진입점
│   ├── (auth)/                 #   인증 관련 그룹
│   │   ├── _layout.tsx
│   │   ├── login.tsx
│   │   └── register.tsx
│   ├── (tabs)/                 #   탭 네비게이션 그룹
│   │   ├── _layout.tsx
│   │   ├── home.tsx
│   │   ├── search.tsx
│   │   └── profile.tsx
│   └── item/
│       ├── [id]/
│       │   ├── _layout.tsx
│       │   └── index.tsx
│       └── create.tsx
├── src/
│   ├── features/               # 기능별 폴더 (co-location)
│   │   ├── auth/
│   │   │   ├── hooks/
│   │   │   ├── components/
│   │   │   └── types.ts
│   │   ├── item/
│   │   └── notification/
│   ├── shared/
│   │   ├── components/         # 공유 UI (Button, Card, Modal, Toast 등)
│   │   ├── hooks/              # 공유 훅 (useToast, useDebounce 등)
│   │   ├── utils/              # 공유 유틸 (formatDate, formatPrice 등)
│   │   ├── constants/          # 전역 상수 (MAX_CONTENT_WIDTH 등)
│   │   ├── types/              # 전역 타입
│   │   └── styles/             # 테마, 색상, 타이포그래피
│   ├── services/               # 외부 서비스 연동 (API, 푸시 등)
│   └── config/                 # 앱 설정 (env.ts 등)
├── assets/                     # 이미지, 폰트 등 정적 리소스
├── app.json / app.config.ts
├── babel.config.js
├── metro.config.js
├── tsconfig.json
├── tailwind.config.js
└── package.json
```

- `app/`: Expo Router 전용 — 라우팅(화면 정의)만 담당. 비즈니스 로직 넣지 않음
- `src/features/`: 각 기능의 컴포넌트, 훅, 타입, 유틸 — co-location 원칙
- `src/shared/`: 2개 이상 기능에서 공유하는 것만 여기에
- 컴포넌트가 1개 기능에서만 사용되면 `shared/`가 아닌 해당 `features/` 안에 배치

---

## 4. 네이밍 컨벤션

> TypeScript 기본 네이밍은 `typescript.md` 참조. 여기서는 React Native 특화만.

### 파일/폴더

| 종류 | 컨벤션 | 예시 |
|------|--------|------|
| 컴포넌트 파일 | PascalCase | `ItemCard.tsx`, `MemberList.tsx` |
| 화면(스크린) 파일 | kebab-case (Expo Router) | `app/item/[id].tsx`, `app/(tabs)/home.tsx` |
| 훅 파일 | camelCase with `use` | `useAuth.ts`, `useItemList.ts` |
| 유틸 파일 | camelCase | `formatDate.ts`, `validators.ts` |
| 타입 파일 | camelCase | `types.ts`, `schemas.ts` |
| 테스트 파일 | 소스파일명 + `.test` | `ItemCard.test.tsx` |
| 이미지 에셋 | kebab-case | `default-avatar.png`, `logo-dark.png` |
| 플랫폼별 파일 | `.{platform}` 접미어 | `DatePicker.ios.tsx`, `DatePicker.android.tsx` |

### 코드 네이밍

| 종류 | 컨벤션 | 예시 |
|------|--------|------|
| 컴포넌트 | PascalCase, 역할 명확 | `ItemCard`, `EmptyListView` |
| 화면 컴포넌트 | PascalCase + Screen (선택) | `DetailScreen`, `CreateScreen` |
| 레이아웃 | PascalCase + Layout | `TabLayout`, `AuthLayout` |
| 커스텀 훅 | use + 동사/명사 | `useAuth`, `useItemList`, `useDebounce` |
| 이벤트 핸들러 | handle + 동사 | `handlePress`, `handleSubmit`, `handleRefresh` |
| 콜백 prop | on + 동사 | `onPress`, `onSubmit`, `onChange` |
| boolean prop | is/has/can/should 접두어 | `isLoading`, `hasError`, `canEdit` |
| boolean 상태 | is/has/show 접두어 | `isVisible`, `hasPermission`, `showModal` |
| 스타일 | camelCase | `containerStyle`, `headerWrapper` |
| 상수 | UPPER_SNAKE_CASE | `MAX_CONTENT_WIDTH`, `API_TIMEOUT` |
| enum 대체 (`as const` union) 값 | PascalCase 키 | `UserRole.Admin`, `Status.Active` |

---

## 5. 코드 포맷팅

> TypeScript 기본 포맷팅(`typescript.md`)을 따르되, 아래 React Native 특화 규칙을 추가 적용.

| 규칙 | 값 | 이유 |
|------|-----|------|
| 들여쓰기 | 2 spaces | RN 생태계 표준 |
| 줄 길이 | 100자 (JSX는 120자까지 허용) | 모바일 개발 시 긴 className 고려 |
| 따옴표 | 작은따옴표 (JSX 속성은 큰따옴표) | Prettier 기본값 |
| 세미콜론 | 있음 | 명확성 |
| 후행 쉼표 | all | git diff 깔끔 |
| JSX 속성 줄바꿈 | 3개 초과 시 | 가독성 |
| 빈 줄 | 섹션 간 1줄 | 과도한 빈 줄 금지 |

```typescript
// Good: 속성이 많으면 줄바꿈
<ItemCard
  item={item}
  onPress={handlePress}
  isSelected={isSelected}
  showBadge={true}
/>

// Good: 속성이 적으면 한 줄
<Badge text="Active" variant="success" />

// Bad: 한 줄에 속성 4개 이상
<ItemCard item={item} onPress={handlePress} isSelected={isSelected} showBadge={true} />

// JSX 내 삼항: 간단한 경우만 인라인
<Text>{isActive ? 'Active' : 'Inactive'}</Text>

// 복잡한 조건부 렌더링: 변수로 추출
const statusBadge = isActive
  ? <Badge text="Active" variant="success" />
  : <Badge text="Inactive" variant="warning" />;

return <View>{statusBadge}</View>;
```

---

## 6. import/export 규칙

```typescript
// 1. React / React Native 코어
import { useState, useEffect } from 'react';
import { View, Text, Pressable, Platform, ScrollView } from 'react-native';

// 2. Expo SDK
import * as Notifications from 'expo-notifications';
import { Image } from 'expo-image';
import { useRouter, useLocalSearchParams } from 'expo-router';

// 3. 외부 라이브러리
import { useQuery, useMutation } from '@tanstack/react-query';
import { useForm, Controller } from 'react-hook-form';
import { z } from 'zod';

// 4. 내부 절대 경로 (@/ alias)
import { useAuth } from '@/features/auth/hooks/useAuth';
import { Button } from '@/shared/components/Button';
import { colors, spacing } from '@/shared/styles/theme';
import { MAX_CONTENT_WIDTH } from '@/shared/constants';

// 5. 상대 경로
import { ItemCard } from './components/ItemCard';
import { useItemList } from './hooks/useItemList';

// 6. 타입 전용 import (항상 마지막)
import type { Item, ItemStatus } from './types';
```

### export 규칙

> named export 우선/`import type` 등 일반 규칙 → `typescript.md` 참조. RN 특화만:

- **화면 컴포넌트만 `export default`** (Expo Router가 요구) — 그 외는 named export
- **배럴 파일(index.ts)**: 기능 폴더 단위 re-export 전용, 2단계 이상 깊이 금지

---

## 7. 파일 내부 코드 순서 (컴포넌트 파일)

```typescript
// ===== 1. import 문 (위 순서 준수) =====

// ===== 2. 타입 정의 (Props, 내부 타입) =====
interface ItemCardProps {
  item: Item;
  onPress: (id: string) => void;
  isSelected?: boolean;
}

// ===== 3. 상수 =====
const ANIMATION_DURATION = 200;

// ===== 4. 메인 컴포넌트 (export) =====
export function ItemCard({ item, onPress, isSelected = false }: ItemCardProps) {
  // 4-1. 훅 (useState, useRef, 커스텀 훅)
  const [isExpanded, setIsExpanded] = useState(false);
  const { theme } = useTheme();

  // 4-2. 파생 값 — 렌더 중 계산 (React Compiler가 자동 메모이제이션, 수동 useMemo 불필요)
  const formattedDate = formatDate(item.createdAt);

  // 4-3. 핸들러 (수동 useCallback 불필요 — 컴파일러 위임)
  const handlePress = () => onPress(item.id);

  // 4-4. 부수 효과 (useEffect) — 최소화
  useEffect(() => {
    // 구독 등 필요한 경우만
    return () => { /* cleanup */ };
  }, []);

  // 4-5. 조기 반환 (로딩, 에러, 빈 상태)
  if (!item) return null;

  // 4-6. JSX 반환
  return (
    <Pressable onPress={handlePress}>
      <Text>{item.title}</Text>
    </Pressable>
  );
}

// ===== 5. 하위 컴포넌트 (이 파일 내에서만 사용) =====
function ItemBadge({ status }: { status: ItemStatus }) {
  return <Badge text={status} />;
}

// ===== 6. 스타일 (StyleSheet.create) =====
const styles = StyleSheet.create({
  container: { flex: 1 },
});
```

---

## 8. 컴포넌트 설계 패턴

### 기본 컴포넌트

```typescript
interface ItemCardProps {
  item: Item;
  onPress: (itemId: string) => void;
  isHighlighted?: boolean;
}

export function ItemCard({ item, onPress, isHighlighted = false }: ItemCardProps) {
  const handlePress = () => onPress(item.id); // React Compiler가 최적화 — useCallback 불필요

  return (
    <Pressable
      onPress={handlePress}
      accessibilityRole="button"
      accessibilityLabel={`${item.title} 상세 보기`}
      style={({ pressed }) => [styles.card, pressed && styles.cardPressed]}
    >
      <Text className="text-base font-bold text-gray-900">{item.title}</Text>
      <Text className="text-sm text-gray-500 mt-1">{item.description}</Text>
      {isHighlighted && <Badge text="Featured" variant="success" />}
    </Pressable>
  );
}
```

### 컴포넌트 분리 기준

- **100줄 초과** -> 하위 컴포넌트로 분리
- **재사용 가능** -> `shared/components/`로 이동
- **조건부 렌더링이 복잡** -> 별도 컴포넌트로 추출
- **리스트 아이템** -> 반드시 별도 컴포넌트 (렌더 단위 격리 — 컴파일러 최적화 경계 + 재사용)

### 합성(Composition) 패턴

```typescript
// Good: 합성으로 유연한 구조
<Card>
  <Card.Header title="Item Detail" />
  <Card.Body>
    <ItemInfo item={item} />
  </Card.Body>
  <Card.Footer>
    <ActionButton onPress={handleAction} />
  </Card.Footer>
</Card>

// 합성 컴포넌트 구현 — 정적 프로퍼티로 하위 컴포넌트 부착
function Card({ children }: PropsWithChildren) {
  return <View className="bg-white rounded-xl border border-gray-200">{children}</View>;
}
Card.Header = function CardHeader({ title }: { title: string }) {
  return <View className="px-4 py-3 border-b border-gray-100"><Text className="text-lg font-bold">{title}</Text></View>;
};
Card.Body = function CardBody({ children }: PropsWithChildren) {
  return <View className="p-4">{children}</View>;
};
// Card.Footer 등 동일 패턴
```

### 조건부 렌더링

```typescript
// Good: early return으로 상태별 처리
export default function ListScreen() {
  const query = useItemList();

  if (query.isLoading) return <LoadingSkeleton />;
  if (query.error) return <ErrorView error={query.error} onRetry={query.refetch} />;
  if (!query.data?.length) return <EmptyView message="No items yet" />;

  return <ItemList items={query.data} />;
}

// Bad: JSX 내 복잡한 중첩 삼항
return (
  <View>
    {isLoading ? <Spinner /> : error ? <Error /> : data ? <List /> : <Empty />}
  </View>
);
```

---

## 9. 훅(Hooks) 규칙

### 기본 규칙
- 훅은 컴포넌트/훅의 **최상위**에서만 호출 (조건문/루프 안 금지)
- 커스텀 훅은 `use` 접두어 필수
- 하나의 훅은 **하나의 관심사**만 담당
- 훅이 3개 이상의 상태를 관리하면 커스텀 훅으로 추출

### useState

```typescript
// Good: 관련 상태는 객체로 묶기
const [form, setForm] = useState<CreateItemForm>({
  title: '',
  description: '',
  maxCount: 8,
  price: 0,
});

// 불변 업데이트 패턴
setForm(prev => ({ ...prev, title: 'New Title' }));

// Bad: 관련 상태를 개별로 선언
const [title, setTitle] = useState('');
const [description, setDescription] = useState('');
const [maxCount, setMaxCount] = useState(8);
```

### useEffect

```typescript
// 규칙: useEffect는 최소화
// 데이터 fetching -> react-query 사용
// 이벤트 구독 -> useEffect OK (cleanup 필수!)
// 파생 값 계산 -> 렌더링 중 계산 (React Compiler가 최적화 — 수동 useMemo 지양, § 성능 참조)

// Good: cleanup 포함한 구독
useEffect(() => {
  const subscription = eventEmitter.addListener('update', handleUpdate);
  return () => subscription.remove(); // cleanup 필수!
}, [handleUpdate]);

// Good: isMounted 패턴 (setState after unmount 방지)
useEffect(() => {
  let isMounted = true;

  async function loadData() {
    const result = await fetchData();
    if (isMounted) {
      setData(result); // unmount 후 setState 방지
    }
  }

  loadData();
  return () => { isMounted = false; };
}, []);

// Bad: cleanup 누락 (메모리 릭)
useEffect(() => {
  eventEmitter.addListener('update', handleUpdate);
  // return 없음 -> 리스너 누적!
}, []);

// Bad: useEffect 내 직접 async
useEffect(async () => { // 에러! useEffect는 async 함수를 받지 않음
  const data = await fetchData();
}, []);
```

### useCallback / useMemo / React.memo — React Compiler 시대의 규칙

> **Expo SDK 54+는 React Compiler가 기본 활성** — 컴파일러가 자동 메모이제이션한다.
> 수동 `useMemo`/`useCallback`/`React.memo`는 이제 노이즈 + 의존성 배열 버그 표면일 뿐이다.

```typescript
// ❌ Bad: 습관적 수동 메모이제이션 — 컴파일러와 중복, 의존성 누락 시 stale closure 버그
const handlePress = useCallback((id: string) => router.push(`/item/${id}`), [router]);
const sorted = useMemo(() => [...items].sort(byDate), [items]);

// ✅ Good: 그냥 쓴다 — 컴파일러가 최적화
const handlePress = (id: string) => router.push(`/item/${id}`);
const sorted = [...items].sort(byDate);
```

**수동 메모이제이션이 정당한 예외** (이 경우만 허용):
- React Compiler 비활성 레거시 프로젝트 — 이때도 **측정 후** 선별 적용
- `eslint-plugin-react-compiler`가 "최적화 불가"로 지적하는 컴포넌트 — 단, 먼저 Rules of React 위반(조건부 훅, 렌더 중 부수효과)을 수정하는 것이 정석

### 커스텀 훅 패턴

```typescript
// Good: 하나의 관심사, 명확한 반환 타입
function useItemList(categoryId: string) {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['items', categoryId],
    queryFn: () => fetchItems(categoryId),
    enabled: !!categoryId,
  });

  return {
    items: data ?? [],
    isLoading,
    error,
    refetch,
  } as const;
}

// Good: mutation도 훅으로 캡슐화 — onSuccess(무효화+이동)/onError(토스트)까지 포함해 반환
function useCreateItem() {
  const queryClient = useQueryClient();
  const { mutate, isPending } = useMutation({
    mutationFn: createItem,
    onSuccess: (newItem) => queryClient.invalidateQueries({ queryKey: ['items'] }),
    onError: (error) => showToast(getErrorMessage(error)),
  });
  return { create: mutate, isCreating: isPending };
}
```

---

## 10. 폼 패턴 (react-hook-form + zod)

### zod 스키마 정의

```typescript
import { z } from 'zod';

// 스키마를 컴포넌트 외부에 정의 (재사용 + 렌더링 시 재생성 방지)
const createItemSchema = z.object({
  title: z.string().min(2, '제목은 2자 이상 입력해주세요').max(50, '제목은 50자 이하로 입력해주세요'),
  description: z.string().max(500, '설명은 500자 이하로 입력해주세요').optional(),
  maxCount: z.number().min(2, '최소 2 이상').max(100, '최대 100'),
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/, '올바른 날짜 형식이 아닙니다'),
});

// 스키마에서 타입 추출
type CreateItemForm = z.infer<typeof createItemSchema>;
```

### Controller 패턴 (TextInput)

```typescript
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

export function CreateItemScreen() {
  const { create, isCreating } = useCreateItem();

  const {
    control,
    handleSubmit,
    formState: { errors, isValid },
  } = useForm<CreateItemForm>({
    resolver: zodResolver(createItemSchema),
    defaultValues: { title: '', description: '', maxCount: 8, date: '' },
    mode: 'onBlur', // 포커스 해제 시 검증 (타이핑 중 에러 방지)
  });

  const onSubmit = handleSubmit((data) => {
    create(data);
  });

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <KeyboardAvoidingView behavior={Platform.OS === 'ios' ? 'padding' : 'height'} style={{ flex: 1 }}>
        <ScrollView contentContainerStyle={{ padding: 16 }} keyboardShouldPersistTaps="handled">
          <Controller
            control={control}
            name="title"
            render={({ field: { onChange, onBlur, value } }) => (
              <View className="mb-4">
                <Text className="text-sm font-medium text-gray-700 mb-1">Title</Text>
                <TextInput
                  className="border border-gray-300 rounded-lg px-3 py-2 text-base"
                  value={value}
                  onChangeText={onChange}
                  onBlur={onBlur}
                  maxLength={50}
                />
                {errors.title && <Text className="text-red-500 text-xs mt-1">{errors.title.message}</Text>}
              </View>
            )}
          />
          {/* 숫자 입력: 증감 버튼(onChange(Math.max(2, value - 1))) 또는 keyboardType="numeric" */}
          <Button title={isCreating ? 'Creating...' : 'Create'} onPress={onSubmit} disabled={!isValid || isCreating} />
        </ScrollView>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}
```

### 공통 폼 입력 컴포넌트

```typescript
// 재사용 폼 필드 래퍼 — 라벨 + children + 에러 텍스트를 한 곳에서
function FormField({ label, error, children }: { label: string; error?: string; children: ReactNode }) {
  return (
    <View className="mb-4">
      <Text className="text-sm font-medium text-gray-700 mb-1">{label}</Text>
      {children}
      {error && <Text className="text-red-500 text-xs mt-1">{error}</Text>}
    </View>
  );
}
// 사용: <FormField label="Title" error={errors.title?.message}><Controller ... /></FormField>
```

### 폼 규칙

- 스키마는 컴포넌트 밖에 정의 (렌더링 시 재생성 방지)
- `mode: 'onBlur'` 기본 — 타이핑 중 에러 메시지 깜빡임 방지
- 숫자 입력은 `TextInput` + `keyboardType="numeric"` 또는 증감 버튼
- 제출 중 버튼 비활성화 (`disabled={!isValid || isPending}`)
- `keyboardShouldPersistTaps="handled"` — 키보드 열린 상태에서 버튼 탭 가능
- 에러 메시지는 필드 바로 아래 빨간 텍스트로 표시

---

## 11. 상태 관리 (zustand + react-query)

### 역할 분리 원칙

| 데이터 종류 | 도구 | 예시 |
|------------|------|------|
| 서버 상태 (DB 데이터) | **react-query** | 아이템 목록, 프로필, 알림 |
| UI 상태 (화면 내) | **useState** | 모달 열림, 폼 입력값, 토글 |
| 글로벌 클라이언트 상태 | **zustand** | 인증 세션, 앱 설정, 테마 |
| URL 상태 | **expo-router params** | 현재 선택된 ID, 필터 |

> **원칙**: 서버에서 온 데이터는 반드시 react-query로 관리한다. zustand에 서버 데이터를 복사하지 않는다.

### 로컬 스토리지 선택 기준

| 데이터 | 저장소 | 이유 |
|------|------|------|
| 상태 persist (zustand 등) | **react-native-mmkv** | 동기 API, AsyncStorage 대비 ~30배 빠름 |
| 토큰/비밀번호/민감 정보 | **expo-secure-store** | iOS Keychain / Android Keystore 암호화 |
| AsyncStorage API가 필요한 기존 코드 | **expo-sqlite/kv-store** | AsyncStorage 호환 드롭인 대체 |
| 쿼리가 필요한 구조화 데이터 | **expo-sqlite** | SQL, 오프라인 캐시 |

> `@react-native-async-storage/async-storage` 신규 도입 지양 — 느린 비동기 왕복 + 암호화 없음. 위 표의 대체재를 쓴다.

### zustand 스토어 패턴 (persist는 MMKV)

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage, type StateStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';

const mmkv = new MMKV();

const mmkvStorage: StateStorage = {
  setItem: (name, value) => mmkv.set(name, value),
  getItem: (name) => mmkv.getString(name) ?? null,
  removeItem: (name) => mmkv.delete(name),
};

// 슬라이스 인터페이스 (기능별로 분리해 합성)
interface AuthSlice {
  session: Session | null;
  setSession: (session: Session | null) => void;
  clearSession: () => void;
}

interface SettingsSlice {
  theme: 'light' | 'dark' | 'system';
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
}

interface AppStore extends AuthSlice, SettingsSlice {}

export const useAppStore = create<AppStore>()(
  persist(
    (set) => ({
      session: null,
      setSession: (session) => set({ session }),
      clearSession: () => set({ session: null }),
      theme: 'system',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'app-store',
      storage: createJSONStorage(() => mmkvStorage),
      // 영속할 상태만 선택 — session(토큰 포함)은 secure-store에 별도 저장, persist 금지
      partialize: (state) => ({ theme: state.theme }),
    },
  ),
);
```

### zustand 셀렉터 (리렌더 최적화)

```typescript
// Good: 필요한 값만 셀렉터로 구독 -> 해당 값 변경 시에만 리렌더
const theme = useAppStore((state) => state.theme);
const session = useAppStore((state) => state.session);

// Good: 파생 값 셀렉터 (shallow 비교 필요)
import { useShallow } from 'zustand/react/shallow';

const { isLoggedIn, userName } = useAppStore(
  useShallow((state) => ({
    isLoggedIn: !!state.session,
    userName: state.session?.user?.name ?? 'Guest',
  })),
);

// Bad: 전체 스토어 구독 -> 어떤 값이든 변경 시 리렌더
const store = useAppStore(); // 모든 상태 변경에 리렌더!
```

### react-query 패턴

```typescript
// QueryClient 설정 (루트 레이아웃에서)
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,    // 5분 — 과도한 리페치 방지
      gcTime: 30 * 60 * 1000,      // 30분 캐시 유지 (구 cacheTime)
      retry: 2,
      refetchOnWindowFocus: false,   // 모바일에서는 불필요
    },
    mutations: {
      retry: 0, // mutation은 재시도 않음
    },
  },
});

// 쿼리 키 팩토리 패턴
const itemKeys = {
  all: ['items'] as const,
  lists: () => [...itemKeys.all, 'list'] as const,
  list: (filters: ItemFilters) => [...itemKeys.lists(), filters] as const,
  details: () => [...itemKeys.all, 'detail'] as const,
  detail: (id: string) => [...itemKeys.details(), id] as const,
};

// 사용
const { data } = useQuery({
  queryKey: itemKeys.detail(itemId),
  queryFn: () => fetchItem(itemId),
});

// 관련 캐시 전체 무효화
queryClient.invalidateQueries({ queryKey: itemKeys.all });
```

### 낙관적 업데이트 패턴

```typescript
function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (itemId: string) => toggleFavoriteApi(itemId),
    onMutate: async (itemId) => {
      // 1. 진행 중인 리페치 취소
      await queryClient.cancelQueries({ queryKey: itemKeys.detail(itemId) });

      // 2. 이전 값 스냅샷
      const previous = queryClient.getQueryData(itemKeys.detail(itemId));

      // 3. 낙관적 업데이트
      queryClient.setQueryData(itemKeys.detail(itemId), (old: Item | undefined) =>
        old ? { ...old, isFavorite: !old.isFavorite } : old,
      );

      return { previous };
    },
    onError: (_err, itemId, context) => {
      // 4. 에러 시 롤백
      if (context?.previous) {
        queryClient.setQueryData(itemKeys.detail(itemId), context.previous);
      }
    },
    onSettled: (_data, _err, itemId) => {
      // 5. 성공/실패 무관하게 서버 데이터 동기화
      queryClient.invalidateQueries({ queryKey: itemKeys.detail(itemId) });
    },
  });
}
```

---

## 12. 네비게이션 (Expo Router)

> 🔴 **SDK 56+: `@react-navigation/*` 직접 import 금지.** Expo Router가 React Navigation을 **포크해 내장**한다 —
> `@react-navigation/native` 등을 별도 설치/직접 import하면 이중 인스턴스로 네비게이션 상태가 파손되고 SDK 업그레이드마다 깨진다.
> 네비게이션 API는 **expo-router가 re-export하는 것만** 사용한다.

```typescript
import { useRouter, useLocalSearchParams, Link, Redirect } from 'expo-router';

// 화면 이동
const router = useRouter();
router.push('/item/123');           // push (뒤로가기 가능)
router.replace('/login');           // replace (히스토리 교체)
router.back();                      // 뒤로가기
router.dismiss();                   // 모달 닫기
router.dismissAll();                // 모든 모달 닫기

// 파라미터 받기
const { id } = useLocalSearchParams<{ id: string }>();

// 선언적 링크
<Link href="/item/123" asChild>
  <Pressable>
    <Text>Go to item</Text>
  </Pressable>
</Link>

// 조건부 리다이렉트 (인증 가드)
if (!session) return <Redirect href="/login" />;
```

### 네비게이션 규칙

- 화면 컴포넌트에서만 `useRouter` 사용 — 하위 컴포넌트는 `onPress` 콜백으로 전달
- 딥링크를 고려한 라우트 설계 (의미 있는 URL 구조)
- 인증 필요 화면은 `_layout.tsx`에서 가드 처리
- 화면 전환 시 과도한 데이터를 파라미터로 넘기지 않기 — ID만 넘기고 화면에서 fetch
- 중첩 레이아웃 활용: `(tabs)/_layout.tsx`, `(auth)/_layout.tsx`

### 인증 가드 패턴

```typescript
// app/(tabs)/_layout.tsx
import { Redirect, Tabs } from 'expo-router';

export default function TabLayout() {
  const { session, isLoading } = useAuth();

  if (isLoading) return <SplashScreen />;
  if (!session) return <Redirect href="/login" />;

  return (
    <Tabs screenOptions={{ headerShown: false }}>
      <Tabs.Screen name="home" options={{ title: 'Home' }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile' }} />
    </Tabs>
  );
}
```

### 라우트 파라미터 — 제네릭은 캐스팅일 뿐, 검증이 아니다

URL 파라미터는 런타임에 **항상 `string | string[]`** 이다. `useLocalSearchParams<T>()`의 제네릭은 단언에 불과하다 — 숫자/불리언이 필요하면 경계에서 파싱한다.

```typescript
import { z } from 'zod';

const itemParamsSchema = z.object({
  id: z.string().min(1),
  tab: z.enum(['info', 'reviews']).catch('info'),  // 잘못된 값은 기본값으로
  page: z.coerce.number().int().positive().catch(1), // "3" → 3 변환 + 검증
});

export default function ItemDetailScreen() {
  const raw = useLocalSearchParams();
  const parsed = itemParamsSchema.safeParse(raw);

  if (!parsed.success) return <ErrorView message="Invalid item ID" />;

  return <ItemDetail itemId={parsed.data.id} page={parsed.data.page} />;
}

// ❌ Bad: 제네릭만 믿고 숫자 연산 — 런타임엔 string이라 "1" + 1 = "11"
const { page } = useLocalSearchParams<{ page: number }>(); // 거짓 타입
```

---

## 13. 스타일링 (NativeWind / Tailwind)

> NativeWind는 **v4가 프로덕션 표준** (v5는 pre-release — 프로덕션 도입 금지).

### NativeWind 기본 규칙

```typescript
// Good: NativeWind 클래스로 스타일링
<View className="flex-1 bg-white px-4 py-6">
  <Text className="text-lg font-bold text-gray-900">Title</Text>
  <Text className="text-sm text-gray-500 mt-1">Subtitle</Text>
</View>

// 조건부 스타일
<Pressable
  className={`rounded-xl p-4 border ${
    isSelected ? 'bg-blue-50 border-blue-300' : 'bg-white border-gray-200'
  }`}
>

// Bad: 인라인 style 객체 (매 렌더마다 새 객체)
<View style={{ flex: 1, backgroundColor: 'white', paddingHorizontal: 16 }}>
```

### style vs className 사용 기준

| 상황 | 사용 | 이유 |
|------|------|------|
| 일반 View/Text 시각 스타일 | `className` | 간결, 반응형 지원 |
| ScrollView/FlatList `contentContainerStyle` | **`style` 필수** | 웹 호환성 (className 깨짐) |
| `maxWidth`, `width: '100%'`, `alignSelf: 'center'` | **`style` 권장** | 레이아웃 핵심 속성은 명시적으로 |
| 동적 계산 스타일 (애니메이션, 조건부 크기) | `style` | NativeWind는 정적 클래스 |
| 색상, 폰트, 패딩 등 시각 스타일 | `className` | Tailwind 유틸리티가 효율적 |
| Pressable `pressed` 상태 | `style` (함수형) | `({ pressed }) => [...]` 패턴 |

> **원칙**: 레이아웃 구조(너비 제한, 중앙 정렬, flex 배치)는 `style`로, 시각적 장식(색상, 폰트, 패딩)은 `className`으로.

### 테마/디자인 토큰

```typescript
// shared/styles/theme.ts — 색상/간격/radius를 as const 토큰으로 정의
export const colors = {
  primary: '#2e7d32',
  error: '#e53935',
  background: '#f0f2f5',
  surface: '#ffffff',
  gray: { 100: '#f5f5f5', 300: '#e0e0e0', 500: '#9e9e9e', 900: '#212121' },
} as const;

export const spacing = { xs: 4, sm: 8, md: 16, lg: 24, xl: 32 } as const;
```

- 색상 값을 컴포넌트에 직접 쓰지 않는다 -> 테마 토큰 또는 Tailwind 설정 참조
- NativeWind 사용 시 `tailwind.config.js`에서 테마 확장

### 접근성 (a11y)

- 모든 터치 영역: 최소 44x44pt (Apple HIG 기준)
- `Pressable`에 `accessibilityLabel` 필수 (아이콘/이미지 버튼)
- 색상만으로 정보 전달하지 않는다 (색각 이상 사용자 고려)
- `accessibilityRole` 적절히 설정 (`button`, `link`, `header` 등)
- `accessibilityState` 동적 상태 반영 (`{ selected: true }`, `{ disabled: true }`)

```typescript
<Pressable
  onPress={handleToggle}
  accessibilityRole="switch"
  accessibilityLabel="Enable notifications"
  accessibilityState={{ checked: isEnabled }}
  className="w-12 h-7 rounded-full justify-center px-1"
>
  <View className={isEnabled ? 'bg-green-500' : 'bg-gray-300'} />
</Pressable>
```

---

## 14. 반응형 디자인 + 웹 호환성

### 브레이크포인트 기준

| 이름 | 너비 | 대상 디바이스 | NativeWind 접두어 |
|------|------|-------------|-----------------|
| 기본 (모바일) | ~767px | 스마트폰 | (접두어 없음) |
| md (태블릿) | 768~1023px | 태블릿, 작은 노트북 | `md:` |
| lg (데스크톱) | 1024px~ | 데스크톱, 큰 노트북 | `lg:` |

### 모바일 퍼스트 원칙

```typescript
// Good: 모바일 퍼스트 — 기본이 모바일, md/lg로 확장
<View className="flex-1 px-4 md:px-8 lg:px-16">
  <View className="flex-col md:flex-row md:flex-wrap">
    <View className="w-full md:w-1/2 lg:w-1/3 p-2">
      <ItemCard item={item} />
    </View>
  </View>
</View>

// Bad: 데스크톱 먼저 작성
<View className="flex-row flex-wrap px-16 lg:px-16 md:px-8 sm:px-4">
```

### 반응형 훅

```typescript
import { useWindowDimensions } from 'react-native'; // Dimensions.get() 금지 — 회전/리사이즈 미대응

function useDeviceType() {
  const { width } = useWindowDimensions();
  return {
    isMobile: width < 768,
    isTablet: width >= 768 && width < 1024,
    isDesktop: width >= 1024,
  } as const;
}
// 사용 예: isDesktop이면 사이드바 레이아웃, 아니면 풀스크린 — 구조 분기는 이 훅으로
```

### 웹 호환성 필수 규칙

**1. `contentContainerClassName` 사용 금지 (웹에서 깨짐)**

```typescript
// Bad: 웹에서 레이아웃 깨짐 또는 런타임 에러
<ScrollView contentContainerClassName="flex-1 p-4 items-center">

// Good: style 객체로 레이아웃 처리
<ScrollView
  contentContainerStyle={{
    padding: 16,
    maxWidth: MAX_CONTENT_WIDTH,
    width: '100%',
    alignSelf: 'center',
  }}
>
```

**2. `Alert.alert()` 웹에서 작동 안 함**

```typescript
// ❌ Bad: 웹에서 조용히 무시됨 — 확인 없이 파괴적 동작이 그냥 실행됨
Alert.alert('Confirm', 'Are you sure?', [{ text: 'OK', onPress: handleConfirm }]);

// ✅ Good: 크로스플랫폼 confirm 헬퍼 — 웹은 window.confirm, 네이티브는 Alert.alert
function confirmDialog(title: string, message: string, onConfirm: () => void) {
  if (Platform.OS === 'web') {
    if (window.confirm(`${title}\n${message}`)) onConfirm();
  } else {
    Alert.alert(title, message, [
      { text: 'Cancel', style: 'cancel' },
      { text: 'OK', onPress: onConfirm },
    ]);
  }
}
```

**3. SafeAreaView 필수 래핑 — 반드시 react-native-safe-area-context**

- 🔴 RN 코어 `SafeAreaView`는 0.81에서 deprecated (iOS 전용 + 불완전) — 코어에서 import 금지
- 모든 화면 최상위를 `SafeAreaView`(safe-area-context)로 래핑 (`flex: 1` + `backgroundColor` 명시) — 아래 "화면 생성 템플릿" 참조

### 폭 제한 필수 규칙 (웹 반응형)

| 용도 | 기준 | 적용 대상 |
|------|------|----------|
| 폼/모달 (좁은 콘텐츠) | 400~500px | 로그인, 프로필 수정, 생성 폼 |
| 일반 콘텐츠 | 700~900px | 상세 화면, 단일 카드 목록 |
| 넓은 콘텐츠 (대시보드) | 1000~1200px | 탐색, 그리드 목록, 대시보드 |

```typescript
// Good: maxWidth 3종 세트
<ScrollView
  contentContainerStyle={{
    padding: 16,
    maxWidth: MAX_FORM_WIDTH,   // 상수 사용
    width: '100%',              // 모바일에서 꽉 채움
    alignSelf: 'center',        // 웹에서 중앙 정렬
  }}
>

// Bad: 하드코딩 + alignSelf 누락
<ScrollView contentContainerStyle={{ padding: 16, maxWidth: 448 }}>
```

### 화면 생성 템플릿

> 새 화면 추가 시 점검 항목 → "검증 체크리스트" 섹션의 **"새 화면 추가 시"** 를 따른다 (단일 원천).

```typescript
// 화면 생성 템플릿
import { ScrollView } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context'; // RN 코어 것 금지 (0.81 deprecated)

export default function NewScreen() {
  return (
    <SafeAreaView style={{ flex: 1, backgroundColor: '#f0f2f5' }}>
      <ScrollView
        contentContainerStyle={{
          padding: 16,
          maxWidth: MAX_CONTENT_WIDTH,
          width: '100%',
          alignSelf: 'center',
        }}
      >
        {/* content */}
      </ScrollView>
    </SafeAreaView>
  );
}
```

### 반응형 컴포넌트 패턴

```typescript
// 반응형 컨테이너 (공용 래퍼 1개로 통일 — 화면마다 브레이크포인트 반복 금지)
function Container({ children, className }: PropsWithChildren<{ className?: string }>) {
  return (
    <View className={`flex-1 px-4 md:px-8 lg:max-w-[1200px] lg:mx-auto lg:px-12 ${className ?? ''}`}>
      {children}
    </View>
  );
}

// 웹에서만 hover
<Pressable className="bg-white active:bg-gray-100 web:hover:bg-gray-50 rounded-xl p-4">
```

---

## 15. 애니메이션 & 제스처

### 언제 어떤 방법을 쓸 것인가 (Reanimated 4 기준)

> Reanimated 4는 **New Architecture 전용**이며, worklet 런타임이 **별도 패키지(`react-native-worklets`)로 분리**됐다.
> `LayoutAnimation` + `UIManager.setLayoutAnimationEnabledExperimental`은 구 아키텍처 API — 신규 코드 생성 금지.

| 상황 | 방법 | 예시 |
|------|------|------|
| 단순 상태 전환 (색상, 크기, 투명도) | Reanimated 4 **CSS 애니메이션 API** (`transitionProperty` 등 스타일 선언) | 버튼 스케일, 뱃지 강조 |
| 마운트/언마운트/재배치 전환 | Reanimated `entering`/`exiting`/`layout` | 리스트 아이템 추가/삭제 |
| 제스처 연동 60fps | worklet (`useSharedValue` + `useAnimatedStyle`) | 스와이프 삭제, 바텀시트 드래그 |
| 제스처 인식 | `react-native-gesture-handler` | 핀치, 팬, 롱프레스 |

### entering/exiting (마운트 전환 — 가장 간단)

```typescript
import Animated, { FadeIn, FadeOut, LinearTransition } from 'react-native-reanimated';

// 리스트 아이템 추가/삭제 전환 — LayoutAnimation 대체
<Animated.View
  entering={FadeIn.duration(200)}
  exiting={FadeOut.duration(150)}
  layout={LinearTransition}
>
  <ItemCard item={item} />
</Animated.View>
```

### Gesture Handler + Reanimated worklet (스와이프 삭제)

```typescript
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  runOnJS,
} from 'react-native-reanimated';

function SwipeableItem({ onDelete }: { onDelete: () => void }) {
  const translateX = useSharedValue(0);
  const SWIPE_THRESHOLD = -100;

  const panGesture = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = Math.min(0, event.translationX); // 왼쪽만 허용
    })
    .onEnd(() => {
      if (translateX.value < SWIPE_THRESHOLD) {
        translateX.value = withTiming(-300, {}, () => {
          runOnJS(onDelete)(); // UI 스레드 → JS 스레드 전환 — worklet에서 JS 함수 직접 호출 금지
        });
      } else {
        translateX.value = withTiming(0); // 원래 위치로 복원
      }
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }));

  return (
    <GestureDetector gesture={panGesture}>
      <Animated.View style={animatedStyle}>
        {/* item content */}
      </Animated.View>
    </GestureDetector>
  );
}
```

### 애니메이션 규칙
- 애니메이션 duration: 150~300ms (짧아야 빠른 느낌)
- worklet 안에서 일반 JS 함수 직접 호출 금지 (런타임 크래시) -> `runOnJS(fn)(args)`로 감싸기
- worklet은 생성 시점의 클로저를 **캡처(직렬화)** 한다 — 바깥 변수의 이후 변경은 반영되지 않음. 변하는 값은 `useSharedValue`로 전달 (실전 함정 참조)
- 레거시 RN `Animated` 유지보수 시에만: `useNativeDriver: true` (transform/opacity 한정)
- 웹에서 Reanimated/GestureHandler 호환성 확인 필수

---

## 16. 키보드 처리

### KeyboardAvoidingView (iOS/Android 차이)

```typescript
import { KeyboardAvoidingView, Platform, ScrollView } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

// 폼 화면의 기본 구조
export default function FormScreen() {
  // KeyboardAvoidingView는 "화면 최상단"부터 계산한다 — 위에 헤더가 있으면
  // 그 높이를 keyboardVerticalOffset으로 알려줘야 iOS에서 입력창이 가려지지 않는다.
  // 커스텀 헤더(headerShown: false + 자체 헤더)면 자기가 아는 높이, 네이티브 헤더면 onLayout 실측.
  const insets = useSafeAreaInsets();
  const headerOffset = insets.top + APP_HEADER_HEIGHT; // 임의 매직 넘버 금지 — 실제 헤더 구성에서 도출

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={{ flex: 1 }}
        keyboardVerticalOffset={headerOffset}
      >
        <ScrollView
          contentContainerStyle={{ padding: 16 }}
          keyboardShouldPersistTaps="handled" // 키보드 열린 상태에서 버튼 탭 가능
          showsVerticalScrollIndicator={false}
        >
          {/* form fields */}
          <TextInput placeholder="Title" />
          <TextInput placeholder="Description" multiline />
          <Button title="Submit" onPress={handleSubmit} />
        </ScrollView>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}
```

### 키보드 상태 감지 / 닫기

- 상태 감지: `Keyboard.addListener` — iOS는 `keyboardWillShow/Hide`, Android는 `keyboardDidShow/Hide` (플랫폼별 이벤트명이 다름). 구독은 cleanup에서 반드시 `remove()`
- 화면 탭으로 닫기: `TouchableWithoutFeedback onPress={Keyboard.dismiss}` 래퍼 (`accessible={false}`)

```typescript
// TextInput에서 다음 입력으로 포커스 이동
const descriptionRef = useRef<TextInput>(null);

<TextInput
  placeholder="Title"
  returnKeyType="next"
  onSubmitEditing={() => descriptionRef.current?.focus()}
/>
<TextInput
  ref={descriptionRef}
  placeholder="Description"
  returnKeyType="done"
  onSubmitEditing={handleSubmit}
/>
```

### 키보드 처리 규칙
- 모든 폼 화면에 `KeyboardAvoidingView` 필수
- `behavior`: iOS는 `'padding'`, Android는 `'height'`
- **헤더가 있는 화면**: `keyboardVerticalOffset`에 실제 헤더 높이(자체 헤더 상수 또는 onLayout 실측) 전달 — 누락 시 iOS에서 입력창이 헤더 높이만큼 가려짐 (실전 함정 참조)
- `keyboardShouldPersistTaps="handled"` — 키보드 열린 상태에서 버튼 탭 가능
- 다중 입력 폼: `returnKeyType` + `onSubmitEditing`으로 포커스 연결
- 키보드 위 고정 버튼: `position: 'absolute'` + keyboard height offset

---

## 17. 리스트 최적화 (FlashList v2)

> **FlashList v2는 New Architecture 전용**이며 아이템 높이를 자동 측정한다 —
> v1의 `estimatedItemSize`는 **불필요** (v1 문서/예제 복사 금지).

```typescript
import { FlashList } from '@shopify/flash-list';

<FlashList
  data={items}
  renderItem={({ item }) => <ItemCard item={item} />}
  keyExtractor={(item) => item.id} // 고유 키 (index 금지)
  ItemSeparatorComponent={() => <View className="h-2" />}
  ListEmptyComponent={<EmptyView message="No items" />}
  ListHeaderComponent={<SearchBar />}
  onEndReached={fetchNextPage}
  onEndReachedThreshold={0.5}
  refreshing={isRefreshing}
  onRefresh={handleRefresh}
/>
```

### 리스트 아이템은 별도 컴포넌트로

- 아이템을 **별도 컴포넌트로 분리**하는 것은 여전히 필수 — 렌더 단위 격리가 컴파일러 최적화의 경계가 된다
- `React.memo` 수동 래핑은 불필요 (React Compiler가 처리 — "훅 규칙" 참조). 컴파일러 비활성 레거시에서만 측정 후 적용
- `renderItem`에 인라인으로 JSX 수십 줄 작성 금지 — 분리된 컴포넌트 호출만

### 무한 스크롤 (react-query + FlashList)

```typescript
function useInfiniteItems() {
  return useInfiniteQuery({
    queryKey: ['items'],
    queryFn: ({ pageParam = 0 }) => fetchItems({ offset: pageParam, limit: 20 }),
    getNextPageParam: (lastPage, allPages) =>
      lastPage.length === 20 ? allPages.length * 20 : undefined,
    initialPageParam: 0,
  });
}

function ItemListScreen() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isLoading } = useInfiniteItems();

  const items = data?.pages.flatMap(page => page) ?? []; // 파생 값 — 렌더 중 계산 (컴파일러 최적화)

  return (
    <FlashList
      data={items}
      renderItem={({ item }) => <ItemCard item={item} />}
      keyExtractor={(item) => item.id}
      onEndReached={() => {
        if (hasNextPage && !isFetchingNextPage) fetchNextPage();
      }}
      onEndReachedThreshold={0.5}
      ListFooterComponent={isFetchingNextPage ? <ActivityIndicator /> : null}
    />
  );
}
```

### 리스트 규칙

- 10개 이상: `FlatList` 또는 `FlashList` 필수 (`ScrollView` + `.map()` 금지)
- 리스트 아이템: 별도 컴포넌트로 분리 (memo 수동 래핑은 컴파일러 위임)
- `keyExtractor`: 고유 ID 사용 (index 사용 금지 — 순서 변경 시 버그)
- 이미지 리스트: `expo-image`의 캐싱 + `recyclingKey` 활용
- 섹션 리스트: `SectionList` 또는 FlashList 섹션 지원 활용
- 빈 리스트: `ListEmptyComponent`로 빈 상태 메시지 표시

---

## 18. 안티패턴 (하지 말 것)

### React Native 일반

| 안티패턴 | 실패 결과 | 대안 |
|---------|----------|------|
| `ScrollView` + `.map()` (긴 리스트) | 수백 아이템 전부 마운트 → 진입 수 초 프리즈 + 메모리 폭증 → 저사양 기기 OOM 종료 | `FlashList` / `FlatList` |
| `@react-navigation/*` 직접 import (SDK 56+) | Expo Router 내장 포크와 이중 인스턴스 → 네비게이션 상태 파손, SDK 업그레이드마다 파손 | expo-router가 export하는 API만 |
| `newArchEnabled: false` 옵트아웃 시도 | 0.82+에서 설정 무시 — 문제를 해결한 척만 하고 비호환은 그대로 | 비호환 라이브러리 교체/업데이트 |
| 수동 `useMemo`/`useCallback`/`React.memo` 습관 적용 (SDK 54+) | React Compiler와 중복 — 의존성 배열 실수 시 stale closure 버그만 추가 | 컴파일러에 위임 (훅 규칙 참조) |
| `SafeAreaView` (RN 코어) | 0.81 deprecated + iOS 전용/불완전 → Android 노치·제스처 영역 침범 | `react-native-safe-area-context` |
| 인라인 `style={{}}` 객체 | 매 렌더 새 객체 → 자식 전파 리렌더 (인라인 생성 자체는 컴파일러도 제거 못 함) | `StyleSheet.create` / NativeWind |
| `useEffect`로 데이터 fetch | 로딩/에러/캐시/경합을 수동 관리 → 빠른 화면 전환 시 낡은 응답이 최신 데이터를 덮음 | `@tanstack/react-query` |
| `Image` (RN 기본) | 캐싱 없음 → 리스트 스크롤마다 재다운로드, 깜빡임 + 데이터 낭비 | `expo-image` |
| `TouchableOpacity` | 유지보수 모드 — hover/focus 등 신규 인터랙션 미지원, 웹 접근성 결손 | `Pressable` |
| `Dimensions.get()` 직접 사용 | 회전/창 리사이즈/폴더블 전개 미반영 → 태블릿에서 레이아웃 파손 | `useWindowDimensions` |
| `AsyncStorage` 신규 도입 | 느린 비동기 왕복 + 무암호화 → 토큰 저장 시 루팅 기기에서 평문 노출 | mmkv / secure-store / expo-sqlite kv-store |
| 화면 컴포넌트에 비즈니스 로직 | 테스트 불가 + 유사 화면 복제 시 로직 중복 → 한쪽만 수정되는 불일치 버그 | 커스텀 훅으로 분리 |
| Context로 앱 전체 상태 관리 | 값 하나 변경에 Provider 하위 전체 리렌더 → 타이핑마다 화면 전체 갱신 | zustand / react-query |
| `Alert.alert()` (웹 포함 앱) | 웹에서 no-op — 확인창 없이 삭제 등 파괴적 동작이 그냥 실행됨 | 크로스플랫폼 confirm 헬퍼 |
| `contentContainerClassName` | 웹에서 스타일 미적용 → 중앙 정렬/패딩 소실로 레이아웃 붕괴 | `contentContainerStyle` |
| `window.location` 직접 접근 | 네이티브에서 ReferenceError 크래시 | `Platform.OS === 'web'` 체크 후 접근 |
| zustand에 서버 데이터 복사 | 캐시 이중화 → 무효화 누락 시 낡은 데이터가 화면에 잔존 | react-query 단일 소스 |

### AI가 흔히 생성하는 실수

| 실수 | 실패 결과 | 수정 방법 |
|------|----------|----------|
| `<View>` 안에 텍스트 직접 넣기 | "Text strings must be rendered within a \<Text\>" 즉시 크래시 | 반드시 `<Text>` 감싸기 |
| 웹 HTML 태그 (`<div>`, `<span>`) | 네이티브에 해당 컴포넌트 없음 → 렌더 크래시 | `<View>`, `<Text>` |
| `onClick` 사용 | RN에 없는 prop — 에러 없이 조용히 무시되어 버튼 무반응 | `onPress` |
| CSS 속성 (`background-color`) | 스타일 무시/경고 — 화면은 뜨지만 디자인 미적용 | camelCase 또는 NativeWind |
| `shouldShowAlert` (구 예제 복사) | SDK 53+ deprecated — iOS 포그라운드 알림이 표시되지 않음 | `shouldShowBanner` + `shouldShowList` |
| FlashList에 `estimatedItemSize` (v1 예제 복사) | v2에서 무의미한 prop — v1 기준 낡은 코드 신호 | v2는 자동 측정, prop 제거 |
| `npm install`로 Expo 패키지 설치 | SDK 비호환 버전 설치 → 빌드 실패 또는 런타임 네이티브 크래시 | `npx expo install` |
| `useEffect(async () => {})` | effect가 Promise 반환 → cleanup 무시 (구독 해제 누락) | 내부에 async 함수 정의 후 호출 |
| FlatList/FlashList `key={index}` | 삽입/삭제 시 상태 꼬임 — 입력값이 엉뚱한 행에 남음 | 고유 ID |
| useEffect cleanup 누락 | 리스너 누적 → 이벤트 1회에 핸들러 N회 실행, 메모리 릭 | 구독은 반드시 return으로 해제 |
| setState after unmount | 경고 + 리소스 릭 신호 — fetch 완료가 죽은 화면 갱신 시도 | isMounted 패턴 / AbortController |
| 웹 전용 API (`window`, `document`) | 네이티브 크래시 | `Platform.OS === 'web'` 체크 |
| KeyboardAvoidingView 누락 (폼) | 키보드가 입력창 가림 — 사용자가 입력 내용을 못 봄 | KeyboardAvoidingView + 헤더 오프셋 |
| `console.log` 프로덕션 잔존 | 릴리즈 성능 저하 + 민감 데이터 로그 노출 | 제거 플러그인 또는 로거 |

### 실전에서 발견된 지뢰

> 4요소(증상/원인/해결/오답) 형식으로 **"실전 함정 & 지뢰" 섹션에 누적**한다 — 여기 중복 기재하지 않는다.

---

## 19. Null/빈 상태 처리

- 모든 화면에 **4가지 상태**를 처리한다: 로딩 / 에러 / 빈 데이터 / 정상
- 빈 상태에 의미 있는 메시지 + CTA(Call to Action) 버튼 제공
- 에러 상태에 재시도 버튼 제공
- 스켈레톤 UI로 로딩 상태를 부드럽게 처리

### 공통 상태 처리 패턴

```typescript
// 재사용 가능한 상태 처리 훅
function useAsyncState<T>(queryResult: UseQueryResult<T>) {
  const { data, isLoading, error, refetch } = queryResult;

  if (isLoading) return { status: 'loading' as const };
  if (error) return { status: 'error' as const, error, refetch };
  if (!data || (Array.isArray(data) && data.length === 0)) {
    return { status: 'empty' as const };
  }
  return { status: 'success' as const, data };
}

// 사용 — discriminated union이라 각 case에서 data/error가 자동으로 좁혀짐
function ItemListScreen() {
  const state = useAsyncState(useItemList());

  switch (state.status) {
    case 'loading': return <SkeletonList count={5} />;
    case 'error':   return <ErrorView error={state.error} onRetry={state.refetch} />;
    case 'empty':   return <EmptyView message="No items yet" actionLabel="Create first item" onAction={goCreate} />;
    case 'success': return <ItemList items={state.data} />;
  }
}
```

### 공통 빈 상태 컴포넌트

```typescript
// 아이콘 + 메시지 + (선택) CTA 버튼 — 빈 상태에는 "다음 행동"을 항상 제시
interface EmptyViewProps {
  icon?: string;
  message: string;
  actionLabel?: string;
  onAction?: () => void;
}
// 구현: 중앙 정렬 View + 아이콘(48) + gray 메시지 + actionLabel && onAction 조건부 버튼
```

### JSX `&&` 렌더링의 0/NaN 함정 (RN 특화)

> 옵셔널 체이닝 / `??` / 타입 가드 등 일반 null 처리 → `typescript.md` 참조.

```typescript
// ❌ Bad: falsy 숫자가 그대로 렌더 — RN에서는 <Text> 밖 문자열이라 크래시까지 감
{count && <Badge text={count} />}  // count=0 → "0" 렌더 (웹) / Text 미포장 크래시 (네이티브)

// ✅ Good: 명시적 boolean 비교
{count > 0 && <Badge text={count} />}
{count != null && <Badge text={count} />}
```

---

## 20. 비동기 처리

> TypeScript 기본 비동기 규칙(`typescript.md`)을 따르되, React Native 특화:

### 데이터 fetching은 react-query

```typescript
// Good: react-query로 캐싱/재시도/무효화 자동 관리
const { data, isLoading, error, refetch } = useQuery({
  queryKey: ['items', filters],
  queryFn: () => fetchItems(filters),
  staleTime: 5 * 60 * 1000,  // 5분
  retry: 2,
  enabled: !!filters.categoryId, // 조건부 실행
});

// Bad: useEffect + useState로 직접 관리
useEffect(() => {
  setLoading(true);
  fetchItems(filters).then(setData).catch(setError).finally(() => setLoading(false));
}, [filters]);
```

### 병렬 실행

> `Promise.all`/`allSettled`/타임아웃 등 일반 비동기 패턴 → `typescript.md` 참조. RN에서는 쿼리 병렬화:

```typescript
// 여러 쿼리 병렬 실행 — useQuery는 각각 독립적으로 병렬 fetch
const itemQuery = useQuery({ queryKey: ['item', id], queryFn: () => fetchItem(id) });
const reviewsQuery = useQuery({ queryKey: ['reviews', id], queryFn: () => fetchReviews(id) });

const isLoading = itemQuery.isLoading || reviewsQuery.isLoading;
```

### 요청 취소 (AbortController)

```typescript
// react-query의 내장 취소 지원
const { data } = useQuery({
  queryKey: ['search', keyword],
  queryFn: ({ signal }) => searchItems(keyword, signal), // signal 전달
  enabled: keyword.length >= 2,
});

// API 함수에서 signal 활용
async function searchItems(keyword: string, signal?: AbortSignal): Promise<Item[]> {
  const response = await fetch(`/api/items?q=${keyword}`, { signal });
  if (!response.ok) throw new ApiError(response.status);
  return response.json();
}

// 수동 취소가 필요하면 useRef(new AbortController()) + unmount cleanup에서 abort()
// (AbortController 일반 패턴 → typescript.md 참조)
```

### 디바운스 검색

```typescript
function useDebounce<T>(value: T, delay: number = 300): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// 사용: 검색어를 디바운스한 값으로 useQuery — queryKey: ['search', debouncedKeyword],
//       enabled: debouncedKeyword.length >= 2 (짧은 입력엔 요청 안 나감)
```

---

## 21. 에러/예외 처리

> TypeScript 기본 에러 처리(`typescript.md`)를 따르되, React Native 특화:

### 커스텀 에러 + 사용자 메시지 매핑

> `AppError` 기본 클래스, catch-unknown 패턴, 경계에서만 catch 원칙 → `typescript.md` 참조.
> RN 특화: HTTP 상태를 아는 `NetworkError` + 화면 표시용 메시지 매핑.

```typescript
class NetworkError extends AppError {
  constructor(public readonly statusCode: number, userMessage: string, message?: string) {
    super(message ?? userMessage, 'NETWORK_ERROR', userMessage, statusCode);
    this.name = 'NetworkError';
  }
  get isUnauthorized() { return this.statusCode === 401; }
  get isNotFound() { return this.statusCode === 404; }
  get isServerError() { return this.statusCode >= 500; }
}

function getErrorMessage(error: unknown): string {
  if (error instanceof NetworkError) {
    if (error.isUnauthorized) return 'Login session has expired. Please log in again.';
    if (error.isNotFound) return 'The requested content does not exist.';
    if (error.isServerError) return 'A server error occurred. Please try again later.';
  }
  if (error instanceof AppError) return error.userMessage;
  return 'An unexpected error occurred. Please try again.'; // 내부 에러 노출 금지
}
```

### Error Boundary

```typescript
import { Component, type ErrorInfo, type ReactNode } from 'react';

interface ErrorBoundaryState { hasError: boolean; error: Error | null; }

class ErrorBoundary extends Component<{ children: ReactNode; fallback?: ReactNode }, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    reportCrash(error, errorInfo); // Sentry/Crashlytics 등에 보고 — 삼키지 않는다
  }

  private reset = () => this.setState({ hasError: false, error: null });

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <ErrorFallback error={this.state.error} onRetry={this.reset} />;
    }
    return this.props.children;
  }
}

// 사용: 화면/레이아웃 단위 래핑 — app/(tabs)/_layout.tsx 에서 <ErrorBoundary><Tabs /></ErrorBoundary>
```

### API 에러 처리 (react-query + fetch)

```typescript
// fetch 래퍼: HTTP 에러를 NetworkError로 변환 (호출부는 instanceof 분기만)
async function apiRequest<T>(url: string, options?: RequestInit): Promise<T> {
  try {
    const response = await fetch(url, {
      ...options,
      headers: { 'Content-Type': 'application/json', ...options?.headers },
    });
    if (!response.ok) {
      const body = await response.json().catch(() => null);
      throw new NetworkError(response.status, body?.message ?? `Request failed (${response.status})`);
    }
    return response.json();
  } catch (error) {
    if (error instanceof NetworkError) throw error;
    if (error instanceof TypeError) {
      // fetch의 네트워크 단절은 TypeError로 옴 — 오프라인 에러로 변환
      throw new AppError('fetch failed', 'NETWORK_OFFLINE', 'No internet connection.');
    }
    throw error;
  }
}

// react-query 글로벌 에러 처리
const queryClient = new QueryClient({
  defaultOptions: {
    mutations: {
      onError: (error) => {
        if (error instanceof NetworkError && error.isUnauthorized) {
          // 세션 만료 -> 로그인으로 리다이렉트
          useAppStore.getState().clearSession();
          router.replace('/login');
          return;
        }
        // 일반 에러 -> 토스트
        showToast(getErrorMessage(error));
      },
    },
  },
});
```

### 재시도 전략

```typescript
// react-query 쿼리별 재시도 설정
const { data } = useQuery({
  queryKey: ['item', id],
  queryFn: () => fetchItem(id),
  retry: (failureCount, error) => {
    // 401, 403, 404는 재시도 불필요
    if (error instanceof NetworkError) {
      if (error.isUnauthorized || error.isNotFound) return false;
      if (error.isServerError) return failureCount < 3;
    }
    return failureCount < 2;
  },
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 10000), // 지수 백오프
});
```

---

## 22. 앱 생명주기 (AppState)

```typescript
import { AppState, type AppStateStatus } from 'react-native';

function useAppState(onChange?: (status: AppStateStatus) => void) {
  const appState = useRef(AppState.currentState);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', (nextState) => {
      const prevState = appState.current;

      // 백그라운드 -> 포그라운드 전환
      if (prevState.match(/inactive|background/) && nextState === 'active') {
        onChange?.(nextState);
      }

      appState.current = nextState;
    });

    return () => subscription.remove();
  }, [onChange]);

  return appState;
}
```

### 포그라운드 복귀 시 토큰 재검증

```typescript
// 루트 레이아웃에서 useSessionRevalidation() 호출
function useSessionRevalidation() {
  const { session, clearSession } = useAppStore(
    useShallow((s) => ({ session: s.session, clearSession: s.clearSession })),
  );
  const router = useRouter();

  useAppState(async () => {
    if (!session) return;
    try {
      const isValid = await validateToken(session.token);
      if (!isValid) {
        clearSession();
        router.replace('/login');
      }
    } catch {
      // 네트워크 실패 -> 무시 (오프라인 허용)
    }
  });
}
```

### 백그라운드 전환 시 리소스 정리

- `background` 진입: 실시간 구독 disconnect, 진행 중 업로드 pause — 위 `useAppState` 훅에서 상태 분기로 처리
- `active` 복귀: 재연결/재개 + 토큰 재검증 (위 패턴)
- 백그라운드에서 소켓/타이머를 살려두면 OS가 앱을 강제 종료하거나 배터리 이슈로 리젝 사유가 된다

### react-query + AppState 자동 리프레시

```typescript
import { focusManager } from '@tanstack/react-query';

// 앱이 포그라운드로 돌아올 때 stale 쿼리 자동 리페치
function useAppStateRefetch() {
  useEffect(() => {
    const subscription = AppState.addEventListener('change', (status) => {
      focusManager.setFocused(status === 'active');
    });
    return () => subscription.remove();
  }, []);
}
```

---

## 23. 보안

> TypeScript 기본 보안(`typescript.md`)을 따르되, React Native 특화:

### 토큰 저장

```typescript
import * as SecureStore from 'expo-secure-store';

// Good: 민감 정보는 SecureStore (iOS Keychain / Android Keystore)
async function saveToken(token: string): Promise<void> {
  await SecureStore.setItemAsync('auth_token', token);
}

async function getToken(): Promise<string | null> {
  return SecureStore.getItemAsync('auth_token');
}

async function deleteToken(): Promise<void> {
  await SecureStore.deleteItemAsync('auth_token');
}

// Bad: AsyncStorage에 토큰 저장 (암호화 안 됨)
await AsyncStorage.setItem('token', token); // 보안 취약!
```

### 딥링크 검증

```typescript
// 외부에서 들어오는 딥링크 파라미터는 반드시 스키마 검증 — 임의 URL로 내부 화면 조작 방지
import { useURL } from 'expo-linking';
import { z } from 'zod';

const deepLinkSchema = z.object({
  itemId: z.string().uuid().optional(),
  action: z.enum(['view', 'edit']).optional(),
});

function useDeepLinkHandler() {
  const url = useURL();
  const router = useRouter();

  useEffect(() => {
    if (!url) return;
    const validated = deepLinkSchema.safeParse(parseURLParams(url));
    if (!validated.success) return console.warn('Invalid deep link:', url); // 무시 + 로깅만
    if (validated.data.itemId) router.push(`/item/${validated.data.itemId}`);
  }, [url]);
}
```

### WebView XSS 방지

```typescript
import { WebView } from 'react-native-webview';

// Good: 콘텐츠 보안 정책 적용
<WebView
  source={{ uri: sanitizedUrl }}
  originWhitelist={['https://*']}  // HTTPS만 허용
  javaScriptEnabled={false}        // 필요 없으면 비활성화
  onNavigationStateChange={(event) => {
    // 허용된 도메인만 네비게이션 허용
    if (!isAllowedDomain(event.url)) {
      webViewRef.current?.stopLoading();
    }
  }}
/>

// Bad: 사용자 입력을 직접 HTML에 삽입
<WebView source={{ html: `<div>${userInput}</div>` }} /> // XSS 취약!
```

### 토큰 리프레시 패턴

```typescript
// 401 시 토큰 리프레시 후 1회 재시도 — 핵심은 "동시 401 다발 시 리프레시 1회만" (모듈 레벨 공유 Promise)
let refreshPromise: Promise<string> | null = null;

async function authenticatedFetch(url: string, options?: RequestInit): Promise<Response> {
  const token = await getToken();
  const response = await fetch(url, {
    ...options,
    headers: { ...options?.headers, Authorization: `Bearer ${token}` },
  });
  if (response.status !== 401) return response;

  refreshPromise ??= refreshToken(); // 이미 진행 중이면 같은 Promise를 공유 (중복 리프레시 방지)
  try {
    const newToken = await refreshPromise;
    await saveToken(newToken);
    return fetch(url, {
      ...options,
      headers: { ...options?.headers, Authorization: `Bearer ${newToken}` },
    });
  } catch {
    await deleteToken();
    throw new NetworkError(401, 'Session expired');
  } finally {
    refreshPromise = null;
  }
}
```

### 보안 규칙 요약
- 토큰/비밀번호: `expo-secure-store`만 사용
- API 키: 앱 번들에 포함 금지 -> 백엔드 프록시 또는 `EXPO_PUBLIC_` 환경 변수
- 딥링크: zod로 파라미터 검증
- WebView: 도메인 화이트리스트 + HTTPS 강제
- 인증 상태: 앱 복귀 시 토큰 유효성 재확인
- 화면 캡처 방지: 민감 화면에서 `expo-screen-capture` 사용
- 코드 난독화: EAS Build 릴리즈 모드 (Hermes 자동 적용)

---

## 24. 성능 최적화

### 렌더링 성능 — React Compiler 시대 (Expo SDK 54+ 기본 활성)

```typescript
// 규칙 1. 수동 useMemo/useCallback/React.memo 지양 — 컴파일러가 자동 메모이제이션
//   ❌ const sorted = useMemo(() => [...items].sort(byDate), [items]);
//   ✅ const sorted = [...items].sort(byDate);

// 규칙 2. Rules of React 준수가 전제 — 위반한 컴포넌트는 컴파일러가 최적화를 건너뜀
//   (훅은 최상위에서만, 렌더 중 부수효과 금지, props/state 직접 변형 금지)

// 규칙 3. eslint-plugin-react-compiler 경고 = 그 컴포넌트는 최적화 제외 중
//   → 메모이제이션을 손으로 추가하는 게 아니라 규칙 위반을 수정한다

// 규칙 4. 여전히 사람 몫인 최적화:
//   - 인라인 style 객체/배열 생성 지양 (StyleSheet.create / NativeWind)
//   - 리스트 가상화 (FlashList) + 아이템 컴포넌트 분리
//   - 이미지 리사이즈/캐싱 (expo-image)
//   - 화면 단위 코드 분할 (lazy import)
```

> 레거시(컴파일러 비활성) 프로젝트만: `React.memo`/`useMemo`/`useCallback`을 **Profiler 측정 후** 선별 적용.

### React Profiler 사용

```typescript
import { Profiler, type ProfilerOnRenderCallback } from 'react';

const onRender: ProfilerOnRenderCallback = (id, phase, actualDuration) => {
  if (actualDuration > 16) { // 16ms = 60fps 기준
    console.warn(`[Profiler] ${id} ${phase} took ${actualDuration.toFixed(1)}ms`);
  }
};

// 성능 측정이 필요한 구간에 래핑
<Profiler id="ItemList" onRender={onRender}>
  <ItemList items={items} />
</Profiler>
```

### 메모리 관리

- 이미지 리스트: `expo-image` 캐시 사이즈 제한 설정
- 구독/리스너: useEffect cleanup에서 반드시 해제
- 대량 데이터: 가상화 리스트 사용 (FlashList)
- Timer: clearTimeout/clearInterval 필수
- WebSocket: 화면 언마운트 시 disconnect

### 앱 시작 최적화

- 초기 화면에 필요한 데이터만 먼저 fetch
- 무거운 라이브러리 lazy import
- `expo-splash-screen`으로 초기 로드 동안 스플래시 유지
- Hermes 엔진 사용 (Expo 기본값) — 앱 시작 시간 + 메모리 개선

### 번들 최적화

- `npx expo install`로 호환 버전 관리
- 사용하지 않는 라이브러리 정리
- `console.log` 프로덕션 제거 (`babel-plugin-transform-remove-console`)
- 이미지 에셋 최적화 (WebP 사용 권장)
- Tree-shaking 지원 라이브러리 선택 (`date-fns` > `moment`)

---

## 25. 이미지 처리

### expo-image 기본 설정

```typescript
import { Image } from 'expo-image';

// 기본 사용 (캐싱 자동)
<Image
  source={{ uri: item.imageUrl }}
  style={{ width: '100%', aspectRatio: 16 / 9 }}
  contentFit="cover"
  placeholder={{ blurhash: item.blurhash }}  // 블러 해시 플레이스홀더
  transition={200}                             // 페이드인 전환
  recyclingKey={item.id}                       // 리스트에서 재활용 키
/>

// 정적 이미지
<Image
  source={require('@/assets/images/placeholder.png')}
  style={{ width: 48, height: 48 }}
/>
```

### 이미지 업로드 + 압축

```typescript
import * as ImagePicker from 'expo-image-picker';
import * as ImageManipulator from 'expo-image-manipulator';

async function pickAndCompressImage(): Promise<string | null> {
  // 1. 권한 확인 — 거부 케이스 반드시 처리
  const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
  if (status !== 'granted') {
    showToast('Photo library permission is required');
    return null;
  }

  // 2. 이미지 선택
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ['images'],
    allowsEditing: true,
    quality: 0.8,
  });
  if (result.canceled || !result.assets[0]) return null;

  // 3. 리사이즈 + 압축 — 원본(수 MB~4K)을 그대로 업로드하지 않는다
  const manipulated = await ImageManipulator.manipulateAsync(
    result.assets[0].uri,
    [{ resize: { width: 800 } }],
    { compress: 0.7, format: ImageManipulator.SaveFormat.JPEG },
  );
  return manipulated.uri;
}

// 4. 업로드: fetch(uri) → blob() → 스토리지 업로드 (실패 시 AppError로 변환해 전파)
```

### 이미지 캐시 전략

- expo-image는 디스크 캐시 자동 관리 — 미리 캐싱은 `Image.prefetch(url)` (리스트 진입 전 `Promise.all`)
- 캐시 초기화가 필요할 때만 `Image.clearDiskCache()` / `Image.clearMemoryCache()`

### 이미지 규칙
- 항상 `expo-image` 사용 (RN 기본 `Image` 사용 금지)
- `contentFit` 지정 필수: `cover`(카드), `contain`(프로필), `fill`(배경)
- `placeholder` 사용: `blurhash` 또는 저해상도 이미지
- `transition` 사용: 200~300ms 페이드인
- 리스트 내 이미지: `recyclingKey` 지정 (이미지 깜빡임 방지)
- 업로드 전 반드시 리사이즈 (원본 4K 전송 금지)
- `aspectRatio` + `width: '100%'`로 비율 유지

---

## 26. 환경변수 관리

### Expo 환경변수 규칙

```typescript
// .env 파일 분리
// .env              — 기본값 (개발)
// .env.production   — 프로덕션
// .env.local        — 로컬 오버라이드 (gitignore)

// EXPO_PUBLIC_ 접두어 규칙
EXPO_PUBLIC_API_URL=https://api.example.com      // 클라이언트 접근 가능
EXPO_PUBLIC_SUPABASE_URL=https://xxx.supabase.co  // 클라이언트 접근 가능
SUPABASE_SERVICE_ROLE_KEY=secret_xxx               // EXPO_PUBLIC_ 없음 -> 클라이언트 접근 불가
```

### 타입 안전한 환경변수 접근

```typescript
// src/config/env.ts — 앱 기동 시점에 검증해서 누락을 즉시 발견 (런타임 깊은 곳 크래시 방지)
import { z } from 'zod';

const envSchema = z.object({
  EXPO_PUBLIC_API_URL: z.string().url(),
  EXPO_PUBLIC_BACKEND_KEY: z.string().min(1),
});

const result = envSchema.safeParse({
  EXPO_PUBLIC_API_URL: process.env.EXPO_PUBLIC_API_URL,
  EXPO_PUBLIC_BACKEND_KEY: process.env.EXPO_PUBLIC_BACKEND_KEY,
});
if (!result.success) throw new Error(`Invalid environment variables: ${result.error.message}`);

export const env = result.data;
// 사용처: import { env } from '@/config/env'; — process.env 직접 접근 금지 (env.ts 한 곳으로)
```

### 환경변수 규칙
- `EXPO_PUBLIC_` 접두어: 클라이언트에 노출되어도 안전한 것만
- 비밀 키(service role, API secret): `EXPO_PUBLIC_` 접두어 금지 -> Edge Function에서만 사용
- 환경변수 변경 후 Expo dev server 재시작 필수
- `.env.local`은 `.gitignore`에 포함
- `expo-constants`로 빌드 타임 설정 접근

---

## 27. 폰트 & 스플래시

### 폰트 로딩 + 스플래시 스크린 연동

```typescript
import { useFonts } from 'expo-font';
import * as SplashScreen from 'expo-splash-screen';
import { useEffect } from 'react';

// 스플래시 유지 (폰트 로딩 완료까지)
SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [fontsLoaded, fontError] = useFonts({
    'Pretendard-Regular': require('@/assets/fonts/Pretendard-Regular.otf'),
    'Pretendard-Bold': require('@/assets/fonts/Pretendard-Bold.otf'),
  });

  useEffect(() => {
    if (fontsLoaded || fontError) {
      SplashScreen.hideAsync(); // 폰트 로드 완료 후 스플래시 숨김
    }
  }, [fontsLoaded, fontError]);

  if (!fontsLoaded && !fontError) {
    return null; // 스플래시가 보이는 동안 빈 화면
  }

  return <Stack />;
}
```

### 폰트 규칙
- `SplashScreen.preventAutoHideAsync()`는 모듈 최상위에서 호출 (컴포넌트 밖)
- 폰트 로딩 실패 시에도 앱 시작 가능하도록 처리 (`fontError` 체크)
- 커스텀 폰트는 `assets/fonts/`에 배치
- NativeWind에서 폰트 사용: `tailwind.config.js`의 `fontFamily` 확장

---

## 28. StatusBar

```typescript
import { StatusBar } from 'expo-status-bar';

// 루트 레이아웃에서 전역 설정 — 화면별로 바꿀 땐 해당 화면에서 <StatusBar style="light" /> 재선언
export default function RootLayout() {
  return (
    <>
      <StatusBar style="auto" /> {/* light/dark/auto */}
      <Stack />
    </>
  );
}
```

### StatusBar 규칙
- `expo-status-bar` 사용 (react-native StatusBar보다 간편)
- `style="auto"`: 시스템 테마에 따라 자동 전환
- 어두운 배경 화면: `style="light"` (흰색 텍스트)
- 밝은 배경 화면: `style="dark"` (검정 텍스트)
- Android: `StatusBar.translucent = true`가 기본 (Expo)
- iOS: `SafeAreaView`와 조합하여 노치 영역 처리

---

## 29. 푸시 알림 (expo-notifications)

```typescript
import * as Notifications from 'expo-notifications';
import Constants from 'expo-constants';

// 알림 표시 설정 (포그라운드)
// 🔴 shouldShowAlert는 SDK 53+ deprecated — iOS 14의 배너/알림센터 분리에 맞춰
//    shouldShowBanner(화면 상단 배너) + shouldShowList(알림 센터 목록)로 나뉘었다.
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowBanner: true,
    shouldShowList: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

// 권한 요청
async function requestNotificationPermission(): Promise<boolean> {
  const { status: existing } = await Notifications.getPermissionsAsync();
  if (existing === 'granted') return true;

  const { status } = await Notifications.requestPermissionsAsync();
  return status === 'granted';
}

// 토큰 가져오기
async function getPushToken(): Promise<string | null> {
  const permission = await requestNotificationPermission();
  if (!permission) return null;

  const token = await Notifications.getExpoPushTokenAsync({
    projectId: Constants.expoConfig?.extra?.eas?.projectId,
  });
  return token.data;
}

// 알림 리스너 설정 (루트 레이아웃) — 두 구독 모두 cleanup 필수
function useNotificationListeners() {
  const router = useRouter();

  useEffect(() => {
    // 포그라운드 수신 (뱃지 갱신 등) / 알림 탭 → 화면 이동
    const receivedSub = Notifications.addNotificationReceivedListener(() => {});
    const responseSub = Notifications.addNotificationResponseReceivedListener((response) => {
      const data = response.notification.request.content.data;
      if (typeof data?.itemId === 'string') router.push(`/item/${data.itemId}`); // 페이로드도 검증 후 사용
    });

    return () => {
      receivedSub.remove();
      responseSub.remove();
    };
  }, [router]);
}
```

---

## 30. 플랫폼 분기

```typescript
import { Platform } from 'react-native';

// 간단한 분기
const paddingTop = Platform.OS === 'ios' ? 44 : 0;

// Platform.select (3+ 플랫폼)
const shadow = Platform.select({
  ios: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
  },
  android: {
    elevation: 4,
  },
  web: {
    boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
  },
  default: {},
});

// 플랫폼별 파일 (복잡한 분기)
// DatePicker.ios.tsx / DatePicker.android.tsx / DatePicker.web.tsx
// import 시 자동 선택: import { DatePicker } from './DatePicker';

// 웹 전용 API 안전 접근
function openExternalLink(url: string) {
  if (Platform.OS === 'web') {
    window.open(url, '_blank'); // 웹에서만 window 접근
  } else {
    Linking.openURL(url); // 네이티브
  }
}
```

### 플랫폼 분기 규칙
- `Platform.OS` 비교는 `===` 사용 (`==` 금지)
- 3개 이상 플랫폼 분기: `Platform.select` 사용
- `window`, `document` 직접 접근 금지 -> `Platform.OS === 'web'` 체크 후
- 복잡한 플랫폼 차이: 별도 파일로 분리 (`.ios.tsx`, `.android.tsx`, `.web.tsx`)
- `Modal` `presentationStyle`: 웹 미지원 -> `Platform.select` 사용

---

## 31. 테스트

### 테스트 도구

| 용도 | 도구 |
|------|------|
| 컴포넌트 테스트 | `@testing-library/react-native` |
| 훅 테스트 | `renderHook` (from `@testing-library/react-native`) |
| 네비게이션 mock | `expo-router` mock |
| API mock | `msw` (Mock Service Worker) |
| react-query 테스트 | `QueryClientProvider` 래퍼 |
| E2E | `detox` 또는 `maestro` |

### 컴포넌트 테스트

```typescript
import { render, fireEvent, screen } from '@testing-library/react-native';

describe('ItemCard', () => {
  const mockItem = { id: '1', title: 'Test Item', description: 'Description' };

  it('renders item title and description', () => {
    render(<ItemCard item={mockItem} onPress={jest.fn()} />);

    expect(screen.getByText('Test Item')).toBeTruthy();
    expect(screen.getByText('Description')).toBeTruthy();
  });

  it('calls onPress with item id when pressed', () => {
    const onPress = jest.fn();
    render(<ItemCard item={mockItem} onPress={onPress} />);

    fireEvent.press(screen.getByText('Test Item'));
    expect(onPress).toHaveBeenCalledWith('1');
  });

  it('shows badge when isHighlighted is true', () => {
    render(<ItemCard item={mockItem} onPress={jest.fn()} isHighlighted />);

    expect(screen.getByText('Featured')).toBeTruthy();
  });
});
```

### 훅 테스트

```typescript
import { renderHook, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

// react-query 훅 테스트 래퍼 (retry: false — 테스트에서 재시도 대기 방지)
function createQueryWrapper() {
  const queryClient = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return ({ children }: PropsWithChildren) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}

describe('useItemList', () => {
  it('returns items on success', async () => {
    // API mock 설정 (msw 등)
    server.use(
      rest.get('/api/items', (req, res, ctx) =>
        res(ctx.json([{ id: '1', title: 'Item 1' }])),
      ),
    );

    const { result } = renderHook(() => useItemList('cat-1'), {
      wrapper: createQueryWrapper(),
    });

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].title).toBe('Item 1');
  });
});
```

### 네비게이션 mock

```typescript
// __mocks__/expo-router.ts
const mockRouter = { push: jest.fn(), replace: jest.fn(), back: jest.fn(), dismiss: jest.fn() };

export const useRouter = () => mockRouter;
export const useLocalSearchParams = jest.fn().mockReturnValue({});
export const Link = ({ children }: PropsWithChildren) => children;
export const Redirect = () => null;

// 테스트: expect(useRouter().push).toHaveBeenCalledWith('/item/1');
```

### 테스트 규칙
- 테스트 파일: `__tests__/` 폴더 또는 소스 파일 옆 `.test.tsx`
- 컴포넌트: 렌더링 + 사용자 인터랙션 + 결과 확인
- 훅: `renderHook` + react-query 래퍼
- API: `msw`로 네트워크 mock (fetch/axios mock 대신)
- 스냅샷 테스트: 디자인 변경 감지용 (과용 금지 — 깨지기 쉬움)
- E2E: 핵심 유저 플로우만 (로그인 -> 목록 -> 상세 -> 액션)

---

## 32. 디버깅

### React DevTools

```bash
# React DevTools 설치 및 실행
npx react-devtools

# Expo 개발 서버와 자동 연결
# 컴포넌트 트리, props, state, hooks 확인
# Profiler 탭에서 리렌더 분석
```

### 네트워크 디버깅

- `npx expo start` 후 `j` 키 — Chrome DevTools (네트워크 탭 포함)
- react-query devtools (개발 모드) — 쿼리 상태/캐시 확인
- 필요 시 `__DEV__`에서 `global.fetch` 래핑해 요청/응답 로깅

### 성능 프로파일링

- React Profiler (성능 최적화 섹션 참조) — 16ms 초과 컴포넌트 식별
- Hermes CPU 프로파일 — 함수별 실행 시간
- React DevTools → Settings → "Highlight updates" — 불필요 리렌더 시각 확인

### 일반적인 디버깅 전략
- `console.log` 대신 `console.debug` (프로덕션 빌드에서 자동 제거 가능)
- 상태 디버깅: zustand `devtools` 미들웨어 (`{ enabled: __DEV__ }`)
- 네트워크: react-query devtools (개발 모드)
- 크래시: `expo-updates` 에러 리포팅 또는 Sentry
- 레이아웃: `borderWidth: 1, borderColor: 'red'`로 경계 확인 (개발 중)

---

## 33. 문서화 규칙

> TypeScript 기본 문서화(`typescript.md`)를 따르되, React Native 특화 추가.

### 화면(Screen) 컴포넌트

```typescript
// Good: 화면 역할을 한 줄로 설명
/** Monthly calendar view with task management */
export default function CalendarScreen() { ... }

// Bad: 없거나 너무 장황
export default function CalendarScreen() { ... }
```

### 커스텀 훅

```typescript
// Good: 복잡한 훅은 사용 예시 포함
/**
 * Manages paginated item list with infinite scroll.
 *
 * @example
 * const { items, isLoading, fetchNextPage, hasNextPage } = useItemList('category-1');
 */
function useItemList(categoryId: string) { ... }

// Bad: 이름으로 충분히 명확한데 불필요한 주석
/** Auth hook */
function useAuth() { ... }
```

### Props 문서화

```typescript
// Good: 비직관적 Props만 설명
interface DatePickerProps {
  /** ISO 8601 format: "2026-03-25" */
  value: string;
  /** Returns ISO date string */
  onChange: (date: string) => void;
  /** Minimum selectable date. Default: today */
  minDate?: string;
}

// Bad: 자명한 prop에 주석 — 노이즈
interface ButtonProps {
  /** Button text */ // title: string 자체로 충분
  title: string;
  /** Press handler */ // onPress만으로 충분
  onPress: () => void;
}
```

### 주석이 필요 없는 경우 (쓰지 마라)

```typescript
// Bad: 코드가 이미 설명하는 것을 반복
// Show spinner if loading
if (isLoading) return <LoadingSpinner />;

// Bad: 누가 봐도 아는 것
const [isVisible, setIsVisible] = useState(false); // visibility state

// Good: 비직관적 로직에만 Why 설명
// iOS 16.4+ only supports PWA push — fallback to in-app notification
if (Platform.OS === 'web' && !isPushSupported) {
  showInAppNotification(message);
}

// Good: 플랫폼 차이 설명
// Android returns 'keyboard' type focus, needs explicit dismissal
if (Platform.OS === 'android') {
  Keyboard.dismiss();
}
```

---

## 34. 빌드 & 설정

### app.json / app.config.ts 필수 설정

```typescript
// app.config.ts — 동적 설정 (환경변수 활용)
import { ExpoConfig, ConfigContext } from 'expo/config';

export default ({ config }: ConfigContext): ExpoConfig => ({
  ...config,
  name: 'MyApp',
  slug: 'my-app',
  version: '1.0.0',
  scheme: 'myapp', // 딥링크 스킴
  orientation: 'portrait',
  icon: './assets/icon.png',
  // newArchEnabled: New Architecture는 현행 SDK 기본 — 끄는 설정을 추가하지 않는다 (0.82+ 무시됨)
  plugins: [
    'expo-router',
    'expo-notifications',
    'expo-secure-store',
    ['expo-image-picker', { photosPermission: 'Allow access to select photos' }],
  ],
  ios: { supportsTablet: true, bundleIdentifier: 'com.example.myapp' },
  android: {
    adaptiveIcon: { foregroundImage: './assets/adaptive-icon.png', backgroundColor: '#ffffff' },
    package: 'com.example.myapp',
  },
  web: { bundler: 'metro', favicon: './assets/favicon.png' },
  experiments: {
    typedRoutes: true, // 타입 안전 라우트
  },
});
```

### EAS Build 설정 (eas.json)

```json
{
  "build": {
    "development": { "developmentClient": true, "distribution": "internal" },
    "preview": { "distribution": "internal", "channel": "preview" },
    "production": { "channel": "production", "autoIncrement": true }
  }
}
```

### tsconfig.json 권장 설정

> 컴파일러 안전 옵션(`noUncheckedIndexedAccess` 등)의 근거 → `typescript.md` 참조. RN에서는 expo 베이스 확장이 기본:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "types": ["nativewind/types"]
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```

### metro.config.js (NativeWind)

```javascript
const { getDefaultConfig } = require('expo/metro-config');
const { withNativeWind } = require('nativewind/metro');

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: './global.css' });
```

### babel.config.js

```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      'nativewind/babel',
      'react-native-worklets/plugin', // Reanimated 4: worklets 분리 패키지의 플러그인 — 반드시 마지막
    ],
  };
};
```

> **주의**: worklets 플러그인은 반드시 plugins 배열의 **마지막**에 위치해야 한다.
> Reanimated 3 이하 레거시 프로젝트는 기존 `react-native-reanimated/plugin` 유지.

---

## 35. 자주 쓰는 라이브러리/도구

### 권장

| 용도 | 라이브러리 | 이유 |
|------|-----------|------|
| 라우팅 | **expo-router** | 파일 기반, 딥링크 자동, 웹 호환 (SDK 56+: React Navigation 포크 내장) |
| 서버 상태 | **@tanstack/react-query** | 캐싱, 재시도, 무효화, 낙관적 업데이트 |
| 클라이언트 상태 | **zustand** | 간결, 보일러플레이트 최소, 셀렉터 — react-query와의 조합이 표준 |
| 스타일링 | **NativeWind v4** | Tailwind 문법, 웹/네이티브 통일 (v4가 프로덕션 표준 — v5는 pre-release) |
| 이미지 | **expo-image** | 캐싱, 블러 해시, 트랜지션 |
| 리스트 | **@shopify/flash-list v2** | 자동 측정(estimatedItemSize 불필요), New Arch 전용 |
| 알림 | **expo-notifications** | FCM/APNs 통합 |
| 상태 persist | **react-native-mmkv** | 동기 API, AsyncStorage 대비 ~30배 |
| 토큰/민감 저장 | **expo-secure-store** | iOS Keychain / Android Keystore |
| AsyncStorage 대체 | **expo-sqlite/kv-store** | AsyncStorage API 호환 드롭인 |
| 폼 | **react-hook-form + zod** | 성능 최적, 타입 안전 검증 |
| 날짜 | **date-fns** | 경량, tree-shaking |
| 토스트 | **burnt** 또는 **react-native-toast-message** | 네이티브 느낌 |
| 아이콘 | **@expo/vector-icons** | Expo 기본 포함 |
| 애니메이션 | **react-native-reanimated 4** | CSS 애니메이션 API + worklet, New Arch 전용 (worklets 분리 패키지) |
| 제스처 | **react-native-gesture-handler** | 네이티브 제스처 인식 |
| 바텀시트 | **@gorhom/bottom-sheet** | reanimated 기반, 고성능 |

### 피해야 할 것

| 라이브러리/API | 실패 결과 | 대안 |
|-----------|------|------|
| `@react-navigation/*` 직접 설치·import (SDK 56+) | Expo Router 내장 포크와 이중 인스턴스 → 네비게이션 상태 파손 | `expo-router` export만 사용 |
| `Image` (RN 기본) | 캐싱 없음 → 스크롤마다 재다운로드/깜빡임 | `expo-image` |
| `SafeAreaView` (RN 코어) | 0.81 deprecated, iOS 전용/불완전 | `react-native-safe-area-context` |
| `TouchableOpacity` | 유지보수 모드 — 신규 인터랙션/웹 미지원 | `Pressable` |
| `@react-native-async-storage/async-storage` (신규 도입) | 느림 + 무암호화 — 토큰 저장 시 평문 노출 | mmkv / secure-store / expo-sqlite kv-store |
| `moment.js` | deprecated, 번들 290KB+ | `date-fns`, `dayjs` |
| `redux` (소규모) | 과도한 보일러플레이트 — 개발 속도 저하 | `zustand` |
| `axios` | 대부분 과도함 + 번들 증가 | `fetch` + react-query |
| `react-native-fast-image` | New Arch/Expo 호환 뒤처짐 | `expo-image` |

---

## 36. 실전 함정 & 지뢰

> 함정 = "정상으로 보이지만 특정 조건에서 깨지는 것" — 안티패턴보다 위험하다 (경험자도 당함).
> 작업 중 발견한 함정은 범용화(프로젝트 용어 제거) 후 여기에 4요소로 누적한다.

| # | 증상 | 원인 | 해결 | 오답 (하지 말 것) |
|---|------|------|------|-------------------|
| 1 | New Arch 전환 후 짧은 강조/로딩 등 "중간 UI"가 화면에 안 나타남 | New Architecture는 모든 setState를 항상 배칭 — 레거시에서 `setTimeout`/네이티브 콜백 안 setState가 각각 렌더되던 동작에 의존한 코드 | 중간 상태 의존 제거 — 시간차 UI는 명시적 스케줄링/애니메이션으로 (New Architecture 섹션 참조) | `setTimeout(0)` 중첩으로 렌더 틈새 만들기 |
| 2 | 특정 라이브러리 기능이 New Arch에서 미동작/크래시 (레거시에선 정상이었음) | 레거시 네이티브 모듈이 interop layer로 도는 중 — interop은 한시적이며 concurrent 미지원 | New Arch 지원 라이브러리로 교체 (reactnative.directory 확인 + `npx expo-doctor`) | `newArchEnabled: false` 옵트아웃 (0.82+ 무시됨, SDK 55+ 레거시 제거) |
| 3 | SDK 업그레이드 후 네비게이션 훅/컴포넌트가 파손 | `@react-navigation/*`을 직접 설치·import — SDK 56+에서 Expo Router가 React Navigation을 포크 내장, 별도 설치분과 이중 인스턴스 | expo-router가 export하는 API로 전량 교체 + 직접 설치분 제거 | `@react-navigation/*` 버전 고정으로 억지 유지 (다음 SDK에서 또 파손) |
| 4 | iOS에서 키보드가 올라오면 입력창이 헤더 높이만큼 가려짐 (KeyboardAvoidingView 썼는데도) | `behavior="padding"`은 화면 최상단 기준 계산 — 네비게이션 헤더 높이를 모름 (`keyboardVerticalOffset` 미전달) | 실제 헤더 높이(자체 헤더 상수/onLayout 실측)를 `keyboardVerticalOffset`에 전달 | 특정 기기에서 맞는 매직 넘버(예: 88) 하드코딩 |
| 5 | Reanimated worklet 안에서 읽는 바깥 변수가 항상 옛 값 | worklet은 생성 시점 클로저를 UI 스레드로 **직렬화 캡처** — JS 스레드에서의 이후 변경을 모름 | 변하는 값은 `useSharedValue`로 전달, JS 함수 호출은 `runOnJS` | 전역 변수/ref로 공유 시도 (같은 이유로 안 됨) |
| 6 | 라우트 파라미터로 받은 숫자 연산이 오동작 (`"1" + 1 = "11"`, 필터 미적용) | URL 파라미터는 런타임에 항상 `string` 또는 `string[]` — `useLocalSearchParams<T>()` 제네릭은 캐스팅일 뿐 | 경계에서 zod로 파싱 (`z.coerce.number()` 등) 후 사용 (네비게이션 섹션 참조) | `as unknown as number` 이중 단언 |
| 7 | iOS 포그라운드에서 푸시 알림 배너가 표시되지 않음 (권한도 정상) | 구 예제의 `shouldShowAlert` 복사 — SDK 53+에서 deprecated (iOS 14 배너/알림센터 분리) | handler에 `shouldShowBanner: true` + `shouldShowList: true` 지정 | 알림 권한 요청 로직만 반복 수정 (원인이 아님) |
| 8 | Android 뒤로가기(버튼/제스처)에 커스텀 처리가 안 먹거나 앱이 그냥 종료 | `BackHandler`로 화면 이탈을 막으려 함 — 네비게이터의 back 처리와 경합, 제스처 내비게이션에서 동작 상이 | 화면 이탈 방어는 네비게이션 이벤트(이탈 방지 훅/리스너)로 처리, BackHandler는 최후 수단 | BackHandler에서 항상 `true` 반환 (시스템 back 전부 삼켜 앱 탈출 불가) |
| 9 | 웹 새로고침(F5) 시 로그인 풀림 | 인증 스토리지에 네이티브 전용 storage 지정 — 웹에서 미작동 | `Platform.OS === 'web' ? undefined : nativeStorage`로 분기 (웹은 기본 localStorage) | 네이티브 storage를 웹 polyfill로 대체 (불필요한 복잡성) |
| 10 | 탭 외 화면(상세 등)에서 탭바 미표시 | 탭바가 `(tabs)/_layout.tsx`에만 존재 — Stack 밖 화면에는 적용 안 됨 | 커스텀 탭바를 루트 레이아웃에 배치 또는 탭 내 중첩 네비게이션 | 모든 화면을 탭 안에 억지로 넣기 |
| 11 | 모바일에서 탭바/하단 버튼이 홈 인디케이터에 잘림 | 하단 안전 영역 미반영 | `useSafeAreaInsets()` + `paddingBottom: insets.bottom` | 고정 값 하드코딩 (기기마다 다름) |
| 12 | `contentContainerClassName` 스타일이 웹에서 깨짐 | NativeWind의 해당 prop은 웹 미지원 | `contentContainerStyle` 사용 | 다른 className 조합으로 재시도 |
| 13 | 이미지 피커가 웹에서 크래시 | `launchCameraAsync`는 웹 미지원 | `Platform.OS === 'web'`이면 `launchImageLibraryAsync`만 사용 | 카메라 기능을 웹에서도 시도 |

---

## 37. 검증 체크리스트

### 새 화면 추가 시

```
- [ ] SafeAreaView(react-native-safe-area-context) 최상위 래핑 (flex: 1, backgroundColor 명시)
- [ ] 4가지 상태 처리 (로딩/에러/빈 데이터/정상)
- [ ] maxWidth 상수 + width: '100%' + alignSelf: 'center' 3종 세트 (웹 대비)
- [ ] ScrollView는 contentContainerStyle 사용 (contentContainerClassName 금지)
- [ ] 폼 화면: KeyboardAvoidingView + keyboardShouldPersistTaps="handled" + 헤더 오프셋
- [ ] 인증 필요 여부 확인 (_layout.tsx 가드)
- [ ] 웹 확인: npx expo start --web (데스크톱 너비 포함)
```

### 실기기 테스트 전

```
- [ ] npx expo-doctor — 의존성/설정 진단 통과
- [ ] npx expo install --check — SDK 비호환 패키지 0건
- [ ] 네이티브 모듈 추가/변경 시 dev build 재생성 (eas build --profile development) — Expo Go로는 검증 불가
- [ ] 권한 플로우를 "거부" 케이스 포함해 확인 (알림/카메라/사진)
- [ ] 노치·제스처 내비게이션 기기에서 SafeArea 확인 (상단 겹침/하단 잘림)
- [ ] 성능 체감은 release 빌드로만 판단 (npx expo run:android --variant release) — dev 빌드는 수 배 느림
```

### 스토어 제출 전

```
- [ ] eas build --profile production 성공 + 산출물 실제 설치·실행 확인
- [ ] 버전/빌드넘버 증가 확인 (eas.json autoIncrement)
- [ ] iOS 권한 문구(NS*UsageDescription)가 실제 사용 기능과 일치 — 불일치 시 심사 리젝
- [ ] 프로덕션 환경변수 적용 확인 — 개발 API URL 잔존 금지 (.env.production)
- [ ] console.log 제거 확인 (babel-plugin-transform-remove-console 또는 grep -rn "console.log" src/ app/)
- [ ] 딥링크 동작 확인 (npx uri-scheme open myapp://item/1 --ios)
- [ ] EAS Update 채널이 production인지 확인 (preview 채널로 배포 사고 방지)
```

### 성능 점검

```
- [ ] release 빌드에서 측정 (dev 모드 측정치는 무의미)
- [ ] React DevTools "Highlight updates"로 불필요 리렌더 확인
- [ ] Profiler로 16ms(60fps) 초과 컴포넌트 식별
- [ ] 긴 리스트가 FlashList/FlatList인지 + 아이템 컴포넌트 분리 확인 (ScrollView + .map 금지)
- [ ] eslint-plugin-react-compiler 경고 0건 (컴파일러 최적화 제외 컴포넌트 없음)
- [ ] 이미지: expo-image + 업로드 전 리사이즈 + 리스트 recyclingKey
```

### 웹 호환 확인

```
- [ ] npx expo start --web 부팅 + 콘솔 에러 0건
- [ ] 로그인 → F5 새로고침 → 세션 유지
- [ ] grep -rn "Alert.alert" src/ app/ — 전 사용처에 웹 분기 존재
- [ ] grep -rn "contentContainerClassName" src/ app/ — 0건
- [ ] window/document 직접 접근에 Platform.OS === 'web' 가드
- [ ] 데스크톱 너비(1280px+)에서 maxWidth 제한 동작
```

### 에러 발생 시 디버깅 흐름

1. **JS 에러인지 네이티브 크래시인지 판별** — 레드박스/LogBox(JS) vs 앱 즉사(네이티브: `adb logcat` / Xcode 콘솔)
2. **재현 환경 좁히기** — Expo Go / dev build / release 중 어디서만 나는가 (네이티브 모듈 문제는 Expo Go에서 재현 안 됨)
3. 최근 추가한 라이브러리의 **New Architecture 호환** 확인 (reactnative.directory + `npx expo-doctor`)
4. 플랫폼 한정이면 `Platform` 분기/플랫폼별 파일(`.ios.tsx` 등) 확인
5. 상태/리렌더 문제면 React DevTools로 상태 흐름 추적 (react-query devtools 포함)
6. 재현 최소화 — 새 화면에 해당 컴포넌트만 격리해 원인 이분탐색

---

## 38. 우회/핵 금지 원칙

> 증상 땜빵이 아닌 근본 원인을 진단한다. "일단 돌아가게"는 다음 SDK 업그레이드 때 시한폭탄이 된다.

| 우회 패턴 (금지) | 왜 위험한가 | 정석 해결 |
|-----------------|-----------|----------|
| `newArchEnabled: false`로 호환 문제 회피 | 0.82+에서 무시 + SDK 55에서 레거시 제거 — 해결된 척만 하고 파국을 이월 | 비호환 라이브러리 교체/업데이트 (New Architecture 섹션) |
| Expo Go에서만 확인하고 "동작 확인" 선언 | 네이티브 모듈/권한/딥링크는 Expo Go와 실빌드 동작이 다름 | dev build + 실기기 검증 (체크리스트 참조) |
| `npm install --force` / `--legacy-peer-deps`로 버전 충돌 강행 | Expo SDK 호환 매트릭스 파괴 → 빌드는 되는데 런타임 네이티브 크래시 | `npx expo install` + `npx expo-doctor`로 정합 유지 |
| `LogBox.ignoreLogs`로 경고 침묵 | 실제 결함(cleanup 누락, 중복 키, deprecated API) 은폐 — 릴리즈에서 버그로 발현 | 경고의 원인을 수정, ignore는 서드파티 발 소음 한정 + 사유 주석 |
| 웹 깨짐을 `Platform.OS === 'web'` 대량 분기로 덮기 | 코드 이중화 — 한쪽만 수정되는 불일치가 누적 | 크로스플랫폼 API 우선, 분기는 최소 지점(헬퍼 1곳)으로 격리 |
| 키보드/SafeArea 문제를 매직 넘버 padding으로 봉합 | 특정 기기에서만 맞음 — 다른 기기/회전/폴더블에서 파손 | insets/실측 높이 기반 계산 (키보드 처리 섹션) |
| 타입 에러를 `as any`로 봉합 | `typescript.md` 우회 금지 원칙 참조 | 타입 가드/스키마 검증 |

---

## 39. 프로젝트별 확장 포인트

> 아래 항목들은 프로젝트마다 다르므로 `/CLAUDE.md`에서 정의합니다:

- 앱 이름, 번들 ID, 딥링크 스킴 / EAS 프로젝트 ID
- 프로젝트 구조 상세 (features 폴더 목록)
- 빌드/실행 명령 (`npx expo start`, `eas build`) + CI/CD 파이프라인
- 환경 변수 목록 및 값
- 배포 전략 (EAS Update 채널, 스토어)
- 외부 서비스 연동 (백엔드, 푸시, 에러 리포팅 등)
- 디자인 시스템 (색상, 폰트, 간격 토큰) / 앱 아이콘, 스플래시
- 소셜 로그인 프로바이더 설정
- API 엔드포인트 및 인증 방식
- maxWidth 상수 값 (MAX_FORM_WIDTH, MAX_CONTENT_WIDTH 등)

<!-- 개선 이력
- 2026-03-27: 실전 함정 섹션 추가 — AsyncStorage 웹 세션 유실, 탭바 미표시/잘림 발견
- 2026-03-27: 검증 체크리스트 추가 — 새 화면/배포 전 크로스 플랫폼 확인
- 2026-07-03: 2026-07 기준 전면 개편 (74점 → Level 3 상위 목표 + 100KB→65KB 다이어트).
  신설: New Architecture 섹션(0.82+ 옵트아웃 무시/SDK 55 레거시 제거/state 배칭/interop 한시성/호환 확인법),
  우회·핵 금지 원칙(7행). deprecated 청소: shouldShowAlert→shouldShowBanner+shouldShowList,
  RN 코어 SafeAreaView→safe-area-context, LayoutAnimation→Reanimated entering/exiting.
  방향 반전: React Compiler(SDK 54+) 기본 활성 — 수동 useMemo/useCallback/React.memo 지양으로 성능·훅·리스트 지침 교체.
  최신화: FlashList v2(estimatedItemSize 제거, New Arch 전용), Reanimated 4(worklets 분리 패키지/CSS API),
  Expo Router SDK 56+ React Navigation 포크(직접 import 금지), 스토리지 기준(mmkv/secure-store/expo-sqlite kv-store).
  실전 함정 5→13행(4요소), 검증 체크리스트 2→6종+디버깅 흐름, 안티패턴 "이유"→"실패 결과" 전환.
  다이어트: typescript.md 중복 위임(비동기 타임아웃/커스텀 에러 계층/옵셔널 처리/export 일반론),
  중복 예시 통합(AsyncList, memo 블록 3중, useConfirmDialog, LayoutAnimation 등). 근거: 2026-07 웹 리서치 확정 팩트.
-->
