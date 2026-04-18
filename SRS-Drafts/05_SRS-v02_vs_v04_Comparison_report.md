# SRS v0.2 → v0.4 변경사항 비교 보고서

> **비교 대상:**  
> - **SRS-v0_2.md** — Revision 1.1 (v0.3), 1,569 lines / 88,573 bytes  
> - **SRS-v0_4.md** — Revision 1.2 (v0.4.1), 1,576 lines / 86,484 bytes  
> **분석 일시:** 2026-04-18  
> **최종 갱신:** 2026-04-18 (v0.4.1 패치 반영 — 잔존 결함 5건 수정, 엔티티 2건 추가)

---

## 1. 기술 스택의 명확성

> v0.2에서 v0.4로 전환하면서 **외부 API 의존성 3건을 전면 제거**하고, Next.js 내부 DB망에서 독립 작동하는 구조로 전환하였다. 이 변경은 기술 스택의 경계를 한층 명확히 하고, MVP 바이브코딩에서 필요한 사전 API 키 확보 0건을 달성한다.

### 1.1 외부 시스템 인터페이스 (§3.1)

| 항목 | v0.2 (Before) | v0.4 (After) | 변경 유형 |
|:---|:---|:---|:---:|
| **EXT-01** PG 에스크로 (토스/나이스) | REST API, HTTPS TLS 1.3, PCI-DSS Level 1, 타임아웃 10초 | **삭제** — Admin 수동 입출금 확인으로 대체 | 🔴 삭제 |
| **EXT-02** NICE 평가정보 API | REST API, HTTPS TLS 1.3, 일일 500건 한도, 캐시 TTL 30일 | **삭제** — 운영팀 사전 DB 업데이트(정적 데이터)로 대체 | 🔴 삭제 |
| **EXT-03** 카카오 알림톡 | REST API, HTTPS, 건당 10원 | 유지 (변경 없음) | ⚪ 유지 |
| **EXT-04** SMS 발송 서비스 | REST API, HTTPS, 건당 20원 | 유지 (변경 없음) | ⚪ 유지 |
| **EXT-05** 금융 파트너 API (RaaS/리스) | REST API, HTTPS TLS 1.3, 응답 5초, 장애 시 대안 경로 | **삭제** — 수기 견적 요청 더미 폼(QUOTE_LEAD)으로 대체 | 🔴 삭제 |

### 1.2 API Overview (§3.3)

| API ID | v0.2 (Before) | v0.4 (After) | 변경 유형 |
|:---|:---|:---|:---:|
| **API-01** | PG 에스크로 결제 (외부 Outbound, Route Handler) | Admin 에스크로 상태 변경 (**내부** Server Action) | 🟡 변경 |
| **API-02** | PG 에스크로 자금 방출 (외부 Outbound, Route Handler) | Admin 자금 방출 확인 (**내부** Server Action) | 🟡 변경 |
| **API-03** | NICE 기업 신용정보 조회 (외부 Outbound) | **삭제** | 🔴 삭제 |
| **API-06** | 금융 파트너 RaaS 견적 (외부 Outbound) | **삭제** | 🔴 삭제 |
| **API-08** | RaaS 비용 계산 엔진 — "금융 파트너 API 의존" | RaaS 비용 계산 엔진 — **"내부 DB 기반 계산"** | 🟡 변경 |
| **API-13** | (없음) | **수기 견적 요청 더미 폼** (Server Action `requestManualQuote`) | 🟢 신규 |

### 1.3 Component Diagram (§3.1.1)

| 컴포넌트 | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| **EXT_PG** (PG사 에스크로 API) | 외부 시스템 노드 존재, `M_ESC → EXT_PG` 연결 | **삭제** | 🔴 삭제 |
| **EXT_NICE** (NICE평가정보 API) | 외부 시스템 노드 존재, `M_CREDIT → EXT_NICE` 연결 | **삭제** | 🔴 삭제 |
| **EXT_FIN** (금융파트너 API) | 외부 시스템 노드 존재, `M_RAAS → EXT_FIN` 연결 | **삭제** | 🔴 삭제 |
| **M_ESC** 도메인 모듈 | `escrow/ (결제 예치/방출/분쟁)` | `escrow/ (**Admin 수동 예치/방출 확인**)` | 🟡 변경 |
| **M_CREDIT** 도메인 모듈 | `credit/ (NICE 캐시, TTL=30d)` | `credit/ (**운영팀 사전 DB 업데이트 기반**)` | 🟡 변경 |
| **M_RAAS** 도메인 모듈 | `raas/ (3옵션 비교 계산)` | `raas/ (**비용 비교 계산 + 수기견적폼**)` | 🟡 변경 |
| **CRON** (Vercel Cron Jobs) | 뱃지만료 스캔/**NICE배치갱신**/검수기한/SLA | 뱃지만료 스캔/검수기한/SLA — **NICE배치갱신 삭제** | 🟡 변경 |

