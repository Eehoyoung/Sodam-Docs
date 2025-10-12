# FE-BE 협업 가이드 (Frontend-Backend Collaboration Guide)

## 📋 문서 정보

- **작성일**: 2025-10-13
- **작성자**: Backend Team (Junie)
- **문서 유형**: 협업 가이드 / API 레퍼런스
- **대상**: Frontend 개발팀
- **목적**: Week 1-2 작업 진행을 위한 BE API 정보 제공
- **관련 문서**:
    - [BE_Implementation_Status_Report_v1.0_2025-10-11.md](BE_Implementation_Status_Report_v1.0_2025-10-11.md)
    - [Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md](Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md)

---

## 🎯 개요

본 문서는 FE팀이 Week 1-2 작업을 진행하는데 필요한 BE API 정보를 제공합니다.

### 핵심 요약

- ✅ **전체 BE 구현 완료율**: 100% (2025-10-11 기준)
- ✅ **즉시 사용 가능 API**: Attendance(9), Store(10), Payroll(8), LaborInfo(4)
- ✅ **인증 방식**: JWT (Access Token + Refresh Token)
- ✅ **베이스 URL**: `{API_BASE_URL}` (환경변수 설정 필요)

---

## 📅 Week 1-2 작업별 필요 API 매핑

### Week 1 Sprint 작업

#### 1. [F-001] Attendance 엔드포인트 표준화

**필요 API**: ✅ 이미 표준화 완료

- `POST /api/attendance/verify/nfc` - NFC 인증
- `POST /api/attendance/verify/location` - 위치 인증
- `POST /api/attendance/check-in` - 출근
- `POST /api/attendance/check-out` - 퇴근

**BE 상태**: 완료 | **FE 조치**: attendanceService.ts 엔드포인트 경로 확인

#### 2. [T-001] CI 스캐너 통합

**필요 API**: 없음 (FE 자체 작업)

#### 3. [T-002] Jest 스위트 핵심 안정화

**필요 API**: 없음 (테스트 작업)

### Week 2 Sprint 작업

#### 4. [CRITICAL-003] PersonalUserScreen API 연동

**필요 API**:

- `GET /api/stores/master/{userId}` - 사용자 매장 목록 조회 ✅
- `POST /api/attendance/check-in` - 출근 기록 생성 ✅
- `POST /api/attendance/check-out` - 퇴근 기록 생성 ✅
- `GET /api/attendance/employee/{employeeId}` - 출퇴근 기록 조회 ✅

**BE 상태**: 전체 완료 | **FE 조치**: storeService, attendanceService 연동

#### 5. [S-001] InfoListScreen 네비게이션 통합

**필요 API**:

- `GET /api/labor-info` - 노무 정보 목록 ✅
- `GET /api/labor-info/{id}` - 노무 정보 상세 ✅
- `GET /api/labor-info/current` - 현재 노무 기준값 ✅

**BE 상태**: 전체 완료 | **FE 조치**: laborInfoService 연동

#### 6. [S-002] SalaryListScreen 기본 연결

**필요 API**:

- `GET /api/payroll/employee/{employeeId}` - 직원 급여 내역 조회 ✅
- `GET /api/payroll/{payrollId}/details` - 급여 상세 내역 ✅
- `GET /api/payroll/{payrollId}/pdf` - 급여명세서 PDF ✅

**BE 상태**: 전체 완료 (PDF 기본 구현) | **FE 조치**: payrollService 연동

---

## 🔐 인증/권한 체계

### JWT 토큰 구조

```typescript
// 로그인 응답
interface LoginResponse {
    success: boolean;
    message: string;
    data: {
        accessToken: string;      // 1시간 유효
        refreshToken: string;     // 7일 유효
        userId: number;
        userGrade: string;        // "EMPLOYEE" | "MANAGER" | "MASTER"
    };
}
```

### 인증 헤더 설정

모든 인증이 필요한 API 요청에는 다음 헤더 필수:

```typescript
headers: {
    'Authorization'
:
    `Bearer ${accessToken}`,
        'Content-Type'
:
    'application/json'
}
```

### 토큰 갱신 플로우

