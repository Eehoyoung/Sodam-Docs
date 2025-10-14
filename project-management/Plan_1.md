# 일반 유저가 사장/직원으로 변경 시 추가정보 입력 기능 구현 계획

## 1. 개요

이 문서는 일반 유저가 사장(MASTER) 또는 직원(EMPLOYEE)으로 역할을 변경할 때 필요한 추가 정보 입력 기능의 구현 계획을 상세히 설명합니다.

## 2. 현재 구현 상태

### 2.1 사장(MASTER) 역할 변경

- 사용자 권한을 MASTER로 변경하는 기능 구현됨
- MasterProfile 엔티티 생성 및 사용자와 연결됨
- 사업자 등록번호 저장 기능 구현됨
- 매장 등록 기능 구현됨

### 2.2 직원(EMPLOYEE) 역할 변경

- 사용자 권한을 EMPLOYEE로 변경하는 기능 구현됨
- EmployeeProfile 엔티티 생성 및 사용자와 연결됨
- 직원 번호 자동 생성 기능 구현됨

## 3. 사장(MASTER) 역할 변경 기능 개선 계획

### 3.1 사업자 등록번호 유효성 검증 로직

#### 3.1.1 기능 요구사항

- 사업자 등록번호 형식 검증 (10자리 숫자)
- 국세청 API 연동을 통한 실제 사업자 등록번호 유효성 검증
- 이미 등록된 사업자 등록번호 중복 검증

#### 3.1.2 구현 계획

1. **BusinessNumberValidator 클래스 구현**
   ```java
   public class BusinessNumberValidator {
       // 형식 검증 (10자리 숫자)
       public boolean validateFormat(String businessNumber) {
           return businessNumber != null && businessNumber.matches("\\d{10}");
       }
       
       // 국세청 API 연동 검증
       public boolean validateWithTaxOffice(String businessNumber) {
           // 국세청 API 연동 로직 구현
           // 실제 API 연동이 어려울 경우 모의 검증 로직 구현
           return true; // 임시 반환값
       }
       
       // 중복 검증
       public boolean isDuplicate(String businessNumber) {
           // 데이터베이스에서 중복 확인 로직
           return false; // 임시 반환값
       }
   }
   ```

2. **StoreManagementService에 검증 로직 추가**
   ```java
   @Service
   public class StoreManagementServiceImpl implements StoreManagementService {
       private final BusinessNumberValidator businessNumberValidator;
       
       // 기존 코드...
       
       @Override
       @Transactional
       public Store registerStoreWithMaster(Long userId, StoreRegistrationDto storeDto) {
           // 사업자 등록번호 유효성 검증
           if (!businessNumberValidator.validateFormat(storeDto.getBusinessNumber())) {
               throw new IllegalArgumentException("사업자 등록번호 형식이 올바르지 않습니다.");
           }
           
           if (!businessNumberValidator.validateWithTaxOffice(storeDto.getBusinessNumber())) {
               throw new IllegalArgumentException("유효하지 않은 사업자 등록번호입니다.");
           }
           
           if (businessNumberValidator.isDuplicate(storeDto.getBusinessNumber())) {
               throw new IllegalArgumentException("이미 등록된 사업자 등록번호입니다.");
           }
           
           // 기존 로직 계속 진행...
       }
   }
   ```

### 3.2 사업자 등록증 이미지 업로드 및 검증

#### 3.2.1 기능 요구사항

- 사업자 등록증 이미지 업로드 기능
- 이미지 파일 형식 및 크기 검증
- 업로드된 이미지와 사업자 등록번호 연결

#### 3.2.2 구현 계획

1. **MasterProfile 엔티티 수정**
   ```java
   @Entity
   public class MasterProfile {
       // 기존 필드...
       
       // 사업자 등록증 이미지 경로 추가
       private String businessLicenseImagePath;
       
       // 사업자 등록증 검증 상태 추가
       @Enumerated(EnumType.STRING)
       private VerificationStatus verificationStatus = VerificationStatus.PENDING;
       
       // 기존 메소드...
   }
   
   public enum VerificationStatus {
       PENDING, // 검증 대기
       VERIFIED, // 검증 완료
       REJECTED // 검증 거부
   }
   ```