### 1.4 데이터 모델 변경 (§6.2 ESCROW_TX)

| 필드 | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| `pg_tx_id` (VARCHAR 100) | PG사 트랜잭션 ID (NOT NULL) | **삭제** | 🔴 삭제 |
| `held_at` | TIMESTAMP, NOT NULL | TIMESTAMP, **NULLABLE** | 🟡 변경 |
| `failure_count` (INT) | 결제 실패 횟수 (DEFAULT 0) | **삭제** | 🔴 삭제 |
| `admin_verified_at` | (없음) | TIMESTAMP, NULLABLE — **Admin 입금 확인 시각** | 🟢 신규 |
| `admin_memo` | (없음) | TEXT, NULLABLE — **관리자 메모(입금자명, 증빙)** | 🟢 신규 |

### 1.5 데이터 모델 변경 (§6.2 CONTRACT)

| 필드 | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| `status` ENUM | `pending, escrow_held, inspecting, completed, disputed` | + **`release_pending`** 상태 추가 (Admin 수동 송금 대기) | 🟢 신규 |

### 1.6 신규 엔티티 추가 (v0.4.1 패치)

| 엔티티 (§6.2) | v0.2 | v0.4 | 주요 필드 |
|:---|:---|:---|:---|
| **WARRANTY** (§6.2.10) | 엔티티 정의 없음 | **신규 추가** — AS 보증서 | contract_id, escrow_tx_id, as_company_name, coverage_scope, pdf_url (11필드) |
| **QUOTE_LEAD** (§6.2.11) | 엔티티 정의 없음 | **신규 추가** — 수기 견적 요청 | buyer_company_id, robot_model, quantity, term_months, status, response_data (13필드) |

### 1.7 Class Diagram 변경 (§6.2.13 Class)

| 클래스/메서드 | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| `EscrowTx.pgTxId` | String 필드 존재 | **삭제** | 🔴 삭제 |
| `EscrowTx.failureCount` | Int 필드 존재 | **삭제** | 🔴 삭제 |
| `EscrowTx.deposit()` | `deposit(contractId, amount)` | `adminConfirmDeposit(contractId, amount, memo)` | 🟡 변경 |
| `EscrowTx.release()` | `release()` | `adminConfirmRelease(memo)` | 🟡 변경 |
| `EscrowTx.retryPayment()` | `retryPayment() → Boolean` | **삭제** | 🔴 삭제 |
| `EscrowTx.recordFailure()` | `recordFailure()` | **삭제** | 🔴 삭제 |
| `EscrowTx.adminVerifiedAt` | (없음) | Timestamp 필드 **추가** | 🟢 신규 |
| `EscrowTx.adminMemo` | (없음) | String 필드 **추가** | 🟢 신규 |
| `Contract.status` enum | `pending\|escrow_held\|inspecting\|completed\|disputed` | + **`release_pending`** 상태 추가 | 🟢 신규 |
| **Warranty** 클래스 | (없음) | **신규 추가** — `issue()`, `generatePdf()` 포함 | 🟢 신규 |
| **QuoteLead** 클래스 | (없음) | **신규 추가** — `submit()`, `respond()`, `close()` 포함 | 🟢 신규 |

### 1.8 ER Diagram 변경 (§6.2.12)

| 관계 | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| `CONTRACT → WARRANTY` | (없음) | `CONTRACT \|\|--\|\| WARRANTY : "guaranteed by"` | 🟢 신규 |
| `BUYER_COMPANY → QUOTE_LEAD` | (없음) | `BUYER_COMPANY \|\|--o{ QUOTE_LEAD : submits` | 🟢 신규 |
| `EscrowTx → Warranty` | (없음) | `EscrowTx ..> Warranty : triggers issuance` | 🟢 신규 |
| `Contract → Warranty` | (없음) | `Contract "1" --> "1" Warranty : guarantees` | 🟢 신규 |
| `BuyerCompany → QuoteLead` | (없음) | `BuyerCompany "1" --> "0..*" QuoteLead : submits` | 🟢 신규 |

