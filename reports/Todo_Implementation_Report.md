# 소담 프로젝트 기능 구현 현황 보고서

이 문서는 '소담' 프로젝트의 기능 구현 현황을 상세하게 정리한 보고서입니다. 각 기능별로 구현 완료된 기능의 동작 프로세스와 부분적으로 구현된 기능의 현재 상태 및 추가 구현 필요사항을 기술합니다.

## 1. 사용자 인증 및 계정 관리

### 구현 완료된 기능

#### 1.1 일반 회원가입 (이메일/비밀번호)

- **구현 클래스**: `UserService`
- **동작 프로세스**:
    1. 클라이언트가 이메일, 비밀번호, 이름을 포함한 회원가입 요청을 전송
    2. 서버는 비밀번호를 BCrypt로 암호화하여 저장
    3. 사용자에게 NORMAL 권한을 부여하고 생성 시간 기록
    4. 사용자 정보를 데이터베이스에 저장
    5. 성공 응답 반환

#### 1.2 일반 로그인 (이메일/비밀번호)

- **구현 클래스**: `LoginController`, `UserService`
- **동작 프로세스**:
    1. 클라이언트가 이메일과 비밀번호로 로그인 요청
    2. 서버는 이메일로 사용자 조회
    3. BCrypt를 사용하여 비밀번호 검증
    4. 인증 성공 시 JWT 토큰 생성
    5. Redis에 토큰 저장 (만료 시간: 600초)
    6. 쿠키에 JWT 토큰 설정
    7. 토큰, 사용자 ID, 권한 정보를 응답으로 반환

#### 1.3 카카오 소셜 로그인

- **구현 클래스**: `LoginController`, `KakaoAuthService`
- **동작 프로세스**:
    1. 클라이언트가 카카오 인증 코드를 서버에 전송
    2. 서버는 인증 코드로 카카오 API에 액세스 토큰 요청
    3. 액세스 토큰으로 카카오 API에서 사용자 정보 조회
    4. 이메일로 기존 사용자 확인, 없으면 신규 사용자 생성
    5. JWT 토큰 생성 및 Redis에 저장
    6. 쿠키에 JWT 토큰 설정
    7. 토큰, 사용자 ID, 권한 정보를 응답으로 반환

#### 1.4 토큰 관리 (JWT)

- **구현 클래스**: `JwtTokenProvider`, `TokenService`
- **동작 프로세스**:
    1. 사용자 인증 성공 시 JWT 토큰 생성
    2. 토큰에 사용자 ID, 이메일, 권한 정보 포함
    3. 토큰에 만료 시간 설정
    4. 토큰 서명을 위한 비밀 키 사용
    5. 토큰 검증 시 서명 확인 및 만료 여부 확인

#### 1.5 Redis를 이용한 토큰 저장 및 관리

- **구현 클래스**: `RedisService`
- **동작 프로세스**:
    1. 토큰 생성 시 SHA-256으로 해시하여 Redis에 저장
    2. 사용자 ID를 키로 사용하여 토큰 저장
    3. 토큰에 만료 시간 설정 (TTL)
    4. 토큰 검증 시 Redis에서 해시된 토큰 조회
    5. 로그아웃 시 Redis에서 토큰 삭제

#### 1.6 쿠키 기반 인증 처리

- **구현 클래스**: `TokenService`
- **동작 프로세스**:
    1. JWT 토큰을 쿠키에 저장
    2. HttpOnly 속성 설정으로 JavaScript 접근 방지
    3. 쿠키 경로를 "/"로 설정하여 전체 도메인에서 사용 가능
    4. 쿠키 만료 시간 설정
    5. 로그아웃 시 만료된 쿠키 생성하여 기존 쿠키 무효화

#### 1.7 사용자 권한 관리

- **구현 클래스**: `User` (도메인 모델)
- **동작 프로세스**:
    1. 사용자 생성 시 기본 권한은 NORMAL
    2. 사장으로 변경 시 MASTER 권한 부여
    3. 직원으로 변경 시 EMPLOYEE 권한 부여
    4. 권한에 따라 접근 가능한 API 제한

### 부분 구현된 기능

#### 1.8 일반 유저가 사장으로 변경 시 추가정보 입력

