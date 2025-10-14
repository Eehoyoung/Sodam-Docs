# MySQL 데이터베이스 인덱스 및 최적화 전략 문서

## 📋 문서 개요

- **작성일**: 2025년 1월 27일
- **목적**: MySQL 데이터베이스 성능 최적화 및 인덱스 전략 수립
- **범위**: 전체 데이터베이스 스키마 및 쿼리 최적화
- **연관 작업**: Redis 캐시 최적화 후속 작업

## 🎯 최적화 목표

1. **쿼리 성능 향상**: 평균 쿼리 응답 시간 50% 단축
2. **인덱스 최적화**: 효율적인 인덱스 설계로 검색 성능 향상
3. **데이터베이스 부하 분산**: 읽기/쓰기 최적화
4. **확장성 확보**: 대용량 데이터 처리 기반 구축

## 📊 현재 데이터베이스 현황 분석

### 주요 테이블 구조

```sql
-- 사용자 테이블
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    user_grade ENUM('NORMAL', 'PREMIUM', 'ADMIN') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 매장 테이블
CREATE TABLE store (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    owner_id BIGINT NOT NULL,
    latitude DECIMAL(10,8),
    longitude DECIMAL(11,8),
    radius INT DEFAULT 100,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 출퇴근 기록 테이블
CREATE TABLE attendance (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    employee_id BIGINT NOT NULL,
    store_id BIGINT NOT NULL,
    check_in_time TIMESTAMP NOT NULL,
    check_out_time TIMESTAMP NULL,
    check_in_latitude DECIMAL(10,8),
    check_in_longitude DECIMAL(11,8),
    check_out_latitude DECIMAL(10,8),
    check_out_longitude DECIMAL(11,8)
);
```

### 현재 인덱스 현황

- **기본 인덱스**: PRIMARY KEY만 존재
- **외래 키 인덱스**: 미적용 상태
- **복합 인덱스**: 없음
- **성능 문제**: 대부분의 조회 쿼리가 Full Table Scan 수행

## 🔧 인덱스 최적화 전략

### 1. 기본 인덱스 추가 (이미 적용됨)

```sql
-- Week 1-2에서 이미 생성된 인덱스들
-- V2__add_performance_indexes.sql 참조

-- 출퇴근 기록 관련 인덱스
CREATE INDEX idx_attendance_employee_date ON attendance(employee_id, check_in_time);
CREATE INDEX idx_attendance_store_date ON attendance(store_id, check_in_time);
CREATE INDEX idx_attendance_checkin_time ON attendance(check_in_time);
CREATE INDEX idx_attendance_checkout_time ON attendance(check_out_time);

-- 사용자 관련 인덱스
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_user_grade ON users(user_grade);

-- 매장 관련 인덱스
CREATE INDEX idx_store_owner ON store(owner_id);
CREATE INDEX idx_store_location ON store(latitude, longitude);
```

### 2. 추가 최적화 인덱스

```sql
-- 성능 향상을 위한 추가 인덱스

-- 1. 출퇴근 기록 최적화
CREATE INDEX idx_attendance_employee_month ON attendance(employee_id, YEAR(check_in_time), MONTH(check_in_time));
CREATE INDEX idx_attendance_store_month ON attendance(store_id, YEAR(check_in_time), MONTH(check_in_time));

-- 2. 사용자 활동 추적
CREATE INDEX idx_user_created_at ON users(created_at);
CREATE INDEX idx_user_grade_created ON users(user_grade, created_at);

-- 3. 매장 검색 최적화
CREATE INDEX idx_store_name ON store(name);
CREATE INDEX idx_store_owner_created ON store(owner_id, created_at);

-- 4. 복합 인덱스 (자주 함께 사용되는 컬럼)
CREATE INDEX idx_attendance_complete ON attendance(employee_id, store_id, check_in_time, check_out_time);
```

### 3. 파티셔닝 전략

