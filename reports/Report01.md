# 사장 마이페이지 기능 구현 보고서

## 1. 개요

본 보고서는 '소담' 애플리케이션의 사장 마이페이지 기능 구현에 관한 상세 내용을 담고 있습니다. 사장 마이페이지는 매장 운영자(사장)가 자신의 프로필 정보, 소유한 매장 목록, 통합 통계, 휴가 신청 내역 등을
한눈에 확인할 수 있는 중요한 기능입니다.

## 2. 시스템 아키텍처

사장 마이페이지 기능은 다음과 같은 계층 구조로 구현되었습니다:

```
[클라이언트 (React Native)] <-> [컨트롤러 계층] <-> [서비스 계층] <-> [레포지토리 계층] <-> [데이터베이스]
```

각 계층별 주요 컴포넌트:

- **컨트롤러 계층**: `MasterController` - API 엔드포인트 제공
- **서비스 계층**: `MasterProfileService`, `TimeOffService` - 비즈니스 로직 처리
- **레포지토리 계층**: `MasterProfileRepository`, `StoreRepository`, `TimeOffRepository` 등 - 데이터 접근
- **DTO 계층**: `MasterMyPageResponseDto`, `StoreDto`, `TimeOffResponseDto` 등 - 데이터 전송 객체
- **도메인 계층**: `MasterProfile`, `Store`, `TimeOff` 등 - 핵심 비즈니스 엔티티

## 3. 데이터 흐름

사장 마이페이지의 주요 데이터 흐름은 다음과 같습니다:

1. 클라이언트(React Native)에서 사장 ID를 포함한 요청을 `/api/master/mypage` 엔드포인트로 전송
2. `MasterController`가 요청을 받아 `MasterProfileService`와 `TimeOffService`를 호출
3. 서비스 계층에서 필요한 데이터를 레포지토리를 통해 조회:
    - 사장 프로필 정보
    - 소유한 매장 목록
    - 통합 통계 데이터
    - 대기 중인 휴가 신청 내역
4. 조회된 데이터를 DTO 객체로 변환하여 클라이언트에 응답

## 4. 주요 기능

### 4.1 사장 마이페이지 데이터 조회

```java
@GetMapping("/mypage")
public ResponseEntity<MasterMyPageResponseDto> getMasterMyPage(@RequestParam Long masterId) {
    // 사장 프로필 조회
    MasterProfile masterProfile = masterProfileService.getMasterProfile(masterId);

    // 사장이 소유한 매장 목록 조회
    List<Store> stores = masterProfileService.getStoresByMaster(masterId);

    // 통합 통계 조회
    Map<String, Object> statsMap = masterProfileService.getCombinedStats(masterId);
    CombinedStatsDto combinedStats = new CombinedStatsDto(
            (int) statsMap.get("totalStores"),
            (int) statsMap.get("totalEmployees"),
            (long) statsMap.get("totalLaborCost"),
            (int) statsMap.get("pendingTimeOffRequests")
    );

    // 휴가 신청 내역 조회
    List<TimeOff> timeOffRequests = timeOffService.getPendingTimeOffsByMaster(masterId);

    // DTO 변환 및 반환
    MasterMyPageResponseDto responseDto = MasterMyPageResponseDto.fromEntities(
            masterProfile, stores, combinedStats, timeOffRequests);

    return ResponseEntity.ok(responseDto);
}
```

이 엔드포인트는 사장 마이페이지에 필요한 모든 데이터를 한 번에 조회하여 반환합니다.

### 4.2 매장 통계 조회

```java
public Map<String, Object> getStoreStats(Long storeId, String month) {
    Store store = storeRepository.findById(storeId)
            .orElseThrow(() -> new NoSuchElementException("매장을 찾을 수 없습니다."));

    // 매장의 직원 수 조회
    List<EmployeeStoreRelation> relations = employeeStoreRelationRepository.findByStore(store);
    int employeeCount = relations.size();

    // 매장의 인건비 조회 (해당 월의 급여 합계)
    LocalDate startDate = LocalDate.parse(month + "-01");
    LocalDate endDate = startDate.plusMonths(1).minusDays(1);

    List<Payroll> payrolls = payrollRepository.findByStoreIdAndPeriod(storeId, startDate, endDate);
    long laborCost = payrolls.stream()
            .mapToLong(payroll -> payroll.getGrossWage() != null ? payroll.getGrossWage() : 0)
            .sum();

    // 평균 시급 계산 (인건비 / (직원 수 * 평균 근무 시간))
    int averageHourlyWage = employeeCount > 0 ?
            Math.round(laborCost / (employeeCount * 160)) : 0;

    // 대기 중인 휴가 신청 수 조회
    int pendingTimeOffRequests = timeOffRepository.countTimeOffsByStoreIdAndStatus(
            storeId, TimeOffStatus.PENDING);

    Map<String, Object> stats = new HashMap<>();
    stats.put("totalEmployees", employeeCount);
    stats.put("totalLaborCost", laborCost);
    stats.put("averageHourlyWage", averageHourlyWage);
    stats.put("pendingTimeOffRequests", pendingTimeOffRequests);
    stats.put("month", month);

    return stats;
}
```

이 메서드는 특정 매장의 통계 데이터를 계산하여 반환합니다. 직원 수, 인건비, 평균 시급, 대기 중인 휴가 신청 수 등의 정보를 포함합니다.

### 4.3 통합 통계 조회

