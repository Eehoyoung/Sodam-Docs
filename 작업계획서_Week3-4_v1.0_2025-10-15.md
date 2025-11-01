# 작업계획서: Week 3-4 스프린트

## 📋 문서 정보

- **작성일**: 2025-10-15
- **작성자**: Junie (React Native Engineer / QA Specialist)
- **문서 유형**: 작업 계획서 (Work Plan)
- **목표**: Week 3-4 핵심 기능 완성 및 품질 안정화
- **예상 소요 시간**: 2주 (80-100시간)
- **관련 문서**:
  - [Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md](Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md)
  - [작업완료_보고서_Week1-2_완료_v1.0_2025-10-13.md](작업완료_보고서_Week1-2_완료_v1.0_2025-10-13.md)
  - [FE_BE_협업_가이드_v1.0_2025-10-13.md](FE_BE_협업_가이드_v1.0_2025-10-13.md)
  - [BE_Implementation_Status_Report_v1.0_2025-10-11.md](BE_Implementation_Status_Report_v1.0_2025-10-11.md)
  - [ApiList.yaml](ApiList.yaml)

---

## 🎯 목적 및 범위

### 작업 배경
Week 1-2에서 Critical 문제 해결 및 기본 API 연동을 완료했습니다. Week 3-4는 남은 Quick Wins 작업을 완료하고, 중요도가 높은 기능(Salary 상세, Workplace 관리)을 구현하는 단계입니다.

### 작업 범위

#### Week 1-2 완료 항목 (2025-10-13 기준)
- ✅ [CRITICAL-001~003] StoreDetailScreen, ManagerMyPageScreen, PersonalUserScreen
- ✅ [HIGH-001~002] MasterMyPageScreen, 노무정보 서비스 연동
- ✅ [F-001] Attendance 엔드포인트 표준화
- ✅ [S-001~002] InfoListScreen, SalaryListScreen 네비게이션 통합

#### Week 3-4 목표 작업
1. **[T-001] CI 스캐너 통합** (Quick Win - 1일)
2. **[T-002] Jest 스위트 핵심 안정화** (Quick Win - 2-3일)
3. **[D-001] 문서 인덱스 정리** (Quick Win - 0.5일)
4. **[F-002] Salary 상세 화면 구현** (전략적 - 3일)
5. **[F-003] Workplace 관리 기능 구현** (전략적 - 4일)
6. **[S-003] Attendance UI 개선** (중기 - 2일)

### 제외 항목 (Week 5-6 이후)
- [D-002] 디자인 시스템 전체 적용 (5일)
- [T-003] 전체 Jest 스위트 안정화 (3일)
- [T-004] E2E 시나리오 자동화 (2일)

---

## 📊 작업 우선순위 및 의존성

### 우선순위 매트릭스

```
높은 임팩트 / 낮은 복잡도 (Quick Wins - Week 3 우선)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ [T-001] CI 스캐너 통합 (1일)
✓ [D-001] 문서 인덱스 정리 (0.5일)
✓ [T-002] Jest 스위트 핵심 안정화 (2-3일)

높은 임팩트 / 높은 복잡도 (전략적 - Week 3-4)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ [F-002] Salary 상세 화면 구현 (3일)
✓ [F-003] Workplace 관리 기능 구현 (4일)
✓ [S-003] Attendance UI 개선 (2일)
```

### 작업 의존성

```
Week 3 초반:
  [T-001] CI 스캐너 통합 (독립)
  [D-001] 문서 인덱스 정리 (독립)
  ↓
Week 3 중반:
  [T-002] Jest 안정화 (독립, 병렬 가능)
  [F-002] Salary 상세 화면 (SalaryListScreen 완료 전제)
  ↓
Week 3 후반 ~ Week 4:
  [F-003] Workplace 관리 (storeService 완료 전제)
  [S-003] Attendance UI 개선 (attendanceService 표준화 완료 전제)
```

---

## 📅 작업 일정 (Week 3-4)

