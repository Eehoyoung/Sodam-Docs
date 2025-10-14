# Phase Plan Compliance Audit — Final Checklist & Gap Work Plan (v1.0)

## 📋 Overview

- Created: 2025-10-05 19:33 KST
- Owner/Role: Junie — Release Coordinator / QA Specialist / React Native Engineer / Document Consistency Verifier
- Repository: Sodam_Front_End (RN 0.81.0, React 19.1.0)
- Purpose: Verify that each Phase (0–3) followed the plan; publish a final completion checklist; if gaps exist, output
  an actionable follow-up work plan.
- Source Plans/Reports:
    - docs\api\Sodam_API_Phase_Plan_Detailed_v1.0_2025-10-02.md
    - docs\api\Phase_0_Attendance_Standardization_Report_2025-10-02.md
    - docs\api\Phase_1_Wage_Payroll_Store_Integration_Report_2025-10-02.md
    - docs\api\Phase_2_QnA_TimeOff_Master_User_Integration_Report_2025-10-03.md
    - docs\api\Phase_3_Automation_Guardrails_Kickoff_Report_2025-10-03.md
    - docs\api\Phase_3_Double_Check_And_Coverage_Fix_Report_2025-10-03.md
    - docs\api\Session_Mid_Report_Full_Trace_2025-10-02.md

## 🧭 Environment / Constraints

- OS/Shell: Windows + PowerShell (paths use backslashes \\\)
- RN/React/Node: React Native 0.81.0 (New Architecture, Hermes) / React 19.1.0 / Node >= 18
- Android: Compile/Target SDK 36, Min 24
- Testing: Jest preset 'react-native', jest.setup.js present
- Product policy: NFC-only attendance; QR residual scanner active; initial route fixed to Welcome
- Principle: Change Scale Decision Framework — choose appropriate change scope

## ✅ Rubric (Quality Gate for this audit)

- Accuracy: Status and evidence must reflect current repo state (paths, tests, scripts). PASS if all items link to
  actual artifacts.
- Logical Consistency: Phase goals ↔ implemented changes ↔ tests ↔ docs align. PASS if no contradictions.
- Completeness: All phases audited; final checklist covers ACs; gaps enumerated with plans. PASS if all phases present.
- Developer-Friendliness: Clear paths, commands, owners, AC for follow-ups. PASS if next steps are actionable.
- Security/Policy Compliance: NFC-only policy, scanner guardrails respected. PASS if scanner reports PASS and allowlist
  is honored.
- Fact Verification: Verified against docs and curated Jest/Scanner outputs shown in session history. PASS if
  discrepancies are noted.

Self-Validation (Attempt 1 of max 3): PASS — If reviewers find gaps, iterate up to 3 times; uncertainties are noted in
the Gaps section.

---

## Phase-by-Phase Compliance Checklist

### Phase 0 — Attendance 표준화 전환

- Plan Goals: Replace legacy /location-verify, /nfc-verify with POST /api/attendance/check-in|check-out; add tests;
  update scanner disallow list.
- Evidence:
    - Code: src\features\attendance\services\attendanceService.ts — [API Mapping] checkIn → POST
      /api/attendance/check-in; checkOutStandard → POST /api/attendance/check-out; legacy mapping removed.
    - Scanner: scripts\scan-qr-residue.ps1 includes disallowed keywords; latest run PASS (logs\qr-scan-report.md).
    - Tests: __tests__\attendance\attendanceService.test.ts asserts endpoint/payload mapping.
- Status: MET

### Phase 1 — 임금/급여/스토어 정합

- Plan Goals: Wage/Payroll services; Store putLocation/changeOwner; tests.
- Evidence:
    - Wage: src\features\wage\services\wageService.ts (service), __tests__\wage\wageService.test.ts (PASS in curated
      runs).
    - Payroll: src\features\salary\services\payrollService.ts, __tests__\salary\payrollService.test.ts (PASS curated).
    - Store: src\features\store\services\storeService.ts with [API Mapping]; __tests__\store\storeService.test.ts (PASS
      curated).
- Status: MET

### Phase 2 — 정보/문의/휴가/유저

- Plan Goals: QnA, TimeOff, User, Master services callable from RN with tests.
- Evidence:
    - QnA: src\features\qna\services\qnaService.ts; __tests__\qna\qnaService.test.ts (multipart polyfill; PASS curated).
    - TimeOff: src\features\myPage\services\timeOffService.ts; __tests__\myPage\timeOffService.test.ts (PASS curated).
    - Master: src\features\myPage\services\masterService.ts; __tests__\myPage\masterService.test.ts (PASS curated).
    - User: src\features\auth\services\userService.ts; __tests__\auth\userService.test.ts (PASS curated).
- Status: MET

### Phase 3 — 자동화/가드레일

- Plan Goals: Extend scanner; integrate into CI; ensure >=70% coverage; provide verification tooling.
- Evidence:
    - Scanner: scripts\scan-qr-residue.ps1 updated; local runs PASS with -FailOnMatch (report generated).
    - Verification Script: scripts\run-phase3-verification.ps1 aggregates scanner + curated tests;
      logs\phase3-verification-report.md is produced on run.
    - Coverage: jest.config.js collects coverage for target services; curated suites show >90% statements coverage on
      targeted scope.
    - CI Integration: Not yet wired; local-only at this time.
    - Full Suite Stability: Global `npm test -- --coverage` reported multiple failures unrelated to Phase services (
      assertion/teardown), while curated Phase suites PASS.
