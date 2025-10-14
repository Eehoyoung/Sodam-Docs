# 사장 마이페이지 완전 구현 가이드 (Master MyPage Complete Implementation Guide)

## 1. 개요

본 문서는 '소담' 애플리케이션의 사장 마이페이지 기능 구현에 관한 완전한 가이드입니다. 백엔드 API 구현부터 React Native 클라이언트 구현까지 모든 과정을 포함합니다. 사장 마이페이지는 매장 운영자(
사장)가 자신의 프로필 정보, 소유한 매장 목록, 통합 통계, 휴가 신청 내역 등을 한눈에 확인할 수 있는 중요한 기능입니다.

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

## 4. 백엔드 구현

### 4.1 사장 마이페이지 데이터 조회 API

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
    // 사장이 소유한 매장 목록 조회
    List<Store> stores = getStoresByMaster(masterId);
    
    int totalStores = stores.size();
    int totalEmployees = 0;
    long totalLaborCost = 0;
    int totalPendingTimeOffRequests = 0;

    // 현재 월 기준으로 통계 계산
    String currentMonth = LocalDate.now().format(DateTimeFormatter.ofPattern("yyyy-MM"));

    for (Store store : stores) {
        Map<String, Object> storeStats = getStoreStats(store.getId(), currentMonth);
        totalEmployees += (int) storeStats.get("totalEmployees");
        totalLaborCost += (long) storeStats.get("totalLaborCost");
        totalPendingTimeOffRequests += (int) storeStats.get("pendingTimeOffRequests");
    }

    Map<String, Object> combinedStats = new HashMap<>();
    combinedStats.put("totalStores", totalStores);
    combinedStats.put("totalEmployees", totalEmployees);
    combinedStats.put("totalLaborCost", totalLaborCost);
    combinedStats.put("pendingTimeOffRequests", totalPendingTimeOffRequests);
    combinedStats.put("month", currentMonth);

    return combinedStats;
}
```

## 5. React Native 클라이언트 구현

### 5.1 데이터 타입 정의

```javascript
// 네비게이션 타입 정의
type RootStackParamList = {
    Home: undefined;
    Login: undefined;
    StoreDetail: { storeId: number };
    EmployeeManagement: { storeId: number };
    StoreSettings: { storeId: number };
    PayrollManagement: { storeId: number };
    TimeOffApprovals: { storeId: number };
    ProfileEdit: undefined;
    AddStore: undefined;
};

type MasterMyPageScreenNavigationProp = StackNavigationProp<RootStackParamList>;

// 매장 타입 정의
interface Store {
    id: number;
    name: string;
    address: string;
    employeeCount: number;
    monthlyLaborCost: number;
    logoUrl?: string;
}

// 매장 통계 타입 정의
interface StoreStats {
    totalEmployees: number;
    totalLaborCost: number;
    averageHourlyWage: number;
    pendingTimeOffRequests: number;
    month: string;
}

// 직원 타입 정의
interface Employee {
    id: number;
    name: string;
    position: string;
    hourlyWage: number;
    workHours: number;
    profileImageUrl?: string;
}

// 급여 내역 타입 정의
interface Payroll {
    id: number;
    storeId: number;
    storeName: string;
    month: string;
    totalAmount: number;
    employeeCount: number;
    status: 'PENDING' | 'PROCESSED' | 'COMPLETED';
    processedDate: string | null;
}

// 휴가 신청 타입
interface TimeOff {
    id: number;
    employeeId: number;
    employeeName: string;
    storeId: number;
    storeName: string;
    startDate: string;
    endDate: string;
    reason: string;
    status: 'PENDING' | 'APPROVED' | 'REJECTED';
}

// 매출 데이터 타입
interface Revenue {
    month: string;
    amount: number;
}

// 인건비 데이터 타입
interface LaborCost {
    month: string;
    amount: number;
}
```

### 5.2 메인 컴포넌트 구현

```javascript
import React, {useEffect, useState} from 'react';
import {
    ActivityIndicator,
    Alert,
    FlatList,
    Image,
    RefreshControl,
    SafeAreaView,
    ScrollView,
    StyleSheet,
    Text,
    TouchableOpacity,
    View,
} from 'react-native';
import {useNavigation} from '@react-navigation/native';
import {StackNavigationProp} from '@react-navigation/stack';

