---
paths:
  - "**/*.cs"
  - "**/*.csproj"
  - "**/*.sln"
---

# C# 개발 규칙

> 이 파일은 C# 프로젝트에 범용적으로 적용되는 규칙입니다.
> 프로젝트별 기술 스택, 빌드 명령, 폴더 구조는 `/CLAUDE.md`에 정의합니다.

## 핵심 규칙

- 상수(URL, 경로, 키 등)는 전용 상수 클래스 또는 설정 파일에 정의 (매직 넘버/문자열 금지)
- Enum은 전용 파일에 정의, 프로젝트 네이밍 컨벤션 준수
- 예외 처리: try-catch + 프로젝트에 정의된 로깅 패턴 준수
- UI 프레임워크(WinForms, WPF, MAUI 등) 사용 시 UI 스레드와 백그라운드 작업 분리

## 보안

- API 키, 토큰, 비밀번호를 소스 코드에 하드코딩하지 않음
- 시크릿은 환경 변수, User Secrets, 외부 설정 파일로 분리
- public 저장소 공개 시 민감 정보 반드시 제거
- 외부 입력값 검증 (사용자 입력, API 응답)
- 파일 경로 조합 시 `Path.Combine` 사용

## 빌드

- 빌드 명령은 `/CLAUDE.md`에 정의된 것을 사용
- 빌드 결과: `Build succeeded` + 0 Error 필수
- Warning은 프로젝트 기준에 따라 허용 범위 판단

## 네이밍 컨벤션

- PascalCase: 클래스, 메서드, 프로퍼티, public 필드, 상수
- camelCase: 지역 변수, 파라미터
- _camelCase: private 필드
- I 접두어: 인터페이스 (`IDisposable`, `IComparable`)
- Is/Has/Can/Should 접두어: bool 변수/프로퍼티
- 프로젝트별 접두어/접미어는 `/CLAUDE.md`에 정의

## 구조 & 설계

- 한 파일에 한 클래스 원칙
- 접근 제한자 명시적 작성 + 최소 권한 원칙
- 클래스 멤버 순서: 필드 > 프로퍼티 > 생성자 > public 메서드 > private 메서드 > 이벤트 핸들러
- 메서드 길이 30줄 이내 권장, 초과 시 분리
- 파라미터 3개 이하 (초과 시 객체로 묶기)
- early return 패턴 사용 (중첩 if 최소화)
- SOLID 원칙 준수

## 예외 처리

- catch 블록에서 예외를 삼키지 않음 (최소한 로깅)
- 구체적 예외 타입 우선 (너무 넓은 `Exception` 캐치 지양)
- `throw;` 사용 (스택트레이스 보존, `throw ex;` 지양)
- 예외를 흐름 제어용으로 사용하지 않음

## Null 안전성

- null 조건 연산자 적절히 사용 (`?.`, `??`)
- 메서드 파라미터 null 체크 (public API 경계)
- 빈 컬렉션 반환: null 대신 `Array.Empty<T>()`, `Enumerable.Empty<T>()`
- `string.IsNullOrEmpty()` / `string.IsNullOrWhiteSpace()` 활용

## 리소스 관리

- IDisposable 리소스는 반드시 `using` 문으로 감싸기
- Stream, DbConnection, HttpClient 등 해제 누락 주의
- 이벤트 핸들러 등록 후 해제 누락 없음 (메모리 릭 방지)
- Dispose 패턴이 필요한 클래스는 IDisposable 구현

## 스레딩 & 동시성

- UI 스레드에서 장시간 작업 금지 (async/await, Task, BackgroundWorker 등 활용)
- 공유 자원 접근 시 lock 또는 thread-safe 컬렉션 사용
- async 메서드 네이밍: `~Async` 접미어

## 성능

- 문자열 반복 조합 시 `StringBuilder` 사용
- 불필요한 객체 생성 주의 (루프 내 new)
- 불필요한 리플렉션 사용 지양
- LINQ에서 불필요한 `ToList()`/`ToArray()` 호출 주의

## 컬렉션 & LINQ

- 적절한 컬렉션 타입 사용 (List vs Dictionary vs HashSet)
- 반환 타입: 외부 노출 시 IEnumerable, IReadOnlyList 등 인터페이스
- foreach에서 컬렉션 수정하지 않기

## 패키지 관리

- 패키지 관리 방식은 `/CLAUDE.md`에 정의 (PackageReference / packages.config)
- 새 패키지 추가 시 대상 프레임워크 호환 확인
- 불필요한 패키지 참조 정리

## 프로젝트별 확장 포인트

> 아래 항목들은 프로젝트마다 다르므로 `/CLAUDE.md`에서 정의합니다:
> - 기술 스택 (프레임워크, 버전, 주요 라이브러리)
> - 폴더 구조 및 파일 배치 규칙
> - 빌드 명령 및 CI/CD 설정
> - 브랜치 전략
> - 외부 연동 패턴 (래퍼 클래스 등)
> - 직렬화/데이터 저장 전략
> - UI 프레임워크별 규칙 (테마, 디자이너 파일 등)
> - 로깅 유틸리티 및 패턴
> - 테스트 전략
