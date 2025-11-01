# 연동 API 목록 (전체 프런트엔드 기준)

최신 코드 기준으로 리포지토리에서 실제 사용 중인 백엔드 API 엔드포인트를 모두 수집하여 표로 정리했습니다.
요청하신 형식(엔드포인트/기능/사용이유)을 따르되, 추적 용이성을 위해 위치(파일/함수)도 함께 표기했습니다.

생성 시각: 2025-10-28 01:42
업데이트 시각: 2025-10-28 01:46

주의사항
- 베이스 URL은 src/common/utils/api.ts에서 getBaseUrl()로 관리됩니다. 현재 개발 기본값은 http://10.0.2.2:8080 입니다.
- 일부 응답은 { data: ... } 래핑/비래핑 모두 허용하도록 방어적으로 파싱합니다.
- 몇몇 인증 라우트는 404/405 시 레거시 경로로 폴백하도록 구현되어 있습니다.

## 1) 인증/Auth
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치(파일/함수) |
|---|---|---|---|
| /api/login (POST) | 이메일/비밀번호 로그인 | 액세스/리프레시 토큰 발급 및 사용자 세션 시작 | src/features/auth/services/authService.ts: login / src/features/auth/services/authApi.ts: login |
| /api/join (POST) | 회원가입 | 신규 사용자 생성 및 초기 권한/목적 설정 | src/features/auth/services/authService.ts: signup / src/features/auth/services/authApi.ts: join |
| /api/auth/refresh (POST) | 토큰 재발급 | 액세스 토큰 만료 시 리프레시 토큰으로 재발급 | src/common/utils/api.ts: refreshAccessToken / src/features/auth/services/authApi.ts: refresh |
| /api/refresh (POST) | 토큰 재발급(폴백) | /api/auth/refresh 미제공 서버 대응 | src/common/utils/api.ts: refreshAccessToken |
| /api/auth/logout (POST) | 로그아웃 통지 | 서버 세션 정리(선택적) | src/features/auth/services/authService.ts: logout |
| /api/logout (POST) | 로그아웃(폴백) | 레거시 로그아웃 엔드포인트 대응 | src/features/auth/services/authService.ts: logout |
| /api/auth/me (GET) | 현재 사용자 정보 | 세션 사용자 정보 조회 | src/features/auth/services/authService.ts: getCurrentUser |
| /api/me (GET) | 현재 사용자 정보(폴백) | 레거시 me 엔드포인트 대응 | src/features/auth/services/authService.ts: getCurrentUser |
| /api/auth/password/reset/request (POST) | 비밀번호 재설정 메일 요청 | 비밀번호 분실 플로우 시작 | src/features/auth/services/authService.ts: requestPasswordReset |
| /api/password-reset-request (POST) | 재설정 요청(폴백) | 레거시 대응 | src/features/auth/services/authService.ts: requestPasswordReset |
| /api/auth/password/reset (POST) | 비밀번호 재설정 | 토큰으로 비밀번호 변경 | src/features/auth/services/authService.ts: resetPassword |
| /api/password-reset (POST) | 비밀번호 재설정(폴백) | 레거시 대응 | src/features/auth/services/authService.ts: resetPassword |
| /kakao/auth/proc?code=... (GET) | 카카오 OAuth 코드 처리 | 소셜 로그인(백엔드 리디렉션 처리) | src/features/auth/services/authService.ts: kakaoLogin |

외부 연동(백엔드 외부):
- https://kauth.kakao.com/oauth/authorize (GET) — 카카오 동의 화면 열기. src/features/auth/services/authApi.ts: openKakaoLogin

## 2) 사용자/User
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/users/{userId}/purpose (POST) | 사용자 목적/역할 설정 | 개인/직원/사장 목적 확정 | src/features/auth/services/userService.ts: setPurpose / src/features/auth/services/authApi.ts: setPurpose |
| /api/user/{userId} (GET) | 사용자 단건 조회 | 사용자 상세 조회 | src/features/auth/services/userService.ts: getUser |
| /api/user/{employeeId} (PUT) | 직원 프로필 수정 | 직원 정보 갱신 | src/features/auth/services/userService.ts: updateEmployee |

