# LoginController — 인증/토큰/회원가입 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

RN 앱에서 카카오 로그인, 자체 로그인, 회원가입, 토큰 갱신, 내 정보 조회 흐름을 즉시 구현할 수 있도록 상세 API 계약과 예시를 제공합니다.

## 🔐 인증/헤더

- 인증 필요 엔드포인트: /api/auth/refresh(Refresh 토큰), /api/auth/me, /api/me (Access 토큰)
- 공통 헤더
    - Authorization: Bearer {accessToken} (필요 시)
    - Content-Type: application/json

## 📦 데이터 모델

- Login { email: string, password: string }
- JoinDto { email: string, password: string, name: string, ... }
- RefreshToken 요청 바디: { refreshToken: string }

## 📚 엔드포인트 요약

- GET /kakao/auth/proc — 카카오 인증 코드 처리(서버-카카오 연동)
- POST /api/login — 자체 로그인(이메일/비밀번호)
- POST /api/join — 회원가입
- POST /api/auth/refresh — 토큰 갱신(Refresh → 새로운 Access/Refresh)
- GET /api/auth/me 또는 /api/me — 현재 로그인 사용자 정보 조회

---

## 1) 카카오 로그인 처리

- 메서드/경로: GET /kakao/auth/proc?code=xxx
- 요청: code (카카오 OAuth 인증 코드)
- 응답: ApiResponse<{ redirectUrl, userGrade, accessToken, refreshToken, userId }>
- 상태 코드: 200 | 500

예시

```
curl "{HOST}/kakao/auth/proc?code={KAKAO_CODE}"
```

---

## 2) 자체 로그인(이메일/비밀번호)

- 메서드/경로: POST /api/login
- 요청 바디(Login)
    - email: string
    - password: string
- 응답: ApiResponse<{ accessToken, refreshToken, userId, userGrade }>
- 상태 코드: 200 | 401 | 500

Axios 예시

```ts
import axios from "axios";
export async function login(email: string, password: string) {
  const res = await axios.post(`${process.env.API_BASE_URL}/api/login`, { email, password });
  return res.data; // { success, message, data: { accessToken, refreshToken, userId, userGrade } }
}
```

---

## 3) 회원가입

- 메서드/경로: POST /api/join
- 헤더(선택): X-User-Purpose, X-User-Grade (서버 로깅용 메타데이터)
- 요청 바디(JoinDto)
    - email, password, name, ... (서버 구현 기준)
- 응답: ApiResponse<void>
- 상태 코드: 200

예시

```
curl -X POST "{HOST}/api/join" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@sodam.com","password":"pw","name":"홍길동"}'
```

---

## 4) 토큰 갱신

- 메서드/경로: POST /api/auth/refresh
- 요청 바디: { refreshToken: string }
- 응답: ApiResponse<{ accessToken, refreshToken, userId, userGrade }>
- 상태 코드: 200 | 400(파라미터 누락) | 401(만료/무효) | 500

Axios 예시

```ts
export async function refreshToken(refreshToken: string) {
  const res = await axios.post(`${process.env.API_BASE_URL}/api/auth/refresh`, { refreshToken });
  return res.data; // 새 토큰 정보
}
```

---

## 5) 내 정보 조회

- 메서드/경로: GET /api/auth/me 또는 /api/me
- 인증: Access 토큰
- 응답: { id, email, name, roles: string[] }
- 상태 코드: 200 | 401 | 404 | 500

예시

```
curl "{HOST}/api/auth/me" -H "Authorization: Bearer {ACCESS_TOKEN}"
```

---

## 🚨 오류 코드(요약)

- 400: refreshToken 누락
- 401: 로그인 실패/토큰 무효 또는 만료
- 404: 사용자 없음(me)
- 500: 카카오 처리 실패/내부 오류

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
