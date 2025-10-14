# WageController — 임금 관리 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

직원 임금 업데이트, 매장 기준 시급 변경, 특정 직원의 매장별 임금 조회/할당을 RN에서 연동할 수 있도록 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT (사장 권한 권장)
- Content-Type: application/json

## 🌐 베이스 경로

- /api/wages

## 📚 엔드포인트 요약

- POST /employee — 직원 임금 업데이트(EmployeeWageUpdateDto)
- PUT /store/{storeId}/standard?standardHourlyWage= — 매장 기본 시급 업데이트
- GET /employee/{employeeId}/store/{storeId} — 직원의 매장별 임금 조회(Integer)
- POST /employee/{employeeId}/store/{storeId}?customHourlyWage= — 직원 할당 및 임금 설정

---

## 1) 직원 임금 업데이트

- 메서드/경로: POST /api/wages/employee
- 요청 바디: EmployeeWageUpdateDto
- 응답: 200 OK

---

## 2) 매장 기본 시급 업데이트

- 메서드/경로: PUT /api/wages/store/{storeId}/standard
- 쿼리: standardHourlyWage(int)
- 응답: 200 OK

예시

```
curl -X PUT "{HOST}/api/wages/store/101/standard?standardHourlyWage=9860" -H "Authorization: Bearer {ACCESS_TOKEN}"
```

---

## 3) 직원 매장별 임금 조회

- 메서드/경로: GET /api/wages/employee/{employeeId}/store/{storeId}
- 응답: number (적용 임금)

---

## 4) 직원 할당 및 임금 설정

- 메서드/경로: POST /api/wages/employee/{employeeId}/store/{storeId}
- 쿼리: customHourlyWage(optional)
- 동작: 해당 직원을 매장에 할당하고 임금 설정(미지정 시 매장 기준 시급 사용)
- 응답: 200 OK

## 🚨 오류 코드(요약)

- 400: 잘못된 요청
- 401: 인증 실패
- 404: 직원/매장 없음

## 🔗 관련 문서

- StoreController 문서(직원 할당 관련)
- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
