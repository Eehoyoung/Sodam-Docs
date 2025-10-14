# Attendance BE As-Is 통합 보고서 v1.0

Role: API Contract Reviewer · Spring Boot Integrator · Exception Flow Designer

## 📋 개요

- 작성일: 2025-09-30 15:23 (KST)
- 작성자: Junie (Backend)
- 문서 유형: 보고서 / 통합 / 정합성
- 관련 이슈: FE↔BE Attendance 동작 정합성 설명 및 협의 기반 자료

## 🎯 목적

프런트엔드 코드를 변경하지 않고, 현재 서버(백엔드)가 “있는 그대로” Attendance(근태) 기능을 어떻게 처리하는지 매우 상세히 설명합니다. 실제 엔드포인트, 요청/응답 스키마, 내부 검증(위치 인증), 예외
및 오류 응답 형식, 캐싱/무효화 전략을 모두 담습니다. 이 문서는 RN 팀과의 타협 및 정합성 합의를 위한 1급 보고서입니다.

## 🧭 환경/조건

- 백엔드: Spring Boot 3.x, Java 17
- 인증: JWT (Authorization: Bearer {accessToken})
- API Base Path: /api/attendance (컨트롤러 레벨 @RequestMapping)
- 시간 형식: ISO-8601 (예: 2025-05-01T09:00:00)
- 전역 예외 처리: GlobalExceptionHandler 적용, ApiResponse 포맷으로 오류 응답
- 캐시: Spring Cache(이름: "attendance") 일부 조회 API에 @Cacheable, 쓰기 API에 @CacheEvict(allEntries=true)

## 🌐 베이스 경로 및 컨트롤러

- 컨트롤러: com.rich.sodam.controller.AttendanceController (lines 29–169)
- 베이스 경로: /api/attendance

## 📚 엔드포인트 상세 (As-Is)

1) 출근 등록 — 위치 인증 포함

- 메서드/경로: POST /api/attendance/check-in
- 요청 바디: AttendanceRequestDto
    - employeeId: number(Long) 필수
    - storeId: number(Long) 필수
    - latitude: number(Double) 필수
    - longitude: number(Double) 필수
- 처리 흐름 (AttendanceService.checkInWithVerification)
    - 위치 인증: LocationVerificationService.verifyUserInStore(storeId, lat, lng)
        - 실패 시 LocationVerificationException.outOfRange() 발생 → 403 매핑(아래 오류 응답 참조)
    - 비즈니스 처리: checkIn(employeeId, storeId, lat, lng)
        - 오늘자 출근 기록 존재 여부 확인 → 존재 시 InvalidOperationException("이미 오늘 출근 기록이 있습니다.")
        - 시급(매장-직원 관계) 조회 후 Attendance 엔티티 생성 및 checkIn(lat, lng, hourlyWage)
        - 저장 후 AttendanceResponseDto로 변환하여 반환
- 응답: AttendanceResponseDto (상세 필드 아래 참조)
- 상태 코드: 200 | 400 | 401 | 403

2) 퇴근 등록 — 위치 인증 포함

- 메서드/경로: POST /api/attendance/check-out
- 요청 바디: AttendanceRequestDto (출근과 동일 필드)
- 처리 흐름 (AttendanceService.checkOutWithVerification)
    - 위치 인증 동일
    - 비즈니스 처리: checkOut(employeeId, storeId, lat, lng)
        - 오늘자 출근 기록 존재 여부 확인 → 없으면 InvalidOperationException("오늘 출근 기록이 없습니다.")
        - 가장 최근 출근 기록(todayAttendances.get(0))에 대해 checkOut(lat, lng)
        - 저장 후 AttendanceResponseDto로 변환하여 반환
- 응답: AttendanceResponseDto
- 상태 코드: 200 | 400 | 401 | 403 | 404(특정 경우, 상세는 오류 응답 섹션 참고)

3) 직원 기간별 출퇴근 조회

