# 백엔드 팀 협조 요청서 (Backend Team Collaboration Request)

## 📋 문서 정보 (Document Information)

- **문서 번호**: COLLAB-BE-2025-10-11-001
- **작성일**: 2025-10-11
- **작성자**: 프론트엔드 팀 (Front-End Team Lead - Junie)
- **수신**: 백엔드 개발팀
- **문서 유형**: 공식 업무 협조 요청서 (Official Collaboration Request)
- **우선순위**: High (긴급 - 2개월 내 프로젝트 완성 목표)
- **관련 프로젝트**: 소담(SODAM) 아르바이트 근태 및 급여 관리 애플리케이션
- **기술 스택**: React Native 0.81.0, React 19.1.0, Spring Boot (Backend)

---

## 🎯 요청 목적 (Purpose)

본 협조 요청서는 소담 프로젝트의 2개월 내 완성을 목표로, 프론트엔드 팀에서 발견한 **Critical 및 High 우선순위 API 관련 문제**를 해결하고, **백엔드 서비스 연동을 완성**하기 위해 백엔드 팀의
긴밀한 협조를 요청하는 공식 문서입니다.

### 배경 (Background)

- **프로젝트 완성 기한**: ~2025-12-11 (약 2개월)
- **현재 API 통합 현황**: Phase 0-2 완료 (Attendance, Wage, Payroll, Store, QnA, TimeOff, User, Master)
- **발견된 주요 문제**: API 엔드포인트 불일치 3건, 서비스 미연동 5건, 스키마 미확정 2건
- **영향 범위**: 4개 역할(Personal, Employee, Master, Manager) 사용자 기능 전체

---

## ⚠️ 현재 프론트엔드 팀 문제사항 (Current Issues)

### Critical 문제 (즉시 수정 필요)

#### **[CRITICAL-BE-001] Attendance 엔드포인트 경로 불일치** ⛔

- **문제 설명**:
    - 현재 FE는 NFC verify를 `/api/attendance/nfc-verify`로 호출
    - 위치 verify는 `/attendance/location-verify`로 호출 (api prefix 누락)
    - 본 처리는 `/attendance/check-in`, `/attendance/{id}/check-out` 사용
    - **결과**: 엔드포인트 패턴이 혼재되어 유지보수성 저하 및 신규 개발자 혼란 유발

- **요청 사항**:
    1. 모든 Attendance 엔드포인트를 `/api/attendance/*` 패턴으로 통일
    2. Verify 엔드포인트를 `/api/attendance/verify/nfc`, `/api/attendance/verify/location`으로 재설계
    3. 본 처리 엔드포인트를 `/api/attendance/check-in`, `/api/attendance/check-out`으로 변경 (경로 파라미터 제거 또는 유지 여부 협의)

- **FE 영향 파일**:
    - `src\features\attendance\services\attendanceService.ts`
    - `src\features\attendance\services\nfcAttendanceService.ts`
    - `src\features\attendance\services\locationAttendanceService.ts`

- **협조 요청 일정**: Week 1 (즉시)
- **담당자 협의 필요**: BE Lead + FE Lead

- **참고 문서**:
    - [Attendance FE As-Is 보고서](../features/attendance/Attendance_FE_AsIs_통합_보고서_v1.0_2025-09-30.md) (현재 사용 중인 엔드포인트 상세)
    - [Phase Final Compliance Checklist](../api/Phase_Final_Compliance_Checklist_and_Gap_Plan_2025-10-05.md) (GAP-1 항목)

---

#### **[CRITICAL-BE-002] storeService API 미연동** ⛔

- **문제 설명**:
    - `MasterMyPageScreen`에서 매장 목록을 하드코딩된 `mockStores` 배열로 표시 중
    - `storeService.getMasterStores(userId)` 메서드는 구현되어 있으나 실제 BE 엔드포인트 미연동
    - `PersonalUserScreen`에서도 다중 매장 데이터를 로컬 하드코딩으로 관리 중
    - **결과**: 사장(Master) 및 개인(Personal) 역할 사용자의 매장 데이터가 영속화되지 않음, 앱 재시작 시 데이터 소실

- **요청 사항**:
    1. **GET** `/api/stores/master/{userId}` 엔드포인트 정상 동작 확인
        - 응답 스키마 확정 필요:
          ```json
          {
            "success": true,
            "data": [
              {
                "id": number,
                "storeName": string,
                "businessNumber": string,
                "storePhoneNumber": string,
                "businessType": string,
                "storeCode": string,
                "fullAddress": string,
                "storeStandardHourWage": number,
                "employeeCount": number,
                "createdAt": string (ISO 8601),
                "updatedAt": string
              }
            ]
          }
          ```
    2. **POST** `/api/stores` (매장 생성) 엔드포인트 요청/응답 스키마 확정
    3. **PUT** `/api/stores/{storeId}/location` 정상 동작 확인 (이미 구현됨, 검증 필요)

