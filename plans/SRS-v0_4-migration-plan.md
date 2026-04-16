# SRS v0.4 마이그레이션 플랜: C-TEC 기술 스택 전환

**Plan ID:** PLAN-SRS-v0.4-MIGRATION  
**작성일:** 2026-04-16  
**원본:** `SRS-Drafts/SRS-v0_3_Opus.md` (v1.1, 1,920 lines)  
**목표:** `SRS-Drafts/SRS-v0_4_Opus.md` (C-TEC 스택 적용)  
**상태:** 🟡 리뷰 대기

---

## 1. 목적

SRS v0.3의 아키텍처(마이크로서비스형 BFF + 분리 백엔드 서비스 + Redis + 범용 클라우드)를  
**C-TEC-001~007 기술 스택**(Next.js App Router 단일 풀스택 + Supabase + Vercel)으로 전환한다.

전환 시 **MVP 핵심 사용자 경험(가치 전달)이 훼손되지 않음**을 검증하고,  
기능적·비기능적 요구사항의 커버리지를 보장하는 것을 목적으로 한다.

---

## 2. C-TEC 기술 스택 정의

| ID | 제약사항 | 카테고리 |
|:---:|:---|:---:|
| **C-TEC-001** | 모든 서비스는 Next.js (App Router) 기반 단일 풀스택 프레임워크로 구현 | 내부 |
| **C-TEC-002** | 서버 측 로직은 Server Actions 또는 Route Handlers로 구현 (별도 백엔드 서버 없음) | 내부 |
| **C-TEC-003** | DB는 Prisma + SQLite(로컬) / Supabase PostgreSQL(배포) | 내부 |
| **C-TEC-004** | UI/스타일링은 Tailwind CSS + shadcn/ui | 내부 |
| **C-TEC-005** | LLM 오케스트레이션은 Vercel AI SDK로 Next.js 내부 구현 | 외부 |
| **C-TEC-006** | LLM은 Google Gemini API 기본, 환경 변수로 모델 교체 가능 | 외부 |
| **C-TEC-007** | 배포/인프라는 Vercel 단일화, Git Push 자동 배포 | 외부 |

---

## 3. 변경 범위 매트릭스

### 3.1 SRS 섹션별 영향도

| SRS 섹션 | 현재 (v0.3) | C-TEC 적용 후 (v0.4) | 영향도 | 작업 유형 |
|:---|:---|:---|:---:|:---:|
| **1.2.3** Constraints | CON-01~10 | C-TEC-001~007 **추가** (CON-11~17) | 🟡 중 | 추가 |
| **1.2.4** Assumptions | ASM-01~05 | ASM-06 (Vercel Pro 플랜 전제) **추가** | 🟢 소 | 추가 |
| **1.3** Definitions | 용어 정의 | Next.js, Prisma, Supabase, Vercel AI SDK 등 용어 **추가** | 🟢 소 | 추가 |
| **1.4** References | REF-01~03 | REF-04~06 (Next.js Docs, Prisma Docs, Vercel Docs) **추가** | 🟢 소 | 추가 |
| **3.1** External Systems | EXT-01~05 | EXT-06 (Supabase), EXT-07 (Google Gemini) **추가** | 🟡 중 | 추가 |
| **3.1.0** Fallback Strategy | Redis 캐시 기반 | DB 레벨 캐시 + Vercel KV 대체 | 🟡 중 | 수정 |
| **3.1.1** Component Diagram | BFF + 10개 분리 서비스 | **Next.js 단일 앱 구조로 전면 재설계** | 🔴 대 | 재작성 |
| **3.2** Client Applications | 4개 독립 포털 | Next.js Route Groups로 논리 분리 | 🟡 중 | 수정 |
| **3.3** API Overview | 별도 서비스별 API | Route Handlers로 동일 엔드포인트 유지, 구현 방식 변경 | 🟢 소 | 수정 |
| **3.4** Sequence Diagrams | BFF/DB 참여자 | 참여자명 일부 조정 (BFF → Next.js Server) | 🟢 소 | 수정 |
| **4.1** Functional Req | REQ-FUNC-001~036 | **변경 없음** (구현 방식만 변경) | ✅ 없음 | — |
| **4.2.1** 성능 | Datadog RUM | Vercel Analytics + Speed Insights | 🟡 중 | 수정 |
| **4.2.4** 비용 | 월 500만 원 | Vercel Pro + Supabase Pro ≈ 월 6만 원 | 🟢 소 | 수정 |
| **4.2.5** 확장성 | 범용 수평 확장 | Vercel Serverless auto-scaling | 🟢 소 | 수정 |
| **6.2** Data Model | Prisma 없음 | Prisma 스키마 명시 (기존 테이블 구조 유지) | 🟢 소 | 보완 |
| **8.2** ADR | ADR-001~003 | **ADR-004** (Next.js 단일 풀스택 채택) **추가** | 🟡 중 | 추가 |
| **10** Deployment | 3환경 범용 클라우드 | Vercel Preview + Production, Git Push CI/CD | 🟡 중 | 수정 |