- Status: PARTIALLY MET — CI integration and full-suite stabilization remain outstanding.

---

## Final Completion Checklist (Global)

- [✔] All API domains in scope are callable from RN through service modules with [API Mapping] comments.
- [✔] Curated Phase service tests PASS locally; targeted coverage ≥70% (actually >90% on targeted set).
- [✔] Disallowed endpoints scanner updated; local scanner PASS with -FailOnMatch; report stored in logs.
- [✖] CI pipeline integration of scanner (-FailOnMatch) — NOT MET (pending).
- [✖] Full test suite (`npm test -- --coverage`) green — NOT MET (known unrelated suite failures; needs triage/fix).
- [△] Minimal UI hooks/screens for Wage/Payroll behind feature toggles — PARTIAL (planned; not required for API
  callability).
- [△] Attendance edge-case tests (conversion/error mapping/verify flows) — PARTIAL (basic mapping tests exist).
- [△] Docs index links to all new reports — PARTIAL.

Notes:

- The acceptance criteria for this issue focus on plan adherence and checklists. The API callability goal is achieved;
  remaining items are quality/automation enhancements to be executed next.

---

## Gaps Identified & Additional Work Plan

### GAP-1 — CI Scanner Integration

- Problem: Scanner not wired into CI; only local script exists.
- Actions:
    1) Add CI step to run `powershell -ExecutionPolicy Bypass -File .\scripts\scan-qr-residue.ps1 -FailOnMatch` before
       tests.
    2) Publish logs\qr-scan-report.md as pipeline artifact; fail build on matches.
- Owner: QA / RN Lead
- AC: CI fails on disallowed keywords outside allowlist; artifact present in CI run.
- ETA: 1 day
- Rollback: Disable CI step temporarily via pipeline variable if causing false positives; adjust allowlist.

### GAP-2 — Global Jest Suite Stabilization

- Problem: `npm test -- --coverage` shows multiple failing suites (assertions/teardown). Curated Phase suites PASS.
- Actions:
    1) Identify failing test files and categorize causes (missing mocks for RN native modules; lingering timers;
       environment mismatch).
    2) Add/adjust mocks in jest.setup.js; ensure `react-native-reanimated`, `react-native-gesture-handler`, and NFC
       Manager mocks are correct.
    3) Fix improper teardown (ensure `jest.useFakeTimers().setSystemTime/resetAllMocks/clearAllTimers` usage is
       balanced; add afterEach cleanup).
    4) Re-run full suite until green; ensure coverage ≥70% global per jest.config.js.
- Owner: QA / RN Engineer(s)
- AC: `npm test -- --coverage` PASS; global coverage ≥70%.
- ETA: 2–3 days
- Rollback: Temporarily skip flaky tests with TODO tags; open follow-up tasks to re-enable.

### GAP-3 — Attendance Edge-Case & Verify Tests

- Problem: Only mapping tests exist; need edge cases for invalid storeId, 401 refresh flow, error code mapping, verify
  flows.
- Actions: Add tests to __tests__\attendance\ with these scenarios; mock api.ts interceptors as needed.
- Owner: QA / RN
- AC: New tests PASS; branch coverage for attendanceService increases by ≥10%.
- ETA: 1–2 days

### GAP-4 — Minimal UI Hooks for Wage/Payroll

- Problem: Services not yet wired into UI (behind feature toggles).
- Actions: Create hooks (useWage, usePayroll) and wire to placeholder screens with hidden entry; add smoke tests.
- Owner: RN Wage/Payroll
- AC: Hooks/screens render and call services (mocked) in smoke tests; behind feature flags.
- ETA: 2 days

### GAP-5 — Documentation Index Linking

- Problem: New reports not linked from docs index/overview.
- Actions: Update docs index to reference all new Phase reports and this checklist.
- Owner: Release Coordinator / Docs
- AC: docs index updated and committed.
- ETA: 0.5 day

---

## How to Verify (Windows PowerShell)

- Scanner (local):
    - powershell -ExecutionPolicy Bypass -File .\scripts\scan-qr-residue.ps1 -FailOnMatch
- Curated Phase suites (with coverage on targeted files):
    - npx jest --coverage __tests__/store/storeService.test.ts __tests__/qna/qnaService.test.ts __tests__
      /myPage/timeOffService.test.ts __tests__/myPage/masterService.test.ts __tests__/auth/userService.test.ts __tests__
      /attendance/attendanceService.test.ts __tests__/wage/wageService.test.ts __tests__/salary/payrollService.test.ts
- Full suite (expected to fail currently; triage under GAP-2):
    - npm test -- --coverage

---

## Risks & Notes

- Backend variations in response wrappers are handled defensively in services.
- Scanner allowlist limited to docs/scripts/lock files. False positives require allowlist adjustments.
- CI environment PowerShell availability assumed; if not, use Node runner to invoke PowerShell.

---

## 📅 Change History

 Date       | Version | Changes                                              | Author |
------------|---------|------------------------------------------------------|--------|
 2025-10-05 | 1.0     | Initial final compliance checklist and gap work plan | Junie  |
