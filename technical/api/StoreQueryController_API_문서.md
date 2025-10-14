# StoreQueryController — 매장 조회/통계 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

StoreRepository 기반의 조회/통계 API를 RN에서 쉽게 사용하도록 엔드포인트, 파라미터, 예시를 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT 권장(운영 정책에 따라 공개 가능)
- 공통 헤더: Authorization

## 🌐 베이스 경로

- /api/store-queries

## 📚 엔드포인트 요약

- 활성 매장
    - GET /active — 전체
    - GET /active/{id} — 단건
    - GET /active/by-business-number?businessNumber= — 사업자번호로 단건
- 삭제된 매장
    - GET /deleted — 전체
    - GET /deleted/{id} — 단건
- 소유권/소속 확인
    - GET /by-master/{userId} — 사장이 관리하는 활성 매장 목록
    - GET /by-employee/{userId} — 직원이 소속된 활성 매장 목록
    - GET /ownership/exists?storeId&userId — 사장 여부(Boolean)
    - GET /membership/exists?storeId&userId — 직원 여부(Boolean)
- 통계/부가정보
    - GET /{storeId}/stats/employees/active/count — 활성 직원 수
    - GET /{storeId}/stats/attendance/count — 출근 기록 수
    - GET /{storeId}/stats/payroll/count — 급여 기록 수
    - GET /{storeId}/stats/payroll/unpaid/count — 미지급 급여 수
    - GET /{storeId}/last-activity — 마지막 활동 시각(LocalDateTime)
- 검색/필터
    - GET /search/by-name?storeName= — 이름으로 활성 매장 검색
    - GET /search/by-created-at?startDate&endDate — 특정 기간 생성된 활성 매장
    - GET /search/deleted-by-deleted-at?startDate&endDate — 특정 기간 삭제된 매장

---

## 파라미터 규칙

- DATETIME: ISO-8601 (예: 2025-01-01T00:00:00)
- 문자열 길이: storeName 최대 100자

예시

```
curl "{HOST}/api/store-queries/search/by-name?storeName=소담"
```

## 🚨 오류 코드(요약)

- 400: 잘못된 요청(형식, 길이 등)
- 401: 인증 실패(정책에 따라)
- 404: 대상 없음

## 🔗 관련 문서

- StoreController 문서(등록/갱신/직원 관리)

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