### 3.2 기술 스택 매핑표

| 현재 SRS (v0.3) | C-TEC 스택 (v0.4) | 비고 |
|:---|:---|:---|
| BFF / API Gateway | Next.js App Router + Middleware | C-TEC-001 |
| 분리 백엔드 서비스 (Escrow, Contract, AS 등) | Server Actions + Route Handlers | C-TEC-002 |
| PostgreSQL (범용) | Prisma + SQLite(dev) / Supabase PostgreSQL(prod) | C-TEC-003 |
| Redis Cache | Supabase DB 캐시 (TTL 칼럼 비교) 또는 Vercel KV | C-TEC-003 보완 |
| 프론트엔드 (별도 명시 없음) | Tailwind CSS + shadcn/ui | C-TEC-004 |
| — (LLM 없음) | Vercel AI SDK + Google Gemini API | C-TEC-005, 006 |
| 범용 클라우드 (3환경) | Vercel (Preview + Production) | C-TEC-007 |
| Datadog + PagerDuty | Vercel Analytics + Sentry + Slack Webhook | 대체 |
| Batch Scheduler (범용) | Vercel Cron Jobs | Pro 플랜 필수 |
| OAuth 2.0 + MFA (범용) | NextAuth.js (Auth.js) + TOTP | 구현 대체 |
| PDF Generator (범용) | @react-pdf/renderer (Server Action 내) | 구현 대체 |
| 파일 스토리지 (미명시) | Vercel Blob 또는 Supabase Storage | 신규 추가 |

---

## 4. 섹션별 구체적 수정 계획

### TASK-01: Constraints 확장 (Section 1.2.3)

**작업:** 기존 CON-01~10을 유지하고 C-TEC-001~007을 CON-11~17로 추가

| 신규 ID | 내용 | 근거 |
|:---:|:---|:---|
| CON-11 | 모든 서비스는 Next.js (App Router) 기반 단일 풀스택으로 구현한다 | C-TEC-001 |
| CON-12 | 서버 측 로직은 Server Actions / Route Handlers로 구현한다 | C-TEC-002 |
| CON-13 | DB는 Prisma ORM + Supabase(PostgreSQL)를 사용한다 | C-TEC-003 |
| CON-14 | UI는 Tailwind CSS + shadcn/ui로 구현한다 | C-TEC-004 |
| CON-15 | LLM 오케스트레이션은 Vercel AI SDK를 사용한다 | C-TEC-005 |
| CON-16 | LLM은 Google Gemini API를 기본으로 사용한다 | C-TEC-006 |
| CON-17 | 배포는 Vercel 플랫폼으로 단일화한다 | C-TEC-007 |

### TASK-02: Assumptions 추가 (Section 1.2.4)

| 신규 ID | 가정 | 검증 방안 | 검증 시한 |
|:---:|:---|:---|:---|
| ASM-06 | Vercel Pro 플랜을 사용하며, Serverless 함수 실행 시간 300초, Cron 1분 간격을 지원한다 | Vercel Pro 플랜 가격 및 제한사항 공식 문서 확인. 테스트 프로젝트에서 300초 함수 실행 검증 | D-30일 |
| ASM-07 | Supabase(PostgreSQL)가 MVP 규모의 동시 접속(500 CCU)과 데이터 규모를 지원한다 | Supabase Pro 플랜 connection pooling 설정 확인. Prisma connection pool 테스트 | D-14일 |

### TASK-03: 용어집 확장 (Section 1.3)

추가 용어: `Next.js App Router`, `Server Actions`, `Route Handlers`, `Prisma ORM`, `Supabase`, `Vercel AI SDK`, `Vercel Cron`, `Vercel KV`, `shadcn/ui`, `Sentry`

### TASK-04: References 추가 (Section 1.4)

| 신규 ID | 문서명 | 설명 |
|:---:|:---|:---|
| REF-04 | Next.js Documentation (App Router) | 프레임워크 공식 문서 |
| REF-05 | Prisma Documentation | ORM 공식 문서 |
| REF-06 | Vercel AI SDK Documentation | LLM 오케스트레이션 공식 문서 |

### TASK-05: External Systems 확장 (Section 3.1)

기존 EXT-01~05 유지 + 아래 추가:

