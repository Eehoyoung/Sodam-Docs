# Attendance FE As-Is 통합 보고서 v1.0

## 📋 Overview

- Created: 2025-09-30 02:32 (KST)
- Author: Junie (RN Lead)
- Type: Report / Integration / Alignment
- Related Issue: FE↔BE Attendance 동작 정합성 설명 및 협의 기반 자료

## 🎯 Purpose

백엔드 팀과의 정확한 정합성을 위해, 현재(As-Is) React Native 프런트엔드에서 Attendance(근태) 기능이 어떻게 동작하는지 “코드 변경 없이 그대로” 상세히 설명합니다. 실제 호출 경로,
요청/응답 페이로드, 검증 순서, 캐시 전략, 에러 처리, 네이티브 연동(NFC/위치)까지 모두 포함합니다.
이 문서는 상호 타협점을 찾기 위한 1급 보고서(최상세)입니다.

## 🧭 환경/조건

- OS/셸: Windows + PowerShell, 경로는 \\ 사용
- RN/React/Node: RN 0.81.0(New Arch, Hermes) / React 19.1.0 / Node >=18
- Android: Compile/Target SDK 36, Min 24
- 인증: Kakao OAuth + JWT, Authorization: Bearer {accessToken}
- API Base URL: http://10.0.2.2:8080 (에뮬레이터 기준) — src\common\utils\api.ts
- 요청 타임아웃: 10초
- 토큰 자동 주입/갱신: axios 인터셉터로 Access 자동 주입, 401 시 Refresh 큐(single-flight)로 재시도
- 테스트: Jest preset ‘react-native’ (본 문서는 동작 기록/명세, 코드 변경/테스트 추가 없음)

## 🧩 FE 아키텍처(Attendance 관련)

- 클라이언트: src\common\utils\api.ts (Axios 인스턴스, 인터셉터)
- 도메인 서비스:
    - src\features\attendance\services\attendanceService.ts — 표준 CRUD/체크인/체크아웃/통계/검증(위치) 집약
    - src\features\attendance\services\nfcAttendanceService.ts — NFC 사전 인증(verify) 및 태그 파싱/유효성
    - src\features\attendance\services\locationAttendanceService.ts — 위치 사전 인증(verify) 및 거리 계산
- 훅/상태: src\features\attendance\hooks\useAttendanceQueries.ts (TanStack Query)
- UI(예): src\features\attendance\components\NFCAttendance.tsx, LocationAttendance.tsx, screens/AttendanceScreen.tsx

## 🔗 엔드포인트 사용 현황(As-Is)

중요: FE는 ‘사전 인증(verify)’ 단계와 ‘본 처리(check-in/out)’ 단계를 분리 호출합니다.

1) 사전 인증(verify)

- NFC: POST /api/attendance/nfc-verify
    - 요청(JSON): { employeeId: number, storeId: number, nfcTagId: string } (+ 퇴근 시 isCheckOut: true 추가)
    - 응답(JSON): { success: boolean, message?: string, timestamp?: string }
    - 구현: src\features\attendance\services\nfcAttendanceService.ts
        - verifyCheckInByNFC(request) → POST '/api/attendance/nfc-verify'
        - verifyCheckOutByNFC({...request, isCheckOut: true}) → 同 엔드포인트
- 위치: POST /attendance/location-verify
    - 요청(JSON): { employeeId: string, workplaceId: string, latitude: number, longitude: number }
    - 응답(JSON): { success: boolean, distance?: number, message?: string }
    - 구현: src\features\attendance\services\attendanceService.ts
        - verifyLocationAttendance(employeeId, workplaceId, latitude, longitude) → POST '/attendance/location-verify'
    - 참고: 별도 유틸 src\features\attendance\services\locationAttendanceService.ts
        - verifyCheckInByLocation/verifyCheckOutByLocation 은 '/api/attendance/location-verify' 로 호출하도록 정의되어 있으나,
          실제 스크린/훅에서는 attendanceService.verifyLocationAttendance('/attendance/location-verify')를 주로 사용합니다.