---

## 2. MVP 목표 및 가치전달 조정 내용

> 사용자 경험(UX)의 핵심 가치 — **에스크로 보호**, **평판 투명성**, **비용 비교** — 는 모두 유지된다. 변경된 것은 "**어떤 수단으로 전달하느냐**"이며, 자동화 API → Admin 수동/정적 DB로 전환하면서 MVP 출시 속도를 극대화하였다.

### 2.1 기능 요구사항 삭제 (총 5건)

| 삭제 REQ ID | v0.2 내용 | 삭제 사유 | 가치전달 보전 방안 |
|:---|:---|:---|:---|
| **REQ-FUNC-004** | PG 응답 타임아웃(>10초) 시 자동 재시도 1회 + CS 접수 유도 | PG API 미사용으로 불필요 | Admin 수동 확인이므로 타임아웃 개념 자체 소멸 |
| **REQ-FUNC-011** | NICE API 장애/한도 초과 시 캐시 폴백 + "실시간 조회 불가" 배너 | NICE API 미사용으로 불필요 | 정적 DB 서빙, 갱신일 표시로 신뢰성 보전 |
| **REQ-FUNC-012** | NICE 잔여 한도 50건 미만 Ops 알림 + 캐시 모드 전환 | NICE API 미사용으로 불필요 | 외부 API 한도 개념 소멸 |
| **REQ-FUNC-022** | 금융 파트너 API 타임아웃(>5초)/5xx 시 토스트 + 30초 재시도 + 이메일 대안 | 금융 API 미사용으로 불필요 | 수기 견적 폼이 곧 대안 경로 |
| **REQ-NF-003** | 에스크로 결제 API(PG 연동) 응답 시간 p95 ≤ 3,000ms, 타임아웃 ≤ 10s | PG API 미사용으로 불필요 | Admin 수동 확인에 NFR 불필요 |

### 2.2 기능 요구사항 변경 (총 8건)

| REQ ID | v0.2 | v0.4 | 가치 영향 |
|:---|:---|:---|:---|
| **REQ-FUNC-001** | PG사 에스크로 API로 자금 예치 (≤3분, 실패율 <0.5%) | Admin **수동** 입금 확인 후 'held' 상태 변경 (admin_verified_at 기록) | 보호 가치 동일, 처리 시간 ↑ 허용 |
| **REQ-FUNC-002** | PG사 에스크로 API 호출하여 자금 방출 (≤24시간) | Admin 대시보드 **'방출 대기'** 알림 → 수기 송금 후 'released' 변경 | 보호 가치 동일 |
| **REQ-FUNC-009** | 재무 등급·시공 성공률·리뷰 통합 표시 (갱신 주기 ≤30일) | 재무 등급(**운영팀 사전 DB 업데이트**) 표시 + **갱신일** 표시 | 정보 제공 동일, 실시간→정적 전환 |
| **REQ-FUNC-018** | 일시불·리스·RaaS 3옵션 자동 계산 표시 | **내부 DB 기반**으로 3옵션 자동 계산 표시 | 동일 (데이터 소스만 변경) |
| **REQ-FUNC-020** | 금융 파트너 API 호출 → 보증금·월납입금·금리 100% 표시 | **"운영팀에 맞춤 견적 요청하기"** 팝업 → QUOTE_LEAD DB 저장 + Admin 알림 | 실시간→리드 수집 전환 |
| **REQ-FUNC-021** | 인라인 에러 표시 + **API 호출 차단** | 인라인 에러 표시 + **계산 요청 차단** | 동일 (문구 변경) |
| **REQ-FUNC-033** | **에스크로 결제 실패율** > 0.5% → Slack 알림 | **Admin 에스크로 상태 변경 오류율** > 0.5% → Slack 알림 | 모니터링 대상만 변경 |
| **REQ-FUNC-035** | RaaS 엔진 p95>3초 or **금융 API 실패율>5%** → Eng+Biz 알림 | RaaS 엔진 p95>3초 → **Eng팀만** 알림 (금융 API 조건 삭제) | 알림 범위 축소 |

### 2.3 비기능 요구사항 변경 (총 3건)

