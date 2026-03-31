---
paths:
  - "**/*.tsx"
  - "**/*.ts"
  - "app.json"
  - "app.config.ts"
  - "babel.config.js"
  - "metro.config.js"
---

# React Native + Expo 개발 규칙

> 이 파일은 React Native (Expo) 프로젝트에 범용적으로 적용되는 시니어 수준 규칙입니다.
> TypeScript 기본 규칙은 `rules/typescript.md`를 참조합니다. 여기서는 React Native/Expo 특화 규칙만 다룹니다.
> 프로젝트별 설정, 빌드 명령, 폴더 구조는 `/CLAUDE.md`에 정의합니다.

---

## 1. 핵심 원칙

- **모바일 퍼스트**: 모바일 경험이 최우선. 웹은 보조 — 터치 영역(44x44pt), 스크롤 성능, 오프라인 대응을 항상 고려한다.
- **네이티브 느낌**: 웹을 모바일에 옮긴 것이 아니라, 네이티브 앱처럼 느껴져야 한다 (애니메이션, 제스처, 햅틱).
- **성능 = UX**: 60fps 유지가 기본. 렌더링 병목, 메모리 릭, 과도한 리렌더를 적극적으로 방지한다.
- **플랫폼 차이 인식**: iOS/Android/Web은 동작이 다르다. `Platform.OS`로 분기가 필요한 지점을 미리 파악한다.
- **Expo 생태계 우선**: 가능하면 Expo SDK 모듈을 사용한다. 네이티브 모듈이 꼭 필요한 경우만 개발 빌드(dev client)로 전환한다.
- **4가지 상태 필수**: 모든 화면은 로딩/에러/빈 데이터/정상 상태를 반드시 처리한다.

---

## 2. 프로젝트 구조 (Expo Router 기반)

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

## 3. 네이밍 컨벤션

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
| enum 값 | PascalCase | `UserRole.Admin`, `Status.Active` |

```typescript
// 컴포넌트: PascalCase, 역할이 드러나는 이름
export function ItemCard({ item }: ItemCardProps) { ... }
export function EmptyListView() { ... }

// 화면: ~Screen 접미어 (프로젝트에서 통일)
export default function DetailScreen() { ... }

// 레이아웃: ~Layout
export default function TabLayout() { ... }
```

---

## 4. 코드 포맷팅

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

## 5. import/export 규칙

```typescript
// 1. React / React Native 코어
import React, { useState, useCallback, useMemo } from 'react';
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

- **화면 컴포넌트**: `export default` (Expo Router 필수)
- **재사용 컴포넌트**: `named export` 우선 (tree-shaking, 자동 import 용이)
- **훅**: `named export`
- **타입**: `export type` / `export interface` 사용
- **배럴 파일(index.ts)**: 기능 폴더 단위로 re-export, 2단계 이상 깊이 금지

```typescript
// src/features/auth/index.ts — 배럴 파일
export { useAuth } from './hooks/useAuth';
export { LoginButton } from './components/LoginButton';
export type { AuthUser, AuthSession } from './types';
```

---

## 6. 파일 내부 코드 순서 (컴포넌트 파일)

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

  // 4-2. 파생 값 (useMemo, 단순 계산)
  const formattedDate = useMemo(() => formatDate(item.createdAt), [item.createdAt]);

  // 4-3. 콜백 (useCallback)
  const handlePress = useCallback(() => {
    onPress(item.id);
  }, [item.id, onPress]);

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

## 7. 컴포넌트 설계 패턴

### 기본 컴포넌트

```typescript
interface ItemCardProps {
  item: Item;
  onPress: (itemId: string) => void;
  isHighlighted?: boolean;
}

export function ItemCard({ item, onPress, isHighlighted = false }: ItemCardProps) {
  const handlePress = useCallback(() => {
    onPress(item.id);
  }, [item.id, onPress]);

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
- **리스트 아이템** -> 반드시 별도 컴포넌트 (`React.memo` 적용 위해)

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

// 합성 컴포넌트 구현
function Card({ children }: PropsWithChildren) {
  return <View className="bg-white rounded-xl border border-gray-200">{children}</View>;
}

Card.Header = function CardHeader({ title }: { title: string }) {
  return (
    <View className="px-4 py-3 border-b border-gray-100">
      <Text className="text-lg font-bold">{title}</Text>
    </View>
  );
};

Card.Body = function CardBody({ children }: PropsWithChildren) {
  return <View className="p-4">{children}</View>;
};

Card.Footer = function CardFooter({ children }: PropsWithChildren) {
  return <View className="px-4 py-3 border-t border-gray-100">{children}</View>;
};
```

### Render Props 패턴 (고급)

```typescript
// 리스트 래퍼: 로딩/에러/빈 상태를 자동 처리
interface AsyncListProps<T> {
  queryResult: UseQueryResult<T[]>;
  renderItem: (item: T) => ReactNode;
  emptyMessage: string;
  estimatedItemSize: number;
}

function AsyncList<T extends { id: string }>({
  queryResult,
  renderItem,
  emptyMessage,
  estimatedItemSize,
}: AsyncListProps<T>) {
  const { data, isLoading, error, refetch } = queryResult;

  if (isLoading) return <LoadingSkeleton />;
  if (error) return <ErrorView error={error} onRetry={refetch} />;
  if (!data?.length) return <EmptyView message={emptyMessage} />;

  return (
    <FlashList
      data={data}
      renderItem={({ item }) => renderItem(item)}
      estimatedItemSize={estimatedItemSize}
      keyExtractor={(item) => item.id}
    />
  );
}
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

## 8. 훅(Hooks) 규칙

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
// 파생 값 계산 -> useMemo 또는 렌더링 중 계산

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

### useCallback / useMemo

```typescript
// useCallback: memo된 자식에 전달하는 콜백에만 사용
const handlePress = useCallback((id: string) => {
  router.push(`/item/${id}`);
}, [router]);

// useMemo: 비용이 큰 계산에만
const sortedItems = useMemo(
  () => [...items].sort((a, b) => a.date.localeCompare(b.date)),
  [items],
);

// useMemo: 불변 참조 유지 (자식 컴포넌트 리렌더 방지)
const filterOptions = useMemo(
  () => ({ status: activeStatus, category: selectedCategory }),
  [activeStatus, selectedCategory],
);

// Bad: 모든 곳에 남용 -> 오히려 메모리 낭비
const title = useMemo(() => `Hello ${name}`, [name]); // 단순 문자열에 불필요
```

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

// Good: mutation 로직 캡슐화
function useCreateItem() {
  const queryClient = useQueryClient();
  const router = useRouter();

  const { mutate, isPending } = useMutation({
    mutationFn: createItem,
    onSuccess: (newItem) => {
      queryClient.invalidateQueries({ queryKey: ['items'] });
      router.push(`/item/${newItem.id}`);
    },
    onError: (error) => {
      showToast(error instanceof AppError ? error.userMessage : 'Failed to create');
    },
  });

  return { create: mutate, isCreating: isPending };
}
```

---

## 9. 폼 패턴 (react-hook-form + zod)

### zod 스키마 정의

```typescript
import { z } from 'zod';

// 스키마를 컴포넌트 외부에 정의 (재사용 + 렌더링 시 재생성 방지)
const createItemSchema = z.object({
  title: z
    .string()
    .min(2, '제목은 2자 이상 입력해주세요')
    .max(50, '제목은 50자 이하로 입력해주세요'),
  description: z
    .string()
    .max(500, '설명은 500자 이하로 입력해주세요')
    .optional(),
  maxCount: z
    .number()
    .min(2, '최소 2명 이상')
    .max(100, '최대 100명'),
  price: z
    .number()
    .min(0, '가격은 0 이상이어야 합니다'),
  date: z
    .string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, '올바른 날짜 형식이 아닙니다'),
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
    defaultValues: {
      title: '',
      description: '',
      maxCount: 8,
      price: 0,
      date: '',
    },
    mode: 'onBlur', // 포커스 해제 시 검증 (타이핑 중 에러 방지)
  });

  const onSubmit = handleSubmit((data) => {
    create(data);
  });

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={{ flex: 1 }}
      >
        <ScrollView
          contentContainerStyle={{ padding: 16 }}
          keyboardShouldPersistTaps="handled"
        >
          {/* 텍스트 입력 */}
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
                  placeholder="Enter title"
                  maxLength={50}
                />
                {errors.title && (
                  <Text className="text-red-500 text-xs mt-1">{errors.title.message}</Text>
                )}
              </View>
            )}
          />

          {/* 숫자 입력 */}
          <Controller
            control={control}
            name="maxCount"
            render={({ field: { onChange, value } }) => (
              <View className="mb-4">
                <Text className="text-sm font-medium text-gray-700 mb-1">Max Count</Text>
                <View className="flex-row items-center">
                  <Pressable
                    onPress={() => onChange(Math.max(2, value - 1))}
                    className="w-10 h-10 items-center justify-center bg-gray-100 rounded-lg"
                  >
                    <Text className="text-lg">-</Text>
                  </Pressable>
                  <Text className="mx-4 text-lg font-bold">{value}</Text>
                  <Pressable
                    onPress={() => onChange(Math.min(100, value + 1))}
                    className="w-10 h-10 items-center justify-center bg-gray-100 rounded-lg"
                  >
                    <Text className="text-lg">+</Text>
                  </Pressable>
                </View>
                {errors.maxCount && (
                  <Text className="text-red-500 text-xs mt-1">{errors.maxCount.message}</Text>
                )}
              </View>
            )}
          />

          {/* 제출 버튼 */}
          <Button
            title={isCreating ? 'Creating...' : 'Create'}
            onPress={onSubmit}
            disabled={!isValid || isCreating}
          />
        </ScrollView>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}
