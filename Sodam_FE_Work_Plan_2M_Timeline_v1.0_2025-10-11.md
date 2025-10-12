# Sodam Front End 작업 목표 및 팀 협의 계획 (2개월 타임라인)

## 📋 Overview
- **작성일**: 2025-10-11
- **작성자**: Junie (Flow Analyst / React Native Engineer / Release Coordinator)
- **문서 유형**: Work Plan / Team Coordination Strategy
- **타임라인**: 2개월 (Week 1-8, ~2025-12-11)
- **목표**: 프로젝트 완성을 위한 단기/중기/장기 작업목표 수립 및 팀 간 협의 프레임워크 정의

## 🎯 목적
프로젝트 완성 기한까지 2개월이 남은 시점에서, 팀원들의 성취감을 극대화할 수 있도록 기능 단위와 스크린 단위로 작업을 분해하고, 각 팀과의 협의 포인트를 명확히 하여 효율적인 프로젝트 완성을 달성합니다.

## 📊 현재 상태 요약 (As-Is Analysis)

### 완료된 항목 ✅
- **API 통합**: Phase 0-2 완료 (Attendance, Wage/Payroll, Store, QnA, TimeOff, User, Master 서비스)
- **네비게이션**: 3개 네비게이터 완성 (App, Auth, Home), 19개 스크린 통합
- **역할 기반 라우팅**: EMPLOYEE, MASTER, MANAGER, Personal 4개 역할 지원
- **인증**: Kakao OAuth + JWT 완료
- **핵심 화면**: Login, Signup, AttendanceScreen, MyPage(4종), StoreRegistration 완료

### 미완성/갭 항목 ❌
- **미통합 스크린**: 5개 (InfoListScreen, SalaryListScreen, WorkplaceList/Detail, HybridMainScreen)
- **테스트 품질**: 전체 테스트 스위트 불안정 (curated suites PASS, 전체 실행 실패)
- **CI/CD**: 스캐너 CI 통합 미완, 자동화 가드레일 부재
- **API 엔드포인트**: 불일치 이슈 (attendance verify 경로 혼재)
- **UI 훅 미연결**: Wage/Payroll 서비스는 완성되었으나 UI 훅/스크린 미연결
- **문서 갭**: 문서 인덱스 미링크, 신규 개발자 온보딩 가이드 부재
- **디자인 일관성**: 디자인 시스템 가이드 존재하나 전체 화면 적용 미완

### 통계 요약
| 항목 | 계획 | 완료 | 진행률 |
|------|------|------|--------|
| 총 스크린 수 | 60+ | 19 | 32% |
| API 서비스 통합 | 8개 도메인 | 8개 | 100% |
| 네비게이션 통합 | 24개 | 19개 | 79% |
| 테스트 커버리지 | ≥70% | ~40% (전체) | 57% |
| 디자인 시스템 적용 | 전체 | 일부 | ~30% |

---

## 🎯 작업 우선순위 매트릭스

### 비즈니스 임팩트 vs 구현 복잡도

```
높은 임팩트 / 낮은 복잡도 (Quick Wins - 단기 우선)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Attendance 엔드포인트 표준화
✓ Salary UI 훅 연결 및 기본 화면 통합
✓ InfoListScreen 네비게이션 통합
✓ 테스트 스위트 안정화 (핵심만)
✓ CI 스캐너 통합

높은 임팩트 / 높은 복잡도 (전략적 - 중기)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Salary 기능 완전 구현 (리스트, 상세, 차트)
✓ Workplace 관리 기능 구현
✓ 디자인 시스템 전체 적용
✓ E2E 테스트 자동화

낮은 임팩트 / 낮은 복잡도 (쉬운 개선 - 중장기)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 문서 인덱스 링크 정리
✓ HybridMainScreen 실험 평가 및 결정
✓ 설정 화면 세부 기능 추가

낮은 임팩트 / 높은 복잡도 (보류/장기)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 완전한 오프라인 지원
✓ 고급 통계 대시보드
✓ 다국어 지원
```

---

## 📅 단기/중기/장기 목표 (기능/스크린 단위)

### 🚀 단기 목표 (Week 1-2): Critical 문제 해결 + 핵심 안정화

#### Week 1 Sprint
**목표**: Critical 문제 즉시 수정 및 안정성 확보
**진척도**: 4/7 완료 (57%) ✅

