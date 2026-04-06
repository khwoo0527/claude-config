---
paths:
  - "**/*.cs"
  - "**/*.csproj"
  - "**/*.sln"
---

# C# 개발 규칙

> 이 파일은 C# 프로젝트에 범용적으로 적용되는 시니어 수준 규칙입니다.
> 프로젝트별 기술 스택, 빌드 명령, 폴더 구조는 `/CLAUDE.md`에 정의합니다.

## 핵심 원칙

1. **타입 안전성 최우선** — `object`, `dynamic` 남용 금지. 컴파일 타임에 잡을 수 있는 오류는 런타임까지 미루지 않는다. `var`는 우변에서 타입이 명확할 때만 사용한다.
2. **Nullable 활성화 필수** — `<Nullable>enable</Nullable>`로 프로젝트 전체 적용. null 가능성은 `?`로 명시. `#pragma warning disable` 로 경고를 숨기면 NullReferenceException이 프로덕션에서 터진다.
3. **최소 권한 접근** — 모든 멤버는 가능한 가장 제한적인 접근 제한자를 사용한다. 접근 제한자를 생략하면 C# 기본값(`internal`/`private`)이 적용되지만 **의도가 불명확**하므로 항상 명시한다.
4. **불변성 선호** — 변경할 필요 없는 데이터는 `readonly`, `init`, `record`로 불변 보장. 가변 상태가 많을수록 멀티스레드에서 버그가 기하급수적으로 늘어난다.
5. **예외는 무시하지 않는다** — `catch { }` 빈 블록은 시한폭탄. 에러가 조용히 묻히면 데이터 손실 원인 추적이 불가능하다. 최소한 로깅하고 상위로 전파한다.

---

## 프로젝트 구조

```
ProjectRoot/
├── Models/              ← 데이터 모델, DTO, Enum (외부 의존성 없는 순수 POCO)
├── Analysis/            ← 비즈니스 로직, 알고리즘 (UI 프레임워크 의존 금지)
├── Services/            ← 외부 연동, 데이터 접근
├── Rendering/           ← 시각화, 그래프 렌더링
├── UI/                  ← UI 관련 (WinForms: Form, Control / WPF: View, ViewModel)
├── Helpers/             ← 유틸리티, 확장 메서드
└── Tests/               ← 테스트 프로젝트 (별도 프로젝트)
```

### 구조 규칙
- **레이어 분리 엄수**: UI → Analysis → Model 단방향 의존. Analysis 레이어에 `System.Windows.Forms` using이 있으면 설계 위반이다.
- **파일 1개 = 타입 1개**: 클래스, 인터페이스, enum 각각 별도 파일
- **파일명 = 타입명**: `TypeNode.cs` → `class TypeNode`
- **네임스페이스 = 폴더 경로**: `ProjectName.Models`, `ProjectName.Analysis`
- **파일 범위 네임스페이스** 사용 (중괄호 없이 한 줄):
```csharp
namespace MyApp.Models;   // ✅ 파일 범위

namespace MyApp.Models { } // ❌ 블록 범위 — 불필요한 들여쓰기 추가
```

---

## 네이밍 컨벤션

| 대상 | 규칙 | 예시 |
|------|------|------|
| 클래스 / 구조체 / 레코드 | PascalCase | `TypeNode`, `AnalysisResult` |
| 인터페이스 | `I` + PascalCase | `IFolderScanner`, `IAnalyzer` |
| Enum 타입 / 값 | PascalCase | `EdgeType`, `Inheritance` |
| 메서드 / 프로퍼티 / 이벤트 | PascalCase | `GetCsFiles()`, `FullName`, `OnCompleted` |
| private / protected 필드 | `_camelCase` | `_lastFolderPath`, `_nodes` |
| 로컬 변수 / 파라미터 | camelCase | `folderPath`, `nodeList`, `sourceFile` |
| 상수 (`const` / `static readonly`) | PascalCase | `MaxNodeCount`, `DefaultLayout` |
| 제네릭 타입 파라미터 | `T` 또는 `T` + 설명 | `T`, `TResult`, `TNode` |
| bool 변수 / 프로퍼티 | `is`/`has`/`can`/`should` 접두사 | `isVisible`, `hasErrors`, `canRefresh` |
| 비동기 메서드 | `Async` 접미사 | `AnalyzeAsync()`, `LoadFilesAsync()` |
| 이벤트 핸들러 | `대상_이벤트명` | `btnOpen_Click`, `clbNamespaces_ItemCheck` |

> **이름은 축약하지 않는다.** `mgr` → `manager`, `btn` → `button` (단, 관용적 약어 `Id`, `Url`, `Html` 등은 허용)

---

## 코드 포맷팅

- **들여쓰기**: 4 spaces (탭 사용 금지)
- **중괄호**: Allman 스타일 (새 줄에 배치)
- **줄 길이**: 120자 이내 권장
- **빈 줄**: 논리 블록 사이에 1줄, 2줄 이상 금지
- **단일 표현식**: 람다 바디 `=>` 허용

```csharp
// ✅ Allman 스타일
if (condition)
{
    DoSomething();
}

// ❌ K&R 스타일
if (condition) { DoSomething(); }

// ✅ 단일 표현식 프로퍼티/메서드
public string FullName => $"{Namespace}.{Name}";
```

---

## import/export 규칙

```csharp
// 1. System 네임스페이스
using System.Collections.Concurrent;
using System.Reflection;

// 2. 서드파티 라이브러리
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;

// 3. 프로젝트 내부
using MyApp.Models;
using MyApp.Analysis;
```