| 신규 ID | 외부 시스템 | 인터페이스 | 프로토콜 | 역할 | 제약 사항 |
|:---:|:---|:---|:---:|:---|:---|
| EXT-06 | Supabase (PostgreSQL + Storage) | Prisma Client / REST | HTTPS | 운영 DB 및 파일 스토리지. 로컬 dev는 SQLite 사용 | Connection pooling 필수, Row Level Security 설정 |
| EXT-07 | Google Gemini API | Vercel AI SDK | HTTPS | LLM 기반 SI 추천, 리포트 초안 생성 등 AI 기능 | 분당 60 RPM (무료). 모델 교체 시 환경 변수만 변경 |

### TASK-06: Fallback Strategy 수정 (Section 3.1.0)

| 변경 항목 | 현재 | 변경 후 |
|:---|:---|:---|
| EXT-02 NICE 캐시 | "내부 DB 캐시(TTL 30일)" | "Supabase DB의 `financial_grade_updated_at` 칼럼 기준 30일 비교. Vercel Cron 일일 배치로 사전 갱신" |
| EXT-05 금융 API | "내부 DB 기본 금리 테이블" | "Supabase `default_finance_rates` 테이블의 사전 확보 데이터로 참고용 예상치 표시" |
| **신규** EXT-07 Gemini | — | "Gemini API 장애 시 LLM 보조 기능 비활성화 (핵심 기능에 영향 없음). 환경 변수로 대안 모델(OpenAI 등) 즉시 전환 가능" |

### TASK-07: Component Diagram 전면 재설계 (Section 3.1.1) 🔴

**현재:** BFF + CoreServices(4) + MatchingServices(3) + FinanceServices(2) + InfraServices(4) + DB + Redis  
**변경:** Next.js 단일 앱 내 논리적 모듈 분리

```
[v0.4 Component Diagram 구조안]

┌─ Next.js App Router ─────────────────────────────────────┐
│                                                           │
│  ┌─ Route Groups (Pages) ──────────────────────────────┐  │
│  │  /app/(buyer)/    — 수요기업 포털                   │  │
│  │  /app/(si)/       — SI 파트너 포털                  │  │
│  │  /app/(mfr)/      — 제조사 포털                     │  │
│  │  /app/(admin)/    — Admin 대시보드                   │  │
│  │  Tailwind CSS + shadcn/ui                           │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│  ┌─ API Layer (Route Handlers) ────────────────────────┐  │
│  │  /api/v1/escrow/*      — 에스크로 결제/방출/분쟁    │  │
│  │  /api/v1/contracts/*   — 계약/검수 관리             │  │
│  │  /api/v1/as-tickets/*  — AS 티켓/배정/SLA           │  │
│  │  /api/v1/badges/*      — 뱃지 발급/관리             │  │
│  │  /api/v1/raas/*        — RaaS 비용 계산/금융 연결   │  │
│  │  /api/v1/si-partners/* — SI 검색/프로필/재무        │  │
│  │  /api/v1/ai/*          — Gemini LLM 호출            │  │
│  │  /api/cron/*           — Vercel Cron 엔드포인트     │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│  ┌─ Server Logic ──────────────────────────────────────┐  │
│  │  Server Actions        — 폼 처리, 데이터 변경       │  │
│  │  Prisma Client         — DB 접근 계층               │  │
│  │  Vercel AI SDK         — Gemini LLM 오케스트레이션  │  │
│  │  NextAuth.js (Auth.js) — 인증/인가 (OAuth 2.0+MFA)  │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│  ┌─ Middleware ────────────────────────────────────────┐  │
│  │  인증 검증, RBAC, Rate Limiting, 로깅              │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────┘
          │              │              │
          ▼              ▼              ▼
┌─ Vercel Infra ──┐ ┌─ Supabase ──┐ ┌─ External APIs ────────┐
│ Vercel KV       │ │ PostgreSQL  │ │ PG사 에스크로 (EXT-01) │
│ Vercel Blob     │ │ Storage     │ │ NICE 평가정보 (EXT-02) │
│ Vercel Cron     │ │ Auth (보조) │ │ 카카오 알림톡 (EXT-03) │
│ Vercel Analytics│ │             │ │ SMS 게이트웨이 (EXT-04)│
│ Sentry (에러)   │ │             │ │ 금융 파트너   (EXT-05) │
└─────────────────┘ └─────────────┘ │ Gemini API    (EXT-07) │
                                    └────────────────────────┘
```

### TASK-08: Client Applications 수정 (Section 3.2)

| 현재 | 변경 |
|:---|:---|
| CLI-01~04: 4개 독립 반응형 웹 앱 | Next.js App Router의 **Route Group**으로 논리 분리. 단일 빌드 내 4개 영역 |

