# BE Attendance 내부 작업계획 — 실행 프롬프트 v1.0

Role: Spring Boot Integrator · Exception Flow Designer · API Contract Reviewer · Security Compliance Auditor · Test
Designer · Documentation Steward · Observability Engineer · Deployment Readiness Checker

## 📋 개요

- 작성일: 2025-09-30 15:37 (KST)
- 작성자: Junie (Backend)
- 문서 유형: 가이드 / 프롬프트 / 작업계획
- 관련 문서:
    - FE-BE 타협안: ..\reports\FE-BE_Attendance_정합성_타협안_및_작업계획_v1.0_2025-09-30.md
    - BE As-Is: ..\reports\Attendance_BE_AsIs_통합_보고서_v1.0_2025-09-30.md
    - FE As-Is: ..\Attendance_FE_AsIs_통합_보고서_v1.0_2025-09-30.md
    - API 상세: ..\technical\api\AttendanceController_API_문서.md
    - JWT/RN 연동: ..\technical\JWT_인증_API_및_RN_연동_가이드.md

## 🎯 목적

본 문서는 백엔드 팀이 즉시 실행 가능한 형태의 “프롬프트(지시문)”로 구성된 내부 작업계획서입니다. RN 팀과 합의한 타협안을 기반으로, 최소 리스크로 빠르게 구현·검증·배포하기 위한 단계별 체크리스트, 산출물,
수락 기준을 제공합니다.

## 🧩 사용 방법

1) 아래 “전체 프롬프트(템플릿)” 블록을 그대로 복사하여 이슈 트래커 또는 AI 보조 도구에 입력합니다.
2) Vars 섹션의 변수(T0, OWNER, REVIEWERS 등)를 실제 값으로 치환합니다.
3) 산출물/체크리스트에 따라 브랜치/커밋/PR/테스트/문서화를 진행합니다.
4) 완료 후 “수락 기준”을 기준으로 셀프 리뷰 → 리뷰어 승인 → 배포 순서로 진행합니다.

---

## 🧱 전체 프롬프트(템플릿)

다음은 SODAM 백엔드 Attendance 내역에 대한 구현 실행 프롬프트입니다. 지시사항을 순서대로 수행하고, 각 단계 종료 시 산출물을 제출하세요.

[Role]

- 당신은 다음 역할을 동시에 수행합니다: Spring Boot Integrator, Exception Flow Designer, API Contract Reviewer, Security Compliance
  Auditor, Test Designer, Documentation Steward, Observability Engineer, Deployment Readiness Checker.

[Context]

- 레거시 FE 호출 경로와 BE 표준 경로 불일치 해소 및 Verify(사전 인증) 단계 수용이 목적입니다.
- BE 기준 베이스 경로는 /api/attendance 입니다.
- 현재 BE는 위치 인증을 check-in/out 내부에서 수행하며 Verify 전용 엔드포인트는 없습니다.
- RN 팀과 합의한 타협안에 따라 Verify 엔드포인트 추가, 구 경로 임시 수용, 예외 상태코드 정교화가 필요합니다.

[Vars]

- T0: 2025-09-30
- OWNER: Junie (Backend)
- REVIEWERS: {보안담당자}, {리드개발자}
- GRACE_PERIOD_DAYS: 14
- BASE_PATH: /api/attendance
- DEPRECATED_BASE_PATH: /attendance
- TARGET_ENV: local → staging → production

[Objectives]

1. Verify 엔드포인트 2종 추가(위치/카드):
    - POST /api/attendance/verify/location
    - POST /api/attendance/verify/nfc
2. 구 경로(DEPRECATED_BASE_PATH)를 2주간 병행 수용하는 프록시/리다이렉션층 제공(핵심 로직 단일화).
3. GlobalExceptionHandler를 보강하여 LocationVerificationException을 403으로 명시 매핑.
4. API 문서/Swagger 및 Postman 컬렉션 업데이트.
5. 통합 테스트 추가(경계 반경, 중복 출근, 권한, 미존재 엔티티, 예외 매핑).
6. 관측성(로그/MDC/메트릭) 키 추가 및 대시보드 반영.