| REQ ID | v0.2 | v0.4 | 비고 |
|:---|:---|:---|:---|
| **REQ-NF-003** | PG 에스크로 API 응답 p95 ≤ 3,000ms | **삭제** | PG 미사용 |
| **REQ-NF-014** | 결제 데이터 PCI-DSS Level 1 준수 (PG사 위임) | **결제 입출금 증빙 및 Admin 확인 기록** 전자금융거래법 5년 보존 | PG 위임→자체 보존 의무 전환 |
| **REQ-NF-019** | PG 수수료 에스크로 전용 요율 3.5% 이하 | *(Phase 2 이관)* MVP에서는 PG 미사용, Phase 2 도입 시 활성화 | PG 미사용 → Phase 2 이관 |

### 2.4 MVP 핵심 가치 보전 확인

| 핵심 가치 | v0.2 전달 수단 | v0.4 전달 수단 | 보전 여부 |
|:---|:---|:---|:---:|
| **투자금 보호** (에스크로) | PG사 자동 예치/방출 | Admin 수동 예치/방출 확인 | ✅ 보전 |
| **AS 단절 방지** (보증서+AS망) | 에스크로 완료 → 보증서 자동 발급 | 동일 (변경 없음) | ✅ 보전 |
| **역량 투명성** (평판 뷰어) | NICE API 실시간 + 캐시 폴백 | 운영팀 DB 정적 데이터 + 갱신일 표시 | ✅ 보전 |
| **기안 의사결정** (리포트 PDF) | DB 기반 PDF 4섹션 자동 생성 | 동일 (변경 없음) | ✅ 보전 |
| **비용 비교** (RaaS 계산기) | 내부 계산 + 금융 API 실시간 연결 | 내부 DB 계산 + 수기 견적 요청 폼 | ✅ 보전 |
| **파트너 매칭** (뱃지 시스템) | 제조사 뱃지 발급·검색·필터 | 동일 (변경 없음) | ✅ 보전 |

---

## 3. 기타 차이점

### 3.1 제약사항(Constraints) 변경 (§1.2.3)

| ID | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| **CON-01** | PG사 에스크로 API 위임 구조 채택 | 법인 계좌 수기 입출금, 상태값만 기록 | 🟡 변경 |
| **CON-02** | NICE API 캐시 TTL 30일, 일일 500건 | 운영팀 사전 수동 DB 업데이트, Phase 2 이후 외부 API | 🟡 변경 |
| **CON-04** | PG사 B2B 고액 거래(1억 원+) 지원 필요 | **삭제** | 🔴 삭제 |
| **CON-06** | NICE API 법적 허용 필요 | **삭제** | 🔴 삭제 |
| **CON-08** | 결제 데이터 PCI-DSS Level 1 (PG사 위임) | 결제 입출금 증빙 + Admin 확인 기록 5년 보존 | 🟡 변경 |

### 3.2 가정(Assumptions) 변경 (§1.2.4)

| ID | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| **ASM-01** | PG사 에스크로 B2B 고액 거래 지원 가정 | **삭제** | 🔴 삭제 |
| **ASM-03** | NICE API SI 재무 등급 조회 법적 가능 가정 | **삭제** | 🔴 삭제 |

### 3.3 In-Scope 범위 문구 변경 (§1.2.1)

| ID | v0.2 | v0.4 |
|:---|:---|:---|
| **IS-01** | 에스크로 결제 및 자금 방출 로직 | 무통장 입금 기반 **Admin 수동 예치/방출 확인** 시스템 |
| **IS-03** | SI 파트너 재무/시공 투명 평판 뷰어 및 기안 리포트 PDF 생성 | SI 파트너 시공 투명 평판 뷰어 **(운영팀 사전 DB 업데이트 기반)** 및 기안 리포트 PDF 생성 |
| **IS-05** | RaaS 구독 및 OPEX 비용 비교 계산기 | OPEX 비용 비교 계산기 및 **이메일 수기 견적 요청(더미 폼)** |

### 3.4 Fallback Strategy → MVP 아키텍처 원칙 전환 (§3.1)

| 관점 | v0.2 | v0.4 |
|:---|:---|:---|
| **명칭** | "외부 시스템 장애 시 임시 우회 전략 (Fallback Strategy)" | **"MVP 아키텍처 원칙"** |
| **EXT-02 NICE** | 캐시 폴백 + 디폴트 더미 데이터 + "과거 데이터 노출" 명시 | **삭제** (정적 DB 서빙이므로 장애 시나리오 소멸) |
| **EXT-05 금융 API** | 고정 견적 테이블 참조 → 이메일 견적 요청 폼 더미 우회 | **삭제** (견적 폼이 기본 경로이므로 우회 개념 소멸) |
| **EXT-03, EXT-04** | 웹 알림함 + SMTP 이메일 우회 | 동일 유지 |