const MasterMyPageScreen = () => {
    const navigation = useNavigation<MasterMyPageScreenNavigationProp>();

    // 상태 관리
    const [isLoading, setIsLoading] = useState(false);
    const [refreshing, setRefreshing] = useState(false);
    const [selectedStore, setSelectedStore] = useState<Store | null>(null);
    const [stores, setStores] = useState<Store[]>([]);
    const [storeStats, setStoreStats] = useState<StoreStats | null>(null);
    const [employees, setEmployees] = useState<Employee[]>([]);
    const [payrolls, setPayrolls] = useState<Payroll[]>([]);
    const [timeOffRequests, setTimeOffRequests] = useState<TimeOff[]>([]);
    const [monthlyRevenue, setMonthlyRevenue] = useState<Revenue[]>([]);
    const [monthlyLaborCost, setMonthlyLaborCost] = useState<LaborCost[]>([]);

    // 데이터 로드 함수
    const loadMasterMyPageData = async () => {
        setIsLoading(true);
        try {
            // 현재 사용자 정보 가져오기 (AsyncStorage에서)
            const currentUser = await getCurrentUser();
            if (!currentUser) {
                Alert.alert('오류', '사용자 정보를 찾을 수 없습니다.');
                return;
            }

            // 마이페이지 데이터 조회
            const response = await getMasterMyPage(currentUser.id);
            
            if (response.success) {
                setStores(response.data.stores);
                setStoreStats(response.data.combinedStats);
                setTimeOffRequests(response.data.timeOffRequests);
                
                // 첫 번째 매장을 기본 선택
                if (response.data.stores.length > 0) {
                    setSelectedStore(response.data.stores[0]);
                    await loadStoreDetails(response.data.stores[0].id);
                }
            } else {
                Alert.alert('오류', '데이터를 불러오는데 실패했습니다.');
            }
        } catch (error) {
            console.error('Load master mypage data error:', error);
            Alert.alert('오류', '네트워크 오류가 발생했습니다.');
        } finally {
            setIsLoading(false);
        }
    };

    // 매장 상세 정보 로드
    const loadStoreDetails = async (storeId: number) => {
        try {
            // 매장 직원 목록 조회
            const employeesResponse = await getStoreEmployees(storeId);
            if (employeesResponse.success) {
                setEmployees(employeesResponse.data);
            }

            // 급여 내역 조회
            const payrollsResponse = await getStorePayrolls(storeId);
            if (payrollsResponse.success) {
                setPayrolls(payrollsResponse.data);
            }

            // 월별 매출 데이터 조회
            const revenueResponse = await getMonthlyRevenue(storeId);
            if (revenueResponse.success) {
                setMonthlyRevenue(revenueResponse.data);
            }

            // 월별 인건비 데이터 조회
            const laborCostResponse = await getMonthlyLaborCost(storeId);
            if (laborCostResponse.success) {
                setMonthlyLaborCost(laborCostResponse.data);
            }
        } catch (error) {
            console.error('Load store details error:', error);
        }
    };

    // 새로고침 함수
    const onRefresh = async () => {
        setRefreshing(true);
        await loadMasterMyPageData();
        setRefreshing(false);
    };

    // 매장 선택 함수
    const selectStore = async (store: Store) => {
        setSelectedStore(store);
        await loadStoreDetails(store.id);
    };

    // 컴포넌트 마운트 시 데이터 로드
    useEffect(() => {
        loadMasterMyPageData();
    }, []);

    // 로딩 화면
    if (isLoading) {
        return (
            <View style={styles.loadingContainer}>
                <ActivityIndicator size="large" color="#007AFF" />
                <Text style={styles.loadingText}>데이터를 불러오는 중...</Text>
            </View>
        );
    }

    return (
        <SafeAreaView style={styles.container}>
            <ScrollView
                style={styles.scrollView}
                refreshControl={
                    <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
                }
            >
                {/* 헤더 섹션 */}
                <View style={styles.header}>
                    <Text style={styles.headerTitle}>사장 마이페이지</Text>
                    <TouchableOpacity
                        style={styles.profileEditButton}
                        onPress={() => navigation.navigate('ProfileEdit')}
                    >
                        <Text style={styles.profileEditButtonText}>프로필 편집</Text>
                    </TouchableOpacity>
                </View>

                {/* 통합 통계 섹션 */}
                {storeStats && (
                    <View style={styles.statsContainer}>
                        <Text style={styles.sectionTitle}>통합 통계</Text>
                        <View style={styles.statsGrid}>
                            <View style={styles.statItem}>
                                <Text style={styles.statValue}>{storeStats.totalStores}</Text>
                                <Text style={styles.statLabel}>총 매장 수</Text>
                            </View>
                            <View style={styles.statItem}>
                                <Text style={styles.statValue}>{storeStats.totalEmployees}</Text>
                                <Text style={styles.statLabel}>총 직원 수</Text>
                            </View>
                            <View style={styles.statItem}>
                                <Text style={styles.statValue}>
                                    {(storeStats.totalLaborCost / 10000).toFixed(0)}만원
                                </Text>
                                <Text style={styles.statLabel}>총 인건비</Text>
                            </View>
                            <View style={styles.statItem}>
                                <Text style={styles.statValue}>{storeStats.pendingTimeOffRequests}</Text>
                                <Text style={styles.statLabel}>대기 중인 휴가</Text>
                            </View>
                        </View>
                    </View>
                )}

                {/* 매장 목록 섹션 */}
                <View style={styles.storesContainer}>
                    <View style={styles.sectionHeader}>
                        <Text style={styles.sectionTitle}>내 매장</Text>
                        <TouchableOpacity
                            style={styles.addButton}
                            onPress={() => navigation.navigate('AddStore')}
                        >
                            <Text style={styles.addButtonText}>+ 매장 추가</Text>
                        </TouchableOpacity>
                    </View>
                    
                    <FlatList
                        data={stores}
                        horizontal
                        showsHorizontalScrollIndicator={false}
                        keyExtractor={(item) => item.id.toString()}
                        renderItem={({ item }) => (
                            <TouchableOpacity
                                style={[
                                    styles.storeCard,
                                    selectedStore?.id === item.id && styles.selectedStoreCard
                                ]}
                                onPress={() => selectStore(item)}
                            >
                                {item.logoUrl ? (
                                    <Image source={{ uri: item.logoUrl }} style={styles.storeLogo} />
                                ) : (
                                    <View style={styles.defaultLogo}>
                                        <Text style={styles.defaultLogoText}>
                                            {item.name.charAt(0)}
                                        </Text>
                                    </View>
                                )}
                                <Text style={styles.storeName}>{item.name}</Text>
                                <Text style={styles.storeAddress}>{item.address}</Text>
                                <Text style={styles.storeEmployeeCount}>
                                    직원 {item.employeeCount}명
                                </Text>
                            </TouchableOpacity>
                        )}
                    />
                </View>

                {/* 선택된 매장 상세 정보 */}
                {selectedStore && (
                    <View style={styles.storeDetailsContainer}>
                        <Text style={styles.sectionTitle}>{selectedStore.name} 상세 정보</Text>
                        
                        {/* 매장 관리 버튼들 */}
                        <View style={styles.managementButtons}>
                            <TouchableOpacity
                                style={styles.managementButton}
                                onPress={() => navigation.navigate('EmployeeManagement', { storeId: selectedStore.id })}
                            >
                                <Text style={styles.managementButtonText}>직원 관리</Text>
                            </TouchableOpacity>
                            <TouchableOpacity
                                style={styles.managementButton}
                                onPress={() => navigation.navigate('PayrollManagement', { storeId: selectedStore.id })}
                            >
                                <Text style={styles.managementButtonText}>급여 관리</Text>
                            </TouchableOpacity>
                            <TouchableOpacity
                                style={styles.managementButton}
                                onPress={() => navigation.navigate('TimeOffApprovals', { storeId: selectedStore.id })}
                            >
                                <Text style={styles.managementButtonText}>휴가 승인</Text>
                            </TouchableOpacity>
                            <TouchableOpacity
                                style={styles.managementButton}
                                onPress={() => navigation.navigate('StoreSettings', { storeId: selectedStore.id })}
                            >
                                <Text style={styles.managementButtonText}>매장 설정</Text>
                            </TouchableOpacity>
                        </View>

                        {/* 직원 목록 */}
                        {employees.length > 0 && (
                            <View style={styles.employeesSection}>
                                <Text style={styles.subsectionTitle}>직원 목록</Text>
                                <FlatList
                                    data={employees.slice(0, 5)} // 최대 5명만 표시
                                    keyExtractor={(item) => item.id.toString()}
                                    renderItem={({ item }) => (
                                        <View style={styles.employeeItem}>
                                            {item.profileImageUrl ? (
                                                <Image 
                                                    source={{ uri: item.profileImageUrl }} 
                                                    style={styles.employeeImage} 
                                                />
                                            ) : (
                                                <View style={styles.defaultEmployeeImage}>
                                                    <Text style={styles.defaultEmployeeImageText}>
                                                        {item.name.charAt(0)}
                                                    </Text>
                                                </View>
                                            )}
                                            <View style={styles.employeeInfo}>
                                                <Text style={styles.employeeName}>{item.name}</Text>
                                                <Text style={styles.employeePosition}>{item.position}</Text>
                                                <Text style={styles.employeeWage}>
                                                    시급 {item.hourlyWage.toLocaleString()}원
                                                </Text>
                                            </View>
                                        </View>
                                    )}
                                />
                                {employees.length > 5 && (
                                    <TouchableOpacity
                                        style={styles.viewMoreButton}
                                        onPress={() => navigation.navigate('EmployeeManagement', { storeId: selectedStore.id })}
                                    >
                                        <Text style={styles.viewMoreButtonText}>
                                            더 보기 (+{employees.length - 5}명)
                                        </Text>
                                    </TouchableOpacity>
                                )}
                            </View>
                        )}
                    </View>
                )}

                {/* 대기 중인 휴가 신청 */}
                {timeOffRequests.length > 0 && (
                    <View style={styles.timeOffSection}>
                        <Text style={styles.sectionTitle}>대기 중인 휴가 신청</Text>
                        <FlatList
                            data={timeOffRequests.slice(0, 3)} // 최대 3개만 표시
                            keyExtractor={(item) => item.id.toString()}
                            renderItem={({ item }) => (
                                <View style={styles.timeOffItem}>
                                    <View style={styles.timeOffHeader}>
                                        <Text style={styles.timeOffEmployeeName}>{item.employeeName}</Text>
                                        <Text style={styles.timeOffStoreName}>{item.storeName}</Text>
                                    </View>
                                    <Text style={styles.timeOffDates}>
                                        {item.startDate} ~ {item.endDate}
                                    </Text>
                                    <Text style={styles.timeOffReason}>{item.reason}</Text>
                                </View>
                            )}
                        />
                        {timeOffRequests.length > 3 && (
                            <TouchableOpacity
                                style={styles.viewMoreButton}
                                onPress={() => navigation.navigate('TimeOffApprovals', { storeId: selectedStore?.id })}
                            >
                                <Text style={styles.viewMoreButtonText}>
                                    더 보기 (+{timeOffRequests.length - 3}건)
                                </Text>
                            </TouchableOpacity>
                        )}
                    </View>
                )}
            </ScrollView>
        </SafeAreaView>
    );
};

