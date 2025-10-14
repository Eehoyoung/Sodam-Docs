# FE팀 연결 문제 분석 보고서

## 📋 개요

- **작성일**: 2025-01-27
- **작성자**: 백엔드 개발팀
- **문서 유형**: 문제 분석 보고서
- **관련 이슈**: FE팀에서 보고한 연결 문제
- **분석 대상**: 안드로이드 로그 파일 `Medium-Phone-Android-16_2025-07-15_212310.logcat`
- **우선순위**: HIGH

## 🎯 문제 상황

FE팀에서 "연결이 안된다"고 로그를 보내왔으며, 안드로이드 에뮬레이터에서 발생한 로그 파일을 분석하여 문제점을 파악하고 해결 방안을 제시합니다.

## 📊 로그 분석 결과

### ✅ 정상 동작 확인 사항

1. **앱 설치 및 실행**: com.sodam_front_end 앱이 정상적으로 설치되고 실행됨
2. **프로세스 시작**: 프로세스 4170으로 앱이 정상 시작됨
3. **MainActivity 표시**: "Displayed com.sodam_front_end/.MainActivity for user 0: +1s848ms" - 앱이 1.8초 만에 표시됨
4. **앱 상태**: 앱이 정상적으로 실행되고 화면에 표시됨

### ⚠️ 발견된 문제점

#### 1. 앱 재설치 과정에서의 불안정성

**문제 상황:**

```
"Force stopping com.sodam_front_end appid=10215 user=-1: installPackageLI"
"Killing 3979:com.sodam_front_end/u0a215 (adj 0): stop com.sodam_front_end due to install"
"WINDOW DIED Window{2e28467 u0 com.sodam_front_end/com.sodam_front_end.MainActivity}"
```

**분석:**

- 앱 업데이트/재설치 과정에서 기존 프로세스가 강제 종료됨
- 윈도우가 예기치 않게 종료되는 현상 발생
- 이는 개발 중 빈번한 앱 재빌드로 인한 정상적인 현상일 수 있음

#### 2. 반복적인 패키지 정보 누락 경고

**문제 상황:**

```
"Package [com.sodam_front_end] reported as REPLACED, but missing application info. Assuming removed."
```

**분석:**

- 앱 교체 과정에서 애플리케이션 정보가 일시적으로 누락됨
- 시스템이 앱이 제거된 것으로 잘못 인식
- 최종적으로는 정상 복구되었지만 불안정한 상태를 나타냄

## 🔧 백엔드 설정 분석

### ✅ 정상 설정 확인

1. **CORS 설정**: 안드로이드 에뮬레이터용 `10.0.2.2:8081` 포함하여 적절히 구성됨
2. **서버 포트**: 8080 포트에서 정상 실행
3. **환경변수**: JWT_SECRET, DB_PASSWORD 등 필수 설정 완료
4. **API 문서**: 상세한 API 문서 제공으로 FE 개발 지원

### ⚠️ 발견된 설정 문제

#### 1. CORS 설정 오류 (HIGH PRIORITY)

**파일**: `src/main/java/com/rich/sodam/config/WebConfig.java`
**문제 라인**: 17번 라인

```java
config.addAllowedOriginPattern("/**");  // ❌ 잘못된 패턴
```

**문제점:**

- `addAllowedOriginPattern("/**")`는 잘못된 Origin 패턴임
- Origin 패턴은 URL 형식이어야 함 (예: `http://*`, `https://*.example.com`)
- 현재 설정으로는 CORS 에러가 발생할 수 있음

**올바른 설정:**

```java
config.addAllowedOriginPattern("*");  // 모든 Origin 허용
// 또는
config.addAllowedOriginPattern("http://*");  // HTTP 프로토콜만 허용
```

#### 2. 카카오 OAuth 설정 불완전

**파일**: `.env`
**문제점:**

```env
KAKAO_CLIENT_ID=your_kakao_client_id  # 더미값
```

**영향:**

- 카카오 로그인 기능이 동작하지 않음
- FE에서 소셜 로그인 테스트 불가

## 🚨 추정되는 연결 문제 원인

