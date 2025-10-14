# AttendanceController — 출퇴근 관리 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

직원의 출근/퇴근 등록 및 조회 기능을 React Native 앱에서 즉시 연동할 수 있도록 상세 스펙, 헤더, 요청/응답 형식, 오류, 예제 코드를 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT 필수
- 공통 헤더
    - Authorization: Bearer {accessToken}
    - Content-Type: application/json
- 시간 형식: ISO-8601 (예: 2025-05-01T09:00:00)

## 🌐 베이스 경로

- /api/attendance

## 📚 엔드포인트 요약

- POST /check-in — 출근 등록(위치 인증 포함)
- POST /check-out — 퇴근 등록(위치 인증 포함)
- GET /employee/{employeeId} — 기간별 개인 출퇴근 조회
- GET /store/{storeId} — 기간별 매장 전체 출퇴근 조회
- GET /employee/{employeeId}/monthly — 직원 월간 출퇴근 조회
- POST /manual-register — 수동 출퇴근 등록(사업주)

---

## 1) 출근 등록

- 메서드/경로: POST /api/attendance/check-in
- 인증: 필요
- 요청 바디(AttendanceRequestDto)
    - employeeId: number (필수)
    - storeId: number (필수)
    - latitude: number (필수)
    - longitude: number (필수)
- 응답(AttendanceResponseDto)
    - id, employeeId, employeeName, storeId, storeName,
    - checkInTime, checkOutTime,
    - checkInLatitude, checkInLongitude, checkOutLatitude, checkOutLongitude,
    - appliedHourlyWage, workingHours, dailyWage
- 상태 코드: 200 OK | 400 잘못된 요청 | 401 인증 실패 | 403 위치 인증 실패

예시 curl

```
curl -X POST "{HOST}/api/attendance/check-in" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "employeeId": 12,
    "storeId": 101,
    "latitude": 37.5123,
    "longitude": 127.1023
  }'
```

React Native Axios 예시 (TypeScript)

```ts
import axios from "axios";

export async function checkIn(accessToken: string, employeeId: number, storeId: number, lat: number, lng: number) {
  const res = await axios.post(
    `${process.env.API_BASE_URL}/api/attendance/check-in`,
    { employeeId, storeId, latitude: lat, longitude: lng },
    { headers: { Authorization: `Bearer ${accessToken}` } }
  );
  return res.data; // AttendanceResponseDto
}
```

---

## 2) 퇴근 등록

- 메서드/경로: POST /api/attendance/check-out
- 인증: 필요
- 요청 바디: AttendanceRequestDto (위와 동일 필드)
- 응답: AttendanceResponseDto (상동)
- 상태 코드: 200 | 400 | 401 | 403 | 404(출근 기록 없음)

---

## 3) 직원 기간별 출퇴근 조회

- 메서드/경로: GET /api/attendance/employee/{employeeId}
- 쿼리 파라미터
    - startDate: string(ISO DATETIME) 필수
    - endDate: string(ISO DATETIME) 필수
- 응답: AttendanceResponseDto[]

예시

```
curl -G "{HOST}/api/attendance/employee/12" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  --data-urlencode "startDate=2025-05-01T00:00:00" \
  --data-urlencode "endDate=2025-05-31T23:59:59"
```

---

## 4) 매장 기간별 출퇴근 조회

- 메서드/경로: GET /api/attendance/store/{storeId}
- 쿼리: startDate, endDate (ISO DATETIME)
- 응답: AttendanceResponseDto[]

---

## 5) 직원 월간 출퇴근 조회

- 메서드/경로: GET /api/attendance/employee/{employeeId}/monthly
- 쿼리 파라미터
    - year: number (예: 2025)
    - month: number (1-12)
- 응답: AttendanceResponseDto[]

---

## 6) 수동 출퇴근 등록(사업주)

- 메서드/경로: POST /api/attendance/manual-register
- 인증: 사업주 권한 필요(백엔드에서 권한 검증)
- 요청 바디(ManualAttendanceRequestDto)
    - employeeId: number (필수)
    - storeId: number (필수)
    - registeredBy: number (필수, 등록자 userId)
    - checkInTime: string(ISO DATETIME) (필수)
    - checkOutTime: string(ISO DATETIME) (선택)
    - reason: string(최대 500) (선택)
- 응답: AttendanceResponseDto

예시

```
curl -X POST "{HOST}/api/attendance/manual-register" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "employeeId": 12,
    "storeId": 101,
    "registeredBy": 1,
    "checkInTime": "2025-05-01T09:00:00",
    "checkOutTime": "2025-05-01T18:00:00",
    "reason": "지문인식기 오류로 수동 등록"
  }'
```

## 🚨 오류 코드

- 400: 파라미터/시간 유효성 오류, 중복 기록 등
- 401: 토큰 누락/만료
- 403: 위치 인증 실패, 권한 부족
- 404: 직원/매장/출근기록 없음

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 

---

## 7) 위치 사전 검증 (v1.1 추가)

- 메서드/경로: POST /api/attendance/verify/location
- 인증: 필요
- 요청 바디(LocationVerifyRequest)
    - storeId: number (필수)
    - latitude: number (필수)
    - longitude: number (필수)
- 응답(ApiResponse<LocationVerifyResponse>)
    - success: boolean
    - reason?: string (OUT_OF_RANGE 등)
    - distance?: number (미터)
- 상태 코드: 200 | 400 | 404(매장 없음)

예시

```
curl -X POST "{HOST}/api/attendance/verify/location" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "storeId": 101,
    "latitude": 37.5123,
    "longitude": 127.1023
  }'
```

## 8) NFC 사전 검증 (v1.1 추가)

- 메서드/경로: POST /api/attendance/verify/nfc
- 인증: 필요
- 요청 바디(NfcVerifyRequest)
    - storeId?: number (정책 확정 시 사용)
    - tagId: string (필수, "SODAM-" 접두사, 길이≥10)
- 응답(ApiResponse<NfcVerifyResponse>)
    - success: boolean
    - reason?: string (INVALID_TAG 등)
- 상태 코드: 200 | 400

예시

```
curl -X POST "{HOST}/api/attendance/verify/nfc" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "tagId": "SODAM-123456"
  }'
```

## 🗑️ 레거시 경로 병행 안내 (v1.1)

- 레거시 경로: /attendance/check-in, /attendance/check-out
- 동작: 표준 로직으로 위임, 응답 헤더에 Deprecation/Warning 포함
- 전환 권고: /api/attendance/* 경로로 전환

## 📅 변경 이력 (v1.1 추가)

 날짜         | 버전  | 변경 내용                         | 작성자             
------------|-----|-------------------------------|-----------------
 2025-09-30 | 1.1 | Verify 2종 추가, 레거시 경로 병행 안내 반영 | Junie (Backend) 