각 포털은 `(buyer)`, `(si)`, `(mfr)`, `(admin)` Route Group으로 분리되며, 공통 레이아웃(`/app/layout.tsx`)과 독립 레이아웃을 병용한다.

### TASK-09: API Overview 수정 (Section 3.3)

- 기존 API-01~12 엔드포인트 **URL 및 기능 동일 유지**
- "방향" 열의 "내부" → "Next.js Route Handler (내부)" 로 표현 변경
- "외부 (Outbound)" → REST 호출은 Route Handler 내 `fetch()` 또는 SDK로 구현

### TASK-10: Sequence Diagrams 참여자명 조정 (Section 3.4)

| 현재 참여자 | 변경 참여자 | 적용 다이어그램 |
|:---|:---|:---|
| `BFF as 플랫폼 BFF` | `SRV as Next.js Server` | 6.3.1~6.3.6 상세 시퀀스 |
| `DB as 데이터베이스` | `DB as Supabase(Prisma)` | 전체 |
| `CACHE as DB캐시(TTL=30일)` | `DB as Supabase(Prisma)` (통합) | 6.3.4 |
| `BATCH as 배치 스캐너` | `CRON as Vercel Cron` | 6.3.2, 6.3.4 |
| `LOG as 로그/모니터링` | `MON as Sentry/Analytics` | 6.3.1, 6.3.3 |

> 3.4절의 핵심 시퀀스는 추상화 수준이 높아 참여자명만 변경. 6.3절의 상세 시퀀스는 전면 조정.

### TASK-11: 성능 요구사항 도구 변경 (Section 4.2.1)

| Req ID | 현재 측정 도구 | 변경 측정 도구 |
|:---:|:---|:---|
| REQ-NF-001 | Datadog RUM Core Web Vitals | **Vercel Speed Insights** Core Web Vitals |
| REQ-NF-005 | k6 또는 Locust 부하 테스트 | k6 (외부 부하 주입) → Vercel Serverless 대상 |
| REQ-NF-006 | k6, Staging 환경 | k6, **Vercel Preview 환경** |

### TASK-12: 비용 요구사항 수정 (Section 4.2.4)

| Req ID | 현재 | 변경 |
|:---:|:---|:---|
| REQ-NF-018 | "MVP 인프라 비용 월 500만 원 이하" | "MVP 인프라 비용 월 500만 원 이하 (Vercel Pro $20 + Supabase Pro $25 + 외부 API 종량제 기준 월 약 10만 원 이내 예상)" |

### TASK-13: ADR-004 신규 추가 (Section 8.2)

#### ADR-004: Next.js 단일 풀스택 아키텍처 채택

| 항목 | 내용 |
|:---|:---|
| **결정** | 마이크로서비스형 BFF + 분리 백엔드 대신 Next.js (App Router) 단일 풀스택 프레임워크를 채택한다 |
| **배경/제약** | MVP 단계에서 1~2명 소규모 팀이 10+ 분리 서비스를 운영하는 것은 DevOps 부담 과다. 기능 요구사항 36개를 빠르게 구현하려면 풀스택 단일 코드베이스가 효율적 |
| **검토한 대안** | ① BFF + 마이크로서비스 (운영 복잡도 상) ② Next.js + 별도 FastAPI 백엔드 (Python/TS 이중 관리) ③ Next.js 단일 풀스택 (단순, 빠른 반복) |
| **결론** | ③ 채택. Server Actions + Route Handlers로 백엔드 로직 완전 대체. Vercel 배포로 인프라 관리 제거. Prisma + Supabase로 DB 계층 단순화 |
| **리스크 잔존** | 서비스 규모 확장 시 단일 앱의 빌드 시간 증가 → Turbopack 사용으로 완화. 트래픽 급증 시 Serverless Cold Start → Vercel Edge Functions 검토 |
| **관련 제약** | CON-11~17 (C-TEC-001~007) |

### TASK-14: Deployment 섹션 수정 (Section 10)

| 항목 | 현재 | 변경 |
|:---|:---|:---|
| 환경 구성 | Development / Staging / Production (3환경) | **Local (SQLite)** / **Preview (Vercel, Supabase dev)** / **Production (Vercel, Supabase prod)** |
| 배포 방식 | 미명시 | Git Push → Vercel 자동 빌드·배포. PR 생성 시 Preview 자동 배포 |
| CI/CD | 미명시 | **Vercel 내장** (별도 CI/CD 설정 불필요) |
| 부하 테스트 | MVP D-14 Staging | MVP D-14 **Preview 환경** 대상 k6 외부 부하 주입 |

### TASK-15: 모니터링 요구사항 조정 (Section 4.1.9)

