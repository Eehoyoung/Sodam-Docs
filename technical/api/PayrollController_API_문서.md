# PayrollController — 급여 관리 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

직원 급여 계산/조회/상태변경 등 급여 관리 기능을 RN에서 안전하게 연동할 수 있도록 상세 계약을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT (권장)
- 공통 헤더
    - Authorization: Bearer {accessToken}
    - Content-Type: application/json
- 날짜 형식: DATE(yyyy-MM-dd), DATETIME(ISO-8601)

## 🌐 베이스 경로

- /api/payroll

## 📚 엔드포인트 요약

- GET /employee/{employeeId}/store/{storeId}?startDate&endDate — 기간 급여 합계 계산(Integer)
- GET /employee/{employeeId}/wages — 직원의 전체 매장별 임금 정보(EmployeeWageInfoDto[])
- GET /employee/{employeeId}/store/{storeId}/monthly?year&month — 특정 월 급여 합계(Integer)
- POST /calculate — 급여 계산 및 급여 레코드 생성(PayrollDto)
- PUT /{payrollId}/status — 급여 상태 변경(PayrollDto)
- GET /employee/{employeeId}?from&to — 직원 급여 내역(PayrollDto[])
- GET /store/{storeId}?from&to — 매장 급여 내역(PayrollDto[])
- GET /{payrollId}/details — 급여 상세(PayrollDetailDto[])

---

## 1) 기간 급여 계산

- 메서드/경로: GET /api/payroll/employee/{employeeId}/store/{storeId}
- 쿼리: startDate(ISO DATETIME), endDate(ISO DATETIME)
- 응답: number (정수, 금액)

예시

```
curl -G "{HOST}/api/payroll/employee/12/store/101" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  --data-urlencode "startDate=2025-05-01T00:00:00" \
  --data-urlencode "endDate=2025-05-31T23:59:59"
```

---

## 2) 전체 매장 임금 정보 조회

- 메서드/경로: GET /api/payroll/employee/{employeeId}/wages
- 응답: EmployeeWageInfoDto[]

---

## 3) 월별 급여 계산

- 메서드/경로: GET /api/payroll/employee/{employeeId}/store/{storeId}/monthly
- 쿼리: year(int), month(1-12)
- 응답: number

---

## 4) 급여 계산 및 생성

- 메서드/경로: POST /api/payroll/calculate
- 요청 바디(PayrollCalculationRequestDto)
    - employeeId: number
    - storeId: number
    - startDate: string(yyyy-MM-dd)
    - endDate: string(yyyy-MM-dd)
    - recalculate: boolean(optional)
- 응답: PayrollDto

Axios 예시

```ts
import axios from "axios";
export async function calculatePayroll(accessToken: string, body: any) {
  const res = await axios.post(`${process.env.API_BASE_URL}/api/payroll/calculate`, body, {
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  return res.data; // PayrollDto
}
```

---

## 5) 급여 상태 업데이트

- 메서드/경로: PUT /api/payroll/{payrollId}/status
- 요청 바디(PayrollStatusUpdateDto)
    - status: string(PENDING/CONFIRMED/PAID/CANCELLED 등)
    - paymentDate: string(yyyy-MM-dd, 선택)
    - cancelReason: string(선택)
- 응답: PayrollDto

---

## 6) 급여 내역 조회

- 직원: GET /api/payroll/employee/{employeeId}?from=YYYY-MM-DD&to=YYYY-MM-DD
- 매장: GET /api/payroll/store/{storeId}?from=YYYY-MM-DD&to=YYYY-MM-DD
- 응답: PayrollDto[]

---

## 7) 급여 상세 조회

- 메서드/경로: GET /api/payroll/{payrollId}/details
- 응답: PayrollDetailDto[]

## 🚨 오류 코드(요약)

- 400: 잘못된 요청 (파라미터 누락/타입 오류)
- 401: 인증 실패
- 404: 직원/매장/급여 없음
- 500: 서버 오류

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