- **ImplicitUsings**: `enable`이면 `System`, `System.Collections.Generic`, `System.Linq` 등 생략
- **using 별칭**: 네임스페이스 충돌 시에만 사용
```csharp
// ✅ System.Drawing.Color와 Microsoft.Msagl.Drawing.Color 충돌 시
using MsaglColor = Microsoft.Msagl.Drawing.Color;
```

---

## 파일 내부 코드 순서

```csharp
namespace MyApp.Services;

public class AnalysisService
{
    // 1. const / static readonly 필드
    private const int MaxRetries = 3;
    private static readonly TimeSpan Timeout = TimeSpan.FromSeconds(30);

    // 2. private 인스턴스 필드
    private readonly IAnalyzer _analyzer;
    private string? _lastPath;

    // 3. 생성자
    public AnalysisService(IAnalyzer analyzer)
    {
        _analyzer = analyzer;
    }

    // 4. public 프로퍼티
    public bool IsAnalyzing { get; private set; }

    // 5. public 메서드
    public AnalysisResult Analyze(string path) { ... }

    // 6. private / protected 메서드
    private void ValidatePath(string path) { ... }

    // 7. 이벤트 핸들러 (UI 클래스에서)
    private void btnOpen_Click(object? sender, EventArgs e) { ... }
}
```

---

## 코드 구조 & 패턴

### 기본 패턴

#### var 사용 기준
```csharp
var nodes = new List<TypeNode>();           // ✅ 우변에서 타입 명확
var graph = new Microsoft.Msagl.Drawing.Graph();  // ✅ 명확
AnalysisResult result = GetResult();        // ✅ 우변 타입 불명확 — 명시
```

#### 컬렉션 초기화
```csharp
public List<TypeNode> Nodes { get; set; } = new();      // ✅ target-typed new
public List<string> Errors { get; set; } = new List<string>(); // ❌ 장황
```

#### Early Return — 중첩 최소화
```csharp
// ✅ early return
public void Process(string? path)
{
    if (string.IsNullOrEmpty(path)) return;
    if (!Directory.Exists(path)) return;

    DoWork(path);
}

// ❌ 중첩 지옥 — 4단계 이상 들여쓰기가 생기면 설계 재고
public void Process(string? path)
{
    if (!string.IsNullOrEmpty(path))
    {
        if (Directory.Exists(path))
        {
            DoWork(path);
        }
    }
}
```

#### LINQ — 메서드 체이닝 스타일
```csharp
// ✅ 한 줄이 짧으면 한 줄
var classNodes = nodes.Where(n => n.Kind == TypeKind.Class).ToList();

// ✅ 길면 각 메서드 줄 바꿈
var result = nodes
    .Where(n => n.Kind == TypeKind.Class)
    .OrderBy(n => n.Name)
    .Select(n => n.FullName)
    .ToList();

// ✅ GroupBy + Select — partial class 병합 등
var merged = result.Nodes
    .GroupBy(n => n.FullName)
    .Select(g => new TypeNode
    {
        Name        = g.First().Name,
        FieldCount  = g.Sum(n => n.FieldCount),
        MethodCount = g.Sum(n => n.MethodCount),
        FieldNames  = g.SelectMany(n => n.FieldNames).ToList()
    })
    .ToList();

// ❌ 쿼리 구문 — 가독성 떨어지고 C# 커뮤니티에서 비선호
var result = (from n in nodes where n.Kind == TypeKind.Class select n).ToList();
```

### 고급 패턴

#### Pattern Matching (C# 9+)
```csharp
// ✅ switch expression — 타입별 분기 (exhaustive 체크)
public static string GetLabel(TypeKind kind) => kind switch
{
    TypeKind.Class     => "클래스",
    TypeKind.Interface => "인터페이스",
    TypeKind.Struct    => "구조체",
    TypeKind.Record    => "레코드",
    TypeKind.Enum      => "열거형",
    _ => throw new ArgumentOutOfRangeException(nameof(kind))
};

// ✅ 재귀적 타입 추출 — switch expression + 재귀 호출
private static string? ExtractTypeName(TypeSyntax typeSyntax) => typeSyntax switch
{
    PredefinedTypeSyntax         => null,              // int, string 스킵
    GenericNameSyntax generic    => generic.TypeArgumentList
        .Arguments.Select(ExtractTypeName)
        .FirstOrDefault(n => n != null),               // List<Service> → "Service"
    SimpleNameSyntax simple      => simple.Identifier.Text,
    NullableTypeSyntax nullable  => ExtractTypeName(nullable.ElementType),
    ArrayTypeSyntax array        => ExtractTypeName(array.ElementType),
    _ => null
};

// ✅ 프로퍼티 패턴 — 조건부 분기
if (node is { Kind: TypeKind.Class, FieldCount: > 10 })
{
    // God Class 후보 — 필드가 10개 초과
}

// ✅ 타입 패턴 + 캐스팅 한 번에
if (sender is IViewerNode viewerNode)
{
    var nodeId = viewerNode.Node.Id;
}
```

#### Record 타입 — 불변 DTO에 사용
```csharp
// ✅ 불변 데이터에 record — 값 비교, ToString, Deconstruct 자동 생성
public record AnalysisMetrics(int ClassCount, int InterfaceCount, int EdgeCount);

// ✅ with 표현식으로 복사 + 수정
var updated = metrics with { ClassCount = 15 };

// ❌ 변경 가능한 상태가 많은 객체에 record 사용 — class가 적절
```

#### 인터페이스 기반 레이어 분리
```csharp
// ✅ 인터페이스를 통해 의존성 역전 — UI가 구현체를 직접 참조하지 않음
public interface IAnalyzer
{
    AnalysisResult Analyze(IReadOnlyList<string> filePaths);
}

// UI 레이어 — 인터페이스에만 의존
IFolderScanner scanner = new FolderScanner();  // 구현체 교체 가능
IAnalyzer analyzer = new RoslynAnalyzer();
var files = scanner.GetCsFiles(folderPath);
var result = analyzer.Analyze(files);
```

