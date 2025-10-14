# Sodam 백엔드 개발 가이드

이 문서는 Sodam 프론트엔드 애플리케이션과 연동할 백엔드 API 개발에 필요한 정보를 제공합니다.

## 목차

1. [개요](#개요)
2. [인증 및 권한](#인증-및-권한)
3. [API 엔드포인트 명세](#api-엔드포인트-명세)
4. [데이터 모델](#데이터-모델)
5. [에러 처리](#에러-처리)
6. [개발 환경 설정](#개발-환경-설정)

## 개요

Sodam은 소상공인과 직원들을 위한 근무 관리 및 정보 제공 플랫폼입니다. 백엔드는 다음 기능을 제공해야 합니다:

- 사용자 인증 및 계정 관리
- 매장 정보 관리
- 근무 기록 및 출퇴근 관리
- 급여 정보 관리
- 노무 정보 제공 및 북마크 기능

## 인증 및 권한

### 인증 방식

- JWT(JSON Web Token) 기반 인증 구현
- 토큰 만료 시간: Access Token(1시간), Refresh Token(2주)
- 모든 API 요청에는 Authorization 헤더에 Bearer 토큰 포함 필요

### 사용자 역할

- 일반 사용자: 기본 정보 조회만 가능
- 사장(Owner): 매장 관리, 직원 관리, 급여 관리 권한
- 직원(Employee): 출퇴근 기록, 급여 조회 권한

### 인증 관련 코드 예시

```typescript
// 프론트엔드 인증 요청 예시
const login = async (email: string, password: string) => {
  try {
    const response = await axios.post('/api/auth/login', { email, password });
    const { accessToken, refreshToken } = response.data;

    // 토큰 저장
    localStorage.setItem('accessToken', accessToken);
    localStorage.setItem('refreshToken', refreshToken);

    return true;
  } catch (error) {
    console.error('Login failed:', error);
    return false;
  }
};
```

## API 엔드포인트 명세

### 1. 사용자 관리 API

#### 사용자 정보 조회

- **엔드포인트**: `GET /api/users/profile`
- **인증**: 필수
- **응답 데이터**:

```json
{
  "id": "user123",
  "name": "홍길동",
  "email": "user@example.com",
  "profileImageUrl": "https://example.com/profile.jpg",
  "role": "USER"
}
```

- **참고**: `role` 필드는 "USER", "OWNER", "EMPLOYEE" 중 하나의 값을 가집니다.
- **구현 위치**: `UserMyPageScreen.tsx` (Line 24)
- **구현 예시**:

```typescript
// UserMyPageScreen.tsx에 추가할 코드
const fetchUserProfile = async () => {
  try {
    const response = await axios.get('/api/users/profile', {
      headers: { Authorization: `Bearer ${accessToken}` }
    });
    setUser(response.data);
  } catch (error) {
    console.error('Failed to fetch user profile:', error);
    // 에러 처리
  }
};

useEffect(() => {
  fetchUserProfile();
}, []);
```

#### 사용자 역할 변경 (사장으로)

- **엔드포인트**: `PUT /api/users/role/owner`
- **인증**: 필수
- **응답 데이터**:

```json
{
  "success": true,
  "message": "Role changed successfully",
  "role": "OWNER"
}
```

- **구현 위치**: `UserMyPageScreen.tsx` (Line 44)
- **구현 예시**:

```typescript
// UserMyPageScreen.tsx에 추가할 코드
const changeToOwnerRole = async () => {
  try {
    const response = await axios.put('/api/users/role/owner', {}, {
      headers: { Authorization: `Bearer ${accessToken}` }
    });

    if (response.data.success) {
      Alert.alert('성공', '사장 계정으로 전환되었습니다.');
      navigation.navigate('MasterMyPageScreen');
    }
  } catch (error) {
    console.error('Failed to change role:', error);
    Alert.alert('오류', '역할 변경 중 오류가 발생했습니다.');
  }
};
```

#### 사용자 역할 변경 (직원으로)

- **엔드포인트**: `PUT /api/users/role/employee`
- **인증**: 필수
- **응답 데이터**: 사장 역할 변경과 동일
- **구현 위치**: `UserMyPageScreen.tsx` (Line 67)
- **구현 예시**: 사장 역할 변경과 유사 (엔드포인트만 변경)

### 2. 노무 정보 API

#### 노무 정보 상세 조회

- **엔드포인트**: `GET /api/labor-info/{infoId}`
- **인증**: 필수
- **응답 데이터**:

```json
{
  "id": 123,
  "title": "2024년 최저임금 변경에 따른 급여 계산 방법",
  "date": "2024-05-15",
  "content": "# 2024년 최저임금 변경 안내\n\n2024년 최저임금이 시간당 9,860원으로 인상되었습니다...",
  "author": "소담 노무팀",
  "views": 1245,
  "category": "노무 정보",
  "isBookmarked": false
}
```

- **구현 위치**: `LaborInfoDetailScreen.tsx` (Line 46)
- **구현 예시**:

```typescript
// LaborInfoDetailScreen.tsx에 추가할 코드
const fetchLaborInfo = async () => {
  try {
    setLoading(true);
    const response = await axios.get(`/api/labor-info/${infoId}`, {
      headers: { Authorization: `Bearer ${accessToken}` }
    });
    setLaborInfo(response.data);
    setIsBookmarked(response.data.isBookmarked);
    setLoading(false);
  } catch (error) {
    console.error('노무 정보를 불러오는 중 오류가 발생했습니다:', error);
    setToastMessage('정보를 불러오는 중 오류가 발생했습니다.');
    setToastType('error');
    setShowToast(true);
    setLoading(false);
  }
};
```

#### 북마크 상태 저장

- **엔드포인트**: `POST /api/labor-info/{infoId}/bookmark`
- **인증**: 필수
- **요청 데이터**:

```json
{
  "isBookmarked": true
}
```

- **응답 데이터**:

```json
{
  "success": true,
  "message": "Bookmark updated successfully"
}
```

- **구현 위치**: `LaborInfoDetailScreen.tsx` (Line 113)
- **구현 예시**:

```typescript
// LaborInfoDetailScreen.tsx에 추가할 코드
const toggleBookmark = async () => {
  try {
    const newBookmarkState = !isBookmarked;
    const response = await axios.post(`/api/labor-info/${infoId}/bookmark`, 
      { isBookmarked: newBookmarkState },
      { headers: { Authorization: `Bearer ${accessToken}` } }
    );

    if (response.data.success) {
      setIsBookmarked(newBookmarkState);
      setToastMessage(newBookmarkState ? '북마크에 추가되었습니다.' : '북마크가 해제되었습니다.');
      setToastType('success');
      setShowToast(true);
    }
  } catch (error) {
    console.error('북마크 상태 변경 중 오류가 발생했습니다:', error);
    setToastMessage('북마크 상태 변경 중 오류가 발생했습니다.');
    setToastType('error');
    setShowToast(true);
  }
};
```

### 3. 매장 관리 API

#### 매장 목록 조회

- **엔드포인트**: `GET /api/workplaces`
- **인증**: 필수
- **응답 데이터**:

```json
[
  {
    "id": "1",
    "name": "카페 소담",
    "address": "서울시 강남구 역삼동 123-45"
  },
  {
    "id": "2",
    "name": "레스토랑 소담",
    "address": "서울시 서초구 서초동 456-78"
  }
]
```

- **구현 위치**: `workplaceService.ts` (Line 24)
- **구현 예시**:

```typescript
// workplaceService.ts에 추가할 코드
export const getWorkplaces = async (): Promise<Workplace[]> => {
  try {
    const response = await axios.get('/api/workplaces', {
      headers: { Authorization: `Bearer ${accessToken}` }
    });
    return response.data;
  } catch (error) {
    console.error('Failed to fetch workplaces:', error);
    throw new Error('매장 목록을 불러오는데 실패했습니다.');
  }
};
```

#### 특정 매장 정보 조회

- **엔드포인트**: `GET /api/workplaces/{id}`
- **인증**: 필수
- **응답 데이터**:

```json
{
  "id": "1",
  "name": "카페 소담",
  "address": "서울시 강남구 역삼동 123-45",
  "businessNumber": "123-45-67890",
  "ownerName": "홍길동",
  "contactNumber": "02-1234-5678",
  "employeeCount": 5
}
```

- **구현 위치**: `workplaceService.ts` (Line 35)
- **구현 예시**:

```typescript
// workplaceService.ts에 추가할 코드
export const getWorkplaceById = async (id: string): Promise<Workplace | undefined> => {
  try {
    const response = await axios.get(`/api/workplaces/${id}`, {
      headers: { Authorization: `Bearer ${accessToken}` }
    });
    return response.data;
  } catch (error) {
    console.error(`Failed to fetch workplace with id ${id}:`, error);
    throw new Error('매장 정보를 불러오는데 실패했습니다.');
  }
};
```

### 4. 출퇴근 관리 API

#### 출퇴근 정보 조회

- **엔드포인트**: `GET /api/attendance/today`
- **인증**: 필수
- **응답 데이터**:

```json
{
  "id": "1",
  "date": "2024-06-25",
  "checkInTime": "09:00",
  "checkOutTime": null,
  "workingHours": null
}
```

- **구현 위치**: `useAttendance.ts` (Line 22)
- **구현 예시**:

```typescript
// useAttendance.ts에 추가할 코드
const fetchAttendance = async () => {
  try {
    setIsLoading(true);
    const response = await axios.get('/api/attendance/today', {
      headers: { Authorization: `Bearer ${accessToken}` }
    });
    setTodayAttendance(response.data);
    setIsLoading(false);
  } catch (err) {
    setError(err instanceof Error ? err : new Error('Unknown error'));
    setIsLoading(false);
  }
};
```

### 5. 회원가입 API

#### 이메일 중복 확인

- **엔드포인트**: `GET /api/auth/check-email?email={email}`
- **인증**: 불필요
- **응답 데이터**:

```json
{
  "exists": false,
  "message": "사용 가능한 이메일입니다."
}
```

- **구현 위치**: `SignupScreen.tsx` (Line 68)
- **구현 예시**:

```typescript
// SignupScreen.tsx에 추가할 코드
const checkEmailDuplicate = async () => {
  if (!email) {
    setErrors(prev => ({...prev, email: '이메일을 입력해주세요.'}));
    return;
  }

  if (!validateEmail(email)) {
    setErrors(prev => ({...prev, email: '유효한 이메일 형식이 아닙니다.'}));
    return;
  }

  setIsDuplicateChecking(true);

  try {
    const response = await axios.get(`/api/auth/check-email?email=${encodeURIComponent(email)}`);

    if (response.data.exists) {
      setErrors(prev => ({...prev, email: '이미 사용중인 이메일입니다.'}));
      setIsEmailVerified(false);
    } else {
      setErrors(prev => ({...prev, email: ''}));
      setIsEmailVerified(true);
      Alert.alert('확인 완료', '사용 가능한 이메일입니다.');
    }
  } catch (error) {
    Alert.alert('오류', '중복 확인 중 오류가 발생했습니다. 다시 시도해주세요.');
  } finally {
    setIsDuplicateChecking(false);
  }
};
```

### 6. 급여 정보 API

#### 급여 정보 조회

- **엔드포인트**: `GET /api/salary/monthly`
- **인증**: 필수
- **응답 데이터**:

```json
{
  "amount": 1850000,
  "workingHours": 160,
  "deductions": 185000
}
```

- **구현 위치**: `useSalary.ts` (Line 20)
- **구현 예시**:

```typescript
// useSalary.ts에 추가할 코드
const fetchSalary = async () => {
  try {
    setIsLoading(true);
    const response = await axios.get('/api/salary/monthly', {
      headers: { Authorization: `Bearer ${accessToken}` }
    });
    setMonthlySalary(response.data);
    setIsLoading(false);
  } catch (err) {
    setError(err instanceof Error ? err : new Error('Unknown error'));
    setIsLoading(false);
  }
};
```

## 데이터 모델

### 사용자(User)

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string; // 저장 시 해시 처리 필요
  profileImageUrl?: string;
  role: 'USER' | 'OWNER' | 'EMPLOYEE';
  createdAt: Date;
  updatedAt: Date;
}
```

### 매장(Workplace)

```typescript
interface Workplace {
  id: string;
  name: string;
  address: string;
  businessNumber?: string;
  ownerName?: string;
  contactNumber?: string;
  ownerId: string; // 사장 사용자 ID
  employees?: string[]; // 직원 사용자 ID 배열
  createdAt: Date;
  updatedAt: Date;
}
```

### 출퇴근 기록(Attendance)

```typescript
interface Attendance {
  id: string;
  userId: string;
  workplaceId: string;
  date: string; // YYYY-MM-DD 형식
  checkInTime: string | null; // HH:MM 형식
  checkOutTime: string | null; // HH:MM 형식
  workingHours: number | null; // 근무 시간(분)
  createdAt: Date;
  updatedAt: Date;
}
```

### 노무 정보(LaborInfo)

```typescript
interface LaborInfo {
  id: number;
  title: string;
  date: string;
  content: string;
  author: string;
  views: number;
  category: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### 북마크(Bookmark)

```typescript
interface Bookmark {
  id: string;
  userId: string;
  laborInfoId: number;
  createdAt: Date;
}
```

### 급여 정보(Salary)

```typescript
interface Salary {
  id: string;
  userId: string;
  workplaceId: string;
  year: number;
  month: number;
  amount: number;
  workingHours: number;
  deductions: number;
  createdAt: Date;
  updatedAt: Date;
}
```

## 에러 처리

### 에러 응답 형식

```json
{
  "status": 400,
  "message": "Invalid request",
  "errors": [
    {
      "field": "email",
      "message": "Email is required"
    }
  ]
}
```

### HTTP 상태 코드

- 200: 성공
- 201: 리소스 생성 성공
- 400: 잘못된 요청
- 401: 인증 실패
- 403: 권한 없음
- 404: 리소스 없음
- 500: 서버 오류

### 프론트엔드 에러 처리 예시

```typescript
try {
  const response = await axios.get('/api/resource');
  // 성공 처리
} catch (error) {
  if (axios.isAxiosError(error)) {
    if (error.response) {
      // 서버에서 응답이 왔지만 에러 상태 코드인 경우
      const status = error.response.status;
      const errorData = error.response.data;

      if (status === 401) {
        // 인증 실패 처리 (로그인 페이지로 리다이렉트 등)
      } else if (status === 403) {
        // 권한 없음 처리
      } else {
        // 기타 에러 처리
      }
    } else if (error.request) {
      // 요청은 보냈지만 응답이 없는 경우 (네트워크 오류 등)
    }
  } else {
    // 일반 에러 처리
  }
}
```

## 개발 환경 설정

### 백엔드 기술 스택 권장사항

- 언어: Node.js + TypeScript 또는 Java + Spring Boot
- 데이터베이스: PostgreSQL 또는 MySQL
- ORM: TypeORM(Node.js) 또는 JPA(Spring Boot)
- 인증: JWT(jsonwebtoken)
- API 문서화: Swagger/OpenAPI

### 로컬 개발 환경 설정

1. 데이터베이스 설치 및 설정
2. 백엔드 프로젝트 생성
3. 환경 변수 설정 (.env 파일)
   ```
   PORT=8080
   DATABASE_URL=postgresql://username:password@localhost:5432/sodam
   JWT_SECRET=your_jwt_secret_key
   JWT_EXPIRES_IN=1h
   REFRESH_TOKEN_EXPIRES_IN=14d
   ```
4. 필요한 패키지 설치
5. 데이터베이스 마이그레이션 실행
6. 개발 서버 실행

### API 테스트

- Postman 또는 Insomnia를 사용하여 API 테스트
- 자동화된 테스트 작성 (Jest, JUnit 등)