- **현재 구현 상태**:
    - 사용자 권한을 MASTER로 변경하는 기능 구현
    - MasterProfile 엔티티 생성 및 사용자와 연결
    - 사업자 등록번호 저장 기능 구현
- **추가 구현 필요사항**:
    - 사업자 등록번호 유효성 검증 로직
    - 추가 사장 정보 입력 UI/UX
    - 사업자 등록증 이미지 업로드 및 검증

#### 1.9 일반 유저가 직원으로 변경 시 추가정보 입력

- **현재 구현 상태**:
    - 사용자 권한을 EMPLOYEE로 변경하는 기능 구현
    - EmployeeProfile 엔티티 생성 및 사용자와 연결
- **추가 구현 필요사항**:
    - 직원 추가 정보(연락처, 주소 등) 입력 기능
    - 직원 프로필 이미지 업로드
    - 직원 계약 정보 관리

### 미구현 기능

#### 비밀번호 재설정 기능

- 이메일 인증을 통한 비밀번호 재설정 링크 발송 기능
- 비밀번호 재설정 페이지 구현
- 새 비밀번호 유효성 검사 및 저장 기능

#### 계정 삭제 기능

- 사용자 계정 비활성화 기능
- 개인 정보 익명화 처리
- 관련 데이터 보존 정책 적용

#### 소셜 로그인 추가(구글, 네이버)

- OAuth2 클라이언트 설정
- 사용자 프로필 정보 매핑
- 기존 계정과 연동 기능

## 2. 매장 관리

```java
/**
 * TODO: 매장-직원 대량 등록 기능
 * - CSV/Excel 파일 업로드 지원
 * - 데이터 유효성 검증
 * - 일괄 처리 및 오류 보고
 */
public class BulkEmployeeRegistrationService {
    // 구현 필요
}

/**
 * TODO: 위치 기반 서비스 정확도 개선
 * - 위치 데이터 정확도 향상
 * - 지오펜싱 기능 최적화
 * - 위치 기반 알림 기능
 */
public class LocationService {
    // 구현 필요
}

/**
 * TODO: 매장 이미지 다중 업로드 기능
 * - 다중 이미지 업로드 UI/API
 * - 이미지 저장 및 관리
 * - 이미지 최적화 및 CDN 연동
 */
public class StoreImageService {
    // 구현 필요
}

/**
 * TODO: 매장 영업 시간 관리 기능
 * - 요일별 영업 시간 설정
 * - 특별 영업일/휴무일 관리
 * - 영업 시간 변경 이력 관리
 */
public class BusinessHoursService {
    // 구현 필요
}

/**
 * TODO: 매장 통계 데이터 시각화 API
 * - 매출/방문객/직원 통계 데이터 수집
 * - 데이터 분석 및 가공
 * - 시각화 데이터 제공 API
 */
public class StoreAnalyticsService {
    // 구현 필요
}
```

## 3. 근태 관리

```java
/**
 * TODO: 근태 일괄 수정 기능
 * - 다중 근태 기록 선택 및 수정
 * - 수정 이력 관리
 * - 권한 기반 접근 제어
 */
public class BulkAttendanceModificationService {
    // 구현 필요
}

/**
 * TODO: 위치 기반 출퇴근 정확도 개선
 * - GPS 정확도 향상 알고리즘
 * - 위치 오차 보정 기능
 * - 위치 인증 강화
 */
public class LocationBasedAttendanceService {
    // 구현 필요
}

/**
 * TODO: 휴가 유형 추가
 * - 다양한 휴가 유형 정의 및 관리
 * - 휴가 신청/승인 프로세스
 * - 휴가 잔여일수 계산 및 관리
 */
public class TimeOffTypeService {
    // 구현 필요
}

/**
 * TODO: 근무 일정 관리 기능
 * - 근무 일정 생성 및 관리
 * - 일정 충돌 감지
 * - 일정 변경 알림
 */
public class WorkScheduleService {
    // 구현 필요
}

/**
 * TODO: 대체 근무자 지정 기능
 * - 대체 근무자 검색 및 지정
 * - 대체 근무 요청/승인 프로세스
 * - 대체 근무 이력 관리
 */
public class SubstituteWorkerService {
    // 구현 필요
}

/**
 * TODO: 근태 이상 알림 기능
 * - 지각/결근/조기퇴근 감지
 * - 알림 설정 및 관리
 * - 알림 전송 (이메일, 푸시, SMS)
 */
public class AttendanceAlertService {
    // 구현 필요
}
```

