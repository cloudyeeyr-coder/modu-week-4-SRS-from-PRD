# SRS v0.4 종합 평가 보고서

> **평가 대상:** `SRS-v0_4.md` (Revision 1.2, 1,481 lines / 82,887 bytes)  
> **평가 기준:** ISO/IEC/IEEE 29148:2018  
> **평가 일시:** 2026-04-18  

---

## 1. 종합 평점

| 평가 관점 | 점수 | 등급 | 비고 |
|:---|:---:|:---:|:---|
| **ISO 29148 구조 준수** | 95 / 100 | A | 전 섹션 구비, 일부 엔티티 정의 누락 |
| **기능 요구사항 완성도** | 92 / 100 | A | 32개 REQ 중 5건 결번 처리됨, 나머지 27건 완비 |
| **비기능 요구사항 완성도** | 90 / 100 | A- | 28개 NFR 중 1건 구 버전 잔존 (REQ-NF-019) |
| **추적성 무결성** | 88 / 100 | B+ | Use Case 매핑표 2건 삭제 REQ 잔존 발견 |
| **기술 스택 정합성** | 97 / 100 | A+ | 외부 API 0건, 내부 DB 독립 아키텍처 완전 달성 |
| **다이어그램 충실도** | 93 / 100 | A | 11개 다이어그램 구비, QUOTE_LEAD 테이블 ER 누락 |
| **테스트 가능성** | 95 / 100 | A | Given-When-Then AC 100% 정의, 측정 기준 명시 |
| **MVP 가치 보전**  | 98 / 100 | A+ | 6개 핵심 가치 모두 전달 수단 보전 확인 |
| **종합** | **93 / 100** | **A** | **개발 착수 가능 (잔존 결함 수정 후)** |

---

## 2. 관점별 상세 평가

### 2.1 ISO 29148 구조 준수

| 섹션 | ISO 29148 요구 항목 | 충족 여부 | 비고 |
|:---|:---|:---:|:---|
| §1 Introduction | Purpose, Scope, Constraints, Assumptions, Definitions, References | ✅ | 완비 |
| §2 Stakeholders | 역할, 페르소나, 관심사, Phase | ✅ | 8개 Stakeholder 정의됨 |
| §3 System Context | External Systems, Client Apps, Use Case, API Overview, Sequences | ✅ | UseCase + Component + 6개 핵심 시퀀스 |
| §4 Specific Requirements | Functional / Non-Functional 분리 | ✅ | FR 27건 + NFR 28건 |
| §5 Traceability Matrix | Story↔REQ, Feature↔REQ, KPI↔NFR, Pain↔REQ | ✅ | 4개 매핑표 + 1개 Pain 매핑 |
| §6 Appendix | API List, Data Model, ER/Class, Detailed Sequences, Validation, Risk | ✅ | 완비 |

### 2.2 기능 요구사항 커버리지

| Feature | REQ 수 | Priority | 커버리지 |
|:---|:---:|:---:|:---:|
| F-01 에스크로 (Admin 수동) | 4건 (001-003, 005) | Must | ✅ 100% |
| F-02 AS망 및 보증서 | 3건 (006-008) | Must | ✅ 100% |
| F-03 평판 뷰어 (정적 DB) | 2건 (009, 010) | Must | ✅ 100% |
| F-04 뱃지 시스템 | 5건 (013-017) | Must | ✅ 100% |
| F-05 비용 비교 + 수기 견적 | 4건 (018-021) | Should | ✅ 100% |
| F-06 O2O 파견 (Phase 2) | 4건 (023-026) | Should | ✅ 100% |
| 온보딩/검색 | 3건 (027-029) | Must | ✅ 100% |
| 파트너십 관리 | 3건 (030-032) | Must | ✅ 100% |
| 모니터링 | 4건 (033-036) | Must | ✅ 100% |
| **합계** | **32건 (27 활성 + 5 결번)** | — | ✅ |

### 2.3 기술 스택 정합성

| 기술 스택 요소 | 문서 반영 상태 | 평가 |
|:---|:---|:---:|
| Next.js App Router (C-TEC-001) | CON-11, Component Diagram, Server Actions/Components 명시 | ✅ |
| Server Actions / Route Handlers (C-TEC-002) | CON-12, API Overview 전체, Endpoint List 27행 | ✅ |
| Prisma + SQLite/Supabase (C-TEC-003) | CON-13, ORM 매핑 노트, Prisma Schema 예시 | ✅ |
| Tailwind + shadcn/ui (C-TEC-004) | CON-14 | ✅ |
| Vercel AI SDK + Gemini (C-TEC-005/006) | CON-15, Component Diagram AI 노드 | ✅ |
| Vercel 배포 (C-TEC-007) | CON-16, CON-17, Cron Jobs 명시 | ✅ |
| **외부 API 의존성** | PG 0건 / NICE 0건 / 금융 0건 | ✅ **완전 제거** |