```

### 공통 폼 입력 컴포넌트

```typescript
// 재사용 가능한 폼 필드 래퍼
interface FormFieldProps {
  label: string;
  error?: string;
  children: ReactNode;
}

function FormField({ label, error, children }: FormFieldProps) {
  return (
    <View className="mb-4">
      <Text className="text-sm font-medium text-gray-700 mb-1">{label}</Text>
      {children}
      {error && <Text className="text-red-500 text-xs mt-1">{error}</Text>}
    </View>
  );
}

// 사용
<FormField label="Title" error={errors.title?.message}>
  <Controller
    control={control}
    name="title"
    render={({ field: { onChange, onBlur, value } }) => (
      <TextInput value={value} onChangeText={onChange} onBlur={onBlur} />
    )}
  />
</FormField>
```

### 폼 규칙

- 스키마는 컴포넌트 밖에 정의 (렌더링 시 재생성 방지)
- `mode: 'onBlur'` 기본 — 타이핑 중 에러 메시지 깜빡임 방지
- 숫자 입력은 `TextInput` + `keyboardType="numeric"` 또는 증감 버튼
- 제출 중 버튼 비활성화 (`disabled={!isValid || isPending}`)
- `keyboardShouldPersistTaps="handled"` — 키보드 열린 상태에서 버튼 탭 가능
- 에러 메시지는 필드 바로 아래 빨간 텍스트로 표시

---

## 10. 상태 관리 (zustand + react-query)

### 역할 분리 원칙

| 데이터 종류 | 도구 | 예시 |
|------------|------|------|
| 서버 상태 (DB 데이터) | **react-query** | 아이템 목록, 프로필, 알림 |
| UI 상태 (화면 내) | **useState** | 모달 열림, 폼 입력값, 토글 |
| 글로벌 클라이언트 상태 | **zustand** | 인증 세션, 앱 설정, 테마 |
| URL 상태 | **expo-router params** | 현재 선택된 ID, 필터 |

> **원칙**: 서버에서 온 데이터는 반드시 react-query로 관리한다. zustand에 서버 데이터를 복사하지 않는다.

### zustand 스토어 패턴

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

// 슬라이스 인터페이스
interface AuthSlice {
  session: Session | null;
  setSession: (session: Session | null) => void;
  clearSession: () => void;
}

interface SettingsSlice {
  theme: 'light' | 'dark' | 'system';
  notificationsEnabled: boolean;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  toggleNotifications: () => void;
}

// 스토어 합성 (슬라이스 패턴)
interface AppStore extends AuthSlice, SettingsSlice {}

export const useAppStore = create<AppStore>()(
  persist(
    (set) => ({
      // Auth slice
      session: null,
      setSession: (session) => set({ session }),
      clearSession: () => set({ session: null }),

      // Settings slice
      theme: 'system',
      notificationsEnabled: true,
      setTheme: (theme) => set({ theme }),
      toggleNotifications: () =>
        set((state) => ({ notificationsEnabled: !state.notificationsEnabled })),
    }),
    {
      name: 'app-store',
      storage: createJSONStorage(() => AsyncStorage),
      partialize: (state) => ({
        // 영속할 상태만 선택 (session은 secure-store에 별도 저장)
        theme: state.theme,
        notificationsEnabled: state.notificationsEnabled,
      }),
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

## 11. 네비게이션 (Expo Router)

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

### 타입 안전한 라우트 파라미터

```typescript
// 라우트 파라미터 타입 정의
type ItemRouteParams = {
  id: string;
};