```typescript
// Access Token 만료 시 (401 에러)
async function refreshAccessToken(refreshToken: string) {
    const response = await axios.post('/api/auth/refresh', {
        refreshToken
    });
    return response.data.data; // { accessToken, refreshToken, ... }
}
```

---

## 📦 주요 API 상세 스펙

### 1. Attendance API

#### 1-1. NFC 인증

```http
POST /api/attendance/verify/nfc
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "employeeId": 101,
  "storeId": 12,
  "nfcTagId": "STORE_12_20250901093000"
}
```

**응답 (200 OK)**:

```json
{
  "success": true,
  "message": "NFC 인증 성공",
  "timestamp": "2025-10-13T09:30:00.000Z"
}
```

**응답 (400 Bad Request)**:

```json
{
  "success": false,
  "message": "유효하지 않은 NFC 태그입니다"
}
```

#### 1-2. 출근 처리

```http
POST /api/attendance/check-in
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "employeeId": 101,
  "storeId": 12,
  "checkInTime": "2025-10-13T09:00:00",
  "latitude": 37.5665,
  "longitude": 126.9780,
  "note": "정상 출근"
}
```

**응답 (200 OK)**:

```json
{
  "id": 1001,
  "employeeId": 101,
  "storeId": 12,
  "checkInTime": "2025-10-13T09:00:00",
  "checkOutTime": null,
  "status": "CHECKED_IN",
  "workHours": 0.0,
  "createdAt": "2025-10-13T09:00:01"
}
```

#### 1-3. 출퇴근 기록 조회

```http
GET /api/attendance/employee/{employeeId}?from=2025-10-01&to=2025-10-13
Authorization: Bearer {accessToken}
```

**응답 (200 OK)**:

```json
[
  {
    "id": 1001,
    "employeeId": 101,
    "employeeName": "홍길동",
    "storeId": 12,
    "storeName": "소담 강남점",
    "checkInTime": "2025-10-13T09:00:00",
    "checkOutTime": "2025-10-13T18:00:00",
    "status": "CHECKED_OUT",
    "workHours": 9.0,
    "note": "정상 근무"
  }
]
```

---

### 2. Store API

#### 2-1. 사장 소유 매장 목록 조회

```http
GET /api/stores/master/{userId}
Authorization: Bearer {accessToken}
```

**응답 (200 OK)**:

```json
[
  {
    "id": 12,
    "storeName": "소담 강남점",
    "businessNumber": "123-45-67890",
    "storePhoneNumber": "02-1234-5678",
    "businessType": "카페",
    "storeCode": "SODAM-GN-001",
    "fullAddress": "서울특별시 강남구 테헤란로 123",
    "latitude": 37.5665,
    "longitude": 126.9780,
    "storeStandardHourWage": 10000,
    "employeeCount": 5,
    "createdAt": "2025-01-01T00:00:00"
  }
]
```

#### 2-2. 매장 생성

```http
POST /api/stores
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "storeName": "소담 신촌점",
  "businessNumber": "987-65-43210",
  "storePhoneNumber": "02-9876-5432",
  "businessType": "식당",
  "fullAddress": "서울특별시 서대문구 신촌로 456",
  "storeStandardHourWage": 11000
}
```

**응답 (200 OK)**:

```json
{
  "id": 13,
  "storeName": "소담 신촌점",
  "storeCode": "SODAM-SC-002",
  "createdAt": "2025-10-13T10:00:00"
}
```

---

### 3. Payroll API

#### 3-1. 직원 급여 내역 조회

```http
GET /api/payroll/employee/{employeeId}?from=2025-10-01&to=2025-10-13
Authorization: Bearer {accessToken}
```

**응답 (200 OK)**:

```json
[
  {
    "id": 2001,
    "employeeId": 101,
    "employeeName": "홍길동",
    "storeId": 12,
    "storeName": "소담 강남점",
    "startDate": "2025-10-01",
    "endDate": "2025-10-13",
    "totalWorkHours": 90.0,
    "hourlyWage": 10000,
    "basePay": 900000,
    "overtimePay": 50000,
    "nightShiftPay": 30000,
    "holidayPay": 20000,
    "totalPay": 1000000,
    "deductions": 50000,
    "netPay": 950000,
    "status": "CONFIRMED"
  }
]
```

