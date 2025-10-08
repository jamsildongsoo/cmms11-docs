# 구매관리시스템 구조 및 아키텍처 (Purchase Management System Structures)

## 1. 시스템 아키텍처 개요

### 1.1 전체 시스템 구조
```
┌─────────────────────────────────────────────────────────────┐
│                    구매관리시스템                              │
├─────────────────────────────────────────────────────────────┤
│  웹 포털 (Vendor Portal)  │  관리자 시스템 (Admin System)    │
│  - 협력사 회원가입        │  - 구매요청 관리                │
│  - 입찰 참가            │  - 입찰 관리                    │
│  - PO 관리              │  - 협력사 관리                  │
│  - 정산 관리            │  - 검수/입고 관리               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    CMMS11 시스템                            │
├─────────────────────────────────────────────────────────────┤
│  Inventory 모듈  │  InventoryTx 모듈  │  Approval 모듈    │
│  - 자재 마스터   │  - 재고 이동      │  - 결재 프로세스   │
│  - 재고 현황     │  - 입고 처리      │  - 승인 워크플로우 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    ERP 시스템                               │
├─────────────────────────────────────────────────────────────┤
│  회계 모듈      │  구매 모듈        │  정산 모듈        │
│  - 전표 생성    │  - 미수금 관리    │  - 세금계산서     │
│  - 원가 계산    │  - 지급 관리      │  - 정산 처리      │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 모듈별 상세 구조

#### 1.2.1 구매요청 모듈
```
구매요청 관리
├── 구매요청 작성
│   ├── 기본 정보 입력
│   ├── 품목별 상세 정보
│   └── 첨부파일 업로드
├── 결재 프로세스
│   ├── 부서장 1차 승인
│   ├── 구매담당자 검토
│   └── 구매팀장 최종 승인
└── 입찰 연계
    ├── 입찰 공고 생성
    └── 입찰 참가업체 관리
```

#### 1.2.2 협력사 관리 모듈
```
협력사 관리
├── 협력사 등록
│   ├── 기본 정보 입력
│   ├── 사업자등록증 업로드
│   └── 분류 정보 설정
├── 승인 프로세스
│   ├── 관리자 검토
│   ├── 승인/반려 처리
│   └── 승인 통지
└── 평가 관리
    ├── 거래 실적 평가
    ├── 품질 평가
    └── 서비스 평가
```

#### 1.2.3 입찰 관리 모듈
```
입찰 관리
├── 입찰 공고
│   ├── 공고 작성
│   ├── 참가업체 선정
│   └── 공고 발행
├── 입찰 참가
│   ├── 입찰서 작성
│   ├── 제출/수정
│   └── 입찰 보증금
└── 입찰 평가
    ├── 기술 평가
    ├── 가격 평가
    └── 낙찰자 선정
```

## 2. API 설계

### 2.1 REST API 구조
```
/api/v1/
├── auth/                    # 인증
│   ├── login
│   ├── logout
│   └── refresh
├── vendors/                 # 협력사 관리
│   ├── GET /               # 목록 조회
│   ├── POST /              # 등록
│   ├── GET /{id}           # 상세 조회
│   ├── PUT /{id}           # 수정
│   ├── DELETE /{id}        # 삭제
│   └── POST /{id}/approve  # 승인
├── purchase-requests/       # 구매요청
│   ├── GET /               # 목록 조회
│   ├── POST /              # 등록
│   ├── GET /{id}           # 상세 조회
│   ├── PUT /{id}           # 수정
│   └── POST /{id}/approve  # 승인
├── bidding/                # 입찰 관리
│   ├── notices/            # 입찰 공고
│   ├── proposals/          # 입찰서
│   └── evaluations/        # 입찰 평가
├── purchase-orders/        # 구매오더
│   ├── GET /               # 목록 조회
│   ├── POST /              # 생성
│   ├── GET /{id}           # 상세 조회
│   └── PUT /{id}/status    # 상태 변경
└── receiving/              # 입고 관리
    ├── GET /               # 목록 조회
    ├── POST /              # 입고 등록
    ├── GET /{id}           # 상세 조회
    └── POST /{id}/inspect  # 검수