### Week 3 Sprint (10월 16일 - 10월 22일)

**목표**: Quick Wins 완료 + Salary 상세 착수

| 일자 | 작업 | 담당 | 예상 시간 |
|------|------|------|----------|
| 10/16 (수) | [T-001] CI 스캐너 통합 | QA | 4-6시간 |
| 10/16 (수) | [D-001] 문서 인덱스 정리 | Doc Lead | 2-3시간 |
| 10/17-18 | [T-002] Jest 스위트 안정화 (Phase 1) | QA + RN | 12-16시간 |
| 10/19-20 | [F-002] Salary 상세 화면 (UI 구현) | RN + Designer | 12-16시간 |
| 10/21 | [T-002] Jest 안정화 (Phase 2 - 마무리) | QA | 4-6시간 |
| 10/22 | [F-002] Salary 상세 (차트/PDF 통합) | RN | 6-8시간 |

### Week 4 Sprint (10월 23일 - 10월 29일)

**목표**: Workplace 관리 완성 + Attendance UX 개선

| 일자 | 작업 | 담당 | 예상 시간 |
|------|------|------|----------|
| 10/23-24 | [F-003] Workplace 관리 (서비스 레이어) | RN + BE | 12-16시간 |
| 10/25-26 | [F-003] Workplace 관리 (UI 구현) | RN + Designer | 12-16시간 |
| 10/27-28 | [S-003] Attendance UI 개선 | RN + Designer | 12-16시간 |
| 10/29 | Week 3-4 통합 테스트 및 검증 | QA + RN | 6-8시간 |

---

## 🚀 작업 상세 계획

### 작업 1: [T-001] CI 스캐너 통합

**우선순위**: High (Quick Win)  
**예상 소요**: 4-6시간 (0.5-1일)  
**담당**: QA Specialist

#### 1.1 작업 배경

현재 `scripts\scan-qr-residue.ps1` 스캐너가 존재하나 CI 파이프라인에 통합되지 않아 수동으로만 실행 가능합니다. CI 통합을 통해 QR 관련 코드 재유입을 자동으로 방지해야 합니다.

#### 1.2 작업 상세

1. **GitHub Actions Workflow 파일 생성**:
   - 파일 경로: `.github/workflows/code-quality-check.yml`
   - 트리거: Pull Request, Push to main/develop
   - 스캐너 실행: `powershell -ExecutionPolicy Bypass -File .\scripts\scan-qr-residue.ps1 -FailOnMatch`

2. **CI 설정 검증**:
   - 테스트 PR 생성하여 스캐너 정상 작동 확인
   - 실패 케이스 테스트 (QR 키워드 추가 후 실패 확인)

3. **문서화**:
   - README.md에 CI 스캐너 설명 추가
   - 개발자 가이드에 예외 추가 방법 안내

#### 1.3 수락 기준 (AC)

- ✅ GitHub Actions workflow 파일 생성 및 활성화
- ✅ PR 생성 시 자동으로 스캐너 실행
- ✅ disallowed keywords 발견 시 CI 빌드 실패
- ✅ 허용된 경로(docs, scripts)는 검사 제외
- ✅ README 또는 CONTRIBUTING.md에 CI 스캐너 설명 추가

#### 1.4 산출물

- `.github/workflows/code-quality-check.yml`
- CI 통합 완료 보고서 (선택)

---

### 작업 2: [T-002] Jest 스위트 핵심 안정화

**우선순위**: High (Quick Win)  
**예상 소요**: 12-20시간 (2-3일)  
**담당**: QA Specialist + RN Engineer

#### 2.1 작업 배경

현재 전체 테스트 스위트(`npm test`) 실행 시 다수 실패하나, curated tests는 통과합니다. 핵심 도메인 테스트를 우선 안정화하여 CI 통합 기반을 마련합니다.

#### 2.2 작업 상세

