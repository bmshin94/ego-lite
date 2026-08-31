# ego lite 완전 정리 (한국어 가이드)

> 이 문서는 `ego-lite` 저장소를 분석하고 정리한 한국어 요약 가이드입니다.

## 저장소 주소

| 구분 | 주소 |
|---|---|
| 원본 (Upstream) | https://github.com/citrolabs/ego-lite |
| 내 포크 (Fork) | https://github.com/bmshin94/ego-lite |
| 공식 문서 | https://lite.ego.app/document/ |
| 로드맵 | https://lite.ego.app/roadmap |
| Discord | https://discord.gg/5eGZVvHbTq |
| X (Twitter) | https://x.com/ego_agent |

라이선스: **MIT** (저장소 내용) / ego lite 브라우저 앱은 별도 무료 다운로드

---

## 1. 이게 뭐야?

**한 줄 요약: "나(사람)와 AI 에이전트가 함께 쓰는 브라우저"**

`ego lite`는 Chromium을 커널 레벨부터 수정해서 만든 브라우저로,
사람과 AI 에이전트가 **같은 브라우저를 동시에** 쓸 수 있게 설계됐습니다.

### 기존 도구들의 문제

| 도구 유형 | 예시 | 문제점 |
|---|---|---|
| 자동화 프레임워크 | browser-use, agent-browser(Vercel), Playwright | 브라우저를 따로 띄워야 함 → **내 로그인 정보가 안 넘어감**, 탭 충돌 |
| AI 내장 브라우저 | ChatGPT Atlas, Perplexity Comet | **내장 AI만** 조종 가능 → Claude Code 같은 외부 에이전트 못 씀 |

### ego lite의 해결책

- **Space(작업공간)**: 에이전트마다 격리된 브라우징 컨텍스트를 배정
  - 에이전트는 자기 Space에서 작업 → 내 탭은 그대로
- **로그인 상태 상속**: Space는 사용자의 쿠키/세션을 물려받음
  - → 로그인이 필요한 사이트도 바로 자동화 가능 (**최대 강점**)
- **병렬 실행**: Space를 여러 개 만들어 여러 에이전트/작업 동시 진행
- **통제권 핸드오프**: 언제든 사용자가 통제권을 가져오거나 넘겨줄 수 있음

### 비교표 (README 기준)

| 기능 | ego lite | Browser-Use | agent-browser | Atlas | Comet |
|---|:---:|:---:|:---:|:---:|:---:|
| 병렬 멀티태스킹 | ✓ | — | — | — | — |
| 재사용 가능한 스킬 | ✓ | — | — | — | — |
| 크롬 데이터 상속 | ✓ | — | — | ✓ | ✓ |
| 외부 에이전트 제어 | ✓ | ✓ | ✓ | — | — |
| 로컬 데이터 저장 | ✓ | ✓ | ✓ | — | — |
| 일상용 브라우저 | ✓ | — | — | ✓ | ✓ |
| 무료 | ✓ | ✓ | ✓ | — | — |

---

## 2. 왜 빠르고 저렴한가? (핵심 기술 포인트)

### 2-1. CLI 방식이 아니라 "코드 방식"

기존 MCP/CLI 방식은 한 동작마다 왕복합니다.

```
"클릭해" → "했어" → "지금 뭐 보여?" → "이거" → "입력해" → "했어" → ...
```

ego-browser는 **JS 스크립트를 한 번에** 실행합니다.

```bash
ego-browser nodejs <<'EOF'
const task = await useOrCreateTaskSpace('작업 이름')
await openOrReuseTab('https://example.com', { wait: true })
cliLog(await snapshotText())
EOF
```

→ 벤치마크상 agent-browser 대비 **최대 2.5배 빠름, 토큰 대폭 절감**

### 2-2. learnings — 쓸수록 똑똑해지는 구조

`skills/ego-browser/learnings/<사이트>/` 에 사이트별 공략집을 축적합니다.

```
learnings/x-com/
├── manifest.json          # 도구 정의 (nodeTools / browserTools)
├── notes/overview.md      # "트윗 본문은 [data-testid=tweetText]" 같은 메모
├── notes/timeline.md
├── tools/timeline.js      # 재사용 가능한 Node 함수
└── browser-tools/extract-post.js
```