## 4. 급여 관리

```java
/**
 * TODO: 급여 명세서 PDF 출력 기능
 * - PDF 템플릿 디자인
 * - 데이터 매핑 및 PDF 생성
 * - 다운로드 및 이메일 전송 기능
 */
public class PayrollPdfService {
    // 구현 필요
}

/**
 * TODO: 급여 일괄 처리 기능
 * - 다중 급여 계산 및 처리
 * - 일괄 승인 프로세스
 * - 처리 결과 보고
 */
public class BulkPayrollProcessingService {
    // 구현 필요
}

/**
 * TODO: 세금 계산 정확도 개선
 * - 최신 세금 정책 반영
 * - 복잡한 세금 계산 규칙 구현
 * - 세금 계산 검증 및 테스트
 */
public class TaxCalculationService {
    // 구현 필요
}

/**
 * TODO: 급여 이체 연동 기능
 * - 은행 API 연동
 * - 이체 요청 및 처리
 * - 이체 결과 확인 및 기록
 */
public class PayrollTransferService {
    // 구현 필요
}

/**
 * TODO: 연말정산 지원 기능
 * - 연간 급여 데이터 집계
 * - 세금 공제 항목 관리
 * - 연말정산 보고서 생성
 */
public class YearEndTaxService {
    // 구현 필요
}

/**
 * TODO: 급여 통계 리포트 기능
 * - 급여 데이터 분석
 * - 통계 리포트 생성
 * - 데이터 시각화
 */
public class PayrollAnalyticsService {
    // 구현 필요
}
```

## 5. 정책 및 정보 관리

```java
/**
 * TODO: 정책 변경 알림 기능
 * - 정책 변경 감지
 * - 영향 받는 사용자 식별
 * - 알림 생성 및 전송
 */
public class PolicyChangeNotificationService {
    // 구현 필요
}

/**
 * TODO: 맞춤형 정책 추천 기능
 * - 사용자 프로필 분석
 * - 관련 정책 매칭
 * - 개인화된 추천 제공
 */
public class PolicyRecommendationService {
    // 구현 필요
}

/**
 * TODO: 정책 정보 검색 기능 강화
 * - 전문 검색 엔진 통합
 * - 검색 결과 관련성 향상
 * - 검색 필터 및 정렬 기능
 */
public class PolicySearchService {
    // 구현 필요
}
```

## 6. 시스템 최적화

```java
/**
 * TODO: 데이터베이스 추가 인덱싱
 * - 성능 병목 지점 분석
 * - 최적의 인덱스 설계
 * - 인덱스 성능 테스트
 */
public class DatabaseIndexingService {
    // 구현 필요
}

/**
 * TODO: API 응답 추가 최적화
 * - 응답 데이터 최소화
 * - JSON 직렬화 최적화
 * - 캐싱 전략 개선
 */
public class ApiOptimizationService {
    // 구현 필요
}

/**
 * TODO: 이미지 처리 개선
 * - 이미지 압축 알고리즘 개선
 * - 이미지 포맷 최적화
 * - 이미지 CDN 연동
 */
public class ImageProcessingService {
    // 구현 필요
}

/**
 * TODO: 상세 모니터링 구현
 * - 애플리케이션 성능 모니터링
 * - 사용자 행동 분석
 * - 오류 추적 및 알림
 */
public class MonitoringService {
    // 구현 필요
}

/**
 * TODO: CDN 도입
 * - CDN 서비스 선택 및 설정
 * - 정적 자원 배포 자동화
 * - 캐시 무효화 전략
 */
public class CdnIntegrationService {
    // 구현 필요
}

/**
 * TODO: 비동기 처리 확대
 * - 비동기 처리 대상 작업 식별
 * - 메시지 큐 시스템 도입
 * - 비동기 작업 모니터링
 */
public class AsyncProcessingService {
    // 구현 필요
}

/**
 * TODO: 읽기 전용 복제본 구성
 * - 데이터베이스 복제 설정
 * - 읽기/쓰기 분리 로직 구현
 * - 복제 지연 모니터링
 */
public class ReadReplicaService {
    // 구현 필요
}
```

## 7. 웹 프론트엔드

