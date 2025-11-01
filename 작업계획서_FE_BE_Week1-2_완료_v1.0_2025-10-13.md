# 작업계획서: FE-BE Week 1-2 나머지 작업 완료

## 📋 문서 정보

- **작성일**: 2025-10-13
- **작성자**: Junie (React Native Engineer / QA Specialist)
- **문서 유형**: 작업 계획서 (Work Plan)
- **목표**: Week 1-2 나머지 작업 완수 (Attendance 표준화, PersonalUserScreen 연동, InfoListScreen, SalaryListScreen)
- **예상 소요 시간**: 6-8시간
- **관련 문서**:
  - [Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md](Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md)
  - [FE_BE_협업_가이드_v1.0_2025-10-13.md](FE_BE_협업_가이드_v1.0_2025-10-13.md)

---

## 🎯 목적 및 범위

Backend 팀이 제공한 API 스펙(FE_BE_협업_가이드_v1.0_2025-10-13.md)에 따라, Week 1-2 작업 계획의 나머지 항목을 완수합니다.

### 작업 범위
- **[F-001]** Attendance 엔드포인트 표준화 (Week 1)
- **[CRITICAL-003]** PersonalUserScreen API 연동 (Week 2)
- **[S-001]** InfoListScreen 네비게이션 통합 (Week 2)
- **[S-002]** SalaryListScreen 기본 연결 (Week 2)

### 제외 항목 (추후 작업)
- [T-001] CI 스캐너 통합 (CI/CD 작업)
- [T-002] Jest 스위트 핵심 안정화 (별도 QA 세션)

---

## 📊 루브릭 (작업 품질 기준)

### 1. 정확성 (Accuracy) - 필수
- ✅ BE API 스펙 100% 준수 (엔드포인트 경로, 요청/응답 구조)
- ✅ 타입 안정성 (TypeScript 컴파일 에러 0건)
- ✅ 기존 기능 회귀 없음 (변경된 화면 정상 동작)

### 2. 논리적 일관성 (Logical Consistency) - 필수
- ✅ 인증 헤더 모든 API 호출에 적용
- ✅ 에러 처리 패턴 통일 (401 → 토큰 갱신, 400/500 → 사용자 안내)
- ✅ 데이터 흐름 명확성 (Service → Hook → Screen)

### 3. 완전성 (Completeness) - 필수
- ✅ 각 작업의 수락기준(AC) 100% 충족
- ✅ 관련 타입 정의 업데이트
- ✅ 주석 및 JSDoc 추가

### 4. 테스트 가능성 (Testability) - 권장
- ✅ 변경된 서비스/훅 단위 테스트 가능
- ✅ Mock 데이터 제거 후 실제 API 호출 검증

### 5. 문서화 (Documentation) - 필수
- ✅ 작업 완료 보고서 작성
- ✅ Master_Task_Log.md 업데이트
- ✅ 코드 내 주석으로 변경 이유 명시

---

## 🚀 작업 상세 계획

### 작업 1: [F-001] Attendance 엔드포인트 표준화

**우선순위**: High  
**예상 소요**: 1-2시간  
**담당**: React Native Engineer

#### 1.1 현황 분석
- **문제**: attendanceService.ts의 엔드포인트 경로가 BE 표준과 불일치
- **BE 표준**:
  - `POST /api/attendance/verify/nfc` (기존: `/api/attendance/nfc-verify`)
  - `POST /api/attendance/verify/location` (기존: `/attendance/location-verify`)

#### 1.2 작업 상세
1. **attendanceService.ts 수정**:
   - `verifyNFC()`: `/api/attendance/verify/nfc` 경로로 변경
   - `verifyLocation()`: `/api/attendance/verify/location` 경로로 변경
   - 요청/응답 타입 확인 및 업데이트

2. **AttendanceScreen.tsx 검증**:
   - NFC/위치 인증 호출 부분 확인
   - 에러 처리 로직 검증

