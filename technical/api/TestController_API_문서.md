# TestController — 표준 응답/예외 테스트(내부) API

## 📋 개요

- 작성일: 2025-09-30
- 작성자: Backend Team
- 문서 유형: 내부 테스트 가이드
- 관련 이슈: RN 연동용 컨트롤러별 API 문서화 (내부 품질 점검용)

## 🎯 목적

백엔드 표준 ApiResponse 포맷과 글로벌 예외 처리 동작을 확인하기 위한 내부 테스트 엔드포인트를 문서화합니다. 운영에 노출하지 않는 것을 권장합니다.

## 🔐 인증/헤더

- 인증: 불필요(테스트용)
- 베이스 경로: /api/test

## 📚 엔드포인트 요약

- GET /success — 성공 응답 샘플
- GET /illegal-argument?value= — IllegalArgumentException 발생 테스트
- GET /no-such-element?id= — NoSuchElementException 발생 테스트
- GET /general-exception — 일반 Exception 발생 테스트

---

## 1) 성공 응답 테스트

- 메서드/경로: GET /api/test/success
- 응답: ApiResponse<String> { success: true, message: "테스트 성공", data: "성공적으로 처리되었습니다." }

예시

```
curl "{HOST}/api/test/success"
```

---

## 2) IllegalArgumentException 테스트

- 메서드/경로: GET /api/test/illegal-argument?value=error
- 동작: value가 "error"이면 IllegalArgumentException 발생
- 기대 응답: 글로벌 예외 처리에 따른 표준 오류 응답

---

## 3) NoSuchElementException 테스트

- 메서드/경로: GET /api/test/no-such-element?id=999
- 동작: id=999일 때 NoSuchElementException 발생
- 기대 응답: 표준 오류 응답

---

## 4) 일반 Exception 테스트

- 메서드/경로: GET /api/test/general-exception
- 동작: RuntimeException 강제 발생
- 기대 응답: 표준 오류 응답

## 주의 사항

- 본 엔드포인트는 내부 검증 및 데모용입니다. 운영 환경 노출을 지양하세요.

## 📅 변경 이력

 날짜         | 버전  | 변경 내용 | 작성자          
------------|-----|-------|--------------
 2025-09-30 | 1.0 | 초기 작성 | Backend Team 