```javascript
/**
 * TODO: 직원 마이페이지 추가 기능
 * - 개인 통계 대시보드
 * - 알림 설정 관리
 * - 근무 이력 상세 조회
 */
function EmployeeDashboardComponent() {
    // 구현 필요
}

/**
 * TODO: 매장 관리 UI 개선
 * - 사용자 경험 최적화
 * - 반응형 디자인 적용
 * - 데이터 시각화 개선
 */
function StoreManagementComponent() {
    // 구현 필요
}

/**
 * TODO: 근태 관리 캘린더 뷰 개선
 * - 직관적인 캘린더 인터페이스
 * - 드래그 앤 드롭 일정 관리
 * - 필터링 및 검색 기능
 */
function AttendanceCalendarComponent() {
    // 구현 필요
}

/**
 * TODO: 급여 관리 상세 보기 및 출력 기능
 * - 상세 급여 정보 표시
 * - 인쇄 최적화 뷰
 * - PDF 다운로드 기능
 */
function PayrollDetailComponent() {
    // 구현 필요
}

/**
 * TODO: 정책 정보 검색 기능
 * - 실시간 검색 기능
 * - 검색 결과 하이라이팅
 * - 필터링 및 정렬 옵션
 */
function PolicySearchComponent() {
    // 구현 필요
}

/**
 * TODO: 반응형 디자인 개선
 * - 모바일 최적화
 * - 다양한 화면 크기 지원
 * - 터치 인터페이스 최적화
 */
function ResponsiveDesignSystem() {
    // 구현 필요
}

/**
 * TODO: 다크 모드 지원
 * - 테마 전환 시스템
 * - 다크 모드 색상 팔레트
 * - 사용자 설정 저장
 */
function ThemeToggleSystem() {
    // 구현 필요
}

/**
 * TODO: 접근성 개선
 * - 스크린 리더 지원
 * - 키보드 네비게이션
 * - 색상 대비 최적화
 */
function AccessibilityEnhancements() {
    // 구현 필요
}

/**
 * TODO: 데이터 시각화 강화
 * - 차트 및 그래프 컴포넌트
 * - 실시간 데이터 업데이트
 * - 인터랙티브 시각화
 */
function DataVisualizationComponents() {
    // 구현 필요
}
```

## 8. 모바일 앱

```kotlin
/**
 * TODO: 소셜 로그인 완성
 * - 구글 로그인 통합
 * - 네이버 로그인 통합
 * - 로그인 상태 관리 개선
 */
class SocialLoginManager {
    // 구현 필요
}

/**
 * TODO: 사장 마이페이지 기능 완성
 * - 매장 성과 대시보드
 * - 직원 관리 기능
 * - 알림 센터
 */
class OwnerDashboardScreen {
    // 구현 필요
}

/**
 * TODO: 직원 마이페이지 추가 기능
 * - 근무 일정 관리
 * - 급여 내역 조회
 * - 휴가 신청 및 관리
 */
class EmployeeDashboardScreen {
    // 구현 필요
}

/**
 * TODO: 위치 기반 출퇴근 기능 개선
 * - GPS 정확도 향상
 * - 오프라인 모드 지원
 * - 위치 인증 보안 강화
 */
class LocationBasedAttendanceManager {
    // 구현 필요
}

/**
 * TODO: 급여 조회 상세 정보 표시 개선
 * - 급여 내역 상세 화면
 * - 그래프 및 차트 시각화
 * - 세금 및 공제 항목 상세 표시
 */
class PayrollDetailScreen {
    // 구현 필요
}

/**
 * TODO: 알림 기능 완성
 * - 푸시 알림 시스템
 * - 알림 설정 관리
 * - 알림 이력 관리
 */
class NotificationManager {
    // 구현 필요
}

/**
 * TODO: 오프라인 지원 완성
 * - 로컬 데이터 저장
 * - 동기화 메커니즘
 * - 충돌 해결 전략
 */
class OfflineSupportManager {
    // 구현 필요
}

/**
 * TODO: 생체 인증 로그인
 * - 지문 인증
 * - 얼굴 인증
 * - 보안 토큰 관리
 */
class BiometricAuthManager {
    // 구현 필요
}

/**
 * TODO: 푸시 알림 구현
 * - FCM/APNS 연동
 * - 알림 토큰 관리
 * - 알림 콘텐츠 관리
 */
class PushNotificationManager {
    // 구현 필요
}

/**
 * TODO: 위젯 지원
 * - 홈 화면 위젯 개발
 * - 실시간 정보 업데이트
 * - 위젯 설정 관리
 */
class AppWidgetProvider {
    // 구현 필요
}

/**
 * TODO: iOS 지원 강화
 * - iOS 네이티브 기능 활용
 * - 애플 디자인 가이드라인 준수
 * - iOS 특화 기능 구현
 */
class IosEnhancementManager {
    // 구현 필요
}
```

