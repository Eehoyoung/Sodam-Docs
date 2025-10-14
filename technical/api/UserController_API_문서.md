# UserController — 사용자 관리 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

사용자 정보 조회, 사업주 전환, 직원 정보 수정 기능을 RN에서 연동할 수 있도록 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT 권장(역할/권한 정책 적용)
- 공통 헤더: Authorization, Content-Type: application/json

## 🌐 베이스 경로

- /api/user

## 📚 엔드포인트 요약

- POST /{userId}/convert-to-owner — 일반 사용자를 사업주로 전환
- GET /{userId} — 사용자 정보 조회
- PUT /{employeeId} — 직원 정보 수정(EmployeeUpdateDto)

---

## 1) 사업주 전환 (AUTH-008)

- 메서드/경로: POST /api/user/{userId}/convert-to-owner
- 응답: ApiResponse<User>
- 오류: 이미 사업주이거나 전환 불가 조건 시 400

예시

```
curl -X POST "{HOST}/api/user/10/convert-to-owner" -H "Authorization: Bearer {ACCESS_TOKEN}"
```

---

## 2) 사용자 정보 조회

- 메서드/경로: GET /api/user/{userId}
- 응답: ApiResponse<User>
- 오류: USER_NOT_FOUND(400)

---

## 3) 직원 정보 수정 (STORE-012)

- 메서드/경로: PUT /api/user/{employeeId}
- 요청 바디(EmployeeUpdateDto): 이름/이메일/직책 등 수정 필드
- 응답: ApiResponse<User>

Axios 예시

```ts
import axios from "axios";
export async function updateEmployee(accessToken: string, employeeId: number, updateDto: any) {
  const res = await axios.put(`${process.env.API_BASE_URL}/api/user/${employeeId}`, updateDto, {
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  return res.data; // ApiResponse<User>
}
```

## 🚨 오류 코드(요약)

- 400: INVALID_REQUEST, USER_NOT_FOUND
- 401: UNAUTHORIZED
- 500: INTERNAL_ERROR

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
