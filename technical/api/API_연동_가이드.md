# 소담(SODAM) API 연동 가이드

## 📋 개요

- **작성일**: 2025-10-14
- **작성자**: 소담 백엔드팀
- **문서 유형**: API 연동 가이드
- **대상**: 프론트엔드 개발자 (초급~중급)

## 🎯 목적

본 문서는 소담 API를 React Native 프론트엔드에 연동하는 방법을 단계별로 설명합니다.
초급 개발자도 쉽게 이해하고 바로 작업할 수 있도록 상세한 예시와 설명을 제공합니다.

---

## 📚 목차

1. [API 기본 정보](#1-api-기본-정보)
2. [인증 및 토큰 관리](#2-인증-및-토큰-관리)
3. [주요 API 사용 예시](#3-주요-api-사용-예시)
4. [에러 처리 가이드](#4-에러-처리-가이드)
5. [환경별 설정](#5-환경별-설정)
6. [체크리스트](#6-체크리스트)

---

## 1. API 기본 정보

### 1.1 Base URL

```javascript
// 환경별 Base URL
const API_BASE_URL = {
  local: 'http://localhost:8080',
  dev: 'https://dev-api.sodam.com',
  prod: 'https://sodam-api.com'
};
```

### 1.2 API 명세서 위치

- **OpenAPI 문서**: `ApiList.yaml` (프로젝트 루트)
- **Swagger UI**: `http://localhost:8080/swagger-ui/index.html` (로컬 서버 실행 시)

### 1.3 공통 헤더

```javascript
// 모든 API 요청에 포함해야 하는 헤더
const headers = {
  'Content-Type': 'application/json',
  'Authorization': `Bearer ${accessToken}`, // 인증 필요 API만
};
```

---

## 2. 인증 및 토큰 관리

### 2.1 JWT 토큰 구조

소담 API는 JWT(JSON Web Token) 기반 인증을 사용합니다.

```javascript
// 토큰 구조
{
  accessToken: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",  // 1시간 유효
  refreshToken: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...", // 7일 유효
  tokenType: "Bearer"
}
```

### 2.2 로그인 (이메일/비밀번호)

**엔드포인트**: `POST /api/auth/login`

```javascript
import axios from 'axios';

// 로그인 함수
const login = async (email, password) => {
  try {
    const response = await axios.post(
      `${API_BASE_URL.dev}/api/auth/login`,
      {
        email: email,
        password: password
      }
    );
    
    // 토큰 저장
    const { accessToken, refreshToken } = response.data;
    await AsyncStorage.setItem('accessToken', accessToken);
    await AsyncStorage.setItem('refreshToken', refreshToken);
    
    console.log('로그인 성공!');
    return response.data;
  } catch (error) {
    console.error('로그인 실패:', error.response?.data);
    throw error;
  }
};

// 사용 예시
login('user@example.com', 'password123');
```

### 2.3 카카오 소셜 로그인

**엔드포인트**: `POST /api/auth/kakao`

```javascript
import { login as kakaoLogin } from '@react-native-seoul/kakao-login';

const loginWithKakao = async () => {
  try {
    // 1. 카카오 로그인으로 토큰 획득
    const kakaoToken = await kakaoLogin();
    
    // 2. 백엔드에 카카오 토큰 전송
    const response = await axios.post(
      `${API_BASE_URL.dev}/api/auth/kakao`,
      {
        kakaoAccessToken: kakaoToken.accessToken
      }
    );
    
    // 3. JWT 토큰 저장
    const { accessToken, refreshToken } = response.data;
    await AsyncStorage.setItem('accessToken', accessToken);
    await AsyncStorage.setItem('refreshToken', refreshToken);
    
    return response.data;
  } catch (error) {
    console.error('카카오 로그인 실패:', error);
    throw error;
  }
};
```

### 2.4 토큰 갱신

**엔드포인트**: `POST /api/auth/refresh`

```javascript
// 토큰 갱신 함수
const refreshAccessToken = async () => {
  try {
    const refreshToken = await AsyncStorage.getItem('refreshToken');
    
    const response = await axios.post(
      `${API_BASE_URL.dev}/api/auth/refresh`,
      {
        refreshToken: refreshToken
      }
    );
    
    const { accessToken } = response.data;
    await AsyncStorage.setItem('accessToken', accessToken);
    
    return accessToken;
  } catch (error) {
    // Refresh Token도 만료된 경우 재로그인 필요
    console.error('토큰 갱신 실패:', error);
    // 로그인 화면으로 이동
    throw error;
  }
};
```

### 2.5 Axios Interceptor 설정 (권장)

```javascript
import axios from 'axios';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Axios 인스턴스 생성
const apiClient = axios.create({
  baseURL: API_BASE_URL.dev,
  timeout: 10000,
});

// 요청 인터셉터: 모든 요청에 토큰 자동 포함
apiClient.interceptors.request.use(
  async (config) => {
    const token = await AsyncStorage.getItem('accessToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// 응답 인터셉터: 401 에러 시 자동 토큰 갱신
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    
    // 401 에러이고, 재시도하지 않은 요청인 경우
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        // 토큰 갱신 시도
        const newAccessToken = await refreshAccessToken();
        originalRequest.headers.Authorization = `Bearer ${newAccessToken}`;
        
        // 원래 요청 재시도
        return apiClient(originalRequest);
      } catch (refreshError) {
        // 토큰 갱신 실패 시 로그인 화면으로
        // navigation.navigate('Login');
        return Promise.reject(refreshError);
      }
    }
    
    return Promise.reject(error);
  }
);

export default apiClient;
```

---

## 3. 주요 API 사용 예시

### 3.1 출퇴근 관리

#### 3.1.1 출근 등록 (NFC 태그)

**엔드포인트**: `POST /api/attendance/check-in`

**목적**: 직원이 출근 시 NFC 태그를 통해 출근 기록

**비즈니스 로직**:

1. NFC 태그 ID와 매장 정보 확인
2. 중복 출근 체크 (이미 출근 상태인지)
3. 출근 시간 기록
4. 위치 정보 검증 (옵션)

```javascript
import apiClient from './apiClient';

const checkIn = async (storeId, nfcTagId, location) => {
  try {
    const response = await apiClient.post('/api/attendance/check-in', {
      storeId: storeId,
      nfcTagId: nfcTagId,
      latitude: location.latitude,   // 선택사항
      longitude: location.longitude   // 선택사항
    });
    
    console.log('출근 등록 성공:', response.data);
    alert('출근이 등록되었습니다!');
    
    return response.data;
  } catch (error) {
    if (error.response?.status === 400) {
      alert('이미 출근 상태입니다.');
    } else if (error.response?.status === 404) {
      alert('매장 정보를 찾을 수 없습니다.');
    } else {
      alert('출근 등록에 실패했습니다.');
    }
    throw error;
  }
};

// 사용 예시
checkIn(1, 'NFC123456', { latitude: 37.5665, longitude: 126.9780 });
```

#### 3.1.2 퇴근 등록

**엔드포인트**: `POST /api/attendance/check-out`

```javascript
const checkOut = async (attendanceId) => {
  try {
    const response = await apiClient.post('/api/attendance/check-out', {
      attendanceId: attendanceId
    });
    
    const { workHours, overtimeHours } = response.data;
    console.log(`근무 시간: ${workHours}시간, 연장 근무: ${overtimeHours}시간`);
    alert('퇴근이 등록되었습니다!');
    
    return response.data;
  } catch (error) {
    console.error('퇴근 등록 실패:', error);
    throw error;
  }
};
```

#### 3.1.3 출퇴근 기록 조회

**엔드포인트**: `GET /api/attendance/employee/{employeeId}`

```javascript
const getAttendanceRecords = async (employeeId, startDate, endDate) => {
  try {
    const response = await apiClient.get(
      `/api/attendance/employee/${employeeId}`,
      {
        params: {
          startDate: startDate,  // 'YYYY-MM-DD' 형식
          endDate: endDate
        }
      }
    );
    
    console.log('출퇴근 기록:', response.data);
    return response.data;
  } catch (error) {
    console.error('조회 실패:', error);
    throw error;
  }
};

// 사용 예시: 이번 달 기록 조회
const now = new Date();
const startDate = new Date(now.getFullYear(), now.getMonth(), 1)
  .toISOString().split('T')[0];
const endDate = new Date(now.getFullYear(), now.getMonth() + 1, 0)
  .toISOString().split('T')[0];

getAttendanceRecords(10, startDate, endDate);
```

### 3.2 급여 관리

#### 3.2.1 급여 계산

**엔드포인트**: `POST /api/payroll/calculate`

**목적**: 특정 직원의 월별 급여 자동 계산

**비즈니스 로직**:

1. 해당 월의 출퇴근 기록 조회
2. 근무 시간 합산 (정규 + 연장 + 야간)
3. 시급 및 수당 적용
4. 공제 항목 차감
5. 최종 급여 산출

```javascript
const calculatePayroll = async (employeeId, storeId, year, month) => {
  try {
    const response = await apiClient.post('/api/payroll/calculate', {
      employeeId: employeeId,
      storeId: storeId,
      year: year,
      month: month
    });
    
    const payroll = response.data;
    console.log('급여 계산 결과:');
    console.log(`- 기본급: ${payroll.basePay.toLocaleString()}원`);
    console.log(`- 수당: ${payroll.allowances.toLocaleString()}원`);
    console.log(`- 공제: ${payroll.deductions.toLocaleString()}원`);
    console.log(`- 실수령액: ${payroll.netPay.toLocaleString()}원`);
    
    return payroll;
  } catch (error) {
    console.error('급여 계산 실패:', error);
    throw error;
  }
};

// 사용 예시: 2025년 10월 급여 계산
calculatePayroll(10, 1, 2025, 10);
```

#### 3.2.2 급여 명세서 조회

**엔드포인트**: `GET /api/payroll/{payrollId}`

```javascript
const getPayrollDetails = async (payrollId) => {
  try {
    const response = await apiClient.get(`/api/payroll/${payrollId}`);
    
    // 급여 명세서 화면에 표시
    return response.data;
  } catch (error) {
    console.error('명세서 조회 실패:', error);
    throw error;
  }
};
```

### 3.3 사용자 관리

#### 3.3.1 내 정보 조회

**엔드포인트**: `GET /api/auth/me`

```javascript
const getMyProfile = async () => {
  try {
    const response = await apiClient.get('/api/auth/me');
    
    const user = response.data;
    console.log('사용자 정보:', user);
    // { id, email, name, role, storeId, ... }
    
    return user;
  } catch (error) {
    console.error('프로필 조회 실패:', error);
    throw error;
  }
};
```

#### 3.3.2 직원 정보 수정 (관리자)

**엔드포인트**: `PUT /api/user/{employeeId}`

```javascript
const updateEmployee = async (employeeId, updateData) => {
  try {
    const response = await apiClient.put(
      `/api/user/${employeeId}`,
      updateData
    );
    
    alert('직원 정보가 수정되었습니다.');
    return response.data;
  } catch (error) {
    if (error.response?.status === 403) {
      alert('권한이 없습니다.');
    }
    throw error;
  }
};

// 사용 예시
updateEmployee(10, {
  name: '홍길동',
  position: '매니저',
  hourlyWage: 12000
});
```

### 3.4 매장 관리

#### 3.4.1 매장 등록

**엔드포인트**: `POST /api/stores/registration`

```javascript
const registerStore = async (storeData) => {
  try {
    const response = await apiClient.post(
      '/api/stores/registration',
      {
        name: storeData.name,
        address: storeData.address,
        businessNumber: storeData.businessNumber,
        latitude: storeData.latitude,
        longitude: storeData.longitude,
        standardHourlyWage: storeData.standardHourlyWage || 9860
      }
    );
    
    console.log('매장 등록 성공:', response.data);
    return response.data;
  } catch (error) {
    if (error.response?.status === 400) {
      alert('입력 정보를 확인해주세요.');
    } else if (error.response?.status === 409) {
      alert('이미 등록된 사업자번호입니다.');
    }
    throw error;
  }
};
```

#### 3.4.2 매장별 직원 목록 조회

**엔드포인트**: `GET /api/stores/{storeId}/employees`

```javascript
const getStoreEmployees = async (storeId) => {
  try {
    const response = await apiClient.get(
      `/api/stores/${storeId}/employees`
    );
    
    console.log('직원 목록:', response.data);
    return response.data;
  } catch (error) {
    console.error('직원 목록 조회 실패:', error);
    throw error;
  }
};
```

### 3.5 통계 조회

#### 3.5.1 매장 통계 조회

**엔드포인트**: `GET /api/store-queries/{storeId}/stats/attendance/count`

```javascript
const getAttendanceStats = async (storeId, startDate, endDate) => {
  try {
    const response = await apiClient.get(
      `/api/store-queries/${storeId}/stats/attendance/count`,
      {
        params: { startDate, endDate }
      }
    );
    
    console.log('출근 통계:', response.data);
    return response.data;
  } catch (error) {
    console.error('통계 조회 실패:', error);
    throw error;
  }
};
```

---

## 4. 에러 처리 가이드

### 4.1 HTTP 상태 코드별 의미

| 상태 코드 | 의미     | 처리 방법         |
|-------|--------|---------------|
| 200   | 성공     | 정상 처리         |
| 400   | 잘못된 요청 | 입력값 검증        |
| 401   | 인증 실패  | 토큰 갱신 또는 재로그인 |
| 403   | 권한 없음  | 권한 확인 메시지 표시  |
| 404   | 리소스 없음 | 존재하지 않는 데이터   |
| 409   | 충돌     | 중복 데이터 처리     |
| 500   | 서버 오류  | 서버 관리자에게 문의   |

### 4.2 공통 에러 처리 함수

```javascript
const handleApiError = (error) => {
  if (error.response) {
    // 서버가 응답을 반환한 경우
    const status = error.response.status;
    const message = error.response.data?.message || '오류가 발생했습니다.';
    
    switch (status) {
      case 400:
        alert(`입력 오류: ${message}`);
        break;
      case 401:
        alert('로그인이 필요합니다.');
        // navigation.navigate('Login');
        break;
      case 403:
        alert('권한이 없습니다.');
        break;
      case 404:
        alert('요청한 정보를 찾을 수 없습니다.');
        break;
      case 500:
        alert('서버 오류가 발생했습니다. 잠시 후 다시 시도해주세요.');
        break;
      default:
        alert(message);
    }
  } else if (error.request) {
    // 요청이 전송되었으나 응답이 없는 경우
    alert('서버와 통신할 수 없습니다. 네트워크를 확인해주세요.');
  } else {
    // 요청 설정 중 오류
    alert('요청 처리 중 오류가 발생했습니다.');
  }
  
  console.error('API Error:', error);
};

// 사용 예시
try {
  await apiClient.get('/api/some-endpoint');
} catch (error) {
  handleApiError(error);
}
```

---

## 5. 환경별 설정

### 5.1 환경 변수 설정 (.env)

```bash
# .env.development
REACT_APP_API_BASE_URL=https://dev-api.sodam.com
REACT_APP_ENV=development

# .env.production
REACT_APP_API_BASE_URL=https://sodam-api.com
REACT_APP_ENV=production
```

### 5.2 Config 파일

```javascript
// config/api.config.js
const getApiBaseUrl = () => {
  if (__DEV__) {
    return 'https://dev-api.sodam.com';
  }
  return 'https://sodam-api.com';
};

export const API_CONFIG = {
  BASE_URL: getApiBaseUrl(),
  TIMEOUT: 10000,
};
```

---

## 6. 체크리스트

### 6.1 API 연동 전 확인사항

- [ ] ApiList.yaml 파일 확인
- [ ] Swagger UI에서 API 스펙 확인
- [ ] Base URL 환경별 설정 완료
- [ ] Axios 인스턴스 생성 및 Interceptor 설정
- [ ] AsyncStorage 설정 완료

### 6.2 인증 구현 체크리스트

- [ ] 로그인 API 연동
- [ ] 토큰 저장 로직 구현
- [ ] 토큰 갱신 로직 구현
- [ ] 자동 로그아웃 처리
- [ ] Authorization 헤더 자동 포함

### 6.3 에러 처리 체크리스트

- [ ] HTTP 상태 코드별 처리
- [ ] 네트워크 오류 처리
- [ ] 사용자 친화적 에러 메시지
- [ ] 에러 로깅

---

## 🔗 관련 문서

- [ApiList.yaml](../../../ApiList.yaml) - API 명세서
- [프로젝트 기획서](../../project-management/project_planning_Document_v2.md)
- [개발 가이드라인](../../guidelines/development_guidelines.md)

---

## 📅 변경 이력

| 날짜         | 버전  | 변경 내용 | 작성자  |
|------------|-----|-------|------|
| 2025-10-14 | 1.0 | 초기 작성 | 백엔드팀 |
