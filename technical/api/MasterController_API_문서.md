# MasterController — 사장 마이페이지/관리 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

사장(마스터) 사용자의 마이페이지, 프로필, 매장/통계, 휴가 승인 관련 기능을 RN에서 즉시 연동할 수 있도록 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT (ROLE_MASTER 권장)
- 공통 헤더
    - Authorization: Bearer {accessToken}
    - Content-Type: application/json

## 🌐 베이스 경로

- /api/master

## 📚 엔드포인트 요약

- GET /mypage?masterId — 마이페이지 종합 데이터
- GET /profile?masterId — 사장 프로필 조회
- PUT /profile?masterId&businessLicenseNumber — 사장 프로필 수정
- GET /stores?masterId — 사장이 소유한 매장 목록
- GET /store/stats?storeId&month=YYYY-MM — 특정 매장 통계
- GET /stats?masterId — 사장이 소유한 모든 매장 통합 통계
- GET /timeoff/pending?masterId — 대기중 휴가 신청 목록
- PUT /timeoff/approve?timeOffId — 휴가 승인
- PUT /timeoff/reject?timeOffId — 휴가 거부

---

## 1) 마이페이지 종합 데이터

- 메서드/경로: GET /api/master/mypage?masterId={id}
- 응답: MasterMyPageResponseDto (사장 정보, 매장 목록, 통합 통계, 휴가 신청 내역 등 포함)

예시

```
curl "{HOST}/api/master/mypage?masterId=1" -H "Authorization: Bearer {ACCESS_TOKEN}"
```

---

## 2) 사장 프로필 조회/수정

- 조회: GET /api/master/profile?masterId={id}
- 수정: PUT /api/master/profile?masterId={id}&businessLicenseNumber={no}
- 응답: MasterProfileResponseDto | MasterProfile

Axios 예시(수정)

```ts
import axios from "axios";
export async function updateMasterProfile(accessToken: string, masterId: number, businessLicenseNumber: string) {
  const res = await axios.put(`${process.env.API_BASE_URL}/api/master/profile`, null, {
    params: { masterId, businessLicenseNumber },
    headers: { Authorization: `Bearer ${accessToken}` },
  });
  return res.data;
}
```

---

## 3) 매장 목록 및 통계

- 매장 목록: GET /api/master/stores?masterId={id} → Store[]
- 매장 통계: GET /api/master/store/stats?storeId={id}&month=YYYY-MM → Map<String, Object>
- 통합 통계: GET /api/master/stats?masterId={id} → Map<String, Object>

---

## 4) 휴가 신청 관리

- 대기중 목록: GET /api/master/timeoff/pending?masterId={id} → TimeOff[]
- 승인: PUT /api/master/timeoff/approve?timeOffId={id} → TimeOff
- 거부: PUT /api/master/timeoff/reject?timeOffId={id} → TimeOff

예시(curl)

```
curl -X PUT "{HOST}/api/master/timeoff/approve?timeOffId=123" -H "Authorization: Bearer {ACCESS_TOKEN}"
```

## 🚨 오류 코드(요약)

- 400: 파라미터 오류
- 401: 인증 실패
- 403: 권한 부족(사장 아님)
- 404: 대상 없음(프로필/매장/휴가)
- 500: 서버 오류

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