// 사용
export default function ItemDetailScreen() {
  const { id } = useLocalSearchParams<ItemRouteParams>();

  // id가 없으면 에러 화면
  if (!id) return <ErrorView message="Invalid item ID" />;

  return <ItemDetail itemId={id} />;
}
```

---

## 12. 스타일링 (NativeWind / Tailwind)

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
// shared/styles/theme.ts
export const colors = {
  primary: '#2e7d32',
  primaryDark: '#1b5e20',
  error: '#e53935',
  warning: '#f57c00',
  info: '#1565c0',
  background: '#f0f2f5',
  surface: '#ffffff',
  gray: {
    50: '#fafafa',
    100: '#f5f5f5',
    300: '#e0e0e0',
    500: '#9e9e9e',
    700: '#616161',
    900: '#212121',
  },
} as const;

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
} as const;

export const borderRadius = {
  sm: 8,
  md: 12,
  lg: 16,
  xl: 20,
  full: 9999,
} as const;
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

## 13. 반응형 디자인 + 웹 호환성 + 화면 생성 체크리스트

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
import { useWindowDimensions } from 'react-native';

function useDeviceType() {
  const { width } = useWindowDimensions();
  return {
    isMobile: width < 768,
    isTablet: width >= 768 && width < 1024,
    isDesktop: width >= 1024,
  } as const;
}

// 사용: 모바일은 풀스크린, 데스크톱은 사이드바
function AppLayout({ children }: PropsWithChildren) {
  const { isDesktop } = useDeviceType();

  if (isDesktop) {
    return (
      <View className="flex-row max-w-[1200px] mx-auto">
        <View className="w-64 border-r border-gray-200">
          <Sidebar />
        </View>
        <View className="flex-1">{children}</View>
      </View>
    );
  }

  return <View className="flex-1">{children}</View>;
}
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
// Bad: 웹에서 크래시
Alert.alert('Confirm', 'Are you sure?', [
  { text: 'Cancel' },
  { text: 'OK', onPress: handleConfirm },
]);

// Good: 크로스플랫폼 확인 다이얼로그
function useConfirmDialog() {
  const [state, setState] = useState<{
    visible: boolean;
    title: string;
    message: string;
    onConfirm: () => void;
  }>({ visible: false, title: '', message: '', onConfirm: () => {} });

  const confirm = useCallback((title: string, message: string, onConfirm: () => void) => {
    if (Platform.OS === 'web') {
      // 웹: window.confirm 사용
      if (window.confirm(`${title}\n${message}`)) {
        onConfirm();
      }
    } else {
      // 네이티브: Alert.alert 사용
      Alert.alert(title, message, [
        { text: 'Cancel', style: 'cancel' },
        { text: 'OK', onPress: onConfirm },
      ]);
    }
  }, []);

  return { confirm };
}
```

**3. SafeAreaView 필수 래핑**

```typescript
import { SafeAreaView } from 'react-native-safe-area-context';

// Good: 모든 화면 최상위
export default function SomeScreen() {
  return (
    <SafeAreaView style={{ flex: 1, backgroundColor: '#f0f2f5' }}>
      <ScrollView contentContainerStyle={{ padding: 16 }}>
        {/* content */}
      </ScrollView>
    </SafeAreaView>
  );
}
```

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

### 화면 생성 체크리스트

| # | 항목 | 확인 |
|---|------|------|
| 1 | `SafeAreaView`로 최상위 래핑 | `style={{ flex: 1, backgroundColor }}` |
| 2 | `maxWidth` 제한 적용 | 용도에 맞는 상수 사용 |
| 3 | `width: '100%'` + `alignSelf: 'center'` 세트 | maxWidth와 함께 3종 세트 |
| 4 | ScrollView는 `contentContainerStyle` 사용 | `contentContainerClassName` 금지 |
| 5 | 배경색 명시 | 투명 배경 -> 플랫폼별 기본색 불일치 방지 |
| 6 | 4가지 상태 처리 | 로딩 / 에러 / 빈 데이터 / 정상 |
| 7 | 키보드 회피 (폼 화면) | `KeyboardAvoidingView` + `keyboardShouldPersistTaps` |
| 8 | 웹 확인 | `npx expo start --web`으로 데스크톱 너비에서 확인 |

```typescript
// 화면 생성 템플릿
import { SafeAreaView, ScrollView } from 'react-native';

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
// 반응형 컨테이너
function Container({ children, className }: PropsWithChildren<{ className?: string }>) {
  return (
    <View className={`flex-1 px-4 md:px-8 lg:max-w-[1200px] lg:mx-auto lg:px-12 ${className ?? ''}`}>
      {children}
    </View>
  );
}

// 반응형 그리드
function ResponsiveGrid({ children }: PropsWithChildren) {
  return (
    <View className="flex-col md:flex-row md:flex-wrap">
      {React.Children.map(children, (child) => (
        <View className="w-full md:w-1/2 lg:w-1/3 p-2">{child}</View>
      ))}
    </View>
  );
}

// 웹에서만 hover
<Pressable className="bg-white active:bg-gray-100 web:hover:bg-gray-50 rounded-xl p-4">
```

---

## 14. 애니메이션 & 제스처

### 언제 어떤 방법을 쓸 것인가

| 상황 | 방법 | 예시 |
|------|------|------|
| 단순 마운트/언마운트 전환 | `LayoutAnimation` | 리스트 아이템 추가/삭제 |
| Animated API (기본) | `Animated` (RN 내장) | 페이드인, 슬라이드 (간단) |
| 60fps 필수 제스처 연동 | `react-native-reanimated` | 스와이프 삭제, 바텀시트 드래그 |
| 제스처 인식 | `react-native-gesture-handler` | 핀치, 팬, 롱프레스 |
| 간단한 Animated.View 전환 | `Animated.timing / spring` | 버튼 스케일, 뱃지 펄스 |

### LayoutAnimation (가장 간단)

```typescript
import { LayoutAnimation, UIManager, Platform } from 'react-native';

// Android에서는 수동 활성화 필요
if (Platform.OS === 'android' && UIManager.setLayoutAnimationEnabledExperimental) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

function ItemList() {
  const [items, setItems] = useState<Item[]>([]);

  const addItem = (item: Item) => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
    setItems(prev => [item, ...prev]);
  };

  const removeItem = (id: string) => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
    setItems(prev => prev.filter(i => i.id !== id));
  };

  return (/* ... */);
}
```

### Reanimated Worklet (고급)

```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
  interpolate,
  Extrapolation,
  runOnJS,
} from 'react-native-reanimated';

function AnimatedCard({ onDismiss }: { onDismiss: () => void }) {
  const translateX = useSharedValue(0);
  const opacity = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
    opacity: opacity.value,
  }));

  const dismiss = () => {
    translateX.value = withTiming(-300, { duration: 200 });
    opacity.value = withTiming(0, { duration: 200 }, (finished) => {
      if (finished) {
        runOnJS(onDismiss)(); // UI 스레드 -> JS 스레드 전환
      }
    });
  };

  return (
    <Animated.View style={animatedStyle}>
      {/* card content */}
    </Animated.View>
  );
}
```

### Gesture Handler + Reanimated (스와이프 삭제)

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
          runOnJS(onDelete)();
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
- `useNativeDriver: true` 가능하면 항상 사용 (transform, opacity만 지원)
- Reanimated worklet에서 `console.log` 금지 (크래시) -> `runOnJS`로 감싸기
- 웹에서 Reanimated/GestureHandler 호환성 확인 필수

---

## 15. 키보드 처리

### KeyboardAvoidingView (iOS/Android 차이)

```typescript
import { KeyboardAvoidingView, Platform, ScrollView } from 'react-native';

// 폼 화면의 기본 구조
export default function FormScreen() {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={{ flex: 1 }}
        keyboardVerticalOffset={Platform.OS === 'ios' ? 0 : 20}
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

### 키보드 상태 감지

```typescript
import { Keyboard, Platform } from 'react-native';