#### 접근 제한자 순서
```csharp
// ✅ 접근제한자 → static → readonly → 타입 → 이름
private static readonly int MaxCount = 100;
public static string DefaultName { get; } = "Unnamed";
```

---

## 안티패턴 (하지 말 것)

### 일반적 안티패턴

| 안티패턴 | 실패 결과 | 대안 |
|---------|----------|------|
| `catch (Exception) { }` 빈 catch | 파일 파싱 실패가 조용히 묻힘 → 분석 결과에 데이터 누락되지만 원인 추적 불가 | `result.Errors.Add($"{file}: {ex.Message}")` 기록 |
| `throw ex;` 재throw | 스택 트레이스가 이 줄에서 시작 → 원래 발생 위치(3단계 하위 메서드) 추적 불가 | `throw;` — 원래 스택 보존 |
| public 필드 직접 노출 | setter에서 유효성 검증 우회, 값 변경 추적 불가 | 프로퍼티 `{ get; set; }` 사용 |
| `async void` 메서드 | 예외가 SynchronizationContext로 직접 전달 → 앱 크래시, 디버거에서 안 잡힘 | `async Task` 반환 (이벤트 핸들러만 예외) |
| 문자열 `+` 루프 연결 | 100개 항목 연결 시 100개의 중간 string 객체 생성 → GC 압박, O(N²) 메모리 | `StringBuilder` |
| `Thread.Sleep()` in UI | WinForms: "응답 없음" 표시 + 화면 하얗게 됨 | `await Task.Delay()` |
| `using` 없이 IDisposable | FileStream/DbConnection 누수 → 장시간 실행 시 핸들 고갈 | `using var` 선언 |
| `List<T>` 멀티스레드 공유 | 동시 Add 시 데이터 손실, IndexOutOfRangeException | `ConcurrentBag<T>` 또는 `lock` |

### AI가 흔히 생성하는 실수

| 안티패턴 | 실패 결과 | 대안 |
|---------|----------|------|
| 모든 메서드에 `try-catch(Exception)` | 에러 전파 차단 → 상위 레이어가 문제를 인지 못함 → 빈 화면 | 경계(UI 이벤트 핸들러, API 엔드포인트)에서만 catch |
| `?.`와 `!` 남용 | `!`는 "null이 아님을 확신" 선언 — 확신이 틀리면 NullReferenceException | 적절한 null 검증 + early return |
| 매 메서드마다 파라미터 null 체크 | 내부 메서드에서 동일한 검증 코드 10번 반복 | public API 경계에서만 검증 |
| `Task.Run(() => asyncMethod())` | async 메서드를 Thread Pool에서 다시 래핑 → 불필요한 스레드 전환 오버헤드 | 직접 `await asyncMethod()` |
| `ToList()` 중간마다 호출 | 3단계 LINQ 각각 ToList → 3배 메모리 할당 | 최종 소비 지점에서만 `ToList()` |
| 접근 제한자 생략 | C# 기본값이 적용되지만 의도 불명확 → 리뷰어가 "일부러인지 실수인지" 판단 불가 | 항상 명시 |
| `GViewer` 등 UI 컨트롤을 백그라운드 스레드에서 생성 | `InvalidOperationException` — 크로스 스레드 UI 접근 | UI 컨트롤은 반드시 UI 스레드에서 생성 |
| 이벤트 핸들러 해제 안 함 | 메모리 릭 — GC가 이벤트 소스를 통해 리스너를 살려둠 | `-=`로 해제, 또는 WeakEventManager |

---

## Null/빈값 처리

```csharp
// ✅ null 병합 + 옵셔널 체이닝
var length = text?.Length ?? 0;
var name = node?.Namespace?.ToUpper() ?? "Unknown";

// ✅ 패턴 매칭으로 null 체크 (is not null 권장)
if (result is not null)
{
    ProcessResult(result);
}

// ✅ string 검증 — IsNullOrEmpty vs IsNullOrWhiteSpace 구분
if (string.IsNullOrWhiteSpace(path))  // " " 도 걸러냄
{
    SetStatus("경로를 입력해 주세요.");
    return;
}

// ✅ 빈 컬렉션 반환 — null 반환 금지 (호출자 null 체크 부담)
public IReadOnlyList<string> GetErrors()
    => _errors.Count > 0 ? _errors : Array.Empty<string>();

// ✅ null-forgiving (!) 사용 — 확신이 있을 때만
var selectedNs = Enumerable.Range(0, list.Items.Count)
    .Where(i => list.GetItemChecked(i))
    .Select(i => list.Items[i]!.ToString()!)  // Range 범위 내이므로 null 불가능
    .ToHashSet();

// ❌ ! 남용 — 확신 없이 사용하면 런타임 크래시
var name = GetUser()!.Name!;  // GetUser()가 null이면 NullReferenceException
```

---

## 비동기 처리

### CPU 바운드 vs I/O 바운드

```csharp
// ✅ CPU 바운드 (분석, 계산) → Task.Run + await
public async Task<AnalysisResult> AnalyzeAsync(string folderPath)
{
    SetStatus("분석 중...");
    var result = await Task.Run(() =>
    {
        var files = _scanner.GetCsFiles(folderPath);
        return _analyzer.Analyze(files);
    });
    // await 복귀 후 UI 스레드 — UI 업데이트 안전
    SetStatus($"분석 완료 — {result.Nodes.Count}개 타입");
    return result;
}

// ✅ I/O 바운드 → 네이티브 async API (스레드 소비 없음)
var code = await File.ReadAllTextAsync(filePath);

// ✅ 병렬 비동기 — 독립적인 작업을 동시에
var task1 = LoadConfigAsync();
var task2 = LoadDataAsync();
await Task.WhenAll(task1, task2);
```