2. **FileUploadController 구현**
   ```java
   @RestController
   @RequestMapping("/api/files")
   public class FileUploadController {
       private final FileUploadService fileUploadService;
       private final MasterProfileService masterProfileService;
       
       @PostMapping("/business-license")
       public ResponseEntity<String> uploadBusinessLicense(
               @RequestParam("file") MultipartFile file,
               @RequestParam("userId") Long userId) {
           try {
               String filePath = fileUploadService.uploadImage(file);
               masterProfileService.updateBusinessLicenseImage(userId, filePath);
               return ResponseEntity.ok(filePath);
           } catch (IOException e) {
               return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                       .body("파일 업로드 중 오류가 발생했습니다: " + e.getMessage());
           }
       }
   }
   ```

3. **MasterProfileService 수정**
   ```java
   @Service
   public class MasterProfileService {
       // 기존 코드...
       
       @Transactional
       public MasterProfile updateBusinessLicenseImage(Long masterId, String imagePath) {
           MasterProfile masterProfile = masterProfileRepository.findById(masterId)
                   .orElseThrow(() -> new NoSuchElementException("사장 프로필을 찾을 수 없습니다."));
           
           masterProfile.setBusinessLicenseImagePath(imagePath);
           masterProfile.setVerificationStatus(VerificationStatus.PENDING);
           
           return masterProfileRepository.save(masterProfile);
       }
       
       @Transactional
       public MasterProfile verifyBusinessLicense(Long masterId, boolean isVerified) {
           MasterProfile masterProfile = masterProfileRepository.findById(masterId)
                   .orElseThrow(() -> new NoSuchElementException("사장 프로필을 찾을 수 없습니다."));
           
           masterProfile.setVerificationStatus(isVerified ? 
                   VerificationStatus.VERIFIED : VerificationStatus.REJECTED);
           
           return masterProfileRepository.save(masterProfile);
       }
   }
   ```

### 3.3 추가 사장 정보 입력 UI/UX

#### 3.3.1 기능 요구사항

- 사업자 정보 입력 폼 구현
- 사업자 등록증 이미지 업로드 UI
- 입력 정보 유효성 검증 및 피드백 제공

#### 3.3.2 구현 계획

1. **사장 정보 입력 페이지 구현**
    - 사업자 등록번호 입력 필드
    - 사업자 등록증 이미지 업로드 컴포넌트
    - 매장 정보 입력 필드 (매장명, 업종, 전화번호 등)
    - 위치 정보 입력 (주소 검색 및 지도 표시)

2. **입력 폼 유효성 검증**
    - 실시간 사업자 등록번호 형식 검증
    - 이미지 파일 형식 및 크기 검증
    - 필수 입력 필드 검증

3. **사용자 피드백 UI**
    - 검증 결과 표시 (성공/실패)
    - 오류 메시지 표시
    - 진행 상태 표시 (로딩 인디케이터)

## 4. 직원(EMPLOYEE) 역할 변경 기능 개선 계획

### 4.1 직원 등록 프로세스

#### 4.1.1 기능 요구사항

- 직원이 되고자 하는 매장의 사업자 등록번호 입력
- 입력된 사업자 등록번호로 매장 조회
- 매장 사장의 승인 프로세스

#### 4.1.2 구현 계획

1. **EmployeeRegistrationDto 구현**
   ```java
   @Getter
   @Setter
   public class EmployeeRegistrationDto {
       private Long userId;
       private String businessNumber; // 매장 사업자 등록번호
       private String contactNumber; // 연락처
       private String address; // 주소
       // 추가 정보...
   }
   ```

2. **EmployeeController 구현**
   ```java
   @RestController
   @RequestMapping("/api/employees")
   public class EmployeeController {
       private final StoreManagementService storeManagementService;
       private final EmployeeProfileService employeeProfileService;
       
       @PostMapping("/register")
       public ResponseEntity<?> registerAsEmployee(@RequestBody EmployeeRegistrationDto dto) {
           try {
               // 사업자 등록번호로 매장 조회
               Store store = storeManagementService.findStoreByBusinessNumber(dto.getBusinessNumber());
               
               // 직원 프로필 생성 및 추가 정보 저장
               EmployeeProfile profile = employeeProfileService.createOrUpdateProfile(
                       dto.getUserId(), dto.getContactNumber(), dto.getAddress());
               
               // 직원-매장 관계 생성 (승인 대기 상태)
               storeManagementService.requestEmployeeAssignment(dto.getUserId(), store.getId());
               
               return ResponseEntity.ok(Map.of(
                       "message", "직원 등록 요청이 완료되었습니다. 매장 사장의 승인을 기다려주세요.",
                       "storeId", store.getId(),
                       "storeName", store.getStoreName()
               ));
           } catch (Exception e) {
               return ResponseEntity.badRequest().body(Map.of(
                       "error", e.getMessage()
               ));
           }
       }
   }
   ```

