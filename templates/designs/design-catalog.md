# Design System Recommendation Catalog

> 이 카탈로그는 54개 DESIGN.md 레퍼런스를 분류하고,
> 디자인을 모르는 사용자에게 최적의 디자인을 추천하기 위한 가이드입니다.
> `project-init` 에이전트가 디자인 선택 단계에서 이 파일을 참조합니다.

---

## 사용법

1. project-init이 이 파일을 읽는다
2. 아래 인터뷰 질문으로 사용자의 프로젝트를 파악한다
3. 태그 매트릭스에서 조건에 맞는 디자인을 필터링한다
4. 상위 2~3개를 추천하고 설명한다
5. 사용자가 선택하면 `templates/designs/{brand}.md`를 프로젝트 루트에 `DESIGN.md`로 복사한다

---

## 디자인 인터뷰 질문

> 모든 질문을 기계적으로 던지지 않는다. 대화 흐름에 맞게 자연스럽게 질문하되,
> 추천 정확도를 위해 **최소 Phase 1-2는 반드시 파악**한다.
> 사용자가 "Notion 같은 느낌"처럼 직접 레퍼런스를 말하면 Phase 2-3 대부분 스킵 가능.

### Phase 1 — 앱 이해 (필수, 4개)

| # | 질문 | 파악 목적 | 태그 매핑 |
|---|------|----------|----------|
| 1 | "어떤 앱이에요? 핵심 기능이 뭐예요?" | 앱 유형 | best_for |
| 2 | "누가 사용하나요? 개발자? 일반인? 기업 담당자?" | 대상 사용자 | audience |
| 3 | "주요 화면이 어떤 게 있을 것 같아요? (대시보드, 목록, 에디터, 채팅 등)" | 기능적 요구사항 | density, has_data_tables, best_for 교차 확인 |
| 4 | "비슷한 느낌의 앱이나 사이트가 있으면 말해주세요 (없어도 괜찮아요)" | 직접 레퍼런스 | 있으면 즉시 매칭 가능 |

### Phase 2 — 시각적 선호 (필수, 4개)

| # | 질문 | 파악 목적 | 태그 매핑 |
|---|------|----------|----------|
| 5 | "밝은 화면이 좋아요? 어두운 화면? 상관없어요?" | 테마 | theme |
| 6 | "화면에 정보가 빽빽한 게 좋아요? 여백이 넉넉한 게 좋아요?" | 정보 밀도 | density |
| 7 | "어떤 분위기가 끌려요? 예를 들어: 깔끔하고 차분한? 따뜻하고 친근한? 세련되고 고급스러운? 대담하고 강렬한? 미래적인?" | 무드 | mood |
| 8 | "좋아하는 색상이나 싫어하는 색상 있어요? (없으면 괜찮아요)" | 색상 선호 | color_strategy, accent_color 보조 필터 |

### Phase 3 — 기능적 디자인 (해당 시, 2~3개)

> Phase 1-2 답변으로 불충분할 때만 추가 질문.
> Phase 1의 "주요 화면" 답변에서 이미 파악되면 스킵.

| # | 질문 | 파악 목적 | 태그 매핑 |
|---|------|----------|----------|
| 9 | "데이터 테이블이나 차트가 주요 기능인가요?" | 데이터 밀도 | has_data_tables, density:dense |
| 10 | "글/문서를 읽고 쓰는 게 메인인가요?" | 콘텐츠 중심 | density:spacious, mood:editorial |
| 11 | "이미지나 영상이 핵심 콘텐츠인가요?" | 미디어 중심 | best_for:media |

### 질문 흐름 가이드

```
[Phase 1] 앱 이해
  → "Notion 같은 느낌이요" → 바로 Notion 기반 추천 (Phase 2-3 스킵 가능)
  → "할일 관리 앱이요" → Phase 2로
  → "데이터 분석 대시보드요" → Phase 2 + Q9 필수

[Phase 2] 시각적 선호
  → 밝은 + 여백 + 따뜻 → Airbnb, Claude, Zapier 계열
  → 어두운 + 빽빽 + 전문적 → Linear, Raycast, Composio 계열
  → 밝은 + 고급 → Stripe, Apple, Superhuman 계열
  → 상관없음 다수 → 앱 유형 기반으로만 추천

[Phase 3] 필요 시만
  → 데이터 테이블 Yes → Airtable, ClickHouse, Supabase, Linear
  → 콘텐츠 중심 Yes → Notion, Claude, Mintlify
  → 미디어 중심 Yes → Pinterest, RunwayML, Spotify
```