function useKeyboardVisible() {
  const [isKeyboardVisible, setKeyboardVisible] = useState(false);

  useEffect(() => {
    const showEvent = Platform.OS === 'ios' ? 'keyboardWillShow' : 'keyboardDidShow';
    const hideEvent = Platform.OS === 'ios' ? 'keyboardWillHide' : 'keyboardDidHide';

    const showSub = Keyboard.addListener(showEvent, () => setKeyboardVisible(true));
    const hideSub = Keyboard.addListener(hideEvent, () => setKeyboardVisible(false));

    return () => {
      showSub.remove();
      hideSub.remove();
    };
  }, []);

  return isKeyboardVisible;
}
```

### 키보드 닫기

```typescript
import { Keyboard, TouchableWithoutFeedback } from 'react-native';

// 화면 아무 곳 탭 시 키보드 닫기
function DismissKeyboardView({ children }: PropsWithChildren) {
  return (
    <TouchableWithoutFeedback onPress={Keyboard.dismiss} accessible={false}>
      <View style={{ flex: 1 }}>{children}</View>
    </TouchableWithoutFeedback>
  );
}

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
- `keyboardShouldPersistTaps="handled"` — 키보드 열린 상태에서 버튼 탭 가능
- 다중 입력 폼: `returnKeyType` + `onSubmitEditing`으로 포커스 연결
- 키보드 위 고정 버튼: `position: 'absolute'` + keyboard height offset

---

## 16. 리스트 최적화

```typescript
import { FlashList } from '@shopify/flash-list';

<FlashList
  data={items}
  renderItem={({ item }) => <ItemCard item={item} />}
  estimatedItemSize={80}           // 필수: 대략적 아이템 높이
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

### 리스트 아이템 memo

```typescript
// 반드시 memo — 리스트 스크롤 성능의 핵심
const ItemCard = React.memo(function ItemCard({ item, onPress }: ItemCardProps) {
  const handlePress = useCallback(() => {
    onPress(item.id);
  }, [item.id, onPress]);

  return (
    <Pressable onPress={handlePress} className="bg-white rounded-xl p-4 border border-gray-200">
      <Text className="text-base font-bold">{item.title}</Text>
    </Pressable>
  );
});

// 커스텀 비교 함수 (선택 — 복잡한 객체에서 특정 필드만 비교)
const ItemCard = React.memo(
  function ItemCard({ item, onPress }: ItemCardProps) { /* ... */ },
  (prev, next) => prev.item.id === next.item.id && prev.item.updatedAt === next.item.updatedAt,
);
```

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

  const items = useMemo(
    () => data?.pages.flatMap(page => page) ?? [],
    [data],
  );

  return (
    <FlashList
      data={items}
      renderItem={({ item }) => <ItemCard item={item} />}
      estimatedItemSize={80}
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
- 리스트 아이템: `React.memo` 적용 필수
- `keyExtractor`: 고유 ID 사용 (index 사용 금지 — 순서 변경 시 버그)
- 이미지 리스트: `expo-image`의 캐싱 + `recyclingKey` 활용
- 섹션 리스트: `SectionList` 또는 FlashList 섹션 지원 활용
- 빈 리스트: `ListEmptyComponent`로 빈 상태 메시지 표시

---

## 17. 안티패턴 (하지 말 것)

### React Native 일반

| 안티패턴 | 이유 | 대안 |
|---------|------|------|
| `ScrollView` + `.map()` (긴 리스트) | 모든 아이템 한번에 렌더 -> 메모리 폭발 | `FlatList` / `FlashList` |
| 인라인 `style={{}}` 객체 | 매 렌더 새 객체 -> memo 무력화 | `StyleSheet.create` / NativeWind |
| 인라인 `onPress={() => {}}` | 매 렌더 새 함수 -> memo 무력화 | `useCallback` + 별도 핸들러 |
| `useEffect`로 데이터 fetch | 로딩/에러/캐시 직접 관리 -> 버그 | `@tanstack/react-query` |
| `Image` (RN 기본) | 캐싱 없음, 성능 나쁨 | `expo-image` |
| `TouchableOpacity` | deprecated 추세 | `Pressable` |
| `Dimensions.get()` 직접 사용 | 회전/리사이즈 미대응 | `useWindowDimensions` |
| `AsyncStorage` 대용량/민감 데이터 | 느림, 암호화 안 됨 | MMKV / `expo-secure-store` |
| 화면 컴포넌트에 비즈니스 로직 | 테스트 어렵고 재사용 불가 | 커스텀 훅으로 분리 |
| Context 남용 (앱 전체 상태) | 불필요한 리렌더 전파 | zustand / react-query |
| `Alert.alert()` (웹 포함 앱) | 웹에서 작동 안 함 | 크로스플랫폼 다이얼로그 |
| `contentContainerClassName` | 웹에서 레이아웃 깨짐 | `contentContainerStyle` |
| `window.location` 직접 접근 | 네이티브에서 크래시 | `Platform.OS` 체크 후 접근 |
| zustand에 서버 데이터 복사 | 캐시 불일치, 이중 관리 | react-query에서만 관리 |

### AI가 흔히 생성하는 실수

| 실수 | 수정 방법 |
|------|----------|
| `<View>` 안에 텍스트 직접 넣기 | 반드시 `<Text>` 감싸기 (RN 규칙) |
| 웹 HTML 태그 (`<div>`, `<span>`) | `<View>`, `<Text>` 사용 |
| CSS 속성 (`background-color`) | camelCase (`backgroundColor`) 또는 NativeWind |
| `onClick` 사용 | `onPress` 사용 |
| `className` (NativeWind 미설정) | NativeWind 설정 확인 필요 |
| 웹 전용 API (`window`, `document`) | `Platform.OS === 'web'` 체크 후 접근 |
| `useEffect(async () => {})` | 내부에 별도 async 함수 정의 후 호출 |
| FlatList `key={index}` | 고유 ID 사용 |
| `console.log` 프로덕션에 남기기 | 제거 또는 로거 사용 |
| `npm install` Expo 패키지 | `npx expo install` 사용 (호환 버전) |
| useEffect cleanup 누락 | 구독/리스너는 반드시 return으로 정리 |
| KeyboardAvoidingView 누락 (폼) | 키보드가 입력 필드를 가림 |
| `Alert.alert` 웹에서 사용 | `Platform.OS` 분기 또는 커스텀 모달 |
| `Modal` `presentationStyle` 웹에서 | 웹 미지원, `Platform.select` 사용 |
| setState after unmount | isMounted 패턴 또는 AbortController |

---

## 18. Null/빈 상태 처리

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

// 사용
function ItemListScreen() {
  const query = useItemList();
  const state = useAsyncState(query);

  switch (state.status) {
    case 'loading':
      return <SkeletonList count={5} />;
    case 'error':
      return <ErrorView error={state.error} onRetry={state.refetch} />;
    case 'empty':
      return (
        <EmptyView
          icon="inbox"
          message="No items yet"
          actionLabel="Create first item"
          onAction={() => router.push('/item/create')}
        />
      );
    case 'success':
      return <ItemList items={state.data} />;
  }
}
```

### 공통 빈 상태 컴포넌트