**Phase 1: 테스트 현황 분석** (4시간)
1. `npm test` 실행하여 실패 테스트 목록 수집
2. 실패 원인 분류:
   - Mock 설정 문제
   - 비동기 처리 문제
   - 타입 불일치
   - 의존성 문제
3. 우선순위 설정: Attendance > Store > Wage > Auth

**Phase 2: 핵심 테스트 수정** (12-16시간)
1. **Attendance 도메인**:
   - `attendanceService.test.ts`: Mock API 응답 보완
   - `nfcAttendanceService.test.ts`: NFC Manager mock 수정
   
2. **Store 도메인**:
   - `storeService.test.ts`: Repository mock 보완
   
3. **Wage/Payroll 도메인**:
   - `payrollService.test.ts`: 급여 계산 로직 테스트 수정
   
4. **Auth 도메인**:
   - `authService.test.ts`: JWT 토큰 mock 보완

#### 2.3 수락 기준 (AC)

- ✅ 핵심 도메인 테스트 100% 통과:
  - `npm test -- attendanceService`
  - `npm test -- storeService`
  - `npm test -- payrollService`
  - `npm test -- authService`
- ✅ 테스트 커버리지 유지 또는 향상 (≥40%)
- ✅ Mock 설정 문서화 (jest.setup.js 주석)

#### 2.4 산출물

- 수정된 테스트 파일들
- `docs/testing/Test_Stabilization_Report_Week3.md`

---

### 작업 3: [D-001] 문서 인덱스 정리

**우선순위**: Medium (Quick Win)  
**예상 소요**: 2-3시간 (0.5일)  
**담당**: Documentation Lead

#### 3.1 작업 배경

현재 `docs/` 디렉토리에 다수의 보고서와 가이드가 존재하나 중앙 인덱스가 없어 접근성이 떨어집니다.

#### 3.2 작업 상세

1. **docs/README.md 생성**:
   - 카테고리별 문서 분류:
     - API 통합 보고서
     - 기능별 구현 가이드
     - 테스트 보고서
     - 문제 해결 가이드
   - 각 문서에 대한 간단한 설명 추가

2. **기존 문서 정리**:
   - 중복 문서 식별 및 통합
   - 오래된 문서는 `docs/archive/` 이동
   - 최신 문서에 날짜 표기 확인

3. **프로젝트 루트 README 업데이트**:
   - `docs/README.md` 링크 추가
   - 신규 개발자 온보딩 섹션 추가

#### 3.3 수락 기준 (AC)

- ✅ `docs/README.md` 생성 및 전체 문서 인덱싱
- ✅ 카테고리별 분류 (최소 5개 카테고리)
- ✅ 각 문서에 1-2줄 설명 추가
- ✅ 프로젝트 루트 README에 문서 포털 링크

#### 3.4 산출물

- `docs/README.md`
- 업데이트된 프로젝트 루트 `README.md`

---

### 작업 4: [F-002] Salary 상세 화면 구현

**우선순위**: High (전략적)  
**예상 소요**: 18-24시간 (3일)  
**담당**: RN Engineer + Designer

#### 4.1 작업 배경

`SalaryListScreen`은 이미 통합되었으나 상세 화면이 없어 급여 명세서 확인 및 차트 시각화가 불가능합니다.

#### 4.2 작업 상세

**Day 1: SalaryDetailScreen UI 구현** (6-8시간)
1. **화면 구조 설계**:
   - 급여 요약 카드 (총 급여, 근무 시간, 시급)
   - 상세 내역 섹션 (기본급, 수당, 공제)
   - 액션 버튼 (PDF 다운로드, 공유)

2. **컴포넌트 생성**:
   - `src/features/salary/screens/SalaryDetailScreen.tsx`
   - `src/features/salary/components/SalaryDetailCard.tsx`
   - `src/features/salary/components/PayrollBreakdown.tsx`

**Day 2: API 연동 및 데이터 처리** (6-8시간)
1. **API 통합**:
   - `payrollService.getPayrollDetails(payrollId)` 연동
   - `payrollService.downloadPayrollPDF(payrollId)` 구현
   
