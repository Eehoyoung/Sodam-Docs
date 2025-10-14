# FE-BE Attendance 정합성 타협안 및 작업계획 v1.0

Role: API Contract Reviewer · Spring Boot Integrator · Exception Flow Designer · Deployment Readiness Checker

## 📋 개요

- 작성일: 2025-09-30 15:23 (KST)
- 작성자: Junie (Backend)
- 문서 유형: 보고서 / 합의안 / 작업계획
- 관련 문서:
    - RN As-Is: ..\Attendance_FE_AsIs_통합_보고서_v1.0_2025-09-30.md
    - BE As-Is: .\Attendance_BE_AsIs_통합_보고서_v1.0_2025-09-30.md
    - API 상세: ..\technical\api\AttendanceController_API_문서.md

## 🎯 목적

RN 팀이 작성한 As-Is 문서를 근거로, 현재 FE와 BE 사이의 경로/스키마/흐름 차이를 정리하고, 상호 수용 가능한 **타협안**과 **단계별 작업계획**(BE와 RN 각각)을 문서화합니다. 본 문서는 실행
가능한 일정, 책임, 수락 기준을 포함하는 1급 보고서입니다.

## 🧩 RN As-Is 요약(핵심)

- 호출 패턴: "사전 인증(verify) → 본 처리(check-in/out)" 2단 호출(특히 NFC/위치)
- 경로 접두사: NFC verify는 "/api/attendance/nfc-verify" 사용, 위치 verify 및 다수 엔드포인트는 "/attendance/*" 사용
- ID/타입: storeId(number)와 workplaceId(string) 혼용 사례 존재
- 퇴근 경로: POST /attendance/{attendanceId}/check-out 형태 사례 문서화
- 에러 처리: 401 시 Refresh Queue 단일 비행, 거리(distance) 기반 UX 메시지, TanStack Query 캐시 무효화 전략 존재

## 🧭 BE As-Is 핵심

- 베이스 경로: /api/attendance (컨트롤러 레벨)
- Verify 엔드포인트: 별도 제공 없음(위치 인증은 check-in/out 내부에서 수행)
- DTO: employeeId(Long), storeId(Long), latitude/longitude(Double) 강제
- 퇴근 경로: POST /api/attendance/check-out (attendanceId path 변수 없음)
- 오류 응답: ApiResponse 표준 포맷 + errorCode (GlobalExceptionHandler)

## ⚠️ 정합성 이슈 목록(팩트)

1. 경로 접두사 혼용(/api/attendance vs /attendance)
2. Verify 엔드포인트 존재/부재( RN: 존재, BE: 비존재 )
3. ID/타입 불일치(storeId: number vs workplaceId: string)
4. 퇴근 경로의 path 변수 사용 여부( RN: /{attendanceId}/check-out vs BE: /check-out )
5. 오류코드/HTTP 상태 해석 차이(403 명세 vs 실제 400 가능 — BusinessException 매핑)
6. 캐시 무효화 타이밍/키 구성의 상이(문제는 아님, 공지 필요)

## 🤝 타협안(제안 및 합의 대상)

A. 경로 통일 정책

- 권장(합의안 A1): FE와 BE 모두 "/api/attendance/*"로 통일
    - RN은 baseURL을 {HOST}로 유지하고, 모든 경로를 "/api/attendance" 하위로 변경
    - BE는 기존과 동일(변경 없음)
- 이행기(옵션 A2): BE가 2주간 다음 경로를 임시 동시 허용(개발 후 배포 필요)
    - POST /attendance/check-in → /api/attendance/check-in 에 위임(리다이렉션/프록시)
    - POST /attendance/check-out → /api/attendance/check-out 에 위임
    - POST /attendance/location-verify → 신규 제공(아래 B안)

B. Verify 단계 호환성

- RN 2단 호출 유지 요청을 수용하여 BE에 다음 신규 엔드포인트 추가(설계 제안)
    - POST /api/attendance/verify/nfc
        - Req: { employeeId: Long, storeId: Long, nfcTagId: string, isCheckOut?: boolean }
        - Res: { success: boolean, reason?: 'OUT_OF_RANGE'|'INVALID_TAG'|'DUPLICATE'|'PERMISSION', timestamp?: string }
    - POST /api/attendance/verify/location
        - Req: { employeeId: Long, storeId: Long, latitude: number, longitude: number }
        - Res: { success: boolean, distance?: number, reason?: string }
- 본 처리(check-in/out)는 현재와 동일 스키마 유지(AttendanceRequestDto)

C. ID/타입 용어 통일

- 용어: storeId로 통일(Long)
- RN 작업: workplaceId(string) → storeId(number) 변환(파싱/검증 포함)

D. 퇴근 경로 표준화

- 표준: POST /api/attendance/check-out (attendanceId path 변수 사용하지 않음)
- RN 작업: 기존 /{attendanceId}/check-out 호출부를 /check-out 본문(AttendanceRequestDto) 방식으로 전환

E. 오류 응답 합의

- RN은 HTTP 상태 외에 errorCode 기반 처리(권장)
- BE는 위치 인증 실패를 403으로 명확히 매핑하도록 향후 개선(핸들러 분기) — 변경 작업은 BE 계획에 포함