```

### 2.2 API 응답 형식
```json
{
  "success": true,
  "data": {
    "content": [...],
    "pageable": {
      "pageNumber": 0,
      "pageSize": 20,
      "totalElements": 100,
      "totalPages": 5
    }
  },
  "message": "조회 성공",
  "timestamp": "2025-01-15T10:30:00Z"
}
```

## 3. 보안 설계

### 3.1 인증/인가
```
JWT 기반 인증
├── Access Token (15분)
├── Refresh Token (7일)
└── Role-based Authorization
    ├── ADMIN (관리자)
    ├── PURCHASER (구매담당자)
    ├── VENDOR (협력사)
    └── USER (일반사용자)
```

### 3.2 데이터 보안
- **암호화**: AES-256 암호화
- **해싱**: BCrypt 패스워드 해싱
- **HTTPS**: SSL/TLS 통신
- **CORS**: Cross-Origin Resource Sharing 설정

## 4. 성능 최적화

### 4.1 캐싱 전략
```
Redis 캐싱
├── 사용자 세션 (30분)
├── 협력사 목록 (1시간)
├── 입찰 공고 목록 (30분)
└── 구매요청 목록 (15분)
```

### 4.2 데이터베이스 최적화
- **Connection Pool**: HikariCP 사용
- **Query Optimization**: 인덱스 활용
- **Pagination**: 페이지네이션 적용
- **Lazy Loading**: 지연 로딩 적용

## 5. 모니터링 및 로깅

### 5.1 로깅 전략
```
로그 레벨별 분류
├── ERROR: 시스템 오류
├── WARN: 경고 사항
├── INFO: 주요 비즈니스 로직
└── DEBUG: 디버깅 정보
```

### 5.2 모니터링 지표
- **시스템 성능**: CPU, Memory, Disk 사용률
- **응답 시간**: API 응답 시간
- **에러율**: 에러 발생률
- **사용자 활동**: 로그인, 페이지 조회

## 6. 배포 전략

### 6.1 환경 구성
```
개발 환경 (Development)
├── 로컬 개발 환경
├── 개발 서버
└── 개발 데이터베이스

테스트 환경 (Testing)
├── 통합 테스트 서버
├── 테스트 데이터베이스
└── 테스트 데이터