2. **상태 관리**:
   - 로딩, 에러, 성공 상태 처리
   - PDF 다운로드 진행 상태 표시

**Day 3: 차트 통합 및 테스트** (6-8시간)
1. **월별 급여 차트**:
   - `react-native-chart-kit` 사용
   - 최근 6개월 급여 추이 시각화
   
2. **네비게이션 연결**:
   - `SalaryListScreen` → `SalaryDetailScreen` 전환
   - 파라미터 전달 (payrollId)

3. **테스트**:
   - 단위 테스트 작성
   - 수동 UI 테스트

#### 4.3 필요 API (BE 완료 상태)

- ✅ `GET /api/payroll/{payrollId}/details` - 급여 상세 조회
- ✅ `GET /api/payroll/{payrollId}/pdf` - PDF 다운로드
- ✅ `GET /api/payroll/employee/{employeeId}` - 직원 급여 목록 (차트용)

#### 4.4 수락 기준 (AC)

- ✅ SalaryDetailScreen 화면 구현 완료
- ✅ API 연동하여 실제 급여 데이터 표시
- ✅ PDF 다운로드 기능 동작
- ✅ 월별 급여 차트 시각화
- ✅ SalaryList → SalaryDetail 네비게이션 정상 작동
- ✅ 로딩/에러 처리 구현

#### 4.5 산출물

- `src/features/salary/screens/SalaryDetailScreen.tsx`
- `src/features/salary/components/` (신규 컴포넌트들)
- 테스트 파일
- 구현 완료 보고서

---

### 작업 5: [F-003] Workplace 관리 기능 구현

**우선순위**: High (전략적)  
**예상 소요**: 24-32시간 (4일)  
**담당**: RN Engineer + Backend Engineer + Designer

#### 5.1 작업 배경

현재 매장 관리 기능이 부분적으로만 구현되어 있습니다. Master 사용자가 여러 매장을 등록하고 관리할 수 있는 완전한 Workplace 관리 기능이 필요합니다.

#### 5.2 작업 상세

**Day 1-2: WorkplaceService 및 WorkplaceList 구현** (12-16시간)

1. **workplaceService 확장**:
   - `createWorkplace(data)`: 매장 생성
   - `updateWorkplace(id, data)`: 매장 정보 수정
   - `deleteWorkplace(id)`: 매장 삭제
   - `getWorkplaceEmployees(id)`: 직원 목록 조회
   - `addEmployee(workplaceId, employeeData)`: 직원 추가

2. **WorkplaceListScreen 구현**:
   - `src/features/workplace/screens/WorkplaceListScreen.tsx`
   - Master 사용자의 전체 매장 목록 표시
   - 매장 추가 버튼 (+ FAB)
   - 각 매장 카드: 이름, 주소, 직원 수, 상태
   - 매장 클릭 → WorkplaceDetail 이동

3. **WorkplaceCreateScreen 구현**:
   - `src/features/workplace/screens/WorkplaceCreateScreen.tsx`
   - 매장 정보 입력 폼 (이름, 주소, 전화번호, 영업시간)
   - 위치 선택 (지도 또는 주소 검색)
   - 유효성 검증

**Day 3-4: WorkplaceDetail 및 직원 관리** (12-16시간)

1. **WorkplaceDetailScreen 구현**:
   - `src/features/workplace/screens/WorkplaceDetailScreen.tsx`
   - 매장 정보 섹션 (수정 가능)
   - 직원 목록 섹션
   - 출퇴근 통계 요약
   - 매장 설정 (기본 시급, 근무 규칙)

2. **직원 관리 기능**:
   - 직원 초대 (이메일/전화번호)
   - 직원 역할 설정 (EMPLOYEE/MANAGER)
   - 직원 삭제/비활성화

3. **네비게이션 통합**:
   - MasterMyPageScreen → WorkplaceList 진입점 추가
   - HomeNavigator에 Workplace 관련 라우트 추가
   - 타입 정의 업데이트