1. **[CRITICAL-001] StoreDetailScreen 라우트 추가** (0.5일) ✅ **완료 (2025-10-12)**
   - 담당: RN Engineer
   - 작업: HomeNavigator에 StoreDetailScreen 라우트 추가, 타입 정의 업데이트
   - 산출물: HomeNavigator.tsx, StoreDetailScreen.tsx (신규)
   - AC: MasterMyPage에서 매장 카드 탭 시 크래시 없이 상세 화면 이동
   - 성취감: "매장 상세 크래시 해결! 🚨→✅"

2. **[CRITICAL-002] ManagerMyPageScreen 복구** (1일) ✅ **완료 (2025-10-12)**
   - 담당: RN Engineer
   - 작업: 파일 인코딩 확인, 손상 시 Employee/Master 참고하여 재작성
   - 산출물: ManagerMyPageScreen.tsx 완전 재작성 (346줄)
   - AC: Manager 역할 사용자 로그인 시 MyPage 정상 렌더링
   - 성취감: "Manager 화면 복구 완료! 🔧"

3. **[HIGH-001] MasterMyPageScreen 서비스 연동** (1-2일) ✅ **완료 (2025-10-12)**
   - 담당: RN Engineer + Backend Engineer
   - 작업: storeService.getMasterStores() 연동, 빈 상태 처리
   - 산출물: MasterMyPageScreen.tsx 수정, mockStores 제거 (51줄 삭제)
   - AC: 실제 매장 데이터 표시, 매장 없을 때 "매장 추가" 안내
   - 성취감: "사장 화면 실데이터 연동! 🏪"

4. **[HIGH-002] 노무 정보 서비스 연동** (1일) ✅ **완료 (2025-10-12)**
   - 담당: RN Engineer
   - 작업: laborInfoService 구현/연동, Employee/Master 화면 적용
   - 산출물: laborInfoService.ts (신규), EmployeeMyPageRNScreen.tsx, MasterMyPageScreen.tsx 수정
   - AC: 하드코딩된 minimumWage 제거, 실시간 노무 정보 표시
   - 성취감: "노무 정보 실시간 업데이트! 📋"

5. **[F-001] Attendance 엔드포인트 표준화** (1-2일)
   - 담당: FE Lead + Backend Engineer
   - 작업: `/api/attendance/nfc-verify` → `/api/attendance/verify/nfc` 통일
   - 작업: `/attendance/location-verify` → `/api/attendance/verify/location` 통일
   - 산출물: attendanceService.ts 수정, 백엔드 라우트 조정
   - AC: 모든 verify 엔드포인트가 `/api/attendance/verify/*` 패턴 준수
   - 성취감: "출퇴근 API 완전 정리 완료! 🎉"

2. **[T-001] CI 스캐너 통합** (1일)
   - 담당: QA Specialist
   - 작업: CI 파이프라인에 `scan-qr-residue.ps1 -FailOnMatch` 추가
   - 산출물: .github/workflows 또는 CI 스크립트 업데이트
   - AC: CI 빌드 시 disallowed keywords 자동 검출 및 실패
   - 성취감: "자동화 가드레일 완성! 🛡️"

3. **[T-002] Jest 스위트 핵심 안정화** (2-3일)
   - 담당: QA + RN Engineer
   - 작업: 실패하는 테스트 파일 식별 및 mock 보완
   - 범위: Attendance, Wage, Store, Auth 서비스 테스트만 우선
   - AC: `npm test -- <curated-tests>` 100% PASS
   - 성취감: "핵심 테스트 그린 라이트! ✅"

#### Week 2 Sprint
**목표**: PersonalUser 연동 + UI 레이어 긴급 통합

6. **[CRITICAL-003] PersonalUserScreen API 연동** (3-4일) ⛔
   - 담당: RN Engineer + Backend Engineer
   - 작업: storeService 연동 (다중 매장 조회), attendanceService 연동 (CRUD)
   - 작업: 로컬 상태 → 백엔드 동기화 전환, workSessions/allRecords 영속화
   - 산출물: PersonalUserScreen.tsx 대폭 수정, 하드코딩 stores 제거
   - AC: 매장 데이터 백엔드 조회, 출퇴근 기록 영속화, 앱 재시작 후에도 데이터 유지
   - 성취감: "개인 사용자 모드 완전 연동! 🎯"