### 2.4 다이어그램 충실도

| 다이어그램 유형 | 개수 | 상태 |
|:---|:---:|:---:|
| Component Diagram (§3.1.1) | 1 | ✅ 완비, 외부 API 3노드 삭제 반영 |
| Use Case Diagram (§3.2.1) | 1 | ✅ 22 UC 정의, 5 Actor 매핑 |
| 핵심 시퀀스 다이어그램 (§3.4) | 6 | ✅ Admin 수동 프로세스 반영 |
| ER Diagram (§6.2.10) | 1 | ⚠️ QUOTE_LEAD 테이블 미반영 |
| Class Diagram (§6.2.11) | 1 | ⚠️ QuoteLead 클래스 미반영 |
| 상세 시퀀스 다이어그램 (§6.3) | 6 | ✅ 외부 API 제거 완전 반영 |
| **합계** | **16** | 14 완비 / 2 보완 필요 |

---

## 3. 발견된 잔존 결함 (5건)

> [!WARNING]
> 아래 5건은 **v0.3→v0.4 전환 과정에서 미처 수정되지 않은 항목**으로, 개발 착수 전 반드시 보완해야 합니다.

### 결함 3.1: Use Case 매핑표 — 삭제된 REQ 참조 잔존

| 위치 | 항목 | 잔존 내용 | 수정 방안 |
|:---|:---|:---|:---|
| §3.2.1 (L286) | UC-05 관련 REQ | `REQ-FUNC-001, 004` | → `REQ-FUNC-001` (004는 삭제됨) |
| §3.2.1 (L292) | UC-11 관련 REQ | `REQ-FUNC-020, 022` | → `REQ-FUNC-020` (022는 삭제됨) |

**심각도:** 중 — Traceability 무결성 위반  
**영향:** UC→REQ 역추적 시 존재하지 않는 REQ를 참조하게 됨

### 결함 3.2: REQ-NF-019 — PG 수수료 참조 잔존

| 위치 | 현재 내용 | 문제 |
|:---|:---|:---|
| §4.2.4 (L593) | "PG 수수료는 에스크로 전용 요율 기준 3.5% 이하" | v0.4는 PG를 사용하지 않으므로 "PG 수수료" 개념이 부적합 |

**수정 방안:** PG 미사용 시 삭제 또는 "향후 PG 도입 시 적용" 문구로 Phase 2 이관  
**심각도:** 하 — MVP 실행에 직접 영향 없으나 문서 정합성 불일치

### 결함 3.3: QUOTE_LEAD 엔티티 정의 누락

| 위치 | 현재 상태 | 영향 |
|:---|:---|:---|
| §6.2 Data Model | QUOTE_LEAD 테이블 정의 없음 | 시퀀스(§3.4.6, §6.3.3)에서 `QUOTE_LEAD INSERT/UPDATE` 참조하나 스키마 미정의 |
| §6.2.10 ER Diagram | QUOTE_LEAD 엔티티 미반영 | ER 관계도에서 누락 |
| §6.2.11 Class Diagram | QuoteLead 클래스 미반영 | 도메인 객체 구조에서 누락 |

**수정 방안:** `6.2.10` 이전에 QUOTE_LEAD 엔티티 정의 추가 (id, buyer_company_id, robot_model, quantity, term_months, contact_info, status, response_data, created_at)  
**심각도:** 중 — 개발 시 테이블 스키마 참조 불가

### 결함 3.4: WARRANTY 엔티티 정의 누락

| 위치 | 현재 상태 | 영향 |
|:---|:---|:---|
| §6.2 Data Model | WARRANTY 테이블 정의 없음 | 상세 시퀀스(§6.3.1)에서 `WARRANTY INSERT` 참조하나 스키마 미정의 |
| API-12 | 보증서 발급 엔드포인트 정의됨 | 데이터 모델 없이 구현 불가 |

**수정 방안:** WARRANTY 엔티티 정의 추가 (id, contract_id, escrow_tx_id, as_company_name, as_contact_info, coverage_scope, issued_at, pdf_url)  
**심각도:** 중 — REQ-FUNC-006 구현 시 테이블 스키마 필요

### 결함 3.5: CONTRACT 상태 enum 불일치

| 위치 | 정의 | 문제 |
|:---|:---|:---|
| §6.2.4 (L779) | `'pending', 'escrow_held', 'inspecting', 'completed', 'disputed'` | `release_pending` 누락 |
| §6.2.11 Class Diagram (L1020) | `pending\|escrow_held\|inspecting\|release_pending\|completed\|disputed` | `release_pending` 포함 |

