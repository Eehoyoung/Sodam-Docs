# 모바일 통합 완전 가이드 (Mobile Integration Complete Guide)

## 개요 (Overview)

이 문서는 Sodam 프로젝트를 React Native 모바일 애플리케이션과 연결하기 위한 완전한 가이드입니다. 백엔드 변경 사항부터 React Native 클라이언트 구현까지 모든 과정을 포함합니다.

## 1. 백엔드 변경 사항 (Backend Changes)

### 1.1 CORS 구성 업데이트 (CORS Configuration Update)

React Native 클라이언트가 API에 접근할 수 있도록 CORS 설정을 업데이트했습니다.

**파일:** `src\main\java\com\rich\sodam\config\WebConfig.java`

```java
// 원하는 도메인 허용
config.addAllowedOrigin("http://localhost:3000"); // React 웹 개발 서버
config.addAllowedOrigin("http://localhost:8081"); // React Native 개발 서버
config.addAllowedOrigin("capacitor://localhost"); // Capacitor (하이브리드 앱)
config.addAllowedOrigin("ionic://localhost"); // Ionic (하이브리드 앱)
config.addAllowedOrigin("http://10.0.2.2:8081"); // Android 에뮬레이터에서 localhost 접근
config.addAllowedOrigin("exp://localhost:19000"); // Expo 개발 서버
config.addAllowedOrigin("exp://127.0.0.1:19000"); // Expo 개발 서버 (IP 주소)
```

이 변경으로 다양한 React Native 개발 환경(표준 React Native, Expo, Android 에뮬레이터)에서 API에 접근할 수 있습니다.

### 1.2 Security 구성 수정 (Security Configuration Update)

Spring Security에서 CORS 설정이 올바르게 적용되도록 수정했습니다.

**파일:** `src\main\java\com\rich\sodam\config\SecurityConfig.java`

```java
.cors(cors -> cors.configure(http)) // WebConfig의 corsFilter 빈을 사용하여 CORS 활성화
```

이전에는 CORS가 비활성화되어 있어 WebConfig의 CORS 설정이 적용되지 않았습니다.

### 1.3 인증 메커니즘 개선 (Authentication Mechanism Enhancement)

모바일 클라이언트를 위해 인증 응답에 JWT 토큰을 포함하도록 수정했습니다.

**파일:** `src\main\java\com\rich\sodam\controller\LoginController.java`

카카오 로그인:

```java
// 성공 응답 생성
Map<String, Object> result = new HashMap<>();
result.put("success", true);
result.put("redirectUrl", "/index.html");
result.put("userGrade", authenticationUser.getUserGrade().getValue());
result.put("token", jwtToken); // JWT 토큰을 응답 본문에 포함 (모바일 클라이언트용)
result.put("userId", authenticationUser.getId()); // 사용자 ID도 포함
```

일반 로그인:

```java
// 모바일 클라이언트를 위해 응답 본문에 토큰과 사용자 정보 포함
Map<String, Object> result = new HashMap<>();
result.put("success", true);
result.put("token", jwtToken);
result.put("userId", authenticationUser.get().getId());
result.put("userGrade", authenticationUser.get().getUserGrade().getValue());
```

이 변경으로 React Native 클라이언트가 JWT 토큰을 받아 AsyncStorage에 저장하고 후속 요청에 사용할 수 있습니다.

## 2. React Native 클라이언트 구현 (React Native Client Implementation)

### 2.1 사전 요구사항 (Prerequisites)

- Node.js (v14 or later)
- npm or yarn
- React Native CLI or Expo CLI
- Android Studio (for Android development)
- Xcode (for iOS development, macOS only)

### 2.2 새 React Native 프로젝트 설정 (Setting Up a New React Native Project)

#### React Native CLI 사용:

```bash
npx react-native init SodamMobile
cd SodamMobile
```

#### Expo 사용 (권장):

```bash
npx create-expo-app SodamMobile
cd SodamMobile
```

### 2.3 필수 의존성 설치 (Required Dependencies)

API 통신 및 상태 관리를 위한 의존성을 설치합니다:

```bash
npm install axios @react-native-async-storage/async-storage
```

또는 yarn 사용:

```bash
yarn add axios @react-native-async-storage/async-storage
```

### 2.4 API 구성 (API Configuration)

백엔드 통신을 관리하기 위한 API 구성 파일을 생성합니다:

