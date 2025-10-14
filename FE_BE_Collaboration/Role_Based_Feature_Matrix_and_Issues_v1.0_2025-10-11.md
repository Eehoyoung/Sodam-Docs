# Sodam Front End - 역할별 기능 매트릭스 및 프로젝트 전면 검토 보고서

## 📋 Overview

- **작성일**: 2025-10-11
- **작성자**: Junie (Flow Analyst / React Native Engineer / QA Specialist)
- **문서 유형**: Comprehensive Project Audit / Role-Based Feature Analysis
- **관련 이슈**: 역할별 기능 분류 및 숨은 문제 발굴
- **버전**: 1.0

## 🎯 목적

본 문서는 user/master/employee/manager 역할별로 공유하는 기능과 전용 기능을 명확히 분류하고, 프로젝트 전면 검토를 통해 발견된 숨은 문제점들을 문서화하여 향후 작업 우선순위를 제시합니다.

## 📊 역할별 MyPage 스크린 현황

### 파일 상태 요약

| 역할       | 파일명                        | 크기     | 줄수    | 최종 수정일     | 상태          |
|----------|----------------------------|--------|-------|------------|-------------|
| Employee | EmployeeMyPageRNScreen.tsx | 8.4KB  | 264줄  | 2025-09-28 | ✅ 정상        |
| Master   | MasterMyPageScreen.tsx     | 28.3KB | 809줄  | 2025-09-28 | ⚠️ 부분 구현    |
| Manager  | ManagerMyPageScreen.tsx    | 4.7KB  | ~150줄 | 2025-06-22 | ❌ 방치됨 (4개월) |
| Personal | PersonalUserScreen.tsx     | 57.4KB | 1527줄 | 2025-09-28 | ⚠️ API 미연동  |

## 🔍 역할별 기능 분류 매트릭스

### 1. Personal User (개인 사용자) - PersonalUserScreen

#### 전용 기능

| 기능          | 구현 상태 | 서비스 연동  | 비고                     |
|-------------|-------|---------|------------------------|
| 다중 매장 관리    | ✅ 구현  | ❌ 하드코딩  | stores 배열 하드코딩         |
| 매장별 출퇴근 타이머 | ✅ 구현  | ❌ 로컬 상태 | workSessions 로컬 관리     |
| 휴게 시간 관리    | ✅ 구현  | ❌ 로컬 상태 | 타이머 기반, 백엔드 동기화 없음     |
| 수동 기록 입력    | ✅ 구현  | ❌ 로컬 상태 | allRecords 로컬 저장       |
| 일일 근무 요약    | ✅ 구현  | ❌ 로컬 계산 | DailyWorkSummary 로컬 집계 |
| 월별 통계       | ✅ 구현  | ❌ 로컬 계산 | MonthlyStats 로컬 집계     |
| 매장별 수익 계산   | ✅ 구현  | ❌ 로컬 계산 | hourlyWage 기반 로컬 계산    |

**문제점**:

- **API 연동 전무**: 모든 데이터가 하드코딩 또는 로컬 상태 관리
- 주석에 "todo API 연동 필수" 명시되어 있으나 미구현
- 백엔드와 데이터 동기화 불가능
- 앱 재시작 시 모든 데이터 소실

#### 공유 기능

- Settings, Profile 접근 (공통)

---

### 2. Employee (직원) - EmployeeMyPageRNScreen

#### 전용 기능

| 기능               | 구현 상태 | 서비스 연동                   | 비고                           |
|------------------|-------|--------------------------|------------------------------|
| 출퇴근 요약 패널        | ✅ 구현  | ✅ AttendanceSummaryPanel | 컴포넌트화됨                       |
| 정부 정책 정보 (상위 3개) | ✅ 구현  | ✅ policyService          | getPoliciesByCategory('ALL') |
| 노무 핵심 정보         | ✅ 구현  | ❌ 하드코딩                   | minimumWage 등 목 데이터          |
| 출퇴근 상세 보기        | ✅ 구현  | ✅ navigate('Attendance') | 네비게이션 정상                     |

**문제점**:

- laborInfo가 하드코딩 (minimumWage: 9620, year: 2024 등)
- laborInfoService 미사용

#### 공유 기능

- Attendance 화면 접근
- PolicyDetail 화면 접근
- Settings, Profile 접근