| Req ID | 현재 도구 | 변경 도구 | 기능 동일성 |
|:---:|:---|:---|:---:|
| REQ-FUNC-033 | PagerDuty 알림 | **Sentry Alert** → Slack Webhook | ✅ |
| REQ-FUNC-034 | Slack 즉시 알림 | **Vercel Cron** 스캔 → Slack Webhook | ✅ |
| REQ-FUNC-035 | Eng+Biz 동시 알림 | **Sentry Performance** → Slack 다중 채널 | ✅ |
| REQ-FUNC-036 | Datadog RUM | **Vercel Speed Insights** → Slack Alert | ✅ |

### TASK-16: LLM 기능 신규 요구사항 도출

C-TEC-005, 006에 따라 Gemini LLM을 활용하는 **신규 기능 요구사항**을 추가한다.

> **MVP LLM 활용 범위 (사용자 확인 필요)**

| 신규 Req ID | 요구사항 (안) | Priority | 관련 Feature |
|:---:|:---|:---:|:---:|
| REQ-FUNC-037 | 시스템은 SI 파트너 검색 시 사용자의 자연어 요구사항을 LLM으로 해석하여 최적 SI 3개사를 추천해야 한다 | Could | F-03, F-04 |
| REQ-FUNC-038 | 시스템은 기안용 리포트 PDF의 '종합 평가' 섹션을 LLM으로 자동 생성하여 초안을 제공해야 한다 | Could | F-03 |
| REQ-FUNC-039 | 시스템은 RaaS 비용 비교 결과에 대해 LLM 기반 의사결정 보조 코멘트를 생성해야 한다 | Could | F-05 |

> **중요:** LLM 기능은 모두 **Could** 우선순위로 설정하며, 핵심 기능(Must/Should)의 동작에 의존하지 않는다. Gemini API 장애 시 해당 기능만 비활성화되며, 에스크로·뱃지·AS 등 핵심 흐름에 영향 없음.

### TASK-17: 파일 스토리지 요구사항 추가

| 관련 REQ | 저장 대상 | 저장소 |
|:---:|:---|:---|
| REQ-FUNC-006 | AS 보증서 PDF | Vercel Blob 또는 Supabase Storage |
| REQ-FUNC-010 | 기안용 리포트 PDF | 동일 |
| REQ-FUNC-019 | RaaS 비교 결과 PDF | 동일 |
| REQ-FUNC-025 | O2O 방문 보고서 | 동일 |

### TASK-18: 인증 구현 방식 명시

| Req ID | 현재 | 변경 |
|:---:|:---|:---|
| REQ-NF-016 | "OAuth 2.0 + MFA 필수" (구현 미명시) | "**NextAuth.js (Auth.js)** + OAuth 2.0 Provider (Google/Kakao). B2B 관리자 계정에 **TOTP 기반 MFA** 적용" |

---

## 5. MVP 핵심 사용자 경험 보존 검증

### 5.1 검증 기준

PRD에서 정의된 **5대 Pain → Feature → 사용자 가치 전달 경로**가 기술 스택 변경에 의해 훼손되지 않는지 검증한다.

### 5.2 Pain → Feature → UX 보존 검증표

#### P-01: SI 업체 파산/잠적 → 유지보수 단절 트라우마 (AOS=4.00, 최고 Pain)

| 핵심 UX | 사용자가 느끼는 가치 | v0.3 구현 | v0.4 (C-TEC) 구현 | 보존 |
|:---|:---|:---|:---|:---:|
| **계약금이 안전하게 보호됨을 체감** | "내 돈이 PG사에 안전하게 예치되어 있다" | BFF → PG API 호출 | Route Handler → PG API 호출 | ✅ |
| **보증서를 즉시 받아 안심** | "결제 완료 즉시 보증서가 내 손에" | BFF → PDF 생성 → 반환 | Server Action → @react-pdf/renderer → Vercel Blob 저장 → URL 반환 | ✅ |
| **SI 잠적 시 24시간 내 대체 AS 출동** | "누가 와도 빨리 와서 고쳐주면 된다" | BFF → DB 매칭 → 알림 | Server Action → Prisma 매칭 → 카카오/SMS API | ✅ |
| **에스크로 분쟁 시 중재팀 자동 개입** | "문제가 생겨도 플랫폼이 나서준다" | Scheduler → 분쟁 전환 | Vercel Cron → Route Handler → Prisma 상태 변경 | ✅ |

> **결론:** P-01 핵심 UX **100% 보존**. PG API 연동, PDF 생성, 알림 발송 모두 Next.js Route Handlers/Server Actions로 동일하게 구현 가능.

---

#### P-02: 업체 역량 증빙 불가, 기안 반려 반복 (AOS=2.88)

