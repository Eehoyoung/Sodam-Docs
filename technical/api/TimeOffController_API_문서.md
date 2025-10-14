# TimeOffController — 휴가 신청/승인 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

직원 휴가 신청 생성/조회와 사장의 승인/거부 흐름을 RN에서 연동할 수 있도록 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT (직원/사장 권한 구분 적용 권장)
- Content-Type: application/json (POST/PUT이지만 @RequestParam 기반 → 쿼리 또는 form 전송 가능)
- Date 형식: yyyy-MM-dd

## 🌐 베이스 경로

- /api/timeoff

## 📚 엔드포인트 요약

- POST / — 휴가 신청 생성(@RequestParam)
- GET /store?storeId — 특정 매장의 모든 휴가 신청
- GET /store/status?storeId&status — 특정 매장의 특정 상태 휴가 신청
- GET /employee?employeeId — 특정 직원의 모든 휴가 신청
- PUT /{timeOffId}/approve — 휴가 승인(사장)
- PUT /{timeOffId}/reject — 휴가 거부(사장)

---

## 1) 휴가 신청 생성

- 메서드/경로: POST /api/timeoff
- 파라미터(@RequestParam)
    - employeeId: number
    - storeId: number
    - startDate: string(yyyy-MM-dd)
    - endDate: string(yyyy-MM-dd)
    - reason: string
- 응답: TimeOff

curl 예시(폼 전송)

```
curl -X POST "{HOST}/api/timeoff" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -d "employeeId=12" -d "storeId=101" \
  -d "startDate=2025-05-01" -d "endDate=2025-05-03" \
  -d "reason=가족 행사"
```

Axios 예시(쿼리)

```ts
import axios from "axios";
export async function createTimeOff(accessToken: string, params: { employeeId: number; storeId: number; startDate: string; endDate: string; reason: string; }) {
  const res = await axios.post(`${process.env.API_BASE_URL}/api/timeoff`, null, {
    params,
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  return res.data; // TimeOff
}
```

---

## 2) 조회

- 매장 전체: GET /api/timeoff/store?storeId=101 → TimeOff[]
- 매장 상태별: GET /api/timeoff/store/status?storeId=101&status=PENDING → TimeOff[]
- 직원 전체: GET /api/timeoff/employee?employeeId=12 → TimeOff[]

---

## 3) 승인/거부(사장)

- 승인: PUT /api/timeoff/{timeOffId}/approve → TimeOff
- 거부: PUT /api/timeoff/{timeOffId}/reject → TimeOff

## 🚨 오류 코드(요약)

- 400: 파라미터 오류
- 401: 인증 실패
- 403: 권한 부족(사장만 승인/거부 가능)
- 404: 대상 없음

## 🔗 관련 문서

- MasterController 문서(사장 마이페이지/휴가 관련)
- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
