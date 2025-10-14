# JWT 인증 API 및 RN 연동 가이드

## 📋 개요

- 작성일: 2025-09-11
- 작성자: 백엔드팀 (소담)
- 문서 유형: 가이드/구현
- 관련 이슈: RN팀 요청 — 인증 기능 및 화면/컴포넌트/훅 연동 전반 문서화
- 대상 독자: RN 개발팀, 웹 프론트엔드, 백엔드 통합 담당자

## 🎯 목적

본 문서는 RN(React Native) 클라이언트에서 소담(SODAM) 백엔드 인증 기능을 구현하는 데 필요한 모든 정보를 제공합니다.

- 인증 아키텍처(Access/Refresh/JWT, Redis 블랙리스트)
- 환경 변수 및 보안 설정
- API 계약(요청/응답, 상태코드, 에러 코드)
- 화면/흐름과 API 매핑
- RN 연동 예시(axios 인터셉터, AuthProvider, 훅, 컴포넌트 스니펫)
- 테스트 방법(cURL/Postman) 및 문제 해결 가이드

## 🧭 범위 및 전제

- 백엔드 스택: Spring Boot 3.4.x, Spring Security, JWT, Redis, MySQL, MyBatis/JPA 혼용 가능
- 인증 전략: Stateless Access Token + Refresh Token, 로그아웃/강제만료 시 블랙리스트 관리(Redis)
- 역할/권한: ROLE_USER, ROLE_EMPLOYEE, ROLE_MASTER, ROLE_MANAGER
- 관련 코드 참고:
    - SecurityConfig: `src/main/java/com/rich/sodam/config/SecurityConfig.java`
    - LoginController: `src/main/java/com/rich/sodam/controller/LoginController.java`
    - JwtTokenProvider/Filter: `src/main/java/com/rich/sodam/security/*` (구현 위치는 리팩토링에 따라 상이할 수 있음)
    - AUTH 기능 흐름: `docs/technical/AUTH_기능_데이터_처리_절차_문서.md`

---

## 1. 인증 아키텍처 개요

### 1.1 토큰 모델

- Access Token
    - 용도: 요청 인증, 유효시간 짧게(예: 1시간)
    - 전달: Authorization 헤더 `Bearer <accessToken>`
- Refresh Token
    - 용도: Access 재발급(예: 7일)
    - 저장: httpOnly 쿠키 또는 안전 저장소(모바일은 EncryptedStorage 권장)
- 블랙리스트(Redis)
    - 용도: 로그아웃/탈취 의심 토큰 즉시 무효화
    - 키: `blacklist:{jti}` 또는 `blacklist:{accessToken}` TTL = 남은 만료 시간

### 1.2 기본 흐름

1) 회원가입 → 2) 로그인(Access, Refresh 발급) → 3) 보호 API 호출(Access 사용) → 4) 만료 시 Refresh로 재발급 → 5) 로그아웃(블랙리스트 등록)

---

## 2. 환경 설정 및 보안

### 2.1 application.yml 예시

```yaml
spring:
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:local}
  application:
    name: sodam
  datasource:
    url: ${DB_URL:jdbc:mysql://localhost:3306/sodam}
    username: ${DB_USERNAME:sodam}
    password: ${DB_PASSWORD:#{null}}
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: ${DDL_AUTO:validate}
    show-sql: ${SHOW_SQL:false}
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQL8Dialect
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:#{null}}
      timeout: 2000ms

jwt:
  secret: ${JWT_SECRET:#{null}}
  expiration: ${JWT_EXPIRATION:3600000}        # Access 1h
  refresh-expiration: ${JWT_REFRESH_EXPIRATION:604800000} # Refresh 7d

logging:
  level:
    com.rich.sodam: ${LOG_LEVEL:INFO}
    org.springframework.security: ${SECURITY_LOG_LEVEL:WARN}
```

### 2.2 CORS/CSRF/보안 헤더