2) 본 처리

- 출근: POST /attendance/check-in
    - 요청(JSON, CheckInRequest): { workplaceId: string, note?: string, latitude?: number, longitude?: number }
    - 응답(JSON, AttendanceRecord)
    - 구현: attendanceService.checkIn(checkInData)
- 퇴근: POST /attendance/{attendanceId}/check-out
    - 요청(JSON, CheckOutRequest): { workplaceId: string, note?: string, latitude?: number, longitude?: number }
    - 응답(JSON, AttendanceRecord)
    - 구현: attendanceService.checkOut(attendanceId, checkOutData)

3) 조회/통계/관리

- 목록: GET /attendance?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD[&workplaceId=string][&employeeId=string][&status=enum]
    - 구현: attendanceService.getAttendanceRecords(filter)
- 단건: GET /attendance/{attendanceId}
    - 구현: attendanceService.getAttendanceById
- 현재상태: GET /attendance/current?workplaceId=string
    - 구현: attendanceService.getCurrentAttendance
- 통계(기간): GET /attendance/statistics?startDate=...&endDate=...
    - 구현: attendanceService.getAttendanceStatistics
- 통계(직원): GET /attendance/statistics/employee/{employeeId}?startDate=...&endDate=...
    - 구현: attendanceService.getEmployeeAttendanceStatistics
- 통계(매장): GET /attendance/statistics/workplace/{workplaceId}?startDate=...&endDate=...
    - 구현: attendanceService.getWorkplaceAttendanceStatistics
- 상태 일괄 변경: PUT /attendance/batch-status { attendanceIds: string[], status: AttendanceStatus }
    - 구현: attendanceService.batchUpdateStatus
- 수정/삭제: PUT /attendance/{id}, DELETE /attendance/{id}

주의(경로 접두사):

- NFC verify는 '/api/attendance/nfc-verify' (api prefix 포함)
- 위치 verify는 '/attendance/location-verify' (api prefix 미포함)
- 본 처리/조회/통계 대부분은 '/attendance/*' (api prefix 미포함)
- Axios baseURL은 'http://10.0.2.2:8080' 이므로 최종 호출은 위 경로 그대로 결합됩니다.

## 🧾 데이터 타입(As-Is)

- AttendanceRecord (일부 주요 필드)
    - id: string, employeeId: string, employeeName: string, workplaceId: string, workplaceName: string
    - date: string, checkInTime: string, checkOutTime?: string, status: 'PENDING'|'CHECKED_IN'|'CHECKED_OUT'|...
    - workHours?: number, breakTime?: number, note?: string, createdAt: string, updatedAt: string
- CheckInRequest / CheckOutRequest
    - { workplaceId: string, note?: string, latitude?: number, longitude?: number }
- AttendanceFilter
    - { startDate: string, endDate: string, workplaceId?: string, employeeId?: string, status?: AttendanceStatus }
- NFCVerifyRequest/Response
    - Req: { employeeId: number, storeId: number, nfcTagId: string }
    - Res: { success: boolean, message?: string, timestamp?: string }
- LocationVerifyRequest/Response
    - Req: { employeeId: number, storeId: number, latitude: number, longitude: number } (locationAttendanceService.ts)
    - Res: { success: boolean, distance?: number, message?: string, timestamp?: string }
    - 실제 verifyLocationAttendance 호출부는 { employeeId: string, workplaceId: string, latitude, longitude } 형태로 '
      /attendance/location-verify' 사용

## 🔁 호출 시퀀스(UX 관점) — 예시

1) NFC 출근

- 사용자: NFC 스캔 버튼 → react-native-nfc-manager로 태그 읽기
- FE: 태그 파싱(parseNFCTagData) 및 유효성 검사(isNFCTagValid)
- FE→BE: POST /api/attendance/nfc-verify { employeeId, storeId, nfcTagId }
- 응답: { success: true } → 이후 FE에서 출근 본 처리 호출
- FE→BE: POST /attendance/check-in { workplaceId, latitude?, longitude? }
- 캐시: TanStack Query로 current/records/store/employee 관련 캐시 set/invalidate

