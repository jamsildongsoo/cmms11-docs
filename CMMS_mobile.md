# CMMS11 모바일 전략 (Flutter)
작성일: 2025-01-25
최종 업데이트: 2025-01-25

## 1. 프로젝트 구조

### 1.1 전체 구조
```
codex/
├── server/
│   └── cmms11/                    # Spring Boot 백엔드
│       └── src/main/java/com/cmms11/
│           ├── api/mobile/       # 모바일 전용 API
│           └── ...
│
└── mobile/
    └── cmms_mobile/               # Flutter 모바일 앱
        ├── lib/
        │   ├── main.dart
        │   ├── screens/          # 화면
        │   │   ├── home/        # 홈 화면
        │   │   ├── auth/        # 인증 (로그인)
        │   │   ├── memo/        # 메모/게시판
        │   │   ├── approval/    # 결재
        │   │   ├── inspection/  # 점검
        │   │   ├── workorder/   # 작업지시
        │   │   ├── workpermit/  # 작업허가
        │   │   ├── plant/       # 설비 조회
        │   │   └── inventory/   # 재고 조회
        │   ├── services/         # API 서비스
        │   │   ├── api_service.dart
        │   │   ├── auth_service.dart
        │   │   ├── inspection_service.dart
        │   │   ├── workorder_service.dart
        │   │   ├── approval_service.dart
        │   │   └── memo_service.dart
        │   ├── models/           # 데이터 모델
        │   │   ├── auth/
        │   │   ├── inspection/
        │   │   ├── workorder/
        │   │   ├── approval/
        │   │   └── common/
        │   ├── widgets/          # 공통 위젯
        │   │   ├── common_card.dart
        │   │   ├── status_badge.dart
        │   │   ├── file_upload_widget.dart
        │   │   └── custom_app_bar.dart
        │   ├── providers/        # 상태 관리 (Provider)
        │   │   ├── auth_provider.dart
        │   │   ├── inspection_provider.dart
        │   │   └── workorder_provider.dart
        │   └── utils/            # 유틸리티
        │       ├── constants.dart
        │       ├── api_client.dart
        │       ├── secure_storage.dart
        │       └── date_formatter.dart
        ├── android/
        ├── ios/
        ├── pubspec.yaml
        └── README.md
```

## 2. 기능 범위 및 제한사항

### 2.1 지원 기능
- ✅ **조회 기능**: 목록 조회, 상세 조회 (모든 모듈)
- ✅ **결과 입력**: 점검 결과, 작업지시 결과 입력
- ✅ **결재 승인**: 결재 승인/반려, 코멘트 작성
- ✅ **메모/게시판**: 조회, 작성 (간소화된 형태)
- ✅ **파일 첨부**: 사진 촬영 및 첨부

### 2.2 제한사항 (신규 작성 제외)
- ❌ **계획 수립**: 점검 계획, 작업지시 생성은 웹에서만 가능
- ❌ **마스터 등록**: 설비, 재고 마스터 등록은 웹에서만 가능
- ❌ **복잡한 양식**: 다중 항목 입력 등 복잡한 양식은 제외
- ✅ **조회만 가능**: Plant(설비), Inventory(재고)는 조회 전용

### 2.3 대상 모듈
```
모바일 앱 대상 모듈
├── Memo (메모/게시판)      → 조회, 작성 (간소화)
├── Approval (결재)          → 조회, 승인/반려
├── Inspection (점검)        → 조회, 결과 입력
├── WorkOrder (작업지시)     → 조회, 결과 입력
├── WorkPermit (작업허가)    → 조회, 결과 입력
├── Plant (설비)             → 조회 전용
└── Inventory (재고)         → 조회 전용
```

## 3. 백엔드 API 설계

### 3.1 API 경로 구조
```
server/cmms11/src/main/java/com/cmms11/api/mobile/
├── MobileAuthApiController.java        # 인증 API
├── MobileMemoApiController.java        # 메모 API
├── MobileApprovalApiController.java    # 결재 API
├── MobileInspectionApiController.java  # 점검 API
├── MobileWorkOrderApiController.java   # 작업지시 API
├── MobileWorkPermitApiController.java  # 작업허가 API
├── MobilePlantApiController.java       # 설비 API
└── MobileInventoryApiController.java   # 재고 API
```