| 핵심 UX | 사용자가 느끼는 가치 | v0.3 구현 | v0.4 (C-TEC) 구현 | 보존 |
|:---|:---|:---|:---|:---:|
| **SI 프로필에서 재무+기술+리뷰를 한눈에** | "이 업체가 믿을 만한지 숫자로 보인다" | BFF → NICE API + DB → 렌더링 | Route Handler → NICE API + Prisma → RSC 렌더링 | ✅ |
| **기안 리포트 PDF를 5초 내에 받아 바로 제출** | "클릭 한 번으로 경영진 보고 끝" | BFF → PDF 생성 | Server Action → @react-pdf/renderer + (선택) Gemini 종합평가 초안 | ✅+ |
| **뱃지 SI가 검색 상단에 노출** | "제조사가 보증한 업체를 바로 찾는다" | 검색 엔진 정렬 | Prisma `orderBy` + `where` 조건 (뱃지 우선 정렬) | ✅ |

> **결론:** P-02 핵심 UX **100% 보존 + LLM으로 선택적 향상 가능** (기안 리포트 종합평가 자동 초안).

---

#### P-03: 비대면 계약 불신, 온라인 전환 거부 (AOS=1.35, Phase 2)

| 핵심 UX | 사용자가 느끼는 가치 | 보존 |
|:---|:---|:---:|
| **O2O 매니저 파견 예약 캘린더** | "멱살 잡을 수 있는 담당자가 직접 온다" | ✅ |
| **방문 보고서로 추천 SI 3개사 확인** | "직접 만나서 추천받으니 신뢰된다" | ✅ |

> **결론:** P-03은 Phase 2 범위이며, 구현 경로 변경 없이 **보존**.

---

#### P-04: CAPEX 부담, 구독형 상품 부재 (AOS=2.00)

| 핵심 UX | 사용자가 느끼는 가치 | v0.3 구현 | v0.4 (C-TEC) 구현 | 보존 |
|:---|:---|:---|:---|:---:|
| **일시불 vs 리스 vs RaaS 3옵션 비교를 2초 내** | "우리 상황에 뭐가 유리한지 한눈에" | 계산 엔진 서비스 → JSON | Server Action 내 계산 로직 → JSON | ✅ |
| **비교 결과 PDF 즉시 다운로드** | "대표님한테 바로 보여줄 수 있다" | PDF 생성 서비스 | Server Action → @react-pdf/renderer | ✅ |
| **금융 파트너 실시간 견적 연결** | "은행 가지 않고 월비용을 바로 안다" | 금융 API 서비스 | Route Handler → 금융 파트너 API | ✅ |

> **결론:** P-04 핵심 UX **100% 보존**.

---

#### P-05: SI 파트너 탐색 비용 과다 (AOS=2.88)

| 핵심 UX | 사용자가 느끼는 가치 | v0.3 구현 | v0.4 (C-TEC) 구현 | 보존 |
|:---|:---|:---|:---|:---:|
| **지역·브랜드·역량 태그로 1초 내 검색** | "박람회 안 가도 적합한 업체가 바로 나온다" | 검색 엔진 서비스 | Prisma full-text search + where 필터 | ✅ |
| **파트너 제안 → 수락/거절 → 뱃지 자동 발급** | "이 SI와 공식 파트너가 되었다" | 파트너십 서비스 | Server Action → Prisma 트랜잭션 | ✅ |
| **미응답 시 자동 리마인더 + 대안 SI 추천** | "응답 없으면 플랫폼이 대안을 찾아준다" | Batch Scheduler | Vercel Cron → Route Handler → Prisma 쿼리 | ✅ |

> **결론:** P-05 핵심 UX **100% 보존**.

---

### 5.3 보존 검증 종합 결과

| Pain | AOS | 핵심 Feature | UX 보존 | 비고 |
|:---:|:---:|:---|:---:|:---|
| **P-01** | 4.00 | 에스크로 + AS 보증 | ✅ 100% | PG API 연동 동일 |
| **P-02** | 2.88 | 평판 뷰어 + 뱃지 | ✅ 100% (+LLM 향상 가능) | Gemini로 종합평가 초안 가능 |
| **P-03** | 1.35 | O2O 매니저 파견 | ✅ 100% | Phase 2, 경로 동일 |
| **P-04** | 2.00 | RaaS 계산기 | ✅ 100% | 계산 로직 내부 구현 |
| **P-05** | 2.88 | SI 검색 + 파트너십 | ✅ 100% | Prisma 검색 동일 |

> **판정: 5대 Pain에 대한 핵심 사용자 경험은 기술 스택 변경에 의해 일체 훼손되지 않는다.**  
> 기술 스택 변경은 "내부 구현 방식"의 변경이며, 사용자가 체감하는 API 응답, UI 흐름, 알림 수단은 모두 동일하게 유지된다.