- CORS: RN 앱 스킴/도메인 허용, Credential 사용 시 `allowCredentials` 활성화
- CSRF: JWT 기반 stateless이므로 비활성화, 단 서버-서버 통신 제외
- 보안 헤더: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, HSTS, CSP 적용 권장

### 2.3 Rate Limiting(선택)

- IP/사용자 단위 분당 60회 제한 예시(필요 시 `RateLimitingFilter` 참고)

---

## 3. 데이터 모델(요약)

### 3.1 Users

- user_id, email(uniq), name, password(BCrypt), user_grade, created_at

### 3.2 Refresh Tokens

- id, user_id(FK), token, expires_at, created_at
- 운영 정책에 따라 DB 또는 Redis에 저장 가능

### 3.3 블랙리스트(Redis)

- key: `blacklist:{jti}` value: true, TTL: access 토큰 잔여 시간

자세한 스키마: `docs/technical/AUTH_기능_데이터_처리_절차_문서.md` 참고

---

## 4. API 계약

### 4.1 회원가입 POST /api/join

- 요청(JSON)

```json
{
  "email": "user@example.com",
  "password": "P@ssw0rd!",
  "name": "홍길동"
}
```

- 응답(201)

```json
{ "userId": 1, "email": "user@example.com", "name": "홍길동", "grade": "ROLE_USER" }
```

- 오류
    - 400 INVALID_INPUT
    - 409 EMAIL_ALREADY_EXISTS

### 4.2 로그인 POST /api/login

- 요청(JSON)

```json
{ "email": "user@example.com", "password": "P@ssw0rd!" }
```

- 응답(200)

```json
{
  "accessToken": "<JWT>",
  "refreshToken": "<JWT>",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "user": { "id": 1, "email": "user@example.com", "name": "홍길동", "roles": ["ROLE_USER"] }
}
```

- 오류
    - 401 INVALID_CREDENTIALS
    - 423 ACCOUNT_LOCKED

### 4.3 토큰 갱신 POST /api/auth/refresh

- 요청(JSON)

```json
{ "refreshToken": "<refresh-jwt>" }
```

- 응답(200)

```json
{ "accessToken": "<JWT>", "expiresIn": 3600, "tokenType": "Bearer" }
```

- 오류
    - 401 INVALID_REFRESH_TOKEN
    - 401 REFRESH_TOKEN_EXPIRED

### 4.4 로그아웃 POST /api/auth/logout

- 헤더: Authorization: Bearer <access>
- 요청(JSON)

```json
{ "refreshToken": "<refresh-jwt>" }
```

- 처리: 현재 Access의 jti 블랙리스트 등록, Refresh 폐기(DB/Redis 삭제)
- 응답(204) 바디 없음

### 4.5 내 정보 GET /api/auth/me

- 헤더: Authorization: Bearer <access>
- 응답(200)

```json
{ "id": 1, "email": "user@example.com", "name": "홍길동", "roles": ["ROLE_USER"] }
```

### 4.6 비밀번호 변경 POST /api/auth/password/change

- 헤더: Authorization: Bearer <access>
- 요청(JSON)

```json
{ "currentPassword": "OldP@ss", "newPassword": "NewP@ss!" }
```

- 응답(200) `{ "success": true }`
- 오류: 400 WEAK_PASSWORD, 401 INVALID_CREDENTIALS

### 4.7 권한 확인 예시 GET /api/user/profile

- 권한: ROLE_USER 이상
- 200 정상 응답, 403 FORBIDDEN(권한 없음), 401 UNAUTHORIZED(미인증)

참고: 실제 엔드포인트는 `LoginController` 및 관련 컨트롤러 구현을 기준으로 확정하세요.

---

## 5. 에러 모델 및 상태 코드

### 5.1 오류 응답 형태

