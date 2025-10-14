# 📘 소담(SODAM) API 문서 - Quick Reference

> **초급 개발자를 위한 빠른 시작 가이드**

---

## 🚀 빠른 시작 (5분 안에 API 연동하기)

### 1단계: API 명세서 확인

```bash
# 프로젝트 루트의 API 명세서 파일
📄 ApiList.yaml  # ← OpenAPI 3.0 형식의 전체 API 명세서
```

### 2단계: 상세 가이드 읽기

```bash
# 상세한 연동 가이드 문서
📄 docs/technical/api/API_연동_가이드.md  # ← 단계별 구현 가이드
```

### 3단계: 로그인 구현

```javascript
// 예제 코드 - 바로 복사해서 사용 가능
import axios from 'axios';

const login = async (email, password) => {
  const response = await axios.post('https://dev-api.sodam.com/api/auth/login', {
    email: email,
    password: password
  });
  
  const { accessToken, refreshToken } = response.data;
  // 토큰 저장 후 사용
  return { accessToken, refreshToken };
};
```

---

## 📋 API 명세서 파일 설명

### ApiList.yaml 파일이란?

- **형식**: OpenAPI 3.0.1 표준
- **용도**: 모든 API 엔드포인트의 상세 스펙
- **크기**: 6900+ 줄 (100개+ API)
- **언어**: 한국어 (초급 개발자 친화적)

### 파일 구조

```yaml
# ApiList.yaml 구조
openapi: 3.0.1
info:               # API 기본 정보
servers:            # 환경별 서버 URL (로컬/개발/운영)
security:           # 인증 방식 (JWT Bearer Token)
tags:               # API 카테고리 (14개)
paths:              # API 엔드포인트 목록 (100개+)
  /api/auth/login:  # 각 API의 상세 스펙
    post:
      summary: ...
      description: ...
      parameters: ...
      responses: ...
components:         # 공통 스키마 및 보안 정의
```

---

## 🎯 API 카테고리별 주요 엔드포인트

### 1. 인증 (Authentication)

| 메서드  | 엔드포인트               | 설명           | 권한   |
|------|---------------------|--------------|------|
| POST | `/api/auth/login`   | 이메일/비밀번호 로그인 | 없음   |
| POST | `/api/auth/kakao`   | 카카오 소셜 로그인   | 없음   |
| POST | `/api/auth/refresh` | 토큰 갱신        | 없음   |
| GET  | `/api/auth/me`      | 내 정보 조회      | USER |

### 2. 출퇴근 관리 (Attendance)

| 메서드  | 엔드포인트                             | 설명          | 권한   |
|------|-----------------------------------|-------------|------|
| POST | `/api/attendance/check-in`        | 출근 등록 (NFC) | USER |
| POST | `/api/attendance/check-out`       | 퇴근 등록       | USER |
| GET  | `/api/attendance/employee/{id}`   | 출퇴근 기록 조회   | USER |
| POST | `/api/attendance/verify/nfc`      | NFC 태그 검증   | USER |
| POST | `/api/attendance/verify/location` | 위치 검증       | USER |

### 3. 급여 관리 (Payroll)

| 메서드  | 엔드포인트                             | 설명          | 권한    |
|------|-----------------------------------|-------------|-------|
| POST | `/api/payroll/calculate`          | 급여 자동 계산    | ADMIN |
| GET  | `/api/payroll/{payrollId}`        | 급여 명세서 조회   | USER  |
| PUT  | `/api/payroll/{payrollId}/status` | 급여 지급 상태 변경 | ADMIN |

### 4. 사용자 관리 (User)

| 메서드  | 엔드포인트                                 | 설명        | 권한         |
|------|---------------------------------------|-----------|------------|
| GET  | `/api/user/{userId}`                  | 사용자 정보 조회 | USER       |
| PUT  | `/api/user/{userId}`                  | 사용자 정보 수정 | USER/ADMIN |
| POST | `/api/users/{userId}/purpose`         | 사용자 목적 설정 | USER       |
| PUT  | `/api/user/{userId}/convert-to-owner` | 직원→사업주 전환 | ADMIN      |

### 5. 매장 관리 (Store)