## 3) 마이페이지(사장)/Master
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/master/mypage (GET) | 대시보드 데이터 | 사장 홈 요약 | src/features/myPage/services/masterService.ts: mypage |
| /api/master/profile (GET) | 프로필 조회 | 사장 프로필 표시 | src/features/myPage/services/masterService.ts: getProfile |
| /api/master/profile (PUT) | 프로필 수정 | 사장 정보 수정 | src/features/myPage/services/masterService.ts: putProfile |
| /api/master/stores (GET) | 내 매장 목록 | 사장 보유 매장 리스트 | src/features/myPage/services/masterService.ts: stores |
| /api/master/stats/store/{storeId} (GET) | 매장 통계 | 매장별 주요 지표 | src/features/myPage/services/masterService.ts: storeStats |
| /api/master/stats/overall (GET) | 전체 통계 | 전체 매장 통합 지표 | src/features/myPage/services/masterService.ts: overallStats |
| /api/master/timeoff/pending (GET) | 휴가 승인 대기 목록 | 인사 승인 처리 | src/features/myPage/services/masterService.ts: timeoffPending |
| /api/master/timeoff/{requestId}/approve (PUT) | 휴가 승인 | 요청 처리 | src/features/myPage/services/masterService.ts: approve |
| /api/master/timeoff/{requestId}/reject (PUT) | 휴가 반려 | 요청 처리 | src/features/myPage/services/masterService.ts: reject |

## 4) 매장/Stores
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/stores/master/{userIdOrCurrent} (GET) | 사장의 모든 매장 조회 | 매장 선택/관리 | src/features/workplace/services/workplaceService.ts: getWorkplaces / src/features/store/services/storeService.ts: getMasterStores |
| /api/stores/{storeId} (GET) | 매장 단건 조회 | 상세 정보 표시 | src/features/workplace/services/workplaceService.ts: getWorkplaceById / src/features/store/services/storeService.ts: getStoreById |
| /api/stores/registration (POST) | 매장 등록 | 신규 매장 생성 | src/features/workplace/services/workplaceService.ts: createWorkplace / src/features/store/services/storeService.ts: createStore |
| /api/stores/{storeId} (PUT) | 매장 정보 수정 | 주소/이름 등 수정 | src/features/workplace/services/workplaceService.ts: updateWorkplace |
| /api/stores/{storeId} (DELETE) | 매장 삭제 | 매장 제거 | src/features/workplace/services/workplaceService.ts: deleteWorkplace |
| /api/stores/{storeId}/employees (GET) | 매장 직원 조회 | 직원 목록 표시 | src/features/workplace/services/workplaceService.ts: getWorkplaceEmployees |
| /api/stores/{storeId}/location (PUT) | 매장 좌표/반경 설정 | 위치 기반 근태 반경 설정 | src/features/store/services/storeService.ts: putLocation |
| /api/stores/change/master (POST) | 매장 소유자 변경 | 사장 권한 이양 | src/features/store/services/storeService.ts: changeOwner |
| /api/stores/{storeId}/nfc-tag (GET) | NFC 태그 생성 데이터 | 태그 발급/표시 | src/features/attendance/services/nfcAttendanceService.ts: generateStoreNFCTag |
| /api/stores/{storeId}/nfc-settings (GET) | NFC 설정 조회 | 매장별 NFC 정책 확인 | src/features/attendance/services/nfcAttendanceService.ts: getStoreNFCSettings |
| /api/stores/{storeId}/nfc-settings (PUT) | NFC 설정 수정 | 매장별 NFC 정책 변경 | src/features/attendance/services/nfcAttendanceService.ts: updateStoreNFCSettings |