- 메서드/경로: GET /api/attendance/employee/{employeeId}
- 쿼리 파라미터: startDate(ISO DATETIME), endDate(ISO DATETIME)
- 처리: AttendanceService.getAttendancesByEmployeeAndPeriod
    - @Cacheable("attendance", key = 'employee:' + employeeId + ':' + startDate + ':' + endDate)
- 응답: AttendanceResponseDto[] (내림차순 정렬)
- 상태 코드: 200 | 400 | 401 | 404(직원 미존재 등)

4) 매장 기간별 출퇴근 조회

- 메서드/경로: GET /api/attendance/store/{storeId}
- 쿼리 파라미터: startDate, endDate (ISO DATETIME)
- 처리: AttendanceService.getAttendancesByStoreAndPeriod
    - @Cacheable("attendance", key = 'store:' + storeId + ':' + startDate + ':' + endDate)
- 응답: AttendanceResponseDto[] (내림차순 정렬)
- 상태 코드: 200 | 400 | 401 | 404(매장 미존재 등)

5) 직원 월간 출퇴근 조회

- 메서드/경로: GET /api/attendance/employee/{employeeId}/monthly
- 쿼리: year(number), month(1-12)
- 처리: AttendanceService.getMonthlyAttendancesByEmployee → 내부적으로 3) 재사용
- 응답: AttendanceResponseDto[]

6) 수동 출퇴근 등록(사업주)

- 메서드/경로: POST /api/attendance/manual-register
- 요청 바디: ManualAttendanceRequestDto
    - employeeId: number(Long) 필수
    - storeId: number(Long) 필수
    - registeredBy: number(Long, 등록자 userId) 필수
    - checkInTime: string(ISO DATETIME) 필수
    - checkOutTime: string(ISO DATETIME) 선택
    - reason: string 선택
- 처리: AttendanceService.registerManualAttendance
    - userService.validateMasterPermission(registeredBy)로 권한 검증
    - 사원-매장 관계 조회, 해당 날짜 중복 기록 검증(validateNoDuplicateAttendance)
    - 수동 checkIn/checkOut 처리 후 저장
- 응답: AttendanceResponseDto
- 상태 코드: 200 | 400 | 401 | 403 | 404

## 🧾 DTO 스키마

- AttendanceRequestDto
    - employeeId(Long), storeId(Long), latitude(Double), longitude(Double) — 모두 @NotNull
- AttendanceResponseDto
    - id(Long), employeeId(Long), employeeName(String), storeId(Long), storeName(String)
    - checkInTime(LocalDateTime), checkOutTime(LocalDateTime?)
    - checkInLatitude(Double), checkInLongitude(Double), checkOutLatitude(Double?), checkOutLongitude(Double?)
    - appliedHourlyWage(Integer), workingHours(Double), dailyWage(Integer?)
- ManualAttendanceRequestDto (요약)
    - employeeId, storeId, registeredBy, checkInTime, checkOutTime?, reason?

## 📡 위치 인증(As-Is)

- LocationVerificationService.verifyUserInStore(storeId, userLat, userLng)
    - StoreRepository에서 Store 조회 실패 시 EntityNotFoundException("Store", storeId)
    - Store.isUserInRadius(userLat, userLng)로 반경 내 여부 판단
    - null 위/경도는 false 처리
- AttendanceService.checkInWithVerification / checkOutWithVerification에서 선 검증 후 비즈니스 처리
- 검증 실패 시 LocationVerificationException.outOfRange() → 403

## 🧪 캐시/무효화

- 조회 API 일부에 @Cacheable("attendance") 적용
    - 직원 기간별, 매장 기간별 조회 키 구성: employee:/store: + 식별자 + 기간 문자열
- 쓰기 API(check-in, check-out, manual-register)는 @CacheEvict(value="attendance", allEntries=true)로 전체 무효화
- TTL/캐시 저장소 설정은 본 파일에서 직접 지정하지 않으며, 전역 캐시 설정에 따릅니다.

## 🚨 예외 및 오류 응답(표준 포맷)