### CancellationToken — 사용자가 빠르게 조작할 때

```csharp
// ✅ 이전 작업 취소 후 새 작업 시작 (필터 변경, 검색 입력 등)
_rebuildCts?.Cancel();
_rebuildCts?.Dispose();
_rebuildCts = new CancellationTokenSource();
var ct = _rebuildCts.Token;

try
{
    var graph = await Task.Run(() => renderer.BuildGraph(result), ct);
    if (ct.IsCancellationRequested) return;  // 이중 체크
    UpdateUI(graph);
}
catch (OperationCanceledException)
{
    return;  // 더 최신 요청이 있으므로 결과 버림
}
```

### IProgress<T> — 진행률 보고

```csharp
// ✅ 파일별 진행률 UI 업데이트
var progress = new Progress<(int current, int total)>(p =>
{
    SetStatus($"분석 중... {p.current}/{p.total}");
});

await Task.Run(() => analyzer.Analyze(files, progress));
```

### 규칙 요약

| 상황 | 패턴 | 이유 |
|------|------|------|
| `async void` | 이벤트 핸들러에서만 허용 | 예외가 호출자에게 전파되지 않아 앱 크래시 |
| `.Result` / `.Wait()` | 사용 금지 | UI 스레드에서 호출 시 데드락 |
| `ConfigureAwait(false)` | 라이브러리 코드에서만 | UI 코드에서는 UI 컨텍스트가 필요하므로 기본값 유지 |
| `Task.Run(async () => ...)` | 지양 | async 메서드를 다시 래핑 → 불필요한 전환 |

---

## 에러/예외 처리

### 파일별 부분 실패 허용 패턴

```csharp
// ✅ 파일 하나가 실패해도 나머지는 계속 분석
public AnalysisResult Analyze(IReadOnlyList<string> filePaths)
{
    var result = new AnalysisResult();
    foreach (var filePath in filePaths)
    {
        try
        {
            var code = File.ReadAllText(filePath);
            var tree = CSharpSyntaxTree.ParseText(code);
            // ... 분석 로직
        }
        catch (IOException ex)
        {
            result.Errors.Add($"{Path.GetFileName(filePath)}: 파일 읽기 실패 — {ex.Message}");
        }
        catch (Exception ex)
        {
            result.Errors.Add($"{Path.GetFileName(filePath)}: {ex.Message}");
        }
    }
    return result;  // Errors 리스트로 실패 파일 추적 가능
}
```

### throw; vs throw ex;

```csharp
// ✅ throw; — 원래 스택 트레이스 보존
catch (Exception ex)
{
    Logger.Error(ex);
    throw;  // 스택 트레이스: 원래 발생 지점 → 여기 → 호출자
}

// ❌ throw ex; — 스택 트레이스가 이 줄에서 시작
catch (Exception ex)
{
    Logger.Error(ex);
    throw ex;  // 스택 트레이스: 여기 → 호출자 (원래 지점 소실)
}
```

### using 선언으로 리소스 해제

```csharp
// ✅ using var — 스코프 종료 시 자동 Dispose
using var dialog = new FolderBrowserDialog();
using var stream = File.OpenRead(path);
using var bitmap = new Bitmap(width, height);

// 여러 리소스가 중첩될 때도 깔끔
using var bitmap = new Bitmap(rect.Width, rect.Height);
using var graphics = Graphics.FromImage(bitmap);
graphics.CopyFromScreen(rect.Location, Point.Empty, rect.Size);
bitmap.Save(path, ImageFormat.Png);
```

---

## 병렬/동시성 처리

### Parallel.ForEach — CPU 바운드 병렬 처리

```csharp
// ✅ 스레드 안전 컬렉션 + 병렬 처리
var walkerBag = new ConcurrentBag<TypeWalker>();
var errorBag = new ConcurrentBag<string>();

Parallel.ForEach(filePaths,
    new ParallelOptions { MaxDegreeOfParallelism = Environment.ProcessorCount },
    filePath =>
{
    try
    {
        var code = File.ReadAllText(filePath);
        var tree = CSharpSyntaxTree.ParseText(code);
        var walker = new TypeWalker(filePath);
        walker.Visit(tree.GetCompilationUnitRoot());
        walkerBag.Add(walker);
    }
    catch (Exception ex)
    {
        errorBag.Add($"{Path.GetFileName(filePath)}: {ex.Message}");
    }
    finally
    {
        progress?.Report((Interlocked.Increment(ref current), total));
    }
});

result.Errors.AddRange(errorBag);  // 병렬 작업 완료 후 집계
```

### 스레드 안전 규칙

| 상황 | 안전한 방법 | 위험한 방법 |
|------|-----------|-----------|
| 여러 스레드에서 컬렉션 추가 | `ConcurrentBag<T>` | `List<T>` — 동시 Add 시 데이터 손실 |
| 카운터 증가 | `Interlocked.Increment()` | `count++` — race condition |
| UI 업데이트 | `IProgress<T>`, `BeginInvoke` | 직접 컨트롤 접근 — 크로스 스레드 예외 |
| 공유 상태 읽기/쓰기 | `lock` 또는 `ReaderWriterLockSlim` | 동기화 없는 접근 |

---

## WinForms 고유 패턴

> WinForms는 .NET 8에서도 공식 지원되는 데스크톱 UI 프레임워크이다.
> 아래는 WinForms 특유의 함정과 베스트 프랙티스이다.

### SplitContainer 초기화 — Shown 이벤트 필수