```typescript
interface EmptyViewProps {
  icon?: string;
  message: string;
  actionLabel?: string;
  onAction?: () => void;
}

function EmptyView({ icon = 'inbox', message, actionLabel, onAction }: EmptyViewProps) {
  return (
    <View className="flex-1 items-center justify-center py-16">
      <IconSymbol name={icon} size={48} className="text-gray-300 mb-4" />
      <Text className="text-gray-500 text-base mb-4">{message}</Text>
      {actionLabel && onAction && (
        <Pressable
          onPress={onAction}
          className="bg-primary px-6 py-3 rounded-full"
        >
          <Text className="text-white font-medium">{actionLabel}</Text>
        </Pressable>
      )}
    </View>
  );
}
```

### 옵셔널 값 안전 처리

```typescript
// Good: nullish coalescing + optional chaining
const displayName = user?.profile?.name ?? 'Guest';
const itemCount = items?.length ?? 0;

// Good: early return으로 null 체크
function ItemDetail({ itemId }: { itemId: string }) {
  const { data: item, isLoading } = useItem(itemId);

  if (isLoading) return <LoadingSkeleton />;
  if (!item) return <ErrorView message="Item not found" />;

  // 여기부터 item은 non-null
  return <Text>{item.title}</Text>;
}

// Bad: JSX 내 && 연산자로 0 표시 버그
{count && <Badge text={count} />}  // count=0 이면 "0" 텍스트 렌더!

// Good: 명시적 비교
{count > 0 && <Badge text={count} />}
{count != null && <Badge text={count} />}
```

---

## 19. 비동기 처리

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

```typescript
// 여러 쿼리 병렬 실행
const itemQuery = useQuery({ queryKey: ['item', id], queryFn: () => fetchItem(id) });
const reviewsQuery = useQuery({ queryKey: ['reviews', id], queryFn: () => fetchReviews(id) });

const isLoading = itemQuery.isLoading || reviewsQuery.isLoading;

// Promise.all로 병렬 (훅 외부에서)
async function loadInitialData(userId: string) {
  const [profile, settings, notifications] = await Promise.all([
    fetchProfile(userId),
    fetchSettings(userId),
    fetchNotifications(userId),
  ]);
  return { profile, settings, notifications };
}
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

// 수동 취소
const abortController = useRef(new AbortController());

useEffect(() => {
  return () => abortController.current.abort(); // 컴포넌트 unmount 시 취소
}, []);
```

### 타임아웃 패턴

```typescript
async function fetchWithTimeout<T>(
  fn: (signal: AbortSignal) => Promise<T>,
  timeoutMs: number = 10000,
): Promise<T> {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const result = await fn(controller.signal);
    return result;
  } catch (error) {
    if (error instanceof DOMException && error.name === 'AbortError') {
      throw new AppError('REQUEST_TIMEOUT', 'Request timed out');
    }
    throw error;
  } finally {
    clearTimeout(timeout);
  }
}
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

// 사용
function SearchScreen() {
  const [keyword, setKeyword] = useState('');
  const debouncedKeyword = useDebounce(keyword, 300);

  const { data: results } = useQuery({
    queryKey: ['search', debouncedKeyword],
    queryFn: () => searchItems(debouncedKeyword),
    enabled: debouncedKeyword.length >= 2,
  });

  return (
    <TextInput value={keyword} onChangeText={setKeyword} placeholder="Search..." />
  );
}
```

---

## 20. 에러/예외 처리

> TypeScript 기본 에러 처리(`typescript.md`)를 따르되, React Native 특화:

### 커스텀 에러 클래스

```typescript
// 앱 전역 에러 기본 클래스
class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly userMessage: string, // 사용자에게 표시할 메시지
    message?: string,                     // 개발자용 디버그 메시지
  ) {
    super(message ?? userMessage);
    this.name = 'AppError';
  }
}

// 네트워크 에러
class NetworkError extends AppError {
  constructor(
    public readonly statusCode: number,
    userMessage: string,
    message?: string,
  ) {
    super('NETWORK_ERROR', userMessage, message);
    this.name = 'NetworkError';
  }

  get isUnauthorized(): boolean {
    return this.statusCode === 401;
  }

  get isNotFound(): boolean {
    return this.statusCode === 404;
  }

  get isServerError(): boolean {
    return this.statusCode >= 500;
  }
}

// 검증 에러
class ValidationError extends AppError {
  constructor(
    public readonly field: string,
    userMessage: string,
  ) {
    super('VALIDATION_ERROR', userMessage);
    this.name = 'ValidationError';
  }
}
```

### 에러 분류 및 사용자 메시지 매핑

```typescript
function getErrorMessage(error: unknown): string {
  if (error instanceof NetworkError) {
    if (error.isUnauthorized) return 'Login session has expired. Please log in again.';
    if (error.isNotFound) return 'The requested content does not exist.';
    if (error.isServerError) return 'A server error occurred. Please try again later.';
    return error.userMessage;
  }

  if (error instanceof ValidationError) {
    return error.userMessage;
  }

  if (error instanceof AppError) {
    return error.userMessage;
  }

  // 알 수 없는 에러 -> 일반 메시지 (내부 에러 노출 금지)
  return 'An unexpected error occurred. Please try again.';
}
```

### Error Boundary

```typescript
import React, { Component, type ErrorInfo, type ReactNode } from 'react';

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.props.onError?.(error, errorInfo);
    // Sentry, Crashlytics 등에 보고
    // reportCrash(error, errorInfo);
  }

  private reset = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <ErrorFallback error={this.state.error} onRetry={this.reset} />
      );
    }
    return this.props.children;
  }
}

// 사용: 화면 단위 ErrorBoundary 래핑
// app/(tabs)/_layout.tsx
<ErrorBoundary>
  <Tabs>...</Tabs>
</ErrorBoundary>
```

### API 에러 처리 (react-query + fetch)

```typescript
// fetch 래퍼: HTTP 에러를 NetworkError로 변환
async function apiRequest<T>(url: string, options?: RequestInit): Promise<T> {
  try {
    const response = await fetch(url, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options?.headers,
      },
    });

    if (!response.ok) {
      const body = await response.json().catch(() => null);
      throw new NetworkError(
        response.status,
        body?.message ?? `Request failed (${response.status})`,
      );
    }

    return response.json();
  } catch (error) {
    if (error instanceof NetworkError) throw error;
    if (error instanceof TypeError) {
      throw new AppError('NETWORK_OFFLINE', 'No internet connection.');
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

## 21. 앱 생명주기 (AppState)

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
function useSessionRevalidation() {
  const { session, clearSession } = useAppStore(
    useShallow((s) => ({ session: s.session, clearSession: s.clearSession })),
  );
  const router = useRouter();

  const handleAppActive = useCallback(async () => {
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
  }, [session, clearSession, router]);

  useAppState(handleAppActive);
}

// 루트 레이아웃에서 사용
export default function RootLayout() {
  useSessionRevalidation();
  return <Stack />;
}
```

### 백그라운드 전환 시 리소스 정리