- **FE 영향 파일**:
    - `src\features\myPage\screens\MasterMyPageScreen.tsx` (809줄, line 83-126 mockStores 제거 필요)
    - `src\features\myPage\screens\PersonalUserScreen.tsx` (1527줄, line 84-89 stores 하드코딩 제거 필요)
    - `src\features\store\services\storeService.ts`

- **협조 요청 일정**: Week 1 (즉시)
- **담당자 협의 필요**: BE Engineer + FE Lead

- **참고 문서**:
    - [Role-Based Feature Matrix](../overview/Role_Based_Feature_Matrix_and_Issues_v1.0_2025-10-11.md) (Critical-3,
      High-4 항목)

---

#### **[CRITICAL-BE-003] laborInfoService API 미구현** ⛔

- **문제 설명**:
    - `EmployeeMyPageRNScreen` 및 `MasterMyPageScreen`에서 노무 정보(최저임금 등)를 하드코딩으로 표시
    - 현재 코드: `{ minimumWage: 9620, year: 2024, weeklyMaxHours: 40, overtimeRate: 1.5 }` (고정값)
    - **결과**: 실시간 노무 정보 업데이트 불가, 법적 기준 변경 시 앱 업데이트 필수

- **요청 사항**:
    1. **GET** `/api/labor-info/current` 엔드포인트 신규 구현 요청
        - 응답 스키마:
          ```json
          {
            "success": true,
            "data": {
              "minimumWage": number,
              "year": number,
              "weeklyMaxHours": number,
              "overtimeRate": number,
              "updatedAt": string (ISO 8601)
            }
          }
          ```
    2. 또는 기존 `/api/labor-info` 엔드포인트 활용 방안 협의 (현재 개별 ID 기반 조회만 가능)

- **FE 영향 파일**:
    - `src\features\myPage\screens\EmployeeMyPageRNScreen.tsx` (line 58 하드코딩)
    - `src\features\myPage\screens\MasterMyPageScreen.tsx` (line 152 하드코딩)
    - `src\features\info\services\laborInfoService.ts` (신규 메서드 추가 필요)

- **협조 요청 일정**: Week 1-2
- **담당자 협의 필요**: BE Engineer + FE Engineer

- **참고 문서**:
    - [Role-Based Feature Matrix](../overview/Role_Based_Feature_Matrix_and_Issues_v1.0_2025-10-11.md) (High-5 항목)

---

### High 우선순위 문제

#### **[HIGH-BE-001] Workplace API 전체 구현 요청** 🔴

- **문제 설명**:
    - `WorkplaceListScreen` 및 `WorkplaceDetailScreen`이 구현되어 있으나 네비게이션 미통합 (API 미연동)
    - Master 사용자의 다중 매장 관리 기능이 불완전
    - **결과**: 매장 위치 관리, 매장별 직원 배치, 매장별 근태 현황 조회 불가

- **요청 사항**:
    1. Workplace CRUD API 전체 스펙 확정:
        - **GET** `/api/workplaces` (전체 조회)
        - **GET** `/api/workplaces/{workplaceId}` (단건 조회)
        - **POST** `/api/workplaces` (생성)
        - **PUT** `/api/workplaces/{workplaceId}` (수정)
        - **DELETE** `/api/workplaces/{workplaceId}` (삭제)
    2. Workplace-Employee 관계 API:
        - **GET** `/api/workplaces/{workplaceId}/employees` (매장 직원 목록)
        - **POST** `/api/workplaces/{workplaceId}/employees` (직원 배치)
    3. 응답 스키마 확정 (위치 좌표, 근무 반경 등 포함)

- **FE 영향 파일**:
    - `src\features\workplace\screens\WorkplaceListScreen.tsx` (구현됨, 미통합)
    - `src\features\workplace\screens\WorkplaceDetailScreen.tsx` (구현됨, 미통합)
    - `src\features\workplace\services\workplaceService.ts` (신규 생성 필요)

- **협조 요청 일정**: Week 3-4
- **담당자 협의 필요**: BE Lead + FE Lead

- **참고 문서**:
    - [Work Plan v1.1](../project-management/Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md) (Week 3-4 Sprint, F-003
      항목)

---

#### **[HIGH-BE-002] Salary/Payroll API 응답 스키마 재확인** 🔴

- **문제 설명**:
    - `payrollService`는 Phase 1에서 구현 완료되었으나, UI 훅/스크린 미연결 상태
    - 급여명세서 PDF 생성 로직 및 데이터 포맷 미확정
    - **결과**: `SalaryListScreen` 통합 지연, 급여 조회 기능 미완성