F. 이행 기간(Grace) 및 폐지 일정

- T0(오늘)~T0+14일: BE가 구 경로( /attendance/* )와 신규 verify 엔드포인트 병행 제공 (BE 개발/배포 후 유효)
- T0+7일: RN이 신경로(/api/attendance/*)로 전환 완료 목표(부분 적용 가능)
- T0+21일: /attendance/* 구 경로 폐지 공지 및 모니터링 종료

## 🛠️ 작업 계획(우리의 계획 — BE)

- 담당 역할: Spring Boot Integrator, Exception Flow Designer, API Contract Reviewer
- 일정(제안)
    1) 설계·합의 고정: T0+1일 내
    2) 구현: T0+7일 내
    3) 스테이징 배포/검증: T0+10일 내
    4) 운영 배포: T0+12일 내
- 작업 항목
    - BE-1: Verify 엔드포인트 2종 추가(/verify/nfc, /verify/location) — 컨트롤러/서비스/문서
    - BE-2: /attendance/* → /api/attendance/* 프록시/리다이렉션 임시 수용층 추가(2주)
    - BE-3: GlobalExceptionHandler에서 LocationVerificationException → 403 매핑 분기 추가
    - BE-4: API 문서/Swagger 업데이트 및 Postman 컬렉션 배포
    - BE-5: 통합 테스트 추가(위치 범위 경계, 중복 출근, 권한 검증)
    - BE-6: 모니터링 대시보드/로그 키(엔드포인트, errorCode) 추가
- 산출물
    - API 스펙 문서 v1.1, 변경 로그, Postman v1.1, 스테이징 결과 보고서
- 리스크/완화
    - 경로 병행 운영 시 캐시/보안 정책 이중화 주의 → 동일 서비스 로직으로 위임(핵심 로직 단일화)

## 📱 작업 계획(전달용 — RN)

- 담당 역할: RN Lead (Junie), App 팀
- 일정(제안)
    1) 경로 전환 초안 PR: T0+3일 내 (/attendance/* → /api/attendance/*)
    2) Verify 호출 통합: T0+5일 내 (신규 /api/attendance/verify/* 사용)
    3) 타입 정합화: T0+7일 내 (workplaceId→storeId number 변환, 유효성 추가)
    4) QA/릴리즈: T0+10~12일
- 작업 항목
    - RN-1: Axios 클라이언트 기본 경로 점검 및 하드코딩 경로 정리
    - RN-2: NFC/위치 verify 호출부를 /api/attendance/verify/* 로 정규화
    - RN-3: check-out 호출을 /api/attendance/check-out 으로 통일(attendanceId path 제거)
    - RN-4: errorCode 기반 에러 UX 공통화(LOCATION_VERIFICATION_FAILED, ENTITY_NOT_FOUND 등)
    - RN-5: TanStack Query 캐시 키/무효화 정책을 BE 기준 엔드포인트에 맞게 재검토
- 산출물
    - RN API 연동 PR, 릴리즈 노트, QA 시트(검증 케이스 포함)

## 🔁 매핑 가이드(요약)

- FE 현재 → BE 표준
    - POST /attendance/check-in → POST /api/attendance/check-in (본문 동일)
    - POST /attendance/{id}/check-out → POST /api/attendance/check-out (본문: AttendanceRequestDto)
    - POST /attendance/location-verify → POST /api/attendance/verify/location (신규 제공 예정)
    - POST /api/attendance/nfc-verify → POST /api/attendance/verify/nfc (경로 표준화)
    - workplaceId(string) → storeId(number)

## 🧪 테스트/검증 전략

- BE: 통합 테스트(경계값 반경, 중복 출근/미출근 퇴근, 권한 실패, 404 엔티티 미존재), Swagger 문서 검증
- RN: 단위+통합 테스트(Jest/React Native Testing Library), 401 Refresh 큐 상호작용 검증, 실제 디바이스 위치/NFC 샘플링

## 📢 커뮤니케이션/릴리즈 정책

- 주 2회 싱크(화/금), 데일리 슬랙 스레드로 진행 상황 공유
- 스테이징 환경 공개 및 Postman/Swagger 링크 고정 공지
- 폐지 예정 경로(/attendance/*) 2주 전 미리 공지, 실제 폐지 1주 전 재공지

## ✅ 수락 기준

- RN PR 머지 및 신규 경로/스키마로 앱 정상 동작 확인(체크인/체크아웃/조회)
- BE 스테이징/운영 배포 후 로그/메트릭에 오류율 증가 없음(±1%p 내)
- 기존 구 경로에 대한 접근이 운영 종료 시점(T0+21일)에 0으로 수렴

## 🔗 관련 문서

- RN As-Is: ..\Attendance_FE_AsIs_통합_보고서_v1.0_2025-09-30.md
- BE As-Is: .\Attendance_BE_AsIs_통합_보고서_v1.0_2025-09-30.md
- API 상세: ..\technical\api\AttendanceController_API_문서.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자             |
------------|-----|-------|-----------------|
 2025-09-30 | 1.0 | 최초 작성 | Junie (Backend) |