#### 3-2. 급여명세서 PDF 다운로드

```http
GET /api/payroll/{payrollId}/pdf
Authorization: Bearer {accessToken}
```

**응답**: Binary (application/pdf)
**Content-Disposition**: `attachment; filename="payroll_{payrollId}.pdf"`

---

### 4. LaborInfo API

#### 4-1. 현재 노무 기준값 조회

```http
GET /api/labor-info/current
Authorization: Bearer {accessToken}
```

**응답 (200 OK)**:

```json
{
  "id": 123,
  "title": "2025년 최저임금 및 근로기준",
  "content": "2025년 노무 기준...",
  "imagePath": "/uploads/labor-info/...",
  "minimumWage": 10030,
  "year": 2025,
  "weeklyMaxHours": 40,
  "overtimeRate": 1.5,
  "createdAt": "2025-01-01T00:00:00"
}
```

#### 4-2. 노무 정보 목록 조회

```http
GET /api/labor-info
Authorization: Bearer {accessToken}
```

**응답 (200 OK)**: 노무 정보 배열

---

## 💡 React Native 사용 예시

### 1. AttendanceService 연동

```typescript
// services/attendanceService.ts
import axios from 'axios';
import {getAccessToken} from './authService';

const API_BASE_URL = process.env.API_BASE_URL;

export const attendanceService = {
    async verifyNFC(employeeId: number, storeId: number, nfcTagId: string) {
        const token = await getAccessToken();
        const response = await axios.post(
            `${API_BASE_URL}/api/attendance/verify/nfc`,
            {employeeId, storeId, nfcTagId},
            {headers: {Authorization: `Bearer ${token}`}}
        );
        return response.data;
    },

    async checkIn(employeeId: number, storeId: number, location: { latitude: number, longitude: number }) {
        const token = await getAccessToken();
        const response = await axios.post(
            `${API_BASE_URL}/api/attendance/check-in`,
            {
                employeeId,
                storeId,
                checkInTime: new Date().toISOString(),
                latitude: location.latitude,
                longitude: location.longitude
            },
            {headers: {Authorization: `Bearer ${token}`}}
        );
        return response.data;
    },

    async getEmployeeAttendance(employeeId: number, from: string, to: string) {
        const token = await getAccessToken();
        const response = await axios.get(
            `${API_BASE_URL}/api/attendance/employee/${employeeId}?from=${from}&to=${to}`,
            {headers: {Authorization: `Bearer ${token}`}}
        );
        return response.data;
    }
};
```

### 2. StoreService 연동

```typescript
// services/storeService.ts
import axios from 'axios';
import {getAccessToken} from './authService';

const API_BASE_URL = process.env.API_BASE_URL;

export const storeService = {
    async getMasterStores(userId: number) {
        const token = await getAccessToken();
        const response = await axios.get(
            `${API_BASE_URL}/api/stores/master/${userId}`,
            {headers: {Authorization: `Bearer ${token}`}}
        );
        return response.data;
    },

    async createStore(storeData: any) {
        const token = await getAccessToken();
        const response = await axios.post(
            `${API_BASE_URL}/api/stores`,
            storeData,
            {headers: {Authorization: `Bearer ${token}`}}
        );
        return response.data;
    }
};
```

### 3. PayrollService 연동

```typescript
// services/payrollService.ts
import axios from 'axios';
import {getAccessToken} from './authService';

const API_BASE_URL = process.env.API_BASE_URL;

export const payrollService = {
    async getEmployeePayrolls(employeeId: number, from: string, to: string) {
        const token = await getAccessToken();
        const response = await axios.get(
            `${API_BASE_URL}/api/payroll/employee/${employeeId}?from=${from}&to=${to}`,
            {headers: {Authorization: `Bearer ${token}`}}
        );
        return response.data;
    },

    async downloadPayrollPDF(payrollId: number) {
        const token = await getAccessToken();
        const response = await axios.get(
            `${API_BASE_URL}/api/payroll/${payrollId}/pdf`,
            {
                headers: {Authorization: `Bearer ${token}`},
                responseType: 'blob'
            }
        );
        return response.data; // Blob 데이터
    }
};
```

