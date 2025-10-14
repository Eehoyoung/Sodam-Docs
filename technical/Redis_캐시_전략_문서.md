# Redis 캐시 전략 문서

## 📋 개요

- **작성일**: 2025년 1월 27일
- **목적**: Week 1-2 로드맵 - Redis 캐시 전략 사전 수립
- **적용 범위**: 전체 애플리케이션 캐시 시스템

## 🎯 캐시 전략 목표

1. **성능 향상**: 데이터베이스 부하 감소 및 응답 시간 단축
2. **확장성**: 사용자 증가에 따른 시스템 확장성 확보
3. **안정성**: 캐시 실패 시에도 정상 동작 보장
4. **일관성**: 데이터 일관성 유지

## 🏗️ Redis 아키텍처 설계

### 데이터베이스 분리 전략

```
Redis Database 0: JWT 토큰 저장
Redis Database 1: 애플리케이션 캐시
```

### 연결 설정

- **호스트**: `${spring.data.redis.host:localhost}`
- **포트**: `${spring.data.redis.port:6379}`
- **타임아웃**: 5초
- **연결 풀**: Lettuce 사용

## 📊 캐시 분류 및 TTL 전략

### 1. 사용자 정보 캐시 (`users`)

- **TTL**: 1시간
- **용도**: 사용자 프로필, 권한 정보
- **갱신 조건**: 사용자 정보 변경 시
- **키 패턴**: `user:{userId}`

### 2. 매장 정보 캐시 (`stores`)

- **TTL**: 2시간
- **용도**: 매장 기본 정보, 위치 정보
- **갱신 조건**: 매장 정보 변경 시
- **키 패턴**: `store:{storeId}`

### 3. 출근 기록 캐시 (`attendance`)

- **TTL**: 15분
- **용도**: 실시간 출근 현황, 최근 출근 기록
- **갱신 조건**: 출퇴근 기록 생성/수정 시
- **키 패턴**: `attendance:{employeeId}:{date}`

### 4. 급여 정보 캐시 (`payroll`)

- **TTL**: 4시간
- **용도**: 급여 계산 결과, 급여 명세서
- **갱신 조건**: 급여 계산 완료 시
- **키 패턴**: `payroll:{employeeId}:{month}`

### 5. JWT 토큰 캐시

- **TTL**: 토큰 만료 시간과 동일
- **용도**: 액세스 토큰, 리프레시 토큰 저장
- **갱신 조건**: 토큰 생성/갱신 시
- **키 패턴**: `jwt:{userId}`, `refresh:{tokenId}`

## 🔄 캐시 갱신 전략

### 1. Cache-Aside 패턴 (기본)

```java
@Cacheable(value = "users", key = "#userId")
public User getUserById(Long userId) {
    return userRepository.findById(userId);
}

@CacheEvict(value = "users", key = "#user.id")
public User updateUser(User user) {
    return userRepository.save(user);
}
```

### 2. Write-Through 패턴 (중요 데이터)

```java
@CachePut(value = "stores", key = "#store.id")
public Store updateStore(Store store) {
    Store savedStore = storeRepository.save(store);
    // 캐시에 즉시 반영
    return savedStore;
}
```

### 3. Write-Behind 패턴 (로그 데이터)

```java
// 출근 기록 등 대용량 데이터에 적용
@Async
public void updateAttendanceCache(AttendanceRecord record) {
    // 비동기로 캐시 업데이트
}
```

## 📈 성능 최적화 방안

### 1. 캐시 워밍 (Cache Warming)

```java
@EventListener(ApplicationReadyEvent.class)
public void warmUpCache() {
    // 애플리케이션 시작 시 자주 사용되는 데이터 미리 캐시
    List<Store> popularStores = storeService.getPopularStores();
    popularStores.forEach(store -> 
        cacheManager.getCache("stores").put(store.getId(), store));
}
```

### 2. 캐시 압축

```java
// 대용량 데이터의 경우 압축 적용
@Bean
public RedisTemplate<String, Object> compressedRedisTemplate() {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setValueSerializer(new CompressedJsonRedisSerializer());
    return template;
}
```

### 3. 배치 캐시 갱신

```java
@Scheduled(fixedRate = 300000) // 5분마다
public void batchUpdateCache() {
    // 배치로 캐시 갱신하여 DB 부하 분산
}
```