운영 환경 (Production)
├── 운영 서버 (Load Balancer)
├── 운영 데이터베이스 (Master-Slave)
└── 백업 시스템
```

### 6.2 CI/CD 파이프라인
```
Git Push → Jenkins → Build → Test → Deploy
├── 코드 품질 검사 (SonarQube)
├── 단위 테스트 실행
├── 통합 테스트 실행
├── Docker 이미지 빌드
└── 배포 (Blue-Green Deployment)
```

## 7. 확장성 고려사항

### 7.1 수평 확장
- **로드 밸런서**: Nginx 또는 AWS ALB
- **마이크로서비스**: 모듈별 서비스 분리
- **데이터베이스 샤딩**: 대용량 데이터 처리

### 7.2 수직 확장
- **서버 스펙 업그레이드**: CPU, Memory 증설
- **데이터베이스 최적화**: 인덱스, 쿼리 튜닝
- **캐시 증설**: Redis 클러스터 구성

## 8. 프로젝트 구조 및 기술 스택

### 8.1 전체 프로젝트 구조 (모노레포)
```
codex/                              # 프로젝트 루트
├── docs/                           # 공통 문서
│   ├── PIMS_PRD.md                # PIMS 제품 요구사항
│   ├── PIMS_STRUCTURES.md         # PIMS 기술 아키텍처 (본 문서)
│   ├── PIMS_TABLES.md             # PIMS 데이터 모델
│   ├── CMMS_PRD.md                # CMMS 제품 요구사항
│   └── CMMS_STRUCTURES.md         # CMMS 기술 아키텍처
│
├── server/                         # 백엔드 서버 프로젝트
│   ├── pims11/                    # PIMS 백엔드 (Spring Boot)
│   │   ├── src/main/java/com/pims11/
│   │   │   ├── Pims11Application.java
│   │   │   ├── config/           # 설정
│   │   │   ├── security/         # 보안
│   │   │   ├── web/              # 웹 컨트롤러
│   │   │   ├── api/              # REST API
│   │   │   │   ├── mobile/      # 모바일 전용 API
│   │   │   │   └── v1/          # 일반 API
│   │   │   ├── vendor/           # 협력사 관리
│   │   │   ├── purchase/         # 구매요청
│   │   │   ├── bidding/          # 입찰 관리
│   │   │   ├── po/               # 구매오더
│   │   │   ├── receiving/        # 입고 관리
│   │   │   ├── settlement/       # 정산 관리
│   │   │   ├── evaluation/       # 평가 관리
│   │   │   └── common/           # 공통 모듈
│   │   ├── src/main/resources/
│   │   │   ├── application.yml
│   │   │   ├── messages/
│   │   │   ├── templates/        # Thymeleaf 템플릿
│   │   │   └── static/           # 정적 리소스
│   │   ├── build.gradle
│   │   └── README.md
│   │
│   └── cmms11/                    # CMMS 백엔드 (Spring Boot)
│       └── (기존 CMMS 구조)
│
└── mobile/                         # 모바일 앱 프로젝트
    ├── pims_mobile/               # PIMS Flutter 앱 (선택)
    │   ├── lib/
    │   │   ├── main.dart
    │   │   ├── screens/          # 화면
    │   │   │   ├── vendor/       # 협력사 화면
    │   │   │   ├── bidding/      # 입찰 화면
    │   │   │   ├── po/           # PO 화면
    │   │   │   └── receiving/    # 입고 화면
    │   │   ├── services/         # API 서비스
    │   │   ├── models/           # 데이터 모델
    │   │   ├── widgets/          # 공통 위젯
    │   │   └── utils/            # 유틸리티
    │   ├── android/
    │   ├── ios/
    │   └── pubspec.yaml
    │
    └── cmms_mobile/               # CMMS Flutter 앱
        └── (기존 CMMS 모바일 구조)