7. **[S-001] InfoListScreen 네비게이션 통합** (1일)
   - 담당: RN Engineer
   - 작업: HomeNavigator에 InfoList 라우트 추가, HomeScreen에서 진입점 연결
   - AC: HomeScreen → InfoList → Detail 플로우 동작
   - 성취감: "정보 서비스 완전 개통! 📚"

5. **[S-002] SalaryListScreen 기본 연결** (2일)
   - 담당: RN Engineer
   - 작업: payrollService 기반 useSalaryList 훅 생성, 기본 리스트 UI 구현
   - AC: MyPage에서 급여 조회 버튼 → SalaryList 화면 이동 및 데이터 표시
   - 성취감: "급여 조회 화면 오픈! 💰"

6. **[D-001] 문서 인덱스 정리** (0.5일)
   - 담당: Documentation Lead
   - 작업: docs/README.md 생성 및 모든 Phase 보고서 링크 추가
   - AC: docs/README.md에서 모든 주요 문서 접근 가능
   - 성취감: "문서 포털 완성! 📖"

---

### 🎯 중기 목표 (Week 3-6): 미통합 기능 완성 + UX 개선

#### Week 3-4 Sprint
**목표**: Salary 기능 완전 구현

7. **[F-002] Salary 상세 화면 구현** (3일)
   - 담당: RN Engineer + Designer
   - 작업: SalaryDetailScreen, 급여명세서 PDF 뷰어, 차트 통합
   - AC: 급여 리스트 → 상세 → 차트/PDF 다운로드 플로우 완성
   - 성취감: "급여 기능 완전 정복! 📊"

8. **[F-003] Workplace 관리 기능 구현** (4일)
   - 담당: RN Engineer + Backend Engineer
   - 작업: WorkplaceList, WorkplaceDetail 화면, workplaceService 완성
   - AC: Master 사용자가 여러 매장 등록/관리 가능
   - 성취감: "다중 매장 관리 시스템 완성! 🏢"

9. **[S-003] Attendance UI 개선** (2일)
   - 담당: RN Engineer + Designer
   - 작업: NFC 인식 UI 개선, 위치 인증 피드백 강화
   - AC: 출퇴근 처리 시 명확한 시각적 피드백 및 에러 안내
   - 성취감: "출퇴근 UX 대폭 개선! 📱"

#### Week 5-6 Sprint
**목표**: 디자인 일관성 및 테스트 확대

10. **[D-002] 디자인 시스템 전체 적용** (5일)
    - 담당: RN Engineer + Designer
    - 작업: 공통 디자인 가이드 기반 전체 화면 스타일 통일
    - 범위: Auth, MyPage, Attendance, Salary, Info 스크린
    - AC: 모든 주요 화면이 COLORS, 그라디언트, spacing 가이드 준수
    - 성취감: "앱 디자인 완전 통일! 🎨"

11. **[T-003] 전체 Jest 스위트 안정화** (3일)
    - 담당: QA + RN Engineer
    - 작업: 모든 테스트 파일 수정 및 mock 보완
    - AC: `npm test -- --coverage` PASS, 커버리지 ≥70%
    - 성취감: "전체 테스트 그린 라이트! 🟢"

12. **[T-004] E2E 시나리오 자동화** (2일)
    - 담당: QA Specialist
    - 작업: Detox/Appium 기반 핵심 플로우 자동화 (Login → Attendance → Logout)
    - AC: CI에서 E2E 테스트 자동 실행 및 결과 보고
    - 성취감: "E2E 자동화 완성! 🤖"

---

### 🏁 장기 목표 (Week 7-8): 배포 준비 + 최종 검증

#### Week 7 Sprint
**목표**: 배포 전 최종 점검

13. **[Q-001] 전수 품질 검증** (3일)
    - 담당: QA Lead + 전체 팀
    - 작업: 모든 스크린/기능 수동 테스트, 버그 트리아지
    - AC: Critical/High 버그 0건, Medium 버그 리스트화
    - 성취감: "품질 검증 완료! ✅"

14. **[D-003] 사용자 가이드 작성** (2일)
    - 담당: Product + Documentation
    - 작업: 최종 사용자용 기능 가이드, FAQ, 스크린샷
    - AC: docs/user-guide/ 생성 및 모든 주요 기능 설명
    - 성취감: "사용자 가이드 완성! 📘"