3. **타입 정의 업데이트**:
   - `src/types/attendance.ts` 확인
   - 필요 시 응답 타입 보강

#### 1.3 수락 기준 (AC)
- ✅ attendanceService.ts 모든 verify 엔드포인트가 `/api/attendance/verify/*` 패턴 준수
- ✅ TypeScript 컴파일 에러 없음
- ✅ AttendanceScreen 동작 확인 (수동 테스트)

---

### 작업 2: [CRITICAL-003] PersonalUserScreen API 연동

**우선순위**: Critical  
**예상 소요**: 3-4시간  
**담당**: React Native Engineer + Backend Engineer 협업

#### 2.1 현황 분석
- **문제**: PersonalUserScreen이 로컬 상태(하드코딩된 stores)만 사용
- **목표**: 실제 BE API 연동 (다중 매장 조회, 출퇴근 CRUD, 데이터 영속화)

#### 2.2 작업 상세

##### 2.2.1 storeService 연동
1. **getMasterStores() 호출 추가**:
   ```typescript
   // PersonalUserScreen.tsx
   const [stores, setStores] = useState<Store[]>([]);
   const [loading, setLoading] = useState(true);
   
   useEffect(() => {
     loadStores();
   }, []);
   
   const loadStores = async () => {
     try {
       const userId = await getUserId(); // AuthContext에서 가져오기
       const data = await storeService.getMasterStores(userId);
       setStores(data);
     } catch (error) {
       console.error('매장 로드 실패:', error);
       Alert.alert('오류', '매장 정보를 불러올 수 없습니다.');
     } finally {
       setLoading(false);
     }
   };
   ```

##### 2.2.2 attendanceService 연동
1. **출근 처리 (checkIn)**:
   ```typescript
   const handleCheckIn = async (storeId: number) => {
     try {
       const employeeId = await getEmployeeId();
       const location = await getCurrentLocation();
       const result = await attendanceService.checkIn(employeeId, storeId, location);
       
       // 로컬 상태 업데이트
       setWorkSessions(prev => [...prev, result]);
     } catch (error) {
       handleError(error);
     }
   };
   ```

2. **퇴근 처리 (checkOut)**:
   ```typescript
   const handleCheckOut = async (attendanceId: number) => {
     try {
       const result = await attendanceService.checkOut(attendanceId);
       
       // 로컬 상태 업데이트
       setWorkSessions(prev => 
         prev.map(s => s.id === attendanceId ? result : s)
       );
     } catch (error) {
       handleError(error);
     }
   };
   ```

3. **출퇴근 기록 조회**:
   ```typescript
   const loadAttendanceRecords = async () => {
     try {
       const employeeId = await getEmployeeId();
       const from = '2025-10-01';
       const to = new Date().toISOString().split('T')[0];
       const records = await attendanceService.getEmployeeAttendance(employeeId, from, to);
       
       setAllRecords(records);
     } catch (error) {
       console.error('출퇴근 기록 로드 실패:', error);
     }
   };
   ```

##### 2.2.3 하드코딩 제거
- Mock stores 배열 삭제
- workSessions/allRecords 로컬 상태 → BE 동기화

#### 2.3 수락 기준 (AC)
- ✅ PersonalUserScreen에서 실제 매장 데이터 표시
- ✅ 출근/퇴근 처리 시 BE API 호출 및 응답 반영
- ✅ 앱 재시작 후에도 출퇴근 기록 유지 (BE에서 조회)
- ✅ 매장이 없을 때 "매장 추가" 안내 표시

---

### 작업 3: [S-001] InfoListScreen 네비게이션 통합

**우선순위**: Medium  
**예상 소요**: 1시간  
**담당**: React Native Engineer

#### 3.1 현황 분석
- **문제**: InfoListScreen이 HomeNavigator에 통합되지 않음
- **목표**: HomeScreen → InfoList → Detail 플로우 구축