- 전역 핸들러: GlobalExceptionHandler
- 오류 응답 포맷: ApiResponse
    - { success: false, message: string, errorCode: string, timestamp: string, data?: any }
- 매핑 요약
    - BusinessException (기본): 400
        - InvalidOperationException → 400 (예: "이미 오늘 출근 기록이 있습니다.", "오늘 출근 기록이 없습니다.")
        - LocationVerificationException → 400 기본이지만, 스펙에서 출근/퇴근 API는 403을 명세적으로 표기 (실제 핸들러는 타입별 분기 없이
          BusinessException=400, 다만 컨트롤러 @ApiResponses 문서화는 403을 포함)
    - EntityNotFoundException: 404
    - MethodArgumentNotValidException: 400 (필드 에러 맵 포함)
    - IllegalArgumentException: 400
    - NoSuchElementException: 404
    - Exception: 500

참고: 컨트롤러의 @ApiResponses는 의도한 상태코드를 설명하지만, 실 런타임 상태코드는 GlobalExceptionHandler 매핑에 따릅니다. 위치 인증 실패(현재는 BusinessException
계열) 시 400으로 응답될 수 있으므로, FE 단에서는 errorCode(예: LOCATION_VERIFICATION_FAILED) 기반 처리 권장.

## 🔗 요청/응답 예시

- 출근 등록(curl)

```
curl -X POST "{HOST}/api/attendance/check-in" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "employeeId": 101,
    "storeId": 12,
    "latitude": 37.5665,
    "longitude": 126.9780
  }'
```

- 퇴근 등록(curl)

```
curl -X POST "{HOST}/api/attendance/check-out" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "employeeId": 101,
    "storeId": 12,
    "latitude": 37.5665,
    "longitude": 126.9780
  }'
```

- 직원 기간별 조회(curl)

```
curl -G "{HOST}/api/attendance/employee/101" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  --data-urlencode "startDate=2025-09-01T00:00:00" \
  --data-urlencode "endDate=2025-09-30T23:59:59"
```

## 🔍 RN As-Is 대비 핵심 차이(사실 기반)

- 경로 접두사
    - BE: 모든 Attendance REST가 "/api/attendance/*"
    - RN: NFC verify만 "/api/attendance/nfc-verify"를 사용, 그 외는 "/attendance/*"를 혼용
- Verify 단계
    - BE: 별도 verify 엔드포인트 없음. check-in/out 내부에서 위치 인증 수행
    - RN: verify(nfc/location) → check-in/out 2단 분리 호출
- ID/타입 명칭
    - BE: storeId(Long), employeeId(Long)
    - RN: workplaceId(string) 표현 사용 사례 존재
- 퇴근 경로
    - BE: POST /api/attendance/check-out (attendanceId path 변수 사용 안 함)
    - RN: POST /attendance/{attendanceId}/check-out 형태를 일부 문서에서 사용
- 오류 응답/코드 처리
    - BE: ApiResponse 표준 포맷 + errorCode (예: LOCATION_VERIFICATION_FAILED)
    - RN: 401 자동 refresh, distance 기반 메시지 처리 등 FE 고유 로직 존재

## ✅ 수락 기준(Alignment 관점)

- RN 팀이 본 문서를 통해 BE As-Is 엔드포인트/스키마/검증/오류 응답을 정확히 이해한다.
- RN 보고서(\docs\Attendance_FE_AsIs_통합_보고서_v1.0_2025-09-30.md)와의 차이점이 명확히 정리된다.
- 차이 해소를 위한 타협안/로드맵 문서(별첨)를 토대로 상호 작업 계획이 수립된다.

## 🔗 관련 문서

- RN팀 As-Is: ..\Attendance_FE_AsIs_통합_보고서_v1.0_2025-09-30.md
- API 상세: ..\technical\api\AttendanceController_API_문서.md
- JWT 연동: ..\technical\JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자             |
------------|-----|-------|-----------------|
 2025-09-30 | 1.0 | 최초 작성 | Junie (Backend) |