#### Week 8 Sprint
**목표**: 배포 및 모니터링 준비

15. **[R-001] 프로덕션 빌드 최적화** (2일)
    - 담당: RN Lead + DevOps
    - 작업: Bundle size 최적화, Hermes 최적화, 성능 프로파일링
    - AC: Android APK <50MB, 앱 시작 시간 <2초
    - 성취감: "빌드 최적화 완료! ⚡"

16. **[R-002] 모니터링 및 로깅 구축** (2일)
    - 담당: Backend + DevOps
    - 작업: Crashlytics, Analytics, 에러 로깅 통합
    - AC: 실시간 크래시/에러 모니터링 대시보드 구축
    - 성취감: "모니터링 시스템 가동! 📈"

17. **[R-003] 최종 배포 및 검증** (1일)
    - 담당: 전체 팀
    - 작업: 스테이징 배포 → 검증 → 프로덕션 배포
    - AC: 프로덕션 환경에서 모든 핵심 기능 정상 동작
    - 성취감: "프로젝트 완성! 🎉🚀"

---

## 👥 팀별 협의 포인트 및 프로세스

### Backend 팀과의 협의

#### 협의 주기
- **정기 미팅**: 매주 월/목 오전 10시 (30분)
- **긴급 협의**: Slack #fe-be-sync 채널 실시간

#### 협의 항목

| 항목 | 우선순위 | 담당자 | 협의 포인트 | 예상 일정 |
|------|----------|--------|-------------|-----------|
| **API-001** Attendance 엔드포인트 표준화 | High | FE Lead + BE Engineer | verify 엔드포인트 경로 통일, 응답 스키마 표준화 | Week 1 |
| **API-002** Workplace API 스펙 확정 | High | FE + BE | CRUD 엔드포인트, 매장-직원 관계 스키마 | Week 3 |
| **API-003** Salary API 응답 구조 확정 | Medium | FE + BE | 급여명세서 데이터 포맷, PDF 생성 로직 | Week 3 |
| **API-004** 인증/권한 표준화 | High | FE + BE + Security | JWT refresh 로직, 역할 기반 권한 체계 | Week 2 |
| **API-005** 에러 코드 표준화 | Medium | FE + BE | 통일된 에러 응답 포맷, 에러 코드 매핑 | Week 4 |

#### 협의 프로세스
1. FE Lead가 Slack에 협의 요청 + 구체적 이슈 명시
2. BE Engineer 응답 및 회의 일정 조율
3. 회의 후 결정사항을 docs/api-decisions/ 에 문서화
4. 양측 구현 완료 후 통합 테스트

---

### QA 팀과의 협의

#### 협의 주기
- **정기 미팅**: 매주 화/금 오후 3시 (30분)
- **긴급 버그 리뷰**: 일일 스탠드업 (오전 9:30)

#### 협의 항목

| 항목 | 우선순위 | 담당자 | 협의 포인트 | 예상 일정 |
|------|----------|--------|-------------|-----------|
| **QA-001** CI 통합 전략 | High | QA + DevOps | 스캐너, 테스트 자동화, 빌드 검증 | Week 1 |
| **QA-002** 테스트 우선순위 | High | QA + FE | 핵심 플로우 식별, 회귀 테스트 범위 | Week 2 |
| **QA-003** E2E 시나리오 정의 | Medium | QA + Product | 사용자 시나리오 기반 자동화 범위 | Week 5 |
| **QA-004** 버그 트리아지 프로세스 | Medium | QA + 전체 | 버그 우선순위 기준, 수정 일정 조율 | Week 3 |
| **QA-005** 성능 테스트 | Low | QA + DevOps | 부하 테스트, 응답 시간 벤치마크 | Week 7 |

#### 테스트 전략
- **Unit Tests**: 각 기능 개발 시 동시 작성 (FE)
- **Integration Tests**: API 연동 완료 시 검증 (FE + QA)
- **E2E Tests**: 주요 플로우만 자동화 (QA)
- **Manual Tests**: 전수 검증 (QA, Week 7)

---

### Design 팀과의 협의

#### 협의 주기
- **정기 미팅**: 매주 수요일 오후 2시 (1시간)
- **디자인 리뷰**: 신규 화면 개발 전 사전 협의