#### 5.3 필요 API (BE 완료 상태)

- ✅ `GET /api/stores/master/{userId}` - 매장 목록 조회
- ✅ `POST /api/stores` - 매장 생성
- ✅ `PUT /api/stores/{storeId}` - 매장 수정
- ✅ `DELETE /api/stores/{storeId}` - 매장 삭제
- ✅ `GET /api/stores/{storeId}/employees` - 직원 목록
- ✅ `POST /api/stores/{storeId}/employees` - 직원 추가
- ✅ `PUT /api/stores/{storeId}/location` - 매장 위치 설정

#### 5.4 수락 기준 (AC)

- ✅ WorkplaceListScreen 구현 및 Master 사용자 매장 목록 표시
- ✅ WorkplaceCreateScreen으로 신규 매장 생성 가능
- ✅ WorkplaceDetailScreen에서 매장 정보 및 직원 관리
- ✅ 직원 추가/삭제 기능 동작
- ✅ MasterMyPageScreen에서 Workplace 관리 진입점 추가
- ✅ 모든 CRUD 작업 API 연동 완료
- ✅ 로딩/에러 처리 구현

#### 5.5 산출물

- `src/features/workplace/screens/WorkplaceListScreen.tsx`
- `src/features/workplace/screens/WorkplaceCreateScreen.tsx`
- `src/features/workplace/screens/WorkplaceDetailScreen.tsx`
- `src/features/workplace/services/workplaceService.ts` (확장)
- `src/features/workplace/components/` (신규 컴포넌트들)
- 테스트 파일
- 구현 완료 보고서

---

### 작업 6: [S-003] Attendance UI 개선

**우선순위**: Medium  
**예상 소요**: 12-16시간 (2일)  
**담당**: RN Engineer + Designer

#### 6.1 작업 배경

현재 출퇴근 기능은 동작하나 사용자 피드백이 부족하고 UI가 직관적이지 않습니다. NFC 인식 및 위치 인증 프로세스의 UX를 개선합니다.

#### 6.2 작업 상세

**Day 1: NFC 인식 UI 개선** (6-8시간)

1. **NFC 스캔 프로세스 시각화**:
   - 스캔 대기 애니메이션 추가
   - 태그 인식 진행 상태 표시
   - 성공/실패 피드백 강화 (햅틱, 사운드, 애니메이션)

2. **AttendanceScreen 리팩토링**:
   - `src/features/attendance/screens/AttendanceScreen.tsx`
   - 출근/퇴근 버튼 상태에 따라 다른 색상/아이콘
   - 현재 근무 상태 명확히 표시 (출근 전/근무 중/퇴근 완료)

3. **에러 처리 개선**:
   - NFC 미지원 디바이스 안내
   - 태그 인식 실패 시 재시도 안내
   - 네트워크 오류 시 명확한 안내

**Day 2: 위치 인증 피드백 및 히스토리 개선** (6-8시간)

1. **위치 인증 UI**:
   - 현재 위치 표시 (주소 또는 좌표)
   - 매장 위치와의 거리 표시
   - 인증 범위 시각화 (지도 위 반경)

2. **출퇴근 히스토리 개선**:
   - `AttendanceHistoryScreen` 생성 또는 AttendanceScreen에 통합
   - 최근 7일 출퇴근 기록 카드
   - 근무 시간 통계 (일별, 주별)

3. **디자인 시스템 적용**:
   - COLORS, 그라디언트, spacing 가이드 준수
   - 공통 컴포넌트 재사용 (Button, Card 등)

#### 6.3 수락 기준 (AC)

- ✅ NFC 스캔 프로세스에 애니메이션 및 피드백 추가
- ✅ 출근/퇴근 버튼 상태별 시각적 구분 명확
- ✅ 위치 인증 시 현재 위치 및 거리 표시
- ✅ 에러 메시지 개선 (사용자 친화적)
- ✅ 최근 출퇴근 기록 표시 (히스토리)
- ✅ 디자인 시스템 가이드 준수