3. **StoreManagementService 수정**
   ```java
   @Service
   public class StoreManagementServiceImpl implements StoreManagementService {
       // 기존 코드...
       
       @Override
       public Store findStoreByBusinessNumber(String businessNumber) {
           return storeRepository.findByBusinessNumber(businessNumber)
                   .orElseThrow(() -> new EntityNotFoundException("해당 사업자 등록번호의 매장을 찾을 수 없습니다."));
       }
       
       @Override
       @Transactional
       public void requestEmployeeAssignment(Long userId, Long storeId) {
           User user = userRepository.findById(userId)
                   .orElseThrow(() -> new EntityNotFoundException("사용자를 찾을 수 없습니다."));
           
           Store store = storeRepository.findById(storeId)
                   .orElseThrow(() -> new EntityNotFoundException("매장을 찾을 수 없습니다."));
           
           // 사용자를 EMPLOYEE로 변경
           user.changeToEmployee();
           
           // EmployeeProfile 생성 또는 조회
           EmployeeProfile employeeProfile;
           Optional<EmployeeProfile> employeeProfileOptional = employeeProfileRepository.findById(userId);
           
           if (employeeProfileOptional.isPresent()) {
               employeeProfile = employeeProfileOptional.get();
           } else {
               employeeProfile = new EmployeeProfile(user);
               employeeProfileRepository.save(employeeProfile);
           }
           
           // 직원-매장 관계 생성 (승인 대기 상태)
           EmployeeStoreRelation relation = new EmployeeStoreRelation(employeeProfile, store);
           relation.setPendingApproval(true); // 승인 대기 상태 설정
           employeeStoreRelationRepository.save(relation);
       }
       
       @Override
       @Transactional
       public void approveEmployeeAssignment(Long employeeId, Long storeId) {
           EmployeeProfile employeeProfile = employeeProfileRepository.findById(employeeId)
                   .orElseThrow(() -> new EntityNotFoundException("직원 프로필을 찾을 수 없습니다."));
           
           Store store = storeRepository.findById(storeId)
                   .orElseThrow(() -> new EntityNotFoundException("매장을 찾을 수 없습니다."));
           
           EmployeeStoreRelation relation = employeeStoreRelationRepository
                   .findByEmployeeProfileAndStore(employeeProfile, store)
                   .orElseThrow(() -> new EntityNotFoundException("직원-매장 관계를 찾을 수 없습니다."));
           
           relation.setPendingApproval(false); // 승인 완료
           employeeStoreRelationRepository.save(relation);
       }
   }
   ```

### 4.2 직원 추가 정보 입력 기능

#### 4.2.1 기능 요구사항

- 직원 연락처, 주소 등 추가 정보 입력
- 직원 프로필 이미지 업로드
- 직원 계약 정보 관리

#### 4.2.2 구현 계획

1. **EmployeeProfile 엔티티 수정**
   ```java
   @Entity
   public class EmployeeProfile {
       // 기존 필드...
       
       private String contactNumber; // 연락처
       private String address; // 주소
       private String profileImagePath; // 프로필 이미지 경로
       
       // 계약 정보
       private LocalDate contractStartDate; // 계약 시작일
       private LocalDate contractEndDate; // 계약 종료일
       private String contractType; // 계약 유형 (정규직, 계약직, 아르바이트 등)
       
       // 기존 메소드...
   }
   ```