---

## 추천 시 설명 방식

추천할 때 각 디자인을 이렇게 설명한다:

```
추천 1: Linear
  - 분위기: 어두운 배경에 인디고 포인트, 정교하고 전문적인 느낌
  - 특징: 개발자 도구에 최적화된 깔끔한 다크 UI
  - 적합: 프로젝트 관리, 이슈 트래커, 생산성 도구
  - 한마디: "개발자가 매일 쓰는 도구라면 이 스타일"

추천 2: Notion
  - 분위기: 밝고 따뜻한 배경에 조용한 보더, 집중되는 느낌
  - 특징: 콘텐츠 중심의 깔끔한 UI, 위스퍼급 보더
  - 적합: 생산성 도구, 위키, 문서, 협업 플랫폼
  - 한마디: "글과 데이터를 다루는 도구라면 이 스타일"
```

---

## 플랫폼별 DESIGN.md 적용 가이드

> DESIGN.md는 웹 기반으로 작성되어 있다. 플랫폼에 따라 적용 범위가 다르다.
> AI는 이 가이드를 참조하여 **가능한 부분만 자연스럽게 적용**하고,
> 적용 불가한 부분은 건너뛴다. 92→95를 위해 DESIGN.md를 다시 쓰지 않는다.

### 웹 (React, Vue, Next.js, HTML/CSS 등) — 적용도 95%+

DESIGN.md의 모든 섹션을 그대로 적용:
- 색상 팔레트 → CSS 변수 / Tailwind 설정
- 타이포그래피 → font-family, font-size, line-height, letter-spacing
- 컴포넌트 → 버튼, 카드, 인풋 스타일 그대로
- 그림자/깊이 → box-shadow 그대로
- 반응형 → breakpoint 그대로
- border-radius → 그대로

### 모바일 (React Native, Flutter) — 적용도 75%

적용 가능:
- 색상 팔레트 → 그대로 사용
- 타이포그래피 계층 → 크기/굵기/행간 적용 (폰트는 플랫폼 대체)
- 간격 시스템 → 8px 그리드 그대로
- 컴포넌트 구조 → 버튼/카드/인풋 스타일 개념 적용
- Do's/Don'ts → 그대로

변환 필요:
- box-shadow → React Native: `elevation` + `shadowColor/Offset/Opacity/Radius` / Flutter: `BoxShadow`
- border-radius → 수치는 같지만 API가 다름
- 반응형 breakpoint → 화면 크기 기반 조건부 스타일
- 웹 전용 CSS (backdrop-filter, linear-gradient 등) → 플랫폼 대안 사용

### 데스크톱 (WinForms, WPF, Electron, Tauri) — 적용도 40~60%

적용 가능:
- **색상 팔레트** → 배경색, 텍스트색, 강조색, 보더색 적용
- **타이포그래피 계층** → 제목/본문/캡션 크기 비율과 굵기 적용
- **분위기/철학** → 전체적 톤앤매너 (밝은/어두운, 여백감, 색상 톤)
- **Do's/Don'ts** → 디자인 원칙 적용

적용 불가 (건너뜀):
- box-shadow → WinForms에서 그림자 지원 제한적
- border-radius → WinForms 기본 컨트롤은 각진 형태
- 반응형 breakpoint → 윈도우 리사이즈 대응은 다른 패러다임
- 웹 컴포넌트 스타일 → 네이티브 컨트롤 자체 스타일 활용
- pill 형태 버튼 → WinForms에서 구현 어려움

WPF/MAUI는 ControlTemplate으로 커스텀 가능하므로 적용도 60~70%까지 올라감.

### CLI / 터미널 — 적용도 10~15%

적용 가능:
- **색상 팔레트에서 ANSI 매핑** → Primary → Bold, Accent → Color, Muted → Dim
- **분위기** → 전체적 톤 참조

나머지는 해당 없음.

### 핵심 원칙

