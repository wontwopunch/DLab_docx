# Dlab통합관리프로그램 기술 개발문서 (Technical Specification)

> **문서 유형**: 개발팀 내부 기술 문서
> **대상**: 백엔드·프론트엔드·DevOps 팀원, AI 코드 어시스턴트
> **버전**: v1.2 (Draft, 2026-07-13) — v1.1(2026-06-23) + 7/10 미팅 + 리뷰 피드백 + 신규 기능(데일리루틴·학습계획·메시지·상담 리포트) 반영
> **범위**: DLab 통합 관리 프로그램(웹, React) 전용 — 재원생 앱(Flutter)은 `APP_기술_개발문서.md` 별도
> **리포지토리**: https://github.com/wontwopunch/DLab_docx.git
> **상태 표기**: ✅ 확정 · 🟡 내부 확정필요 · 🔴 외부/대성 요청필요 · ⚠️ 불일치·주의 · 🆕 7/10 반영 · 🔧 리뷰 반영

---

## 목차
0. [v1.1 → v1.2 변경 요약](#0-v11--v12-변경-요약)
1. [시스템 개요](#1-시스템-개요)
2. [기술 스택](#2-기술-스택)
3. [아키텍처](#3-아키텍처)
4. [DB 설계 원칙](#4-db-설계-원칙)
5. [API 설계 원칙](#5-api-설계-원칙)
6. [외부 연동 명세](#6-외부-연동-명세)
7. [공통 모듈 설계](#7-공통-모듈-설계)
8. [기능별 구현 명세](#8-기능별-구현-명세)
9. [보안·인증·권한](#9-보안인증권한)
10. [배포·인프라](#10-배포인프라)
11. [개발 착수 선결 조건](#11-개발-착수-선결-조건)
12. [Phase별 개발 범위](#12-phase별-개발-범위)
13. [리스크 및 주의사항](#13-리스크-및-주의사항)

---

## 0. v1.1 → v1.2 변경 요약

| # | 변경 | 위치 |
|---|---|---|
| 1 | 🔧 **급식 가상계좌 연결** — 8.4 `createOrder`가 카드만 호출 → **카드/가상계좌 분기 + 만료 처리** 연결 | 6.5 · 8.4 |
| 2 | 🔧 **급식 공휴일 필터** — `mealPolicy`에 공휴일 필터 추가(주말만 있었음), 요일별/달력 명세 | 8.4 |
| 3 | 🔧 **급식 취소 2트랙** — 앱 3일전(PG 자동환불) / 데스크 즉시(관리자), **데스크 당일 신청 flow 신규** | 8.4 |
| 4 | 🔧 **설문 템플릿** — `survey_templates` 스키마·API 추가 | 4.4 · 8.6 |
| 5 | 🔧 **디멤버 회원 구조** — 재원생=앱 / 교사=디멤버 사이트 관리 명시 | 6.1 |
| 6 | 🔧 **양방향 동기화** — 대성전산 Dlab통합관리프로그램 병행 운영 전제(단방향 이관 금지) | 6.1 · 13 |
| 7 | ⚠️ **등록비(TUITION) 결제 주체** — `TuitionPgClient`(학원 명의) vs 수납=대성전산 상충 → 확정 필요 | 6.5 · 11 |
| 8 | 🆕 **데일리 루틴 관리**(신규 도메인) | 4.4 · 8.7 |
| 9 | 🆕 **주·일 학습 계획 관리**(신규 도메인, 비교 그래프) | 4.4 · 8.8 |
| 10 | 🆕 **메시지 관리**(공지 범위 권한·1:1 채팅 React 연동·행정요청 수신) | 4.4 · 8.9 |
| 11 | 🆕 **상담 리포트**(성적 상세 상담용 + 엑셀 업로드 자동화) | 8.5 · 8.10 |
| 12 | ✅ **알림 채널 이중화** — 카카오 알림톡(트랜잭션) + FCM(Daily Report·앱 알림) | 6.6 |
| 13 | 🆕 **승인 라우팅** — 학생 신청 항목별 승인 주체(학부모/선생님/자동) | 8.11 |
| 14 | 📌 **ISMS 책임 범위** — 인증 자체는 학원 주관, 개발은 기술적 보호대책만 | 9.4 |
| 15 | 🖼️ **Daily Report 대시보드 집계**(0710) — 순공랭킹(전체+지점)·달력·데일리테스트 횟수·질의응답 횟수·셀프 피드백 저장 | 8.12 |
| 16 | 🖼️ **상담 리포트 구조 확정**(0710) — 과목별 **이행률 별점** + **상담 항목 마스터(선택형·관리자 편집·최대 노출)** + **담임 스티커** | 8.10 |
| 17 | 🖼️ **학습계획 그리드**(0710) — 교시×요일 그리드·교재/내용·관리자 편집(교시·시간·주말 의무자습·선택목록)·**과목/형태별 비율 파이(주차)** | 8.8 |
| 18 | 🖼️ **성적 상세**(0710) — 6과목 회차별 표(원/표/백/등급) + 단원별 정답률·전국 대비 | 8.5 |
| 19 | 🖼️ **질의응답 ON/OFF**(0710) — off(대면) 상담실 예약 스케줄 관리 + **시험(디랩 콘텐츠) 관리** | 8.13 |
| 20 | 🖼️ **메시지 — 가족 채팅방**(0710) 추가(학부모+학생+관리자) | 8.9 |

> 🖼️ = `디랩_Daily_Report_0710.pdf`(강지웅 과장) 클라이언트 레이아웃 반영 항목.

---

## 1. 시스템 개요

### 1.1 프로젝트 설명
D.Lab(대성학력개발연구소) 전용 학원 관리 프로그램 신규 개발. 현재 대성전산이 개발·운영 중인 Dlab통합관리프로그램(구 Dlab통합관리프로그램)를 자체 시스템으로 대체한다. 재원생 앱과 **동일한 Spring Boot API 서버 + Supabase**를 공유한다.

### 1.2 운영 주체 및 관계사
| 주체 | 역할 | 비고 |
|---|---|---|
| 대성학력개발연구소(DSD) | D.Lab 사이트 운영, 발주처, **앱 스토어 계정 보유** | |
| 대성전산 | 현 Dlab통합관리프로그램·**디멤버**·수납 프로그램 개발·운영 | DB 포함 전체 소유 |
| 자이엘코리아 | Zyxel 방화벽 장비 관리 | 김상원 팀장 / 010-6412-8131 |
| 급식업체 | 급식 공급·PG 가맹점(벤더 명의) | 연락처·환불 처리 |
| 개발팀(숀아이비엔엘) | 신규 Dlab통합관리프로그램·앱 개발 | |

### 1.3 문서 관계
| 문서 | 대상 |
|---|---|
| `Dlab통합관리프로그램(구 DSA)_기술_개발문서.md` | 통합 관리 프로그램(웹, React) — **본 문서**, 공통 백엔드 명세 포함 |
| `APP_기술_개발문서.md` | 재원생 앱(Flutter) — 앱 소비 규약 |

---

## 2. 기술 스택

### 2.1 프론트엔드 (Dlab통합관리프로그램(구 DSA) 웹)
```
Framework  : React 18 (또는 Vue 3 — 최종 확정 필요 🟡)
Language   : TypeScript
UI Library : Ant Design 또는 Shadcn/ui
State      : Zustand(전역) + React Query(서버 상태)
Build      : Vite
HTTP       : Axios (인터셉터로 JWT 자동 첨부)
Excel      : SheetJS (xlsx)  — 성적 업로드 파싱·명단 다운로드
Chart      : Recharts        — 학습계획 비교 그래프·성적 추이·수납 통계
Calendar   : 급식 현황용 월 뷰
Realtime   : (선택) SSE — 출결·사유 실시간 반영
```

### 2.2 백엔드
```
Framework  : Spring Boot 3.x
Language   : Java 21
ORM        : Spring Data JPA + QueryDSL
DB         : Supabase (PostgreSQL 15)
Auth       : Supabase Auth + Spring Security (JWT 검증)
Excel      : Apache POI (성적 업로드 파서·명단 Export)
HTTP Client: WebClient(Reactive) + Resilience4j(Circuit Breaker)
Scheduler  : Spring @Scheduled (가상계좌 만료·결석 배치·Daily Report)
Messaging  : (필요 시) Redis Pub/Sub 또는 SSE
```

### 2.3 인프라
```
Hosting : AWS(EC2/ECS — 확정 필요 🟡) · DB : Supabase Cloud · Storage : Supabase Storage(PDF·이미지)
CI/CD   : GitHub Actions · Container : Docker(+Compose) · Reverse Proxy : Nginx
```

### 2.4 외부 연동
| 시스템 | 방식 | 용도 |
|---|---|---|
| 대성전산 API | REST | **디멤버·수납** 데이터 연동, 양방향 동기화 |
| 키오스크 | Webhook 수신 | 출결·식사체크 실시간 |
| Zyxel Nebula | REST | 방화벽 해제 |
| 더프리미엄모의고사 | REST | 전화번호 기반 성적 조회 |
| 카카오 알림톡 | Kakao API | 트랜잭션 알림 |
| **FCM** | Firebase | 앱 푸시(Daily Report·승인 등) |
| PG(등록비) | REST + Webhook | 카드·가상계좌 (학원 명의) ⚠️ 주체 확정(D-9) |
| PG(급식) | REST + Webhook | 카드·가상계좌 (벤더 명의) |
| 1:1 메신저(외부 유료) | SDK/REST | 학생-담임 채팅 (React 연동, 담당 유현님) 🔴 선정 |

---

## 3. 아키텍처

### 3.1 전체 시스템 구조
```
┌──────────────────────────────────────────────────┐
│  [Dlab통합관리프로그램(구 DSA) 웹 (React)]     [재원생 앱 (Flutter)]         │
└───────────┬───────────────────────┬──────────────┘
            │ HTTPS / REST          │
            ▼                       ▼
┌──────────────────────────────────────────────────┐
│              Spring Boot API Server               │
│  Auth/RBAC │ Business Logic │ External Gateway     │
│  (JWT+Role) │ (도메인)       │ (Circuit Breaker)    │
└───────┬───────────────────────────────┬──────────┘
        ▼                               ▼
┌────────────────┐            ┌─────────────────────┐
│ Supabase       │            │ 외부 시스템           │
│ DB/Auth/Storage│            │ 대성전산(디멤버·수납)   │
└────────────────┘            │ 키오스크 Webhook       │
                              │ Nebula·더프리미엄       │
                              │ 카카오·FCM             │
                              │ PG(등록비·급식 분리)    │
                              │ 외부 메신저(1:1)        │
                              └─────────────────────┘
```

### 3.2 프론트엔드 구조 (React)
```
src/
├── api/            # Axios 인스턴스, API 호출 함수
├── components/
│   ├── common/     # SearchForm, DataTable, ExcelButton, DateRange 등
│   └── features/   # 기능별 컴포넌트
├── pages/          # 라우트별 페이지
├── store/          # Zustand
├── hooks/          # React Query 훅
├── types/          # TS 타입
└── utils/          # 날짜·엑셀·포맷·공휴일
```

### 3.3 백엔드 패키지 구조 (신규 도메인 반영)
```
com.dlab.Dlab통합관리프로그램(구 DSA)/
├── auth/           # JWT·RBAC
├── common/
│   ├── excel/      # POI Import/Export
│   ├── notification/ # 알림톡·FCM·SMS 추상화(채널 라우팅)
│   ├── search/     # QueryDSL 동적 검색
│   ├── webhook/    # Webhook 서명 검증
│   ├── holiday/    # 🆕 공휴일 provider (급식·일정)
│   └── ruleengine/ # 🆕 상벌점 규칙 매핑 엔진
├── external/
│   ├── daesungdb/  # 대성전산 API(디멤버·수납), 양방향 동기화
│   ├── kiosk/      # 키오스크 Webhook
│   ├── nebula/     # Zyxel Nebula
│   ├── pg/         # PG 어댑터 (Tuition / Lunch 분리)
│   ├── premium/    # 더프리미엄
│   ├── kakao/      # 알림톡
│   ├── fcm/        # 🆕 FCM 발송
│   └── messenger/  # 🆕 외부 1:1 메신저 어댑터
├── domain/
│   ├── student/    │  attendance/  │  score/     │  meal/
│   ├── payment/    │  survey/      │  notice/    │  admin/
│   ├── routine/    # 🆕 데일리 루틴
│   ├── learningplan/# 🆕 주·일 학습 계획
│   ├── message/    # 🆕 공지·행정요청·채팅 메타
│   ├── consult/    # 🆕 상담 일지·리포트
│   └── approval/   # 🆕 신청 승인 라우팅
└── config/
```

---

## 4. DB 설계 원칙

### 4.1 공통 컬럼 규칙
```sql
id          UUID PRIMARY KEY DEFAULT gen_random_uuid()
year        INTEGER NOT NULL          -- 학년도(매년 초기화 기준)
branch_id   UUID NOT NULL             -- 지점
created_at  TIMESTAMPTZ DEFAULT now()
updated_at  TIMESTAMPTZ DEFAULT now()
created_by  UUID REFERENCES users(id)
is_deleted  BOOLEAN DEFAULT FALSE     -- soft delete
```

### 4.2 연도/지점 버전 관리 (전년도 복사)
```sql
copied_from_id UUID REFERENCES self(id) NULL
```
복사 순서(의존): `department → course_type → class_group → curriculum → penalty_item → tuition`
단순 `INSERT SELECT` 금지 — `YearlySnapshotService`로 그래프 순회 후 새 PK 매핑.
🆕 **데일리 루틴/학습계획 템플릿**도 "전월/전년도 복사" 지원(과목 월별 상이).

### 4.3 학번 초기화
```sql
students ( id UUID PK, year INTEGER, student_no VARCHAR(10), UNIQUE(year, student_no) )
-- 학번 = year + seq 채번, 학번을 PK로 사용 금지
```

### 4.4 주요 테이블 목록 (신규 ★)
```sql
-- 기초
departments, course_types, class_groups, curriculums, penalty_items

-- 학생/대기자
students, student_waitlist, admission_consults, class_assignments

-- 출결
attendance_logs, absence_requests, study_sessions, patrol_logs ★

-- 결제·수납
payment_transactions,           -- TUITION | MEAL
meal_orders, meal_order_items,
meal_policies ★                 -- 급식 가능일 규칙(주말·공휴일 제외)

-- 성적·설문
score_records,
score_detail_uploads ★,         -- 엑셀 업로드 원본·매핑 이력(상담용)
surveys, survey_questions, survey_responses, survey_answers,
survey_templates ★              -- 설문 템플릿(재사용)

-- 🆕 학습
daily_routines ★,               -- 월별 루틴/테스트 정의(과목·배점·권장)
daily_routine_results ★,        -- 학생별 수행 결과(완료/미완료/점수)
learning_plans ★,               -- 주/일 계획 헤더
learning_plan_items ★,          -- 과목×유형(수업·인강·자습) 계획/실제
penalty_rules ★                 -- 루틴·순찰·출결 → 상벌점 매핑 규칙

-- 🆕 메시지/상담/승인
notices ★,                      -- 공지(scope: ALL|BRANCH|CLASS|INDIVIDUAL)
admin_requests ★,               -- 행정 요청(사전정의 항목)
chat_threads ★,                 -- 1:1 채팅 메타(외부 메신저 참조ID)
consult_logs ★, consult_reports ★,
approval_items ★,               -- 신청 항목 마스터(approver_type)
approval_requests ★             -- 신청·승인 인스턴스

-- 🖼️ 0710 반영
consult_tags ★,                 -- 과목별 상담 항목 마스터(선택형·관리자 편집·max_display)
consult_report_subjects ★,      -- 상담 리포트 과목별(이행률 별점·선택 태그·스티커·코멘트)
learning_plan_options ★,        -- 학습계획 선택목록(드롭다운) 마스터(과목×형태)
qna_offline_slots ★, qna_offline_reservations ★,  -- 대면 질의응답 예약
vocab_tests ★, vocab_test_results ★,              -- 영단어 시험(디랩 콘텐츠)
daily_report_feedback ★,        -- 학생 셀프 피드백(다짐 한마디)
study_time_rankings ★           -- 순공 랭킹 집계(전체/지점, 일·주·월)

-- 알림/관리자
notification_logs, notification_templates,
users, roles, branch_configs
```

---

## 5. API 설계 원칙

### 5.1 URL 규칙
```
GET/POST/PUT/PATCH/DELETE  /api/v1/{resource}[/{id}]
POST  /api/v1/{resource}/copy-from-year          -- 연도/월 복사
GET   /api/v1/{resource}/export                  -- 엑셀 다운로드
POST  /api/v1/{resource}/import[/preview]        -- 엑셀 업로드(미리보기)
```

### 5.2 공통 응답
```json
{ "success": true, "data": {}, "meta": {"page":1,"size":20,"total":150}, "error": null }
```
에러: `{ "success": false, "error": {"code","message","details"} }`

### 5.3 공통 검색 파라미터
```
?year=2026&branchId=xxx&keyword=홍길동&departmentId=&classGroupId=&status=&page=0&size=20&sort=name,asc
```
모든 목록 API는 `year`·`branchId` 필수.

---

## 6. 외부 연동 명세

### 6.1 대성전산 API (디멤버·수납) 🔧
- **용도**: 디멤버 사이트 데이터 조회, 수납 프로그램 연동.
- 🔧 **회원 구조(7/10):** 재원생은 **앱** 사용 / 교사는 **디멤버 사이트**에서 관리 → 연동 시 회원 주체 구분 반영.
- 🔧 **양방향 동기화:** 대성전산 Dlab통합관리프로그램(구 DSA) 계약 종료 전 **병행 운영** 가능 → 단방향 마이그레이션 가정 금지. `SyncAdapter`로 변경분 동기화(충돌 정책·정합성 배치).
- **원칙**: 대성전산 DB 직접 접근 금지, API 레이어 경유. **Circuit Breaker 필수**.
```java
@Component
public class DaesungApiClient {
    @CircuitBreaker(name="daesungApi", fallbackMethod="fallback")
    public SunapData fetchSunapData(String studentId) { ... }
    public void syncMealCancel(MealOrderItem item) { ... }        // 급식 취소 반영
    public void notifyScorePdfUploaded(UUID studentId, String url){...}
}
```
연동 흐름(급식 결제): `PG Webhook → Dlab통합관리프로그램(구 DSA) 저장 → 대성전산 수납 API 자동 반영`.

### 6.2 키오스크 Webhook
- `POST /webhooks/kiosk/attendance` · HMAC-SHA256 서명(지점별 시크릿 `branch_configs`).
- 처리: 서명검증 → `attendance_logs` 저장 → 상태 계산(등원/지각/결석) → 알림톡(학부모) → 앱 반영(FCM/SSE).
- 🆕 **식사 체크 키오스크**: 당일 급식 신청 여부 대조 후 수령 처리(`meal_order_items` 상태 갱신).
- 페이로드(추정, D-2 확인): `{branchId, studentNo, taggedAt, deviceType(ENTRANCE|MEAL), nfcCardId}`.

### 6.3 Zyxel Nebula (방화벽)
- 승인 → `POST Nebula /firewall/whitelist` → 학생 단말 일시 허용 → 만료 시 자동 차단.
- 🔴 E-1: 제어 단위(단말 vs 정책) 확인. 장비: USG Flex 700H + WBE510D/WBE630S + GS1920-24HP.

### 6.4 더프리미엄모의고사 (성적)
- `GET /scores?phone=010xxxx` → `score_records` 저장·동기화. 스펙 Phase 0 확인(E-2).

### 6.5 PG 결제 (이중 구조) 🔧⚠️
```java
interface PaymentGatewayClient {
    VirtualAccountResult issueVirtualAccount(PaymentRequest req);
    CardPaymentResult    requestCardPayment(PaymentRequest req);
    void verifyWebhookSignature(String payload, String signature);
    RefundResult requestRefund(String txId, int amount); // 등록비 전용
}
@Component class TuitionPgClient implements PaymentGatewayClient { /* 학원 명의 */ }
@Component class LunchVendorPgClient implements PaymentGatewayClient {
    // 벤더 명의. 환불은 벤더 직접 → Dlab통합관리프로그램(구 DSA)는 취소요청·기록만(requestRefund 미구현)
}
```
Webhook 분리: `/webhooks/pg/tuition`, `/webhooks/pg/lunch`.
가상계좌 상태: `PENDING → ISSUED → PAID → EXPIRED` + **만료 스케줄러(@Scheduled)**.
- ⚠️ **D-9 등록비 결제 주체**: `TuitionPgClient`(학원 직접 PG) vs 수납=대성전산 상충 → 확정 후 반영.
- 🔧 **급식 가상계좌**: 신청 flow(8.4)에서 카드/가상계좌 분기 + MEAL 만료 처리 연결(11-A).
```sql
payment_transactions (
  id UUID PK, student_id UUID, payment_type VARCHAR(10),   -- TUITION|MEAL
  pg_provider VARCHAR(30), pg_transaction_id VARCHAR(100), amount INTEGER,
  status VARCHAR(20),                                       -- PENDING|ISSUED|PAID|CANCELLED|EXPIRED
  virtual_account_no VARCHAR(30), virtual_account_bank VARCHAR(10),
  expires_at TIMESTAMPTZ, paid_at TIMESTAMPTZ, settled_at TIMESTAMPTZ,
  year INTEGER, branch_id UUID, created_at TIMESTAMPTZ DEFAULT now()
)
```

### 6.6 알림 채널 (알림톡 + FCM 이중화) ✅🔧
| 채널 | 용도 |
|---|---|
| 카카오 알림톡 | 출결·수납 등 **트랜잭션** + 심사 통과 항목 |
| **FCM** | **Daily Report**·앱 내 알림·승인 요청 대부분 |
```java
@Service
public class NotificationService {           // 채널 라우팅 추상화
    public void send(NotifyEvent e) {
        Channel ch = routingPolicy.resolve(e.type()); // KAKAO | FCM | SMS(fallback)
        ...
    }
}
```
Dlab통합관리프로그램(구 DSA) 발송 알림톡 템플릿(예): 출결(등원/지각/결석), 상벌점, 대기자 순번, 미납 안내, 급식 결제완료, 성적 업로드.
🔧 **Daily Report는 FCM 우선**(알림톡 심사 리스크) — 알림톡 병행은 심사 결과로 확정(I-14).

---

## 7. 공통 모듈 설계

### 7.1 동적 검색 빌더 (QueryDSL)
```java
@Component
public class StudentSearchBuilder {
    public BooleanExpression build(StudentSearchCondition c) {
        return Expressions.allOf(yearEq(c.getYear()), branchEq(c.getBranchId()),
            departmentEq(c.getDepartmentId()), classGroupEq(c.getClassGroupId()),
            statusEq(c.getStatus()), keywordLike(c.getKeyword())); // 이름·학번·전화 OR
    }
}
```

### 7.2 엑셀 Import/Export 프레임워크
```java
public interface ExcelExporter<T> { List<String> getHeaders(); List<Object> toRow(T e); String getSheetName(); }
public interface ExcelImporter<T> { T fromRow(Row row); List<ValidationError> validate(T e); }
public record ImportPreviewResult(int totalRows,int validRows,int errorRows,
    List<ValidationError> errors, List<Object> preview) {}
```
🔧 **성적 상세 엑셀 업로드**(8.5/8.10)에서 재사용. **양식 변경 대응**: 헤더 고정 대신 `ColumnMapping`(별칭→필드) 테이블로 유연 매핑(I-11).

### 7.3 연도 초기화 서비스
```java
@Service class YearlySnapshotService {
  @Transactional public void copyFromYear(int src,int tgt,UUID branch){ /* 그래프 순회 복사 */ }
}
```

### 7.4 Webhook 서명 검증
```java
@Component class WebhookSignatureVerifier {
  public void verify(String payload,String sig,String secret){
    String exp=HmacUtils.hmacSha256Hex(secret,payload);
    if(!MessageDigest.isEqual(exp.getBytes(),sig.getBytes())) throw new WebhookSignatureException("서명 불일치");
  }
}
```

### 7.5 🆕 상벌점 규칙 엔진 (ruleengine)
```java
// penalty_rules: trigger(ROUTINE_MISS|PATROL_SLEEP_2|ABSENT|LATE...) → point, reason
@Service
public class PenaltyRuleEngine {
    public void apply(PenaltyTrigger t, Student s) {
        PenaltyRule rule = ruleRepo.findActive(t, s.getBranchId(), s.getYear());
        penaltyService.grant(s, rule.getPoint(), rule.getReason());
        notificationService.send(...);   // 학부모 알림
        dailyReportAggregator.mark(s, t); // Daily Report 반영
    }
}
```

---

## 8. 기능별 구현 명세

### 8.1 학생 관리
```
GET  /api/v1/students            (동적 검색·정렬·조건저장)
POST /api/v1/students            (신규 등록, 학번 자동)
PUT  /api/v1/students/{id}
GET  /api/v1/students/export
POST /api/v1/students/import[/preview]
POST /api/v1/students/{id}/graduate  (대기자→원생)
```
학번 채번: `year + %04d(nextSeq)` → `2026-0001`.

### 8.2 대기자 관리 (신규)
```
GET/POST/DELETE /api/v1/waitlist
POST /api/v1/waitlist/{id}/notify    (순번 처리 알림 — 등록 가능 날짜·시간)
POST /api/v1/waitlist/{id}/message   (개별 메시지)
POST /api/v1/waitlist/{id}/convert   (원생 전환 + 앱 초대 알림)
```
🔧 대기자 원천 = **디멤버 사이트 입학예약폼**(D-7: 폼 수정본 대성 전달). 연동 방식(API/DB) S-1 확정.

### 8.3 출결 관리
```java
@PostMapping("/webhooks/kiosk/attendance")
@Transactional
public void handleAttendance(@RequestBody KioskPayload p){
  signatureVerifier.verify(...);
  Student s = studentRepo.findByNfcCardOrNo(p);          // 카드ID 또는 학번
  AttendanceType type = attendanceCalculator.calc(s, p.getTaggedAt());
  attendanceLogRepo.save(AttendanceLog.of(s,type,p));
  notificationService.send(attendanceEvent(s,type));      // 학부모 알림톡
  penaltyRuleEngine.apply(mapToTrigger(type), s);         // 지각/결석 벌점
}
```
상태 enum: `ON_TIME/LATE/ABSENT/OUT/EXCUSED`. 결석은 스케줄러 배치.

### 8.4 급식 관리 🔧
**정책(mealPolicy) — 주말+공휴일 제외**
```java
@Component
public class MealPolicy {
    // 🔧 주말 + 공휴일 제외 (기존 filterWeekdays만 있었음)
    public List<LocalDate> availableDates(YearMonth month, UUID branchId){
        return month.datesStream()
           .filter(d -> !isWeekend(d))
           .filter(d -> !holidayProvider.isHoliday(d))   // 🆕 공휴일 provider
           .toList();
    }
    public void validateOrderPeriod(YearMonth targetMonth){ /* 월말 신청 기간 */ }
    public void validateCancelDeadline(LocalDate mealDate){ /* 3일 전 */ }
}
```
**신청 — 카드/가상계좌 분기** 🔧
```java
@PostMapping("/api/v1/meals/orders")
@Transactional
public MealOrder createOrder(MealOrderRequest req){
    mealPolicy.validateOrderPeriod(req.getTargetMonth());
    List<LocalDate> dates = mealPolicy.availableDates(req.getTargetMonth(), req.getBranchId());
    PaymentResult pay = switch (req.getPayMethod()) {     // 🔧 분기 (기존 카드만)
        case CARD    -> lunchPgClient.requestCardPayment(toReq(req));
        case VIRTUAL -> lunchPgClient.issueVirtualAccount(toReq(req)); // 발급→ISSUED
    };
    return mealOrderRepo.save(MealOrder.of(student, dates, pay));
}
```
**취소 2트랙** 🔧
```java
@DeleteMapping("/api/v1/meals/orders/{orderId}/items/{itemId}")
public void cancelItem(@PathVariable UUID itemId, @RequestAttribute User user){
    MealOrderItem item = mealItemRepo.findById(itemId);
    if(!user.isAdmin()){
        mealPolicy.validateCancelDeadline(item.getMealDate());  // 앱: 3일 전
        lunchPgClient/*or refundService*/.requestMealRefund(item); // 🔧 앱 취소 시 PG 환불 자동
    } // 데스크(관리자): 기간 제한 없이 즉시
    item.markCancelRequested(user);
    daesungApiClient.syncMealCancel(item);                       // 수납 반영
}
```
**🔧 데스크 당일 신청 (신규·미확정, I-13)**
```
POST /api/v1/meals/orders/desk   (관리자 전용)
```
- 앱(월말 일괄)과 별개 flow. 🟡 결제 방식(PG카드/현금/수납)·환불 주체(PG vs 수납)·**관리자 직접 등록 화면 필요 여부** 확정 후 착수.

**가상계좌 만료 스케줄러**
```java
@Scheduled(cron="0 */10 * * * *")
public void expireVirtualAccounts(){
    paymentRepo.findExpirable(MEAL).forEach(tx -> { tx.expire(); mealOrderService.markUnpaid(tx); });
}
```

### 8.5 성적 관리 🔧 (API + 엑셀 상세)
```java
@Service
public class ScoreService {
    public ScoreResult fetchAndSync(UUID studentId){
        ExamScore s = premiumExamClient.fetchByPhone(studentRepo.findById(studentId).getPhone());
        return ScoreResult.from(scoreRecordRepo.save(ScoreRecord.of(studentId,s)));
    }
    public String uploadScorePdf(UUID studentId, MultipartFile pdf){
        String url = supabaseStorage.upload("scores/"+studentId, pdf);
        daesungApiClient.notifyScorePdfUploaded(studentId, url);  // D.Lab 사이트 연동(S-2)
        return url;
    }
    // 🔧 상세 성적 엑셀 업로드(상담용) — 양식 변경 대응 매핑
    public ImportPreviewResult previewDetail(MultipartFile xlsx){
        return excelService.preview(xlsx, new ScoreDetailImporter(columnMappingRepo.active()));
    }
}
```
- 앱: 과목별 **추이 그래프**만. **상세 분석은 Dlab통합관리프로그램(구 DSA)(담임 상담용)** 에서 표시(8.10 상담 리포트 연계).
- 🖼️ **성적표 구조(0710):** 과목 = 국어·수학·영어·한국사·탐구1·탐구2 / 지표 = **원점수·표준점수·백분위·등급**(+유형) / **회차별**(3~6월 등 월별 모의고사 + 6월 평가원).
- 🖼️ **과목별 상세 분석(0710, 엑셀 업로드):** 성적 추이 그래프 + **내용영역 단원별 정답률** + 영역별(선택/문학/독서 등) **전국 대비 정답률**. `score_detail_uploads`에 원본·매핑 저장, 상담 리포트(8.10)에서 결합.
- 🟡 I-11 엑셀 양식 자체 제작 여부/내년 양식 변경 매핑/ API↔엑셀 경계.

### 8.6 설문 관리 🔧 (템플릿 추가)
```
POST /api/v1/surveys                     (설문 생성 — 템플릿에서 복제 가능)
POST /api/v1/surveys/{id}/publish
GET  /api/v1/surveys/{id}/responses|results
-- 🔧 템플릿
GET/POST /api/v1/survey-templates
POST /api/v1/survey-templates/{id}/instantiate  (템플릿 → 설문 생성)
```
```sql
survey_templates ( id, title, type, questions_json JSONB, branch_id, is_shared )
surveys ( id, title, type, template_id NULL, target_year, branch_id, starts_at, ends_at, is_published )
```

### 8.7 🆕 데일리 루틴 관리
```
GET/POST /api/v1/routines?month=YYYY-MM          (월별 루틴/테스트 정의: 과목·배점·권장)
POST /api/v1/routines/copy-from-month            (전월 복사)
POST /api/v1/routines/{id}/results               (학생/반별 결과: 완료|미완료|점수)
```
```sql
daily_routines ( id, month, subject, title, max_score, is_recommended, penalty_rule_id, year, branch_id )
daily_routine_results ( id, routine_id, student_id, status, score, recorded_by, recorded_at )
```
결과 입력 → `PenaltyRuleEngine.apply()` → 상벌점·Daily Report 자동 연동. 과목 월별 상이 → 월 재설정+복사.

### 8.8 🆕 주·일 학습 계획 관리
```
GET  /api/v1/learning-plans?studentId=&week=      (주간/일간 — 교시×요일 그리드)
PATCH /api/v1/learning-plans/items/{id}           (파란색=선생님 편집 항목만)
GET  /api/v1/learning-plans/compare?studentId=&week=      (계획 대비 실제 집계)
GET  /api/v1/learning-plans/composition?studentId=&week=&breakdown=subject|type  -- 🖼️ 과목/형태별 시간·비율(주차)
GET/POST/PATCH /api/v1/learning-plan-options       -- 🖼️ 선택목록(드롭다운) 마스터 관리(관리자)
```
```sql
learning_plans (
  id, student_id, week_start, year, branch_id,
  weekend_self_required BOOLEAN     -- 🖼️ 주말 의무자습 여부(관리자 토글)
)
learning_plan_items (
  id, plan_id,
  period,                      -- 🖼️ 교시(0교시~야3), 관리자 편집
  time_slot,                   -- 🖼️ 시간(예: 07:40~08:10), 관리자 편집
  weekday,                     -- MON~SUN
  subject,                     -- 국·수·영·탐구·통합사회·통합과학
  study_type,                  -- LESSON(수업)|VOD(인강)|SELF(자습)
  material_text,               -- 🖼️ 교재/학습 내용 기입(예: '드릴 수1 워크북')
  planned_minutes, actual_minutes,
  editable_by                  -- TEACHER(파란색) | STUDENT | AUTO
)
learning_plan_options (        -- 🖼️ 드롭다운 마스터(국자습·국인강·수자습…)
  id, label, subject, study_type, sort_order, branch_id, is_active
)
```
- 🖼️ **레이아웃(0710):** 주간 = **교시×요일 그리드**, 셀 = 과목+학습형태+교재/내용. **관리자 편집**: 교시·시간 수정, 주말 의무자습 토글, 선택목록(드롭다운) 편집.
- **비교 그래프(Recharts)**: 과목별 × 유형별 계획 대비 실제 → 담임 상담.
- 🖼️ **비율 파이(Recharts)**: 과목/학습형태별 시간·비율을 **주차별(1~5주차/계) 필터**로 표시. 앱은 조회(파란색)·학생 입력분만 수정.

### 8.9 🆕 메시지 관리
```
POST /api/v1/notices        (scope: ALL|BRANCH|CLASS|INDIVIDUAL — 권한별)
GET  /api/v1/admin-requests (행정 요청 수신함)
POST /api/v1/chat/threads   (채팅 스레드 — 외부 메신저 참조. type: DIRECT_1TO1 | FAMILY)
```
- **발송 권한**: 전체=SUPER_ADMIN / 지점=BRANCH_ADMIN / 반=TEACHER / 개별=담임·관리자.
- **1:1 채팅**: 외부 유료 메신저 연동(React, 유현님). `chat_threads`에 메타·참조ID만 저장, 본문은 벤더. ISMS 준용(E-6).
- 🖼️ **가족 채팅방(0710):** **학부모+학생+관리자 공동 그룹 스레드**(`type=FAMILY`). 관리자↔가족 연락 채널. 참여자 구성은 학생-학부모 매핑 기준. 외부 메신저 그룹 채팅으로 구현.
- **행정 요청**: 학생이 앱에서 사전정의 항목 클릭 → `admin_requests` 수신 → 담당자 처리. 항목 목록 I-3 확정.

### 8.10 🆕 상담 (일지·리포트 · 🖼️ 0710 구조 확정)
```
GET/POST /api/v1/consult-logs
POST /api/v1/consult-reports        (과목별 이행률·태그·스티커·코멘트 + 성적 상세 + 학습계획 비교)
GET/POST/PATCH /api/v1/consult-tags -- 🖼️ 상담 항목 마스터(과목별, 관리자 편집, max_display)
```
- 🖼️ **상담 리포트 구조(0710):** 담임/최근 상담일 헤더 + **과목별(국·수·영·탐구)** 로:
  - **학습 계획 이행률 = 별점(1~5)** — 관리자 선택.
  - **학습 계획 상담 = 과목별 준비된 항목(태그) 선택형** — `consult_tags` 마스터에서 선택(예: '인강보다 자습'·'자습시간 확보'·'미적분 개념 보완'·'수1 개념 보완'). **관리자 편집 가능·최대 노출 개수(max_display) 설정**.
  - **담임 스티커**(예: '참 잘했어요!') + **담임 코멘트/전달사항**(한마디).
```sql
consult_tags ( id, subject, label, color, sort_order, is_active, branch_id )   -- 마스터
consult_report_subjects (
  id, report_id, subject,
  achievement_rating SMALLINT,        -- 이행률 별점(1~5)
  selected_tag_ids UUID[],            -- 선택 상담 항목
  sticker_code VARCHAR(30),           -- 담임 스티커
  comment TEXT                        -- 과목/총평 코멘트
)
```
- Weekly ABC test(데일리 루틴, 8.7)의 **맞은갯수/총갯수·피드백**, 수강진도(전주 이행도·다음주 계획)도 상담 리포트에 결합 노출.
- 🟡 상담 양식·**리포트 발송 주기** 확정(I-2). 성적 상세 엑셀(8.5)·학습계획 비교(8.8) 데이터 결합.

### 8.11 🆕 승인 라우팅 관리
```sql
approval_items ( id, code, name, approver_type )   -- PARENT | TEACHER | AUTO
approval_requests ( id, item_id, student_id, payload_json, status, assignee_id, decided_by, decided_at )
```
```
GET  /api/v1/approvals?assignee=me
POST /api/v1/approvals/{id}:approve | :reject
```
- 학생 신청(사유·정기일정·방화벽 등) → `approver_type`에 따라 학부모 앱 또는 **선생님(Dlab통합관리프로그램(구 DSA)/관리자 앱)** 로 라우팅.
- 🟡 I-12 항목별 승인 주체 매트릭스 확정이 선결. 방화벽 해제 승인 주체 미결.

### 8.12 🖼️ Daily Report 집계 (서버, 0710)
> 앱 Daily Report(대시보드)의 데이터 원천. 앱은 표시만, 집계·저장은 서버.
```
GET  /api/v1/daily-reports/{studentId}/{date}
GET  /api/v1/daily-reports/{studentId}/monthly?month=YYYY-MM   -- 달력 뷰
PATCH /api/v1/daily-reports/{studentId}/{date}/self-feedback   -- 학생 다짐 한마디
GET  /api/v1/study-time-rankings?scope=ALL|BRANCH&period=DAY|WEEK|MONTH
```
- **순공 랭킹**: `study_time_rankings`에 전체/지점 × 일·주·월 집계(배치 스케줄러). 🟡 순공시간 산출 정의(입퇴실/순찰/좌석없음 차감, I-6).
- **달력 셀**: 일자별 순공시간·출결 상태·질의응답 횟수·데일리테스트(R) 집계.
- **셀프 피드백**: `daily_report_feedback`(student_id, date, text) — 학생 직접 1줄 입력.
- 매일 밤 **FCM 요약 푸시**(알림톡 병행은 심사 결과, I-14).

### 8.13 🖼️ 대면 질의응답·영단어 시험 관리 (0710)
```
-- 대면(OFF) 질의응답 예약
GET/POST /api/v1/qna/offline/slots            (관리자: 상담실 가능 타임 개설)
GET      /api/v1/qna/offline/reservations     (예약 현황)
-- 영단어 시험(디랩 콘텐츠)
GET/POST /api/v1/vocab-tests                  (관리자: 시험 생성·배포)
GET      /api/v1/vocab-tests/{id}/results     (응시 결과 — Daily Report 참고링크 연동)
```
- **질의응답 ON/OFF**: ON=온라인 게시판(멘토 답변) / OFF=대면 예약. 앱은 8.9(APP 문서), 관리는 스케줄·멘토 배정.
- **영단어 시험**: 앱 응시(APP 8.9) → 결과 저장 → Daily Report '디랩 콘텐츠 신청 링크'로 노출.

---

## 9. 보안·인증·권한

### 9.1 인증
```
Dlab통합관리프로그램(구 DSA) 로그인(직원 ID+PW) → Supabase Auth → JWT(Access 1h / Refresh 7d)
요청 Header: Authorization: Bearer {accessToken} → Spring Security 검증
```

### 9.2 RBAC
```java
enum Role { SUPER_ADMIN, BRANCH_ADMIN, TEACHER, STAFF, READONLY }
```
| 메뉴 | SUPER | BRANCH | TEACHER | STAFF |
|---|:-:|:-:|:-:|:-:|
| 학생 관리 | ✅ | ✅ | 담당 반 | ✅ |
| 수납현황 | ✅ | ✅ | ❌ | 조회 |
| 기초설정 | ✅ | ✅ | ❌ | ❌ |
| 사용자 관리 | ✅ | 지점 내 | ❌ | ❌ |
| 성적/상담 | ✅ | ✅ | 담당 반 | ✅ |
| 🆕 학습계획 편집(파란색) | ✅ | ✅ | 담당 반 | ❌ |
| 🆕 공지 발송 | 전체 | 지점 | 반 | ❌ |
> ⚠️ **RBAC 골격은 Phase 0 필수 선행** — 개인정보 노출 메뉴가 Phase 1부터 개발됨.

### 9.3 개인정보 보호
- 전화·주소·생년월일 포함 응답: `BRANCH_ADMIN` 이상. 엑셀 다운로드 마스킹 기본 ON(`010-****-1234`). 민감 필드 권한 없으면 마스킹.
- 성적(민감정보) 저장 암호화, 접속기록 보관·위변조 방지.

### 9.4 📌 ISMS 책임 범위
- **ISMS-P 인증 취득 자체는 학원(내부) 주관**. 개발팀은 **개발 반영 가능한 기술적 보호대책**(RBAC·암호화·마스킹·로그·시큐어코딩·백업)만 책임. 내부 검토 후 요청분 반영.

---

## 10. 배포·인프라

### 10.1 환경
```
dev  : 로컬 Docker Compose + Supabase local
stg  : AWS + Supabase Cloud(테스트)
prod : AWS + Supabase Cloud
```

### 10.2 환경변수 (Secret Manager 주입)
```properties
spring.datasource.url=${SUPABASE_DB_URL} ...
external.daesung.api-url/key=${...}
external.kiosk.webhook-secret=${...}
external.pg.tuition.*/lunch.*=${...}
external.kakao.sender-key=${...}
external.fcm.credentials=${...}
external.nebula.*=${...}
external.premium-exam.*=${...}
external.messenger.*=${...}   # 🆕 1:1 메신저
```

### 10.3 지점별 설정
```sql
branch_configs ( branch_id UUID PK, kiosk_secret TEXT, pg_merchant_code VARCHAR(50),
  nebula_device_id VARCHAR(50), config_json JSONB )
```

---

## 11. 개발 착수 선결 조건

| # | 대상 | 담당 | 상태 |
|---|---|---|---|
| 1 | 대성전산 API 스펙(디멤버·수납) + **양방향 동기화 범위** | 개발 | 🔴 |
| 2 | 키오스크 Webhook 스펙·인증(출결·식사체크) | 개발 | 🔴 |
| 3 | Zyxel Nebula API 권한·제어 단위 | 개발·자이엘 | 🔴 |
| 4 | 더프리미엄 API 스펙 | 개발 | 🔴 |
| 5 | 급식 PG 선정·계약(가상계좌 포함) | 기획 | 🔴 |
| 6 | 🔧 **대성전산 Dlab통합관리프로그램(구 DSA) 계약 종료 시점(컷오버)** | 행정 | 🔴 |
| 7 | D.Lab 디자인 가이드 | 기획 | 🔴 |
| 8 | 교무업무 구분 항목 정리 | 운영 | 🔴 |
| 9 | 카카오 알림톡 채널·템플릿 심사(+Daily Report 채널) | 운영 | 🔴 |
| 10 | 구글 시트 2개 구조·마이그레이션 | 개발·운영 | 🔴 |
| 11 | ⚠️ **등록비 결제 주체(D-9)** | 기획·행정 | 🔴 |
| 12 | 🆕 승인 주체 매트릭스(I-12) | 운영 | 🔴 |
| 13 | 🆕 행정 요청 항목 목록(I-3) | 운영 | 🔴 |
| 14 | 🆕 1:1 메신저 선정(React 연동) | 개발 | 🔴 |
| 15 | 🆕 상벌점 규칙 매핑 정의(I-5) | 운영 | 🔴 |

**확정 완료:** 디멤버·수납 개발주체=대성전산(API) / 급식 결제=벤더 명의 PG 분리 / 방화벽=Zyxel USG Flex 700H / 알림 채널 이중화(알림톡+FCM).

---

## 12. Phase별 개발 범위

### Phase 0 — 사전 준비
- Supabase 스키마 초안, Spring Boot(Security·JPA·QueryDSL), **RBAC 공통 레이어(필수)**, 공통 모듈(검색·엑셀·알림 라우팅·Webhook 검증·**상벌점 규칙 엔진**·**공휴일 provider**), 외부 게이트웨이(Circuit Breaker), PG 어댑터 IF, 알림톡 템플릿 일괄 신청.
- React 세팅·공통 컴포넌트·로그인/권한 분기·디자인 가이드 적용.

### Phase 1 — MVP
학생·대기자·입학예약 상담·출결·상벌점·문자/알림 발송·관리자 기초설정(전년도 복사).

### Phase 2 — 핵심
급식(가상계좌·공휴일·취소 2트랙·데스크 당일)·수납현황(대성 API·등록비 주체 확정 후)·교무업무(명단 엑셀)·특강·수납관리.

### Phase 3 — 교육·성적·소통
성적(더프리미엄+상세 엑셀, 🖼️ 회차별·단원별 정답률)·**상담 리포트(🖼️ 별점·상담 항목 마스터·스티커)**·설문(+템플릿)·**데일리 루틴**·**주·일 학습 계획(🖼️ 교시 그리드·비율 파이·비교 그래프)**·**메시지(공지·행정요청·🖼️ 가족 채팅방)**·**1:1 채팅 연동**·승인 라우팅·**🖼️ Daily Report 집계·대면 질의응답·영단어 시험**.

### Phase 4 — 부가
통계 대시보드·D.Lab 스토어(2027-12)·관리자 대시보드 고도화.

---

## 13. 리스크 및 주의사항

| # | 리스크 | 대응 |
|---|---|---|
| 1 | 대성전산 API 미공개/연동 거부 | 계약 조건 사전 합의, 최악 시 DB 직접 읽기 협의 |
| 2 | 디멤버 연동 범위·회원 구조 불명확 | Phase 0 상세 스펙 미팅 선행 |
| 3 | 알림톡 심사 지연 | 템플릿 전량 선신청, Daily Report는 FCM 우선 |
| 4 | 수납↔급식 결제 동기화 오류 | Webhook 재시도 + 정합성 배치 |
| 5 | RBAC 미완 상태 개인정보 오픈 | Phase 0 완성 필수, 배포 전 보안 검토 |
| 6 | 대성 Dlab통합관리프로그램(구 DSA) 병행 운영 | **양방향 동기화 어댑터**(단방향 가정 금지), 컷오버 계획(D-8) |
| 7 | ⚠️ 등록비 결제 주체 상충 | D-9 확정 전 결제 도메인 스키마 확장 여지 확보 |
| 8 | 급식 가상계좌 흐름 누락 | 8.4 카드/가상계좌 분기·만료 스케줄러 반영 |
| 9 | 승인 라우팅 미확정 | `approver_type` 파라미터화, 매핑만 주입(I-12) |
| 10 | 2개월 납기 | Phase 0·1 집중, 이후 점진 배포 |

---

*본 문서는 개발팀 내부 전용입니다. 확정 시 `Dlab통합관리프로그램(구 DSA)_docx` 리포에 커밋하고 버전·요약을 갱신하세요. 재원생 앱은 `APP_기술_개발문서.md`를 참조합니다.*
