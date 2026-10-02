# AutoGPT 전수조사 분석 & 수익화 가이드

> 이 문서는 AutoGPT 저장소를 폴더 단위로 전수조사한 결과와,
> 그로부터 도출한 활용·수익화 전략을 정리한 종합 가이드입니다.
>
> - 작성일: 2026-10-02
> - 작성: Claude Code (Karina persona) 와의 대화 정리
> - 원본 저장소: <https://github.com/Significant-Gravitas/AutoGPT>
> - 작업 저장소(포크): <https://github.com/bmshin94/AutoGPT>
> - 작업 브랜치: `claude/elegant-cerf-nvu830`

---

## 목차

1. [프로젝트 정체 — 한눈에 보기](#1-프로젝트-정체--한눈에-보기)
2. [폴더 구조 전수조사 결과](#2-폴더-구조-전수조사-결과)
3. [두 개의 AutoGPT — Classic vs Platform](#3-두-개의-autogpt--classic-vs-platform)
4. [핵심 개념: 블록(Block) 시스템](#4-핵심-개념-블록block-시스템)
5. [시스템 아키텍처](#5-시스템-아키텍처)
6. [설치 및 사용법](#6-설치-및-사용법)
7. [플러그인? 스킬? MCP? — 정체 규명](#7-플러그인-스킬-mcp--정체-규명)
8. [API 토큰 / 인증 정리](#8-api-토큰--인증-정리)
9. [AI 에이전트 구축에 주는 도움](#9-ai-에이전트-구축에-주는-도움)
10. [React / PHP 로 만들 수 있는가](#10-react--php-로-만들-수-있는가)
11. [유튜브 강의 제작 가능성 & 커리큘럼](#11-유튜브-강의-제작-가능성--커리큘럼)
12. [수익화 아이디어 9선](#12-수익화-아이디어-9선)
13. [90일 실행 로드맵](#13-90일-실행-로드맵)
14. [라이선스 주의사항 (중요)](#14-라이선스-주의사항-중요)
15. [리스크 및 대응](#15-리스크-및-대응)
16. [참고 링크](#16-참고-링크)

---

## 1. 프로젝트 정체 — 한눈에 보기

| 항목 | 내용 |
|---|---|
| **한 줄 정의** | AI 에이전트를 블록(노드)으로 조립해 만들고, 실행·예약·판매까지 하는 오픈소스 플랫폼 |
| **GitHub 스타** | 약 185,000+ (AI 자동화 분야 세계 1위) |
| **코드 규모** | Python 1,868 파일 / TypeScript·TSX 2,610 파일 |
| **블록 수** | `backend/blocks` 하위 디렉터리 108개, 블록 관련 `.py` 403개 |
| **외부 연동** | Provider 48종 (`backend/integrations/providers.py` 기준) |
| **기술 스택** | FastAPI · Next.js 15 · React 19 · TypeScript · PostgreSQL(Prisma) · Redis · RabbitMQ · FalkorDB · Docker |
| **라이선스** | `autogpt_platform/` → PolyForm Shield 1.0 / `classic/` 및 그 외 → MIT |

### 핵심 메시지

> **"AI에게 일을 시키는 과정을, 레고 조립하듯 눈으로 보면서 설계하는 도구"**

---

## 2. 폴더 구조 전수조사 결과

```
AutoGPT/
├── autogpt_platform/      65MB   ← 현역 핵심 (실제 제품)
│   ├── backend/                   FastAPI 백엔드
│   │   ├── schema.prisma          DB 스키마 (모델 100개 이상)
│   │   └── backend/
│   │       ├── blocks/            블록 108개 디렉터리 / 403개 .py
│   │       ├── executor/          실행 엔진, 과금, 스케줄러, 시뮬레이터
│   │       ├── integrations/      OAuth · 자격증명 보관 · 웹훅
│   │       ├── api/               REST · WebSocket · 외부 공개 API
│   │       ├── data/              DB 로직 (크레딧/결제/실행기록/조직)
│   │       ├── copilot/           대화형 에이전트 생성(AutoPilot)
│   │       └── sdk/               커스텀 블록 개발 SDK
│   ├── frontend/                  Next.js 15 + React 19 + Tailwind
│   │   └── src/app/(platform)/    build · library · marketplace · copilot ...
│   ├── db/                        Postgres + Prisma 마이그레이션
│   ├── autogpt_libs/              공용 Python 라이브러리
│   ├── docker-compose.yml         개발 스택 진입점
│   ├── docker-compose.platform.yml 15개 서비스 정의
│   ├── single-container/          올인원 컨테이너 빌드
│   └── installer/                 setup-autogpt.sh / .bat
│
├── classic/               4.2MB  ← 2023년 원조 (지원 종료, 교육용)
│   ├── original_autogpt/          초기 자율 에이전트 구현
│   ├── forge/                     에이전트 제작 프레임워크
│   └── direct_benchmark/          벤치마크 하네스
│
├── docs/                  132MB  공식 문서 (대부분 이미지 자산)
├── AGENTS.md                      코딩 에이전트용 기여 가이드
├── CLAUDE.md                      프로젝트 가이드
└── README.md
```

### 블록 카테고리 샘플 (`backend/blocks/`)

| 분류 | 파일/디렉터리 예시 |
|---|---|
| AI 생성 | `llm.py`, `ai_image_generator_block.py`, `ai_music_generator.py`, `ai_shortform_video_block.py`, `ai_image_customizer.py` |
| 코딩 에이전트 | `claude_code.py`, `codex.py`, `code_executor.py`, `code_extraction_block.py` |
| 검색·수집 | `exa/`, `jina/`, `firecrawl/`, `perplexity.py`, `reddit.py`, `rss.py`, `google/`, `screenshotone.py` |
| 업무 도구 | `github/`, `notion/`, `linear/`, `hubspot/`, `airtable/`, `discord/`, `medium.py`, `email_block.py` |
| 로직·제어 | `branching.py`, `iteration.py`, `maths.py`, `json_blocks.py`, `data_manipulation.py`, `sampling.py` |
| 안전장치 | `human_in_the_loop.py` (사람 승인 후 진행) |
| 기억·RAG | `pinecone.py`, `mem0.py`, `persistence.py` |
| 확장 | `mcp/` (MCP 서버 연결), `http.py`, `generic_webhook/` |

### 연동 Provider 48종

```
AIML_API, ANTHROPIC, APOLLO, CODEX, COMPASS, DATABASE, DISCORD, D_ID, E2B,
ELEVENLABS, FAL, GITHUB, GOOGLE, GOOGLE_MAPS, GROQ, HTTP, HUBSPOT, ENRICHLAYER,
IDEOGRAM, JINA, LLAMA_API, MCP, MEDIUM, MEM0, NOTION, NVIDIA, OLLAMA, OPENAI,
OPENWEATHERMAP, OPEN_ROUTER, PINECONE, REDDIT, REPLICATE, REVID, SCREENSHOTONE,
SLACK, SLANT3D, SMARTLEAD, SMTP, STRIPE, STRIPE_LINK, TELEGRAM, TWITTER, TODOIST,
UNREAL_SPEECH, V0, WEBSHARE_PROXY, ZEROBOUNCE
```

> 한국 서비스(네이버·카카오·토스·쿠팡 등)는 **하나도 없음** → 기회 영역.

---

## 3. 두 개의 AutoGPT — Classic vs Platform

| | **Classic** (`classic/`) | **Platform** (`autogpt_platform/`) |
|---|---|---|
| 시기 | 2023년 | 현재 |
| 상태 | `This project is unsupported` (지원 종료) | 활발히 개발 중 |
| 방식 | AI가 스스로 계획·실행 반복 (자율 루프) | 사람이 흐름 설계 + AI는 각 단계 수행 |
| 예측성 | 실행마다 결과가 달라짐 | 동일한 순서로 재현 가능 |
| 비용 | 통제 어려움 | 블록 단위 크레딧 과금 + 사전 견적 |
| 디버깅 | 어려움 | 노드별 실행 로그 추적 |
| 재사용 | 어려움 | 저장 / 공유 / 마켓 판매 |
| 라이선스 | MIT | PolyForm Shield 1.0 |

### 얻을 수 있는 교훈

> **AI는 "운전기사"가 아니라 "엔진"으로 쓸 때 실무에서 작동한다.**
>
> Classic은 AI에게 운전대를 통째로 맡겼다가 비용·예측성 문제로 실패했고,
> Platform은 경로(그래프)를 사람이 그리고 각 구간의 추론만 AI에게 맡겨 성공했다.

---

## 4. 핵심 개념: 블록(Block) 시스템

### 블록의 구조 (`backend/blocks/basic.py` 실제 코드)

```python
class FileStoreBlock(Block):
    class Input(BlockSchemaInput):      # 입력 스키마
        file_in: MediaFileType = SchemaField(description="저장할 파일")

    class Output(BlockSchemaOutput):    # 출력 스키마
        file_out: MediaFileType = SchemaField(description="저장된 파일 참조")

    def __init__(self):
        super().__init__(
            id="cbb50872-625b-42f0-8203-a2ae78242d8a",   # 고유 UUID
            description="URL/데이터URI/로컬경로의 파일을 받아 저장",
            categories={BlockCategory.BASIC, BlockCategory.MULTIMEDIA},
            input_schema=FileStoreBlock.Input,
            output_schema=FileStoreBlock.Output,
            static_output=True,
        )

    async def run(self, input_data: Input, *, execution_context, **kwargs) -> BlockOutput:
        yield "file_out", await store_media_file(...)
```

- **블록(Block)** = 기능 1개짜리 부품. 입력 스키마 / 출력 스키마 / `run()` 으로 구성
- **에이전트(Agent)** = 블록을 선으로 이어 만든 그래프 (DB의 `AgentGraph`, `AgentNode`, `AgentNodeLink`)
- **실행(Run)** = 그래프 1회 수행 (`AgentGraphExecution`, `AgentNodeExecution`)

### 조립 예시 1 — 아침 뉴스 브리핑

```
스케줄러(매일 08:00) → RSS 수집 → 반복 → LLM 요약 → 이메일 발송
```

### 조립 예시 2 — 숏폼 영상 자동 생성

```
주제 입력 → LLM(대본) → 이미지 생성 → ElevenLabs(음성) → 숏폼 영상 합성 → 업로드
```

### 반드시 알아야 할 3가지

| 개념 | 파일 | 의미 |
|---|---|---|
| **Human in the Loop** | `blocks/human_in_the_loop.py` | 중요한 동작 전 사람 승인을 받는 안전장치 |
| **비용 추적 / 사전 견적** | `executor/cost_tracking.py`, `data/block_preflight_estimates.py` | 블록별 단가 산정 + 실행 전 예상 비용 계산 |
| **자격증명 보관** | `integrations/credentials_store.py`, `integrations/oauth/` | OAuth 토큰을 암호화 보관하고 블록이 안전하게 사용 |

---

## 5. 시스템 아키텍처

`docker-compose.platform.yml` 기준 서비스 구성:

```
[ frontend ]                       Next.js UI (빌더 / 라이브러리 / 마켓)
      |
[ rest_server ]  [ websocket_server ]      API / 실시간 진행상황
      |
[ executor ] [ copilot_executor ]          에이전트 실행 워커
[ scheduler_server ]                       예약 실행
[ notification_server ]                    알림
[ database_manager ]                       DB 중앙 관리
[ platform_linking_manager ]               플랫폼 연동
[ migrate ]                                스키마 마이그레이션
      |
[ PostgreSQL(Supabase) ] [ Redis x3 + redis-init ] [ RabbitMQ ] [ FalkorDB ] [ ClamAV ]
```

- **Redis 3노드 + init** → 캐시/락 클러스터 (`executor/cluster_lock.py`)
- **RabbitMQ** → 실행 작업 큐 (`data/rabbitmq.py`)
- **FalkorDB** → 그래프 DB
- **ClamAV** → 업로드 파일 바이러스 검사

> 토이 프로젝트가 아니라 **엔터프라이즈급 분산 시스템** 설계.

### DB 스키마 하이라이트 (`backend/schema.prisma`)

| 영역 | 모델 |
|---|---|
| 에이전트 | `AgentGraph`, `AgentNode`, `AgentNodeLink`, `AgentBlock`, `AgentPreset` |
| 실행 | `AgentGraphExecution`, `AgentNodeExecution`, `AgentNodeExecutionInputOutput`, `PendingHumanReview` |
| 과금 | `CreditTransaction`, `SubscriptionTier`, `AgentGraphExecution` 비용 필드 |
| 마켓 | `StoreListing`, `StoreListingVersion`, `StoreListingReview`, `Profile` |
| 전문가 | `Expert`, `ExpertWorkflow`, `ExpertCredential`, `ExpertPod`, `ExpertPauseEvent` |
| 조직 | `Team`, `TeamJoinPolicy`, `ResourceVisibility`, `CredentialScope` |
| 외부 API | `APIKey`, `APIKeyPermission`, `APIKeyStatus` |
| 알림 | `NotificationEvent`, `PushSubscription`, `UserBriefing`, `AlertCondition` |

---

## 6. 설치 및 사용법

### 방법 A — 호스팅판 (가장 빠름, 유료)

```
https://platform.agpt.co/signup
```

- 설치 불필요, API 키 불필요, 모델 수백 종 즉시 사용
- 사용량 기반 과금
- **처음 체험할 때 권장**

### 방법 B — 자동 설치 스크립트 (무료)

macOS / Linux:

```bash
curl -fsSL https://setup.agpt.co/install.sh -o install.sh && bash install.sh
```

Windows PowerShell:

```powershell
powershell -c "iwr https://setup.agpt.co/install.bat -o install.bat; ./install.bat"
```

### 방법 C — 소스에서 직접 (개발자용)

```bash
git clone https://github.com/bmshin94/AutoGPT.git
cd AutoGPT/autogpt_platform

cp .env.default .env
# POSTGRES_PASSWORD 를 반드시 변경할 것 (기본값: your-super-secret-and-long-postgres-password)

docker compose up -d --build

# 프론트엔드: http://localhost:3000
# API 문서  : http://localhost:8006/docs
```

개별 개발 모드:

```bash
# 백엔드
cd autogpt_platform/backend
poetry install
poetry run prisma migrate dev
poetry run serve
poetry run test        # pytest (docker postgres + prisma)
poetry run format      # 포맷팅

# 프론트엔드
cd autogpt_platform/frontend
pnpm install
pnpm dev
pnpm test:unit         # Vitest + RTL + MSW
pnpm test              # Playwright E2E
pnpm format
pnpm generate:api      # API 훅 재생성 (Orval)
```

### 최소 사양

| 항목 | 권장 |
|---|---|
| RAM | 16GB 이상 (8GB는 매우 느림) |
| 디스크 | 20GB 이상 |
| 필수 | Docker, Node 24.x (`.nvmrc`), Python 3.12, Poetry, pnpm |

### 학습 순서 권장

1. 가입 / 로그인
2. **Marketplace** 에서 기존 에이전트 담아 실행해 보기
3. **Library** 에서 실행 결과 확인
4. **Build** 에서 블록 하나 바꿔 보기
5. 처음부터 직접 설계
6. **Scheduler** 로 자동 실행 등록

---

## 7. 플러그인? 스킬? MCP? — 정체 규명

### 결론: **셋 다 아님. 독립 "플랫폼"이다.**

| 구분 | 플러그인 | 스킬 | MCP | **AutoGPT** |
|---|---|---|---|---|
| 정체 | 앱 확장 부품 | AI 능력 설명서 | 연결 **규격(프로토콜)** | **독립 제품** |
| 단독 실행 | 불가 | 불가 | 불가 | **가능** |
| 자체 DB | 없음 | 없음 | 없음 | **PostgreSQL** |
| 자체 UI | 없음 | 없음 | 없음 | **Next.js 프론트엔드** |
| 비유 | 크롬 확장 | 요리 레시피 | USB-C 규격 | **스마트폰 본체** |

### MCP 와의 관계: AutoGPT 는 **MCP 클라이언트(소비자)**

실제 구현 확인:

```
backend/blocks/mcp/
├── block.py     # "어떤 MCP 서버에도 연결해 툴을 발견·실행하는 동적 블록"
├── client.py    # "MCP Streamable HTTP 전송 구현, JSON-RPC 2.0 over HTTP POST"
├── oauth.py     # MCP 서버 OAuth
├── helpers.py   # URL 정규화, 자격증명 자동 조회
└── _config.py   # ProviderBuilder("mcp")

backend/api/features/mcp/routes.py   # MCP 관련 API
```

> 즉 **"MCP를 품은 플랫폼"**. MCP 서버를 블록처럼 꽂아 쓸 수 있다.

### Claude Code 와의 비교

| | Claude Code | AutoGPT |
|---|---|---|
| 무대 | 터미널 / IDE | 웹 브라우저 |
| 용도 | 코딩 | 업무 자동화 |
| 실행 | 대화형 | 그래프 + 스케줄 + 트리거 |
| 비전공자 | 어려움 | 가능 |

경쟁이 아니라 보완 관계. (AutoGPT 안에 `blocks/claude_code.py` 블록도 존재)

---

## 8. API 토큰 / 인증 정리

### 케이스 1 — 호스팅판 사용

- **모델 API 키 불필요.** AutoGPT 가 대행하고 사용자는 크레딧을 결제.

### 케이스 2 — 셀프호스팅

- README: *"Bring your own API keys"* → **직접 준비 필요**

| 종류 | 예시 | 필수 여부 |
|---|---|---|
| AI 모델 | OpenAI, Anthropic, Groq, OpenRouter, Llama API | 최소 1개 |
| 무료 대안 | **Ollama** (로컬 모델) | 키 없이 가능 |
| 업무 연동 | GitHub, Notion, Slack, Discord, HubSpot | 사용 시 |
| 검색 | Exa, Jina, Firecrawl, Perplexity | 사용 시 |
| 미디어 | ElevenLabs, Replicate, Ideogram, Fal, D-ID | 사용 시 |

> `OLLAMA` 가 Provider 목록에 포함되어 있어, 로컬 모델만 쓰면 **모델 API 비용 0원** 구성이 가능.

### 케이스 3 — 외부 프로그램에서 AutoGPT 호출

AutoGPT 가 **자체 API 키를 발급**한다 (`schema.prisma`):

```prisma
model APIKey {
  head        String             // 식별용 앞부분
  tail        String
  hash        String  @unique    // 원본 미보관, 해시 저장
  salt        String?
  status      APIKeyStatus       // ACTIVE / REVOKED
  permissions APIKeyPermission[] // 권한 세분화
  lastUsedAt  DateTime?
  revokedAt   DateTime?
  organizationId String?
  teamId         String?
}
```

> 해시 + 솔트 저장, 권한 분리, 폐기 추적까지 — 보안 설계가 견고하다.

공개 외부 API 엔드포인트 (`backend/api/external/v1/routes.py`):

| 메서드 | 경로 | 설명 |
|---|---|---|
| GET | `/user` | 사용자 정보 |
| GET | `/blocks` | 블록 목록 |
| POST | `/blocks/{id}/execute` | 블록 단건 실행 |
| POST | `/graphs` | 에이전트(그래프) 생성 |
| POST | `/graphs/{id}/execute` | **에이전트 실행** |
| GET | `/executions/{id}` | **실행 결과 조회** |
| GET | `/store/agents` | 마켓 에이전트 목록 |
| GET | `/store/creators` | 제작자 목록 |

→ 이 API 만으로 **React / PHP / 모바일 어디서든** AutoGPT를 백엔드 엔진으로 사용 가능.

---

## 9. AI 에이전트 구축에 주는 도움

### 레벨 1 — 그냥 사용 (가치 ★★★)

- 아이디어를 1시간 내 PoC 로 검증
- 코딩 없이 데모 제작 → 고객 시연

### 레벨 2 — 커스텀 블록 개발 (가치 ★★★★)

`backend/sdk/__init__.py` 가 블록 개발에 필요한 모든 것을 재수출한다:

```python
from backend.sdk import *   # Block, BlockSchemaInput/Output, SchemaField,
                            # CredentialsField, OAuth2Credentials, ProviderBuilder,
                            # cost, AutoRegistry, Requests ...

class MyBlock(Block):
    class Input(BlockSchemaInput):
        query: str = SchemaField(description="검색어")

    class Output(BlockSchemaOutput):
        result: str = SchemaField(description="결과")

    def __init__(self):
        super().__init__(
            id="<고유 UUID>",
            description="커스텀 블록",
            categories={BlockCategory.BASIC},
            input_schema=MyBlock.Input,
            output_schema=MyBlock.Output,
        )

    async def run(self, input_data: Input, **kwargs) -> BlockOutput:
        yield "result", f"처리됨: {input_data.query}"
```

Provider 등록도 선언적으로 가능:

```python
from backend.sdk import ProviderBuilder

mcp = ProviderBuilder("mcp").with_description("Model Context Protocol servers").build()
```

### 레벨 3 — 설계를 학습 (가치 ★★★★★, 가장 중요)

| 배울 것 | 위치 | 왜 중요한가 |
|---|---|---|
| 블록 추상화 / 스키마 | `blocks/_base.py` | 타입 안전한 입출력 계약 |
| 비용 추적 | `executor/cost_tracking.py`, `data/block_cost_config.py` | AI 서비스의 생존 조건 |
| 사전 비용 견적 | `data/block_preflight_estimates.py` | 실행 전 가격 예측 |
| 자격증명 관리 | `integrations/credentials_store.py`, `creds_manager.py` | OAuth 토큰 암호화·갱신 |
| 사람 개입 | `blocks/human_in_the_loop.py` | AI 오작동 방지 |
| 실행 엔진 / 재시도 | `executor/manager.py` | LLM 실패는 상수 |
| 분산 락 | `executor/cluster_lock.py` | 중복 실행 방지 |
| 실시간 스트리밍 | `api/ws_api.py`, `data/event_bus.py` | 진행상황 UX |
| 멀티테넌시 | `data/tenancy.py`, `data/org_credit.py` | B2B 필수 |
| 시뮬레이터 | `executor/simulator.py` | 실행 전 검증 |

> 다른 프레임워크(LangChain 등)로 만들더라도, **실서비스급 에이전트 시스템의 설계 레퍼런스**로서 가치가 가장 크다.

### 과할 수 있는 경우

- 단순 챗봇 1개 → 가벼운 프레임워크가 적합
- 초저지연 요구 → 블록 실행 오버헤드 존재
- 서버 RAM 8GB 이하 → 셀프호스팅 부담

---

## 10. React / PHP 로 만들 수 있는가

### (1) React 로 쓸 수 있는가 → **이미 React 다**

```
frontend/  =  Next.js 15 + React 19 + TypeScript + Tailwind
```

- 라우트: `src/app/(platform)/` → `build`, `library`, `marketplace`, `copilot`, `settings`, `team`, `admin` ...
- 테스트: Vitest + RTL + MSW (주력), Playwright (E2E), Storybook (디자인 시스템)
- API 훅 자동 생성: `pnpm generate:api` (Orval) → `@/app/api/__generated__/endpoints/`
- 코딩 규칙은 `frontend/CONTRIBUTING.md` 및 루트 `AGENTS.md` 참고

→ React 가능자라면 **UI 커스터마이징이 즉시 가능**.

### (2) PHP 에서 연동할 수 있는가 → **가능**

```php
<?php
$ch = curl_init("https://YOUR_HOST/api/v1/graphs/{$graphId}/execute");
curl_setopt_array($ch, [
    CURLOPT_POST           => true,
    CURLOPT_HTTPHEADER     => [
        "Authorization: Bearer {$apiKey}",   // AutoGPT 발급 API 키
        "Content-Type: application/json",
    ],
    CURLOPT_POSTFIELDS     => json_encode(["inputs" => ["topic" => "AI 뉴스"]]),
    CURLOPT_RETURNTRANSFER => true,
]);
$result = json_decode(curl_exec($ch), true);
curl_close($ch);
```

### (3) PHP 로 AutoGPT 같은 것을 새로 만들 수 있는가 → **비권장**

| 항목 | Python | PHP |
|---|---|---|
| AI SDK 생태계 | 전부 존재 | 거의 없음 |
| 비동기 | 네이티브 `async` | Swoole 등 별도 필요 |
| 장시간 워커 | 자연스러움 | 요청-응답 모델과 불일치 |
| 벡터 DB | 전부 지원 | 빈약 |

### 권장 아키텍처

```
[ 화면 ]  React/Next.js  또는  PHP(Laravel / WordPress)
             |  REST API + API Key
[ 엔진 ]  AutoGPT (Python)  ← 그대로 사용
```

> 엔진을 재발명하지 말고, **자신만의 껍데기(UI/도메인)** 를 만드는 전략이 효율적.

---

## 11. 유튜브 강의 제작 가능성 & 커리큘럼

### 법적 검토

| 항목 | 가능 여부 |
|---|---|
| 코드 리뷰 / 화면 녹화 | 가능 (공개 오픈소스) |
| 강의 수익 창출 | 가능 (교육은 경쟁 호스팅 서비스가 아님) |
| 로고·스크린샷 사용 | 출처 표기 시 가능 |
| **금지** | 코드를 복제해 **경쟁 호스팅 서비스로 판매** (PolyForm Shield 위반) |

### 왜 소재로 좋은가

- 185,000+ 스타 → 검색 수요 확보
- 한국어 심화 자료가 매우 적음 → 블루오션
- 업데이트가 잦아 콘텐츠 지속 생산 가능
- "AI 자동화" 는 현재 최상위 관심 키워드

### 12편 커리큘럼 제안

**시즌 1 — 입문 (유입)**

| # | 제목 | 길이 |
|---|---|---|
| 1 | AutoGPT 2026 총정리 | 10분 |
| 2 | 코딩 없이 AI 직원 만들기 | 15분 |
| 3 | 매일 아침 뉴스 요약 에이전트 | 20분 |
| 4 | 마켓플레이스 추천 에이전트 TOP 10 | 12분 |

**시즌 2 — 실전 (구독)**

| # | 제목 | 길이 |
|---|---|---|
| 5 | 내 PC에 AutoGPT 설치 (Docker) | 25분 |
| 6 | Ollama 연결로 모델 API 비용 0원 | 20분 |
| 7 | 노션 + 슬랙 + 지메일 3단 자동화 | 30분 |
| 8 | 숏폼 영상 자동 생성 파이프라인 | 25분 |

**시즌 3 — 개발자 (전문성)**

| # | 제목 | 길이 |
|---|---|---|
| 9 | 커스텀 블록 만들기 (Python SDK) | 30분 |
| 10 | React 에서 AutoGPT API 호출 | 25분 |
| 11 | PHP / 워드프레스에 AI 심기 | 25분 |
| 12 | 403개 블록 코드로 배우는 AI 설계 | 40분 |

### 수익 경로

```
유튜브 광고 → 전자책 / 온라인 강의 → 구축 외주 문의 → 기업 컨설팅·출강
```

> 유튜브는 신뢰를 쌓는 **광고판**, 실제 수익은 후단에서 발생.

---

## 12. 수익화 아이디어 9선

> 전제: `autogpt_platform/` 은 **PolyForm Shield 1.0**.
> 금지 = 코드를 그대로 **경쟁 호스팅 서비스**로 판매.
> 허용 = 교육 · 컨설팅 · 구축대행 · 결과물 판매 · 사내 사용 · 에이전트 판매 · API 활용 제품.

### TIER 1 — 즉시 시작 가능

#### 1. 에이전트 구축 대행 (외주) — **1순위 추천**

| 항목 | 내용 |
|---|---|
| 난이도 | ★★☆☆☆ |
| 초기비용 | 0원 |
| 수익 시작 | 2~4주 |
| 단가 | 건당 50만 ~ 500만원 |

| 상품 | 가격 | 기간 |
|---|---|---|
| 기본 자동화 1개 | 50~150만원 | 3일 |
| 업무 패키지 3개 | 300~500만원 | 2주 |
| **월 유지보수** | 20~50만원/월 | 지속 |

> 핵심은 **월 유지보수 계약**. 10곳 확보 시 월 300만원 고정 수입.

타겟: 쇼핑몰(상품설명 생성), 부동산(매물 분석), 병원(리뷰 관리), 학원(상담 응대)

#### 2. 마켓플레이스 에이전트 판매

- 난이도 ★★☆☆☆ / 초기비용 0원 / 패시브 인컴
- 판매 인프라가 이미 구현됨: `StoreListing`, `StoreListingVersion`, `StoreListingReview`, `Profile`, `CreditTransaction`
- 유망 아이템: **한국 특화** (네이버 블로그 자동작성, 쿠팡 리뷰 분석, 지역 시세 조사), 업종별 템플릿

#### 3. 유튜브 + 전자책 + 강의

- 난이도 ★★☆☆☆ / 초기비용 0원 / 월 100~1000만원 (성장형)
- 경로: 유튜브(무료) → 전자책 2~5만원 → 온라인 강의 10~30만원 → 1:1 컨설팅 50만원/회 → 기업 출강 300만원/회

### TIER 2 — 3~6개월 투자형

#### 4. 버티컬 SaaS — **최고 추천 (확장성)**

```
[ 업종 전용 UI (React / PHP) ]   ← 내가 만드는 제품
            |  REST API
[ AutoGPT 엔진 ]                 ← 사용 (재판매 아님 → 라이선스 안전)
```

| 서비스 | 기능 | 가격 |
|---|---|---|
| 부동산 AI | 매물 설명·시세분석·블로그 | 월 9.9만원 |
| 쇼핑몰 AI | 상품설명·리뷰분석·경쟁사 모니터링 | 월 14.9만원 |
| 병원 AI | 리뷰 응대·블로그·예약 상담 | 월 19.9만원 |
| 법무 AI | 판례 검색·서면 초안 | 월 29.9만원 |
| 요식업 AI | SNS 운영·리뷰 관리 | 월 7.9만원 |

> 고객 100명 × 월 10만원 = **월 1,000만원 MRR**

#### 5. 워드프레스 플러그인 — **숨은 기회**

- 전 세계 웹사이트의 약 40%가 워드프레스, 그러나 PHP 특성상 AI 연동이 어려움
- 구조: `[WP 플러그인(PHP)] ←→ [AutoGPT API]`
- 기능: 글 자동작성 / 댓글 응대 / SEO 최적화 / 번역 / 상품설명 생성
- 가격: 무료(월 10회) → $19/월 → 에이전시 $99/월
- CodeCanyon 등 글로벌 마켓 등록 가능

#### 6. 커스텀 블록 판매

- Provider 48종에 **한국 서비스 전무** (네이버, 카카오, 토스, 쿠팡, 배민, 당근, 알리고, 아임포트 등)
- "한국형 블록팩" 제작 → 오픈소스 PR 로 인지도 확보 → 유료 지원·커스터마이징으로 수익
- 블록당 5~50만원

### TIER 3 — 장기 승부

#### 7. AI 자동화 교육 아카데미

- 기업 출강 300~1000만원/건, 온라인 과정 월 500만원+
- 정부지원 사업 연계 (K-디지털, 내일배움카드 등)
- 수료증 / 커뮤니티 운영

#### 8. 에이전트 운영 대행 (BPO)

- "AI를 파는 것이 아니라 **AI로 만든 결과물**을 판다"
- 예: 블로그 글 월 100개 = 200만원, 원가(AI 비용) 5만원 수준 → 고마진
- 월 100~500만원/고객

#### 9. 사내 도입 → 성과 → 커리어

- 셀프호스팅 무료 구축 → 팀 업무 자동화
- 정량 성과("월 200시간 절감") → 성과급 / 승진 / 이직 시 몸값 상승

### 종합 비교표

| # | 아이디어 | 난이도 | 초기비용 | 수익 시작 | 수익 규모 | 라이선스 |
|---|---|---|---|---|---|---|
| 1 | 구축 대행 | ★★☆ | 0원 | 2~4주 | 월 300~1000만 | 안전 |
| 2 | 에이전트 판매 | ★★☆ | 0원 | 1~2개월 | 월 50~300만 | 안전 |
| 3 | 유튜브/강의 | ★★☆ | 0원 | 3~6개월 | 월 100~1000만 | 안전 |
| 4 | **버티컬 SaaS** | ★★★★ | 30~100만 | 4~6개월 | **월 1000만+** | 안전 |
| 5 | WP 플러그인 | ★★★ | 낮음 | 3~4개월 | 월 300~2000만 | 안전 |
| 6 | 블록 판매 | ★★★ | 0원 | 2~3개월 | 월 50~200만 | 안전 |
| 7 | 교육 아카데미 | ★★★★ | 중간 | 6개월+ | 월 500만+ | 안전 |
| 8 | 운영 대행(BPO) | ★★★ | 낮음 | 2~3개월 | 월 100~500만 | 안전 |
| 9 | 사내 도입 | ★★☆ | 0원 | 1~2개월 | 간접 | 안전 |

### 추천 전략

> **#1 구축 대행으로 현금흐름 확보 → #4 버티컬 SaaS 로 확장**
>
> 외주를 하면서 (1) 수익을 내고 (2) 고객의 실제 문제를 학습하고 (3) 그것을 제품화하면 실패 확률이 크게 낮아진다.
> 외주는 시간을 파는 일(한계 있음), SaaS 는 자산을 쌓는 일(확장 가능).

---

## 13. 90일 실행 로드맵

```
1~2주차   호스팅판 가입 → 에이전트 5개 직접 제작해 보기
3~4주차   셀프호스팅 설치 완료 → 유튜브 1편 업로드
5~8주차   지인 회사 1곳 무료 구축 → 후기/레퍼런스 확보
          유튜브 주 1편 지속
9~12주차  첫 유료 외주 수주 (50~150만원)
          전자책 집필 시작
3~6개월   외주 3~5건 + 버티컬 SaaS MVP 개발
6~12개월  SaaS 런칭 → MRR 구축
```

---

## 14. 라이선스 주의사항 (중요)

| 폴더 | 라이선스 | 의미 |
|---|---|---|
| `autogpt_platform/` | **PolyForm Shield 1.0** | 개인·사내 사용 가능 / **경쟁 호스팅 서비스로 판매 불가** |
| `classic/` 및 그 외 전체 | **MIT** | 거의 자유로운 사용 |

### 해도 되는 것

- 사내 업무 자동화에 사용
- 셀프호스팅으로 무료 운영
- 교육 / 강의 / 컨설팅 / 구축 대행
- AutoGPT **API를 호출하는** 자체 제품 개발 (버티컬 SaaS, WP 플러그인 등)
- 마켓플레이스에 에이전트 등록·판매
- 커스텀 블록 개발 및 기여

### 하면 안 되는 것

- 코드를 복제해 **AutoGPT 와 경쟁하는 호스팅 서비스**로 판매
- 라이선스 고지 제거

> SaaS 사업화를 진지하게 검토한다면 **반드시 변호사 검토**를 거칠 것.

---

## 15. 리스크 및 대응

| 리스크 | 대응 방안 |
|---|---|
| AutoGPT 종속 | 엔진 교체 가능하도록 **추상화 레이어** 설계 |
| 라이선스 해석 | SaaS 사업화 전 변호사 검토 필수 |
| AI 비용 폭증 | 고객 요금에 충분한 마진 + Ollama(로컬 모델) 혼용 + 사전 견적 활용 |
| 경쟁 심화 | **버티컬(업종) 특화**로 진입장벽 구축 |
| AI 오작동 사고 | `human_in_the_loop` 블록 필수 적용 + 계약서 면책 조항 |
| 서버 비용 | 초기엔 호스팅판, 규모 생기면 셀프호스팅 전환 |

---

## 16. 참고 링크

| 구분 | 링크 |
|---|---|
| 원본 저장소 | <https://github.com/Significant-Gravitas/AutoGPT> |
| 작업 저장소 (포크) | <https://github.com/bmshin94/AutoGPT> |
| 작업 브랜치 | <https://github.com/bmshin94/AutoGPT/tree/claude/elegant-cerf-nvu830> |
| 공식 문서 | <https://docs.agpt.co> |
| 호스팅 플랫폼 | <https://platform.agpt.co> |
| 가격 정책 | <https://agpt.co/pricing> |
| 커뮤니티 (Discord) | <https://discord.gg/autogpt> |
| 이슈 | <https://github.com/Significant-Gravitas/AutoGPT/issues> |
| 토론 | <https://github.com/Significant-Gravitas/AutoGPT/discussions> |
| PolyForm Shield 1.0 | <https://polyformproject.org/licenses/shield/1.0.0> |
| 기여 가이드 | [`CONTRIBUTING.md`](../../CONTRIBUTING.md) · [`AGENTS.md`](../../AGENTS.md) |
| 프론트엔드 규약 | [`autogpt_platform/frontend/CONTRIBUTING.md`](../../autogpt_platform/frontend/CONTRIBUTING.md) |
| 테스트 전략 | [`autogpt_platform/frontend/TESTING.md`](../../autogpt_platform/frontend/TESTING.md) |

---

## 부록 — 저장소 내 주요 파일 빠른 참조

| 알고 싶은 것 | 열어볼 파일 |
|---|---|
| 블록이 뭔지 | `autogpt_platform/backend/backend/blocks/basic.py` |
| 블록 베이스 클래스 | `autogpt_platform/backend/backend/blocks/_base.py` |
| 커스텀 블록 만들기 | `autogpt_platform/backend/backend/sdk/__init__.py` |
| 연동 가능한 서비스 | `autogpt_platform/backend/backend/integrations/providers.py` |
| 실행 엔진 | `autogpt_platform/backend/backend/executor/manager.py` |
| 비용 계산 | `autogpt_platform/backend/backend/executor/cost_tracking.py` |
| 외부 공개 API | `autogpt_platform/backend/backend/api/external/v1/routes.py` |
| DB 전체 스키마 | `autogpt_platform/backend/schema.prisma` |
| MCP 연동 | `autogpt_platform/backend/backend/blocks/mcp/client.py` |
| 사람 승인 블록 | `autogpt_platform/backend/backend/blocks/human_in_the_loop.py` |
| 서비스 구성 | `autogpt_platform/docker-compose.platform.yml` |
| 프론트 라우트 | `autogpt_platform/frontend/src/app/(platform)/` |

---

*이 문서는 AutoGPT 저장소 전수조사를 바탕으로 작성되었습니다.*
*수치(가격·수익 규모)는 시장 추정치이며 실제와 다를 수 있습니다.*