#### 6.4 산출물

- 수정된 `AttendanceScreen.tsx`
- 신규 `AttendanceHistoryScreen.tsx` (선택)
- 애니메이션 컴포넌트
- UX 개선 완료 보고서

---

## 📊 루브릭 (작업 품질 기준)

### 1. 정확성 (Accuracy) - 필수

- ✅ BE API 스펙 100% 준수 (엔드포인트 경로, 요청/응답 구조)
- ✅ TypeScript 컴파일 에러 0건
- ✅ 기존 기능 회귀 없음 (변경된 화면 정상 동작)
- ✅ API 호출 성공/실패 케이스 모두 처리

### 2. 논리적 일관성 (Logical Consistency) - 필수

- ✅ 인증 헤더 모든 API 호출에 적용
- ✅ 에러 처리 패턴 통일 (401 → 토큰 갱신, 400/500 → 사용자 안내)
- ✅ 데이터 흐름 명확성 (Service → Hook → Screen)
- ✅ 네비게이션 플로우 일관성 유지

### 3. 완전성 (Completeness) - 필수

- ✅ 각 작업의 수락기준(AC) 100% 충족
- ✅ 관련 타입 정의 업데이트 (TypeScript interfaces)
- ✅ 주석 및 JSDoc 추가 (주요 함수/컴포넌트)
- ✅ 로딩/에러 상태 처리 구현

### 4. 테스트 가능성 (Testability) - 권장

- ✅ 변경된 서비스/훅 단위 테스트 가능
- ✅ Mock 데이터 제거 후 실제 API 호출 검증
- ✅ 핵심 도메인 테스트 유지 (Attendance, Store, Wage)
- ✅ 테스트 커버리지 유지 또는 향상 (≥40%)

### 5. 사용자 경험 (User Experience) - 권장

- ✅ 로딩 인디케이터 적절히 표시
- ✅ 에러 메시지 사용자 친화적
- ✅ 액션에 대한 피드백 명확 (성공/실패)
- ✅ 디자인 시스템 가이드 준수

### 6. 문서화 (Documentation) - 필수

- ✅ 작업 완료 보고서 작성
- ✅ Master_Task_Log.md 업데이트
- ✅ 코드 내 주석으로 변경 이유 명시
- ✅ README 또는 관련 문서 업데이트

---

## ⚠️ 리스크 및 이슈 관리

### 식별된 리스크

#### 1. 테스트 안정화 지연 (중간 리스크)

**문제**: Jest 스위트 안정화가 예상보다 복잡할 수 있음 (Mock 설정 문제, 비동기 타이밍 이슈)

**완화 방안**:
- Phase 1에서 실패 원인 명확히 분류 후 작업
- 핵심 도메인만 우선 안정화 (Attendance, Store)
- 막히면 즉시 에스컬레이션 (RN Engineer 지원)

**대체 계획**: 일부 테스트 스킵 처리하고 Week 5로 이월

---

#### 2. Workplace 관리 기능 복잡도 (높은 리스크)

**문제**: 직원 관리 로직이 복잡하고 권한 체계가 까다로울 수 있음

**완화 방안**:
- BE 팀과 사전 API 스펙 재확인 (역할 기반 권한)
- UI 먼저 구현하고 API 연동은 단계적으로
- 최소 기능(매장 CRUD)만 Week 4에 완료, 고급 기능은 Week 5로 이월

**대체 계획**: 직원 관리 기능을 Phase 2로 분리하고 매장 관리만 완료

---

#### 3. PDF 다운로드 기능 (낮은 리스크)

**문제**: PDF 생성 및 다운로드가 모바일 환경에서 불안정할 수 있음

**완화 방안**:
- BE 팀의 PDF API 먼저 테스트 (Postman/cURL)
- React Native File System 라이브러리 사전 검증
- PDF 뷰어는 외부 앱 연동으로 대체 가능

**대체 계획**: PDF 다운로드 대신 웹뷰로 표시