#### 3.2 작업 상세

##### 3.2.1 HomeNavigator 라우트 추가
```typescript
// src/navigation/HomeNavigator.tsx
<Stack.Screen
  name="InfoList"
  component={InfoListScreen}
  options={{ title: '정보 서비스' }}
/>
<Stack.Screen
  name="InfoDetail"
  component={InfoDetailScreen}
  options={{ title: '정보 상세' }}
/>
```

##### 3.2.2 HomeScreen 진입점 추가
```typescript
// HomeScreen.tsx
<TouchableOpacity onPress={() => navigation.navigate('InfoList')}>
  <Text>노무 정보 보기</Text>
</TouchableOpacity>
```

##### 3.2.3 laborInfoService 연동
```typescript
// InfoListScreen.tsx
useEffect(() => {
  loadLaborInfo();
}, []);

const loadLaborInfo = async () => {
  try {
    const data = await laborInfoService.getLaborInfoList();
    setInfoList(data);
  } catch (error) {
    console.error('노무 정보 로드 실패:', error);
  }
};
```

#### 3.3 수락 기준 (AC)
- ✅ HomeScreen에서 InfoList 진입 버튼 동작
- ✅ InfoListScreen에서 실제 노무 정보 표시
- ✅ InfoList → Detail 네비게이션 동작

---

### 작업 4: [S-002] SalaryListScreen 기본 연결

**우선순위**: Medium  
**예상 소요**: 2시간  
**담당**: React Native Engineer

#### 4.1 현황 분석
- **문제**: SalaryListScreen이 네비게이터에 미통합, payrollService 미연동
- **목표**: MyPage → SalaryList 플로우 구축 및 데이터 표시

#### 4.2 작업 상세

##### 4.2.1 HomeNavigator 라우트 추가
```typescript
// src/navigation/HomeNavigator.tsx
<Stack.Screen
  name="SalaryList"
  component={SalaryListScreen}
  options={{ title: '급여 내역' }}
/>
```

##### 4.2.2 useSalaryList 훅 생성
```typescript
// src/features/salary/hooks/useSalaryList.ts
export const useSalaryList = (employeeId: number) => {
  const [salaries, setSalaries] = useState<Payroll[]>([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    loadSalaries();
  }, [employeeId]);
  
  const loadSalaries = async () => {
    try {
      const from = '2025-01-01';
      const to = new Date().toISOString().split('T')[0];
      const data = await payrollService.getEmployeePayrolls(employeeId, from, to);
      setSalaries(data);
    } catch (error) {
      console.error('급여 내역 로드 실패:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return { salaries, loading, refresh: loadSalaries };
};
```

##### 4.2.3 SalaryListScreen 연동
```typescript
// SalaryListScreen.tsx
const SalaryListScreen = ({ route }) => {
  const { employeeId } = route.params;
  const { salaries, loading, refresh } = useSalaryList(employeeId);
  
  if (loading) return <LoadingIndicator />;
  
  return (
    <FlatList
      data={salaries}
      renderItem={({ item }) => <SalaryCard salary={item} />}
      refreshControl={<RefreshControl refreshing={loading} onRefresh={refresh} />}
    />
  );
};
```

##### 4.2.4 MyPage 진입점 추가
```typescript
// EmployeeMyPageScreen.tsx
<TouchableOpacity onPress={() => navigation.navigate('SalaryList', { employeeId })}>
  <Text>급여 조회</Text>
</TouchableOpacity>
```

#### 4.3 수락 기준 (AC)
- ✅ MyPage에서 급여 조회 버튼 → SalaryList 이동
- ✅ SalaryListScreen에서 실제 급여 데이터 표시
- ✅ 새로고침 기능 동작

---

## 📝 체크리스트

### 사전 준비
- [x] BE API 문서 학습 완료
- [x] 작업 계획서 작성
- [ ] 개발 환경 설정 확인 (API_BASE_URL)