현재는 **`x-com`(X/트위터), `google` 딱 2개**만 존재 → 기여/선점 기회.

---

## 3. 폴더 구조

| 경로 | 내용 |
|---|---|
| `skills/ego-browser/SKILL.md` | **핵심.** 에이전트가 읽는 사용 설명서 (209줄) |
| `skills/ego-browser/references/install.md` | 설치 가이드 (macOS 전용) |
| `skills/ego-browser/references/video.md` | 화면 녹화(`page.screencast`) 사용법 |
| `skills/ego-browser/scripts/install.sh` | 실제 설치 스크립트 (DMG 다운/설치/실행) |
| `skills/ego-browser/learnings/` | 사이트별 재사용 노하우 팩 |
| `skills/ego-browser/agents/openai.yaml` | Codex/OpenAI 인터페이스 정의 |
| `package/ego-browser/src/` | **엔진.** TypeScript CDP 하네스 |
| `package/ego-browser/src/driver/` | pointer / keyboard / observe / nav / waits / files |
| `package/ego-browser/src/learning/` | 사이트 스킬 탐색·검증·실행 |
| `.claude-plugin/marketplace.json` | Claude Code 플러그인 등록 |
| `.codex-plugin/plugin.json` | Codex 플러그인 등록 |
| `AGENTS.md` | 기여자용 아키텍처 문서 |
| `spec/agent-skills-spec.md` | Agent Skills 스펙 링크 |

> **중요**: 이 저장소에는 **브라우저 본체가 없습니다.**
> 오픈소스로 공개된 건 "에이전트 ↔ 브라우저 연결 계층"뿐이고,
> ego lite 앱(클로즈드소스)이 이 런타임을 내장한 `ego-browser` 바이너리를 제공합니다.

---

## 4. 설치

### 사전 조건

- **macOS 전용** (Windows / Linux는 로드맵 단계)
- Apple Silicon / Intel 모두 지원

### 방법 A. 에이전트에게 시키기 (가장 편함)

```
ego lite 설치해줘: https://github.com/citrolabs/ego-lite
skills/ego-browser/references/install.md 읽고 그대로 따라해줘.
```

### 방법 B. 스킬만 먼저 설치

```bash
npx skills add citrolabs/ego-lite
```

### 방법 C. 설치 스크립트 직접 실행

```bash
sh skills/ego-browser/scripts/install.sh
```

스크립트가 하는 일:
1. 이미 설치돼 있는지 확인 (있으면 다운로드 생략)
2. CPU 아키텍처(arm64/x64) 자동 판별 후 DMG 다운로드
3. `/Applications`에 설치 (실패 시 `~/Applications`)
4. **Gatekeeper 격리 속성 제거** (`xattr -dr com.apple.quarantine`)
5. 앱 실행

### 방법 D. DMG 수동 다운로드

README의 다운로드 배지 클릭 → DMG 실행

### 첫 실행 온보딩 (가장 중요)

앱이 켜지면 **"크롬 데이터를 가져올까요?"** 를 묻습니다.

> **반드시 "예"** 를 선택하세요.
> 로그인 세션 / 쿠키 / 북마크 / 확장 프로그램이 넘어옵니다.
> 이걸 건너뛰면 ego lite를 쓰는 의미가 거의 사라집니다.

온보딩이 끝나면 `ego-browser` 명령어가 **`~/.local/bin`** 에 등록됩니다.

### 설치 확인

```bash
command -v ego-browser
```

`command not found`가 나오면 PATH 문제:

```bash
export PATH="$HOME/.local/bin:$PATH"
command -v ego-browser
```

최종 동작 확인:

```bash
ego-browser nodejs <<'EOF'
console.log('ego-browser ready')
EOF
```

`ego-browser ready`가 출력되면 준비 완료.

---

## 5. 사용법

### 5-1. 사용자는 자연어로 말하면 끝

```
/ego-browser x.com 가서 @ego_agent 팔로우해줘
/ego-browser 지메일 들어가서 오늘 온 메일 제목만 뽑아줘
/ego-browser localhost:3000 열어서 회원가입 폼 테스트하고 버그 알려줘
```