**수정 방안:** §6.2.4 테이블 정의에 `release_pending` 상태 추가  
**심각도:** 하 — Class Diagram에는 반영됨, 테이블 정의만 미갱신

---

## 4. 강점 분석

### 4.1 아키텍처 단순화 성공
- 외부 API 의존성 **8건 → 0건** 달성으로, MVP 개발 시 **API 키 확보·계약·과금 관리 부담을 완전 제거**
- Vercel 내부 DB망에서 독립 작동 가능한 **Self-contained 아키텍처** 확보
- 바이브코딩(AI 기반 개발)에 최적화된 **단일 프레임워크 구조**

### 4.2 Admin 수동 운영 모델의 전략적 장점
- MVP 검증 단계에서 **규제 리스크(전자금융업 등록)를 원천 회피**
- 에스크로 수동 확인 구조가 오히려 **법률 자문 부담을 경감** (자금 직접 보관 없음)
- Phase 2에서 PG 연동 시 **상태 머신(held/released/refunded) 구조를 재활용 가능**

### 4.3 테스트 가능성
- 모든 기능 요구사항에 **Given-When-Then AC** 명시 (100%)
- 수치 기준 명확: ≤3분, ≤1시간, ≤5초, p95, ≥95%, <0.5% 등
- Validation Plan(EXP-01~05)으로 **비즈니스 가설 검증 실험** 설계 완비

### 4.4 추적성 구조
- **5단계 추적 체인:** PRD Story → REQ-FUNC → Feature → Test Case → Pain
- KPI/Goal → NFR 매핑으로 **비즈니스 목표와 기술 요구사항 연결**
- Risk Registry(5건)가 모두 REQ 또는 CON에 역추적 가능

---

## 5. 후속 조치 권고

### 5.1 즉시 수정 (v0.4.1 패치)

| 순번 | 결함 | 예상 소요 | 우선순위 |
|:---:|:---|:---:|:---:|
| 1 | UC-05, UC-11 삭제 REQ 참조 제거 | 2분 | 🔴 High |
| 2 | QUOTE_LEAD 엔티티 정의 + ER/Class 반영 | 15분 | 🔴 High |
| 3 | WARRANTY 엔티티 정의 + ER/Class 반영 | 15분 | 🔴 High |
| 4 | CONTRACT 테이블 정의에 `release_pending` 추가 | 2분 | 🟡 Medium |
| 5 | REQ-NF-019 PG 수수료 → 삭제 또는 Phase 2 이관 | 2분 | 🟡 Medium |

> 예상 총 소요: **~36분**

### 5.2 개발 착수 준비 (v0.4 기준)

| 단계 | 작업 | 선행 조건 |
|:---:|:---|:---|
| 1 | Prisma Schema 초기 생성 (10개 엔티티) | 결함 3.3, 3.4 수정 후 |
| 2 | `updateEscrowStatus` / `confirmRelease` Server Action 구현 | F-01 REQ 확정 |
| 3 | `requestManualQuote` Server Action + QUOTE_LEAD 테이블 | 결함 3.3 수정 후 |
| 4 | Admin 대시보드 (에스크로 상태 관리 UI) | F-01 Server Action 완료 후 |
| 5 | TC-001~TC-032 테스트 케이스 상세 설계 | 모든 REQ AC 확정 후 |

### 5.3 Phase 2 이관 항목

| 항목 | 설명 | 원본 REQ |
|:---|:---|:---|
| PG 에스크로 API 연동 | Admin 수동 → 자동 결제 전환 | (신규) |
| NICE 평가정보 API 연동 | 정적 DB → 실시간 캐시 전환 | (신규) |
| 금융 파트너 API 연동 | 수기 견적 → 실시간 금융 연결 | (신규) |
| REQ-NF-019 PG 수수료 | PG 도입 시 활성화 | REQ-NF-019 |
| O2O 매니저 파견 (F-06) | Phase 2 예정 | REQ-FUNC-023~026 |

---

## 6. 결론

SRS v0.4는 **PRD v0.2의 비즈니스 요구사항을 충실히 반영**하면서, 외부 API 의존성을 완전히 제거하여 **MVP 바이브코딩에 즉시 착수 가능한 수준**의 기술 명세 문서로 평가됩니다.

**잔존 결함 5건**(Use Case 매핑 2건, 엔티티 누락 2건, 상태 enum 1건)은 문서 정합성 이슈이며, **~36분 내 패치 가능**합니다. 이를 수정하면 ISO 29148 완전 준수 상태에 도달합니다.

> **최종 판정: A등급 (93/100) — 잔존 결함 패치 후 개발 착수 가능** ✅
