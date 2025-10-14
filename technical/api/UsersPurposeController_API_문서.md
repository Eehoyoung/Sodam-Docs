# UsersPurposeController — 사용자 목적 설정 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

카카오 최초 가입자 등 사용자가 자신의 목적(personal/employee/boss)을 설정하여 등급을 갱신하는 절차를 RN에서 안전하게 구현할 수 있도록 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT 필수
- 본인 확인: 경로의 {userId}는 토큰의 userId와 일치해야 함
- Content-Type: application/json

## 🌐 베이스 경로

- /api/users

## 📚 엔드포인트 요약

- POST /{userId}/purpose — 목적 설정(PurposeRequest)

---

## 1) 목적 설정

- 메서드/경로: POST /api/users/{userId}/purpose
- 요청 바디(PurposeRequest)
    - purpose: "personal" | "employee" | "boss"
- 인증 검증 흐름
    - Authorization 헤더의 토큰을 검증하고, 토큰의 userId가 경로 {userId}와 같아야 함
- 응답: ApiResponse<{ userId, userGrade }>
- 오류: 401(토큰 무효/본인 아님), 409(정책 충돌), 400(잘못된 요청), 500(내부 오류)

Axios 예시

```ts
import axios from "axios";
export async function setPurpose(accessToken: string, userId: number, purpose: "personal"|"employee"|"boss") {
  const res = await axios.post(`${process.env.API_BASE_URL}/api/users/${userId}/purpose`, { purpose }, {
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  return res.data; // ApiResponse<{ userId, userGrade }>
}
```

## 🚨 오류 코드(요약)

- 401: UNAUTHORIZED (토큰 없음/무효, 본인 아님)
- 409: CONFLICT (정책 충돌, 다운그레이드 등)
- 400: BAD_REQUEST
- 500: INTERNAL_ERROR

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
