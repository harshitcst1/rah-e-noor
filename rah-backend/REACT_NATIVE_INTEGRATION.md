# Connecting React Native App to Laravel Backend

## 📍 API Base URL

Update your React Native app to use the correct API base URL:

```typescript
// Create a config file: app/config/api.ts
export const API_CONFIG = {
  baseURL: __DEV__ 
    ? 'http://10.0.2.2:8000/api'  // Android emulator
    : 'http://localhost:8000/api', // iOS simulator / real device
  timeout: 10000,
};

// For real device testing, use your computer's IP:
// baseURL: 'http://192.168.1.XXX:8000/api'
```

---

## 🔐 Token Storage

```typescript
// Create: app/utils/storage.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

const TOKEN_KEY = '@rah_auth_token';

export const TokenStorage = {
  async save(token: string) {
    await AsyncStorage.setItem(TOKEN_KEY, token);
  },
  
  async get(): Promise<string | null> {
    return await AsyncStorage.getItem(TOKEN_KEY);
  },
  
  async remove() {
    await AsyncStorage.removeItem(TOKEN_KEY);
  },
};
```

Install dependency:
```bash
npm install @react-native-async-storage/async-storage
```

---

## 🌐 API Client

```typescript
// Create: app/services/api.ts
import { API_CONFIG } from '../config/api';
import { TokenStorage } from '../utils/storage';

class ApiClient {
  private baseURL = API_CONFIG.baseURL;

  async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const token = await TokenStorage.get();
    
    const headers: HeadersInit = {
      'Accept': 'application/json',
      'Content-Type': 'application/json',
      ...(options.headers || {}),
    };
    
    if (token) {
      headers['Authorization'] = `Bearer ${token}`;
    }

    const response = await fetch(`${this.baseURL}${endpoint}`, {
      ...options,
      headers,
      timeout: API_CONFIG.timeout,
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.error || 'Request failed');
    }

    return data;
  }

  // Auth
  async register(phone: string, password: string, name: string, city?: string) {
    return this.request('/register/start', {
      method: 'POST',
      body: JSON.stringify({ phone, password, name, city }),
    });
  }

  async completeRegistration(registrationId: string, otp: string) {
    const data = await this.request<{ ok: boolean; token: string; user: any }>(
      '/register/complete',
      {
        method: 'POST',
        body: JSON.stringify({ registration_id: registrationId, otp }),
      }
    );
    
    if (data.token) {
      await TokenStorage.save(data.token);
    }
    
    return data;
  }

  async loginWithPassword(phone: string, password: string) {
    const data = await this.request<{ ok: boolean; token: string; user: any }>(
      '/login/password',
      {
        method: 'POST',
        body: JSON.stringify({ phone, password }),
      }
    );
    
    if (data.token) {
      await TokenStorage.save(data.token);
    }
    
    return data;
  }

  async logout() {
    try {
      await this.request('/logout', { method: 'POST' });
    } finally {
      await TokenStorage.remove();
    }
  }

  // Profile
  async getProfile() {
    return this.request<{ ok: boolean; user: any }>('/profile');
  }

  async updateProfile(data: {
    name: string;
    city?: string;
    daily_goal: number;
    preferred_mode: 'tap' | 'manual';
    privacy_show_initials: boolean;
    privacy_show_city: boolean;
  }) {
    return this.request('/profile', {
      method: 'PATCH',
      body: JSON.stringify(data),
    });
  }

  // Logs
  async createLog(daroodTypeId: number, count: number, source: 'tap' | 'manual') {
    return this.request('/logs', {
      method: 'POST',
      body: JSON.stringify({
        darood_type_id: daroodTypeId,
        count,
        source,
      }),
    });
  }

  async deleteLog(logId: string) {
    return this.request(`/logs/${logId}`, { method: 'DELETE' });
  }

  // Darood Types
  async getDaroodTypes() {
    return this.request<{ ok: boolean; types: any[] }>('/darood-types');
  }

  // Leaderboard
  async getLeaderboard(scope: 'city' | 'global', range: string, city?: string) {
    const params = new URLSearchParams({ scope, range });
    if (city) params.set('city', city);
    return this.request(`/leaderboard?${params.toString()}`);
  }

  // Stats
  async getTodayWeekStats() {
    return this.request('/stats/today-week');
  }

  async getStreak() {
    return this.request('/stats/streak');
  }

  async getSeason() {
    return this.request('/stats/season');
  }
}

export const api = new ApiClient();
```