## 5) 근태/Attendance
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/attendance (GET) | 근태 목록 조회 | 필터별 근태 리스트 | src/features/attendance/services/attendanceService.ts: getAttendanceRecords |
| /api/attendance/{attendanceId} (GET) | 근태 단건 조회 | 상세 보기 | src/features/attendance/services/attendanceService.ts: getAttendanceById |
| /api/attendance/check-in (POST) | 출근 처리 | 출근 기록 생성 | src/features/attendance/services/attendanceService.ts: checkIn |
| /api/attendance/check-out (POST) | 퇴근 처리 | 퇴근 기록 업데이트 | src/features/attendance/services/attendanceService.ts: checkOutStandard/checkOut |
| /api/attendance/{attendanceId} (PUT) | 근태 수정 | 시간/비고 수정 | src/features/attendance/services/attendanceService.ts: updateAttendance |
| /api/attendance/{attendanceId} (DELETE) | 근태 삭제 | 오기록 제거 | src/features/attendance/services/attendanceService.ts: deleteAttendance |
| /api/attendance/current (GET) | 현재 근무 상태 | 실시간 상태 표시 | src/features/attendance/services/attendanceService.ts: getCurrentAttendance |
| /api/attendance/statistics (GET) | 근태 통계 | 기간/필터별 통계 | src/features/attendance/services/attendanceService.ts: getAttendanceStatistics |
| /api/attendance/statistics/employee/{employeeId} (GET) | 직원별 통계 | 개인 근태 분석 | src/features/attendance/services/attendanceService.ts: getEmployeeAttendanceStatistics |
| /api/attendance/statistics/store/{storeId} (GET) | 매장별 통계 | 매장 근태 분석 | src/features/attendance/services/attendanceService.ts: getWorkplaceAttendanceStatistics |
| /api/attendance/batch-status (PUT) | 상태 일괄 변경 | 대량 상태 수정 | src/features/attendance/services/attendanceService.ts: batchUpdateStatus |
| /api/attendance/verify/location (POST) | 위치 기반 인증 | 반경 내 출퇴근 인증 | src/features/attendance/services/locationAttendanceService.ts: verifyCheckInByLocation/verifyCheckOutByLocation / src/features/attendance/services/attendanceService.ts: verifyLocationAttendance |
| /api/attendance/verify/nfc (POST) | NFC 기반 인증 | 태그로 출퇴근 인증 | src/features/attendance/services/nfcAttendanceService.ts: verifyCheckInByNFC/verifyCheckOutByNFC |

## 6) 급여/Payroll & 임금/Wage
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/payroll/calculate (POST) | 급여 계산 | 기간 기준 급여 산정 | src/features/salary/services/payrollService.ts: calculate |
| /api/payroll/employee/{employeeId}/store/{storeId}/monthly (GET) | 월별 급여 요약 | 월 단위 목록 | src/features/salary/services/payrollService.ts: getMonthly |
| /api/payroll/{payrollId}/details (GET) | 급여 상세 | 항목별 내역 조회 | src/features/salary/services/payrollService.ts: getDetails |
| /api/payroll/{payrollId}/status (PUT) | 급여 상태 변경 | 지급/보류 상태 갱신 | src/features/salary/services/payrollService.ts: updateStatus |
| /api/payroll/employee/{employeeId} (GET) | 직원 급여 목록 | 기간 필터 | src/features/salary/services/payrollService.ts: listByEmployee |
| /api/payroll/store/{storeId} (GET) | 매장 급여 목록 | 기간 필터 | src/features/salary/services/payrollService.ts: listByStore |
| /api/wages/store/{storeId}/standard (PUT) | 매장 기준 시급 설정 | 표준 시급 관리 | src/features/wage/services/wageService.ts: putStandardHourlyWage |
| /api/wages/employee/{employeeId}/store/{storeId} (GET) | 직원 시급 조회 | 개인-매장별 시급 | src/features/wage/services/wageService.ts: getEmployeeWage |
| /api/wages/employee (POST) | 직원 시급 등록/갱신 | 시급 upsert | src/features/wage/services/wageService.ts: upsertEmployeeWage |