---

## 6. 기능적 커버리지 전수 검토

### 6.1 Functional Requirements (REQ-FUNC-001~036)

| Req ID | 요구사항 요약 | C-TEC 구현 방식 | 커버 |
|:---:|:---|:---|:---:|
| 001 | 에스크로 예치 (≤3분) | Route Handler → PG API | ✅ |
| 002 | 검수 합격 시 자금 방출 | Route Handler → PG API | ✅ |
| 003 | 분쟁 시 중재 개시 (≤2영업일) | Server Action + Slack 알림 | ✅ |
| 004 | PG 타임아웃 재시도 + CS 유도 | Route Handler 내 retry 로직 | ✅ |
| 005 | 검수 7영업일 미응답 시 자동 분쟁 | **Vercel Cron** → Prisma 상태 변경 | ✅ |
| 006 | 보증서 자동 발급 (≤1분) | Server Action → @react-pdf/renderer → Vercel Blob | ✅ |
| 007 | 로컬 AS 매칭 (≤4시간) | Server Action → Prisma 매칭 쿼리 | ✅ |
| 008 | AS SLA 자동 기록 | Prisma timestamp 비교 | ✅ |
| 009 | SI 프로필 통합 표시 (≤2초) | React Server Component + Prisma | ✅ |
| 010 | 기안 리포트 PDF (≤5초) | Server Action → PDF 생성 → Blob | ✅ |
| 011 | NICE 장애 시 캐시 폴백 | Prisma `financial_grade_updated_at` 비교 | ✅ |
| 012 | NICE 한도 50건 미만 시 알림 | Vercel Cron 스캔 → Slack | ✅ |
| 013 | 뱃지 발급 (≤1시간 반영) | Server Action → Prisma INSERT | ✅ |
| 014 | 뱃지 만료/철회 비노출 (≤10분) | Vercel Cron + Prisma UPDATE | ✅ |
| 015 | 뱃지 필터 (혼입률 0%) | Prisma `where` 조건 | ✅ |
| 016 | 뱃지 만료 D-7 알림 | **Vercel Cron** 일일 스캔 | ✅ |
| 017 | Brand-Agnostic 뱃지 (≥3사) | Prisma 스키마 동일 | ✅ |
| 018 | RaaS 3옵션 비교 (≤2초) | Server Action 계산 | ✅ |
| 019 | 비교 결과 PDF (≤3초) | Server Action → PDF | ✅ |
| 020 | 금융 파트너 API 연결 (≤5초) | Route Handler → 금융 API | ✅ |
| 021 | 잘못된 입력 시 에러 (≤200ms) | 클라이언트 Zod 검증 + Server Action | ✅ |
| 022 | 금융 API 장애 시 재시도 + 대안 | Route Handler retry + DB 폴백 | ✅ |
| 023~026 | O2O 매니저 파견 (Phase 2) | 동일 구조 (Route Handler + Prisma) | ✅ |
| 027~028 | 온보딩 | shadcn/ui 폼 + Server Action | ✅ |
| 029 | SI 검색 (≤1초) | Prisma 쿼리 + 인덱싱 | ✅ |
| 030~032 | 파트너 제안/수락/리마인더 | Server Action + Vercel Cron | ✅ |
| 033~036 | 모니터링 알림 | Sentry + Vercel Analytics + Slack | ✅ |

> **결론: REQ-FUNC-001~036 전체 ✅ 커버 (100%)**

### 6.2 Non-Functional Requirements 영향 요약

| 카테고리 | 영향 | 비고 |
|:---|:---:|:---|
| 성능 (001~006) | 🟢 측정 도구만 변경 | 목표값 동일 유지 |
| 신뢰성 (007~013) | 🟢 변경 없음 | Supabase SLA 99.9% |
| 보안 (014~017) | 🟢 구현 도구 변경 | NextAuth.js + Vercel TLS |
| 비용 (018~020) | 🟢 대폭 하회 | 월 ~10만 원 예상 |
| 확장성 (021) | 🟢 Serverless auto-scale | Vercel 자동 |
| 유지보수 (022) | 🟢 Prisma 스키마 동일 | Brand-Agnostic 유지 |
| KPI (023~028) | 🟢 변경 없음 | 측정 경로 동일 |

---

## 7. 리스크 및 의존성 변경

### 7.1 기존 리스크 영향

| Risk ID | 리스크 | C-TEC 영향 | 변경 사항 |
|:---:|:---|:---:|:---|
| R-01 | SI 공급 부족 (Cold-Start) | 없음 | — |
| R-02 | 에스크로 분쟁 폭주 | 없음 | — |
| R-03 | AS 출동 SLA 미달 | 없음 | — |
| R-04 | 규제 리스크 (에스크로) | 없음 | PG 위임 구조 동일 |
| R-05 | 경쟁사 추격 | 없음 | — |