---

## 🚨 에러 처리 가이드

### 공통 에러 코드

| 코드  | 설명                     | FE 대응         |
|-----|------------------------|---------------|
| 400 | 잘못된 요청 (파라미터 누락/형식 오류) | 요청 데이터 검증     |
| 401 | 인증 실패 (토큰 만료/무효)       | 토큰 갱신 또는 재로그인 |
| 403 | 권한 없음                  | 권한 안내 메시지     |
| 404 | 리소스 없음                 | 적절한 안내 메시지    |
| 500 | 서버 내부 오류               | 재시도 또는 지원 요청  |

### 에러 처리 예시

```typescript
import axios, {AxiosError} from 'axios';

async function handleAPICall<T>(apiCall: () => Promise<T>): Promise<T> {
    try {
        return await apiCall();
    } catch (error) {
        if (axios.isAxiosError(error)) {
            const axiosError = error as AxiosError;

            if (axiosError.response?.status === 401) {
                // 토큰 만료 - 갱신 시도
                const refreshed = await refreshAccessToken();
                if (refreshed) {
                    return await apiCall(); // 재시도
                } else {
                    // 로그아웃 처리
                    await logout();
                    throw new Error('인증이 만료되었습니다. 다시 로그인해주세요.');
                }
            } else if (axiosError.response?.status === 400) {
                throw new Error('잘못된 요청입니다. 입력 내용을 확인해주세요.');
            } else if (axiosError.response?.status === 500) {
                throw new Error('서버 오류가 발생했습니다. 잠시 후 다시 시도해주세요.');
            }
        }
        throw error;
    }
}

// 사용 예시
const attendance = await handleAPICall(() =>
    attendanceService.checkIn(userId, storeId, location)
);
```

---

## 📞 협업 채널

### 긴급 문의

- **Slack**: `#fe-be-sync` 채널 (실시간)
- **GitHub**: Issue에 `[BE-Collaboration]` 태그

### 정기 미팅

- **주기**: 매주 월/목 오전 10:00 (30분)
- **참석**: FE Lead + BE Lead + 관련 Engineer

### API 문제 보고 시 포함 정보

1. 엔드포인트 경로
2. 요청 바디/헤더
3. 응답 코드 및 메시지
4. 에러 발생 시간
5. 재현 방법

---

## 📚 추가 참고 문서

### Backend 상세 문서

- [BE_Implementation_Status_Report_v1.0_2025-10-11.md](BE_Implementation_Status_Report_v1.0_2025-10-11.md) - 전체 BE 구현 현황
- [AttendanceController_API_문서.md](../docs/technical/api/AttendanceController_API_문서.md) - Attendance API 상세
- [LoginController_API_문서.md](../docs/technical/api/LoginController_API_문서.md) - 인증 API 상세
- [StoreController_API_문서.md](../docs/technical/api/StoreController_API_문서.md) - Store API 상세
- [PayrollController_API_문서.md](../docs/technical/api/PayrollController_API_문서.md) - Payroll API 상세

### Frontend 작업 계획

- [Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md](Sodam_FE_Work_Plan_2M_Timeline_v1.0_2025-10-11.md) - FE 2개월 작업 계획

---

## ✅ 체크리스트: Week 1-2 작업 준비

### Week 1

- [ ] Attendance API 엔드포인트 경로 확인 및 수정
- [ ] attendanceService.ts 업데이트
- [ ] CI 스캐너 통합 (BE 작업 불필요)
- [ ] 핵심 테스트 안정화

### Week 2

- [ ] PersonalUserScreen: storeService.getMasterStores() 연동
- [ ] PersonalUserScreen: attendanceService CRUD 연동
- [ ] InfoListScreen: laborInfoService 연동
- [ ] SalaryListScreen: payrollService 연동
- [ ] 에러 처리 로직 구현

---

## 📅 변경 이력

| 날짜         | 버전  | 변경 내용                            | 작성자                  |
|------------|-----|----------------------------------|----------------------|
| 2025-10-13 | 1.0 | FE-BE 협업 가이드 초안 작성 (Week 1-2 중심) | Backend Team (Junie) |