// 스타일 정의
const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f5f5f5',
    },
    loadingContainer: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#f5f5f5',
    },
    loadingText: {
        marginTop: 10,
        fontSize: 16,
        color: '#666',
    },
    scrollView: {
        flex: 1,
    },
    header: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        padding: 20,
        backgroundColor: 'white',
        borderBottomWidth: 1,
        borderBottomColor: '#e0e0e0',
    },
    headerTitle: {
        fontSize: 24,
        fontWeight: 'bold',
        color: '#333',
    },
    profileEditButton: {
        paddingHorizontal: 15,
        paddingVertical: 8,
        backgroundColor: '#007AFF',
        borderRadius: 20,
    },
    profileEditButtonText: {
        color: 'white',
        fontSize: 14,
        fontWeight: '600',
    },
    statsContainer: {
        margin: 20,
        padding: 20,
        backgroundColor: 'white',
        borderRadius: 12,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
        elevation: 3,
    },
    sectionTitle: {
        fontSize: 18,
        fontWeight: 'bold',
        color: '#333',
        marginBottom: 15,
    },
    statsGrid: {
        flexDirection: 'row',
        flexWrap: 'wrap',
        justifyContent: 'space-between',
    },
    statItem: {
        width: '48%',
        alignItems: 'center',
        marginBottom: 15,
    },
    statValue: {
        fontSize: 24,
        fontWeight: 'bold',
        color: '#007AFF',
        marginBottom: 5,
    },
    statLabel: {
        fontSize: 14,
        color: '#666',
        textAlign: 'center',
    },
    storesContainer: {
        margin: 20,
    },
    sectionHeader: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        marginBottom: 15,
    },
    addButton: {
        paddingHorizontal: 12,
        paddingVertical: 6,
        backgroundColor: '#34C759',
        borderRadius: 15,
    },
    addButtonText: {
        color: 'white',
        fontSize: 12,
        fontWeight: '600',
    },
    storeCard: {
        width: 150,
        padding: 15,
        marginRight: 15,
        backgroundColor: 'white',
        borderRadius: 12,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
        elevation: 3,
    },
    selectedStoreCard: {
        borderWidth: 2,
        borderColor: '#007AFF',
    },
    storeLogo: {
        width: 50,
        height: 50,
        borderRadius: 25,
        marginBottom: 10,
        alignSelf: 'center',
    },
    defaultLogo: {
        width: 50,
        height: 50,
        borderRadius: 25,
        backgroundColor: '#007AFF',
        justifyContent: 'center',
        alignItems: 'center',
        marginBottom: 10,
        alignSelf: 'center',
    },
    defaultLogoText: {
        color: 'white',
        fontSize: 20,
        fontWeight: 'bold',
    },
    storeName: {
        fontSize: 16,
        fontWeight: 'bold',
        color: '#333',
        marginBottom: 5,
        textAlign: 'center',
    },
    storeAddress: {
        fontSize: 12,
        color: '#666',
        marginBottom: 5,
        textAlign: 'center',
    },
    storeEmployeeCount: {
        fontSize: 12,
        color: '#007AFF',
        textAlign: 'center',
    },
    storeDetailsContainer: {
        margin: 20,
        padding: 20,
        backgroundColor: 'white',
        borderRadius: 12,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
        elevation: 3,
    },
    managementButtons: {
        flexDirection: 'row',
        flexWrap: 'wrap',
        justifyContent: 'space-between',
        marginBottom: 20,
    },
    managementButton: {
        width: '48%',
        paddingVertical: 12,
        backgroundColor: '#f0f0f0',
        borderRadius: 8,
        alignItems: 'center',
        marginBottom: 10,
    },
    managementButtonText: {
        fontSize: 14,
        fontWeight: '600',
        color: '#333',
    },
    employeesSection: {
        marginTop: 20,
    },
    subsectionTitle: {
        fontSize: 16,
        fontWeight: 'bold',
        color: '#333',
        marginBottom: 10,
    },
    employeeItem: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical: 10,
        borderBottomWidth: 1,
        borderBottomColor: '#f0f0f0',
    },
    employeeImage: {
        width: 40,
        height: 40,
        borderRadius: 20,
        marginRight: 12,
    },
    defaultEmployeeImage: {
        width: 40,
        height: 40,
        borderRadius: 20,
        backgroundColor: '#ccc',
        justifyContent: 'center',
        alignItems: 'center',
        marginRight: 12,
    },
    defaultEmployeeImageText: {
        color: 'white',
        fontSize: 16,
        fontWeight: 'bold',
    },
    employeeInfo: {
        flex: 1,
    },
    employeeName: {
        fontSize: 16,
        fontWeight: '600',
        color: '#333',
        marginBottom: 2,
    },
    employeePosition: {
        fontSize: 14,
        color: '#666',
        marginBottom: 2,
    },
    employeeWage: {
        fontSize: 12,
        color: '#007AFF',
    },
    timeOffSection: {
        margin: 20,
        padding: 20,
        backgroundColor: 'white',
        borderRadius: 12,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
        elevation: 3,
    },
    timeOffItem: {
        paddingVertical: 12,
        borderBottomWidth: 1,
        borderBottomColor: '#f0f0f0',
    },
    timeOffHeader: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        marginBottom: 5,
    },
    timeOffEmployeeName: {
        fontSize: 16,
        fontWeight: '600',
        color: '#333',
    },
    timeOffStoreName: {
        fontSize: 12,
        color: '#666',
    },
    timeOffDates: {
        fontSize: 14,
        color: '#007AFF',
        marginBottom: 5,
    },
    timeOffReason: {
        fontSize: 14,
        color: '#666',
    },
    viewMoreButton: {
        paddingVertical: 10,
        alignItems: 'center',
        borderTopWidth: 1,
        borderTopColor: '#f0f0f0',
        marginTop: 10,
    },
    viewMoreButtonText: {
        fontSize: 14,
        color: '#007AFF',
        fontWeight: '600',
    },
});