### 7.2 신규 리스크

| Risk ID | 리스크 | 영향도 | 완화 전략 |
|:---:|:---|:---:|:---|
| **R-06** | Vercel Serverless Cold Start로 첫 API 응답 지연 (1~3초) | 중 | Vercel Edge Functions 활용, 주요 Route는 streaming 적용 |
| **R-07** | Vercel Cron 실행 실패 시 배치 작업 누락 (뱃지 만료, 분쟁 전환 등) | 중상 | Cron 실행 로그 모니터링 (Sentry), 실패 시 Slack 즉시 알림. 멱등(idempotent) 설계로 재실행 안전 보장 |
| **R-08** | Supabase 무료/Pro 플랜 Connection Pool 한도 초과 | 중 | Prisma connection pooling 설정 (pool_timeout, connection_limit). Supabase PgBouncer 활용 |

### 7.3 의존성 변경

| 기존 DEP | 변경 사항 |
|:---:|:---|
| DEP-01~06 | **변경 없음** (PG, NICE, 제조사, AS, 금융, 카카오/SMS 의존성 동일) |
| **DEP-07 (신규)** | Vercel Pro 플랜 구독 — MVP D-30일 |
| **DEP-08 (신규)** | Supabase Pro 플랜 구독 — MVP D-30일 |

---

## 8. 실행 로드맵

### Phase 1: SRS 문서 수정 (즉시)

| # | 작업 | TASK ID | 예상 소요 |
|:---:|:---|:---:|:---:|
| 1 | Constraints 추가 (CON-11~17) | TASK-01 | 5분 |
| 2 | Assumptions 추가 (ASM-06~07) | TASK-02 | 5분 |
| 3 | 용어집 + References 추가 | TASK-03, 04 | 5분 |
| 4 | External Systems 확장 + Fallback 수정 | TASK-05, 06 | 10분 |
| 5 | **Component Diagram 재설계** | TASK-07 | 15분 |
| 6 | Client Applications + API 수정 | TASK-08, 09 | 10분 |
| 7 | Sequence Diagram 참여자명 조정 | TASK-10 | 10분 |
| 8 | 성능/비용/모니터링 요구사항 조정 | TASK-11, 12, 15 | 10분 |
| 9 | ADR-004 추가 | TASK-13 | 5분 |
| 10 | Deployment 섹션 수정 | TASK-14 | 5분 |
| 11 | LLM/파일/인증 요구사항 추가 | TASK-16, 17, 18 | 10분 |

### Phase 2: 검증 (SRS v0.4 완성 후)

- [ ] 전체 REQ-FUNC/REQ-NF Traceability Matrix 업데이트
- [ ] 변경된 Sequence Diagram Mermaid 렌더링 검증
- [ ] LLM 기능 범위 최종 확정 (사용자 피드백 대기)

### Phase 3: 개발 착수 준비

- [ ] Prisma 스키마 초안 작성 (`schema.prisma`)
- [ ] Next.js 프로젝트 초기화 (`npx create-next-app@latest`)
- [ ] Vercel Pro + Supabase Pro 구독
- [ ] 환경 변수 설정 (PG API키, NICE API키, Gemini API키 등)

---

## 9. 결론

| 항목 | 판정 |
|:---|:---|
| **SRS 기능 요구사항 커버리지** | REQ-FUNC-001~036 **100% 유지** |
| **비기능 요구사항 커버리지** | **100% 유지** (측정 도구만 대체) |
| **MVP 핵심 UX 보존** | 5대 Pain 전체 **100% 보존** |
| **인프라 복잡도** | 10+ 서비스 → **1개 앱** (대폭 단순화) |
| **인프라 비용** | 월 500만 원 → **월 ~10만 원** (95% 절감) |
| **개발 속도** | 단일 코드베이스 + Vercel 자동 배포 → **대폭 향상** |
| **추가 기회** | Gemini LLM으로 SI 추천·리포트 보조·비용 코멘트 **신규 가치 창출 가능** |

> SRS v0.4 작업을 진행하기 전에 **TASK-16 (LLM 기능 범위)** 에 대한 결정이 필요합니다.
> REQ-FUNC-037~039의 포함 여부 및 우선순위를 결정해주세요.

---

*본 플랜은 SRS v0.3 (1,920 lines) 대비 약 18개 TASK를 정의하며, 기존 요구사항의 기능적 완결성을 100% 유지하면서 기술 스택만 전환하는 것을 목표로 합니다.*