| 메서드  | 엔드포인트                             | 설명         | 권한    |
|------|-----------------------------------|------------|-------|
| POST | `/api/stores/registration`        | 매장 등록      | ADMIN |
| GET  | `/api/stores/{storeId}`           | 매장 정보 조회   | ADMIN |
| PUT  | `/api/stores/{storeId}`           | 매장 정보 수정   | ADMIN |
| GET  | `/api/stores/{storeId}/employees` | 매장 직원 목록   | ADMIN |
| PUT  | `/api/stores/{storeId}/location`  | 매장 위치 업데이트 | ADMIN |

### 6. 임금 관리 (Wages)

| 메서드  | 엔드포인트                                              | 설명          | 권한    |
|------|----------------------------------------------------|-------------|-------|
| PUT  | `/api/wages/store/{storeId}/standard`              | 매장 기본 시급 설정 | ADMIN |
| GET  | `/api/wages/employee/{employeeId}/store/{storeId}` | 직원 시급 조회    | ADMIN |
| POST | `/api/wages/employee`                              | 직원별 시급 설정   | ADMIN |

### 7. 통계 및 리포트 (Statistics)

| 메서드 | 엔드포인트                                                       | 설명      | 권한    |
|-----|-------------------------------------------------------------|---------|-------|
| GET | `/api/store-queries/{storeId}/stats/attendance/count`       | 출근 통계   | ADMIN |
| GET | `/api/store-queries/{storeId}/stats/payroll/count`          | 급여 통계   | ADMIN |
| GET | `/api/store-queries/{storeId}/stats/employees/active/count` | 활성 직원 수 | ADMIN |

---

## 🔐 인증 방식 (중요!)

### JWT Bearer Token 인증

모든 API 요청 시 다음 헤더 포함 필수:

```javascript
headers: {
  'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
}
```

### 토큰 유효기간

- **Access Token**: 1시간
- **Refresh Token**: 7일

### 인증 플로우

```
1. 로그인 (/api/auth/login 또는 /api/auth/kakao)
   ↓
2. Access Token + Refresh Token 획득
   ↓
3. Access Token을 Authorization 헤더에 포함하여 API 호출
   ↓
4. Access Token 만료 시 → Refresh Token으로 갱신 (/api/auth/refresh)
   ↓
5. 새로운 Access Token 획득 → 다시 3번으로
```

---

## 🌐 환경별 서버 URL

| 환경 | Base URL                    | 용도           |
|----|-----------------------------|--------------|
| 로컬 | `http://localhost:8080`     | 로컬 개발/테스트    |
| 개발 | `https://dev-api.sodam.com` | FE/BE 통합 테스트 |
| 운영 | `https://sodam-api.com`     | 실제 서비스       |

---

## 📚 각 API의 상세 정보 확인 방법

### 방법 1: ApiList.yaml 파일 직접 열기

```yaml
# ApiList.yaml에서 원하는 API 검색 (Ctrl+F)
paths:
  /api/attendance/check-in:
    post:
      summary: 출근 등록
      description: |
        **API 목적**: NFC 태그를 통한 출근 기록
        **비즈니스 로직**: ...
        **사용 시나리오**: ...
        **주의사항**: ...
```

### 방법 2: Swagger UI 사용 (추천)

```bash
# 1. 백엔드 서버 실행
cd backend
./gradlew bootRun

# 2. 브라우저에서 Swagger UI 열기
http://localhost:8080/swagger-ui/index.html

# 3. API 테스트 가능!
```

### 방법 3: 상세 가이드 문서 참조

```bash
# docs/technical/api/API_연동_가이드.md 파일 열기
- 인증 방법 상세 설명
- 주요 API 사용 예시 코드
- 에러 처리 방법
- Axios Interceptor 설정 등
```

---

## 💡 API 사용 시 주의사항

### ✅ DO (권장사항)

- JWT 토큰을 AsyncStorage에 안전하게 저장
- Axios Interceptor를 사용한 자동 토큰 포함
- 401 에러 발생 시 자동 토큰 갱신 로직 구현
- 에러 발생 시 사용자 친화적인 메시지 표시
- API 요청 전 필수 파라미터 검증

### ❌ DON'T (주의사항)

- 토큰을 일반 변수에 저장 (앱 재시작 시 손실)
- 모든 API에 하드코딩된 토큰 사용
- 에러 메시지를 그대로 사용자에게 노출
- 운영 서버에서 테스트 (개발 서버 사용)
- 대용량 데이터를 한 번에 요청 (페이징 사용)

---

## 🐛 자주 발생하는 에러 및 해결 방법

