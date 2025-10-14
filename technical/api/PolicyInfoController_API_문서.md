# PolicyInfoController — 국가정책 정보 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

국가정책(지원금/정책 안내) 콘텐츠 생성/조회/수정/삭제 및 페이지네이션 조회를 RN에서 구현할 수 있도록 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT 권장(목록/상세는 공개 가능 정책)
- 공통 헤더: Authorization, Content-Type: application/json | multipart/form-data

## 🌐 베이스 경로

- /api/policy-info

## 📚 엔드포인트 요약

- POST / — 정책 생성 (multipart/form-data)
- GET /{id} — 단건 조회
- GET / — 전체 조회(비권장: 데이터 많을 수 있음)
- GET /paged — 페이지네이션 조회(Page<PolicyInfoResponseDto>)
- GET /recent — 최근 5개
- PUT /{id} — 수정 (multipart/form-data)
- DELETE /{id} — 삭제

---

## 1) 정책 생성

- 메서드/경로: POST /api/policy-info
- 요청: multipart/form-data { title, content, image? }
- 응답: PolicyInfoResponseDto

---

## 2) 단건 조회

- 메서드/경로: GET /api/policy-info/{id}
- 응답: PolicyInfoResponseDto

---

## 3) 전체/페이지네이션 조회

- 전체: GET /api/policy-info → PolicyInfoResponseDto[]
- 페이지: GET /api/policy-info/paged?page=0&size=10&sort=id,asc → Page

---

## 4) 최근 컨텐츠

- 메서드/경로: GET /api/policy-info/recent
- 응답: PolicyInfoResponseDto[]

---

## 5) 수정/삭제

- 수정: PUT /api/policy-info/{id} (multipart/form-data)
- 삭제: DELETE /api/policy-info/{id}

Axios 예시(페이지)

```ts
import axios from "axios";
export async function fetchPolicyPage(page=0, size=10) {
  const res = await axios.get(`${process.env.API_BASE_URL}/api/policy-info/paged`, { params: { page, size, sort: "id,desc" } });
  return res.data; // Page
}
```

## 🚨 오류 코드(요약)

- 400: 잘못된 요청
- 401: 인증 실패(정책에 따라)
- 404: 대상 없음
- 500: 서버 오류

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