### 5-2. 내부 동작 (에이전트가 만드는 코드)

```bash
ego-browser nodejs <<'EOF'
const task = await useOrCreateTaskSpace('example 페이지 조사')
await openOrReuseTab('https://example.com', { wait: true, timeout: 20 })
cliLog(await snapshotText())
EOF
```

heredoc 본문은 Node.js에서 실행되고, 헬퍼들이 전역으로 주입됩니다.

### 5-3. 주요 헬퍼

| 분류 | 함수 |
|---|---|
| 작업공간 | `useOrCreateTaskSpace`, `listTaskSpaces`, `claimTaskSpace`, `handOffTaskSpace`, `takeOverTaskSpace`, `completeTaskSpace` |
| 이동 | `listTabs`, `openOrReuseTab`, `gotoAndWait`, `switchTab`, `closeTab`, `pageInfo` |
| 관찰 | `snapshotText`, `captureScreenshot`, `drainEvents` |
| 마우스 | `click`, `doubleClick`, `hover`, `dragMouse`, `scroll`, `scrollBy`, `scrollToBottomUntil` |
| 키보드 | `typeText`, `fillInput`, `pressKey`, `dispatchKey` |
| 파일 | `uploadFile` |
| 대기 | `wait`, `waitForLoad`, `waitForElement`, `waitForNetworkIdle` |
| 네트워크 | `serverFetch`, `browserFetch` |
| 고급 | `js`, `cdp` |
| 출력 | `cliLog`, `help` |
| 녹화 | `page.screencast.start/stop` (ffmpeg 필요) |

### 5-4. 3가지 작업 전략

1. **Semantic (기본값)** — `snapshotText()`로 화면을 의미 트리로 읽고 `@N` ref나 `loc=...`로 조작
   - 일반 웹사이트(폼, 목록, 버튼, 테이블) 대부분
2. **Visual** — `captureScreenshot()` + 좌표 클릭 / 실제 키보드 입력
   - Google Docs/Sheets, Notion, Figma, 지도, 캔버스 에디터 등
   - 이런 앱은 DOM이 실제 편집 영역과 달라서, 큰 내용을 쓰기 전 **작은 테스트 입력(write probe)** 으로 검증 필요
3. **Direct DOM / CDP** — `js(...)` / `cdp(...)`로 브라우저 내부에서 직접 데이터 추출
   - 크롤링, 대량 파싱

### 5-5. 통제권 핸드오프

- 로그인·캡차 등 사람이 개입해야 하면 → 에이전트가 `handOffTaskSpace()` 호출 후 안내
- 사용자가 처리하고 **"계속"** 이라고 하면 → `takeOverTaskSpace()`로 재개
- 사용자가 GUI로 직접 통제권을 가져가면 → 에이전트 작업은 실패하며, **재시도하면 안 됨**
  (에이전트가 임의로 통제권을 뺏지 못하도록 설계된 안전장치)

### 5-6. 주의사항 (SKILL.md Caveats)

- 시간 단위는 **초**. 이름이 `~Ms`로 끝나는 파라미터만 밀리초
- `@N` ref는 **가장 최근 `snapshotText()` 호출에서만 유효** → 장기 참조는 `loc=...` 사용
- `js()`는 **문자열**을 받음. Playwright의 `page.evaluate(fn, args)`처럼 쓰면 안 됨
- `js()` 안의 정규식 백슬래시는 이중 이스케이프(`\\d`) 또는 `String.raw` 사용
- 작업이 끝나면 `completeTaskSpace(name, { keep })` 호출 (기본은 `keep: false`)
- 영상 녹화는 `ffmpeg` 필요 (`brew install ffmpeg`)
- 개인정보는 로컬에만 저장. 외부로 나가는 건 "크롬 마이그레이션 동의 여부"뿐

### 5-7. 문제 해결