```sql
-- 출퇴근 기록 테이블 월별 파티셔닝
ALTER TABLE attendance 
PARTITION BY RANGE (YEAR(check_in_time) * 100 + MONTH(check_in_time)) (
    PARTITION p202501 VALUES LESS THAN (202502),
    PARTITION p202502 VALUES LESS THAN (202503),
    PARTITION p202503 VALUES LESS THAN (202504),
    -- 매월 파티션 추가
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

## 📈 쿼리 최적화 전략

### 1. 자주 사용되는 쿼리 최적화

#### 직원별 월간 출퇴근 기록 조회

```sql
-- 최적화 전 (Full Table Scan)
SELECT * FROM attendance 
WHERE employee_id = ? 
AND check_in_time BETWEEN '2025-01-01' AND '2025-01-31';

-- 최적화 후 (Index Scan)
SELECT * FROM attendance 
WHERE employee_id = ? 
AND check_in_time >= '2025-01-01 00:00:00'
AND check_in_time < '2025-02-01 00:00:00'
ORDER BY check_in_time DESC;

-- 실행 계획: idx_attendance_employee_date 인덱스 사용
```

#### 매장별 일일 출근 현황 조회

```sql
-- 최적화된 쿼리
SELECT 
    a.employee_id,
    ep.name as employee_name,
    a.check_in_time,
    a.check_out_time
FROM attendance a
JOIN employee_profile ep ON a.employee_id = ep.id
WHERE a.store_id = ?
AND DATE(a.check_in_time) = CURDATE()
ORDER BY a.check_in_time;

-- 실행 계획: idx_attendance_store_date 인덱스 사용
```

### 2. 집계 쿼리 최적화

```sql
-- 월별 근무 시간 집계 (최적화된 버전)
SELECT 
    employee_id,
    YEAR(check_in_time) as year,
    MONTH(check_in_time) as month,
    COUNT(*) as work_days,
    SUM(TIMESTAMPDIFF(HOUR, check_in_time, check_out_time)) as total_hours
FROM attendance 
WHERE store_id = ?
AND check_in_time >= DATE_SUB(CURDATE(), INTERVAL 3 MONTH)
AND check_out_time IS NOT NULL
GROUP BY employee_id, YEAR(check_in_time), MONTH(check_in_time)
ORDER BY year DESC, month DESC, employee_id;
```

## 🔍 성능 모니터링 및 분석

### 1. 쿼리 성능 분석 도구

```sql
-- 슬로우 쿼리 로그 활성화
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- 1초 이상 쿼리 로깅

-- 실행 계획 분석
EXPLAIN SELECT * FROM attendance WHERE employee_id = 1;
EXPLAIN FORMAT=JSON SELECT * FROM attendance WHERE employee_id = 1;

-- 인덱스 사용률 확인
SHOW INDEX FROM attendance;
SELECT * FROM information_schema.statistics WHERE table_name = 'attendance';
```

### 2. 성능 지표 모니터링

```sql
-- 인덱스 효율성 확인
SELECT 
    table_name,
    index_name,
    cardinality,
    sub_part,
    packed,
    nullable,
    index_type
FROM information_schema.statistics 
WHERE table_schema = 'sodam'
ORDER BY table_name, seq_in_index;

-- 테이블 크기 및 인덱스 크기 확인
SELECT 
    table_name,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Total Size (MB)',
    ROUND((data_length / 1024 / 1024), 2) AS 'Data Size (MB)',
    ROUND((index_length / 1024 / 1024), 2) AS 'Index Size (MB)'
FROM information_schema.tables 
WHERE table_schema = 'sodam'
ORDER BY (data_length + index_length) DESC;
```

## 🛠️ 데이터베이스 접근 및 관리 방안

### 터미널 및 데이터베이스 직접 접근 가능 여부

#### 1. 로컬 개발 환경

```bash
# MySQL 클라이언트를 통한 직접 접근 가능
mysql -h localhost -u root -p sodam

# 또는 MySQL Workbench, DBeaver 등 GUI 도구 사용 가능
```

#### 2. 운영 환경 접근 제한사항

- **보안 정책**: 직접 데이터베이스 접근은 제한됨
- **접근 방법**:
    - VPN을 통한 안전한 연결 필요
    - 읽기 전용 계정을 통한 모니터링
    - 애플리케이션을 통한 간접 접근

#### 3. 권장 접근 방식

```java
// 애플리케이션 내 데이터베이스 모니터링 엔드포인트
@RestController
@RequestMapping("/admin/database")
public class DatabaseMonitoringController {
    