---

#### 4. CI 스캐너 GitHub Actions 설정 (낮은 리스크)

**문제**: GitHub Actions 경험 부족으로 설정이 지연될 수 있음

**완화 방안**:
- 기존 GitHub Actions 템플릿 활용
- 단순한 workflow 구성 (PowerShell 스크립트 실행만)
- 테스트 저장소에서 먼저 검증

**대체 계획**: GitHub Actions 대신 Git pre-commit hook 사용

---

### 이슈 에스컬레이션 프로세스

1. **작업 블로커 발생 시**: 즉시 팀 리더에게 보고 (Slack/이메일)
2. **리스크 현실화**: 대체 계획 활성화 여부 논의
3. **일정 지연 예상**: 우선순위 재조정 (Quick Wins 우선 완료)

---

## ✅ 검증 절차 (Verification Procedures)

### Phase 1: 단위 검증 (작업별)

각 작업 완료 시 다음 항목 확인:

1. **코드 품질**:
   - TypeScript 컴파일 에러 0건
   - ESLint 경고 해결
   - 코드 리뷰 통과

2. **기능 검증**:
   - 수락 기준(AC) 항목별 체크리스트 작성
   - 주요 시나리오 수동 테스트
   - 에러 케이스 확인

3. **테스트**:
   - 단위 테스트 작성 및 통과
   - 핵심 도메인 테스트 유지
   - 커버리지 확인

---

### Phase 2: 통합 검증 (Week 3 종료 시)

1. **빌드 검증**:
   ```powershell
   npm run android
   npm run ios
   ```
   - Android/iOS 빌드 성공 확인
   - 런타임 에러 없음

2. **네비게이션 검증**:
   - 모든 추가된 화면 진입 가능
   - 파라미터 전달 정상 작동
   - 뒤로가기 동작 확인

3. **API 통합 검증**:
   - 모든 신규 API 호출 성공 확인 (실제 서버 연동)
   - 인증 헤더 정상 적용
   - 에러 응답 핸들링 확인

---

### Phase 3: 회귀 테스트 (Week 4 종료 시)

1. **기존 기능 회귀 확인**:
   - Week 1-2에서 완료한 기능들 재테스트
   - PersonalUserScreen, InfoListScreen, SalaryListScreen 정상 동작
   - Attendance 엔드포인트 표준화 유지

2. **전체 테스트 스위트 실행**:
   ```powershell
   npm test
   ```
   - 핵심 도메인 테스트 100% 통과 확인
   - 전체 커버리지 ≥40% 유지

3. **CI 검증** (T-001 완료 후):
   ```powershell
   # 로컬 스캐너 실행
   powershell -ExecutionPolicy Bypass -File .\scripts\scan-qr-residue.ps1 -FailOnMatch
   ```
   - 테스트 PR 생성하여 CI 자동 실행 확인

---

## 📈 성공 지표 (Success Metrics)

### Week 3 종료 시 (10/22)

- ✅ Quick Wins 3개 완료 (T-001, T-002, D-001)
- ✅ Salary 상세 화면 구현 80% 이상 진행
- ✅ 핵심 테스트 스위트 안정화 (Attendance, Store 도메인)
- ✅ CI 스캐너 통합 완료

### Week 4 종료 시 (10/29)

- ✅ 전체 6개 작업 완료
- ✅ Workplace 관리 기능 CRUD 구현
- ✅ Attendance UI 개선 적용
- ✅ 회귀 테스트 통과 (기존 기능 정상 동작)
- ✅ 전체 진척도: Week 1-2 (8/11) + Week 3-4 (6/6) = 14/17 (82%)

### 품질 지표

- TypeScript 컴파일 에러: 0건
- ESLint 경고: 최소화 (<10건)
- 테스트 커버리지: ≥40%
- 핵심 도메인 테스트 통과율: 100%
- API 통합 성공률: ≥95%

---

## 🎯 다음 단계 권장사항 (Week 5-6)