## 7) 휴가/TimeOff
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/timeoff (POST) | 휴가 신청 | 직원 휴가 요청 생성 | src/features/myPage/services/timeOffService.ts: create |
| /api/timeoff/store/{storeId} (GET) | 매장 휴가 목록 | 매장 단위 관리 | src/features/myPage/services/timeOffService.ts: getStoreAll |
| /api/timeoff/store/{storeId}/status/{status} (GET) | 상태별 목록 | 승인/반려/대기 필터 | src/features/myPage/services/timeOffService.ts: getByStatus |
| /api/timeoff/employee/{employeeId} (GET) | 직원 휴가 목록 | 개인 이력 | src/features/myPage/services/timeOffService.ts: getEmployeeAll |
| /api/timeoff/{requestId}/approve (PUT) | 휴가 승인 | 요청 처리 | src/features/myPage/services/timeOffService.ts: approve |
| /api/timeoff/{requestId}/reject (PUT) | 휴가 반려 | 요청 처리 | src/features/myPage/services/timeOffService.ts: reject |

## 8) 구독/Subscription
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /subscription/plans (GET) | 플랜 목록 | 요금제 표시 | src/features/subscription/services/subscriptionService.ts: getAllPlans |
| /subscription/plans/{planId} (GET) | 플랜 상세 | 상세 비교 | src/features/subscription/services/subscriptionService.ts: getPlanById |
| /subscription/status (GET) | 내 구독 상태 | 플랜/만료 등 표시 | src/features/subscription/services/subscriptionService.ts: getCurrentSubscription |
| /subscription/subscribe (POST) | 구독 신청 | 과금 시작 | src/features/subscription/services/subscriptionService.ts: subscribe |
| /subscription/cancel (POST) | 구독 취소 | 구독 해지 | src/features/subscription/services/subscriptionService.ts: cancelSubscription |
| /subscription/auto-renew (PUT) | 자동갱신 설정 | 자동 결제 토글 | src/features/subscription/services/subscriptionService.ts: updateAutoRenew |
| /subscription/change-plan (PUT) | 플랜 변경 | 등급 변경 | src/features/subscription/services/subscriptionService.ts: changePlan |

## 9) 리포트/Reports
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /reports/operation (GET) | 운영 리포트 | 매장 운영 요약 | src/features/myPage/services/reportService.ts: getOperationReport |
| /reports/labor-cost (GET) | 인건비 리포트 | 인건비 분석 | src/features/myPage/services/reportService.ts: getLaborCostReport |
| /reports/employee-efficiency (GET) | 효율성 리포트 | 직원 생산성 | src/features/myPage/services/reportService.ts: getEmployeeEfficiencyReport |
| /reports/operation-cost (GET) | 운영비 리포트 | 비용 분석 | src/features/myPage/services/reportService.ts: getOperationCostReport |
| /reports/custom (POST) | 맞춤형 리포트 | 사용자 정의 리포트 생성 | src/features/myPage/services/reportService.ts: generateCustomReport |

