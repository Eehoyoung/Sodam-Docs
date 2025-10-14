# RN 연동 API 검토 회신서

## 📋 개요

- 작성일: 2025-09-19
- 작성자: 백엔드 팀 (AI 보조: Junie)
- 문서 유형: 보고서/회신
- 관련 이슈: RN팀 workplaceService.ts 연동 확인 및 API 정렬

## 🎯 목적

RN팀이 제공한 workplaceService.ts의 API 계약과 백엔드 실제 엔드포인트 간 불일치 여부를 점검하고, 필요한 백엔드 수정 및 RN 측 추가 작업 사항을 정리하여 원활한 연동을 보장합니다.

## ✅ 결과 요약(백엔드 반영 사항)

다음 엔드포인트/동작을 RN 코드와 일치하도록 백엔드에 반영했습니다. RN 측 기존 코드 변경 없이 사용 가능합니다.

1. 매장 목록(사장 기준)
    - RN 호출: GET /api/stores/master/{userId 또는 'current'}
    - 백엔드: 'current' 경로 변수 처리 추가. 인증 사용자 ID 자동 사용.

2. 매장 단건 조회
    - RN 호출: GET /api/stores/{id}
    - 백엔드: 활성 매장 단건 조회 엔드포인트 추가.

3. 매장 등록
    - RN 호출: POST /api/stores/registration (바디만 전송)
    - 백엔드: userId 쿼리 파라미터를 선택값으로 변경. 미지정 시 인증 사용자(ID)를 자동 적용.

4. 매장 일반 정보 수정
    - RN 호출: PUT /api/stores/{id}
    - 백엔드: StoreUpdateDto 기반 일반 업데이트(매장명/전화번호/업종/주소/반경/기준시급) 엔드포인트 추가.

5. 매장 삭제
    - RN 호출: DELETE /api/stores/{id}
    - 백엔드: 소프트 삭제(soft delete) 엔드포인트 추가.

6. 매장 직원 목록
    - RN 호출: GET /api/stores/{storeId}/employees
    - 백엔드: 기존 엔드포인트 그대로 사용 가능.

추가로, POST/PUT 시 주소 문자열이 전달되면(등록/위치변경) 지오코딩을 통해 좌표를 자동 갱신합니다.

## 🔄 응답 스키마 유의사항(이름 매핑)

백엔드 Store 엔티티 필드명과 RN Workplace 타입 간에 이름 차이가 있습니다. RN 타입 예시(추정):

- RN: { id, name, address }
- 백엔드: { id, storeName, fullAddress(또는 roadAddress) , ... }

RN에서 Workplace로 매핑 시 다음과 같이 처리해 주세요:

- name ← storeName
- address ← fullAddress(우선) 또는 roadAddress(보조)

예: 목록/상세 조회 후 클라이언트 어댑터

```ts
function toWorkplace(store: any): Workplace {
  return {
    id: String(store.id),
    name: store.storeName,
    address: store.fullAddress ?? store.roadAddress ?? '',
  };
}
```

백엔드에서 별도의 전용 DTO를 추가해 필드명을 완전히 일치시키는 방안도 가능하지만, 현재는 최소 변경 원칙에 따라 RN 측 매핑으로 정렬하는 것을 권장합니다.

## 🔐 인증/보안 연동 주의사항

- 'current' 경로 사용 시 반드시 Authorization: Bearer <JWT> 헤더가 포함되어야 합니다.
- RN axios 공용 인스턴스(api)에서 토큰 자동 첨부를 보장해 주세요.
- 등록(POST /registration)에서 userId를 보내지 않을 경우 서버가 인증 사용자 ID를 사용합니다.

## 🧩 RN 추가 개발 권고 사항

1. 타입 안전성 강화
    - Workplace 타입과 백엔드 응답 간 매퍼 함수 추가(toWorkplace/toStorePayload 등).
2. 에러 처리 일관화
    - 네트워크/권한 오류 분기 처리 및 사용자 피드백(Toast/Alert) 표준화.
3. Mock 데이터 조건부 사용
    - __DEV__에서만 fallback 사용 유지, 운영에서는 서버 오류 화면 노출 및 재시도 UX 제공.
4. 캐싱/상태관리
    - 매장 목록(사장/직원) 캐시 전략(예: react-query) 적용 검토.
5. 입력 검증
    - 매장 등록/수정 시 필수값(매장명/전화/업종/주소 등) 클라이언트 측 선검증.

## 🔗 최종 백엔드 엔드포인트 요약

- GET /api/stores/master/{userId|current}
- GET /api/stores/employee/{userId}
- GET /api/stores/{id}
- POST /api/stores/registration   (userId 선택적 쿼리 파라미터)
- PUT /api/stores/{id}
- PUT /api/stores/{id}/location
- DELETE /api/stores/{id}
- GET /api/stores/{storeId}/employees

모든 응답/오류 메시지/로그는 한국어 기준으로 구성되어 있습니다.

## 📦 요청 바디 스키마 참고(요약)

- StoreRegistrationDto: storeName, businessNumber(10자리), storePhoneNumber, businessType, storeStandardHourWage, query(
  주소), radius 등
- StoreUpdateDto(신규): storeName?, storePhoneNumber?, businessType?, fullAddress?, roadAddress?, jibunAddress?, radius?,
  storeStandardHourWage?

## 🧪 간단 호출 예시

```ts
// 목록(사장)
await api.get('/api/stores/master/current');

// 등록
await api.post('/api/stores/registration', payload); // Authorization 헤더 필수

// 수정
await api.put(`/api/stores/${id}`, updatePayload);

// 삭제
await api.delete(`/api/stores/${id}`);
```

## 📄 RN 프로젝트 설명 프롬프트(.md 템플릿)

아래 템플릿을 복사하여 RN 프로젝트 개요를 작성해 전달해 주세요. 백엔드/QA가 빠르게 파악할 수 있습니다.

```
# RN 프로젝트 개요서

## 📋 기본 정보
- 앱명:
- 대상 플랫폼: (Android/iOS/Web)
- 주요 패키지/폴더 구조:
- 핵심 라이브러리 버전(React Native, React Navigation, axios 등):

## 🔐 인증/네트워킹
- 토큰 저장/갱신 전략(Access/Refresh):
- axios 인터셉터 구성(인증/에러 처리):
- 환경변수 관리(.env):

## 🧭 라우팅
- 스택/탭 구조:
- 접근 제어(보호 라우트):

## 🧱 상태관리
- 사용 라이브러리(redux, zustand, react-query 등):
- 캐싱 정책(매장/근태/급여):

## 🧩 도메인 타입
- Workplace 타입 정의:
- Store/Attendance/Payroll 등 주요 타입:

## 🖼️ UI/UX
- 핵심 화면(목록/상세/등록/수정/삭제) 스크린샷 링크:
- 에러/로딩/빈 상태 처리 패턴:

## 🧪 테스트/품질
- E2E/단위 테스트 도구:
- 코드 검사(ESLint/Prettier) 설정:

## 📌 현 이슈/리스크
- API 연동 이슈:
- 퍼포먼스/UX 개선 포인트:

## 🔗 참고 링크
- 디자인 시스템/피그마:
- API 문서/스웨거:
```

## 📅 변경 이력

 날짜         | 버전  | 변경 내용                | 작성자   
------------|-----|----------------------|-------
 2025-09-19 | 1.0 | 최초 작성 및 백엔드 엔드포인트 정렬 | 백엔드 팀 