```javascript
// src/api/apiConfig.js
import axios from 'axios';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Base URL configuration
const API_URL = __DEV__
    ? 'http://10.0.2.2:8080' // Android Emulator pointing to localhost
    : 'https://your-production-api-url.com';

// Create axios instance
const api = axios.create({
    baseURL: API_URL,
    headers: {
        'Content-Type': 'application/json',
    },
});

// Request interceptor to add auth token to requests
api.interceptors.request.use(
    async (config) => {
        const token = await AsyncStorage.getItem('auth_token');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    },
    (error) => {
        return Promise.reject(error);
    }
);

// Response interceptor for error handling
api.interceptors.response.use(
    (response) => {
        return response;
    },
    async (error) => {
        // Handle token expiration or other auth errors
        if (error.response && error.response.status === 401) {
            await AsyncStorage.removeItem('auth_token');
            // Redirect to login or refresh token logic here
        }
        return Promise.reject(error);
    }
);

export default api;
```

### 2.5 인증 구현 (Authentication Implementation)

로그인 및 토큰 관리를 처리하는 인증 서비스를 생성합니다:

```javascript
// src/services/authService.js
import api from '../api/apiConfig';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const login = async (email, password) => {
    try {
        const response = await api.post('/api/login', {
            email,
            password,
        });

        if (response.data.success && response.data.token) {
            // Store token in AsyncStorage
            await AsyncStorage.setItem('auth_token', response.data.token);
            await AsyncStorage.setItem('user_id', response.data.userId.toString());
            await AsyncStorage.setItem('user_grade', response.data.userGrade);
            
            return {
                success: true,
                user: {
                    id: response.data.userId,
                    grade: response.data.userGrade,
                },
                token: response.data.token,
            };
        } else {
            return {
                success: false,
                message: 'Login failed',
            };
        }
    } catch (error) {
        console.error('Login error:', error);
        return {
            success: false,
            message: error.response?.data?.message || 'Network error',
        };
    }
};

export const logout = async () => {
    try {
        await AsyncStorage.multiRemove(['auth_token', 'user_id', 'user_grade']);
        return { success: true };
    } catch (error) {
        console.error('Logout error:', error);
        return { success: false };
    }
};

export const isAuthenticated = async () => {
    try {
        const token = await AsyncStorage.getItem('auth_token');
        return !!token;
    } catch (error) {
        return false;
    }
};

export const getCurrentUser = async () => {
    try {
        const [userId, userGrade] = await AsyncStorage.multiGet(['user_id', 'user_grade']);
        
        if (userId[1] && userGrade[1]) {
            return {
                id: parseInt(userId[1]),
                grade: userGrade[1],
            };
        }
        return null;
    } catch (error) {
        console.error('Get current user error:', error);
        return null;
    }
};
```

### 2.6 예제 API 서비스 (Example API Service)

```javascript
// src/services/apiService.js
import api from '../api/apiConfig';

export const getUserProfile = async (userId) => {
    try {
        const response = await api.get(`/api/users/${userId}`);
        return response.data;
    } catch (error) {
        console.error('Get user profile error:', error);
        throw error;
    }
};

export const getStores = async () => {
    try {
        const response = await api.get('/api/stores');
        return response.data;
    } catch (error) {
        console.error('Get stores error:', error);
        throw error;
    }
};

export const getMasterMyPage = async (masterId) => {
    try {
        const response = await api.get(`/api/master/mypage?masterId=${masterId}`);
        return response.data;
    } catch (error) {
        console.error('Get master mypage error:', error);
        throw error;
    }
};
```

### 2.7 로그인 화면 구현 (Login Screen Implementation)

```javascript
// src/screens/LoginScreen.js
import React, { useState } from 'react';
import {
    View,
    Text,
    TextInput,
    TouchableOpacity,
    Alert,
    StyleSheet,
    ActivityIndicator,
} from 'react-native';
import { login } from '../services/authService';

const LoginScreen = ({ navigation }) => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [isLoading, setIsLoading] = useState(false);

    const handleLogin = async () => {
        if (!email || !password) {
            Alert.alert('오류', '이메일과 비밀번호를 입력해주세요.');
            return;
        }

        setIsLoading(true);
        try {
            const result = await login(email, password);
            
            if (result.success) {
                // Navigate to main app
                navigation.replace('Main');
            } else {
                Alert.alert('로그인 실패', result.message);
            }
        } catch (error) {
            Alert.alert('오류', '로그인 중 오류가 발생했습니다.');
        } finally {
            setIsLoading(false);
        }
    };

    return (
        <View style={styles.container}>
            <Text style={styles.title}>Sodam</Text>
            
            <TextInput
                style={styles.input}
                placeholder="이메일"
                value={email}
                onChangeText={setEmail}
                keyboardType="email-address"
                autoCapitalize="none"
            />
            
            <TextInput
                style={styles.input}
                placeholder="비밀번호"
                value={password}
                onChangeText={setPassword}
                secureTextEntry
            />
            
            <TouchableOpacity
                style={styles.loginButton}
                onPress={handleLogin}
                disabled={isLoading}
            >
                {isLoading ? (
                    <ActivityIndicator color="white" />
                ) : (
                    <Text style={styles.loginButtonText}>로그인</Text>
                )}
            </TouchableOpacity>
        </View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        paddingHorizontal: 20,
        backgroundColor: '#f5f5f5',
    },
    title: {
        fontSize: 32,
        fontWeight: 'bold',
        textAlign: 'center',
        marginBottom: 40,
        color: '#333',
    },
    input: {
        backgroundColor: 'white',
        paddingHorizontal: 15,
        paddingVertical: 12,
        borderRadius: 8,
        marginBottom: 15,
        fontSize: 16,
        borderWidth: 1,
        borderColor: '#ddd',
    },
    loginButton: {
        backgroundColor: '#007AFF',
        paddingVertical: 15,
        borderRadius: 8,
        alignItems: 'center',
        marginTop: 10,
    },
    loginButtonText: {
        color: 'white',
        fontSize: 16,
        fontWeight: 'bold',
    },
});

export default LoginScreen;
```

