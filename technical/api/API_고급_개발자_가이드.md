# 소담(SODAM) API 고급 개발자 가이드

## 📋 개요

- **작성일**: 2025-10-14
- **작성자**: 소담 백엔드팀
- **문서 유형**: 고급 API 개발 가이드
- **대상**: 고급 백엔드 개발자, 아키텍트
- **관련 문서**: [API 연동 가이드](./API_연동_가이드.md), [API README](../../../API_README.md)

## 🎯 목적

본 문서는 소담 API의 내부 아키텍처, 설계 패턴, 성능 최적화 전략 등 고급 개발자를 위한 심화 내용을 다룹니다.
시스템 확장, 성능 튜닝, 보안 강화를 담당하는 개발자를 위한 기술 문서입니다.

---

## 📚 목차

1. [시스템 아키텍처](#1-시스템-아키텍처)
2. [인증 및 보안 전략](#2-인증-및-보안-전략)
3. [성능 최적화](#3-성능-최적화)
4. [캐싱 전략](#4-캐싱-전략)
5. [데이터베이스 최적화](#5-데이터베이스-최적화)
6. [에러 처리 및 로깅](#6-에러-처리-및-로깅)
7. [API 버저닝 및 확장성](#7-api-버저닝-및-확장성)
8. [모니터링 및 운영](#8-모니터링-및-운영)

---

## 1. 시스템 아키텍처

### 1.1 레이어드 아키텍처

소담 백엔드는 계층형 아키텍처를 기반으로 설계되었습니다.

```
┌─────────────────────────────────────┐
│   Presentation Layer (Controller)    │  ← REST API 엔드포인트
├─────────────────────────────────────┤
│   Business Logic Layer (Service)     │  ← 비즈니스 로직
├─────────────────────────────────────┤
│   Data Access Layer (Repository)     │  ← 데이터 접근
├─────────────────────────────────────┤
│   Domain Layer (Entity, DTO)         │  ← 도메인 모델
└─────────────────────────────────────┘
```

**설계 원칙**:

- **관심사 분리(SoC)**: 각 레이어는 명확한 책임을 가짐
- **의존성 역전(DIP)**: 상위 레이어가 하위 레이어의 추상화에 의존
- **단일 책임(SRP)**: 각 클래스는 하나의 변경 이유만 가짐

### 1.2 핵심 컴포넌트

#### 1.2.1 Controller Layer

```java
@RestController
@RequestMapping("/api/attendance")
@RequiredArgsConstructor
@Slf4j
public class AttendanceController {
    private final AttendanceService attendanceService;
    
    @PostMapping("/check-in")
    @PreAuthorize("hasAnyRole('EMPLOYEE', 'MASTER')")
    public ResponseEntity<ApiResponse<AttendanceDto>> checkIn(
            @RequestBody @Valid CheckInRequest request) {
        // 입력 검증, 응답 포맷팅만 담당
        return ResponseEntity.ok(attendanceService.processCheckIn(request));
    }
}
```

**책임**:

- HTTP 요청/응답 처리
- 입력 검증 (@Valid)
- 인증/인가 확인 (@PreAuthorize)
- DTO 변환

#### 1.2.2 Service Layer

```java
@Service
@Transactional
@RequiredArgsConstructor
public class AttendanceService {
    private final AttendanceRepository attendanceRepository;
    private final StoreRepository storeRepository;
    private final ApplicationEventPublisher eventPublisher;
    
    public AttendanceDto processCheckIn(CheckInRequest request) {
        // 비즈니스 로직 실행
        Store store = storeRepository.findById(request.getStoreId())
            .orElseThrow(() -> new EntityNotFoundException("매장을 찾을 수 없습니다"));
            
        Attendance attendance = Attendance.builder()
            .userId(request.getUserId())
            .storeId(request.getStoreId())
            .checkInTime(LocalDateTime.now())
            .build();
            
        attendance = attendanceRepository.save(attendance);
        
        // 이벤트 발행 (비동기 처리)
        eventPublisher.publishEvent(new CheckInEvent(attendance));
        
        return AttendanceDto.from(attendance);
    }
}
```

**책임**:

- 비즈니스 로직 실행
- 트랜잭션 관리
- 도메인 이벤트 발행
- 도메인 객체 조작

### 1.3 설계 패턴

#### 1.3.1 Factory Pattern (급여 계산)

```java
@Component
public class SalaryCalculatorFactory {
    private final Map<WageType, SalaryCalculator> calculators;
    
    public SalaryCalculatorFactory(List<SalaryCalculator> calculatorList) {
        this.calculators = calculatorList.stream()
            .collect(Collectors.toMap(
                SalaryCalculator::getSupportedType,
                Function.identity()
            ));
    }
    
    public SalaryCalculator getCalculator(WageType wageType) {
        return Optional.ofNullable(calculators.get(wageType))
            .orElseThrow(() -> new UnsupportedOperationException(
                "지원하지 않는 급여 타입입니다: " + wageType));
    }
}
```

#### 1.3.2 Strategy Pattern (급여 계산 전략)

```java
public interface SalaryCalculator {
    WageType getSupportedType();
    BigDecimal calculate(WorkingHours hours, HourlyRate rate);
}

@Component
public class HourlySalaryCalculator implements SalaryCalculator {
    @Override
    public WageType getSupportedType() {
        return WageType.HOURLY;
    }
    
    @Override
    public BigDecimal calculate(WorkingHours hours, HourlyRate rate) {
        return hours.getTotalHours().multiply(rate.getBaseRate());
    }
}

@Component
public class MonthlySalaryCalculator implements SalaryCalculator {
    @Override
    public WageType getSupportedType() {
        return WageType.MONTHLY;
    }
    
    @Override
    public BigDecimal calculate(WorkingHours hours, HourlyRate rate) {
        // 월급제 계산 로직
        return rate.getMonthlyAmount();
    }
}
```

---

## 2. 인증 및 보안 전략

### 2.1 JWT 기반 인증 아키텍처

#### 2.1.1 토큰 구조 및 관리

```java
@Component
@RequiredArgsConstructor
public class JwtTokenProvider {
    private final RedisTemplate<String, String> redisTemplate;
    
    @Value("${jwt.secret}")
    private String secretKey;
    
    @Value("${jwt.expiration}")
    private long accessTokenExpiration;
    
    @Value("${jwt.refresh-expiration}")
    private long refreshTokenExpiration;
    
    public TokenPair generateTokenPair(Authentication authentication) {
        String accessToken = createAccessToken(authentication);
        String refreshToken = createRefreshToken(authentication);
        
        // Refresh Token을 Redis에 저장
        String key = "refresh_token:" + authentication.getName();
        redisTemplate.opsForValue().set(
            key, 
            refreshToken, 
            refreshTokenExpiration, 
            TimeUnit.MILLISECONDS
        );
        
        return new TokenPair(accessToken, refreshToken);
    }
    
    private String createAccessToken(Authentication authentication) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + accessTokenExpiration);
        
        return Jwts.builder()
            .setSubject(authentication.getName())
            .claim("authorities", getAuthorities(authentication))
            .claim("userId", getUserId(authentication))
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(SignatureAlgorithm.HS512, secretKey)
            .compact();
    }
}
```

#### 2.1.2 보안 필터 체인

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .exceptionHandling(exception -> 
                exception.authenticationEntryPoint(jwtAuthenticationEntryPoint))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**", "/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/master/**").hasRole("MASTER")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter, 
                UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

### 2.2 권한 관리 (Role-Based Access Control)

#### 2.2.1 역할 계층 구조

```
ADMIN (최고 관리자)
  └─ MASTER (사업주)
       └─ EMPLOYEE (근로자)
```

#### 2.2.2 메서드 레벨 보안

```java
@Service
public class PayrollService {
    
    @PreAuthorize("hasRole('MASTER')")
    public PayrollDto calculatePayroll(Long employeeId, YearMonth period) {
        // 사업주만 급여 계산 가능
    }
    
    @PreAuthorize("hasRole('EMPLOYEE') and #employeeId == authentication.principal.userId")
    public PayrollDto getMyPayroll(Long employeeId, YearMonth period) {
        // 본인의 급여만 조회 가능
    }
    
    @PreAuthorize("hasRole('MASTER') or " +
                  "(hasRole('EMPLOYEE') and #employeeId == authentication.principal.userId)")
    public PayrollDto getPayroll(Long employeeId, YearMonth period) {
        // 사업주는 모든 급여 조회, 근로자는 본인 것만 조회
    }
}
```

### 2.3 보안 Best Practices

#### 2.3.1 SQL Injection 방지

```java
// ❌ 위험한 방식
@Query(value = "SELECT * FROM users WHERE email = '" + email + "'", nativeQuery = true)
User findByEmailUnsafe(String email);

// ✅ 안전한 방식 (Prepared Statement)
@Query("SELECT u FROM User u WHERE u.email = :email")
Optional<User> findByEmail(@Param("email") String email);
```

#### 2.3.2 XSS 방지

```java
@Component
public class XssFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        XssRequestWrapper wrappedRequest = new XssRequestWrapper(
            (HttpServletRequest) request);
        chain.doFilter(wrappedRequest, response);
    }
}

public class XssRequestWrapper extends HttpServletRequestWrapper {
    @Override
    public String getParameter(String name) {
        String value = super.getParameter(name);
        return sanitize(value);
    }
    
    private String sanitize(String value) {
        if (value == null) return null;
        return value.replaceAll("<", "&lt;")
                   .replaceAll(">", "&gt;")
                   .replaceAll("\"", "&quot;")
                   .replaceAll("'", "&#x27;");
    }
}
```

#### 2.3.3 Rate Limiting

```java
@Component
public class RateLimitingFilter implements Filter {
    private final RedisTemplate<String, String> redisTemplate;
    private static final int MAX_REQUESTS = 100;
    private static final int TIME_WINDOW_SECONDS = 60;
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                        FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String clientId = getClientIdentifier(httpRequest);
        String key = "rate_limit:" + clientId;
        
        String currentCount = redisTemplate.opsForValue().get(key);
        
        if (currentCount == null) {
            redisTemplate.opsForValue().set(key, "1", 
                Duration.ofSeconds(TIME_WINDOW_SECONDS));
        } else if (Integer.parseInt(currentCount) >= MAX_REQUESTS) {
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            httpResponse.getWriter().write("{\"error\": \"요청 한도 초과\"}");
            return;
        } else {
            redisTemplate.opsForValue().increment(key);
        }
        
        chain.doFilter(request, response);
    }
}
```

---

## 3. 성능 최적화

### 3.1 N+1 문제 해결

#### 3.1.1 문제 상황

```java
// ❌ N+1 쿼리 발생
@GetMapping("/users")
public List<UserDto> getUsers() {
    List<User> users = userRepository.findAll(); // 1번의 쿼리
    return users.stream()
        .map(user -> {
            // 각 user마다 추가 쿼리 발생 (N번)
            List<Attendance> attendances = attendanceRepository
                .findByUserId(user.getId());
            return UserDto.of(user, attendances);
        })
        .collect(Collectors.toList());
}
```

#### 3.1.2 해결 방법: Fetch Join

```java
// ✅ Fetch Join 사용
@Query("SELECT DISTINCT u FROM User u " +
       "LEFT JOIN FETCH u.attendances " +
       "WHERE u.storeId = :storeId")
List<User> findAllWithAttendances(@Param("storeId") Long storeId);

// ✅ EntityGraph 사용
@EntityGraph(attributePaths = {"attendances", "payrolls"})
@Query("SELECT u FROM User u WHERE u.storeId = :storeId")
List<User> findAllWithDetails(@Param("storeId") Long storeId);
```

### 3.2 쿼리 최적화

#### 3.2.1 Projection 사용

```java
// ❌ 전체 엔티티 조회
@Query("SELECT u FROM User u WHERE u.storeId = :storeId")
List<User> findByStoreId(@Param("storeId") Long storeId);

// ✅ 필요한 필드만 조회
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

@Query("SELECT u.id as id, u.name as name, u.email as email " +
       "FROM User u WHERE u.storeId = :storeId")
List<UserSummary> findSummaryByStoreId(@Param("storeId") Long storeId);
```

#### 3.2.2 배치 처리

```java
@Service
@RequiredArgsConstructor
public class PayrollBatchService {
    private final PayrollRepository payrollRepository;
    
    @Transactional
    public void calculateMonthlyPayrolls(YearMonth period) {
        int pageSize = 100;
        int pageNumber = 0;
        
        Page<Employee> page;
        do {
            page = employeeRepository.findAll(
                PageRequest.of(pageNumber++, pageSize));
            
            List<Payroll> payrolls = page.getContent().stream()
                .map(emp -> calculatePayroll(emp, period))
                .collect(Collectors.toList());
            
            payrollRepository.saveAll(payrolls);
            
            // 메모리 해제
            entityManager.flush();
            entityManager.clear();
            
        } while (page.hasNext());
    }
}
```

### 3.3 비동기 처리

#### 3.3.1 @Async 활용

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("Async-");
        executor.setRejectedExecutionHandler(
            new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {
    @Async("taskExecutor")
    public CompletableFuture<Void> sendPushNotification(
            Long userId, String message) {
        // 푸시 알림 발송 (비동기)
        log.info("푸시 알림 발송: userId={}, message={}", userId, message);
        // ... 실제 발송 로직
        return CompletableFuture.completedFuture(null);
    }
}
```

#### 3.3.2 이벤트 기반 비동기 처리

```java
// 이벤트 정의
@Getter
@AllArgsConstructor
public class PayrollCalculatedEvent {
    private final Long payrollId;
    private final Long employeeId;
    private final BigDecimal amount;
}

// 이벤트 발행
@Service
public class PayrollService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public Payroll calculatePayroll(Long employeeId, YearMonth period) {
        Payroll payroll = performCalculation(employeeId, period);
        
        // 이벤트 발행
        eventPublisher.publishEvent(
            new PayrollCalculatedEvent(
                payroll.getId(), 
                employeeId, 
                payroll.getTotalAmount()
            )
        );
        
        return payroll;
    }
}

// 이벤트 리스너 (비동기)
@Component
public class PayrollEventListener {
    @Async
    @EventListener
    public void handlePayrollCalculated(PayrollCalculatedEvent event) {
        // 알림 발송, 통계 업데이트 등
        log.info("급여 계산 완료: payrollId={}", event.getPayrollId());
        notificationService.notifyPayrollCalculated(event);
        statisticsService.updatePayrollStats(event);
    }
}
```

---

## 4. 캐싱 전략

### 4.1 Redis 캐시 아키텍처

#### 4.1.1 다층 캐시 전략

```
┌─────────────────┐
│  Application     │
├─────────────────┤
│  L1: Caffeine    │  ← 로컬 캐시 (빠름, 작은 용량)
├─────────────────┤
│  L2: Redis       │  ← 분산 캐시 (느림, 큰 용량)
├─────────────────┤
│  Database        │  ← 영구 저장소
└─────────────────┘
```

#### 4.1.2 Redis 설정

```java
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public RedisCacheManager cacheManager(
            RedisConnectionFactory connectionFactory) {
        
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration
            .defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair
                    .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair
                    .fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        Map<String, RedisCacheConfiguration> cacheConfigurations = Map.of(
            "users", defaultConfig.entryTtl(Duration.ofMinutes(30)),
            "stores", defaultConfig.entryTtl(Duration.ofHours(2)),
            "payrolls", defaultConfig.entryTtl(Duration.ofMinutes(15))
        );
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(defaultConfig)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
}
```

### 4.2 캐시 패턴

#### 4.2.1 Cache-Aside Pattern

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final RedisTemplate<String, User> redisTemplate;
    
    public User findById(Long userId) {
        String key = "user:" + userId;
        
        // 1. 캐시 조회
        User cachedUser = redisTemplate.opsForValue().get(key);
        if (cachedUser != null) {
            return cachedUser;
        }
        
        // 2. DB 조회
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException("사용자 없음"));
        
        // 3. 캐시 저장
        redisTemplate.opsForValue().set(key, user, Duration.ofMinutes(30));
        
        return user;
    }
}
```

#### 4.2.2 Write-Through Pattern

```java
@Service
public class UserService {
    
    @CachePut(value = "users", key = "#user.id")
    public User updateUser(User user) {
        // DB 업데이트와 동시에 캐시 갱신
        return userRepository.save(user);
    }
}
```

#### 4.2.3 Cache Eviction

```java
@Service
public class StoreService {
    
    @CacheEvict(value = "stores", key = "#storeId")
    public void deleteStore(Long storeId) {
        storeRepository.deleteById(storeId);
    }
    
    @Caching(evict = {
        @CacheEvict(value = "stores", key = "#storeId"),
        @CacheEvict(value = "storeStats", key = "#storeId"),
        @CacheEvict(value = "storeEmployees", key = "#storeId")
    })
    public void deleteStoreWithRelations(Long storeId) {
        storeRepository.deleteById(storeId);
    }
}
```

### 4.3 캐시 워밍 (Cache Warming)

```java
@Component
@RequiredArgsConstructor
public class CacheWarmingService {
    private final StoreRepository storeRepository;
    private final CacheManager cacheManager;
    
    @PostConstruct
    @Scheduled(cron = "0 0 * * * *") // 매 시간마다
    public void warmUpCache() {
        log.info("캐시 워밍 시작");
        
        // 자주 조회되는 데이터 미리 로드
        List<Store> activeStores = storeRepository.findAllByActiveTrue();
        Cache storeCache = cacheManager.getCache("stores");
        
        activeStores.forEach(store -> 
            storeCache.put(store.getId(), store));
        
        log.info("캐시 워밍 완료: {} 개 매장", activeStores.size());
    }
}
```

---

## 5. 데이터베이스 최적화

### 5.1 인덱스 전략

#### 5.1.1 효과적인 인덱스 설계

```sql
-- 단일 컬럼 인덱스
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_attendance_date ON attendance(attendance_date);

-- 복합 인덱스 (순서 중요!)
CREATE INDEX idx_attendance_user_date 
ON attendance(user_id, attendance_date);

-- 커버링 인덱스
CREATE INDEX idx_user_store_status 
ON users(store_id, status) 
INCLUDE (name, email);

-- 부분 인덱스
CREATE INDEX idx_active_users 
ON users(email) 
WHERE status = 'ACTIVE';
```

#### 5.1.2 인덱스 모니터링

```java
@Repository
public interface AttendanceRepository extends JpaRepository<Attendance, Long> {
    
    // ✅ 인덱스 활용 (user_id, attendance_date)
    @Query(value = "SELECT * FROM attendance " +
                   "WHERE user_id = :userId " +
                   "AND attendance_date BETWEEN :startDate AND :endDate " +
                   "ORDER BY attendance_date DESC",
           nativeQuery = true)
    List<Attendance> findByUserAndDateRange(
        @Param("userId") Long userId,
        @Param("startDate") LocalDate startDate,
        @Param("endDate") LocalDate endDate
    );
}
```

### 5.2 쿼리 성능 분석

#### 5.2.1 실행 계획 확인

```sql
-- MySQL
EXPLAIN SELECT * FROM attendance 
WHERE user_id = 1 
AND attendance_date BETWEEN '2025-01-01' AND '2025-01-31';

-- 결과 해석
-- type: ref (인덱스 사용)
-- key: idx_attendance_user_date
-- rows: 30 (예상 행 수)
```

#### 5.2.2 슬로우 쿼리 로깅

```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        show_sql: true
        format_sql: true
        use_sql_comments: true
        
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

### 5.3 커넥션 풀 최적화

#### 5.3.1 HikariCP 설정

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      pool-name: SodamHikariCP
      
      # 성능 튜닝
      auto-commit: false
      leak-detection-threshold: 60000
      
      # 커넥션 검증
      connection-test-query: SELECT 1
```

---

## 6. 에러 처리 및 로깅

### 6.1 전역 예외 처리

#### 6.1.1 GlobalExceptionHandler

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEntityNotFound(
            EntityNotFoundException ex, HttpServletRequest request) {
        log.warn("엔티티 없음: {} - URI: {}", 
            ex.getMessage(), request.getRequestURI());
        
        ErrorResponse error = ErrorResponse.builder()
            .code("ENTITY_NOT_FOUND")
            .message(ex.getMessage())
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            ValidationException ex, HttpServletRequest request) {
        log.info("검증 실패: {} - URI: {}", 
            ex.getMessage(), request.getRequestURI());
        
        ErrorResponse error = ErrorResponse.builder()
            .code("VALIDATION_FAILED")
            .message(ex.getMessage())
            .errors(ex.getErrors())
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(
            Exception ex, HttpServletRequest request) {
        log.error("서버 오류: {} - URI: {}", 
            ex.getMessage(), request.getRequestURI(), ex);
        
        ErrorResponse error = ErrorResponse.builder()
            .code("INTERNAL_SERVER_ERROR")
            .message("서버 오류가 발생했습니다")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error);
    }
}
```

### 6.2 구조화된 로깅

#### 6.2.1 MDC (Mapped Diagnostic Context)

```java
@Component
public class RequestLoggingFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                        FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        
        String requestId = UUID.randomUUID().toString();
        String userId = extractUserId(httpRequest);
        
        MDC.put("requestId", requestId);
        MDC.put("userId", userId);
        MDC.put("uri", httpRequest.getRequestURI());
        MDC.put("method", httpRequest.getMethod());
        
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

#### 6.2.2 로그 레벨 전략

```java
@Service
@Slf4j
public class PayrollService {
    
    public Payroll calculatePayroll(Long employeeId, YearMonth period) {
        // DEBUG: 상세한 디버깅 정보
        log.debug("급여 계산 시작: employeeId={}, period={}", 
            employeeId, period);
        
        try {
            // INFO: 주요 비즈니스 이벤트
            log.info("급여 계산 중: employeeId={}", employeeId);
            
            Payroll payroll = performCalculation(employeeId, period);
            
            // INFO: 성공 로그
            log.info("급여 계산 완료: payrollId={}, amount={}", 
                payroll.getId(), payroll.getTotalAmount());
            
            return payroll;
            
        } catch (DataIntegrityException ex) {
            // WARN: 복구 가능한 오류
            log.warn("데이터 정합성 오류: employeeId={}, error={}", 
                employeeId, ex.getMessage());
            throw ex;
            
        } catch (Exception ex) {
            // ERROR: 심각한 오류
            log.error("급여 계산 실패: employeeId={}, period={}", 
                employeeId, period, ex);
            throw ex;
        }
    }
}
```

---

## 7. API 버저닝 및 확장성

### 7.1 API 버저닝 전략

#### 7.1.1 URI 버저닝

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
    // 버전 1 API
}

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    // 버전 2 API (새로운 기능)
}
```

#### 7.1.2 헤더 버저닝

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping(headers = "API-Version=1")
    public ResponseEntity<UserDtoV1> getUserV1(@PathVariable Long id) {
        // 버전 1
    }
    
    @GetMapping(headers = "API-Version=2")
    public ResponseEntity<UserDtoV2> getUserV2(@PathVariable Long id) {
        // 버전 2
    }
}
```

### 7.2 하위 호환성 유지

#### 7.2.1 DTO 버저닝

```java
// V1 DTO
@Data
public class UserDtoV1 {
    private Long id;
    private String name;
    private String email;
}

// V2 DTO (새 필드 추가)
@Data
public class UserDtoV2 {
    private Long id;
    private String name;
    private String email;
    private String phoneNumber; // 새로 추가
    private LocalDateTime createdAt; // 새로 추가
}
```

---

## 8. 모니터링 및 운영

### 8.1 Spring Boot Actuator

#### 8.1.1 Actuator 설정

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    export:
      prometheus:
        enabled: true
```

#### 8.1.2 커스텀 Health Indicator

```java
@Component
public class RedisHealthIndicator implements HealthIndicator {
    private final RedisTemplate<String, String> redisTemplate;
    
    @Override
    public Health health() {
        try {
            redisTemplate.opsForValue().get("health-check");
            return Health.up()
                .withDetail("redis", "Available")
                .build();
        } catch (Exception ex) {
            return Health.down()
                .withDetail("redis", "Unavailable")
                .withException(ex)
                .build();
        }
    }
}
```

### 8.2 메트릭 수집

#### 8.2.1 커스텀 메트릭

```java
@Component
@RequiredArgsConstructor
public class BusinessMetrics {
    private final MeterRegistry meterRegistry;
    
    @PostConstruct
    public void init() {
        // 카운터
        Counter.builder("attendance.checkin.count")
            .description("출근 횟수")
            .register(meterRegistry);
        
        // 게이지
        Gauge.builder("store.active.count", this::getActiveStoreCount)
            .description("활성 매장 수")
            .register(meterRegistry);
        
        // 타이머
        Timer.builder("payroll.calculation.time")
            .description("급여 계산 소요 시간")
            .register(meterRegistry);
    }
    
    private int getActiveStoreCount() {
        return storeRepository.countByActiveTrue();
    }
}
```

---

## 🔗 관련 문서

- [API 연동 가이드](./API_연동_가이드.md) - 초급~중급 개발자용
- [API README](../../../API_README.md) - Quick Reference
- [Redis 캐시 전략 문서](../Redis_캐시_전략_문서.md)
- [MySQL 데이터베이스 최적화 전략 문서](../MySQL_데이터베이스_최적화_전략_문서.md)
- [개발 가이드라인](../../guidelines/aiguidelines.md)

---

## 📅 변경 이력

| 날짜         | 버전  | 변경 내용 | 작성자     |
|------------|-----|-------|---------|
| 2025-10-14 | 1.0 | 초기 작성 | 소담 백엔드팀 |