```
1. DESIGN.md에서 적용 가능한 부분을 최대한 활용한다
2. 적용 불가한 부분은 조용히 건너뛴다 (경고나 에러 아님)
3. 플랫폼 네이티브 방식으로 자연스럽게 번역한다
4. 80% 적용으로 충분하다 — 95%를 위해 DESIGN.md를 다시 쓰지 않는다
5. 색상 팔레트와 분위기는 모든 플랫폼에서 적용 가능한 보편적 가치다
```

---

## 54개 디자인 태그 매트릭스

> 각 디자인의 상세 태그. 추천 필터링에 사용한다.
> `templates/designs/{brand}.md`에 해당 DESIGN.md 원본이 있다.

### 태그 설명

| 태그 | 값 | 설명 |
|------|-----|------|
| theme | dark / light / both | 기본 테마 |
| density | dense / comfortable / spacious / relaxed | 정보 밀도 |
| mood | professional, warm, minimal, bold, premium, playful, futuristic, technical, editorial, cinematic, elegant, ethereal, neutral, immersive, confident, friendly, clean | 시각적 분위기 |
| audience | developer / consumer / enterprise / designer / creative | 주요 사용자층 |
| shape | sharp (0-4px) / rounded (6-12px) / very-rounded (16px+) / pill (9999px) / mixed | 모서리 형태 |
| color_strategy | monochrome / single-accent / multi-color / gradient | 색상 전략 |
| typography | sans / serif / mono / mixed | 주요 서체 유형 |
| best_for | 적합한 앱 유형 (복수) | 추천 근거 |
| has_data_tables | yes / no | 데이터 테이블 포함 여부 |
| key_feature | 이 디자인만의 차별점 | 추천 시 설명에 활용 |
| not_for | 부적합한 상황 | 잘못된 추천 방지 |

### AI & Machine Learning (12개)