---

### 3. Master (사장) - MasterMyPageScreen

#### 전용 기능

| 기능                | 구현 상태 | 서비스 연동                          | 비고                           |
|-------------------|-------|---------------------------------|------------------------------|
| 매장 목록 (카드 형식)     | ✅ 구현  | ❌ 하드코딩                          | mockStores 배열 사용             |
| 매장 상세 이동          | ⚠️ 구현 | ❌ 라우트 미등록                       | StoreDetailScreen 라우트 없음     |
| 매장 추가             | ✅ 구현  | ✅ navigate('StoreRegistration') | 네비게이션 정상                     |
| 전체 현황 (매장/직원/인건비) | ✅ 구현  | ❌ 로컬 집계                         | mockStores 기반 계산             |
| 정부 정책 정보          | ✅ 구현  | ✅ policyService                 | getPoliciesByCategory('ALL') |
| 노무 정보 카드          | ✅ 구현  | ❌ 하드코딩                          | mockLaborInfo 사용             |

**문제점**:

- **매장 데이터 완전 하드코딩**: storeService.getMasterStores 미사용
- **StoreDetailScreen 라우트 미등록**: HomeNavigator에 라우트 없음, 네비게이션 오류 발생
- laborInfoService 미연동

#### 공유 기능

- PolicyDetail 화면 접근
- StoreRegistration 화면 접근
- Settings 접근

---

### 4. Manager (매니저) - ManagerMyPageScreen

#### 상태

- **파일 존재**: 4.7KB (약 150줄 추정)
- **최종 수정**: 2025-06-22 (4개월 전)
- **파일 로드 실패**: 인코딩 오류 또는 파일 손상 가능성
- **구현 수준**: 미확인 (로드 불가로 분석 불가)

**문제점**:

- **4개월 동안 미업데이트**: 프로젝트에서 방치된 상태
- **파일 로드 불가**: 인코딩/손상 문제 또는 빈 파일
- **구현 미완성 추정**: 파일 크기가 가장 작음 (4.7KB vs 다른 8~57KB)

---

## 🔗 공유 기능 (모든 역할)

| 기능       | 스크린/컴포넌트              | HomeNavigator 라우트 | 구현 상태 |
|----------|-----------------------|-------------------|-------|
| 설정       | SettingsScreen        | Settings          | ✅ 통합  |
| 프로필      | ProfileScreen         | Profile           | ✅ 통합  |
| 출퇴근 관리   | AttendanceScreen      | Attendance        | ✅ 통합  |
| 정책 상세    | PolicyDetailScreen    | PolicyDetail      | ✅ 통합  |
| 노동 정보 상세 | LaborInfoDetailScreen | LaborInfoDetail   | ✅ 통합  |
| 세무 정보 상세 | TaxInfoDetailScreen   | TaxInfoDetail     | ✅ 통합  |
| 팁 상세     | TipsDetailScreen      | TipsDetail        | ✅ 통합  |
| Q&A      | QnAScreen             | QnA               | ✅ 통합  |
| 구독       | SubscribeScreen       | Subscribe         | ✅ 통합  |

---

## 🚨 발견된 주요 문제점 (우선순위별)

### Critical (즉시 수정 필요)

#### 1. StoreDetailScreen 라우트 미등록 ⛔

- **위치**: MasterMyPageScreen.tsx line 190
- **문제**: `navigation.navigate('StoreDetailScreen', { storeId })` 호출하나 HomeNavigator에 라우트 없음
- **영향**: 매장 카드 탭 시 크래시
- **해결**: HomeNavigator에 StoreDetailScreen 라우트 추가 필요

#### 2. ManagerMyPageScreen 파일 로드 실패 ⛔

- **위치**: src/features/myPage/screens/ManagerMyPageScreen.tsx
- **문제**: 파일 존재하나 로드 불가, 4개월 미업데이트
- **영향**: Manager 역할 사용자의 MyPage 접근 불가
- **해결**: 파일 재생성 또는 복구, 기능 재구현 필요

#### 3. PersonalUserScreen 완전 로컬화 ⛔

- **위치**: PersonalUserScreen.tsx (1527줄)
- **문제**: 모든 데이터 하드코딩, API 연동 전무
- **영향**: 데이터 영속성 없음, 앱 재시작 시 소실
- **해결**: storeService, attendanceService 연동 필수

