# PayrollPolicyController — 급여 정책 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

매장별 급여 정책(기본 시급, 야간/연장 가산 등 정책)을 조회/수정하는 기능 연동을 위한 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT (ROLE_MASTER 권장)
- 공통 헤더: Authorization, Content-Type: application/json

## 🌐 베이스 경로

- /api/payroll-policy

## 📚 엔드포인트 요약

- GET /store/{storeId} — 매장 급여 정책 조회(PayrollPolicyDto)
- POST /store/{storeId} — 매장 급여 정책 생성/수정(PayrollPolicyUpdateDto → PayrollPolicyDto)

---

## 1) 매장 급여 정책 조회

- 메서드/경로: GET /api/payroll-policy/store/{storeId}
- 응답: PayrollPolicyDto
- 상태 코드: 200 | 401 | 404

---

## 2) 매장 급여 정책 생성/수정

- 메서드/경로: POST /api/payroll-policy/store/{storeId}
- 요청 바디: PayrollPolicyUpdateDto (정책 필드 일체)
- 응답: PayrollPolicyDto

Axios 예시

```ts
import axios from "axios";
export async function upsertPayrollPolicy(accessToken: string, storeId: number, body: any) {
  const res = await axios.post(`${process.env.API_BASE_URL}/api/payroll-policy/store/${storeId}`, body, {
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  return res.data; // PayrollPolicyDto
}
```

## 🚨 오류 코드(요약)

- 400: 잘못된 요청
- 401: 인증 실패
- 404: 매장 정보 없음

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