export default MasterMyPageScreen;
```

## 6. API 서비스 함수

```javascript
// src/services/masterService.js
import api from '../api/apiConfig';

export const getMasterMyPage = async (masterId) => {
    try {
        const response = await api.get(`/api/master/mypage?masterId=${masterId}`);
        return {
            success: true,
            data: response.data,
        };
    } catch (error) {
        console.error('Get master mypage error:', error);
        return {
            success: false,
            message: error.response?.data?.message || 'Network error',
        };
    }
};

export const getStoreEmployees = async (storeId) => {
    try {
        const response = await api.get(`/api/stores/${storeId}/employees`);
        return {
            success: true,
            data: response.data,
        };
    } catch (error) {
        console.error('Get store employees error:', error);
        return {
            success: false,
            message: error.response?.data?.message || 'Network error',
        };
    }
};

export const getStorePayrolls = async (storeId) => {
    try {
        const response = await api.get(`/api/stores/${storeId}/payrolls`);
        return {
            success: true,
            data: response.data,
        };
    } catch (error) {
        console.error('Get store payrolls error:', error);
        return {
            success: false,
            message: error.response?.data?.message || 'Network error',
        };
    }
};

export const getMonthlyRevenue = async (storeId) => {
    try {
        const response = await api.get(`/api/stores/${storeId}/revenue/monthly`);
        return {
            success: true,
            data: response.data,
        };
    } catch (error) {
        console.error('Get monthly revenue error:', error);
        return {
            success: false,
            message: error.response?.data?.message || 'Network error',
        };
    }
};