```java
public Map<String, Object> getCombinedStats(Long masterId) {
    List<Store> stores = getStoresByMaster(masterId);

    int totalStores = stores.size();

    // 모든 매장의 직원 수 합계 계산
    int totalEmployees = 0;
    for (Store store : stores) {
        List<EmployeeStoreRelation> relations = employeeStoreRelationRepository.findByStore(store);
        totalEmployees += relations.size();
    }

    // 현재 월의 모든 매장의 인건비 합계 계산
    String currentMonth = LocalDate.now().toString().substring(0, 7); // YYYY-MM 형식
    LocalDate startDate = LocalDate.parse(currentMonth + "-01");
    LocalDate endDate = startDate.plusMonths(1).minusDays(1);

    long totalLaborCost = 0;
    for (Store store : stores) {
        List<Payroll> payrolls = payrollRepository.findByStoreIdAndPeriod(store.getId(), startDate, endDate);
        totalLaborCost += payrolls.stream()
                .mapToLong(payroll -> payroll.getGrossWage() != null ? payroll.getGrossWage() : 0)
                .sum();
    }

    // 대기 중인 휴가 신청 수 조회
    int pendingTimeOffRequests = timeOffService.countPendingTimeOffsByMaster(masterId);

    Map<String, Object> stats = new HashMap<>();
    stats.put("totalStores", totalStores);
    stats.put("totalEmployees", totalEmployees);
    stats.put("totalLaborCost", totalLaborCost);
    stats.put("pendingTimeOffRequests", pendingTimeOffRequests);

    return stats;
}
```

이 메서드는 사장이 소유한 모든 매장의 통합 통계를 계산합니다. 총 매장 수, 총 직원 수, 총 인건비, 대기 중인 휴가 신청 수 등의 정보를 포함합니다.

### 4.4 휴가 신청 관리

```java
@PutMapping("/timeoff/approve")
public ResponseEntity<TimeOff> approveTimeOff(@RequestParam Long timeOffId) {
    TimeOff timeOff = timeOffService.approveTimeOffRequest(timeOffId);
    return ResponseEntity.ok(timeOff);
}

@PutMapping("/timeoff/reject")
public ResponseEntity<TimeOff> rejectTimeOff(@RequestParam Long timeOffId) {
    TimeOff timeOff = timeOffService.rejectTimeOffRequest(timeOffId);
    return ResponseEntity.ok(timeOff);
}
```

이 엔드포인트들은 사장이 직원의 휴가 신청을 승인하거나 거부할 수 있는 기능을 제공합니다.

## 5. 데이터 모델 및 관계

### 5.1 주요 엔티티

- **MasterProfile**: 사장 프로필 정보
- **Store**: 매장 정보
- **TimeOff**: 휴가 신청 정보
- **EmployeeProfile**: 직원 프로필 정보
- **Payroll**: 급여 정보

### 5.2 주요 관계

- **MasterProfile - Store**: 일대다 관계 (MasterStoreRelation 중간 테이블)
- **Store - EmployeeProfile**: 일대다 관계 (EmployeeStoreRelation 중간 테이블)
- **EmployeeProfile - TimeOff**: 일대다 관계
- **Store - TimeOff**: 일대다 관계
- **EmployeeProfile - Payroll**: 일대다 관계
- **Store - Payroll**: 일대다 관계

## 6. API 엔드포인트

### 6.1 사장 마이페이지 관련 엔드포인트

| 엔드포인트                         | HTTP 메서드 | 설명                            |
|-------------------------------|----------|-------------------------------|
| `/api/master/mypage`          | GET      | 사장 마이페이지 데이터 조회               |
| `/api/master/profile`         | GET      | 사장 프로필 조회                     |
| `/api/master/profile`         | PUT      | 사장 프로필 업데이트                   |
| `/api/master/stores`          | GET      | 사장이 소유한 매장 목록 조회              |
| `/api/master/store/stats`     | GET      | 특정 매장의 통계 조회                  |
| `/api/master/stats`           | GET      | 사장이 소유한 모든 매장의 통합 통계 조회       |
| `/api/master/timeoff/pending` | GET      | 사장이 소유한 모든 매장의 대기 중인 휴가 신청 조회 |
| `/api/master/timeoff/approve` | PUT      | 휴가 신청 승인                      |
| `/api/master/timeoff/reject`  | PUT      | 휴가 신청 거부                      |

## 7. 개선 사항

### 7.1 성능 최적화

현재 구현에서는 여러 번의 데이터베이스 조회가 발생합니다. 특히 통합 통계 조회 시 매장별로 반복적인 쿼리가 실행됩니다. 이를 개선하기 위해 다음과 같은 방법을 고려할 수 있습니다:

1. 쿼리 최적화: JOIN을 활용한 단일 쿼리로 데이터 조회
2. 캐싱 도입: 자주 조회되는 통계 데이터 캐싱
3. 페이징 처리: 대량의 데이터 조회 시 페이징 적용

### 7.2 보안 강화

현재 구현에서는 단순히 masterId 파라미터를 통해 사장 정보에 접근합니다. 보안을 강화하기 위해 다음과 같은 방법을 고려할 수 있습니다:

1. 인증 및 인가 강화: JWT 토큰 기반 인증 및 권한 검증
2. 입력 값 검증: 모든 입력 파라미터에 대한 유효성 검증
3. API 요청 제한: 과도한 요청에 대한 제한 설정

## 8. 결론

사장 마이페이지 기능은 매장 운영자가 자신의 비즈니스를 효율적으로 관리할 수 있도록 다양한 정보와 기능을 제공합니다. 이 구현은 계층화된 아키텍처를 통해 유지보수성과 확장성을 확보하였으며, 다양한 통계 데이터를
제공하여 사용자 경험을 향상시켰습니다.

향후 성능 최적화와 보안 강화를 통해 더욱 안정적이고 효율적인 서비스를 제공할 수 있을 것으로 기대됩니다.