```csharp
// ✅ Shown 이벤트에서 설정 — 컨트롤 크기가 확정된 시점
Shown += (_, _) =>
{
    splitOuter.Panel1MinSize = 120;
    splitOuter.Panel2MinSize = 400;
    splitOuter.SplitterDistance = 190;
};

// ❌ 생성자에서 설정 — 컨트롤 크기 미확정 → InvalidOperationException
public MainForm()
{
    InitializeComponent();
    splitOuter.SplitterDistance = 190;  // 💥 예외 발생 가능
}
```

### ItemCheck 이벤트 — BeginInvoke 필수

```csharp
// ✅ BeginInvoke로 다음 메시지 루프까지 지연
private void clbNamespaces_ItemCheck(object? sender, ItemCheckEventArgs e)
{
    // ItemCheck는 상태 변경 전에 발화 → 바로 읽으면 이전 상태
    BeginInvoke(RebuildGraphFiltered);
}

// ❌ 직접 호출 — 체크 변경 전 상태 기준으로 필터링됨
private void clbNamespaces_ItemCheck(object? sender, ItemCheckEventArgs e)
{
    RebuildGraphFiltered();  // 아직 체크 상태가 안 바뀜!
}
```

### 이벤트 해제/재등록 — 프로그래매틱 업데이트 중 연쇄 방지

```csharp
// ✅ 프로그래매틱 업데이트 시 이벤트 해제 → 완료 후 재등록
private void PopulateNamespaceFilter()
{
    clbNamespaces.ItemCheck -= clbNamespaces_ItemCheck;  // 해제
    clbNamespaces.Items.Clear();

    foreach (var ns in namespaces)
        clbNamespaces.Items.Add(ns, isChecked: true);

    clbNamespaces.ItemCheck += clbNamespaces_ItemCheck;  // 재등록
}
```

### 드래그 vs 클릭 구분

```csharp
// ✅ MouseDown/Move로 드래그 감지 → MouseClick에서 무시
private Point _mouseDownPoint;
private bool _wasDragged;

private void gViewer_MouseDown(object? sender, MouseEventArgs e)
{
    _mouseDownPoint = e.Location;
    _wasDragged = false;
}

private void gViewer_MouseMove(object? sender, MouseEventArgs e)
{
    if (e.Button != MouseButtons.None)
    {
        var dx = e.X - _mouseDownPoint.X;
        var dy = e.Y - _mouseDownPoint.Y;
        if (dx * dx + dy * dy > 25)  // 5px 이상 이동
            _wasDragged = true;
    }
}

private void gViewer_MouseClick(object? sender, MouseEventArgs e)
{
    if (_wasDragged) { _wasDragged = false; return; }  // 드래그 후 클릭 무시
    // 실제 클릭 처리
}
```

### 키보드 전역 처리 — ProcessCmdKey

```csharp
// ✅ KeyPreview = true + ProcessCmdKey 오버라이드
protected override bool ProcessCmdKey(ref Message msg, Keys keyData)
{
    if (keyData == Keys.Space && !_spaceHeld)
    {
        _spaceHeld = true;
        ApplyPanMode(true);
        return true;  // 이벤트 소비 (다른 컨트롤로 전파 안 함)
    }
    return base.ProcessCmdKey(ref msg, keyData);
}
```

### Panel 동적 컨트롤 관리

```csharp
// ✅ Controls 컬렉션 조작 후 BringToFront로 Z순서 제어
pnlGraph.Controls.Clear();
pnlGraph.Controls.Add(_gViewer);

pnlGraph.Controls.Add(pnlLegend);
pnlLegend.BringToFront();  // 오버레이가 그래프 위에 표시

pnlGraph.Controls.Add(btnPanMode);
btnPanMode.BringToFront();
```

### 커스텀 그리기 (OwnerDraw)

```csharp
// ✅ ListBox, ToolStrip 등의 커스텀 렌더링
private void lstErrors_DrawItem(object? sender, DrawItemEventArgs e)
{
    if (e.Index < 0) return;

    var bg = e.Index % 2 == 0
        ? Color.FromArgb(45, 45, 48)   // 줄무늬 배경
        : Color.FromArgb(50, 50, 54);

    e.Graphics.FillRectangle(new SolidBrush(bg), e.Bounds);

    // 빨간 ● 인디케이터
    using var dotBrush = new SolidBrush(Color.FromArgb(220, 80, 80));
    e.Graphics.FillEllipse(dotBrush, e.Bounds.X + 6, e.Bounds.Y + 7, 8, 8);

    // 텍스트
    using var textBrush = new SolidBrush(Color.FromArgb(220, 170, 140));
    e.Graphics.DrawString(text, lstErrors.Font, textBrush, e.Bounds.X + 20, e.Bounds.Y + 4);
}
```

### 화면 캡처 (PNG 내보내기)

```csharp
// ✅ CopyFromScreen — 오버레이(범례 등) 포함 캡처
// DrawToBitmap은 WM_PRINT 기반이라 BringToFront된 오버레이를 캡처 못함
var screenRect = pnlGraph.RectangleToScreen(
    new Rectangle(Point.Empty, pnlGraph.Size));
using var bitmap = new Bitmap(screenRect.Width, screenRect.Height);
using (var g = Graphics.FromImage(bitmap))
    g.CopyFromScreen(screenRect.Location, Point.Empty, screenRect.Size);
bitmap.Save(path, ImageFormat.Png);
```

---

## 보안