[Constraints]

- 보안: JWT 시크릿/DB 비밀번호 하드코딩 금지, 환경변수 기반. 오류 메시지/로그는 한국어.
- 예외: 특정 예외를 구체적으로 매핑(403 위치 실패, 404 미존재, 400 비즈니스), ApiResponse 표준 포맷 유지.
- 아키텍처: Controller → Service → Repository 계층 구조 유지, 중복 로직 금지.
- 문서화: Swagger, Markdown, Javadoc(공개 메서드), 모든 텍스트 한국어.
- 테스트: 모킹 프레임워크 금지. 실제 컴포넌트 기반 테스트 지향.

[Deliverables]

- 코드: 컨트롤러/서비스/예외/설정/테스트 변경 사항 커밋
- 문서: API 스펙 v1.1, 변경 로그, Postman v1.1
- 운영: 대시보드/로그 키, 알림 룰(오류율 상승)

[Step-by-Step Tasks]

- BE-1: Verify 엔드포인트 추가
  a) Controller: POST /verify/location, /verify/nfc (BASE_PATH 하위)
  b) DTO: LocationVerifyRequest/Response, NfcVerifyRequest/Response
  c) Service: LocationVerificationService 확장(거리 반환), NfcVerificationService 신규(태그 유효성)
  d) 응답 스키마: { success: boolean, reason?: OUT_OF_RANGE|INVALID_TAG|DUPLICATE|PERMISSION, distance?: number, timestamp?:
  string }
  e) 에러 코드: LOCATION_VERIFICATION_FAILED 등 표준화
  f) Swagger 문서 및 예시 추가

- BE-2: 구 경로 임시 수용층(2주)
  a) Controller에 @PostMapping("/attendance/check-in") 등 구 경로 핸들러를 추가하되, 내부에서 표준 경로 로직으로 위임
  b) Deprecated 경고 로그, 응답 헤더 Warning 또는 Deprecation 헤더 포함
  c) 보안/권한 동일 적용 검증

- BE-3: 예외 매핑 정교화
  a) GlobalExceptionHandler: LocationVerificationException → 403
  b) 테스트 케이스: 위치 반경 밖 요청이 403으로 응답되는지 확인
  c) 문서/README에 상태코드 명시 업데이트