2) NFC 퇴근

- 사용자: NFC 스캔 →
- FE→BE: POST /api/attendance/nfc-verify { ..., isCheckOut: true }
- success 시 FE→BE: POST /attendance/{attendanceId}/check-out { workplaceId, latitude?, longitude? }
- 캐시 무효화 동일

3) 위치 기반 출근/퇴근

- 사용자: 위치 권한 허용 → 현재 좌표 획득
- FE→BE: POST /attendance/location-verify { employeeId, workplaceId, latitude, longitude }
- 응답 success 및 distance 확인 후,
    - 출근: POST /attendance/check-in {...}
    - 퇴근: POST /attendance/{id}/check-out {...}

4) 조회/통계/현재상태

- useAttendanceQueries.ts의 다양한 useQuery 훅으로 주기적 조회
    - current: 60초 refetch, staleTime 30초
    - store: refetchInterval 5분
    - records: stale 2분, employee: 3분, statistics: 15분 등

## ⚙️ 클라이언트/인증/에러 처리(As-Is)

- Authorization 주입: 요청 전 인터셉터가 TokenManager.getAccess()로 Bearer 헤더 설정
- 401 처리: 응답 인터셉터가 refreshAccessToken()을 통해
    - 1차: POST {BASE}/api/auth/refresh { refreshToken }
    - 2차 폴백: POST {BASE}/api/refresh { refreshToken }
    - 성공 시 원요청 재시도, 실패 시 토큰 정리 및 onUnauthorized 콜백
- 타임아웃: 10초, 네트워크 오류/서버 오류 시 에러 그대로 throw → 훅에서 handleQueryError 사용
- 로깅: [API Request] 로그 + attendanceService 내부 logger.error 호출(일부 메시지는 공란으로 남음)

## 📱 네이티브(NFC/위치) 연동 요약

- NFC: react-native-nfc-manager
    - NfcManager.start(), isSupported(), isEnabled() 확인
    - 태그 읽기 후 parseNFCTagData → { storeId, timestamp } 도출 또는 단순 태그 ID에서 숫자 추출
    - isNFCTagValid(timestamp, 30) 기본 30분 유효 정책
    - 관리자 측 태그 생성 로직(createNFCTagData), writeNFCTag 검증 루틴 포함(실제 태그 쓰기는 네이티브 모듈 처리)
- 위치: Haversine 계산(isWithinRadius/calculateDistance) 유틸 제공
    - 서버 검증 전 FE 단 제어 로직으로 반경/거리 안내 가능

## 🧪 캐싱/동시성(As-Is)

- TanStack Query
    - 출근/퇴근 성공 시 current 캐시 set, store/employee/records 계열 invalidate
    - refetchInterval 사용(현재상태: 60초)
    - retry: 404는 재시도 안 함, 나머지는 2회까지
- Refresh 단일 비행 큐
    - 동시 401 다건 발생 시 1회만 refresh, 나머지는 큐 대기 후 재시도

## ⚠️ 정합성 이슈(사실 기반)

- 경로 접두사 불일치
    - NFC verify: '/api/attendance/nfc-verify'
    - 위치 verify: '/attendance/location-verify'
    - 본 처리/조회: '/attendance/*'
- 명칭/타입 불일치
    - NFC verify는 storeId(number), FE 일반 흐름은 workplaceId(string)
    - verifyLocationAttendance는 employeeId/workplaceId string 파라미터를 그대로 전송
- 사전 인증 엔드포인트 설계
    - NFC/위치 모두 ‘verify → check-in/out’ 2단 분리 호출을 전제로 FE 구현
    - verify 응답은 success/거리 중심이며, 본 처리에서 AttendanceRecord를 반환

## 🤝 타협안 제안(To reach common ground)

1) 경로 통일

- 선택 A(권장): 모든 Attendance REST를 '/api/attendance/*'로 통일 (BE가 게이트웨이 통일)
    - 예) verify: POST /api/attendance/verify/nfc, /api/attendance/verify/location
    - 본 처리: POST /api/attendance/check-in, POST /api/attendance/{id}/check-out 등