```json
{
  "timestamp": "2025-09-11T07:10:00Z",
  "status": 401,
  "error": "UNAUTHORIZED",
  "code": "INVALID_CREDENTIALS",
  "message": "이메일 또는 비밀번호가 올바르지 않습니다.",
  "path": "/api/auth/login"
}
```

### 5.2 에러 코드 표(발췌)

- INVALID_INPUT(400), EMAIL_ALREADY_EXISTS(409)
- INVALID_CREDENTIALS(401), ACCOUNT_LOCKED(423)
- INVALID_REFRESH_TOKEN(401), REFRESH_TOKEN_EXPIRED(401)
- TOKEN_BLACKLISTED(401), TOKEN_EXPIRED(401)
- FORBIDDEN(403), NOT_FOUND(404), TOO_MANY_REQUESTS(429)

---

## 6. RN 연동 가이드

### 6.1 패키지

- axios, react-native-encrypted-storage(권장), @react-navigation/native

### 6.2 axios 인스턴스 및 인터셉터

```typescript
// apiClient.ts
import axios from 'axios';
import EncryptedStorage from 'react-native-encrypted-storage';

export const api = axios.create({
  baseURL: 'https://api.sodam.app', // .env로 분리
  timeout: 10000,
});

api.interceptors.request.use(async (config) => {
  const accessToken = await EncryptedStorage.getItem('accessToken');
  if (accessToken) {
    config.headers = config.headers || {};
    config.headers.Authorization = `Bearer ${accessToken}`;
  }
  return config;
});

let isRefreshing = false;
let queue: Array<(token: string | null) => void> = [];

api.interceptors.response.use(
  (res) => res,
  async (error) => {
    const original = error.config;
    if (error.response?.status === 401 && !original._retry) {
      original._retry = true;
      if (!isRefreshing) {
        isRefreshing = true;
        try {
          const refreshToken = await EncryptedStorage.getItem('refreshToken');
          if (!refreshToken) throw new Error('NO_REFRESH');
          const { data } = await axios.post(`${api.defaults.baseURL}/api/auth/refresh`, { refreshToken });
          await EncryptedStorage.setItem('accessToken', data.accessToken);
          queue.forEach((cb) => cb(data.accessToken));
          queue = [];
          return api(original);
        } catch (e) {
          queue.forEach((cb) => cb(null));
          queue = [];
          await EncryptedStorage.removeItem('accessToken');
          await EncryptedStorage.removeItem('refreshToken');
          // TODO: 네비게이션으로 로그인 화면 이동
          return Promise.reject(e);
        } finally {
          isRefreshing = false;
        }
      }
      return new Promise((resolve, reject) => {
        queue.push((token) => {
          if (!token) return reject(error);
          original.headers.Authorization = `Bearer ${token}`;
          resolve(api(original));
        });
      });
    }
    return Promise.reject(error);
  }
);
```

### 6.3 Auth Context/Provider 및 훅

```typescript
// AuthProvider.tsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import EncryptedStorage from 'react-native-encrypted-storage';
import { api } from './apiClient';

interface User { id: number; email: string; name: string; roles: string[] }
interface AuthContextValue {
  user: User | null;
  loading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextValue | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    (async () => {
      try {
        const token = await EncryptedStorage.getItem('accessToken');
        if (token) {
          const { data } = await api.get('/api/auth/me');
          setUser(data);
        }
      } catch (_) {} finally { setLoading(false); }
    })();
  }, []);

  const login = async (email: string, password: string) => {
    const { data } = await api.post('/api/auth/login', { email, password });
    await EncryptedStorage.setItem('accessToken', data.accessToken);
    await EncryptedStorage.setItem('refreshToken', data.refreshToken);
    setUser(data.user);
  };

  const logout = async () => {
    try {
      const refreshToken = await EncryptedStorage.getItem('refreshToken');
      await api.post('/api/auth/logout', { refreshToken });
    } catch (_) { /* 네트워크 오류시 로컬만 정리 */ }
    await EncryptedStorage.removeItem('accessToken');
    await EncryptedStorage.removeItem('refreshToken');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
};
```