```typescript
function useBackgroundCleanup() {
  useEffect(() => {
    const subscription = AppState.addEventListener('change', (state) => {
      if (state === 'background') {
        // 실시간 구독 일시 중단
        realtimeClient.disconnect();
        // 진행 중인 업로드 일시 중지
        uploadManager.pauseAll();
      }
      if (state === 'active') {
        // 포그라운드 복귀 시 재연결
        realtimeClient.connect();
        uploadManager.resumeAll();
      }
    });

    return () => subscription.remove();
  }, []);
}
```

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

## 22. 보안

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
// 외부에서 들어오는 딥링크 파라미터 반드시 검증
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

    try {
      const params = parseURLParams(url);
      const validated = deepLinkSchema.parse(params);

      if (validated.itemId) {
        router.push(`/item/${validated.itemId}`);
      }
    } catch {
      // 잘못된 딥링크 -> 무시 (에러 로깅만)
      console.warn('Invalid deep link:', url);
    }
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
// 인터셉터: 401 시 토큰 리프레시 후 재시도
let isRefreshing = false;
let refreshPromise: Promise<string> | null = null;

async function authenticatedFetch(url: string, options?: RequestInit): Promise<Response> {
  const token = await getToken();
  const response = await fetch(url, {
    ...options,
    headers: { ...options?.headers, Authorization: `Bearer ${token}` },
  });

  if (response.status === 401) {
    // 리프레시 중복 방지
    if (!isRefreshing) {
      isRefreshing = true;
      refreshPromise = refreshToken();
    }

    try {
      const newToken = await refreshPromise;
      await saveToken(newToken);
      // 새 토큰으로 재시도
      return fetch(url, {
        ...options,
        headers: { ...options?.headers, Authorization: `Bearer ${newToken}` },
      });
    } catch {
      await deleteToken();
      throw new NetworkError(401, 'Session expired');
    } finally {
      isRefreshing = false;
      refreshPromise = null;
    }
  }

  return response;
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

## 23. 성능 최적화

### 렌더링 성능

```typescript
// 1. React.memo: 리스트 아이템, 자주 리렌더되는 부모의 자식
const ItemCard = React.memo(function ItemCard({ item }: ItemCardProps) {
  return <View>...</View>;
});

// 2. 커스텀 비교 함수 (복잡한 객체에서 특정 필드만 비교)
const ItemCard = React.memo(
  function ItemCard({ item }: ItemCardProps) { return <View>...</View>; },
  (prev, next) => prev.item.id === next.item.id && prev.item.updatedAt === next.item.updatedAt,
);

// 3. useMemo: 비용이 큰 필터/정렬
const filteredItems = useMemo(
  () => items.filter(i => i.status === activeFilter).sort(sortByDate),
  [items, activeFilter],
);

// 4. useCallback: memo된 자식에 전달하는 콜백만
const handleItemPress = useCallback((id: string) => {
  router.push(`/item/${id}`);
}, [router]);
```

### 리렌더 추적 (디버깅용)

```typescript
// 개발 모드에서 리렌더 원인 추적
function useRenderTracker(componentName: string, props: Record<string, unknown>) {
  const prevProps = useRef(props);

  useEffect(() => {
    const changes: Record<string, { from: unknown; to: unknown }> = {};

    for (const key of Object.keys(props)) {
      if (prevProps.current[key] !== props[key]) {
        changes[key] = { from: prevProps.current[key], to: props[key] };
      }
    }

    if (Object.keys(changes).length > 0) {
      console.debug(`[${componentName}] re-render caused by:`, changes);
    }

    prevProps.current = props;
  });
}

// 사용 (개발 모드만)
if (__DEV__) {
  useRenderTracker('ItemCard', { item, onPress });
}
```

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

## 24. 이미지 처리

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
  // 1. 권한 확인
  const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
  if (status !== 'granted') {
    showToast('Photo library permission is required');
    return null;
  }

  // 2. 이미지 선택
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    allowsEditing: true,
    aspect: [1, 1],
    quality: 0.8,
  });

  if (result.canceled || !result.assets[0]) return null;

  // 3. 리사이즈 + 압축
  const manipulated = await ImageManipulator.manipulateAsync(
    result.assets[0].uri,
    [{ resize: { width: 800 } }], // 최대 800px
    { compress: 0.7, format: ImageManipulator.SaveFormat.JPEG },
  );

  return manipulated.uri;
}

// 4. 서버 업로드
async function uploadImage(uri: string, path: string): Promise<string> {
  const response = await fetch(uri);
  const blob = await response.blob();

  const { data, error } = await supabase.storage
    .from('images')
    .upload(path, blob, { contentType: 'image/jpeg' });

  if (error) throw new AppError('UPLOAD_FAILED', 'Image upload failed');
  return data.path;
}
```

### 이미지 캐시 전략

```typescript
// expo-image는 디스크 캐시를 자동으로 관리
// 캐시 정책은 Image.prefetch로 제어
import { Image } from 'expo-image';

// 미리 캐싱 (리스트 진입 전)
async function prefetchImages(urls: string[]) {
  await Promise.all(urls.map(url => Image.prefetch(url)));
}

// 캐시 초기화 (필요 시)
async function clearImageCache() {
  await Image.clearDiskCache();
  await Image.clearMemoryCache();
}
```

### 이미지 규칙
- 항상 `expo-image` 사용 (RN 기본 `Image` 사용 금지)
- `contentFit` 지정 필수: `cover`(카드), `contain`(프로필), `fill`(배경)
- `placeholder` 사용: `blurhash` 또는 저해상도 이미지
- `transition` 사용: 200~300ms 페이드인
- 리스트 내 이미지: `recyclingKey` 지정 (이미지 깜빡임 방지)
- 업로드 전 반드시 리사이즈 (원본 4K 전송 금지)
- `aspectRatio` + `width: '100%'`로 비율 유지

---

## 25. 환경변수 관리

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
// src/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  EXPO_PUBLIC_API_URL: z.string().url(),
  EXPO_PUBLIC_SUPABASE_URL: z.string().url(),
  EXPO_PUBLIC_SUPABASE_ANON_KEY: z.string().min(1),
});

function getEnv() {
  const result = envSchema.safeParse({
    EXPO_PUBLIC_API_URL: process.env.EXPO_PUBLIC_API_URL,
    EXPO_PUBLIC_SUPABASE_URL: process.env.EXPO_PUBLIC_SUPABASE_URL,
    EXPO_PUBLIC_SUPABASE_ANON_KEY: process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY,
  });

  if (!result.success) {
    throw new Error(`Invalid environment variables: ${result.error.message}`);
  }

  return result.data;
}

export const env = getEnv();

// 사용
import { env } from '@/config/env';
const client = createClient(env.EXPO_PUBLIC_SUPABASE_URL, env.EXPO_PUBLIC_SUPABASE_ANON_KEY);
```

### 환경변수 규칙
- `EXPO_PUBLIC_` 접두어: 클라이언트에 노출되어도 안전한 것만
- 비밀 키(service role, API secret): `EXPO_PUBLIC_` 접두어 금지 -> Edge Function에서만 사용
- 환경변수 변경 후 Expo dev server 재시작 필수
- `.env.local`은 `.gitignore`에 포함
- `expo-constants`로 빌드 타임 설정 접근