## 10) 사용자 프로필 도메인(/user)
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /user/profile (GET) | 현재 사용자 프로필 | 프로필 표시 | src/features/myPage/services/userProfileService.ts: getCurrentUserProfile |
| /user/profile (PUT) | 프로필 수정 | 사용자 정보 변경 | src/features/myPage/services/userProfileService.ts: updateUserProfile |
| /user/profile/image (POST) | 프로필 이미지 업로드 | 멀티파트 업로드 | src/features/myPage/services/userProfileService.ts: uploadProfileImage |
| /user/employee-profile (GET) | 직원 프로필 | 직원 정보 표시 | src/features/myPage/services/userProfileService.ts: getEmployeeProfile |
| /user/manager-profile (GET) | 매니저 프로필 | 매니저 정보 표시 | src/features/myPage/services/userProfileService.ts: getManagerProfile |
| /user/master-profile (GET) | 사장 프로필 | 사장 정보 표시 | src/features/myPage/services/userProfileService.ts: getMasterProfile |
| /user/workplaces (GET) | 직원 근무지 목록 | 근무지 선택 | src/features/myPage/services/userProfileService.ts: getEmployeeWorkplaces |
| /user/attendance-records (GET) | 출퇴근 기록 | 개인 근태 이력 | src/features/myPage/services/userProfileService.ts: getAttendanceRecords |
| /user/salary-records (GET) | 급여 기록 목록 | 급여 내역 | src/features/myPage/services/userProfileService.ts: getSalaryRecords |
| /user/salary-records/{salaryId} (GET) | 급여 기록 상세 | 상세 확인 | src/features/myPage/services/userProfileService.ts: getSalaryRecordById |
| /user/salary-statistics (GET) | 급여 통계 | 연간 통계 | src/features/myPage/services/userProfileService.ts: getSalaryStatistics |
| /user/work-hour-statistics (GET) | 근무시간 통계 | 연/월 통계 | src/features/myPage/services/userProfileService.ts: getWorkHourStatistics |
| /user/salary-records/{salaryId}/statement (GET) | 급여 명세서 URL | 명세서 다운로드 | src/features/myPage/services/userProfileService.ts: downloadSalaryStatement |

## 11) 홈 정보(/api/v1)
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/v1/events (GET) | 이벤트 목록 | 홈 배너/슬라이더 | src/features/home/services/homeService.ts: fetchEvents |
| /api/v1/labor-info (GET) | 노동법 정보 | 정보성 콘텐츠 | src/features/home/services/homeService.ts: fetchLaborInfo |
| /api/v1/policies (GET) | 정책 정보 | 정보성 콘텐츠 | src/features/home/services/homeService.ts: fetchPolicies |
| /api/v1/tax-info (GET) | 세금 정보 | 정보성 콘텐츠 | src/features/home/services/homeService.ts: fetchTaxInfo |
| /api/v1/tips (GET) | 팁 | 정보성 콘텐츠 | src/features/home/services/homeService.ts: fetchTips |
| /api/v1/testimonials (GET) | 후기 | 정보성 콘텐츠 | src/features/home/services/homeService.ts: fetchTestimonials |
| /api/v1/services (GET) | 서비스 목록 | 제품 소개 | src/features/home/services/homeService.ts: getServices |

## 12) PersonalUser (Personal 모드)
| API ADDRESS (METHOD) | 기능 | 사용이유 | 위치 |
|---|---|---|---|
| /api/personal-users/{userId} (GET) | 개인 프로필 조회 | Personal 모드 프로필 | src/features/personal/services/personalUserService.ts: getProfile |
| /api/personal-users/{userId} (PUT) | 개인 프로필 수정 | 닉네임/기본시급 변경 | src/features/personal/services/personalUserService.ts: updateProfile |
| /api/personal-users/{userId}/workplaces (GET) | 근무지 목록(커서) | 개인 근무지 조회 | src/features/personal/services/personalUserService.ts: getWorkplaces |
| /api/personal-users/{userId}/workplaces (POST) | 근무지 생성 | 개인 근무지 추가 | src/features/personal/services/personalUserService.ts: createWorkplace |
| /api/personal-users/{userId}/workplaces/{workplaceId} (PUT) | 근무지 수정 | 근무지 정보 갱신 | src/features/personal/services/personalUserService.ts: updateWorkplace |
| /api/personal-users/{userId}/workplaces/{workplaceId} (DELETE) | 근무지 삭제 | 근무지 제거 | src/features/personal/services/personalUserService.ts: deleteWorkplace |
| /api/personal-users/{userId}/attendance/check-in (POST) | 출근(멱등성) | 수동 출근 처리 | src/features/personal/services/personalUserService.ts: checkIn |
| /api/personal-users/{userId}/attendance/check-out (POST) | 퇴근(멱등성) | 수동 퇴근 처리 | src/features/personal/services/personalUserService.ts: checkOut |
| /api/personal-users/{userId}/attendance (GET) | 출퇴근 기록 조회 | 기간+커서 기반 조회 | src/features/personal/services/personalUserService.ts: getAttendance |
| /api/personal-users/{userId}/attendance/monthly (GET) | 월별 요약 | 일자별 총 근무 분/세션 | src/features/personal/services/personalUserService.ts: getMonthlySummary |