| 취약점 | 위험 시나리오 | 방어 |
|--------|-------------|------|
| 하드코딩된 비밀키 | Git 푸시 → API 키 노출 → 과금/데이터 유출 | 환경 변수, User Secrets |
| 경로 문자열 연결 | `"C:\data\" + userInput` → `../../etc/passwd` Path Traversal | `Path.Combine()` + `Path.GetFileName()` |
| SQL 문자열 연결 | `"SELECT * FROM users WHERE id = " + input` → SQL Injection | 파라미터화 쿼리, ORM |
| `Process.Start(userInput)` | Command Injection → 임의 명령 실행 | 입력값 화이트리스트 검증 |
| `BinaryFormatter` | MS 공식 deprecated — 원격 코드 실행(RCE) 취약점 | `System.Text.Json` |
| 사용자 입력 미검증 | 예상치 못한 문자, 길이 초과 → 버퍼 오버플로우 | 입력 경계에서 검증 |

```csharp
// ✅ 경로 안전하게 조합
var safePath = Path.Combine(baseDir, Path.GetFileName(userInput));

// ✅ 환경 변수에서 시크릿 로드
var apiKey = Environment.GetEnvironmentVariable("API_KEY")
    ?? throw new InvalidOperationException("API_KEY 환경 변수 미설정");

// ❌ 하드코딩
var apiKey = "sk-abc123...";
```

---

## 성능 최적화

### 문자열 처리
```csharp
// ✅ 반복 연결 시 StringBuilder — O(N) vs O(N²)
var sb = new StringBuilder();
foreach (var node in nodes) sb.AppendLine(node.FullName);
var output = sb.ToString();

// ✅ 단순 조합은 보간 문자열
var label = $"{node.Namespace}.{node.Name}";
```

### 컬렉션 & LINQ
```csharp
// ✅ 용도에 맞는 컬렉션 — O(1) 조회가 필요하면 HashSet/Dictionary
var knownTypes = nodes.Select(n => n.Name).ToHashSet();     // O(1) Contains
var nameMap = nodes.ToDictionary(n => n.FullName);           // O(1) TryGetValue

// ✅ LINQ 지연 실행 활용 — 중간 단계에서 ToList() 금지
var filtered = nodes
    .Where(n => n.Kind == TypeKind.Class)    // 아직 실행 안 됨
    .Select(n => n.FullName);                 // 아직 실행 안 됨
var list = filtered.ToList();                 // 여기서 한 번만 실행

// ✅ 존재 확인은 Any — Count보다 빠름
if (errors.Any())  // ✅ 첫 요소만 확인
if (errors.Count() > 0)  // ❌ 전체 순회 가능 (IEnumerable인 경우)
```

### 리소스 관리
```csharp
// ✅ GDI+ 리소스(Brush, Pen, Font)는 반복 생성 금지 — 클래스 필드로 캐싱
// OwnerDraw 이벤트에서 매번 new SolidBrush() → GC 압박
private static readonly SolidBrush BackgroundBrush = new(Color.FromArgb(45, 45, 48));

// ✅ HttpClient는 재사용 (IHttpClientFactory 권장)
// ❌ 매 요청마다 new HttpClient() → 소켓 고갈 (TIME_WAIT)
```

---

## 테스트

### 규칙
- 테스트 프로젝트: `{ProjectName}.Tests` (별도 프로젝트)
- 테스트 파일명: `{대상클래스}Tests.cs`
- 메서드명: `{메서드명}_{시나리오}_{기대결과}` 패턴
- 테스트 프레임워크: xUnit (.NET 공식 권장)

### Arrange / Act / Assert 구조

```csharp
[Fact]
public void Analyze_SingleClass_ReturnsOneClassNode()
{
    // Arrange
    var analyzer = new RoslynAnalyzer();
    var path = WriteTempFile("""
        namespace MyApp;
        public class Foo { }
        """);

    // Act
    var result = analyzer.Analyze(new[] { path });

    // Assert
    Assert.Single(result.Nodes);
    Assert.Equal("Foo", result.Nodes[0].Name);
    Assert.Equal(TypeKind.Class, result.Nodes[0].Kind);
}

// ✅ 에러 케이스 테스트 — 실패해도 크래시하지 않는지 확인
[Fact]
public void Analyze_InvalidSyntax_AddsToErrorsWithoutCrash()
{
    var path = WriteTempFile("이건 C# 코드가 아닙니다 !!!");
    var result = _analyzer.Analyze(new[] { path });

    Assert.Empty(result.Nodes);
    Assert.NotEmpty(result.Errors);
}

// ✅ 엣지 케이스 — partial class 병합
[Fact]
public void Analyze_PartialClass_MergesIntoSingleNode()
{
    var path = WriteTempFile("""
        public partial class Dog { void Bark() {} }
        public partial class Dog { int _age; }
        """);

    var result = _analyzer.Analyze(new[] { path });

    var dog = Assert.Single(result.Nodes);
    Assert.Equal(1, dog.MethodCount);
    Assert.Equal(1, dog.FieldCount);
}
```

### 실행 명령
```bash
dotnet test --verbosity normal
# 출력: 통과! - 실패: 0, 통과: 21, 건너뜀: 0
```

---

## 문서화 규칙

- **자명한 코드에는 주석 생략** — 코드 자체가 의도를 드러내야 한다
- **"Why"만 주석이 설명한다** — What은 코드가, How는 타입이 설명

```csharp
// ✅ Why — 비직관적 결정의 이유
// Roslyn SyntaxTree만으로는 외부 타입 판별 불가 — 이름 기반 필터링으로 대체
var internalOnly = edges.Where(e => knownTypes.Contains(e.Target));

// ✅ Why — 라이브러리 API 제약 설명
// MSAGL 1.1.6에서 panButton이 internal → 리플렉션으로 접근
var field = viewerType.GetField("panButton", BindingFlags.NonPublic | BindingFlags.Instance);

// ✅ Public API — XML 문서 주석
/// <summary>
/// C# 소스 코드 정적 분석 인터페이스.
/// UI 레이어는 구현체에 직접 의존하지 않고 이 인터페이스를 통해 통신한다.
/// </summary>
public interface IAnalyzer
{
    AnalysisResult Analyze(IReadOnlyList<string> filePaths);
}

// ❌ What 주석 — 코드를 반복하는 무의미한 주석
// nodes 리스트를 순회한다
foreach (var node in nodes) { ... }
```