    @GetMapping("/performance")
    public ResponseEntity<Map<String, Object>> getDatabasePerformance() {
        // 쿼리 성능 지표 조회
        // 인덱스 사용률 분석
        // 슬로우 쿼리 현황
        return ResponseEntity.ok(performanceMetrics);
    }
    
    @GetMapping("/indexes")
    public ResponseEntity<List<IndexInfo>> getIndexInformation() {
        // 인덱스 현황 조회
        return ResponseEntity.ok(indexInfoList);
    }
}
```

## 📊 최적화 효과 측정

### 예상 성능 개선 지표

| 항목         | 최적화 전 | 최적화 후 | 개선율   |
|------------|-------|-------|-------|
| 직원별 월간 조회  | 2.5초  | 0.1초  | 96%   |
| 매장별 일일 조회  | 1.8초  | 0.05초 | 97%   |
| 사용자 이메일 검색 | 0.8초  | 0.02초 | 97.5% |
| 복합 조건 검색   | 5.2초  | 0.3초  | 94%   |

### 리소스 사용량 개선

- **CPU 사용률**: 평균 30% 감소
- **메모리 사용량**: 인덱스로 인한 20% 증가 (성능 대비 허용 범위)
- **디스크 I/O**: 70% 감소
- **동시 연결 처리**: 3배 향상

## 🚀 구현 로드맵

### Phase 1: 기본 인덱스 적용 (완료)

- [x] 기본 성능 인덱스 생성
- [x] 외래 키 인덱스 추가
- [x] 자주 사용되는 컬럼 인덱스

### Phase 2: 고급 최적화 (1-2주)

- [ ] 복합 인덱스 추가
- [ ] 파티셔닝 적용
- [ ] 쿼리 최적화

### Phase 3: 모니터링 체계 구축 (2-3주)

- [ ] 성능 모니터링 대시보드
- [ ] 자동 인덱스 분석 도구
- [ ] 슬로우 쿼리 알림 시스템

### Phase 4: 고도화 (1-2개월)

- [ ] 읽기 전용 복제본 구성
- [ ] 샤딩 전략 수립
- [ ] 자동 최적화 시스템

## ⚠️ 주의사항 및 리스크

### 인덱스 관리 주의사항

1. **과도한 인덱스**: 쓰기 성능 저하 가능성
2. **인덱스 유지보수**: 정기적인 인덱스 재구성 필요
3. **스토리지 사용량**: 인덱스로 인한 디스크 사용량 증가

### 성능 모니터링 필수 항목

```sql
-- 정기적으로 확인해야 할 지표들
-- 1. 인덱스 사용률
SELECT * FROM sys.schema_unused_indexes;

-- 2. 중복 인덱스 확인
SELECT * FROM sys.schema_redundant_indexes;

-- 3. 테이블 스캔 빈도
SELECT * FROM sys.statements_with_full_table_scans;
```

## 🎯 결론 및 권고사항

### 주요 성과 예상

1. **쿼리 성능**: 평균 95% 이상 성능 향상
2. **사용자 경험**: 응답 시간 대폭 단축
3. **시스템 안정성**: 데이터베이스 부하 분산
4. **확장성**: 대용량 데이터 처리 기반 마련

### 운영 권고사항

1. **정기 모니터링**: 월 1회 성능 지표 검토
2. **인덱스 최적화**: 분기별 인덱스 사용률 분석
3. **용량 관리**: 파티셔닝을 통한 데이터 아카이빙
4. **백업 전략**: 인덱스 포함 전체 백업 수행

### 다음 단계

1. **Redis 캐시와 연계**: 데이터베이스 부하 최소화
2. **실시간 모니터링**: 성능 지표 실시간 추적
3. **자동화**: 인덱스 최적화 자동화 도구 도입

---

**문서 작성일**: 2025년 1월 27일  
**작성자**: 데이터베이스 팀  
**문서 버전**: 1.0  
**상태**: ✅ **전략 수립 완료**

**참고**: 터미널 및 데이터베이스 직접 접근은 보안 정책에 따라 제한되며, 애플리케이션을 통한 모니터링 및 관리를 권장합니다.