```

### 8.2 PIMS 백엔드 상세 구조
```
server/pims11/
├── src/main/java/com/pims11/
│   ├── Pims11Application.java
│   │
│   ├── config/                         # 설정
│   │   ├── SecurityConfig.java         # JWT 인증, RBAC
│   │   ├── WebConfig.java              # CORS, 정적 리소스
│   │   ├── RedisConfig.java            # Redis 캐시 설정
│   │   └── AppConfig.java              # 애플리케이션 설정
│   │
│   ├── security/                       # 보안
│   │   ├── JwtTokenProvider.java      # JWT 토큰 생성/검증
│   │   ├── JwtAuthenticationFilter.java
│   │   └── UserDetailsServiceImpl.java
│   │
│   ├── web/                            # 웹 컨트롤러 (관리자용)
│   │   ├── VendorController.java      # 협력사 관리
│   │   ├── PurchaseController.java    # 구매요청 관리
│   │   ├── BiddingController.java     # 입찰 관리
│   │   ├── POController.java          # 구매오더 관리
│   │   └── ReceivingController.java   # 입고 관리
│   │
│   ├── api/                            # REST API 컨트롤러
│   │   ├── mobile/                    # 모바일 전용 API
│   │   │   └── (필요시 추가)
│   │   │
│   │   └── v1/                        # 일반 REST API
│   │       ├── VendorApiController.java
│   │       ├── PurchaseApiController.java
│   │       ├── BiddingApiController.java
│   │       ├── POApiController.java
│   │       └── ReceivingApiController.java
│   │
│   ├── vendor/                         # 협력사 관리
│   │   ├── Vendor.java
│   │   ├── VendorRepository.java
│   │   └── VendorService.java
│   │
│   ├── purchase/                       # 구매요청
│   │   ├── PurchaseRequest.java
│   │   ├── PurchaseRequestItem.java
│   │   └── PurchaseRequestService.java
│   │
│   ├── bidding/                        # 입찰 관리
│   │   ├── BiddingNotice.java         # 입찰 공고
│   │   ├── BiddingProposal.java       # 입찰서
│   │   ├── BiddingEvaluation.java     # 입찰 평가
│   │   └── BiddingService.java
│   │
│   ├── po/                             # 구매오더 (Purchase Order)
│   │   ├── PurchaseOrder.java
│   │   ├── PurchaseOrderItem.java
│   │   └── POService.java
│   │
│   ├── receiving/                      # 입고 관리
│   │   ├── Receiving.java
│   │   ├── ReceivingItem.java
│   │   └── ReceivingService.java
│   │
│   ├── settlement/                     # 정산 관리
│   │   ├── Settlement.java
│   │   └── SettlementService.java
│   │
│   ├── evaluation/                     # 평가 관리
│   │   ├── VendorEvaluation.java
│   │   └── EvaluationService.java
│   │
│   ├── integration/                    # 시스템 연계
│   │   ├── cmms/                      # CMMS 연계
│   │   │   ├── CmmsInventoryClient.java
│   │   │   └── CmmsApprovalClient.java
│   │   └── erp/                       # ERP 연계
│   │       ├── ErpAccountingClient.java
│   │       └── ErpPaymentClient.java
│   │
│   └── common/                         # 공통 모듈
│       ├── seq/                       # 자동번호 채번
│       ├── file/                      # 파일 관리
│       ├── error/                     # 예외 처리
│       ├── audit/                     # 감사 로그
│       └── notification/              # 알림
│
├── src/main/resources/
│   ├── application.yml                 # 기본 설정
│   ├── application-dev.yml             # 개발 환경
│   ├── application-prod.yml            # 운영 환경
│   │
│   ├── messages/                       # 국제화
│   │   ├── messages_ko.properties
│   │   └── messages_en.properties
│   │
│   ├── db/migration/                   # Flyway 마이그레이션
│   │   └── V1__baseline.sql
│   │
│   ├── templates/                      # Thymeleaf 템플릿
│   │   ├── layout/
│   │   ├── vendor/
│   │   ├── purchase/
│   │   ├── bidding/
│   │   ├── po/
│   │   └── receiving/
│   │
│   └── static/                         # 정적 리소스
│       ├── assets/
│       │   ├── css/
│       │   └── js/
│       └── favicon.ico
│
├── build.gradle
├── settings.gradle
└── README.md
```

### 8.3 백엔드 기술 스택
```
Spring Boot 3.x
├── Spring Security (JWT 인증/인가)
├── Spring Data JPA (데이터 접근)
├── Spring Web (REST API)
├── Spring Validation (데이터 검증)
└── Spring Boot Actuator (모니터링)

Database
├── MariaDB 10.6+ (메인 데이터베이스)
├── Redis (캐시 및 세션)
└── HikariCP (커넥션 풀)

Build & Deploy
├── Gradle (빌드 도구)
├── Docker (컨테이너화)
├── Jenkins (CI/CD)
└── SonarQube (코드 품질)
```

### 8.4 프론트엔드 기술 스택
```
관리자 시스템 (Web)
├── HTML5 + Vanilla JS (SPA-like)
├── 또는 Vue.js 3.x (선택)
├── Thymeleaf (서버 사이드 템플릿)
└── Responsive CSS

협력사 포털 (Web)
├── HTML5 + Vanilla JS
├── Bootstrap 또는 Tailwind CSS
└── 반응형 웹 디자인

모바일 앱 (선택)
├── Flutter 3.x
├── Dart
└── Provider 또는 Riverpod (상태 관리)
```

### 8.5 인프라 기술 스택
```
Server
├── Linux Ubuntu 20.04+
├── Nginx (웹 서버, 리버스 프록시)
└── Docker (컨테이너)