- `TODO:` / `FIXME:` / `HACK:` 태그 적극 활용
- `TODO S-XX:` 형태로 관련 태스크 번호 명시 → 파이프라인 연결 후 제거 추적

---

## 빌드 & 설정

### csproj 권장 설정
```xml
<PropertyGroup>
    <TargetFramework>net8.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
</PropertyGroup>
```

### NuGet 중앙 버전 관리 (다중 프로젝트 시)
```xml
<!-- Directory.Packages.props -->
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="xunit" Version="2.9.3" />
    <PackageVersion Include="Microsoft.NET.Test.Sdk" Version="17.13.0" />
  </ItemGroup>
</Project>

<!-- 각 .csproj에서는 버전 없이 선언 -->
<PackageReference Include="xunit" />
```

### 배포 — 단일 실행 파일
```bash
dotnet publish --configuration Release --runtime win-x64 --self-contained -p:PublishSingleFile=true
```

---

## 자주 쓰는 라이브러리/도구

### 권장

| 용도 | 권장 | 이유 |
|------|------|------|
| JSON 직렬화 | `System.Text.Json` | .NET 내장, 고성능, 보안 |
| 단위 테스트 | xUnit | .NET 공식 권장, 병렬 실행 기본 |
| Mocking | Moq / NSubstitute | 인터페이스 기반 테스트 지원 |
| DI | `Microsoft.Extensions.DependencyInjection` | .NET 표준 DI 컨테이너 |
| 로깅 | `Microsoft.Extensions.Logging` + Serilog | 구조화 로깅, 다중 싱크 |
| HTTP | `IHttpClientFactory` | 소켓 재사용, 수명 관리 |
| 코드 분석 | Roslyn (`Microsoft.CodeAnalysis`) | C# 공식 컴파일러 API |
| 그래프 시각화 | Microsoft.Msagl | WinForms 네이티브 GViewer, 계층형 레이아웃 |

### 피해야 할 것

| 라이브러리 | 이유 | 대안 |
|-----------|------|------|
| `Newtonsoft.Json` (신규 프로젝트) | .NET 내장 STJ가 더 빠르고 보안적 | `System.Text.Json` |
| `BinaryFormatter` | MS 공식 deprecated, RCE 취약점 | `System.Text.Json`, Protobuf |
| `WebClient` | deprecated, 비동기 미지원 | `HttpClient` |

---

## 실전 함정 & 지뢰

| 증상 | 원인 | 해결 | 오답 (하지 말 것) |
|------|------|------|-------------------|
| `SplitContainer` 생성자에서 `InvalidOperationException` | `SplitterDistance`를 생성자에서 설정 — 컨트롤 크기 미확정 시점 | `Shown` 이벤트에서 지연 설정 | 예외 무시 try-catch |
| `ItemCheck` 이벤트에서 체크 상태가 변경 전 값 | WinForms `CheckedListBox`는 상태 변경 **전에** 이벤트 발화 | `BeginInvoke()`로 다음 메시지 루프까지 지연 | 타이머로 딜레이 |
| `MouseClick`이 드래그 후에도 발생 | WinForms는 드래그 여부와 무관하게 MouseClick 발생 | `MouseDown`/`MouseMove`에서 이동 거리 체크 후 Click 무시 | MouseClick 이벤트 제거 |
| `async void`에서 예외 → 앱 크래시 | async void는 예외가 SynchronizationContext로 직접 전달, catch 불가 | `async Task` 반환 메서드 분리 + 호출부에서 `_ = Method()` | `AppDomain.UnhandledException`만 추가 |
| 네임스페이스 충돌 (`Color`, `Point` 등) | `System.Drawing.Color`와 라이브러리의 `Color` 타입 동시 사용 | `using 별칭` 또는 전체 경로 명시 | 하나의 using 삭제 |
| `obj/bin` 내 자동 생성 `.cs` 파일이 분석에 포함 | `SearchOption.AllDirectories`가 빌드 출력 폴더도 포함 | 파일 수집 시 `bin`/`obj` 경로 세그먼트 필터링 | 빌드 출력 삭제 |
| MSAGL `PlaneTransformation.Rotation90` 정적 속성 없음 | 패키지 버전에 따라 공개 API 차이 | 직접 변환 행렬 `new PlaneTransformation(0,-1,0,1,0,0)` | 다른 버전 강제 설치 |
| MSAGL `panButton`이 internal | 공개 API로 팬 모드 전환 불가 (1.1.6 기준) | Reflection `GetField(NonPublic \| Instance)` | 다른 라이브러리로 교체 |
| `DrawToBitmap()` 시 오버레이 캡처 안 됨 | WM_PRINT 기반이라 `BringToFront()` 된 자식이 빠짐 | `Graphics.CopyFromScreen()` — 실제 화면 픽셀 복사 | 오버레이 제거 |
| `List<T>` Parallel.ForEach에서 데이터 손실 | `List<T>`는 스레드 안전하지 않음 — 동시 Add 시 내부 배열 race | `ConcurrentBag<T>` 사용 | `lock(list)` 남용 |
| GViewer 백그라운드 스레드에서 생성 시 크래시 | UI 컨트롤은 UI 스레드(STA)에서만 생성 가능 | Graph 계산은 Task.Run, GViewer 생성은 await 후 UI 스레드에서 | `Control.Invoke()` 남용 |
| `GenericNameSyntax`가 `SimpleNameSyntax` 서브타입 | switch expression에서 Simple이 먼저 매칭되면 Generic이 안 걸림 | **구체적 타입을 먼저** 매칭 (Generic → Simple 순서) | `as` 캐스팅 후 null 체크 |

