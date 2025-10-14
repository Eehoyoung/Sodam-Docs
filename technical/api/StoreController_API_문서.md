# StoreController — 매장 등록/직원배치/갱신 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

매장 등록, 직원 할당, 매장/위치 정보 수정, 직원/매장 조회, 삭제 기능을 RN에서 즉시 연동할 수 있도록 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT (사장/직원 권한에 따라 접근 제약)
- 공통 헤더: Authorization: Bearer {accessToken}, Content-Type: application/json

## 🌐 베이스 경로

- /api/stores

## 📚 엔드포인트 요약

- POST /registration — 매장 등록(사장 지정)
- POST /{storeId}/employees — 매장에 직원 할당(+선택적 시급)
- GET /master/{userIdOrCurrent} — 사장 기준 매장 목록
- GET /employee/{userId} — 직원 기준 매장 목록
- GET /{storeId}/employees — 매장 직원 목록
- PUT /{storeId}/location — 매장 위치 정보 업데이트
- GET /{id} — 매장 단건 조회(활성)
- PUT /{storeId} — 매장 일반 정보 업데이트
- DELETE /{storeId} — 매장 삭제(소프트 삭제)

---

## 1) 매장 등록

- 메서드/경로: POST /api/stores/registration
- 쿼리: userId(선택, 미지정 시 현재 로그인 사용자)
- 요청 바디(StoreRegistrationDto)
    - storeName, businessNumber, storePhoneNumber, businessType, businessLicenseNumber,
    - query(전체 주소, 선택) → 서버에서 좌표/주소 정규화, latitude, longitude, roadAddress, jibunAddress 자동 세팅
    - radius(미터, 기본 100), storeStandardHourWage(기준 시급)
- 응답: Store

Axios 예시

```ts
import axios from "axios";
export async function registerStore(accessToken: string, store: any, userId?: number) {
  const res = await axios.post(`${process.env.API_BASE_URL}/api/stores/registration`, store, {
    params: userId ? { userId } : undefined,
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  return res.data; // Store
}
```

---

## 2) 매장에 직원 할당(임금 설정 포함)

- 메서드/경로: POST /api/stores/{storeId}/employees
- 쿼리: userId(required), customHourlyWage(optional)
- 응답: 200 OK

예시

```
curl -X POST "{HOST}/api/stores/123/employees?userId=45&customHourlyWage=10500" -H "Authorization: Bearer {ACCESS_TOKEN}"
```

---

## 3) 매장/직원 목록 조회

- 사장 기준 매장 목록: GET /api/stores/master/{userIdOrCurrent} ("current" 지원) → Store[]
- 직원 기준 매장 목록: GET /api/stores/employee/{userId} → Store[]
- 특정 매장의 직원 목록: GET /api/stores/{storeId}/employees → User[]

---

## 4) 매장 위치 정보 업데이트

- 메서드/경로: PUT /api/stores/{storeId}/location
- 요청 바디(LocationUpdateDto)
    - fullAddress(선택: 주소 변경 시), latitude/longitude(선택), radius(선택)
- 동작: fullAddress가 있으면 서버가 좌표 재계산 후 저장
- 응답: Store

---

## 5) 매장 일반 정보 업데이트/조회/삭제

- 단건 조회(활성): GET /api/stores/{id} → Store
- 일반 정보 업데이트: PUT /api/stores/{storeId} (StoreUpdateDto)
- 삭제(소프트): DELETE /api/stores/{storeId} → 204 No Content

## 🚨 오류 코드(요약)

- 400: 잘못된 요청(파라미터/형식 오류)
- 401: 인증 실패
- 403: 권한 부족
- 404: 사용자/매장 없음
- 500: 서버 오류

## 🔗 관련 문서

- StoreQueryController 문서(조회/통계 전용)
- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