### 3.5 핵심 시퀀스 다이어그램 변경 (§3.4)

| 다이어그램 | v0.2 핵심 변경점 | v0.4 핵심 변경점 |
|:---|:---|:---|
| **3.4.1** 에스크로 | `B → P → PG` (PG 타임아웃 alt/재시도) | `B → P → ADMIN` (수동 입금 확인, `release_pending` 상태) |
| **3.4.2** SI 평판 | `P → NICE` (캐시 Hit/Miss alt, 5xx 폴백) | `P → DB` (정적 재무 등급 직접 조회, **NICE 참여자 삭제**) |
| **3.4.6** RaaS 구독 | `P → FIN` (금융 API 정상/타임아웃 alt) | `P → ADMIN` (수기 견적 폼, QUOTE_LEAD INSERT) |
| **3.4.3~5** | 변경 없음 | 변경 없음 |

### 3.6 상세 시퀀스 다이어그램 변경 (§6.3)

| 다이어그램 | v0.2 | v0.4 |
|:---|:---|:---|
| **6.3.1** 에스크로 상세 | PG participant, 타임아웃 alt, 자동 재시도, `PG → BFF` 방출 | ADMIN participant, 수동 입금 확인, `release_pending` 상태, Admin 수기 송금 |
| **6.3.3** RaaS 금융 연결 상세 | FIN participant, 정상/타임아웃 alt, 30초 재시도, 이메일 대안 | ADMIN participant, "운영팀 맞춤 견적" 팝업 → QUOTE_LEAD INSERT → Admin 응답 |
| **6.3.4** SI 재무 등급 상세 | NICE+CACHE+BATCH participant, 캐시 Hit/Miss alt, 한도 모니터링, 일일 배치 | DB participant만, 정적 조회 + PDF 생성, **"운영팀 주기적 수동 업데이트" 주석** |

### 3.7 Traceability Matrix 변경 (§5)

| 매핑표 | v0.2 | v0.4 | 영향 |
|:---|:---|:---|:---|
| **Story 1** REQ ID | 001~008 (8건) | 001~003, 005~008 (**7건**, 004 삭제) | TC-004 결번 |
| **Story 2** REQ ID | 009~017 (9건) | 009, 010, 013~017 (**7건**, 011·012 삭제) | TC-011, TC-012 결번 |
| **Story 4** REQ ID | 018~022 (5건) | 018~021 (**4건**, 022 삭제) | TC-022 결번 |
| **F-01** Feature 명 | "안심 에스크로 결제 시스템" | "안심 에스크로 결제 시스템 **(Admin 수동 확인)**" | — |
| **F-03** Feature 명 | "SI 파트너 재무/시공 투명 평판 뷰어" | "SI 파트너 시공 투명 평판 뷰어 **(운영팀 DB 기반)**" | — |
| **F-05** Feature 명 | "RaaS 구독 & OPEX 비용 비교 계산기" | "OPEX 비용 비교 계산기 & **수기 견적 요청 (더미 폼)**" | — |
| **P-04** Pain 추적 | REQ-FUNC-018~**022** | REQ-FUNC-018~**021** | — |

### 3.8 Use Case 매핑표 변경 (§3.2.1) — v0.4.1 패치 반영

| Use Case | v0.2 | v0.4 | 변경 유형 |
|:---|:---|:---|:---:|
| **UC-05** 관련 REQ | `REQ-FUNC-001, 004` | `REQ-FUNC-001` (004 삭제됨) | 🟡 변경 |
| **UC-11** 명칭 | "금융 파트너 연결 요청" | **"수기 견적 요청 (운영팀 연결)"** | 🟡 변경 |
| **UC-11** 관련 REQ | `REQ-FUNC-020, 022` | `REQ-FUNC-020` (022 삭제됨) | 🟡 변경 |

### 3.9 Risk Registry 변경 (§6.5)

| Risk ID | v0.2 | v0.4 |
|:---|:---|:---|
| **R-04** | 에스크로 결제가 전자금융업 등록 대상 판정 → **PG사 에스크로 위임 구조 확인** | Admin 수동 입출금 확인 구조가 전자금융업 대상 판정 → **자금 직접 보관 없이 상태값만 기록 구조 유지** |