2. **EmployeeProfileService 구현**
   ```java
   @Service
   public class EmployeeProfileService {
       private final EmployeeProfileRepository employeeProfileRepository;
       private final UserRepository userRepository;
       
       @Transactional
       public EmployeeProfile createOrUpdateProfile(Long userId, String contactNumber, String address) {
           User user = userRepository.findById(userId)
                   .orElseThrow(() -> new EntityNotFoundException("사용자를 찾을 수 없습니다."));
           
           EmployeeProfile profile = employeeProfileRepository.findById(userId)
                   .orElse(new EmployeeProfile(user));
           
           profile.setContactNumber(contactNumber);
           profile.setAddress(address);
           
           return employeeProfileRepository.save(profile);
       }
       
       @Transactional
       public EmployeeProfile updateProfileImage(Long employeeId, String imagePath) {
           EmployeeProfile profile = employeeProfileRepository.findById(employeeId)
                   .orElseThrow(() -> new EntityNotFoundException("직원 프로필을 찾을 수 없습니다."));
           
           profile.setProfileImagePath(imagePath);
           
           return employeeProfileRepository.save(profile);
       }
       
       @Transactional
       public EmployeeProfile updateContractInfo(
               Long employeeId, LocalDate startDate, LocalDate endDate, String contractType) {
           EmployeeProfile profile = employeeProfileRepository.findById(employeeId)
                   .orElseThrow(() -> new EntityNotFoundException("직원 프로필을 찾을 수 없습니다."));
           
           profile.setContractStartDate(startDate);
           profile.setContractEndDate(endDate);
           profile.setContractType(contractType);
           
           return employeeProfileRepository.save(profile);
       }
   }
   ```

3. **EmployeeController 확장**
   ```java
   @RestController
   @RequestMapping("/api/employees")
   public class EmployeeController {
       // 기존 코드...
       
       @PostMapping("/{employeeId}/profile-image")
       public ResponseEntity<String> uploadProfileImage(
               @PathVariable Long employeeId,
               @RequestParam("file") MultipartFile file) {
           try {
               String filePath = fileUploadService.uploadImage(file);
               employeeProfileService.updateProfileImage(employeeId, filePath);
               return ResponseEntity.ok(filePath);
           } catch (IOException e) {
               return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                       .body("파일 업로드 중 오류가 발생했습니다: " + e.getMessage());
           }
       }
       
       @PostMapping("/{employeeId}/contract")
       public ResponseEntity<EmployeeProfile> updateContractInfo(
               @PathVariable Long employeeId,
               @RequestBody ContractInfoDto contractInfo) {
           EmployeeProfile profile = employeeProfileService.updateContractInfo(
                   employeeId,
                   contractInfo.getStartDate(),
                   contractInfo.getEndDate(),
                   contractInfo.getContractType()
           );
           return ResponseEntity.ok(profile);
       }
   }
   ```

### 4.3 직원 정보 입력 UI/UX

#### 4.3.1 기능 요구사항

- 직원 정보 입력 폼 구현
- 프로필 이미지 업로드 UI
- 계약 정보 입력 및 관리 UI

#### 4.3.2 구현 계획

1. **직원 정보 입력 페이지 구현**
    - 사업자 등록번호 입력 필드 (매장 검색)
    - 연락처, 주소 등 개인 정보 입력 필드
    - 프로필 이미지 업로드 컴포넌트

2. **계약 정보 관리 UI**
    - 계약 시작일/종료일 선택 (달력 컴포넌트)
    - 계약 유형 선택 (드롭다운)
    - 계약 정보 조회 및 수정 기능

3. **사용자 피드백 UI**
    - 매장 검색 결과 표시
    - 입력 정보 유효성 검증 및 오류 메시지
    - 등록 요청 상태 표시 (승인 대기/완료)

## 5. 데이터 모델 변경 사항

### 5.1 MasterProfile 엔티티 변경

- `businessLicenseImagePath` 필드 추가: 사업자 등록증 이미지 경로
- `verificationStatus` 필드 추가: 사업자 등록증 검증 상태 (PENDING, VERIFIED, REJECTED)

### 5.2 EmployeeProfile 엔티티 변경

- `contactNumber` 필드 추가: 연락처
- `address` 필드 추가: 주소
- `profileImagePath` 필드 추가: 프로필 이미지 경로
- `contractStartDate` 필드 추가: 계약 시작일
- `contractEndDate` 필드 추가: 계약 종료일
- `contractType` 필드 추가: 계약 유형

### 5.3 EmployeeStoreRelation 엔티티 변경

- `pendingApproval` 필드 추가: 승인 대기 상태