export const getMonthlyLaborCost = async (storeId) => {
    try {
        const response = await api.get(`/api/stores/${storeId}/labor-cost/monthly`);
        return {
            success: true,
            data: response.data,
        };
    } catch (error) {
        console.error('Get monthly labor cost error:', error);
        return {
            success: false,
            message: error.response?.data?.message || 'Network error',
        };
    }
};
```

## 7. 주요 기능 설명

### 7.1 통합 통계 표시

- 사장이 소유한 모든 매장의 통계를 통합하여 표시
- 총 매장 수, 총 직원 수, 총 인건비, 대기 중인 휴가 신청 수

### 7.2 매장 관리

- 매장 목록을 가로 스크롤로 표시
- 매장 선택 시 해당 매장의 상세 정보 표시
- 매장별 직원 관리, 급여 관리, 휴가 승인, 설정 기능 제공

### 7.3 직원 정보 표시

- 선택된 매장의 직원 목록 표시 (최대 5명)
- 직원 프로필 이미지, 이름, 직책, 시급 정보 표시

### 7.4 휴가 신청 관리

- 모든 매장의 대기 중인 휴가 신청을 통합하여 표시
- 직원명, 매장명, 휴가 기간, 사유 정보 표시

### 7.5 새로고침 기능

- Pull-to-refresh 기능으로 최신 데이터 업데이트

## 8. 성능 최적화

### 8.1 데이터 로딩 최적화

- 초기 로드 시 필수 데이터만 먼저 로드
- 매장 선택 시 해당 매장의 상세 데이터를 별도로 로드

### 8.2 UI 최적화

- FlatList 사용으로 대용량 리스트 성능 최적화
- 이미지 로딩 최적화 및 기본 이미지 제공

### 8.3 메모리 관리

- 컴포넌트 언마운트 시 불필요한 상태 정리
- 이미지 캐싱 및 메모리 누수 방지

## 9. 에러 처리

### 9.1 네트워크 에러

- API 호출 실패 시 사용자에게 적절한 메시지 표시
- 재시도 기능 제공

### 9.2 데이터 검증

- 서버에서 받은 데이터의 유효성 검증
- 필수 데이터 누락 시 기본값 제공

### 9.3 사용자 경험

- 로딩 상태 표시
- 에러 발생 시 명확한 안내 메시지

## 10. 향후 개선 사항

### 10.1 실시간 업데이트

- WebSocket을 통한 실시간 데이터 업데이트
- 푸시 알림 연동

### 10.2 차트 및 분석

- 매출 및 인건비 차트 표시
- 월별/연도별 통계 분석

### 10.3 추가 기능

- 매장 간 직원 이동 관리
- 급여 자동 계산 및 처리
- 세무 관련 기능 연동

## 11. 결론

사장 마이페이지는 매장 운영자가 효율적으로 여러 매장을 관리할 수 있도록 설계된 핵심 기능입니다. 통합 통계, 매장별 상세 관리, 직원 정보, 휴가 관리 등의 기능을 통해 사장이 필요한 모든 정보를 한 화면에서
확인하고 관리할 수 있습니다.

백엔드 API와 React Native 클라이언트가 유기적으로 연동되어 안정적이고 사용자 친화적인 인터페이스를 제공하며, 향후 추가 기능 확장을 위한 확장성도 고려하여 설계되었습니다.