---

## 검증 체크리스트

### 새 기능 추가 시
```
- [ ] 빌드 성공 (0 Error)
- [ ] 기존 단위 테스트 전원 통과 (`dotnet test`)
- [ ] 새 기능에 대한 테스트 케이스 추가
- [ ] Nullable 경고 0건 확인
- [ ] 접근 제한자 명시적 작성 확인
- [ ] IDisposable 리소스에 using 사용 확인
- [ ] 비동기 메서드 Async 접미사 확인
- [ ] UI 스레드에서만 컨트롤 접근하는지 확인
```

### 새 클래스/인터페이스 추가 시
```
- [ ] 파일명 = 타입명 일치
- [ ] 네임스페이스가 폴더 경로와 일치
- [ ] 파일 범위 네임스페이스 사용
- [ ] XML 문서 주석 (public API)
- [ ] 올바른 레이어에 배치 (Models? Analysis? UI?)
- [ ] 인터페이스로 의존성 역전 필요 여부 판단
```

### WinForms 새 UI 컴포넌트 추가 시
```
- [ ] SplitContainer 설정은 Shown 이벤트에서
- [ ] ItemCheck 이벤트는 BeginInvoke 패턴
- [ ] 프로그래매틱 컨트롤 업데이트 시 이벤트 해제/재등록
- [ ] Dock/Anchor 설정으로 리사이즈 대응
- [ ] 커스텀 그리기 시 GDI+ 리소스 Dispose 확인
- [ ] 다크 테마 색상 일관성 확인
```

### 배포 전
```
- [ ] Release 빌드 성공
- [ ] 전체 테스트 통과
- [ ] 하드코딩된 경로/시크릿 없음
- [ ] 디버그 코드(Console.WriteLine, Debug.Assert) 제거
- [ ] 에러 시나리오 테스트 (잘못된 입력, 빈 폴더, 문법 오류 파일)
- [ ] 실행 파일에서 직접 동작 확인 (dotnet run)
```

### 에러 발생 시 디버깅 흐름
```
1. 예외 메시지 + 스택 트레이스 확인
2. 예외 발생 지점의 입력 데이터 확인
3. null 가능성 점검 (Nullable 경고 확인)
4. 리소스 해제 누락 확인 (using 누락)
5. 스레드 안전성 확인 (UI 스레드 접근, ConcurrentBag)
6. WinForms 이벤트 타이밍 확인 (ItemCheck, SplitContainer)
7. 재현 가능한 최소 테스트 케이스 작성
```

---

## 우회/핵 금지 원칙

| 우회 패턴 | 위험성 | 정석 해결 |
|----------|--------|----------|
| `#pragma warning disable` 남용 | 컴파일러 경고를 숨겨 실제 버그 간과 → Nullable 경고 무시 시 NullRef 확률 급증 | 경고 원인을 해결 (null 처리, 타입 수정) |
| `!` (null forgiving) 남발 | "null 아님을 확신"이 틀리면 NullReferenceException → 프로덕션 크래시 | 적절한 null 체크 + 패턴 매칭 |
| `catch { }` 빈 블록 | 오류 무시 → 데이터 손실/불일치가 조용히 진행 | 최소 로깅 + Errors 리스트 기록 |
| `Thread.Sleep()` 으로 타이밍 맞추기 | 환경마다 타이밍 달라 간헐적 실패 → "내 PC에선 되는데" | 이벤트/콜백/async await 사용 |
| `dynamic` 타입으로 캐스팅 회피 | 컴파일 타임 체크 무력화 → 런타임에서야 에러 발견 | 제네릭, 인터페이스, 패턴 매칭 |
| `GC.Collect()` 직접 호출 | GC 최적화 방해 → Gen2 수집 강제로 오히려 성능 저하 | 메모리 누수 근본 원인 해결 (이벤트 해제, Dispose) |
| Reflection으로 private 멤버 접근 | 버전 업데이트 시 필드명 변경되면 런타임 크래시 | 공개 API 사용. 불가피하면 `// HACK:` 주석 + 버전 명시 |

**핵심 원칙:**
- 증상 해결(땜빵)이 아닌 **근본 원인**을 먼저 진단한다
- 컴파일러 경고를 숨기지 않고 **원인을 해결**한다
- "일단 동작하게 하고 나중에 고치자"는 기술 부채가 아니라 **시한폭탄**이다
- 보안 메커니즘은 우회하지 않고 **올바르게 설정**한다

---

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
> - 테스트 전략 상세 (커버리지 기준, E2E 등)

<!-- 개선 이력
- 2026-04-06: Level 1(55점) → Level 3(85점+) 전면 재작성
- 2026-04-06: WinForms 고유 패턴 섹션 추가 — SplitContainer/ItemCheck/드래그/키보드/OwnerDraw/화면캡처
- 2026-04-06: 병렬/동시성 섹션 추가 — Parallel.ForEach, ConcurrentBag, CancellationToken, IProgress
- 2026-04-06: 실전 함정 12개 추가 — 실제 프로젝트에서 발견된 WinForms/MSAGL/Roslyn 이슈
- 2026-04-06: 코드 예시 40개+ — 실제 프로젝트 패턴 기반 범용화
- 2026-04-06: 안티패턴 테이블 — 실패 결과를 구체적 시나리오로 명시
- 2026-04-06: 검증 체크리스트 5종 — 새 기능/새 클래스/WinForms UI/배포 전/디버깅
-->