## 🛡️ 캐시 안정성 보장

### 1. 캐시 실패 처리

```java
@Retryable(value = {RedisConnectionFailureException.class}, maxAttempts = 3)
@Cacheable(value = "users", key = "#userId")
public User getUserById(Long userId) {
    return userRepository.findById(userId);
}

@Recover
public User recoverGetUserById(RedisConnectionFailureException ex, Long userId) {
    log.warn("Redis cache failed, falling back to database: {}", ex.getMessage());
    return userRepository.findById(userId);
}
```

### 2. 캐시 일관성 보장

```java
@Transactional
@CacheEvict(value = {"users", "stores"}, allEntries = true)
public void updateUserStoreRelation(Long userId, Long storeId) {
    // 관련된 모든 캐시 무효화
    userStoreService.updateRelation(userId, storeId);
}
```

### 3. 캐시 모니터링

```java
@Component
public class CacheMetrics {
    
    @EventListener
    public void handleCacheHit(CacheHitEvent event) {
        meterRegistry.counter("cache.hit", "cache", event.getCacheName()).increment();
    }
    
    @EventListener
    public void handleCacheMiss(CacheMissEvent event) {
        meterRegistry.counter("cache.miss", "cache", event.getCacheName()).increment();
    }
}
```

## 📊 캐시 성능 지표

### 목표 지표

- **캐시 히트율**: 85% 이상
- **평균 응답 시간**: 50ms 이하
- **메모리 사용률**: 80% 이하
- **연결 풀 사용률**: 70% 이하

### 모니터링 항목

1. **히트율 모니터링**: 캐시별 히트율 추적
2. **메모리 사용량**: Redis 메모리 사용량 모니터링
3. **응답 시간**: 캐시 조회 응답 시간 측정
4. **에러율**: 캐시 관련 에러 발생률 추적

## 🔧 운영 가이드라인

### 1. 캐시 키 네이밍 규칙

```
{service}:{entity}:{id}:{version}
예: user:profile:123:v1, store:info:456:v2
```

### 2. 캐시 크기 제한

```yaml
spring:
  cache:
    redis:
      cache-null-values: false
      time-to-live: 1800000  # 30분
      key-prefix: "sodam:"
```

### 3. 캐시 정리 정책

- **LRU (Least Recently Used)**: 기본 정책
- **수동 정리**: 배치 작업으로 만료된 키 정리
- **메모리 임계값**: 80% 도달 시 알림

### 4. 장애 대응 절차

1. **Redis 장애 시**: 데이터베이스 직접 조회로 폴백
2. **메모리 부족 시**: 캐시 정리 및 TTL 단축
3. **성능 저하 시**: 캐시 전략 재검토

## 📝 구현 체크리스트

### 완료된 항목 ✅

- [x] Redis 연결 설정 (JWT용, 캐시용 분리)
- [x] 캐시 매니저 설정
- [x] 캐시별 TTL 설정
- [x] 기본 캐시 어노테이션 적용

### 추가 구현 필요 항목 📋

- [ ] 캐시 워밍 로직 구현
- [ ] 캐시 모니터링 메트릭 추가
- [ ] 배치 캐시 갱신 스케줄러 구현
- [ ] 캐시 압축 기능 추가
- [ ] 장애 복구 로직 강화

## 🚀 향후 개선 계획

### 단기 (1-2개월)

1. **캐시 히트율 최적화**: 자주 사용되는 데이터 식별 및 캐시 적용
2. **모니터링 강화**: 실시간 캐시 성능 대시보드 구축
3. **자동 스케일링**: 메모리 사용량에 따른 자동 확장

### 중기 (3-6개월)

1. **분산 캐시**: Redis Cluster 도입 검토
2. **캐시 계층화**: L1(로컬), L2(Redis) 캐시 구조
3. **AI 기반 캐시**: 사용 패턴 분석을 통한 지능형 캐시

### 장기 (6개월 이상)

1. **글로벌 캐시**: 지역별 캐시 서버 구축
2. **실시간 동기화**: 캐시 간 실시간 데이터 동기화
3. **예측 캐시**: 사용자 행동 예측 기반 사전 캐시

---

**문서 버전**: 1.0  
**최종 수정일**: 2025년 1월 27일  
**작성자**: 개발팀