Note
- 인증: Bearer JWT 필요, token.userId == path {userId}
- 멱등성 헤더: Idempotency-Key (check-in/out 필수)
- 페이지네이션: cursor, limit (workplaces: 기본20, attendance: 기본50)
- 참고 문서: docs\persnal_user\PersonalUser_API_연동_가이드.md, docs\persnal_user\personal-user.yaml

---

추가 메모
- 토큰 인터셉터/재발급 로직은 src/common/utils/api.ts에 있으며, 모든 api.get/post/put/delete 호출에 공통 적용됩니다.
- NFC/위치 인증은 동일한 verify 엔드포인트를 출근/퇴근에 공용으로 사용하며(isCheckOut 플래그), 서버가 무시해도 무방하도록 설계되어 있습니다.
- 일부 API는 개발 단계에서 목 데이터/임시 반환을 포함할 수 있습니다(예: createStore 실패시 임시 ID 반환).



## 공통 규약 (Common Conventions)
- Base URL
  - 공통 API 클라이언트: src/common/utils/api.ts의 getBaseUrl() → 현재 개발 기본값 http://10.0.2.2:8080
  - 홈 서비스 전용: src/features/home/services/homeService.ts → REACT_APP_API_URL 환경변수(없으면 http://localhost:8080)
- 인증/헤더
  - Authorization: Bearer <access_token> — api 클라이언트가 자동 주입(TokenManager 연동)
  - Refresh Token: 401 발생 시 자동 재발급 시도(/api/auth/refresh → 404/405면 /api/refresh 폴백)
  - Content-Type: application/json 기본, 파일 업로드는 multipart/form-data
- 응답 래핑
  - { data: ... } 또는 비래핑 바디 모두 허용. 서비스 레이어에서 방어적으로 파싱합니다.
- 날짜/시간 형식
  - 기본 ISO 8601(예: 2025-10-28T01:00:00Z). 일부 쿼리는 연/월 등 단순 문자열(year, month)을 사용.
- ID 타입
  - storeId 등은 숫자 기반. 일부 서비스는 문자열 workplaceId를 받아 내부에서 숫자로 변환(attendanceService 등).
- 타임아웃/로그
  - axios 타임아웃 기본 10초. 요청/응답 로그는 개발 환경에서 콘솔에 출력됩니다.

## 오류 응답 규격 및 처리(클라이언트 동작 요약)
- 공통 HTTP Status 활용 예시: 400(유효성), 401(인증), 403(권한), 404(없음), 409(충돌), 422(처리불가), 500(서버).
- 401 처리 흐름
  1) 인터셉터가 refreshAccessToken() 수행 → 성공 시 대기 중이던 요청을 신규 토큰으로 재시도
  2) 실패 시 토큰/세션 적출 후 onUnauthorized 콜백 호출(로그아웃 처리 등)
- 에러 바디 예시(참고):
```
{
  "message": "유효하지 않은 요청입니다.",
  "code": "VALIDATION_ERROR",
  "errors": [ { "field": "email", "message": "형식이 올바르지 않습니다." } ]
}
```
- 폴백 문서화: /api/auth/refresh → 실패 시 /api/refresh, /api/auth/me → 실패 시 /api/me, /api/auth/logout → 실패 시 /api/logout.