### 6.4 보호 화면 가드 컴포넌트

```typescript
// Protected.tsx
import React from 'react';
import { View, ActivityIndicator } from 'react-native';
import { useAuth } from './AuthProvider';

export const Protected: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user, loading } = useAuth();
  if (loading) return <ActivityIndicator />;
  return <View>{user ? children : null /* 네비게이션으로 로그인 이동 */}</View>;
};
```

### 6.5 화면 ↔ API 매핑

- 로그인 화면(LoginScreen)
    - 입력: email, password → POST /api/auth/login
    - 성공 시: access/refresh 저장, 홈으로 이동, /api/auth/me로 프로필 프리페치
- 회원가입 화면(RegisterScreen)
    - 입력: email, password, name → POST /api/auth/register
- 내 정보 화면(ProfileScreen)
    - GET /api/auth/me
- 설정/로그아웃
    - POST /api/auth/logout(실패해도 로컬 토큰 제거)

---

## 7. 테스트 가이드

### 7.1 cURL 예시

```bash
# 회원가입
curl -X POST https://api.sodam.app/api/auth/register \
 -H "Content-Type: application/json" \
 -d '{"email":"user@example.com","password":"P@ssw0rd!","name":"홍길동"}'

# 로그인
curl -X POST https://api.sodam.app/api/auth/login \
 -H "Content-Type: application/json" \
 -d '{"email":"user@example.com","password":"P@ssw0rd!"}'

# 내 정보
curl -H "Authorization: Bearer <access>" https://api.sodam.app/api/auth/me

# 토큰 갱신
curl -X POST https://api.sodam.app/api/auth/refresh \
 -H "Content-Type: application/json" \
 -d '{"refreshToken":"<refresh>"}'

# 로그아웃
curl -X POST https://api.sodam.app/api/auth/logout \
 -H "Authorization: Bearer <access>" \
 -H "Content-Type: application/json" \
 -d '{"refreshToken":"<refresh>"}'
```

### 7.2 Postman 컬렉션 팁

- 환경변수: `baseUrl`, `accessToken`, `refreshToken`
- Pre-request Script로 Authorization 헤더 자동 주입

---

## 8. 보안 베스트 프랙티스(클라이언트)

- 토큰 저장: EncryptedStorage 권장, 디버그 빌드에서도 로그로 토큰 출력 금지
- 네트워크: HTTPS 강제, 인증서 고정(Pinning) 검토
- 입력 검증: 이메일/비밀번호 클라이언트 유효성 검증 + 서버 검증 병행
- 오류 메시지: 과도한 정보 노출 금지(존재하는 이메일인지 노출하지 않음)

---

## 9. 문제 해결(FAQ)

- 401이 반복 발생: Access 만료 → 인터셉터 갱신 실패 여부 확인, Refresh 유효성 점검
- 403 발생: 역할/권한 매핑 확인(SecurityConfig의 authorizeHttpRequests 규칙)
- 429 발생: 레이트 리밋 초과 → 1분 후 재시도, 백엔드 필터 설정 확인
- CORS 오류: RN 개발 서버/앱 패키지 스킴이 백엔드 CORS에 등록되었는지 확인

---

## 10. 추후 추가사항(미개발)

- 권한(USER,EMPLOYEE,MASTER,MANAGER)별 접근가능 페이지 및 에러메시지 개발예정

---

## 🔗 관련 문서

- AUTH 데이터 처리 절차: `docs/technical/AUTH_기능_데이터_처리_절차_문서.md`
- 백엔드 보안/설정 가이드: `docs/guidelines/guidelines.md` 및 사업계획서 내 기술 아키텍처 섹션
- SecurityConfig, LoginController: 소스 경로 참고

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자  
------------|-----|-------|------
 2025-09-11 | 1.0 | 최초 작성 | 백엔드팀 