---

## 26. 폰트 & 스플래시

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

## 27. StatusBar

```typescript
import { StatusBar } from 'expo-status-bar';

// 루트 레이아웃에서 전역 설정
export default function RootLayout() {
  return (
    <>
      <StatusBar style="auto" /> {/* light/dark/auto */}
      <Stack />
    </>
  );
}

// 화면별 StatusBar 스타일 변경
export default function DarkScreen() {
  return (
    <>
      <StatusBar style="light" />
      <View className="flex-1 bg-gray-900">
        {/* dark background content */}
      </View>
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

## 28. 푸시 알림 (expo-notifications)

```typescript
import * as Notifications from 'expo-notifications';
import Constants from 'expo-constants';

// 알림 표시 설정 (포그라운드)
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
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

// 알림 리스너 설정 (루트 레이아웃)
function useNotificationListeners() {
  const router = useRouter();

  useEffect(() => {
    // 포그라운드 알림 수신
    const receivedSub = Notifications.addNotificationReceivedListener((notification) => {
      // 인앱 알림 처리 (뱃지 업데이트 등)
    });

    // 알림 탭 -> 화면 이동
    const responseSub = Notifications.addNotificationResponseReceivedListener((response) => {
      const data = response.notification.request.content.data;
      if (data?.itemId && typeof data.itemId === 'string') {
        router.push(`/item/${data.itemId}`);
      }
    });

    return () => {
      receivedSub.remove();
      responseSub.remove();
    };
  }, [router]);
}

---

## 29. 플랫폼 분기

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

## 30. 테스트

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

// react-query 훅 테스트 래퍼
function createQueryWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });

  return function Wrapper({ children }: PropsWithChildren) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );
  };
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
const mockRouter = {
  push: jest.fn(),
  replace: jest.fn(),
  back: jest.fn(),
  dismiss: jest.fn(),
};

export const useRouter = () => mockRouter;
export const useLocalSearchParams = jest.fn().mockReturnValue({});
export const Link = ({ children }: PropsWithChildren) => children;
export const Redirect = () => null;

// 테스트에서 사용
import { useRouter } from 'expo-router';

it('navigates to detail on press', () => {
  render(<ItemCard item={mockItem} />);
  fireEvent.press(screen.getByText('Test Item'));
  expect(useRouter().push).toHaveBeenCalledWith('/item/1');
});
```

### 비동기 상태 테스트

```typescript
describe('CreateItemScreen', () => {
  it('shows loading state during submission', async () => {
    // 느린 API mock
    server.use(
      rest.post('/api/items', async (req, res, ctx) => {
        await delay(1000);
        return res(ctx.json({ id: '1' }));
      }),
    );

    render(<CreateItemScreen />, { wrapper: createQueryWrapper() });

    // 폼 입력
    fireEvent.changeText(screen.getByPlaceholderText('Title'), 'New Item');
    fireEvent.press(screen.getByText('Create'));

    // 로딩 상태 확인
    expect(screen.getByText('Creating...')).toBeTruthy();

    // 완료 대기
    await waitFor(() => {
      expect(screen.queryByText('Creating...')).toBeNull();
    });
  });
});
```

### 테스트 규칙
- 테스트 파일: `__tests__/` 폴더 또는 소스 파일 옆 `.test.tsx`
- 컴포넌트: 렌더링 + 사용자 인터랙션 + 결과 확인
- 훅: `renderHook` + react-query 래퍼
- API: `msw`로 네트워크 mock (fetch/axios mock 대신)
- 스냅샷 테스트: 디자인 변경 감지용 (과용 금지 — 깨지기 쉬움)
- E2E: 핵심 유저 플로우만 (로그인 -> 목록 -> 상세 -> 액션)

---

## 31. 디버깅

### React DevTools

```bash
# React DevTools 설치 및 실행
npx react-devtools

# Expo 개발 서버와 자동 연결
# 컴포넌트 트리, props, state, hooks 확인
# Profiler 탭에서 리렌더 분석
```

### 네트워크 디버깅

```typescript
// 개발 모드에서 네트워크 요청 로깅
if (__DEV__) {
  // React Native Debugger 또는 Flipper 사용
  // Expo: npx expo start 후 'j' 키로 Chrome DevTools 열기

  // 수동 로깅 (필요 시)
  const originalFetch = global.fetch;
  global.fetch = async (...args) => {
    const [url, options] = args;
    console.debug(`[fetch] ${options?.method ?? 'GET'} ${url}`);
    const response = await originalFetch(...args);
    console.debug(`[fetch] ${response.status} ${url}`);
    return response;
  };
}
```

### 성능 프로파일링

```typescript
// 1. React Profiler (위 23절 참조)
// 2. Hermes 프로파일러
//    - Expo Dev Menu -> "Open React Profiler"
//    - CPU 프로파일: 함수별 실행 시간 분석

// 3. 리렌더 하이라이트
//    - React DevTools -> Settings -> "Highlight updates"
//    - 불필요한 리렌더를 시각적으로 확인
```

### 일반적인 디버깅 전략
- `console.log` 대신 `console.debug` (프로덕션 빌드에서 자동 제거 가능)
- 상태 디버깅: zustand devtools 미들웨어
- 네트워크: react-query devtools (개발 모드)
- 크래시: `expo-updates` 에러 리포팅 또는 Sentry
- 레이아웃: `borderWidth: 1, borderColor: 'red'`로 경계 확인 (개발 중)

```typescript
// zustand devtools (개발 모드)
import { devtools } from 'zustand/middleware';

const useStore = create<AppStore>()(
  devtools(
    persist(
      (set) => ({ /* ... */ }),
      { name: 'app-store' },
    ),
    { name: 'AppStore', enabled: __DEV__ },
  ),
);
```

---

## 32. 문서화 규칙

> TypeScript 기본 문서화(`typescript.md`)를 따르되, React Native 특화 추가.

### 화면(Screen) 컴포넌트

```typescript
// Good: 화면 역할을 한 줄로 설명
/** Monthly calendar view with attendance management */
export default function ScheduleScreen() { ... }

// Bad: 없거나 너무 장황
export default function ScheduleScreen() { ... }
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

## 33. 빌드 & 설정

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
  splash: {
    image: './assets/splash.png',
    resizeMode: 'contain',
    backgroundColor: '#ffffff',
  },
  updates: {
    fallbackToCacheTimeout: 0,
  },
  plugins: [
    'expo-router',
    'expo-notifications',
    'expo-secure-store',
    ['expo-image-picker', { photosPermission: 'Allow access to select photos' }],
  ],
  ios: {
    supportsTablet: true,
    bundleIdentifier: 'com.example.myapp',
  },
  android: {
    adaptiveIcon: {
      foregroundImage: './assets/adaptive-icon.png',
      backgroundColor: '#ffffff',
    },
    package: 'com.example.myapp',
  },
  web: {
    bundler: 'metro',
    favicon: './assets/favicon.png',
  },
  experiments: {
    typedRoutes: true, // 타입 안전 라우트
  },
});
```

### EAS Build 설정 (eas.json)

```json
{
  "cli": {
    "version": ">= 7.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview"
    },
    "production": {
      "channel": "production",
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "user@example.com",
        "ascAppId": "1234567890"
      }
    }
  }
}
```

### tsconfig.json 권장 설정

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
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
      'react-native-reanimated/plugin', // 반드시 마지막
    ],
  };
};
```