### 1. CORS 정책 위반 (가장 가능성 높음)

- 잘못된 `addAllowedOriginPattern("/**")` 설정으로 인한 CORS 에러
- 브라우저/앱에서 API 호출 시 CORS 정책 위반으로 요청 차단

### 2. 네트워크 설정 문제

- 안드로이드 에뮬레이터에서 `localhost:8080` 접근 불가
- `10.0.2.2:8080` 사용 필요 (에뮬레이터의 호스트 머신 접근)

### 3. 개발 환경 불일치

- FE팀이 사용하는 서버 주소와 실제 백엔드 서버 주소 불일치
- API 베이스 URL 설정 오류

## 💡 해결 방안

### 즉시 수정 필요 (HIGH PRIORITY)

#### 1. CORS 설정 수정

```java
// WebConfig.java 17번 라인 수정
// 변경 전
config.addAllowedOriginPattern("/**");

// 변경 후
config.addAllowedOriginPattern("*");
```

#### 2. 개발 환경용 Origin 추가

```java
// 개발 환경을 위한 추가 Origin 설정
config.addAllowedOrigin("http://10.0.2.2:3000");   // 웹 개발 서버
config.addAllowedOrigin("http://10.0.2.2:8081");   // RN 개발 서버
config.addAllowedOrigin("http://192.168.1.100:8081"); // 실제 IP 주소
```

### 권장 사항

#### 1. FE팀과의 협업 체크리스트

- [ ] 백엔드 서버 주소 확인: `http://10.0.2.2:8080` (에뮬레이터) 또는 `http://localhost:8080` (웹)
- [ ] API 베이스 URL 설정 확인
- [ ] 네트워크 연결 상태 확인
- [ ] CORS 에러 발생 여부 브라우저 콘솔에서 확인

#### 2. 디버깅 지원

- [ ] 백엔드 로그에서 API 요청 수신 여부 확인
- [ ] Swagger UI 접근 테스트: `http://10.0.2.2:8080/swagger-ui.html`
- [ ] Health Check 엔드포인트 테스트: `http://10.0.2.2:8080/actuator/health`

#### 3. 카카오 OAuth 설정 완료

```env
# .env 파일에 실제 값 설정 필요
KAKAO_CLIENT_ID=실제_카카오_클라이언트_ID
KAKAO_CLIENT_SECRET=실제_카카오_클라이언트_시크릿
```

## 📋 액션 아이템

### 백엔드 팀 (즉시 수행)

1. **CORS 설정 수정** - WebConfig.java 17번 라인 수정
2. **서버 재시작** - 설정 변경 후 서버 재시작
3. **테스트 수행** - Swagger UI 접근 및 기본 API 테스트

### FE팀 협업 (수정 후 수행)

1. **연결 테스트** - 수정된 백엔드와 연결 테스트
2. **API 호출 테스트** - 기본 인증 API 호출 테스트
3. **에러 로그 공유** - 여전히 문제 발생 시 브라우저/앱 콘솔 로그 공유

### 장기 개선 사항

1. **환경별 CORS 설정 분리** - 개발/스테이징/운영 환경별 CORS 정책 분리
2. **API 응답 시간 모니터링** - 성능 최적화를 위한 모니터링 구축
3. **자동화된 연결 테스트** - CI/CD 파이프라인에 FE-BE 연결 테스트 추가

## 🔗 관련 문서

- [개발 가이드라인](../guidelines/development_guidelines.md)
- [FE개발자를 위한 API 문서](../technical/FE개발자를_위한_API_및_서비스로직_문서.md)
- [로그 분석 보고서](./로그_분석_보고서.md)

## 📅 변경 이력

| 날짜         | 버전  | 변경 내용        | 작성자     |
|------------|-----|--------------|---------|
| 2025-01-27 | 1.0 | 초기 분석 보고서 작성 | 백엔드 개발팀 |

---

**결론**: FE팀의 연결 문제는 주로 CORS 설정 오류로 인한 것으로 추정됩니다. WebConfig.java의 17번 라인을 수정하고 서버를 재시작한 후 재테스트를 권장합니다.
