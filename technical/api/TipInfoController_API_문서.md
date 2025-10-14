# TipInfoController — 소상공인 꿀팁 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

소상공인 팁(콘텐츠) 생성/조회/검색/수정/삭제 기능을 RN에서 연동할 수 있도록 API 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT 권장(목록/상세는 공개 가능 정책)
- 업로드: multipart/form-data

## 🌐 베이스 경로

- /api/tip-info

## 📚 엔드포인트 요약

- POST / — 팁 생성 (multipart/form-data)
- GET /{id} — 단건 조회
- GET / — 전체 조회
- GET /recent — 최근 5개 조회
- GET /search/title?keyword= — 제목 검색
- GET /search/content?keyword= — 내용 검색
- PUT /{id} — 수정 (multipart/form-data)
- DELETE /{id} — 삭제

---

## 1) 팁 생성

- 메서드/경로: POST /api/tip-info
- 요청: multipart/form-data { title, content, image? }
- 응답: TipInfoResponseDto

---

## 2) 조회/검색

- 단건: GET /api/tip-info/{id} → TipInfoResponseDto
- 전체: GET /api/tip-info → TipInfoResponseDto[]
- 최근: GET /api/tip-info/recent → TipInfoResponseDto[]
- 제목 검색: GET /api/tip-info/search/title?keyword=...
- 내용 검색: GET /api/tip-info/search/content?keyword=...

Axios 예시(검색)

```ts
import axios from "axios";
export async function searchTipsByContent(keyword: string) {
  const res = await axios.get(`${process.env.API_BASE_URL}/api/tip-info/search/content`, { params: { keyword } });
  return res.data; // TipInfoResponseDto[]
}
```

---

## 3) 수정/삭제

- 수정: PUT /api/tip-info/{id} (multipart/form-data)
- 삭제: DELETE /api/tip-info/{id}

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
