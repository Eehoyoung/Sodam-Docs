# STORE-001과 AUTH-008 기능 플로우 분석 문서

## 📋 개요

- **작성일**: 2025-07-26
- **작성자**: 소담 개발팀
- **문서 유형**: 기술 분석 문서
- **관련 이슈**: STORE-001과 AUTH-008 기능 간 플로우 분석

## 🎯 목적

본 문서는 소담(SODAM) 애플리케이션의 STORE-001(매장 등록) 기능과 AUTH-008(사업주 전환) 기능 간의 플로우를 상세하게 분석합니다. 두 기능이 동시에 실행되는 경우와 순차적으로 실행되는 경우의
차이점을 명확히 하여 개발자가 올바른 구현 방향을 이해할 수 있도록 합니다.

## 📑 목차

1. [기능 개요](#1-기능-개요)
2. [데이터 모델 관계](#2-데이터-모델-관계)
3. [플로우 시나리오 분석](#3-플로우-시나리오-분석)
4. [구현 상세 분석](#4-구현-상세-분석)
5. [권장 사용 패턴](#5-권장-사용-패턴)

---

## 1. 기능 개요

### 1.1 기능 정보

| 기능ID      | 기능명    | 우선순위 | 상태   | 기능설명              | 관련 클래스                            |
|-----------|--------|------|------|-------------------|-----------------------------------|
| AUTH-008  | 사업주 전환 | High | ✅ 완료 | 일반 사용자의 사업주 전환    | UserService                       |
| STORE-001 | 매장 등록  | High | ✅ 완료 | 매장 정보 등록 및 사업주 지정 | StoreManagementServiceImpl, Store |

### 1.2 기능 간 관계

**핵심 발견**: STORE-001 기능이 AUTH-008 기능을 **내부적으로 포함**하고 있습니다.

```java
// StoreManagementServiceImpl.registerStoreWithMaster() 메서드 내부
@Override
@Transactional
public Store registerStoreWithMaster(Long userId, StoreRegistrationDto storeDto) {
    User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException("사용자를 찾을 수 없습니다."));

    // 사업자 등록번호 검증 로직...

    // 🔥 핵심: 사용자를 MASTER로 변경 (AUTH-008 기능 포함)
    user.changeToMaster();

    // MasterProfile 생성 또는 조회
    MasterProfile masterProfile;
    // ... 매장 등록 로직 계속
}
```

---

## 2. 데이터 모델 관계

### 2.1 엔티티 관계도

```
User (사용자)
├── userGrade: UserGrade (NORMAL → MASTER)
└── 1:1 → MasterProfile (사업주 프로필)
            └── 1:N → MasterStoreRelation (사업주-매장 관계)
                        └── N:1 → Store (매장)
```

### 2.2 주요 엔티티 구조

#### User 엔티티

```java
@Entity
public class User {
    @Id
    private Long id;
    private String email;
    private String name;
    @Enumerated(EnumType.STRING)
    private UserGrade userGrade; // NORMAL → MASTER 변경
    
    public void changeToMaster() {
        this.userGrade = UserGrade.MASTER;
    }
}
```

#### MasterProfile 엔티티

```java
@Entity
public class MasterProfile {
    @Id
    private Long id;  // User의 ID와 동일
    
    @OneToOne
    @MapsId
    @JoinColumn(name = "user_id")
    private User user;
    
    private String businessLicenseNumber;
}
```

#### MasterStoreRelation 엔티티

```java
@Entity
public class MasterStoreRelation {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "master_id")
    private MasterProfile masterProfile;
    
    @ManyToOne
    @JoinColumn(name = "store_id")
    private Store store;
    
    private LocalDateTime registeredAt;
}
```

---

## 3. 플로우 시나리오 분석

### 3.1 시나리오 1: 통합 실행 (STORE-001이 AUTH-008 포함) ⭐ **권장**

#### 3.1.1 실행 흐름

```
일반 사용자 → 매장 등록 요청 → STORE-001 실행
                                    ├── AUTH-008 자동 실행 (user.changeToMaster())
                                    ├── MasterProfile 생성
                                    ├── Store 생성
                                    └── MasterStoreRelation 생성
                                    → 사업주 + 매장 등록 완료
```

#### 3.1.2 코드 흐름

```java
// 1. 클라이언트 요청
POST /api/stores/register
{
    "userId": 1,
    "storeName": "카페 소담",
    "businessNumber": "123-45-67890",
    // ... 기타 매장 정보
}

// 2. StoreManagementServiceImpl.registerStoreWithMaster() 실행
public Store registerStoreWithMaster(Long userId, StoreRegistrationDto storeDto) {
    // 2-1. 사용자 조회
    User user = userRepository.findById(userId);
    
    // 2-2. 사업자 등록번호 검증
    validateBusinessNumber(storeDto.getBusinessNumber());
    
    // 2-3. 🔥 사업주 전환 (AUTH-008)
    user.changeToMaster(); // NORMAL → MASTER
    
    // 2-4. MasterProfile 생성 또는 조회
    MasterProfile masterProfile = createOrGetMasterProfile(user, storeDto);
    
    // 2-5. Store 생성
    Store store = createStore(storeDto);
    
    // 2-6. MasterStoreRelation 생성
    MasterStoreRelation relation = new MasterStoreRelation(masterProfile, store);
    
    return store;
}
```

#### 3.1.3 장점

- **원자성 보장**: 하나의 트랜잭션에서 모든 작업 수행
- **일관성**: 사업주 전환과 매장 등록이 동시에 성공하거나 실패
- **사용자 편의성**: 한 번의 요청으로 사업주가 되어 매장 운영 가능
- **데이터 무결성**: 매장 없는 사업주 또는 사업주 없는 매장 상황 방지

#### 3.1.4 단점

- **기능 결합도 증가**: STORE-001이 AUTH-008에 의존
- **유연성 제한**: 사업주 전환만 별도로 수행하기 어려움

### 3.2 시나리오 2: 순차 실행 (AUTH-008 → STORE-001)

#### 3.2.1 실행 흐름

```
일반 사용자 → 1단계: AUTH-008 실행 → 사업주 전환 완료
                                    ↓
             2단계: STORE-001 실행 → 매장 등록 완료
```

#### 3.2.2 코드 흐름

```java
// 1단계: 사업주 전환
POST /api/user/{userId}/convert-to-owner
{
    "userId": 1
}

// UserService.convertToOwner() 실행
public User convertToOwner(Long userId) {
    User user = userRepository.findById(userId);
    
    // 권한 검증
    if (user.getUserGrade() == UserGrade.MASTER) {
        throw new IllegalArgumentException("이미 사업주 권한을 가진 사용자입니다.");
    }
    
    // 사업주로 전환
    user.changeToMaster();
    return userRepository.save(user);
}

// 2단계: 매장 등록 (이미 사업주인 사용자)
POST /api/stores/register
{
    "userId": 1, // 이미 MASTER 등급
    "storeName": "카페 소담",
    // ... 기타 매장 정보
}
```

#### 3.2.3 장점

- **기능 분리**: 각 기능이 독립적으로 동작
- **유연성**: 사업주 전환 후 원하는 시점에 매장 등록 가능
- **단계별 검증**: 각 단계에서 별도 검증 가능

#### 3.2.4 단점

- **복잡성 증가**: 클라이언트에서 두 번의 API 호출 필요
- **일관성 위험**: 첫 번째 단계 성공 후 두 번째 단계 실패 가능
- **사용자 경험**: 두 단계로 나뉘어 사용자 불편

### 3.3 시나리오 3: 동시 실행 (병렬 처리) ❌ **불가능**

AUTH-008과 STORE-001을 완전히 독립적으로 동시 실행하는 것은 **불가능**합니다.

#### 3.3.1 불가능한 이유

1. **데이터 의존성**: 매장 등록 시 사업주 권한 필요
2. **비즈니스 로직**: 일반 사용자는 매장을 소유할 수 없음
3. **데이터 무결성**: MasterProfile 없이 MasterStoreRelation 생성 불가

---

## 4. 구현 상세 분석

### 4.1 현재 구현 방식

현재 소담 애플리케이션은 **시나리오 1 (통합 실행)** 방식으로 구현되어 있습니다.

#### 4.1.1 registerStoreWithMaster 메서드 분석

```java
@Override
@Transactional
public Store registerStoreWithMaster(Long userId, StoreRegistrationDto storeDto) {
    // 1. 사용자 조회 및 검증
    User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException("사용자를 찾을 수 없습니다."));

    // 2. 사업자 등록번호 검증
    if (!validationService.validateFormat(storeDto.getBusinessNumber())) {
        throw new IllegalArgumentException("사업자 등록번호 형식이 올바르지 않습니다.");
    }
    if (!validationService.validateWithTaxOffice(storeDto.getBusinessNumber())) {
        throw new IllegalArgumentException("유효하지 않은 사업자 등록번호입니다.");
    }
    if (validationService.isDuplicate(storeDto.getBusinessNumber())) {
        throw new IllegalArgumentException("이미 등록된 사업자 등록번호입니다.");
    }

    // 3. 🔥 사업주 전환 (AUTH-008 기능 포함)
    user.changeToMaster();

    // 4. MasterProfile 생성 또는 조회
    MasterProfile masterProfile;
    Optional<MasterProfile> masterProfileOptional = masterProfileRepository.findById(userId);
    
    if (masterProfileOptional.isPresent()) {
        masterProfile = masterProfileOptional.get();
    } else {
        masterProfile = new MasterProfile(user, storeDto.getBusinessLicenseNumber());
        masterProfileRepository.save(masterProfile);
    }

    // 5. 매장 생성
    Store store = getStore(storeDto);
    storeRepository.save(store);

    // 6. 사장-매장 관계 생성
    MasterStoreRelation relation = new MasterStoreRelation(masterProfile, store);
    masterStoreRelationRepository.save(relation);

    return store;
}
```

#### 4.1.2 트랜잭션 처리

```java
@Transactional // 모든 작업이 하나의 트랜잭션에서 수행
public Store registerStoreWithMaster(Long userId, StoreRegistrationDto storeDto) {
    // 모든 데이터베이스 작업이 원자적으로 수행됨
    // 실패 시 모든 변경사항 롤백
}
```

### 4.2 별도 AUTH-008 구현

독립적인 사업주 전환 기능도 구현되어 있습니다:

```java
// UserService.convertToOwner()
@Transactional
@CacheEvict(value = "users", allEntries = true)
public User convertToOwner(Long userId) {
    User user = userRepository.findById(userId)
            .orElseThrow(() -> new IllegalArgumentException("사용자를 찾을 수 없습니다. ID: " + userId));

    // 이미 사업주인 경우 예외 처리
    if (user.getUserGrade() == UserGrade.MASTER) {
        throw new IllegalArgumentException("이미 사업주 권한을 가진 사용자입니다.");
    }

    // 일반 사용자만 사업주로 전환 가능
    if (user.getUserGrade() != UserGrade.NORMAL) {
        throw new IllegalArgumentException("일반 사용자만 사업주로 전환할 수 있습니다. 현재 등급: " + user.getUserGrade());
    }

    // 사업주로 전환
    user.changeToMaster();
    return userRepository.save(user);
}
```

---

## 5. 권장 사용 패턴

### 5.1 일반적인 사용 시나리오 ⭐ **권장**

**매장 등록과 동시에 사업주 전환 (시나리오 1)**

```javascript
// 프론트엔드 코드 예시
const registerStoreAndBecomeOwner = async (userId, storeData) => {
    try {
        // 한 번의 API 호출로 사업주 전환 + 매장 등록
        const response = await fetch('/api/stores/register', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                userId: userId,
                ...storeData
            })
        });
        
        if (response.ok) {
            const store = await response.json();
            console.log('매장 등록 및 사업주 전환 완료:', store);
            // 사용자는 이제 MASTER 등급이 되어 매장을 소유함
        }
    } catch (error) {
        console.error('매장 등록 실패:', error);
        // 모든 변경사항이 롤백됨 (사업주 전환도 취소)
    }
};
```

### 5.2 특수한 사용 시나리오

**사업주 전환 후 나중에 매장 등록 (시나리오 2)**

```javascript
// 1단계: 사업주 전환
const convertToOwner = async (userId) => {
    const response = await fetch(`/api/user/${userId}/convert-to-owner`, {
        method: 'POST'
    });
    return response.json();
};

// 2단계: 매장 등록 (나중에 수행)
const registerStore = async (userId, storeData) => {
    const response = await fetch('/api/stores/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            userId: userId, // 이미 MASTER 등급
            ...storeData
        })
    });
    return response.json();
};

// 사용 예시
const handleBusinessRegistration = async () => {
    try {
        // 1단계: 사업주 전환
        await convertToOwner(userId);
        
        // 사용자가 매장 정보를 입력할 시간을 가짐
        // ...
        
        // 2단계: 매장 등록
        await registerStore(userId, storeData);
    } catch (error) {
        console.error('사업 등록 과정에서 오류 발생:', error);
    }
};
```

### 5.3 권장사항

#### 5.3.1 일반적인 경우

- **시나리오 1 (통합 실행)** 사용 권장
- 사용자 경험이 우수하고 데이터 일관성 보장
- 대부분의 사용자는 매장을 등록하기 위해 사업주가 됨

#### 5.3.2 특수한 경우

- **시나리오 2 (순차 실행)** 사용 고려
- 사업주 전환 후 매장 등록을 나중에 하고 싶은 경우
- 복수 매장 등록 시 첫 번째 매장 등록 후 추가 매장 등록

#### 5.3.3 구현 고려사항

```java
// 매장 등록 시 사용자 상태 확인
public Store registerStoreWithMaster(Long userId, StoreRegistrationDto storeDto) {
    User user = userRepository.findById(userId);
    
    // 이미 사업주인 경우와 일반 사용자인 경우 모두 처리
    if (user.getUserGrade() != UserGrade.MASTER) {
        // 일반 사용자인 경우 사업주로 전환
        user.changeToMaster();
    }
    
    // 나머지 매장 등록 로직...
}
```

---

## 6. 결론

### 6.1 핵심 발견사항

1. **STORE-001이 AUTH-008을 포함**: 매장 등록 과정에서 자동으로 사업주 전환 수행
2. **통합 실행이 기본**: 현재 구현은 한 번의 요청으로 두 기능 모두 수행
3. **순차 실행도 가능**: 별도 API를 통해 사업주 전환 후 매장 등록 가능
4. **동시 실행 불가**: 데이터 의존성으로 인해 완전한 병렬 처리는 불가능

### 6.2 권장 사용 패턴

- **일반적인 경우**: STORE-001 API 사용 (AUTH-008 자동 포함)
- **특수한 경우**: AUTH-008 API → STORE-001 API 순차 호출
- **권장하지 않음**: 동시 실행 시도

### 6.3 개발자 가이드라인

1. **프론트엔드 개발자**: 대부분의 경우 매장 등록 API만 호출하면 됨
2. **백엔드 개발자**: 매장 등록 시 사용자 상태 확인 후 적절한 처리 필요
3. **테스트**: 두 시나리오 모두에 대한 테스트 케이스 작성 권장

---

## 🔗 관련 문서

- [소담 전체 기능 목록 v1.0](../project-management/소담_전체기능목록_v1.0.md)
- [AUTH 기능 데이터 처리 절차 문서](./AUTH_기능_데이터_처리_절차_문서.md)
- [API 문서](./FE개발자를_위한_API_및_서비스로직_문서.md)
- [개발 가이드라인](../guidelines/development_guidelines.md)

## 📅 변경 이력

| 날짜         | 버전  | 변경 내용                                 | 작성자    |
|------------|-----|---------------------------------------|--------|
| 2025-07-26 | 1.0 | 초기 작성 - STORE-001과 AUTH-008 기능 플로우 분석 | 소담 개발팀 |

---

**문서 승인:**

- 개발팀장: 이호영
- 백엔드 개발자: [ ]
- 프론트엔드 개발자: [ ]
- 검토 완료일: 2025-07-26