## 9. 인프라 및 배포 환경

```java
/**
 * TODO: AWS 인프라 추가 서비스 구성
 * - 오토 스케일링 설정
 * - 로드 밸런서 최적화
 * - 클라우드 보안 강화
 */
public class AwsInfrastructureService {
    // 구현 필요
}

/**
 * TODO: CI/CD 테스트 자동화
 * - 단위 테스트 자동화
 * - 통합 테스트 자동화
 * - 배포 전 검증 프로세스
 */
public class CiCdAutomationService {
    // 구현 필요
}

/**
 * TODO: 모니터링 알림 및 대시보드 개선
 * - 실시간 모니터링 대시보드
 * - 임계값 기반 알림
 * - 성능 지표 시각화
 */
public class MonitoringDashboardService {
    // 구현 필요
}

/**
 * TODO: 백업 및 복구 자동화
 * - 자동 백업 스케줄링
 * - 백업 검증
 * - 복구 프로세스 자동화
 */
public class BackupRecoveryService {
    // 구현 필요
}

/**
 * TODO: 보안 강화
 * - 취약점 스캔 자동화
 * - 보안 패치 관리
 * - 침입 탐지 시스템
 */
public class SecurityEnhancementService {
    // 구현 필요
}

/**
 * TODO: 오토 스케일링 구성
 * - 부하 기반 스케일링 규칙
 * - 리소스 사용 최적화
 * - 비용 효율적인 운영
 */
public class AutoScalingService {
    // 구현 필요
}

/**
 * TODO: 멀티 리전 배포 검토
 * - 지역별 성능 분석
 * - 리전 간 데이터 동기화
 * - 재해 복구 전략
 */
public class MultiRegionDeploymentService {
    // 구현 필요
}

/**
 * TODO: 재해 복구 계획 수립
 * - 재해 시나리오 정의
 * - 복구 절차 문서화
 * - 복구 훈련 계획
 */
public class DisasterRecoveryService {
    // 구현 필요
}
```

## 10. 테스트 및 품질 관리

```java
/**
 * TODO: 단위 테스트 커버리지 확대
 * - 핵심 비즈니스 로직 테스트
 * - 경계 조건 테스트
 * - 예외 처리 테스트
 */
public class UnitTestExpansionService {
    // 구현 필요
}

/**
 * TODO: 통합 테스트 케이스 추가
 * - 컴포넌트 간 통합 테스트
 * - 외부 시스템 연동 테스트
 * - 데이터 흐름 테스트
 */
public class IntegrationTestService {
    // 구현 필요
}

/**
 * TODO: E2E 테스트 구현
 * - 사용자 시나리오 기반 테스트
 * - UI 자동화 테스트
 * - 크로스 브라우저/디바이스 테스트
 */
public class E2eTestingService {
    // 구현 필요
}

/**
 * TODO: 성능 테스트 상세 분석
 * - 부하 테스트 시나리오 확장
 * - 병목 지점 식별 및 개선
 * - 확장성 테스트
 */
public class PerformanceTestingService {
    // 구현 필요
}

/**
 * TODO: 보안 테스트 강화
 * - 취약점 스캔
 * - 침투 테스트
 * - 코드 보안 분석
 */
public class SecurityTestingService {
    // 구현 필요
}

/**
 * TODO: 테스트 자동화 확대
 * - CI/CD 파이프라인 통합
 * - 테스트 보고서 자동화
 * - 회귀 테스트 자동화
 */
public class TestAutomationService {
    // 구현 필요
}

/**
 * TODO: 사용성 테스트 실시
 * - 사용자 경험 테스트
 * - 접근성 테스트
 * - 사용자 피드백 수집 및 분석
 */
public class UsabilityTestingService {
    // 구현 필요
}
```