## 페이지네이션/정렬/필터 규칙
- 본 레포지토리의 서비스 구현에서 확인된 주요 쿼리 파라미터
  - Attendance: /api/attendance (filter 객체), /api/attendance/statistics(기간 필터), 직원/매장별 통계(startDate, endDate)
  - Reports: workplaceId, year, month
  - Payroll: year, month 또는 from, to
  - User 도메인: startDate, endDate, workplaceId, year
- 페이지네이션/정렬은 백엔드에서 지원되는 경우 일반적으로 다음을 사용(page, size, sort). 현재 코드에서 하드 요구되지는 않으며, BE가 허용 시 params에 추가해도 무방합니다.

## 권한/역할 접근 가이드(요약)
- 인증 필요: 대부분의 /api/*, /user/*, /reports/*, /subscription/* 엔드포인트
- 공개 가능: /api/login, /api/join (로그인/회원가입)
- MASTER 전용(일반적으로): /api/master/*, /api/stores/change/master, 매장 위치/반경 설정, 임금/급여 관리
- EMPLOYEE 권한: 자신의 근태/휴가 생성, 위치/NFC 인증, /user/* 자기 리소스 조회
- MANAGER 권한: 매장 단위 조회/통계/승인 등(조직 정책에 따름)
- 주의: 실제 권한 검증은 서버가 최종 결정. 클라이언트는 역할 기반 UI만 제한합니다.

## 샘플 요청/응답

1) 로그인 /api/login
- Request
```
POST /api/login
Content-Type: application/json
{
  "email": "boss@sodam.com",
  "password": "P@ssw0rd!"
}
```
- Success Response(예시)
```
{
  "accessToken": "<jwt>",
  "refreshToken": "<jwt>",
  "user": { "id": 1, "name": "사장님", "email": "boss@sodam.com", "role": "MASTER" }
}
```
- cURL
```
curl -X POST "$BASE/api/login" -H "Content-Type: application/json" \
  -d '{"email":"boss@sodam.com","password":"P@ssw0rd!"}'
```

2) 출근 처리 /api/attendance/check-in
- Request 바디(attendanceService에서 workplaceId→storeId로 변환)
```
POST /api/attendance/check-in
{
  "employeeId": 101,
  "workplaceId": "12", // 전달되면 서비스에서 storeId:12로 변환되어 전송
  "checkInTime": "2025-10-28T09:00:00+09:00"
}
```
- 실제 전송 바디(서비스 변환 후)
```
{
  "employeeId": 101,
  "storeId": 12,
  "checkInTime": "2025-10-28T09:00:00+09:00"
}
```

3) 위치 인증 /api/attendance/verify/location
- Request
```
POST /api/attendance/verify/location
{
  "employeeId": 101,
  "storeId": 12,
  "latitude": 37.498,
  "longitude": 127.027
}
```
- Response(예시)
```
{ "success": true, "distance": 8.3, "message": "반경 내 위치" }
```

4) 매장 좌표/반경 설정 /api/stores/{storeId}/location
- Request
```
PUT /api/stores/12/location
{
  "latitude": 37.498,
  "longitude": 127.027,
  "radius": 50
}
```
- Response(예시)
```
{ "success": true }
```

5) 프로필 이미지 업로드 /user/profile/image
- cURL
```
curl -X POST "$BASE/user/profile/image" \
  -H "Authorization: Bearer $ACCESS" \
  -F "image=@profile.jpg"
```

## 변경 이력(Changelog)
- 2025-10-28 01:44 — 상세 문서화(공통 규약, 오류 모델, 권한/역할, 샘플 요청/응답) 추가
- 2025-10-28 01:42 — 초기 전체 엔드포인트 표 정리

## 빠른 링크(섹션 이동)
- 공통 규약 (Common Conventions)
- 오류 응답 규격 및 처리
- 페이지네이션/정렬/필터 규칙
- 권한/역할 접근 가이드
- 샘플 요청/응답
- 도메인별 엔드포인트 표(위의 1)~11) 섹션)