- 선택 B: FE가 baseURL을 {BASE}/api로 고정하고 모든 경로는 '/attendance/*' (단, 현재 코드는 일부 '/api/attendance/*'를 직접 사용 중 → BE가 임시로 동시 지원
  필요)

2) 명칭/타입 정합

- storeId ↔ workplaceId 용어 통일: storeId로 고정(정수) 또는 workplaceId로 고정(문자열) 중 택1
- 권장: 정수형 storeId로 통일, FE는 문자열->정수 변환 수행(현재 NFC 흐름처럼)

3) 사전 인증/본 처리 인터페이스 명확화

- verify 응답 스키마 제안: { success: boolean, reason?: 'OUT_OF_RANGE'|'INVALID_TAG'|'DUPLICATE'|'PERMISSION', distance?:
  number, hint?: string }
- 본 처리 성공 시 AttendanceRecord 반환

4) 이행 기간(그레이스)

- 2주간 BE가 '/api/attendance/nfc-verify'와 '/attendance/location-verify' 모두 허용
- 최종 합의 후 1주 Grace 내 FE가 경로/타입 리팩터링 PR 제출

## 🔍 예시 요청/응답

- NFC 출근 검증(curl)

```
POST {BASE}/api/attendance/nfc-verify
{
  "employeeId": 101,
  "storeId": 12,
  "nfcTagId": "STORE_12_20250901093000"
}
→ { "success": true, "timestamp": "2025-09-30T00:30:00.000Z" }
```

- 위치 출근 검증(curl)

```
POST {BASE}/attendance/location-verify
{
  "employeeId": "101",
  "workplaceId": "12",
  "latitude": 37.5665,
  "longitude": 126.9780
}
→ { "success": true, "distance": 12.4 }
```

- 출근 본 처리(curl)

```
POST {BASE}/attendance/check-in
{
  "workplaceId": "12",
  "latitude": 37.5665,
  "longitude": 126.9780
}
→ AttendanceRecord(JSON)
```

- 퇴근 본 처리(curl)

```
POST {BASE}/attendance/ATT-20250929-0001/check-out
{
  "workplaceId": "12"
}
→ AttendanceRecord(JSON)
```

## 🧷 에러/엣지 케이스

- 401: 자동 refresh 재시도, 실패 시 FE는 토큰 정리 후 로그인 유도
- 타임아웃/네트워크: 10초 후 실패 반환, 훅에서 handleQueryError로 표준화 메시지 노출
- NFC 태그 파싱 실패: parseNFCTagData가 null 반환 → 사용자 안내 Toast/Alert
- NFC 유효시간 초과: isNFCTagValid(timestamp, 30) 실패 시 재스캔 요구
- 위치 오차: isWithinRadius/calculateDistance로 사전 안내 가능, 서버 검증에서 최종 판단
- 중복 처리: 같은 출근/퇴근의 중복 제출 방지는 서버에서 idempotency/by status 판단을 권장

## 📚 참조 경로(코드)

- 공통 API: src\common\utils\api.ts
- Attendance 서비스: src\features\attendance\services\attendanceService.ts
- NFC 서비스: src\features\attendance\services\nfcAttendanceService.ts
- 위치 서비스: src\features\attendance\services\locationAttendanceService.ts
- 쿼리 훅: src\features\attendance\hooks\useAttendanceQueries.ts
- NFC UI: src\features\attendance\components\NFCAttendance.tsx
- 위치 UI: src\features\attendance\components\LocationAttendance.tsx

## ✅ 수락 기준(Alignment 관점)

- BE가 본 문서 기준으로 엔드포인트/스키마/용어( storeId/workplaceId ) 차이를 인지
- 양측이 경로/명칭/사전인증 정책의 통일 플랜에 합의
- 단기적으로 FE As-Is 요청을 모두 수용하는 백엔드 수용층 정의(위 ‘이행 기간’ 참고)

## 📅 Change History

- 2025-09-30 v1.0 — 최초 작성 (Junie)