### 3.10 Endpoint List 변경 (§6.1)

| 항목 | v0.2 (29행) | v0.4 (27행) | 비고 |
|:---|:---|:---|:---|
| 에스크로 예치/방출 | `POST /api/escrow/deposit`, `POST /api/escrow/release` (2행) | `action: updateEscrowStatus` (1행) | 2→1 병합 |
| NICE 재무 조회 | `GET /api/credit/query` | **삭제** | |
| 금융 파트너 연결 | `POST /api/raas/finance` | **삭제** | |
| 수기 견적 요청 | (없음) | `action: requestManualQuote` | **신규 추가** |

### 3.11 문서 메타 정보 변경

| 항목 | v0.2 | v0.4 |
|:---|:---|:---|
| **Revision** | 1.1 (v0.3) | **1.2 (v0.4)** |
| **Change Log** | v0.2 → v0.3 기술 스택 정합성 전환 | v0.3 → v0.4 **외부 API 의존성 제거** 및 Admin 수동 운영 체계 전환 |
| **Footer** | SRS v1.1 (v0.3), v0.2 대비 기술 스택 정합성 전환 | SRS **v1.2 (v0.4)**, v0.3 대비 **외부 API 의존성 제거** 및 Admin 수동 운영 체계 전환 |

### 3.12 섹션 번호 변경 (v0.4.1 패치)

| 섹션 | v0.2 | v0.4 | 사유 |
|:---|:---|:---|:---|
| ER Diagram | §6.2.10 | **§6.2.12** | WARRANTY(§6.2.10), QUOTE_LEAD(§6.2.11) 삽입으로 밀림 |
| Class Diagram | §6.2.11 | **§6.2.13** | 동일 사유 |

---

## 4. 요약 통계

| 구분 | 삭제 | 변경 | 신규 | 유지 |
|:---|:---:|:---:|:---:|:---:|
| 외부 시스템 (EXT) | 3 | 0 | 0 | 2 |
| API Overview (API) | 2 | 2 | 1 | 7 |
| 기능 요구사항 (REQ-FUNC) | 4 | 8 | 0 | 24 |
| 비기능 요구사항 (REQ-NF) | 1 | 2 | 0 | 25 |
| 제약사항 (CON) | 2 | 3 | 0 | 12 |
| 가정 (ASM) | 2 | 0 | 0 | 3 |
| 데이터 엔티티 | 0 | 2 | 2 | 9 |
| 데이터 필드 (ESCROW_TX) | 2 | 1 | 2 | 5 |
| Use Case 매핑 | 0 | 3 | 0 | 19 |
| 시퀀스 다이어그램 | 0 | 6 | 0 | 3 |
| ER/Class 관계 | 0 | 1 | 5 | 8 |
| **합계** | **16** | **28** | **10** | **117** |

---

## 5. v0.4.1 패치 이력

> v0.4 초기 완성 후 종합 평가에서 **잔존 결함 5건**이 식별되어 즉시 패치되었습니다.

| # | 결함 | 심각도 | 수정 내용 |
|:---:|:---|:---:|:---|
| 1 | UC-05에 삭제된 REQ-FUNC-004 참조 잔존 | 중 | `001, 004` → `001` |
| 2 | UC-11에 삭제된 REQ-FUNC-022 참조 잔존 + 명칭 불일치 | 중 | `금융 파트너 연결 요청, 020/022` → `수기 견적 요청 (운영팀 연결), 020` |
| 3 | QUOTE_LEAD 엔티티 정의·ER·Class 누락 | 중 | §6.2.11 테이블(13필드) + ER 관계 + Class `QuoteLead` 추가 |
| 4 | WARRANTY 엔티티 정의·ER·Class 누락 | 중 | §6.2.10 테이블(11필드) + ER 관계 + Class `Warranty` 추가 |
| 5 | CONTRACT 테이블 `release_pending` 상태 누락 | 하 | ENUM에 `release_pending` 추가 |
| +α | REQ-NF-019 PG 수수료 부적합 | 하 | *(Phase 2 이관)* 문구 추가, MVP 미적용 명시 |
| +α | UC-11 Mermaid 다이어그램 라벨 불일치 | 하 | Mermaid 코드 라벨 갱신 |
| +α | 섹션 번호 불일치 | 하 | Class Diagram §6.2.11 → §6.2.13 |

> **패치 후 잔존 결함: 0건** ✅
