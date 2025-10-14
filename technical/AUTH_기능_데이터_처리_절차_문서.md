# AUTH 기능 데이터 처리 절차 문서

## 📋 개요

- **작성일**: 2025-07-26
- **작성자**: 소담 개발팀
- **문서 유형**: 기술 문서
- **관련 이슈**: AUTH-008 사업주 전환 기능 완성 및 AUTH 기능 데이터 처리 절차 문서화

## 🎯 목적

본 문서는 소담(SODAM) 애플리케이션의 인증 및 계정 관리(AUTH) 기능들에 대한 데이터 처리 절차를 상세하게 기술합니다. 각 기능별 데이터 흐름, 데이터베이스 스키마, API 엔드포인트, 비즈니스 로직을
포함하여 개발자가 AUTH 기능을 이해하고 유지보수할 수 있도록 합니다.

## 📑 목차

1. [AUTH 기능 개요](#1-auth-기능-개요)
2. [데이터베이스 스키마](#2-데이터베이스-스키마)
3. [기능별 데이터 처리 절차](#3-기능별-데이터-처리-절차)
4. [보안 및 인증 처리](#4-보안-및-인증-처리)
5. [에러 처리 및 예외 상황](#5-에러-처리-및-예외-상황)
6. [성능 최적화 및 캐싱](#6-성능-최적화-및-캐싱)

---

## 1. AUTH 기능 개요

### 1.1 AUTH 기능 목록

| 기능ID     | 기능명         | 상태     | 우선순위   | 관련 클래스                                |
|----------|-------------|--------|--------|---------------------------------------|
| AUTH-001 | 일반 회원가입     | ✅ 완료   | High   | LoginController, UserService, JoinDto |
| AUTH-002 | 일반 로그인      | ✅ 완료   | High   | LoginController, UserService, Login   |
| AUTH-003 | 카카오 소셜 로그인  | ✅ 완료   | High   | LoginController, KakaoAuthService     |
| AUTH-004 | 자동 로그인      | ✅ 완료   | Medium | JwtTokenProvider, TokenService        |
| AUTH-005 | 토큰 갱신       | ✅ 완료   | High   | LoginController, RefreshTokenService  |
| AUTH-006 | 비밀번호 찾기     | 🔄 개발중 | Medium | UserService                           |
| AUTH-007 | 역할 기반 권한 관리 | ✅ 완료   | High   | SecurityConfig, UserService           |
| AUTH-008 | 사업주 전환      | ✅ 완료   | High   | UserService                           |
| AUTH-009 | 직원 등록       | ✅ 완료   | High   | UserService                           |
| AUTH-010 | 권한 위임       | 📋 계획  | Medium | UserService                           |
| AUTH-011 | 회원 탈퇴       | 📋 계획  | Low    | UserService                           |

### 1.2 사용자 등급 체계

```java
public enum UserGrade {
    NORMAL("ROLE_USER"),      // 일반 사용자
    EMPLOYEE("ROLE_EMPLOYEE"), // 직원
    MASTER("ROLE_MASTER"),     // 사업주
    MANAGER("ROLE_MANAGER");   // 관리자
}
```

---

## 2. 데이터베이스 스키마

### 2.1 Users 테이블

```sql
CREATE TABLE users (
    user_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) NOT NULL UNIQUE,
    name VARCHAR(50) NOT NULL,
    password VARCHAR(255),
    user_grade VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_user_email (email),
    INDEX idx_user_grade (user_grade),
    INDEX idx_user_created_at (created_at)
);
```

### 2.2 RefreshToken 테이블

```sql
CREATE TABLE refresh_tokens (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    token VARCHAR(500) NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX idx_refresh_token (token),
    INDEX idx_refresh_expires (expires_at)
);
```

### 2.3 MasterProfile 테이블

```sql
CREATE TABLE master_profiles (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    business_license_number VARCHAR(50),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    UNIQUE KEY uk_master_user (user_id)
);
```

---

## 3. 기능별 데이터 처리 절차

### 3.1 AUTH-001: 일반 회원가입

#### 3.1.1 데이터 흐름

```
클라이언트 → LoginController.join() → UserService.joinUser() → UserRepository.save() → 데이터베이스
```

#### 3.1.2 처리 절차

1. **입력 데이터 검증**
    - 이메일 형식 검증
    - 비밀번호 강도 검증
    - 이름 길이 검증

2. **중복 검사**
   ```java
   Optional<User> existingUser = userRepository.findByEmail(joinDto.getEmail());
   if (existingUser.isPresent()) {
       throw new IllegalArgumentException("이미 존재하는 이메일입니다.");
   }
   ```

3. **비밀번호 암호화**
   ```java
   String encodedPassword = passwordEncoder.encode(joinDto.getPassword());
   ```

4. **사용자 엔티티 생성 및 저장**
   ```java
   User user = new User();
   user.setEmail(joinDto.getEmail());
   user.setName(joinDto.getName());
   user.setPassword(encodedPassword);
   user.setUserGrade(UserGrade.NORMAL);
   user.setCreatedAt(LocalDateTime.now());
   return userRepository.save(user);
   ```

#### 3.1.3 API 엔드포인트

- **URL**: `POST /api/auth/join`
- **요청 구조**:
  ```json
  {
    "email": "user@example.com",
    "name": "홍길동",
    "password": "securePassword123"
  }
  ```
- **응답 구조**:
  ```json
  {
    "success": true,
    "message": "회원가입이 완료되었습니다.",
    "data": {
      "userId": 1,
      "email": "user@example.com",
      "name": "홍길동",
      "userGrade": "NORMAL"
    }
  }
  ```

### 3.2 AUTH-002: 일반 로그인

#### 3.2.1 데이터 흐름

```
클라이언트 → LoginController.login() → UserService.loadUserByLoginId() → JWT 토큰 생성 → 응답
```

#### 3.2.2 처리 절차

1. **사용자 인증**
   ```java
   Optional<User> user = userRepository.findByEmail(email);
   if (user.isPresent() && passwordEncoder.matches(password, user.get().getPassword())) {
       return user;
   }
   ```

2. **JWT 토큰 생성**
   ```java
   String accessToken = jwtTokenProvider.createAccessToken(user.getId(), user.getUserGrade());
   String refreshToken = jwtTokenProvider.createRefreshToken(user.getId());
   ```

3. **Refresh Token 저장**
   ```java
   RefreshToken refreshTokenEntity = new RefreshToken(user.getId(), refreshToken, expiresAt);
   refreshTokenRepository.save(refreshTokenEntity);
   ```

#### 3.2.3 API 엔드포인트

- **URL**: `POST /api/auth/login`
- **요청 구조**:
  ```json
  {
    "email": "user@example.com",
    "password": "securePassword123"
  }
  ```
- **응답 구조**:
  ```json
  {
    "success": true,
    "message": "로그인 성공",
    "data": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "user": {
        "userId": 1,
        "email": "user@example.com",
        "name": "홍길동",
        "userGrade": "NORMAL"
      }
    }
  }
  ```

### 3.3 AUTH-005: 토큰 갱신

#### 3.3.1 데이터 흐름

```
클라이언트 → LoginController.refreshToken() → RefreshTokenService.validateAndRefresh() → 새 토큰 생성
```

#### 3.3.2 처리 절차

1. **Refresh Token 검증**
   ```java
   Optional<RefreshToken> refreshTokenEntity = refreshTokenRepository.findByToken(refreshToken);
   if (refreshTokenEntity.isEmpty() || refreshTokenEntity.get().isExpired()) {
       throw new IllegalArgumentException("유효하지 않은 리프레시 토큰입니다.");
   }
   ```

2. **새 Access Token 생성**
   ```java
   User user = userRepository.findById(refreshTokenEntity.get().getUserId())
       .orElseThrow(() -> new IllegalArgumentException("사용자를 찾을 수 없습니다."));
   String newAccessToken = jwtTokenProvider.createAccessToken(user.getId(), user.getUserGrade());
   ```

3. **기존 Refresh Token 갱신 (선택적)**
   ```java
   if (shouldRotateRefreshToken(refreshTokenEntity.get())) {
       String newRefreshToken = jwtTokenProvider.createRefreshToken(user.getId());
       refreshTokenEntity.get().updateToken(newRefreshToken);
       refreshTokenRepository.save(refreshTokenEntity.get());
   }
   ```

### 3.4 AUTH-008: 사업주 전환 ⭐ **신규 완성 기능**

#### 3.4.1 데이터 흐름

```
클라이언트 → MasterController.convertToOwner() → UserService.convertToOwner() → MasterProfile 생성
```

#### 3.4.2 처리 절차

1. **사용자 존재 확인**
   ```java
   User user = userRepository.findById(userId)
       .orElseThrow(() -> new IllegalArgumentException("사용자를 찾을 수 없습니다. ID: " + userId));
   ```

2. **권한 검증**
   ```java
   // 이미 사업주인 경우 예외 처리
   if (user.getUserGrade() == UserGrade.MASTER) {
       throw new IllegalArgumentException("이미 사업주 권한을 가진 사용자입니다.");
   }
   
   // 일반 사용자만 사업주로 전환 가능
   if (user.getUserGrade() != UserGrade.NORMAL) {
       throw new IllegalArgumentException("일반 사용자만 사업주로 전환할 수 있습니다. 현재 등급: " + user.getUserGrade());
   }
   ```

3. **사업주로 전환**
   ```java
   user.changeToMaster(); // UserGrade를 MASTER로 변경
   User updatedUser = userRepository.save(user);
   ```

4. **MasterProfile 생성**
   ```java
   MasterProfile masterProfile = new MasterProfile();
   masterProfile.setUserId(user.getId());
   masterProfile.setCreatedAt(LocalDateTime.now());
   masterProfileRepository.save(masterProfile);
   ```

5. **캐시 무효화**
   ```java
   @CacheEvict(value = "users", allEntries = true)
   ```

#### 3.4.3 API 엔드포인트

- **URL**: `POST /api/auth/convert-to-owner`
- **요청 구조**:
  ```json
  {
    "userId": 1
  }
  ```
- **응답 구조**:
  ```json
  {
    "success": true,
    "message": "사업주로 전환되었습니다.",
    "data": {
      "userId": 1,
      "email": "user@example.com",
      "name": "홍길동",
      "userGrade": "MASTER",
      "masterProfile": {
        "id": 1,
        "businessLicenseNumber": null,
        "createdAt": "2025-07-26T15:30:00"
      }
    }
  }
  ```

#### 3.4.4 비즈니스 규칙

- **전환 가능 조건**: NORMAL 등급 사용자만 가능
- **전환 불가 조건**:
    - 이미 MASTER 등급인 사용자
    - EMPLOYEE, MANAGER 등급 사용자
- **후속 처리**: MasterProfile 자동 생성
- **권한 변경**: ROLE_USER → ROLE_MASTER

### 3.5 AUTH-007: 역할 기반 권한 관리

#### 3.5.1 Spring Security 설정

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("MANAGER")
                .requestMatchers("/api/master/**").hasRole("MASTER")
                .requestMatchers("/api/employee/**").hasRole("EMPLOYEE")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
}
```

#### 3.5.2 권한 검증 흐름

1. **JWT 토큰 추출**: Authorization 헤더에서 Bearer 토큰 추출
2. **토큰 검증**: 서명, 만료시간, 형식 검증
3. **사용자 정보 추출**: 토큰에서 userId, userGrade 추출
4. **Spring Security Context 설정**: Authentication 객체 생성
5. **권한 검사**: @PreAuthorize, URL 패턴 매칭으로 접근 권한 확인

---

## 4. 보안 및 인증 처리

### 4.1 JWT 토큰 구조

#### 4.1.1 Access Token

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "1",
    "userGrade": "MASTER",
    "iat": 1690123456,
    "exp": 1690127056
  }
}
```

#### 4.1.2 Refresh Token

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "1",
    "type": "refresh",
    "iat": 1690123456,
    "exp": 1690727856
  }
}
```

### 4.2 비밀번호 보안

#### 4.2.1 암호화 방식

- **알고리즘**: BCrypt
- **라운드**: 12 (기본값)
- **솔트**: 자동 생성

#### 4.2.2 비밀번호 정책

- **최소 길이**: 8자
- **복잡성**: 영문, 숫자, 특수문자 조합 권장
- **재사용 제한**: 이전 3개 비밀번호 재사용 금지 (향후 구현)

### 4.3 세션 관리

#### 4.3.1 토큰 만료 시간

- **Access Token**: 1시간 (3600초)
- **Refresh Token**: 7일 (604800초)

#### 4.3.2 토큰 갱신 전략

- **자동 갱신**: Access Token 만료 15분 전 자동 갱신
- **Refresh Token 로테이션**: 보안 강화를 위한 토큰 순환

---

## 5. 에러 처리 및 예외 상황

### 5.1 공통 예외 처리

#### 5.1.1 GlobalExceptionHandler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiResponse> handleIllegalArgument(IllegalArgumentException e) {
        return ResponseEntity.badRequest()
            .body(ApiResponse.error(ErrorCode.INVALID_INPUT, e.getMessage()));
    }
    
    @ExceptionHandler(JwtException.class)
    public ResponseEntity<ApiResponse> handleJwtException(JwtException e) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(ApiResponse.error(ErrorCode.INVALID_TOKEN, "유효하지 않은 토큰입니다."));
    }
}
```

### 5.2 AUTH 기능별 예외 상황

#### 5.2.1 회원가입 (AUTH-001)

| 예외 상황      | 예외 타입                    | 에러 코드                | 메시지                     |
|------------|--------------------------|----------------------|-------------------------|
| 중복 이메일     | IllegalArgumentException | DUPLICATE_EMAIL      | 이미 존재하는 이메일입니다.         |
| 잘못된 이메일 형식 | ValidationException      | INVALID_EMAIL_FORMAT | 올바른 이메일 형식이 아닙니다.       |
| 비밀번호 정책 위반 | ValidationException      | WEAK_PASSWORD        | 비밀번호가 보안 정책을 만족하지 않습니다. |

#### 5.2.2 로그인 (AUTH-002)

| 예외 상황       | 예외 타입                    | 에러 코드            | 메시지              |
|-------------|--------------------------|------------------|------------------|
| 존재하지 않는 사용자 | IllegalArgumentException | USER_NOT_FOUND   | 존재하지 않는 사용자입니다.  |
| 비밀번호 불일치    | IllegalArgumentException | INVALID_PASSWORD | 비밀번호가 일치하지 않습니다. |
| 계정 비활성화     | IllegalStateException    | ACCOUNT_DISABLED | 비활성화된 계정입니다.     |

#### 5.2.3 사업주 전환 (AUTH-008)

| 예외 상황  | 예외 타입                    | 에러 코드              | 메시지                      |
|--------|--------------------------|--------------------|--------------------------|
| 사용자 없음 | IllegalArgumentException | USER_NOT_FOUND     | 사용자를 찾을 수 없습니다.          |
| 이미 사업주 | IllegalArgumentException | ALREADY_MASTER     | 이미 사업주 권한을 가진 사용자입니다.    |
| 권한 부족  | IllegalArgumentException | INVALID_USER_GRADE | 일반 사용자만 사업주로 전환할 수 있습니다. |

---

## 6. 성능 최적화 및 캐싱

### 6.1 캐시 전략

#### 6.1.1 사용자 정보 캐싱

```java
@Cacheable(value = "users", key = "#email")
public Optional<User> findByEmail(String email) {
    return userRepository.findByEmail(email);
}

@Cacheable(value = "users", key = "#userId")
public Optional<User> findById(Long userId) {
    return userRepository.findById(userId);
}
```

#### 6.1.2 캐시 무효화

```java
@CacheEvict(value = "users", key = "#joinDto.email")
public User joinUser(JoinDto joinDto) {
    // 회원가입 로직
}

@CacheEvict(value = "users", allEntries = true)
public User convertToOwner(Long userId) {
    // 사업주 전환 로직
}
```

### 6.2 데이터베이스 최적화

#### 6.2.1 인덱스 전략

```sql
-- 이메일 기반 조회 최적화
CREATE INDEX idx_user_email ON users(email);

-- 사용자 등급별 조회 최적화
CREATE INDEX idx_user_grade ON users(user_grade);

-- 생성일 기반 정렬 최적화
CREATE INDEX idx_user_created_at ON users(created_at);

-- Refresh Token 조회 최적화
CREATE INDEX idx_refresh_token ON refresh_tokens(token);
CREATE INDEX idx_refresh_expires ON refresh_tokens(expires_at);
```

#### 6.2.2 쿼리 최적화

- **N+1 문제 방지**: @EntityGraph, Fetch Join 활용
- **페이징 처리**: Pageable 인터페이스 활용
- **배치 처리**: @Transactional(readOnly = true) 적용

### 6.3 보안 성능

#### 6.3.1 비밀번호 해싱 최적화

- **BCrypt 라운드 조정**: 보안과 성능의 균형
- **비동기 처리**: 회원가입 시 이메일 발송 등 비동기 처리

#### 6.3.2 JWT 토큰 최적화

- **토큰 크기 최소화**: 필수 정보만 포함
- **서명 알고리즘**: HS256 (대칭키) 사용으로 성능 향상

---

## 7. 모니터링 및 로깅

### 7.1 로깅 전략

#### 7.1.1 보안 관련 로깅

```java
// 로그인 시도 로깅
log.info("로그인 시도 - 이메일: {}, IP: {}", email, request.getRemoteAddr());

// 로그인 성공/실패 로깅
log.info("로그인 성공 - 사용자ID: {}, 등급: {}", user.getId(), user.getUserGrade());
log.warn("로그인 실패 - 이메일: {}, 사유: {}", email, "비밀번호 불일치");

// 권한 변경 로깅
log.info("사업주 전환 - 사용자ID: {}, 이전등급: {}, 변경등급: {}", 
         userId, previousGrade, UserGrade.MASTER);
```

#### 7.1.2 성능 모니터링

```java
// 메서드 실행 시간 측정
@Timed(name = "auth.login.duration", description = "로그인 처리 시간")
public ResponseEntity<ApiResponse> login(@RequestBody Login login) {
    // 로그인 로직
}

// 캐시 히트율 모니터링
@CacheResult(cacheName = "users")
@Timed(name = "auth.user.lookup.duration")
public Optional<User> findByEmail(String email) {
    // 사용자 조회 로직
}
```

### 7.2 메트릭 수집

#### 7.2.1 비즈니스 메트릭

- **회원가입 수**: 일별, 주별, 월별 신규 가입자 수
- **로그인 성공률**: 로그인 시도 대비 성공률
- **사업주 전환율**: 일반 사용자 대비 사업주 전환 비율
- **토큰 갱신 빈도**: Refresh Token 사용 패턴

#### 7.2.2 기술 메트릭

- **응답 시간**: API 엔드포인트별 평균 응답 시간
- **에러율**: HTTP 상태 코드별 에러 발생률
- **캐시 성능**: 캐시 히트율, 미스율
- **데이터베이스 성능**: 쿼리 실행 시간, 커넥션 풀 사용률

---

## 8. 향후 개선 계획

### 8.1 보안 강화

- **2FA (Two-Factor Authentication)**: 이중 인증 도입
- **OAuth 2.0 확장**: 구글, 네이버 등 추가 소셜 로그인
- **비밀번호 정책 강화**: 복잡성 검증, 재사용 방지
- **계정 잠금**: 로그인 실패 시 계정 임시 잠금

### 8.2 사용자 경험 개선

- **소셜 로그인 간소화**: 원클릭 회원가입
- **비밀번호 없는 로그인**: 매직 링크, SMS 인증
- **프로필 관리**: 사용자 정보 수정, 프로필 이미지
- **알림 설정**: 로그인 알림, 보안 알림

### 8.3 관리 기능 확장

- **사용자 관리**: 관리자용 사용자 검색, 상태 변경
- **감사 로그**: 모든 인증 관련 활동 추적
- **통계 대시보드**: 사용자 활동 분석
- **백업 및 복구**: 사용자 데이터 백업 전략

---

## 🔗 관련 문서

- [소담 전체 기능 목록 v1.0](../project-management/소담_전체기능목록_v1.0.md)
- [개발 가이드라인](../guidelines/development_guidelines.md)
- [데이터베이스 설계 문서](./데이터베이스_설계_문서.md)
- [API 문서](./FE개발자를_위한_API_및_서비스로직_문서.md)
- [보안 가이드라인](../guidelines/보안_가이드라인.md)

## 📅 변경 이력

| 날짜         | 버전  | 변경 내용                         | 작성자    |
|------------|-----|-------------------------------|--------|
| 2025-07-26 | 1.0 | 초기 작성 - AUTH 기능 데이터 처리 절차 문서화 | 소담 개발팀 |

---

**문서 승인:**

- 개발팀장: 이호영
- 백엔드 개발자: [ ]
- 검토 완료일: 2025-07-26