### 작업 진행
- [ ] [F-001] Attendance 엔드포인트 표준화
  - [ ] attendanceService.ts 수정
  - [ ] 타입 정의 업데이트
  - [ ] AttendanceScreen 검증
- [ ] [CRITICAL-003] PersonalUserScreen API 연동
  - [ ] storeService.getMasterStores() 연동
  - [ ] attendanceService CRUD 연동
  - [ ] 하드코딩 제거
  - [ ] 데이터 영속화 검증
- [ ] [S-001] InfoListScreen 네비게이션 통합
  - [ ] HomeNavigator 라우트 추가
  - [ ] HomeScreen 진입점 추가
  - [ ] laborInfoService 연동
- [ ] [S-002] SalaryListScreen 기본 연결
  - [ ] HomeNavigator 라우트 추가
  - [ ] useSalaryList 훅 생성
  - [ ] SalaryListScreen 연동
  - [ ] MyPage 진입점 추가

### 검증
- [ ] TypeScript 컴파일 에러 0건
- [ ] 변경된 화면 수동 테스트 완료
- [ ] 네비게이션 플로우 검증

### 문서화
- [ ] 작업 완료 보고서 작성
- [ ] Master_Task_Log.md 업데이트
- [ ] 코드 주석 추가

---

## 🎯 성공 지표 (KPI)

| 지표 | 목표 | 측정 방법 |
|------|------|-----------|
| API 엔드포인트 표준화율 | 100% | attendanceService.ts 경로 검증 |
| PersonalUserScreen API 연동율 | 100% | Mock 데이터 제거 확인 |
| 신규 화면 통합 | 2개 | InfoList, SalaryList 네비게이션 동작 |
| TypeScript 컴파일 에러 | 0건 | npm run tsc --noEmit |
| 작업 소요 시간 | 6-8시간 | 실제 소요 시간 기록 |

---

## 🚨 리스크 및 대응 방안

### 리스크 1: API_BASE_URL 미설정
- **대응**: .env 파일 또는 환경변수 확인, 설정 가이드 참조

### 리스크 2: 인증 토큰 만료
- **대응**: 토큰 갱신 로직 구현 (FE_BE_협업_가이드_v1.0_2025-10-13.md 참조)

### 리스크 3: PersonalUserScreen 복잡도
- **대응**: 작업 단계별 분해, 각 API 호출 독립 테스트

### 리스크 4: 네비게이션 타입 불일치
- **대응**: HomeNavigatorParamList 타입 정의 업데이트

---

## 📅 타임라인

| 작업 | 시작 | 종료 | 소요 시간 |
|------|------|------|-----------|
| 작업계획서 작성 | 00:45 | 01:15 | 0.5시간 |
| [F-001] Attendance 표준화 | 01:15 | 02:45 | 1.5시간 |
| [CRITICAL-003] PersonalUserScreen | 02:45 | 06:15 | 3.5시간 |
| [S-001] InfoListScreen | 06:15 | 07:15 | 1시간 |
| [S-002] SalaryListScreen | 07:15 | 09:15 | 2시간 |
| 검증 및 문서화 | 09:15 | 10:15 | 1시간 |
| **총계** | - | - | **10시간** |

---

## 📚 참조 문서

- [FE_BE_협업_가이드_v1.0_2025-10-13.md](FE_BE_협업_가이드_v1.0_2025-10-13.md) - BE API 스펙
- [Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md](Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md) - 전체 작업 계획
- [BE_Implementation_Status_Report_v1.0_2025-10-11.md](../docs/BackEnd_Report_API/BE_Implementation_Status_Report_v1.0_2025-10-11.md) - BE 구현 현황

---

## 📅 변경 이력

| 날짜 | 버전 | 변경 내용 | 작성자 |
|------|------|-----------|--------|
| 2025-10-13 | 1.0 | 초안 작성 (Week 1-2 나머지 작업 계획) | Junie |