## 6. API 설계

### 6.1 사장(MASTER) 관련 API

#### 6.1.1 사업자 등록번호 검증 API

- **URL**: `/api/business-number/validate`
- **Method**: POST
- **Request Body**:
  ```json
  {
    "businessNumber": "1234567890"
  }
  ```
- **Response**:
  ```json
  {
    "valid": true,
    "message": "유효한 사업자 등록번호입니다."
  }
  ```

#### 6.1.2 사업자 등록증 이미지 업로드 API

- **URL**: `/api/files/business-license`
- **Method**: POST
- **Request**: MultipartFile + userId
- **Response**:
  ```json
  {
    "filePath": "uploads/12345-abcde.jpg",
    "status": "PENDING"
  }
  ```

#### 6.1.3 매장 등록 API (기존 API 확장)

- **URL**: `/api/stores/change/master`
- **Method**: POST
- **Request Body**: StoreRegistrationDto (기존) + businessLicenseImagePath
- **Response**: Store 객체

### 6.2 직원(EMPLOYEE) 관련 API

#### 6.2.1 직원 등록 요청 API

- **URL**: `/api/employees/register`
- **Method**: POST
- **Request Body**:
  ```json
  {
    "userId": 123,
    "businessNumber": "1234567890",
    "contactNumber": "010-1234-5678",
    "address": "서울시 강남구..."
  }
  ```
- **Response**:
  ```json
  {
    "message": "직원 등록 요청이 완료되었습니다. 매장 사장의 승인을 기다려주세요.",
    "storeId": 456,
    "storeName": "매장명"
  }
  ```

#### 6.2.2 직원 프로필 이미지 업로드 API

- **URL**: `/api/employees/{employeeId}/profile-image`
- **Method**: POST
- **Request**: MultipartFile
- **Response**:
  ```json
  {
    "filePath": "uploads/employee-12345.jpg"
  }
  ```

#### 6.2.3 직원 계약 정보 업데이트 API

- **URL**: `/api/employees/{employeeId}/contract`
- **Method**: POST
- **Request Body**:
  ```json
  {
    "startDate": "2023-01-01",
    "endDate": "2023-12-31",
    "contractType": "PART_TIME"
  }
  ```
- **Response**: EmployeeProfile 객체

#### 6.2.4 직원 등록 승인 API (사장용)

- **URL**: `/api/stores/{storeId}/employees/{employeeId}/approve`
- **Method**: POST
- **Response**:
  ```json
  {
    "message": "직원 등록이 승인되었습니다.",
    "employeeId": 123,
    "employeeName": "홍길동",
    "storeId": 456,
    "storeName": "매장명"
  }
  ```

## 7. 구현 우선순위

1. **사업자 등록번호 유효성 검증 로직**
    - 형식 검증
    - 국세청 API 연동 (또는 모의 검증)
    - 중복 검증

2. **사업자 등록증 이미지 업로드 기능**
    - MasterProfile 엔티티 수정
    - 파일 업로드 API 구현
    - 검증 상태 관리

3. **직원 등록 프로세스**
    - 사업자 등록번호로 매장 검색
    - 직원 프로필 생성 및 추가 정보 저장
    - 승인 대기 상태 관리

4. **직원 추가 정보 입력 기능**
    - EmployeeProfile 엔티티 수정
    - 프로필 이미지 업로드
    - 계약 정보 관리

5. **UI/UX 구현**
    - 사장 정보 입력 폼
    - 직원 정보 입력 폼
    - 피드백 및 상태 표시

## 8. 결론

이 구현 계획은 일반 유저가 사장(MASTER) 또는 직원(EMPLOYEE)으로 역할을 변경할 때 필요한 추가 정보 입력 기능을 상세히 설명합니다. 사장 역할 변경 시에는 사업자 등록번호 유효성 검증, 사업자 등록증
이미지 업로드, 매장 등록 기능이 포함되며, 직원 역할 변경 시에는 매장 사업자 등록번호 입력, 직원 추가 정보 입력, 프로필 이미지 업로드, 계약 정보 관리 기능이 포함됩니다.

이 계획에 따라 구현을 진행하면 사용자 역할 변경 프로세스가 더욱 견고하고 사용자 친화적으로 개선될 것입니다.