- **요청 사항**:
    1. **GET** `/api/payroll/employee/{employeeId}` 응답 스키마 재확인:
        - 급여 기간(startDate, endDate), 총 근무시간, 기본급, 수당, 공제액, 실수령액 포함 여부
    2. **GET** `/api/payroll/{payrollId}/pdf` 엔드포인트 구현 여부 확인
        - PDF 응답 타입(blob, base64, URL) 확정
    3. 월별 급여 집계 API 추가 필요 여부 협의

- **FE 영향 파일**:
    - `src\features\salary\services\payrollService.ts` (Phase 1 완료)
    - `src\features\salary\screens\SalaryListScreen.tsx` (미통합)
    - `src\features\salary\screens\SalaryDetailScreen.tsx` (미구현)

- **협조 요청 일정**: Week 2-3
- **담당자 협의 필요**: BE Engineer + FE Engineer

- **참고 문서**:
    - [Work Plan v1.1](../project-management/Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md) (Week 2, S-002 / Week
      3-4, F-002 항목)

---

### Medium 우선순위 요청

#### **[MEDIUM-BE-001] 에러 코드 표준화** 🟡

- **문제 설명**:
    - 현재 BE 응답의 에러 코드 및 메시지 형식이 엔드포인트마다 상이
    - FE에서 통일된 에러 처리 로직 구현 어려움

- **요청 사항**:
    1. 표준 에러 응답 포맷 정의:
       ```json
       {
         "success": false,
         "error": {
           "code": "AUTH_001" | "VALIDATION_002" | "NOT_FOUND_003" | ...,
           "message": "User-friendly error message",
           "details": {} (optional)
         }
       }
       ```
    2. 공통 에러 코드 문서화 (docs/api/error-codes.md 생성)
    3. 401 Unauthorized 시 Refresh Token 로직 표준화

- **협조 요청 일정**: Week 4
- **담당자 협의 필요**: BE Lead + FE Lead + Security

---

#### **[MEDIUM-BE-002] 인증/권한 표준화** 🟡

- **문제 설명**:
    - 현재 Kakao OAuth + JWT 기반 인증은 동작하나, 역할(Role) 기반 권한 체계가 BE에서 일부 누락
    - FE에서는 4개 역할(EMPLOYEE, MASTER, MANAGER, Personal)을 구분하나, BE 응답에서 역할 정보가 불명확한 경우 발생

- **요청 사항**:
    1. 사용자 정보 응답에 `role` 필드 필수 포함 확인
    2. 역할별 접근 가능 엔드포인트 명세 문서화
    3. JWT Refresh 로직 재확인 (FE axios 인터셉터에서 401 시 자동 재시도)

- **협조 요청 일정**: Week 2
- **담당자 협의 필요**: BE + FE + Security

---

## 📊 API 갭 요약표 (API Gap Summary)

| 항목 ID           | 우선순위       | API 엔드포인트                     | 현재 상태   | 요청 사항          | 담당자               | 예상 일정    |
|-----------------|------------|-------------------------------|---------|----------------|-------------------|----------|
| CRITICAL-BE-001 | ⛔ Critical | `/api/attendance/verify/*`    | 경로 불일치  | 엔드포인트 표준화      | BE Lead + FE Lead | Week 1   |
| CRITICAL-BE-002 | ⛔ Critical | `/api/stores/master/{userId}` | 미연동     | 응답 스키마 확정 및 연동 | BE Engineer       | Week 1   |
| CRITICAL-BE-003 | ⛔ Critical | `/api/labor-info/current`     | 미구현     | 신규 엔드포인트 구현    | BE Engineer       | Week 1-2 |
| HIGH-BE-001     | 🔴 High    | `/api/workplaces/*`           | 미구현     | CRUD 전체 구현     | BE Lead           | Week 3-4 |
| HIGH-BE-002     | 🔴 High    | `/api/payroll/*`              | 스키마 미확정 | PDF 생성 및 응답 확정 | BE Engineer       | Week 2-3 |
| MEDIUM-BE-001   | 🟡 Medium  | 전체 API                        | 불일치     | 에러 코드 표준화      | BE Lead           | Week 4   |
| MEDIUM-BE-002   | 🟡 Medium  | 인증 API                        | 부분 누락   | 역할 기반 권한 명세    | BE + Security     | Week 2   |

---

## 🤝 협조 요청 프로세스 (Collaboration Process)

### 정기 미팅 제안

- **주기**: 매주 월요일, 목요일 오전 10:00 (30분)
- **참석자**: FE Lead, BE Lead, 관련 Engineer(s)
- **목적**: API 스펙 확정, 진척도 확인, 이슈 해결

### 긴급 협의 채널

- **Slack**: `#fe-be-sync` 채널 (실시간 소통)
- **GitHub**: Issue 태그 `[BE-Collaboration]` (공식 기록)