#### 협의 항목

| 항목 | 우선순위 | 담당자 | 협의 포인트 | 예상 일정 |
|------|----------|--------|-------------|-----------|
| **DES-001** 디자인 시스템 확정 | High | Designer + FE | 색상, 타이포, 컴포넌트 라이브러리 | Week 2 |
| **DES-002** Salary 화면 디자인 | High | Designer + FE | 리스트, 상세, 차트 레이아웃 | Week 3 |
| **DES-003** Workplace 화면 디자인 | Medium | Designer + FE | 매장 관리 UI/UX | Week 3 |
| **DES-004** Attendance UX 개선 | Medium | Designer + FE | NFC 인식, 위치 인증 피드백 | Week 4 |
| **DES-005** 전체 화면 일관성 검토 | High | Designer + FE | 브랜드 일관성, 접근성 | Week 5-6 |

#### 디자인 워크플로우
1. Designer가 Figma에 디자인 업로드
2. FE Lead 리뷰 및 구현 가능성 피드백
3. 최종 승인 후 Figma 링크를 GitHub Issue에 첨부
4. FE 구현 완료 후 Designer 최종 검수

---

### Product 팀과의 협의

#### 협의 주기
- **정기 미팅**: 매주 월요일 오전 11시 (1시간, 전체 스프린트 리뷰)
- **우선순위 조정**: 격주 목요일 오후 (30분)

#### 협의 항목

| 항목 | 우선순위 | 담당자 | 협의 포인트 | 예상 일정 |
|------|----------|--------|-------------|-----------|
| **PRD-001** MVP 범위 최종 확정 | Critical | Product + 전체 | 2개월 내 구현 가능한 최소 기능 세트 | Week 1 |
| **PRD-002** 기능 우선순위 조정 | High | Product + FE Lead | 비즈니스 임팩트 기반 재조정 | Week 2, 4, 6 |
| **PRD-003** 수락 기준 명확화 | High | Product + QA | 각 기능별 AC 구체화 | Ongoing |
| **PRD-004** 사용자 시나리오 검증 | Medium | Product + QA | 핵심 플로우 사용자 관점 검증 | Week 5 |
| **PRD-005** 런칭 체크리스트 | High | Product + 전체 | 배포 전 필수 점검 항목 | Week 7 |

#### 의사결정 프로세스
1. Product Owner가 기능 요구사항 제시
2. FE/BE Lead가 기술적 실현 가능성 및 공수 산정
3. 전체 팀 회의에서 우선순위 합의
4. GitHub Projects 보드에 반영 및 스프린트 할당

---

## 📊 진척도 추적 및 KPI

### 주간 진척도 지표

| KPI | 목표 | 측정 방법 | 책임자 |
|-----|------|-----------|--------|
| **완료된 기능 수** | 주당 2-3개 | GitHub Issues Closed | FE Lead |
| **통합된 스크린 수** | Week 8까지 +10개 | Navigation 통합 확인 | RN Engineer |
| **테스트 커버리지** | 70% 이상 | Jest coverage report | QA |
| **Critical 버그 수** | 0건 유지 | GitHub Issues (Critical) | QA Lead |
| **CI 빌드 성공률** | 95% 이상 | CI 대시보드 | DevOps |

### 성취감 극대화 전략

#### 1. 기능 단위 분해
- 큰 기능을 1-2일 내 완료 가능한 작은 단위로 분해
- 각 기능 완료 시 팀 Slack에 "🎉 [기능명] 완료!" 공유

#### 2. 스크린 단위 체크리스트
- 각 스크린을 독립적인 마일스톤으로 관리
- 스크린 완료 시 GitHub Projects 보드에서 "Done" 이동

#### 3. 주간 데모 세션
- 매주 금요일 오후 4시: 완성된 기능 데모
- 팀원들이 돌아가며 자신의 작업 시연

#### 4. 시각적 진척도 보드
- docs/ 에 progress.md 생성, 주간 업데이트
- 완료된 항목에 ✅ 표시 및 축하 이모지

---

## 🎯 루브릭 (문서 품질 기준)