> **주의**: `react-native-reanimated/plugin`은 반드시 plugins 배열의 마지막에 위치해야 한다.

---

## 34. 자주 쓰는 라이브러리/도구

### 권장

| 용도 | 라이브러리 | 이유 |
|------|-----------|------|
| 라우팅 | **expo-router** | 파일 기반, 딥링크 자동, 웹 호환 |
| 서버 상태 | **@tanstack/react-query** | 캐싱, 재시도, 무효화, 낙관적 업데이트 |
| 클라이언트 상태 | **zustand** | 간결, 보일러플레이트 최소, 셀렉터 |
| 스타일링 | **NativeWind v4** | Tailwind 문법, 웹/네이티브 통일 |
| 이미지 | **expo-image** | 캐싱, 블러 해시, 트랜지션 |
| 리스트 | **@shopify/flash-list** | FlatList보다 5배 빠름 |
| 알림 | **expo-notifications** | FCM/APNs 통합 |
| 보안 저장 | **expo-secure-store** | iOS Keychain / Android Keystore |
| 폼 | **react-hook-form + zod** | 성능 최적, 타입 안전 검증 |
| 날짜 | **date-fns** | 경량, tree-shaking |
| 토스트 | **burnt** 또는 **react-native-toast-message** | 네이티브 느낌 |
| 아이콘 | **@expo/vector-icons** | Expo 기본 포함 |
| 애니메이션 | **react-native-reanimated** | 60fps UI 스레드 애니메이션 |
| 제스처 | **react-native-gesture-handler** | 네이티브 제스처 인식 |
| 바텀시트 | **@gorhom/bottom-sheet** | reanimated 기반, 고성능 |
| 키 저장 (비암호화) | **react-native-mmkv** | AsyncStorage보다 30배 빠름 |

### 피해야 할 것

| 라이브러리 | 이유 | 대안 |
|-----------|------|------|
| `Image` (RN 기본) | 캐싱 없음, 성능 나쁨 | `expo-image` |
| `TouchableOpacity` | deprecated 추세 | `Pressable` |
| `AsyncStorage` (토큰) | 암호화 안 됨 | `expo-secure-store` |
| `react-navigation` (직접) | Expo Router가 래핑 | `expo-router` |
| `moment.js` | deprecated, 거대 번들 | `date-fns`, `dayjs` |
| `redux` (소규모) | 과도한 보일러플레이트 | `zustand` |
| `axios` | 대부분 과도함 | `fetch` + react-query |
| `react-native-fast-image` | expo-image가 더 나음, Expo 호환 | `expo-image` |
| `@react-native-community/async-storage` (대용량) | 느림 | `react-native-mmkv` |

---

## 35. 프로젝트별 확장 포인트

> 아래 항목들은 프로젝트마다 다르므로 `/CLAUDE.md`에서 정의합니다:

- 앱 이름, 번들 ID, 딥링크 스킴
- 프로젝트 구조 상세 (features 폴더 목록)
- 빌드/실행 명령 (`npx expo start`, `eas build`)
- EAS 프로젝트 ID
- 환경 변수 목록 및 값
- 배포 전략 (EAS Update, 스토어 등)
- 외부 서비스 연동 (Supabase, Firebase, Sentry 등)
- 디자인 시스템 (색상, 폰트, 간격 토큰)
- 앱 아이콘, 스플래시 스크린 설정
- 소셜 로그인 프로바이더 설정
- CI/CD 파이프라인 설정
- API 엔드포인트 및 인증 방식
- maxWidth 상수 값 (MAX_FORM_WIDTH, MAX_CONTENT_WIDTH 등)

---

## 36. 실전 함정 & 지뢰

> 실제 프로젝트에서 발견된 함정. 안티패턴과 다른 점: "정상적으로 보이지만 특정 조건에서 깨지는 것".

| 증상 | 원인 | 해결 | 오답 (하지 말 것) |
|------|------|------|-------------------|
| 웹 새로고침(F5) 시 로그인 풀림 | Supabase Auth에 `storage: AsyncStorage` 설정 — 웹에서 AsyncStorage 미작동 | `Platform.OS === 'web' ? undefined : AsyncStorage`로 분기 | AsyncStorage를 웹 polyfill로 대체 (불필요한 복잡성) |
| 탭 외 화면(상세 등)에서 탭바 미표시 | 탭바가 `(tabs)/_layout.tsx`에만 존재, Stack 네비게이션 밖 화면에는 적용 안 됨 | 커스텀 탭바 컴포넌트를 루트 레이아웃에 배치 또는 탭 내 중첩 네비게이션 | 모든 화면을 탭 안에 억지로 넣기 |
| 모바일에서 탭바 하단 잘림 | SafeAreaView 처리 미흡 — 하단 안전 영역 미반영 | `useSafeAreaInsets()` + `paddingBottom: insets.bottom` | 고정 값 하드코딩 (기기마다 다름) |
| `contentContainerClassName`으로 스타일 적용 시 웹에서 깨짐 | NativeWind의 contentContainerClassName은 웹에서 미지원 | `contentContainerStyle` 사용 | className으로 변경 시도 |
| 이미지 피커 웹에서 크래시 | `expo-image-picker`의 `launchCameraAsync`는 웹 미지원 | `Platform.OS === 'web'`이면 `launchImageLibraryAsync`만 사용 | 카메라 기능을 웹에서도 시도 |

---

## 37. 검증 체크리스트

### 새 화면 추가 시

```
- [ ] SafeAreaView 최상위 래핑 (flex: 1, backgroundColor)
- [ ] maxWidth 제한 적용 (용도에 맞는 상수)
- [ ] width: '100%' + alignSelf: 'center' 세트
- [ ] 4가지 상태 처리 (로딩/에러/빈 데이터/정상)
- [ ] 웹에서 확인 (npx expo start --web)
- [ ] 모바일에서 확인 (SafeArea 잘림, 키보드 가림)
- [ ] 인증 필요 여부 확인 (가드 처리)
```

### 배포 전 크로스 플랫폼 확인

```
- [ ] 웹: 로그인 → 새로고침 → 세션 유지 확인
- [ ] 웹: 데스크톱 너비에서 레이아웃 정상
- [ ] 모바일: 탭바 하단 잘림 없음
- [ ] 모바일: 키보드 올라올 때 입력창 가림 없음
- [ ] 모든 플랫폼: 핵심 CRUD 동작 확인
```

<!-- 개선 이력
- 2026-03-27: 실전 함정 섹션 추가 — AsyncStorage 웹 세션 유실, 탭바 미표시/잘림 발견
- 2026-03-27: 검증 체크리스트 추가 — 새 화면/배포 전 크로스 플랫폼 확인
-->