| 증상 | 해결 |
|---|---|
| `command not found: ego-browser` | 앱 미설치 또는 PATH 문제 → `export PATH="$HOME/.local/bin:$PATH"` |
| "확인되지 않은 개발자" 경고 | 설치 스크립트가 quarantine 속성 제거 → 스크립트로 설치 |
| 로그인이 안 넘어옴 | 온보딩에서 크롬 마이그레이션을 거부한 경우 → 앱 설정에서 재시도 |
| `user is controlling` 에러 | 사용자가 통제 중. 재시도 금지, "계속" 확인 후 재개 |
| `pageInfo()`가 `w:0 / h:0` | 실제 탭으로 전환하거나 새로고침 후 재확인 |

---

## 6. 실행 환경 정리 (어디서 쓸 수 있나)

| 환경 | 가능 여부 |
|---|---|
| macOS + 로컬 터미널에서 `claude` 실행 | 가능 |
| macOS + Claude 데스크톱 앱 | 가능 |
| Windows / Linux | 불가 (로드맵) |
| 클라우드/원격 세션 (claude.ai/code 등) | **불가** — GUI 브라우저와 로컬 로그인 상태가 필요 |

핵심: **"내 브라우저와 내 로그인이 있는 그 컴퓨터"** 에서 에이전트를 실행해야 합니다.

---

## 7. 수익화 아이디어

### 전제

1. 만들려면 macOS 필요
2. `learnings`에 사이트가 2개뿐 → 선점 기회
3. 앱은 무료 + 클로즈드소스 → **브라우저 자체가 아니라 "위에 얹는 것"으로 수익화**

### Tier 1 — 즉시 수익 가능

**① 로그인 자동화 대행 서비스 (가장 현실적)**

기존 크롤링 업체가 가장 못 하는 "로그인 뒤 영역"을 공략.

- 타겟: 쇼핑몰 셀러(상품 일괄 등록, 경쟁사 가격 추적), 중소기업 사무직(ERP/그룹웨어 반복업무), 마케터(광고 대시보드 리포트 취합)
- 가격: 세팅 30~200만원 + 월 유지비
- 시작: 크몽/숨고 등록 → 첫 레퍼런스 확보
- 장점: **고객 PC에서 실행 → 서버비 0원**

**② learnings 팩 선점**

한국 사이트 공략집(네이버 스마트스토어, 쿠팡 윙, 잡코리아 등)을 제작.

- 오픈소스로 공개 → 기여자 인지도 → ①의 영업 자산 (추천)
- 유료 판매 (단, MIT라 복제 방어는 어려움)
- CitroLabs에 기여 → 채용/커미션 기회

### Tier 2 — 제품화

**③ 버티컬 SaaS** (한 업종만 집중)

| 업종 | 아이디어 |
|---|---|
| 부동산 | 매물 자동 수집 → 여러 플랫폼 동시 등록 |
| 채용 | 링크드인/원티드 인재 소싱 자동화 |
| 이커머스 | 경쟁사 가격·리뷰 추적 → 슬랙 알림 |
| 재무 | 공개 API 없는 금융사 거래내역 취합 |

**④ QA 자동화 서비스**

자연어 시나리오로 회귀 테스트 → 매일 실행 → 실패 시 알림. 월 20~50만원 구독.

### Tier 3 — 부수입 / 브랜딩

**⑤ 콘텐츠 선점** — 한국어 자료가 거의 없음. 유튜브/블로그로 검색 선점 후 ①의 유입 채널로 활용.

### 리스크

- **이용약관(ToS)**: 사이트별 자동화 금지 조항 존재. "고객이 자기 계정으로 자기 데이터를" 처리하는 구조가 가장 안전
- **플랫폼 리스크**: 초기 무료 제품 → 유료화/중단 가능성. 핵심 로직은 ego lite에 종속되지 않게 설계
- **macOS 전용**: 잠재 시장의 절반이 제외됨

### 추천 실행 순서

```
1단계: 내 반복업무 하나를 자동화 (실력 + 사례 확보)
2단계: 과정을 블로그/유튜브로 기록 (한국어 선점)
3단계: 크몽에 대행 서비스 등록 → 첫 고객 3명
4단계: 반복 요청을 SaaS로 제품화
```

1~3단계는 자본 없이 시작 가능.

---

## 8. PHP로 만들 수 있나?

### ego lite "자체"를 PHP로 → 불가능

