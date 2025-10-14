# LaborInfoController — 노무 정보 API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화

## 🎯 목적

노무 정보(Labor Info)에 대한 생성/조회/수정/삭제 및 최근/전체 조회 기능을 RN 앱에서 바로 연동할 수 있도록 상세 스펙을 제공합니다.

## 🔐 인증/헤더

- 인증: Bearer JWT 권장(운영 정책에 따라 일부 엔드포인트 공개 가능)
- 공통 헤더
    - Authorization: Bearer {accessToken}
    - Content-Type: application/json | multipart/form-data
- 업로드: 이미지 포함 시 multipart/form-data 사용

## 🌐 베이스 경로

- /api/labor-info

## 📚 엔드포인트 요약

- POST / — 노무 정보 생성 (multipart/form-data)
- GET /{id} — 단건 조회
- GET / — 전체 조회
- GET /recent — 최근 5개 조회
- PUT /{id} — 수정 (multipart/form-data)
- DELETE /{id} — 삭제

---

## 1) 노무 정보 생성

- 메서드/경로: POST /api/labor-info
- 요청 형식: multipart/form-data
    - title: string
    - content: string
    - image: file (선택)
- 응답: LaborInfoResponseDto
- 상태 코드: 200 | 400 | 500

예시 curl

```
curl -X POST "{HOST}/api/labor-info" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -F "title=주휴수당 기준" \
  -F "content=주 15시간 이상 근로 시 주휴수당 발생" \
  -F "image=@/path/image.png"
```

Axios 예시 (TypeScript)

```ts
import axios from "axios";

export async function createLaborInfo(accessToken: string, data: { title: string; content: string; image?: any }) {
  const form = new FormData();
  form.append("title", data.title);
  form.append("content", data.content);
  if (data.image) form.append("image", data.image);
  const res = await axios.post(`${process.env.API_BASE_URL}/api/labor-info`, form, {
    headers: { Authorization: `Bearer ${accessToken}`, "Content-Type": "multipart/form-data" },
  });
  return res.data; // LaborInfoResponseDto
}
```

---

## 2) 노무 정보 단건 조회

- 메서드/경로: GET /api/labor-info/{id}
- 응답: LaborInfoResponseDto
- 상태 코드: 200 | 404 | 500

예시

```
curl "{HOST}/api/labor-info/10" -H "Authorization: Bearer {ACCESS_TOKEN}"
```

---

## 3) 노무 정보 전체 조회

- 메서드/경로: GET /api/labor-info
- 응답: LaborInfoResponseDto[]
- 상태 코드: 200 | 500

---

## 4) 최근 노무 정보 5개

- 메서드/경로: GET /api/labor-info/recent
- 응답: LaborInfoResponseDto[]

---

## 5) 노무 정보 수정

- 메서드/경로: PUT /api/labor-info/{id}
- 요청 형식: multipart/form-data
    - title: string
    - content: string
    - image: file (선택, 교체 시 업로드)
- 응답: LaborInfoResponseDto

예시

```
curl -X PUT "{HOST}/api/labor-info/10" \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -F "title=업데이트된 주휴수당 기준" \
  -F "content=개정 내용 반영" \
  -F "image=@/path/new.png"
```

---

## 6) 노무 정보 삭제

- 메서드/경로: DELETE /api/labor-info/{id}
- 응답: 204 No Content

---

## 📦 DTO 개요

- 요청: LaborInfoRequestDto { title, content, image }
- 응답: LaborInfoResponseDto { id, title, content, imageUrl, createdAt, updatedAt 등 }

## 🚨 오류 코드

- 400: 잘못된 요청
- 401: 인증 실패(정책에 따라 필요 시)
- 404: 대상 없음
- 500: 서버 오류

## 🔗 관련 문서

- JWT 및 RN 연동 가이드: ../JWT_인증_API_및_RN_연동_가이드.md

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