### 협의 결과 문서화

- **위치**: `docs/api-decisions/`
- **형식**: `API_Decision_YYMMDD_<topic>.md`
- **예시**: `API_Decision_251011_Attendance_Endpoint_Standardization.md`

---

## 📅 요청 일정 (Timeline)

### Week 1 (즉시)

- [x] 본 협조 문서 전달
- [ ] CRITICAL-BE-001: Attendance 엔드포인트 표준화 협의 및 구현
- [ ] CRITICAL-BE-002: storeService API 연동 완료
- [ ] 첫 협의 미팅 (월요일 오전 10시)

### Week 1-2

- [ ] CRITICAL-BE-003: laborInfoService API 구현 및 배포
- [ ] MEDIUM-BE-002: 인증/권한 표준화 협의

### Week 2-3

- [ ] HIGH-BE-002: Salary/Payroll API 스키마 최종 확정
- [ ] FE 통합 테스트 지원

### Week 3-4

- [ ] HIGH-BE-001: Workplace API 전체 구현
- [ ] MEDIUM-BE-001: 에러 코드 표준화 문서 작성

---

## 📚 참고 문서 (Reference Documents)

### 프론트엔드 현황 문서

1. [Role-Based Feature Matrix and Issues v1.0](../overview/Role_Based_Feature_Matrix_and_Issues_v1.0_2025-10-11.md)
    - 역할별 기능 분류 및 발견된 7개 문제점 상세

2. [Screen Navigation Flow v1.1](../overview/Screen_Navigation_Flow_v1.0_2025-10-11.md)
    - 24개 스크린 인벤토리 및 네비게이션 아키텍처

3. [Work Plan 2M Timeline v1.1](../project-management/Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md)
    - 2개월 완성 계획 및 팀별 협의 포인트

4. [Attendance FE As-Is 통합 보고서 v1.0](../features/attendance/Attendance_FE_AsIs_통합_보고서_v1.0_2025-09-30.md)
    - 현재 사용 중인 Attendance API 상세 명세

5. [Phase Final Compliance Checklist](../api/Phase_Final_Compliance_Checklist_and_Gap_Plan_2025-10-05.md)
    - Phase 0-3 API 통합 완료 현황 및 갭 분석

### 백엔드 API 문서 (필요 시 참조)

- OpenAPI 3.0 Spec (if available)
- Postman Collection
- API 명세서 (if available)

---

## ✅ 수락 기준 (Acceptance Criteria)

본 협조 요청이 성공적으로 완료되었다고 판단하는 기준:

1. **CRITICAL 항목 (3건) 모두 Week 1-2 내 해결** ✅
    - Attendance 엔드포인트 표준화 완료 및 FE 반영
    - storeService API 정상 동작 및 MasterMyPage 실데이터 표시
    - laborInfoService API 구현 및 실시간 노무 정보 표시

2. **HIGH 항목 (2건) Week 3-4 내 해결** ✅
    - Workplace API CRUD 완성 및 FE 통합
    - Salary/Payroll API 스키마 확정 및 PDF 생성 로직 구현

3. **정기 미팅 2회 이상 진행** ✅
    - 협의 결과가 `docs/api-decisions/`에 문서화됨

4. **통합 테스트 PASS** ✅
    - FE-BE 통합 테스트에서 모든 Critical/High 항목 정상 동작

---

## 🙏 맺음말 (Closing Remarks)

본 협조 요청서는 소담 프로젝트의 성공적인 완성을 위해 프론트엔드 팀에서 발견한 긴급 이슈들을 백엔드 팀과 공유하고, 함께 해결하기 위한 공식 문서입니다.

프론트엔드 팀은 백엔드 팀의 적극적인 협조와 신속한 대응을 요청드리며, 상호 긴밀한 소통을 통해 2개월 내 프로젝트 완성이라는 공동 목표를 달성하고자 합니다.

협조 요청 사항에 대한 질문이나 추가 논의가 필요한 경우, 언제든지 Slack `#fe-be-sync` 채널 또는 이메일로 연락 부탁드립니다.

**감사합니다.**

---

## 📞 연락처 (Contact Information)

- **프론트엔드 팀 리드**: Junie (AI Assistant)
- **Slack**: `#fe-be-sync`, `#sodam-dev`
- **Email**: (프로젝트 공식 이메일 주소)
- **GitHub**: (프로젝트 저장소 URL)

---

## 📅 변경 이력 (Change History)

| 날짜         | 버전  | 변경 내용                                                | 작성자             |
|------------|-----|------------------------------------------------------|-----------------|
| 2025-10-11 | 1.0 | 백엔드 팀 협조 요청서 초안 작성 (Critical 3건, High 2건, Medium 2건) | Junie (FE Team) |