### 401 Unauthorized

```
원인: 토큰이 없거나 만료됨
해결: 토큰 갱신 (/api/auth/refresh) 또는 재로그인
```

### 403 Forbidden

```
원인: 권한 없음 (예: 일반 직원이 관리자 API 호출)
해결: 권한 확인 및 적절한 사용자에게 제한
```

### 404 Not Found

```
원인: 존재하지 않는 리소스 (잘못된 ID 등)
해결: 요청 파라미터 확인
```

### 500 Internal Server Error

```
원인: 서버 내부 오류
해결: 백엔드 팀에 문의 (로그 첨부)
```

---

## 📞 API 관련 문의

### 문서 위치

- **API 명세서**: `ApiList.yaml` (프로젝트 루트)
- **연동 가이드**: `docs/technical/api/API_연동_가이드.md`
- **프로젝트 기획서**: `docs/project-management/project_planning_Document_v2.md`

### 문의 채널

- **슬랙**: #api-support 채널
- **이슈 트래커**: GitHub Issues
- **긴급 문의**: 백엔드 팀 리더에게 직접 연락

---

## 📖 학습 순서 (초급 개발자 추천)

### 1단계: 기본 개념 이해 (30분)

- [ ] API란 무엇인가?
- [ ] REST API 기본 개념
- [ ] HTTP 메서드 (GET, POST, PUT, DELETE)
- [ ] HTTP 상태 코드 의미

### 2단계: 인증 구현 (1시간)

- [ ] JWT 토큰 개념 이해
- [ ] 로그인 API 호출
- [ ] 토큰 저장 및 관리
- [ ] Authorization 헤더 설정

### 3단계: 주요 API 연동 (2시간)

- [ ] 출근/퇴근 API 구현
- [ ] 출퇴근 기록 조회
- [ ] 급여 명세서 조회
- [ ] 내 정보 조회

### 4단계: 고급 기능 (1시간)

- [ ] Axios Interceptor 설정
- [ ] 자동 토큰 갱신
- [ ] 에러 처리 통합
- [ ] 로딩 상태 관리

---

## 🎓 추가 학습 자료

### React Native에서 API 호출

```javascript
// 1. Axios 설치
npm install axios

// 2. AsyncStorage 설치
npm install @react-native-async-storage/async-storage

// 3. API 클라이언트 설정
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://dev-api.sodam.com',
  timeout: 10000,
});

// 4. API 호출
const response = await apiClient.get('/api/auth/me');
```

### 참고 링크

- [Axios 공식 문서](https://axios-http.com/docs/intro)
- [React Native 공식 문서](https://reactnative.dev/docs/network)
- [JWT 개념 설명](https://jwt.io/introduction)

---

## ✨ API 명세서의 특징

### 왜 YAML 형식인가?

- ✅ **가독성**: JSON보다 읽기 쉬움
- ✅ **주석 가능**: 각 API에 대한 상세 설명 포함
- ✅ **표준**: OpenAPI 3.0 표준 준수
- ✅ **도구 지원**: Swagger, Postman 등에서 직접 import 가능

### ApiList.yaml의 강점

1. **초급 개발자 친화적**: 모든 설명이 한국어로 작성됨
2. **상세한 설명**: 각 API의 목적, 비즈니스 로직, 사용 시나리오 포함
3. **예시 코드**: 요청/응답 예시 제공
4. **에러 케이스**: 발생 가능한 모든 에러 코드와 의미 설명
5. **권한 정보**: 각 API의 필요 권한 명시

---

## 🚦 시작하기

### 지금 바로 할 수 있는 것

1. ✅ `ApiList.yaml` 파일 열어보기
2. ✅ Swagger UI로 API 테스트해보기
3. ✅ 로그인 API 먼저 구현하기
4. ✅ 상세 가이드 읽으면서 단계별 구현

### 다음 단계

```bash
# 1. ApiList.yaml 파일 열기
code ApiList.yaml

# 2. 상세 가이드 문서 읽기
code docs/technical/api/API_연동_가이드.md

# 3. 예제 코드 복사해서 프로젝트에 적용

# 4. API 호출 테스트

# 5. 문제 발생 시 문의
```

---

**작성일**: 2025-10-14  
**버전**: 1.0.0  
**문서 관리**: 소담 백엔드팀

**💬 이 문서에 대한 피드백이나 개선 제안은 언제든 환영합니다!**