| brand | theme | density | mood | audience | shape | color | typo | best_for | tables | key_feature | not_for |
|-------|-------|---------|------|----------|-------|-------|------|----------|--------|-------------|---------|
| claude | light | spacious | warm, editorial | consumer | rounded | single-accent (#c96442 Terracotta) | serif | AI 제품, 에디토리얼, 인문학적 브랜드 | no | 세리프 + 따뜻한 양피지(#f5f4ed) = 인간적 기술 브랜드 | 대시보드, 개발자 도구, 속도감 UI |
| cohere | light | comfortable | professional | enterprise | rounded | single-accent (#39594d Forest Green) | mixed | 기업 AI/ML 플랫폼, B2B SaaS | no | 22px 시그니처 border-radius + 세리프/산세리프 이중 시스템 | 소비자 앱, 개발자 터미널 |
| elevenlabs | light | spacious | ethereal | consumer | pill | monochrome | sans | 음성/오디오 AI, 미디어 기술, 고급 브랜드 | no | weight 300 + 0.1 이하 불투명도 그림자 = 공기처럼 가벼운 UI | 데이터 밀도 앱, 굵은 타이포 |
| minimax | light | comfortable | playful | consumer | pill | multi-color | sans | AI 멀티 프로덕트 플랫폼, 크리에이티브 도구 | no | 다중 폰트(4종) + 멀티컬러 카드 = 다양한 AI 제품 쇼케이스 | 미니멀 단일 브랜드, 개발자 문서 |
| mistral.ai | dark | spacious | warm, futuristic | developer | sharp | gradient (#fa520f Orange) | sans | AI/ML 스타트업, 오픈소스 AI 플랫폼 | no | 골든앰버 그라디언트 + 거의 0 라운드 = 따뜻한 우주 미학 | 밝은 소비자 앱, 기업 대시보드 |
| ollama | light | spacious | minimal | developer | pill | monochrome | sans | 로컬 AI 도구, 오픈소스 CLI, 미니멀 랜딩 | no | 그림자 0개 + 순수 그레이스케일 + 필 = 극단적 미니멀 | 풍부한 비주얼, 데이터 대시보드 |
| opencode.ai | dark | dense | technical | developer | sharp | monochrome | mono | CLI 도구, 터미널 앱, 코드 에디터 | no | 단일 모노스페이스 + 따뜻한 블랙(#201d1d) | 소비자 앱, 비개발자 대상 |
| replicate | light | spacious | playful | developer | pill | gradient | sans | AI/ML 플랫폼, 모델 마켓플레이스 | no | 128px 디스플레이 + 필 + 그라디언트 히어로 = 장난스러운 AI | 기업 격식, 보수적 디자인 |
| runwayml | dark | spacious | cinematic | creative | rounded | monochrome | sans | 영상/이미지 AI, 크리에이티브 도구, 미디어 플랫폼 | no | 단일 서체 + 풀블리드 사진 + 그림자 0 = 시네마틱 갤러리 | 텍스트 중심 앱, 데이터 대시보드 |
| together.ai | dark | comfortable | futuristic | developer | sharp | gradient | sans | AI 인프라, GPU 클라우드, ML 플랫폼 | no | "The Future" 폰트 + 파스텔 그라디언트 = 미래지향 AI | 따뜻한 톤, 보수적 기업 |
| voltagent | dark | dense | technical | developer | rounded | single-accent (#00d992 Emerald) | sans + mono | AI 에이전트 프레임워크, 개발자 SDK | no | 카본블랙(#050507) + 에메랄드 그린 = 터미널 네이티브 | 소비자 앱, 비기술 사용자 |
| x.ai | dark | dense | technical | developer | sharp | monochrome | mono + sans | AI 인프라, API 플랫폼, 기술 브랜드 | no | GeistMono 320px + 0px 라운드 = 터미널 브루탈리즘 | 소비자 앱, 부드러운 디자인 |

### Developer Tools & Platforms (14개)

| brand | theme | density | mood | audience | shape | color | typo | best_for | tables | key_feature | not_for |
|-------|-------|---------|------|----------|-------|-------|------|----------|--------|-------------|---------|
| cursor | light | spacious | warm | developer | rounded | single-accent (#f54e00 Orange) | mixed | AI 코드 에디터, 개발자 생산성 도구 | no | 3종 서체(고딕+세리프+모노) 조합의 따뜻한 개발자 도구 | 기업 대시보드, 다크 모드 전용 |
| expo | light | spacious | neutral | developer | pill | monochrome | sans | 개발자 프레임워크, 문서 사이트 | no | 쿨그레이(#f0f0f3) + 필 = 갤러리 같은 기술 문서 | 컬러풀 마케팅, 감성적 브랜드 |
| linear.app | dark | dense | technical | developer | rounded | single-accent (#5e6ad2 Indigo) | sans | 이슈 트래커, 프로젝트 관리, 개발자 생산성 | no | OpenType cv01/ss03 + #08090a 배경 = 정교한 다크 UI | 밝은 소비자 앱, 사진 중심 |
| lovable | light | spacious | warm | consumer | pill | monochrome | sans | AI 제품, 크리에이티브 도구, 감성적 랜딩 | no | 불투명도 기반 색상 시스템 + 필 = 부드러운 감성 | 데이터 대시보드, 기업 도구 |
| mintlify | light | comfortable | clean | developer | very-rounded | single-accent (#18E299 Green) | sans + mono | 개발자 문서, API 레퍼런스, 기술 블로그 | no | 울트라 라운드(16-24px) + 에메랄드 그린 = 현대적 문서 | 소비자 마케팅, 다크 테마 |
| posthog | light | relaxed | playful | developer | rounded | single-accent (#F54E00 Orange hover) | sans | 제품 분석, A/B 테스트, 개발자 중심 분석 | yes | 세이지/올리브 + 호버 시 오렌지 = 장난기 있는 분석 | 기업 격식, 다크 모드 |
| raycast | dark | dense | technical | developer | rounded | single-accent (#FF6363 Red) | sans | macOS 런처, 개발자 생산성, 데스크톱 앱 | no | macOS 네이티브 그림자 + #07080a 배경 = 네이티브 데스크톱 | 웹 앱, 밝은 테마 |
| resend | dark | spacious | premium | developer | rounded | multi-color | mixed | 이메일 API, 개발자 도구, 프리미엄 기술 브랜드 | no | 3중 서체 + 빙하빛 보더 = 갤러리급 다크 UI | 밝은 소비자 앱, 데이터 밀도 |
| sentry | dark | comfortable | bold | developer | rounded | single-accent (#c2ef4e Lime) | sans | 에러 모니터링, DevOps 대시보드 | yes | 다크 퍼플 + 라임그린 + 프로스티드 글래스 = 독특한 관측 | 밝은 소비자 앱, 미니멀 |
| supabase | dark | comfortable | technical | developer | pill | single-accent (#3ecf8e Emerald) | sans | BaaS 플랫폼, 개발자 인프라, 오픈소스 | yes | HSL 기반 디자인 토큰 + 에메랄드 그린 필 = 체계적 다크 개발자 | 밝은 소비자 앱, 비개발자 |
| superhuman | light | spacious | premium | consumer | rounded | single-accent (#cbb7fb Lavender) | sans | 프리미엄 이메일, 럭셔리 생산성, 고급 SaaS | no | 라벤더 + weight 460 + 따뜻한 크림 = 럭셔리 소프트웨어 | 개발자 도구, 저가 시장 |
| vercel | light | spacious | premium | developer | rounded | multi-color | sans | 프론트엔드 플랫폼, 배포 도구, 개발자 인프라 | no | 그림자를 보더처럼 사용 + Geist 극한 트래킹 = 갤러리급 라이트 | 다크 모드 전용, 소비자 마케팅 |
| warp | dark | spacious | warm | developer | pill | monochrome | sans | 터미널 앱, 개발자 생산성, 라이프스타일+개발 혼합 | no | 자연 사진 + 따뜻한 양피지 텍스트 = 캠프파이어 터미널 | 기업 대시보드, 컬러풀 UI |
| zapier | light | comfortable | warm | consumer | rounded | single-accent (#ff4f00 Orange) | mixed | 자동화 플랫폼, SaaS 통합, 비기술 사용자 도구 | no | 3종 서체 + 0.90 line-height = 따뜻한 자동화 | 개발자 터미널, 다크 모드 |

### Infrastructure & Cloud (6개)

| brand | theme | density | mood | audience | shape | color | typo | best_for | tables | key_feature | not_for |
|-------|-------|---------|------|----------|-------|-------|------|----------|--------|-------------|---------|
| clickhouse | dark | dense | bold | developer | sharp | single-accent (#faff69 Neon Yellow) | sans | 데이터베이스 제품, 개발자 인프라, 고성능 도구 | yes | 네온 #faff69 + weight 900 = 공격적 개발자 브랜딩 | 부드러운 소비자 앱 |
| composio | dark | dense | technical | developer | sharp | single-accent (#00ffff Cyan) | sans + mono | 개발자 도구, API 플랫폼, CLI 인터페이스 | no | 전기 시안 + 4px 각진 모서리 = 터미널 미학 | 소비자 앱, 비기술 사용자 |
| hashicorp | both | comfortable | professional | developer | rounded | multi-color (per-product) | sans | 인프라 도구, 멀티 프로덕트 플랫폼, DevOps | yes | 제품별 고유 색상(Terraform 보라, Vault 노랑) 시스템 | 단일 브랜드, 소비자 앱 |
| mongodb | dark | comfortable | bold | developer | pill | single-accent (#00ed64 Neon Green) | mixed | 데이터베이스 플랫폼, 개발자 인프라 | yes | 틸블랙 + 네온 그린 + 세리프 = 독특한 DB 브랜드 | 밝은 소비자 앱, 미니멀 |
| sanity | dark | spacious | bold | developer | pill | single-accent (#f36458 Coral) | sans | CMS 플랫폼, 콘텐츠 인프라, 헤드리스 CMS | no | 극한 letter-spacing + 코랄레드 CTA + 필 = 대담한 CMS | 보수적 기업, 밝은 테마 |
| stripe | light | spacious | premium | developer | rounded | single-accent (#533afd Purple) | sans | 결제 플랫폼, 핀테크 API, 개발자 금융 | yes | weight 300 + 블루틴트 그림자 + 퍼플 = 프리미엄 개발자 금융 | 어두운 테마, 무거운 타이포 |

### Design & Productivity (10개)

| brand | theme | density | mood | audience | shape | color | typo | best_for | tables | key_feature | not_for |
|-------|-------|---------|------|----------|-------|-------|------|----------|--------|-------------|---------|
| airtable | light | comfortable | professional | enterprise | rounded | single-accent (#1b61c9 Blue) | sans | SaaS 대시보드, 데이터 관리, 협업 플랫폼 | yes | 다층 그림자와 12px 라운드 = 친근한 기업 도구 | 소비자 앱, 미니멀 랜딩 |
| cal | both | comfortable | neutral | developer | mixed | monochrome | sans | 스케줄링 앱, 오픈소스 SaaS, 생산성 도구 | no | 다층 그림자 + 극단적 반경 범위(2px~9999px) | 컬러풀 브랜딩, 감성적 디자인 |
| figma | light | comfortable | bold | designer | pill | monochrome | sans | 디자인 도구, 크리에이티브 플랫폼, 협업 도구 | no | 가변 웨이트 320~540 + 순수 흑백 = 타이포그래피가 곧 브랜드 | 다크 모드, 컬러 악센트 필요 |
| framer | dark | spacious | bold | designer | pill | single-accent (#0099ff Blue) | sans | 디자인/프로토타이핑, 크리에이티브 랜딩 | no | letter-spacing -5.5px 극한 트래킹 + 블랙 보이드 | 밝은 테마, 데이터 대시보드 |
| intercom | light | comfortable | warm | enterprise | sharp | single-accent (#ff5600 Orange) | sans | 고객 커뮤니케이션, 챗봇, CRM, B2B SaaS | no | 따뜻한 양피지 + 4px 각진 모서리 = 친근한 B2B | 개발자 터미널, 다크 모드 |
| miro | light | comfortable | friendly | enterprise | rounded | multi-color (#5b76fe Blue) | sans | 협업 화이트보드, 디자인 워크숍, 팀 도구 | no | 파스텔 멀티컬러 + Roobert PRO = 친근한 협업 | 어두운 개발자 도구, 데이터 밀도 |
| notion | light | comfortable | neutral | consumer | rounded | single-accent (#0075de Blue) | sans | 생산성 도구, 위키/문서, 협업 플랫폼 | yes | rgba 0.1 위스퍼 보더 + 따뜻한 뉴트럴 = 집중되는 UI | 화려한 마케팅, 다크 모드 전용 |
| pinterest | light | relaxed | warm | consumer | very-rounded | single-accent (#e60023 Red) | sans | 이미지 큐레이션, 소셜 커머스, 라이프스타일 | no | 넉넉한 라운드(16-40px) + 핀 레드 = 따뜻한 발견의 공간 | 개발자 도구, 다크 테마 |
| clay | light | relaxed | playful | consumer | rounded | multi-color | sans | 크리에이티브 도구, 마케팅 사이트, 데이터 플랫폼 | no | rotateZ 호버 애니메이션 + 멀티컬러 스와치 | 기업 대시보드, 미니멀 개발자 도구 |
| webflow | light | comfortable | confident | designer | sharp | multi-color (#146ef5 Blue) | sans | 노코드 웹빌더, 디자인 도구, 에이전시 포트폴리오 | no | 5층 캐스케이딩 그림자 + 6색 팔레트 = 도구 중심 디자인 | 미니멀 개발자 도구, 모노크롬 |

### Fintech & Crypto (4개)

| brand | theme | density | mood | audience | shape | color | typo | best_for | tables | key_feature | not_for |
|-------|-------|---------|------|----------|-------|-------|------|----------|--------|-------------|---------|
| coinbase | both | comfortable | professional | consumer | pill | single-accent (#0052ff Blue) | sans | 핀테크, 암호화폐, 금융 서비스 | no | 56px 필 버튼 + 4종 전용 폰트 = 신뢰감 핀테크 | 감성적/에디토리얼 디자인 |
| kraken | dark | comfortable | bold | consumer | rounded | single-accent (#7132f5 Purple) | sans | 암호화폐 거래소, 금융 대시보드 | yes | 강렬한 보라 + 12px 라운드 = 대담한 핀테크 | 따뜻한 톤, 에디토리얼 |
| revolut | light | spacious | bold | consumer | pill | monochrome | sans | 핀테크 앱, 네오뱅크, 금융 서비스 마케팅 | no | 136px 디스플레이 + 필 + 그림자 0 = 빌보드 핀테크 | 데이터 대시보드, 복잡한 UI |
| wise | light | spacious | bold | consumer | pill | single-accent (#9fe870 Lime) | sans | 해외 송금, 핀테크, 글로벌 금융 | no | weight 900 + line-height 0.85 = 빌보드 핀테크 | 섬세한 디자인, 다크 테마 |

### Enterprise & Consumer (8개)

| brand | theme | density | mood | audience | shape | color | typo | best_for | tables | key_feature | not_for |
|-------|-------|---------|------|----------|-------|-------|------|----------|--------|-------------|---------|
| airbnb | light | relaxed | warm | consumer | rounded | single-accent (#ff385c Red) | sans | 여행/마켓플레이스, 사진 중심 플랫폼 | no | 사진이 UI 핵심 = 라이프스타일 디자인 | 데이터 대시보드, 개발자 도구 |
| apple | both | spacious | premium | consumer | pill | monochrome (#0071e3 Blue) | sans | 프리미엄 제품 쇼케이스, 테크 브랜드 | no | 라이트/다크 양면 + 필 CTA = 프리미엄 쇼케이스 | 데이터 밀도 높은 앱 |
| bmw | dark | comfortable | bold | consumer | sharp | single-accent (#1c69d4 Blue) | sans | 자동차/제조업, 럭셔리 제품 | no | border-radius 0px + 대문자 = 산업적 정밀함 | 부드러운 소비자 앱 |
| ibm | both | comfortable | professional | enterprise | sharp | single-accent (#0f62fe Blue) | sans | 대기업 플랫폼, 엔터프라이즈 SaaS, 공공 서비스 | yes | border-radius 0px + weight 300 = 정밀한 기업 미학 | 스타트업, 감성적 소비자 앱 |
| nvidia | dark | comfortable | bold | developer | sharp | single-accent (#76b900 Green) | sans | GPU/하드웨어, AI 인프라, 고성능 컴퓨팅 | no | 날카로운 라운드 + NVIDIA 그린 = 하드웨어 정밀 브랜딩 | 부드러운 소비자 앱 |
| spacex | dark | spacious | cinematic | consumer | sharp | monochrome | sans | 우주/항공, 시네마틱 쇼케이스, 사진 전용 랜딩 | no | UI 크롬 0 + 대문자 D-DIN = 극한 시네마틱 | 인터랙티브 앱, 폼/대시보드 |
| spotify | dark | comfortable | immersive | consumer | pill | single-accent (#1ed760 Green) | sans | 음악/오디오 스트리밍, 미디어 앱, 엔터테인먼트 | no | #121212 다크 + 그린 필 = 몰입형 미디어 앱 | 밝은 기업 도구, 문서 중심 |
| uber | both | comfortable | bold | consumer | pill | monochrome | sans | 모빌리티 앱, 배달 플랫폼, 글로벌 서비스 | no | 흑백 전용 + 필 + 따뜻한 일러스트 = 글로벌 서비스 | 데이터 대시보드, 개발자 도구 |

---

## 빠른 검색 인덱스

### 테마별
- **다크**: linear.app, supabase, spotify, raycast, composio, opencode.ai, x.ai, voltagent, clickhouse, sentry, mongodb, framer, sanity, resend, together.ai, mistral.ai, runwayml, warp, nvidia, bmw, spacex, kraken
- **라이트**: airbnb, notion, stripe, airtable, claude, cursor, expo, ollama, elevenlabs, mintlify, posthog, pinterest, figma, intercom, miro, clay, webflow, minimax, lovable, revolut, wise, zapier, superhuman, replicate, vercel
- **양면**: apple, cal, hashicorp, ibm, coinbase, uber

### 데이터 테이블 있는 디자인
airtable, clickhouse, hashicorp, ibm, kraken, mongodb, notion, posthog, sentry, stripe, supabase

### 앱 유형별 추천 (상위 3개)
- **대시보드/분석**: Linear, Airtable, Supabase
- **생산성/문서**: Notion, Cal, Superhuman
- **개발자 도구**: Vercel, Cursor, Mintlify
- **소비자 앱**: Airbnb, Spotify, Uber
- **핀테크/금융**: Stripe, Coinbase, Revolut
- **마케팅/랜딩**: Apple, Replicate, Framer
- **CMS/콘텐츠**: Sanity, Notion, Claude
- **협업 도구**: Miro, Notion, Airtable
- **이메일/메시징**: Resend, Superhuman, Intercom
- **미디어/크리에이티브**: RunwayML, Spotify, Pinterest
- **CLI/터미널**: OpenCode, Warp, Composio
- **디자인 도구**: Figma, Framer, Webflow
- **기업/엔터프라이즈**: IBM, Airtable, HashiCorp