| 구성 | 언어 |
|---|---|
| 브라우저 본체 | C++ (Chromium 커널 수정) |
| 조종 계층 | TypeScript / Node.js |

브라우저 엔진 구현은 PHP의 영역이 아님.

### ego lite "조종"을 PHP로 → 가능

`ego-browser`는 stdin으로 JS를 받는 **CLI 명령어**이므로, PHP에서 프로세스로 호출하면 됩니다.

```php
<?php
function egoRun(string $script): string
{
    $proc = proc_open(
        ['ego-browser', 'nodejs'],
        [0 => ['pipe', 'r'], 1 => ['pipe', 'w'], 2 => ['pipe', 'w']],
        $pipes
    );

    if (!is_resource($proc)) {
        throw new RuntimeException('ego-browser 실행 실패');
    }

    fwrite($pipes[0], $script);
    fclose($pipes[0]);

    $out = stream_get_contents($pipes[1]);
    $err = stream_get_contents($pipes[2]);
    fclose($pipes[1]);
    fclose($pipes[2]);

    if (proc_close($proc) !== 0) {
        throw new RuntimeException("ego-browser 오류: $err");
    }

    return $out;
}

$result = egoRun(<<<'JS'
const task = await useOrCreateTaskSpace('상품 가격 수집')
await openOrReuseTab('https://example.com/product/123', { wait: true })

const data = await js(String.raw`(() => ({
  title: document.querySelector('h1')?.innerText,
  price: document.querySelector('.price')?.innerText,
}))()`)

cliLog(JSON.stringify(data))
JS);

$product = json_decode($result, true);
```

### 권장 아키텍처

```
┌──────────────────────────────┐
│  Laravel (PHP)               │  ← 비즈니스 로직 전부
│  • 고객 관리 / 인증           │
│  • 자동화 작업 등록 UI        │
│  • 스케줄러 (Task Scheduling) │
│  • 결과 DB 저장 + 리포트      │
│  • 결제, 슬랙/메일 알림       │
└──────────┬───────────────────┘
           │ proc_open / Queue Job
           ▼
┌──────────────────────────────┐
│  ego-browser (Node)          │  ← 브라우저 조작 전담
│  → ego lite 앱               │
└──────────────────────────────┘
```

- PHP = 머리(비즈니스 로직)
- ego-browser = 손(브라우저 조작)

### ego lite 없이 순수 PHP로만 하려면

| 라이브러리 | 특징 |
|---|---|
| `chrome-php/chrome` | PHP가 CDP로 크롬 직접 조종. ego-browser와 원리 동일 (가장 유사) |
| Symfony Panther | Laravel/Symfony 궁합 좋음, WebDriver 기반 |
| `php-webdriver/webdriver` | Selenium 표준, 자료 풍부 |
| Guzzle + DomCrawler | JS 실행 없는 단순 페이지 전용, 가장 가벼움 |

```bash
composer require chrome-php/chrome
```

### 제약

`ego-browser`를 쓰려면 **PHP가 실행되는 그 머신에 ego lite 앱이 떠 있어야** 합니다.

- 일반 리눅스 웹호스팅 배포 → 불가
- 고객 맥북에 설치되는 로컬 앱 형태 → 가능
- 맥미니를 전용 서버로 운용 → 가능

이 제약은 오히려 **서버비 0원 + 고객 데이터가 외부로 나가지 않음**이라는 영업 포인트가 됩니다.

---

## 9. 요약

| 질문 | 답 |
|---|---|
| 이게 뭐야? | 사람과 AI가 함께 쓰는, 로그인 상태를 공유하는 브라우저 |
| 제일 큰 강점? | **로그인이 필요한 웹 자동화** + 병렬 Space + 낮은 토큰 비용 |
| 어디서 써? | macOS + 로컬에서 실행하는 에이전트 CLI |
| 언제 써? | 반복 웹 업무, 인증 필요한 데이터 수집, 웹앱 QA, 스크린샷 자동화 |
| 수익화? | 로그인 자동화 대행 → 버티컬 SaaS 순으로 확장 |
| PHP로 가능? | 브라우저 구현은 불가, **조종·제품화는 가능** (Laravel + proc_open) |