### 3.2 API 엔드포인트
```
기본 경로: /api/mobile/v1

인증 API
├── POST   /auth/login              # 로그인 (JWT 토큰 발급)
├── POST   /auth/refresh            # 토큰 갱신
└── POST   /auth/logout             # 로그아웃

점검 API
├── GET    /inspection/list         # 점검 목록 (페이징)
├── GET    /inspection/{id}         # 점검 상세
├── POST   /inspection/{id}/start   # 점검 시작
├── POST   /inspection/{id}/result  # 점검 결과 입력
└── POST   /inspection/{id}/complete # 점검 완료

작업지시 API
├── GET    /workorder/list          # 작업지시 목록
├── GET    /workorder/{id}          # 작업지시 상세
├── POST   /workorder/{id}/start    # 작업 시작
├── POST   /workorder/{id}/result   # 작업 결과 입력
└── POST   /workorder/{id}/complete # 작업 완료

결재 API
├── GET    /approval/pending        # 결재 대기 목록
├── GET    /approval/{id}           # 결재 상세
├── POST   /approval/{id}/approve   # 승인
└── POST   /approval/{id}/reject    # 반려

메모 API
├── GET    /memo/list               # 메모 목록
├── GET    /memo/{id}               # 메모 상세
└── POST   /memo                    # 메모 작성 (간소화)

설비/재고 API (조회 전용)
├── GET    /plant/list              # 설비 목록
├── GET    /plant/{id}              # 설비 상세
├── GET    /inventory/list          # 재고 목록
└── GET    /inventory/{id}          # 재고 상세
```

### 3.3 공통 응답 형식
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
  "timestamp": "2025-01-25T10:30:00Z"
}
```

### 3.4 모바일 전용 DTO
```java
// 축소된 DTO (모바일 최적화)
public record InspectionMobileResponse(
    String inspectionId,
    String plantName,
    String inspectionType,
    LocalDate plannedDate,
    String status,
    String assigneeName
) {}

public record InspectionResultRequest(
    String inspectionId,
    List<InspectionItemResult> items,
    String overallNote,
    List<String> photoUrls
) {}
```

## 4. 인증 및 보안

### 4.1 인증 방식
- **JWT 토큰 기반 인증**: Access Token (15분) + Refresh Token (7일)
- **로그인 흐름**:
  1. 사용자 ID/비밀번호로 로그인 → POST `/api/mobile/v1/auth/login`
  2. JWT Access Token + Refresh Token 발급
  3. 모든 API 요청 시 `Authorization: Bearer {token}` 헤더 포함
  4. Access Token 만료 시 Refresh Token으로 갱신

### 4.2 Multi-Company 처리
```
로그인 시 회사코드 입력 방식
├── 기본 회사: C0001 (자동 선택)
├── 멀티 회사: 회사코드 선택 또는 입력
└── 토큰에 companyId 포함
```

### 4.3 보안 조치
- ✅ HTTPS 강제 (TLS 1.2+)
- ✅ 토큰 만료/자동 갱신
- ✅ 기기 분실 시 토큰 폐기 (서버 측 블랙리스트)
- ✅ 민감 정보 암호화 저장 (flutter_secure_storage)
- ✅ 생체 인증 지원 (지문/Face ID, 선택사항)

## 5. Flutter 앱 설계

### 5.1 화면 설계
```
홈 화면 (카드형 메뉴)
├── [점검 관리] → 점검 목록
├── [작업 관리] → 작업지시 목록
├── [결재 관리] → 결재 대기 목록
├── [메모] → 메모 목록
├── [설비 조회] → 설비 목록
└── [재고 조회] → 재고 목록

점검 화면
├── 목록 → 상세 → 결과 입력
├── 필터: 날짜, 상태, 담당자
└── 결과 입력: 체크리스트, 사진 첨부, 특이사항

작업지시 화면
├── 목록 → 상세 → 결과 입력
├── 필터: 날짜, 상태, 우선순위
└── 결과 입력: 작업 내용, 소요 시간, 사진 첨부