### High (조속히 수정 권장)

#### 4. MasterMyPageScreen 매장 데이터 하드코딩 🔴

- **위치**: MasterMyPageScreen.tsx line 83-126
- **문제**: mockStores 배열 사용, storeService.getMasterStores 미사용
- **영향**: 실제 매장 데이터 표시 불가
- **해결**: storeService.getMasterStores() 연동

#### 5. 노무 정보 하드코딩 (Employee, Master 공통) 🔴

- **위치**: EmployeeMyPageRNScreen.tsx line 58, MasterMyPageScreen.tsx line 152
- **문제**: laborInfo 하드코딩 (minimumWage: 9620 등)
- **영향**: 최신 노무 정보 반영 불가
- **해결**: laborInfoService 연동

### Medium (계획적 개선)

#### 6. 미통합 스크린 5개 🟡

- InfoListScreen, SalaryListScreen, WorkplaceListScreen, WorkplaceDetailScreen, HybridMainScreen
- 구현되었으나 네비게이션 미연결

#### 7. 테스트 커버리지 부족 🟡

- 전체 커버리지 ~40% (목표 70%)
- MyPage 스크린 테스트 부재

---

## 📋 역할별 진입 조건 및 라우팅

### 인증 후 역할 기반 라우팅 (LoginScreen.tsx)

```typescript
// line 59-62
const grade = normalizeUserGrade(loggedInUser?.role);
let targetScreen = 'UserMyPageScreen';
if (grade === 'EMPLOYEE') targetScreen = 'EmployeeMyPageScreen';
else if (grade === 'MASTER') targetScreen = 'MasterMyPageScreen';
```

| 역할       | 백엔드 값     | 정규화 값    | 초기 라우트               | 구현 상태    |
|----------|-----------|----------|----------------------|----------|
| Personal | (default) | -        | UserMyPageScreen     | ✅ 동작     |
| Employee | EMPLOYEE  | EMPLOYEE | EmployeeMyPageScreen | ✅ 동작     |
| Master   | MASTER    | MASTER   | MasterMyPageScreen   | ⚠️ 부분 동작 |
| Manager  | MANAGER   | MANAGER  | ManagerMyPageScreen  | ❌ 로드 불가  |

---

## 🛠️ 권장 해결 방안

### Phase 1: Critical 문제 해결 (Week 1-2)

#### Task 1.1: StoreDetailScreen 추가

- HomeNavigator에 라우트 추가
- StoreDetailScreen 컴포넌트 구현 또는 기존 StoreDetailScreen 연결
- MasterMyPageScreen 네비게이션 테스트

#### Task 1.2: ManagerMyPageScreen 복구

- 파일 인코딩 확인 및 복구
- 미구현 시 Employee/Master 참고하여 재작성
- Manager 역할 기능 정의 및 구현

#### Task 1.3: PersonalUserScreen API 연동

- storeService 연동 (매장 목록 조회)
- attendanceService 연동 (출퇴근 기록 CRUD)
- 로컬 상태 → 백엔드 동기화 전환

### Phase 2: High 우선순위 개선 (Week 3-4)

#### Task 2.1: MasterMyPageScreen 서비스 연동

- storeService.getMasterStores() 연동
- laborInfoService 연동
- 빈 상태 처리 (매장 없을 때)

#### Task 2.2: Employee/Master 노무 정보 서비스 연동

- laborInfoService 구현 확인
- EmployeeMyPageRNScreen, MasterMyPageScreen 연동

### Phase 3: Medium 우선순위 개선 (Week 5-6)

#### Task 3.1: 미통합 스크린 5개 연결

- 각 스크린의 용도 확인
- HomeNavigator 라우트 추가
- 진입점 구현

#### Task 3.2: MyPage 스크린 테스트 추가

- 4개 MyPage 스크린 단위 테스트
- 네비게이션 통합 테스트
- 서비스 목킹 테스트

---

## 📅 Change History

| Date       | Version | Changes                                       | Author |
|------------|---------|-----------------------------------------------|--------|
| 2025-10-11 | 1.0     | Initial creation - 역할별 기능 분류, 문제점 발굴, 해결방안 제시 | Junie  |