## 3. 환경별 설정 (Environment Configuration)

### 3.1 개발 환경 (Development Environment)

```javascript
// config/development.js
export const config = {
    API_URL: 'http://10.0.2.2:8080', // Android Emulator
    // API_URL: 'http://localhost:8080', // iOS Simulator
};
```

### 3.2 프로덕션 환경 (Production Environment)

```javascript
// config/production.js
export const config = {
    API_URL: 'https://your-production-api-url.com',
};
```

## 4. 테스트 방법 (Testing Method)

React Native 클라이언트와의 연결을 테스트하려면:

1. Spring Boot 애플리케이션 실행
2. React Native 앱 실행:
   ```bash
   # React Native CLI 사용
   npx react-native run-android
   
   # Expo 사용
   npx expo start
   ```
3. 로그인 기능 및 API 호출 테스트

## 5. 통합 테스트 (Integration Testing)

### 5.1 API 연결 테스트

```javascript
// src/tests/apiTest.js
import { login, logout, isAuthenticated } from '../services/authService';
import { getUserProfile } from '../services/apiService';

export const runAPITests = async () => {
    try {
        console.log('Testing login...');
        const loginResult = await login('test@example.com', 'password');
        console.log('Login result:', loginResult);

        if (loginResult.success) {
            console.log('Testing authenticated API call...');
            const profile = await getUserProfile(loginResult.user.id);
            console.log('Profile:', profile);

            console.log('Testing logout...');
            const logoutResult = await logout();
            console.log('Logout result:', logoutResult);
        }
    } catch (error) {
        console.error('API test error:', error);
    }
};
```

## 6. 일반적인 문제 해결 (Troubleshooting)

### 6.1 네트워크 연결 문제

**문제:** Android 에뮬레이터에서 localhost에 연결할 수 없음
**해결:** `10.0.2.2`를 사용하여 호스트 머신의 localhost에 접근

**문제:** iOS 시뮬레이터에서 API 호출 실패
**해결:** `localhost` 또는 실제 IP 주소 사용

### 6.2 CORS 오류

**문제:** CORS 정책으로 인한 요청 차단
**해결:** 백엔드 WebConfig에서 올바른 origin 추가 확인

### 6.3 인증 토큰 문제

**문제:** 토큰이 요청에 포함되지 않음
**해결:** axios interceptor 설정 확인 및 AsyncStorage에서 토큰 저장/로드 확인

## 7. 향후 계획 (Future Plans)

### 7.1 iOS 지원 (iOS Support)

iOS 지원을 위해 다음 사항을 고려해야 합니다:

- iOS 시뮬레이터에서 API 엔드포인트 작동 확인 (`localhost` 사용)
- iOS 특화 인증 흐름 처리 (특히 소셜 로그인)
- 다양한 iOS 기기 및 버전에서 테스트

### 7.2 웹 지원 (Web Support)

웹 지원을 위해 다음 사항을 고려해야 합니다:

- React Native Web 사용 고려
- 웹에서는 브라우저 쿠키로 인증 작동하도록 조정
- 다양한 화면 크기에 대한 반응형 레이아웃 테스트

### 7.3 추가 기능

- 푸시 알림 구현
- 오프라인 지원
- 생체 인증 (지문, 얼굴 인식)
- 다국어 지원

## 8. 결론 (Conclusion)

이번 변경으로 Sodam 프로젝트는 이제 React Native 모바일 애플리케이션과 완전히 연결할 수 있는 상태가 되었습니다. 특히 Android 연결을 우선적으로 지원하도록 구성했으며, 향후 iOS 및 웹 연결을
위한 기반도 마련되었습니다. CORS 설정, 보안 구성, 인증 메커니즘 개선을 통해 모바일 클라이언트가 백엔드 API와 원활하게 통신할 수 있습니다.

이 가이드를 따라 구현하면 완전한 모바일 애플리케이션을 구축할 수 있으며, 필요에 따라 추가 기능을 확장할 수 있습니다.