Week 3-4 완료 후 다음 작업을 권장합니다:

### Week 5 우선순위

1. **[D-002] 디자인 시스템 전체 적용** (5일)
   - 모든 화면에 COLORS, 그라디언트, spacing 적용
   - 공통 컴포넌트 라이브러리 확장
   - 디자인 일관성 검증

2. **[T-003] 전체 Jest 스위트 안정화** (3일)
   - Week 3에서 제외된 테스트 수정
   - 전체 테스트 스위트 100% 통과
   - CI 테스트 자동화 완성

3. **Workplace 고급 기능** (3일)
   - 직원 초대 시스템 (이메일/SMS)
   - 매장별 출퇴근 통계 대시보드
   - 급여 정책 설정 UI

### Week 6 우선순위

1. **[T-004] E2E 시나리오 자동화** (2일)
   - Detox 또는 Appium 설정
   - 핵심 플로우 자동화 (Login → Attendance → Logout)

2. **성능 최적화** (2일)
   - 번들 크기 최적화
   - 이미지 최적화
   - 메모리 누수 확인

3. **문서 완성** (1일)
   - 개발자 온보딩 가이드 작성
   - API 통합 사례 문서화
   - 트러블슈팅 가이드 업데이트

---

## 📝 작업 완료 보고서 작성 가이드

Week 3-4 작업 완료 시 다음 항목을 포함한 보고서를 작성합니다:

### 필수 포함 항목

1. **작업 완료 내역**:
   - 각 작업별 완료 상태 (완료/부분완료/미완료)
   - 변경된 파일 목록
   - 수락 기준(AC) 충족 여부

2. **통계 요약**:
   - 작업 완료율 (6/6 또는 실제 비율)
   - 테스트 통과율
   - 코드 변경 통계 (추가/수정/삭제 라인 수)

3. **이슈 및 해결 방법**:
   - 발생한 주요 이슈
   - 해결 방법 또는 우회 방안
   - 미해결 이슈 및 추후 계획

4. **다음 단계**:
   - Week 5-6 작업 권장사항
   - 기술 부채 식별
   - 개선 제안

### 보고서 위치

- `common-docs/작업완료_보고서_Week3-4_v1.0_YYYY-MM-DD.md`

---

## 📚 참고 문서

### API 관련
- [ApiList.yaml](ApiList.yaml) - OpenAPI 3.0 전체 명세
- [FE_BE_협업_가이드_v1.0_2025-10-13.md](FE_BE_협업_가이드_v1.0_2025-10-13.md)
- [BE_Implementation_Status_Report_v1.0_2025-10-11.md](BE_Implementation_Status_Report_v1.0_2025-10-11.md)

### 프로젝트 계획
- [Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md](Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md)
- [작업완료_보고서_Week1-2_완료_v1.0_2025-10-13.md](작업완료_보고서_Week1-2_완료_v1.0_2025-10-13.md)

### 고급 가이드
- [report_251015/API_고급_개발자_가이드.md](report_251015/API_고급_개발자_가이드.md)
- [report_251015/문서_통합_인덱스.md](report_251015/문서_통합_인덱스.md)

### 개발 가이드라인
- [guidelines.md](../../docs/guidelines.md) - 프로젝트 개발 가이드라인
- [프론트엔드_가이드라인_v3.2.md](../../docs/guidelines/프론트엔드_가이드라인_v3.2.md)

---

## 📞 연락처 및 지원

### 작업 관련 문의
- **RN Lead (Junie)**: React Native 구현 관련
- **Backend Team**: API 통합 이슈
- **QA Specialist**: 테스트 및 품질 관련
- **Designer**: UI/UX 디자인 관련

### 이슈 보고
- GitHub Issues 사용
- 긴급 이슈: Slack #sodam-dev 채널

---

**문서 작성**: Junie (React Native Engineer / QA Specialist)  
**최종 검토**: 2025-10-15  
**다음 업데이트 예정**: Week 3-4 완료 후 (2025-10-29)