### 자체 검증 기준
- ✅ **정확성**: 모든 현황 데이터가 실제 코드베이스/문서와 일치
- ✅ **논리적 일관성**: 단기→중기→장기 목표가 시간적으로 연속적이고 의존성 고려됨
- ✅ **창의성**: 기능/스크린 단위 분해로 성취감 극대화 구조 설계
- ✅ **간결성**: 2개월 타임라인에 맞춘 현실적 범위 (17개 주요 작업)
- ✅ **사용자 친화성**: 각 작업마다 담당자, AC, 예상 일정, "성취감" 메시지 포함
- ✅ **사실 검증**: Phase Checklist, Navigation Flow, Attendance AsIs 기반 작성

**자체 검증 결과**: PASS (1차 시도)

---

## 📢 2025-10-12 업데이트: Week 1 Sprint 4개 항목 완료 🎉

### 이번 세션 성과 (Task #2025-10-12-01)

**완료 항목**: 4/7 (Week 1 Sprint 57% 완료)

1. ✅ **[CRITICAL-001] StoreDetailScreen 라우트 추가**
   - HomeNavigator에 StoreDetail 라우트 등록
   - StoreDetailScreen.tsx 신규 생성 (250줄)
   - MasterMyPage 매장 카드 탭 시 크래시 제거

2. ✅ **[CRITICAL-002] ManagerMyPageScreen 복구**
   - 완전 재작성 (346줄, TypeScript)
   - RoleSlots 패턴 적용 (HeroSlot, SummarySlot, ActionsSlot, InfoSlot)
   - TeamMember, PendingApproval 인터페이스 정의

3. ✅ **[HIGH-001] MasterMyPageScreen 서비스 연동**
   - storeService.getMasterStores(userId) API 통합
   - Mock 데이터 완전 제거 (51줄 삭제)
   - 실제 매장 데이터 표시, 빈 상태 처리

4. ✅ **[HIGH-002] 노무 정보 서비스 연동**
   - laborInfoService.getCurrentLaborInfo() 구현 및 통합
   - MasterMyPage, EmployeeMyPage 모두 실시간 노무 정보 표시
   - Mock 하드코딩 제거

### 변경 통계

- **총 변경 파일**: 6개 (신규 2개, 수정 4개)
- **코드 순증**: +261줄 (+436줄 추가, -175줄 삭제)
- **API 연동**: 3개 엔드포인트 (Store Master, Store Detail, LaborInfo Current)

### 다음 우선순위 (Week 1-2 나머지)

1. 🔥 **[F-001] Attendance 엔드포인트 표준화** (High Priority)
   - `/api/attendance/nfc-verify` → `/api/attendance/verify/nfc`
   - `/attendance/location-verify` → `/api/attendance/verify/location`

2. 📱 **[S-001] InfoListScreen 네비게이션 통합**
   - HomeNavigator에 InfoList 라우트 추가

3. 💰 **[S-002] SalaryListScreen 기본 연결**
   - payrollService 기반 useSalaryList 훅 생성

4. 🧪 **[T-002] Jest 스위트 핵심 안정화**
   - Attendance, Wage, Store, Auth 테스트 우선

### 관련 문서

- **작업 계획서**: [작업계획서_FE_BE_통합_구현_v1.0_2025-10-12.md](작업계획서_FE_BE_통합_구현_v1.0_2025-10-12.md)
- **작업 완료 보고서**: [작업완료_보고서_FE_BE_통합_v1.0_2025-10-12.md](../reports/작업완료_보고서_FE_BE_통합_v1.0_2025-10-12.md)
- **Master Task Log**: [Master_Task_Log.md](../../Master_Task_Log.md#task-2025-10-12-01)
- **Backend 보고서**: [BE_Implementation_Status_Report_v1.0_2025-10-11.md](../BackEnd_Report_API/BE_Implementation_Status_Report_v1.0_2025-10-11.md)

---

## 📅 Change History

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| 2025-10-11 | 1.0 | Initial work plan with 2-month timeline | Junie |
| 2025-10-11 | 1.1 | Added 5 Critical/High priority tasks from role-based audit (StoreDetail route, Manager recovery, PersonalUser API, Master service, Labor info) | Junie |
| 2025-10-12 | 1.2 | Week 1 Sprint 4개 항목 완료 표시 (CRITICAL-001, CRITICAL-002, HIGH-001, HIGH-002), 진척도 업데이트 (4/7 완료), 다음 우선순위 명시 | Junie |