- BE-4: 문서/도구 업데이트
  a) docs/technical/api/* 업데이트 및 변경 이력 추가
  b) Postman 컬렉션 v1.1 내보내기 및 팀 공유 링크 업데이트
  c) 스웨거 그룹/태그 정리, 예제 페이로드 반영

- BE-5: 통합 테스트
  a) 반경 경계값(=허용 임계) 성공/실패 케이스
  b) 중복 출근, 미출근 퇴근, 권한 부족(사업주) 케이스
  c) 엔티티 미존재(404), 검증 실패(400/403) 케이스
  d) 캐시 무효화 동작 확인(@CacheEvict)

- BE-6: 관측성/모니터링
  a) 구조화 로깅(MDC: requestId, userId, endpoint) 필터 확인/보강
  b) 메트릭: attendance.checkin.total, attendance.verify.location, error.rate 등 카운터/타이머
  c) 대시보드: 스테이징 오류율/지연시간 모니터링, 알림 임계치 설정

[Checklists]

- 공통 완료 조건(DoD)
    - [ ] 단위/통합 테스트 통과, 커버리지 핵심 로직 ≥ 80%
    - [ ] Swagger 문서 최신화, 예시 포함
    - [ ] ApiResponse 오류코드/메시지 확인(한국어)
    - [ ] 로그에 PII 노출 없음, MDC 동작 확인
    - [ ] 성능 회귀 없음(기준 대비 ±5% 이내)

- BE-1 전용
    - [ ] DTO/Controller/Service/문서 일치
    - [ ] 위치 검증 distance 계산 정확도(허버사인 등) 검증
    - [ ] NFC 태그 유효성 정책 문서화(유효기간/형식)

- BE-2 전용
    - [ ] 구/신 경로 모두 동일 로직 경유
    - [ ] Deprecation 헤더 및 로그 남김
    - [ ] 보안 필터/권한 체크 동일 동작 검증

- BE-3 전용
    - [ ] 403 매핑 테스트 케이스 통과
    - [ ] 문서/예시 코드 상태코드 일치

[Timeline — 기준일 T0=2025-09-30]

- T0+1 (2025-10-01): 설계/스키마/문서 초안 고정
- T0+7 (2025-10-07): 구현 완료(BE-1~3), 스테이징 배포 준비
- T0+10 (2025-10-10): 스테이징 검증/부하 샘플링, RN 연동 점검
- T0+12 (2025-10-12): 운영 배포
- T0~T0+14 (2025-10-14): 구 경로 병행 운영
- T0+21 (2025-10-21): 구 경로 폐지/모니터링 종료

[Risk & Mitigations]

- 경로 병행 운영 중 보안/캐시 정책 이원화 → 동일 서비스 위임, 중복 설정 금지
- RN 전환 지연 → 병행 기간 연장 플랜 및 접근 로그 기반 리스크 알림
- 403 매핑 변경으로 FE 예외 핸들링 영향 → RN에 사전 공지 및 errorCode 기반 처리 가이드 제공

[Backout Plan]

- 운영 배포 24시간 내 주요 오류율 상승 시 즉시 롤백(직전 태그 재배포)
- 예외 매핑만 부분 롤백 가능한 토글 플래그 도입 고려(피처 플래그)

[Communication]

- 주 2회 싱크(화/금), 데일리 슬랙 스레드
- 스테이징 Swagger/Postman 링크 고정 공지, 구 경로 폐지 일정 공지 2회(2주/1주 전)

[Acceptance Criteria]

- RN 앱이 신규 경로/스키마로 체크인/체크아웃 정상 동작
- 403 위치 실패/404 미존재/400 비즈니스가 문서와 일치
- 구 경로 접근량이 폐지 시점에 0에 수렴

---

## 🔧 작업 항목과 세부 체크리스트(실행용)

1) BE-1 Verify 엔드포인트 추가

- 컨트롤러: AttendanceController에 /verify 하위 REST 추가 또는 전용 VerifyController 분리
- DTO 정의: LocationVerifyRequest/Response, NfcVerifyRequest/Response
- 서비스 구현: 거리 계산 반환, 태그 유효성 검증(스텁→정책 확정 후 강화)
- 에러코드/사유: OUT_OF_RANGE/INVALID_TAG/DUPLICATE/PERMISSION 등
- 문서/예시/테스트 포함

2) BE-2 구 경로 임시 수용층

- 컨트롤러에 구 경로 매핑 추가 @RequestMapping("/attendance")
- 내부 위임: 표준 로직 호출, Deprecation 로그/헤더 주입

3) BE-3 예외 매핑 403 분기

- GlobalExceptionHandler에서 LocationVerificationException → HttpStatus.FORBIDDEN
- 테스트 케이스 추가 및 문서 상태코드 일치

4) BE-4 문서/Swagger/Postman 업데이트

- 스웨거 요약/예시 최신화, 태그 정리
- Postman v1.1 내보내기 및 공유

5) BE-5 통합 테스트

- 경계 반경, 중복 출근, 미출근 퇴근, 권한 부족, 404/403/400 라우팅

6) BE-6 관측성 강화

- MDC/구조화 로깅/메트릭, 대시보드

## 🧪 수락 기준(Definition of Done)

- 통합 테스트 통과 + 스테이징 검증 시트 통과
- Swagger와 실제 응답(상태코드/에러코드)이 일치
- RN 측 QA에서 신규 경로/스키마로 정상 동작 확인
- 구 경로 폐지 전 최종 공지 완료

## 🔗 관련 문서

- FE-BE 타협안: ..\reports\FE-BE_Attendance_정합성_타협안_및_작업계획_v1.0_2025-09-30.md
- BE As-Is: ..\reports\Attendance_BE_AsIs_통합_보고서_v1.0_2025-09-30.md
- API 상세: ..\technical\api\AttendanceController_API_문서.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자             |
------------|-----|-------|-----------------|
 2025-09-30 | 1.0 | 최초 작성 | Junie (Backend) |