결재 화면
├── 대기 목록 → 상세
├── 승인/반려 버튼
└── 코멘트 작성
```

### 5.2 주요 패키지
```yaml
# pubspec.yaml
dependencies:
  flutter: sdk: flutter
  provider: ^6.0.0              # 상태 관리
  dio: ^5.0.0                   # HTTP 클라이언트
  flutter_secure_storage: ^9.0.0  # 토큰 저장
  image_picker: ^1.0.0          # 사진 촬영/선택
  cached_network_image: ^3.3.0  # 이미지 캐싱
  intl: ^0.18.0                 # 날짜 포맷팅
  json_annotation: ^4.8.0       # JSON 직렬화
  flutter_local_notifications: ^16.0.0  # 로컬 알림
  connectivity_plus: ^5.0.0     # 네트워크 상태
  
dev_dependencies:
  flutter_test: sdk: flutter
  build_runner: ^2.4.0
  json_serializable: ^6.7.0
```

### 5.3 상태 관리 (Provider)
```dart
// lib/providers/inspection_provider.dart
class InspectionProvider with ChangeNotifier {
  final InspectionService _service;
  List<Inspection> _inspections = [];
  bool _isLoading = false;
  
  Future<void> loadInspections() async {
    _isLoading = true;
    notifyListeners();
    
    _inspections = await _service.getInspections();
    _isLoading = false;
    notifyListeners();
  }
  
  Future<void> submitResult(String id, InspectionResult result) async {
    await _service.submitResult(id, result);
    await loadInspections();
  }
}
```

## 6. 오프라인 지원 (선택사항)

### 6.1 오프라인 전략
- **임시 저장**: 입력 중인 데이터를 로컬에 저장
- **자동 동기화**: 네트워크 연결 시 자동 서버 전송
- **충돌 방지**: 마지막 업데이트 타임스탬프 비교

### 6.2 로컬 저장소
```dart
// Hive 또는 sqflite 사용
dependencies:
  hive: ^2.2.3
  hive_flutter: ^1.1.0
```

## 7. 성능 및 품질

### 7.1 성능 목표
- ✅ 앱 초기 로딩: 1.5초 이내
- ✅ 화면 전환: 300ms 이내
- ✅ 목록 페이징: 20개 단위
- ✅ 이미지 Lazy Loading 적용
- ✅ 메모리 사용: 150MB 이하

### 7.2 품질 관리
```
테스트
├── 단위 테스트 (Unit Test)
├── 위젯 테스트 (Widget Test)
└── 통합 테스트 (Integration Test)

모니터링
├── Firebase Crashlytics    # 크래시 리포팅
├── Firebase Analytics      # 사용자 행동 분석
└── 로깅: Dio Interceptor로 API 요청/응답 로깅
```

## 8. 개발 로드맵 및 체크리스트

### 8.1 Phase 1: 기반 구축 (2주)
- [x] 프로젝트 구조 설정 (`mobile/cmms_mobile/`)
- [ ] Flutter 프로젝트 초기화
- [ ] API 클라이언트 구현 (Dio)
- [ ] 인증 시스템 구현 (JWT)
- [ ] 공통 위젯 개발
- [ ] 백엔드 `/api/mobile/v1` 스켈레톤 생성

### 8.2 Phase 2: 핵심 기능 (3주)
- [ ] 로그인 화면
- [ ] 홈 화면 (카드형 메뉴)
- [ ] 점검 모듈 (목록, 상세, 결과입력)
- [ ] 작업지시 모듈 (목록, 상세, 결과입력)
- [ ] 결재 모듈 (목록, 상세, 승인/반려)
- [ ] 파일 업로드 (사진 촬영/선택)

### 8.3 Phase 3: 부가 기능 (2주)
- [ ] 메모 모듈
- [ ] 설비/재고 조회
- [ ] 오프라인 임시저장 (선택)
- [ ] 푸시 알림 (선택)
- [ ] 다국어 지원 (선택)

### 8.4 Phase 4: 테스트 및 배포 (1주)
- [ ] 단위/통합 테스트
- [ ] UI/UX 개선
- [ ] 성능 최적화
- [ ] Android APK 빌드
- [ ] iOS IPA 빌드
- [ ] Play Store / App Store 내부 테스트 배포

## 9. 참조 문서
- [CMMS_STRUCTURES.md](./CMMS_STRUCTURES.md) - 백엔드 아키텍처
- [CMMS_PRD.md](./CMMS_PRD.md) - 제품 요구사항
- [CMMS_TABLES.md](./CMMS_TABLES.md) - 데이터 모델