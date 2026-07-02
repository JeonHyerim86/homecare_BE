# 재가가요 (Jaegagayo) — Backend

> 재가 요양 서비스 수요자와 요양보호사, 재가요양기관을 연결하는 **재가 요양 매칭 플랫폼**의 백엔드 서버

<br>

## 📌 프로젝트 개요

**재가가요**는 집에서 요양 서비스를 받고자 하는 수요자(어르신·보호자)가 서비스를 신청하면, 조건에 맞는 요양보호사를 매칭하고 일정·바우처(장기요양급여)·정산·리뷰까지 서비스 이용의 전 과정을 관리하는 플랫폼입니다.

이 레포지토리는 재가가요의 **REST API 백엔드 서버**로, 수요자 앱 · 요양보호사 앱 · 관리자(센터) 웹 세 클라이언트의 API와 도메인 비즈니스 로직 전반을 담당합니다.

| 관련 레포지토리 | 설명 |
|---|---|
| [homecare_consumer](https://github.com/jaegagayo/homecare_consumer) | 수요자(신청자) 앱 |
| [homecare_caregiver](https://github.com/jaegagayo/homecare_caregiver) | 요양보호사 앱 |
| [homecare_admin](https://github.com/jaegagayo/homecare_admin) | 관리자(센터) 웹 |
| [homecare_matching](https://github.com/jaegagayo/homecare_matching) | 매칭 엔진 서버 (Python) |

<br>

## 🛠 기술 스택

| 분류 | 기술 |
|---|---|
| Language | ![Java](https://img.shields.io/badge/Java_21-007396?style=flat-square&logo=openjdk&logoColor=white) |
| Framework | ![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.5.3-6DB33F?style=flat-square&logo=springboot&logoColor=white) |
| ORM · 조회 | ![Spring Data JPA](https://img.shields.io/badge/Spring_Data_JPA-6DB33F?style=flat-square&logo=spring&logoColor=white) ![QueryDSL](https://img.shields.io/badge/QueryDSL_5-0769AD?style=flat-square) |
| Database | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) |
| 인증 · 보안 | ![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=flat-square&logo=springsecurity&logoColor=white) (BCrypt 비밀번호 암호화) |
| API 문서화 | ![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=flat-square&logo=swagger&logoColor=black) (springdoc-openapi) |
| 객체 매핑 · 생산성 | ![MapStruct](https://img.shields.io/badge/MapStruct-DD2C00?style=flat-square) ![Lombok](https://img.shields.io/badge/Lombok-BC4521?style=flat-square) |
| 외부 연동 | ![gRPC](https://img.shields.io/badge/gRPC-244c5a?style=flat-square) (매칭 엔진 서버) · ![Spring WebFlux](https://img.shields.io/badge/WebClient-6DB33F?style=flat-square&logo=spring&logoColor=white) (AI 추천 서버) |
| Infra · Build | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Gradle](https://img.shields.io/badge/Gradle-02303A?style=flat-square&logo=gradle&logoColor=white) |

<br>

## ✨ 주요 기능

### 👵 수요자 (Consumer)
- 회원가입 시 **장기요양등급별 월 급여 한도 바우처 자동 발급** (1등급 1,520,700원 ~ 인지지원등급 573,900원)
- 방문 요양 **서비스 신청** (주소·희망 시간·서비스 유형) 및 신청 내역 관리
- 주간/일별 일정 조회, 메인 페이지 가장 가까운 일정 안내
- **바우처 사용 안내** — 서비스 시간별 요금표 기반 예상 사용액 · 본인부담금(15%) 계산, 월별 사용 내역 조회
- 완료된 서비스에 대한 **리뷰 작성**, 미작성 리뷰 알림
- 만족한 요양보호사에게 **정기(반복) 서비스 제안**, 평점 4.0 이상 매칭에 대한 정기 제안 추천
- 블랙리스트 등록/해제로 원치 않는 요양보호사 매칭 제외

### 🧑‍⚕️ 요양보호사 (Caregiver)
- 회원가입(자격증 등록) → **승인 상태 확인** → 소속 센터 선택 온보딩 흐름
- **근무 희망 조건 등록** (근무 지역·가능 시간·요일·서비스 유형·이동 가능 거리) → 매칭 알고리즘의 입력값
- 일정 목록/상세, 오늘·내일 일정 조회 및 단발 일정 거절
- **정기 제안 승인/거절** — 승인 시 시작~종료 기간의 지정 요일마다 반복 일정 자동 생성
- 소속 센터별 정산 내역, 받은 리뷰·평균 평점 조회

### 🏢 센터 (관리자 웹)
- 요양보호사 검색 · 기관 등록 · 말소, 인사카드/근무상태/서비스유형별 조회
- 기간·날짜·요양보호사별 **일정 관리**
- **정산 관리** — 근무 건별 정산액 산정(시간·거리 기반), 미정산 건 집계, 월별/주간 통계
- 대시보드 — 요양보호사 현황 · 정산 · 근무 상태 지표

### 🔄 매칭 시스템
```
서비스 신청(PENDING) → 후보 필터링(활성 보호사 + 근무시간 중복 제외)
  → 매칭 엔진 스코어링(gRPC/REST) → 매칭 확정(CONFIRMED) + 바우처 차감
  → 근무 완료(COMPLETED) → 정산 생성 → 리뷰 작성 → 정기 제안 추천으로 순환
```
- 도메인 상태 머신 기반 매칭 흐름: `ServiceRequest`(신청) → `ServiceMatch`(확정 일정) → `Settlement`(정산)
- 별도 **매칭 엔진 서버(gRPC)** 및 **AI 추천 서버(WebClient)** 연동 구조

<br>

## 📂 프로젝트 구조

도메인 중심 패키지 구조에 **CQRS 스타일 서비스 분리**(`service/command` 쓰기 · `service/query` 읽기)를 적용했습니다. 컨트롤러는 Swagger 문서용 인터페이스와 구현 클래스로 나뉩니다.

```
homecare/src/main/java/jaega/homecare/
├── domain/
│   ├── users/               # 공통 사용자 계정 · 인증 (역할: 수요자/요양보호사/센터/관리자)
│   ├── consumer/            # 수요자 — 회원 · 신청 · 일정 · 마이페이지
│   ├── caregiver/           # 요양보호사 — 회원 · 승인 · 일정 · 마이페이지
│   ├── caregiverPreference/ # 요양보호사 근무 희망 조건 (매칭 입력값)
│   ├── caregiverCenter/     # 요양보호사 ↔ 센터 소속 관계 · 근무 상태
│   ├── center/              # 센터(기관) — 보호사 관리 · 일정 · 정산 · 대시보드
│   ├── serviceRequest/      # 서비스 신청서 (매칭의 출발점)
│   ├── serviceMatch/        # 매칭 확정 일정 (스케줄 · 정산 · 리뷰 · 바우처의 허브)
│   ├── match/               # 매칭 알고리즘 · 외부 매칭 엔진 연동 (gRPC/WebClient)
│   ├── recurringOffer/      # 정기(반복) 서비스 제안
│   ├── voucher/             # 장기요양 바우처 (등급별 월 급여 한도)
│   ├── voucherUsage/        # 바우처 사용 · 본인부담금 계산
│   ├── settlement/          # 요양보호사 정산
│   ├── review/              # 리뷰 · 평점
│   └── blacklist/           # 수요자의 요양보호사 차단
└── global/                  # 공통 설정(config) · 감사(audit) · 유틸 · 더미 데이터
```

각 도메인은 `controller` / `dto` / `entity` / `mapper` / `repository` / `service(command·query)` 계층으로 구성됩니다.

<br>

## 🚀 시작하기

### 1. 사전 준비
- JDK 21, PostgreSQL

### 2. 환경 변수 설정
`homecare/` 디렉토리에 `.env` 파일을 생성합니다.

```properties
DB_URL=jdbc:postgresql://localhost:5432/homecare
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

### 3. 실행

```bash
cd homecare
./gradlew bootRun
```

- Swagger UI: http://localhost:8080/swagger-ui/index.html
- 개발/시연용 더미 데이터 생성: `POST /api/v1/dummy/generate`

### Docker로 실행

```bash
docker build -t homecare-be .
docker run --env-file homecare/.env -p 8080:8080 homecare-be
```

<br>

## 👩‍💻 나의 담당 역할 — Backend Developer

백엔드 개발자로 참여하여 도메인 설계부터 API 구현, 리팩토링까지 담당했습니다.

- **Center · Caregiver 도메인 초기 설계** — 엔티티 설계, 패키지 구조 세팅, QueryDSL 환경 구성
- **센터(관리자 웹) 기능 개발** — 요양보호사 등록/조회/말소 API, 대시보드 · 정산 내역 API, 요양보호사↔센터 소속 관계(`CaregiverCenter`) 중간 테이블 설계 및 전면 리팩토링
- **일정 조회 기능 전반** — 수요자/요양보호사의 일정 목록 · 상세 · 주간 · 특정일 조회, 메인 페이지 오늘/내일/가장 가까운 일정 API
- **바우처 시스템 설계 · 구현** — `Voucher`/`VoucherUsage` 엔티티 설계, 서비스 시간별 요금표 기반 예상 사용액 · 본인부담금 계산 로직, 바우처 사용 내역 조회 API
- **서비스 신청 API** — `ServiceRequest` 엔티티 및 신청 생성/수정 API 구현
- **리뷰 기능** — 리뷰 생성 · 리뷰 요청 API 구현 및 패키지 구조 리팩토링
- **API 전면 리팩토링** — Center · Settlement 컨트롤러를 API 명세에 맞게 정리 (미사용 엔드포인트 제거, 중복 메소드 병합, 네이밍 일관화)