Cloud Services (선택)
├── AWS S3 또는 MinIO (파일 스토리지)
├── AWS RDS 또는 자체 DB 서버
├── AWS ElastiCache 또는 자체 Redis
└── AWS CloudWatch (모니터링)
```

## 9. 시스템 연계 구조

### 9.1 CMMS11 연계 구조
```
구매관리시스템 → CMMS11 연계
├── Inventory 모듈 연계
│   ├── 자재 마스터 조회 API
│   ├── 재고 현황 조회 API
│   └── 자재 정보 동기화
├── InventoryTx 모듈 연계
│   ├── 재고 이동 등록 API
│   ├── 입고 처리 API
│   └── 재고 수량 업데이트
├── Approval 모듈 연계
│   ├── 결재 라인 조회 API
│   ├── 결재 상태 동기화
│   └── 결재 완료 알림
└── Member 모듈 연계
    ├── 사용자 정보 조회 API
    ├── 부서 정보 조회 API
    └── 권한 정보 연계
```

### 9.2 ERP 연계 구조
```
구매관리시스템 → ERP 연계
├── 회계 모듈 연계
│   ├── 구매 전표 생성 API
│   ├── 지급 전표 생성 API
│   └── 원가 계산 연계
├── 구매 모듈 연계
│   ├── 미수금 관리 API
│   ├── 지급 관리 API
│   └── 계약 정보 연계
└── 정산 모듈 연계
    ├── 세금계산서 발행 API
    ├── 정산 처리 API
    └── 정산 완료 알림
```

## 10. 데이터 플로우

### 10.1 구매 프로세스 데이터 플로우
```
구매요청 데이터 플로우
1. 사용자 입력 → 구매요청 테이블
2. 결재 프로세스 → 결재 테이블
3. 입찰 연계 → 입찰 공고 테이블
4. 입찰 참가 → 입찰서 테이블
5. 낙찰 선정 → 구매오더 테이블
6. 입고 처리 → 입고 테이블
7. 재고 반영 → CMMS11 연계
8. 정산 처리 → ERP 연계
```

### 10.2 협력사 관리 데이터 플로우
```
협력사 관리 데이터 플로우
1. 협력사 등록 → 협력사 테이블
2. 관리자 승인 → 승인 상태 업데이트
3. 입찰 참가 → 입찰 참가 테이블
4. 거래 실적 → 구매오더 테이블
5. 평가 처리 → 평가 테이블
6. 등급 관리 → 협력사 테이블 업데이트
```

## 11. 에러 처리 및 예외 상황

### 11.1 시스템 에러 처리
```
에러 처리 구조
├── 글로벌 예외 처리기
│   ├── 비즈니스 예외 처리
│   ├── 시스템 예외 처리
│   └── 검증 예외 처리
├── 에러 응답 형식
│   ├── 에러 코드
│   ├── 에러 메시지
│   └── 상세 정보
└── 로깅 및 모니터링
    ├── 에러 로그 기록
    ├── 알림 발송
    └── 대시보드 표시
```

### 11.2 비즈니스 예외 처리
```
비즈니스 예외 유형
├── 구매요청 예외
│   ├── 권한 부족
│   ├── 예산 초과
│   └── 결재 불가
├── 입찰 예외
│   ├── 참가 자격 부족
│   ├── 제출 기간 초과
│   └── 평가 불가
└── 입고 예외
    ├── 검수 불합격
    ├── 수량 부족
    └── 품질 문제
```

## 12. 시스템 설정 및 구성

### 12.1 애플리케이션 설정
```yaml
# application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/purchase_system
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
  
  redis:
    host: ${REDIS_HOST}
    port: ${REDIS_PORT}
    password: ${REDIS_PASSWORD}
  
  security:
    jwt:
      secret: ${JWT_SECRET}
      expiration: 900000 # 15분
      refresh-expiration: 604800000 # 7일

server:
  port: 8080
  servlet:
    context-path: /api/v1

logging:
  level:
    com.cmms11.purchase: DEBUG
    org.springframework.security: DEBUG
```

### 12.2 환경별 설정
```yaml
# application-dev.yml (개발환경)
spring:
  datasource:
    url: jdbc:mysql://dev-db:3306/purchase_system_dev
  redis:
    host: dev-redis
  profiles:
    active: dev

# application-prod.yml (운영환경)
spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/purchase_system
  redis:
    host: prod-redis-cluster
  profiles:
    active: prod
```