---

## 🔄 Usage Examples

### Login Screen
```typescript
import { api } from '../services/api';

async function handleLogin() {
  try {
    const result = await api.loginWithPassword(phone, password);
    if (result.ok) {
      // Navigate to dashboard
      router.replace('/user/dashboard');
    }
  } catch (error) {
    Alert.alert('Error', error.message);
  }
}
```

### Dashboard - Load Stats
```typescript
useEffect(() => {
  loadDashboardData();
}, []);

async function loadDashboardData() {
  try {
    const [profile, stats, streak, season] = await Promise.all([
      api.getProfile(),
      api.getTodayWeekStats(),
      api.getStreak(),
      api.getSeason(),
    ]);

    setUser(profile.user);
    setTodayTotal(stats.today_total);
    setWeekSeries(stats.week_series);
    setStreakCurrent(streak.current);
    setStreakLongest(streak.longest);
    // ... etc
  } catch (error) {
    console.error('Failed to load dashboard:', error);
  }
}
```

### Log Screen - Submit Darood
```typescript
async function handleSubmit() {
  try {
    await api.createLog(selectedType.id, count, 'tap');
    Alert.alert('Success', 'Logged successfully');
    setCount(0);
  } catch (error) {
    Alert.alert('Error', error.message);
  }
}
```

### Profile Screen - Update Settings
```typescript
async function saveProfile() {
  try {
    await api.updateProfile({
      name,
      city,
      daily_goal: parseInt(dailyGoal),
      preferred_mode: preferredMode,
      privacy_show_initials: privacyInitials,
      privacy_show_city: privacyCity,
    });
    Alert.alert('Success', 'Profile updated');
  } catch (error) {
    Alert.alert('Error', error.message);
  }
}
```

---

## 🧪 Testing Backend Connection

Add this to your Dashboard to verify connection:

```typescript
useEffect(() => {
  testConnection();
}, []);

async function testConnection() {
  try {
    const response = await fetch('http://10.0.2.2:8000/api/darood-types', {
      headers: { 'Accept': 'application/json' },
    });
    const data = await response.json();
    console.log('✅ Backend connected:', data);
  } catch (error) {
    console.error('❌ Backend error:', error);
    Alert.alert('Connection Error', 'Cannot reach backend. Is Laravel server running?');
  }
}
```

---

## ⚙️ Update Existing Controllers

Replace hardcoded fetch calls in your existing controllers:

### Before (app/user/dashboard.tsx):
```typescript
const res = await fetch('/api/profile', { headers: { Accept: 'application/json' } });
```

### After:
```typescript
const data = await api.getProfile();
```

---

## 🚨 Error Handling

```typescript
import { api } from '../services/api';

try {
  await api.createLog(typeId, count, 'tap');
} catch (error) {
  if (error.message === 'unauthenticated') {
    // Token expired, redirect to login
    await TokenStorage.remove();
    router.replace('/auth/login');
  } else {
    Alert.alert('Error', error.message);
  }
}
```

---

## 📦 Required Package

```bash
cd rah-e-noor
npm install @react-native-async-storage/async-storage
```

---

## ✅ Checklist

- [ ] Install `@react-native-async-storage/async-storage`
- [ ] Create `app/config/api.ts`
- [ ] Create `app/utils/storage.ts`
- [ ] Create `app/services/api.ts`
- [ ] Update all screens to use `api` client instead of raw `fetch`
- [ ] Test login flow end-to-end
- [ ] Test token persistence (close/reopen app)
- [ ] Handle token expiration gracefully

---

## 🔥 Quick Start

1. **Start Laravel backend:**
   ```bash
   cd rah-backend
   php artisan serve --host=0.0.0.0 --port=8000
   ```

2. **Start React Native:**
   ```bash
   cd rah-e-noor
   npm start
   ```

3. **Test login** with admin credentials from BACKEND_SETUP.md

---

## 🐛 Common Issues

### "Network request failed"
- Check if Laravel server is running
- Use correct IP (10.0.2.2 for Android emulator, 127.0.0.1 for iOS)
- Disable HTTPS in development

### "401 Unauthenticated"
- Token expired or invalid
- Clear AsyncStorage and login again
- Check Authorization header format

### "CORS error"
- Ensure `config/cors.php` allows `*` origins in development
- Check Laravel logs: `storage/logs/laravel.log`
