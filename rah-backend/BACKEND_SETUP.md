# RAH E NOOR BACKEND - Setup & API Documentation

## 📋 Prerequisites
- PHP 8.2+
- Composer
- MySQL/MariaDB
- Laravel 12

---

## 🚀 Installation Steps

### 1. Install Laravel Sanctum
```powershell
cd rah-backend
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

### 2. Configure Environment (.env)
Ensure these are set in your `.env` file:
```env
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=rah_backend
DB_USERNAME=root
DB_PASSWORD=

# OTP Settings (optional, defaults work)
OTP_MAX_ATTEMPTS=5
OTP_LOCK_MINUTES=30
OTP_RESEND_COOLDOWN_SECONDS=60
OTP_MAX_SENDS_PER_DAY=6

# WhatsApp integration (for production)
ZENDER_API_KEY=your_key_here
ZENDER_DRY_RUN=true
```

### 3. Run Migrations & Seeders
```powershell
php artisan migrate
php artisan db:seed --class=DaroodTypeSeeder
```

### 4. Create Admin User (optional)
Run in `php artisan tinker`:
```php
$user = \App\Models\User::create([
    'name' => 'Admin User',
    'email' => 'admin@rahenoor.com',
    'phone_e164' => '+919876543210',
    'phone_verified_at' => now(),
    'password' => bcrypt('admin123'),
    'is_admin' => true,
    'city' => 'Rajkot',
]);
```

### 5. Start Development Server
```powershell
php artisan serve --host=0.0.0.0 --port=8000
```

Your API will be available at: `http://localhost:8000/api`

---

## 🔑 API Authentication

### Token-Based Auth (Sanctum)
All authenticated routes require the `Authorization` header:
```
Authorization: Bearer {your_token_here}
```

### Obtaining Token

**Option 1: Registration Flow**
```http
POST /api/register/start
Content-Type: application/json

{
  "name": "John Doe",
  "phone": "9876543210",
  "password": "secure123",
  "city": "Rajkot"
}

# Response: { "ok": true, "registration_id": "uuid" }

POST /api/register/complete
Content-Type: application/json

{
  "registration_id": "uuid",
  "otp": "123456"
}

# Response: { "ok": true, "token": "sanctum_token", "user": {...} }
```

**Option 2: Login with OTP**
```http
POST /api/login/start
Content-Type: application/json

{
  "phone": "9876543210"
}

# Response: { "ok": true, "login_id": "uuid" }

# Use verifyApi instead of verify for token response
POST /api/login/verify
Content-Type: application/json

{
  "login_id": "uuid",
  "otp": "123456"
}

# For API: You'll need to update LoginController to use verifyApi or modify verify to return token based on request type
```

**Option 3: Password Login (Fallback)**
```http
POST /api/login/password
Content-Type: application/json

{
  "phone": "9876543210",
  "password": "secure123"
}

# Response: { "ok": true, "token": "sanctum_token", "user": {...} }
```

---

## 📡 API Endpoints

### Public Endpoints (No Auth)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/register/start` | Start registration with OTP |
| POST | `/api/register/resend` | Resend OTP |
| POST | `/api/register/complete` | Complete registration with OTP |
| POST | `/api/login/start` | Start login with OTP |
| POST | `/api/login/resend` | Resend login OTP |
| POST | `/api/login/verify` | Verify login OTP |
| POST | `/api/login/password` | Password-based login |

### Authenticated Endpoints

#### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/logout` | Logout (revoke token) |

#### Profile
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/profile` | Get user profile |
| PATCH | `/api/profile` | Update profile |

**PATCH /api/profile** payload:
```json
{
  "name": "Updated Name",
  "city": "Ahmedabad",
  "daily_goal": 2000,
  "preferred_mode": "tap",
  "privacy_show_initials": false,
  "privacy_show_city": true
}
```

#### Darood Logs
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/logs` | Create new log |
| DELETE | `/api/logs/{id}` | Delete log (undo) |

**POST /api/logs** payload:
```json
{
  "darood_type_id": 1,
  "count": 100,
  "source": "tap"
}
```

#### Darood Types
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/darood-types` | List all active darood types |

#### Leaderboard
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/leaderboard?scope={city/global}&range={season/month/week/today}&city={cityname}` | Get leaderboard |

**Query params:**
- `scope`: `city` or `global`
- `range`: `season`, `month`, `week`, `today`
- `city`: City filter (required when scope=city)

#### Stats
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/stats/today-week` | Today + week stats |
| GET | `/api/stats/streak` | Current & longest streak |
| GET | `/api/stats/season` | Season progress |

### Admin Endpoints (Require `is_admin=true`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/admin/kpi` | Dashboard KPIs |
| GET | `/api/admin/leaderboard` | Admin leaderboard view |
| GET | `/api/admin/activity` | Recent activity |
| GET | `/api/admin/darood-types` | List darood types |
| POST | `/api/admin/darood-types` | Create darood type |
| PATCH | `/api/admin/darood-types/{id}` | Update darood type |
| DELETE | `/api/admin/darood-types/{id}` | Delete darood type |

---

## 🧪 Testing with cURL

### Get Token
```bash
curl -X POST http://localhost:8000/api/login/password \
  -H "Content-Type: application/json" \
  -d '{"phone":"9876543210","password":"admin123"}'
```

### Protected Request
```bash
curl -X GET http://localhost:8000/api/profile \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Accept: application/json"
```

### Create Log
```bash
curl -X POST http://localhost:8000/api/logs \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{"darood_type_id":1,"count":100,"source":"tap"}'
```

---

## 🔧 Troubleshooting

### CORS Issues
Ensure `config/cors.php` is configured (already done):
```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_origins' => ['*'],
'allowed_headers' => ['*'],
```

### Token Not Working
1. Ensure Sanctum is installed: `composer show laravel/sanctum`
2. Check migrations: `php artisan migrate:status`
3. Verify User model has `HasApiTokens` trait
4. Check header format: `Authorization: Bearer {token}`

### OTP Not Sending
For development, set `ZENDER_DRY_RUN=true` in `.env`. Check logs for OTP code.

---

## 📁 Project Structure

```
rah-backend/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── LoginController.php
│   │   │   │   ├── RegistrationController.php
│   │   │   │   ├── ProfileController.php
│   │   │   │   ├── LogsController.php
│   │   │   │   ├── DaroodTypesController.php
│   │   │   │   ├── LeaderboardController.php
│   │   │   │   └── StatsController.php
│   │   │   └── Admin/
│   │   │       ├── KpiAdminController.php
│   │   │       ├── LeaderboardAdminController.php
│   │   │       ├── ActivityAdminController.php
│   │   │       └── DaroodTypesAdminController.php
│   │   └── Middleware/
│   │       └── EnsureAdmin.php
│   ├── Models/
│   │   ├── User.php
│   │   ├── DaroodLog.php
│   │   ├── DaroodType.php
│   │   ├── PendingLogin.php
│   │   └── PendingRegistration.php
│   └── Services/
│       ├── StatsService.php
│       ├── StreakService.php
│       ├── LeaderboardService.php
│       ├── OtpService.php
│       └── ZenderWhatsappService.php
├── config/
│   └── cors.php
├── database/
│   ├── migrations/
│   └── seeders/
│       └── DaroodTypeSeeder.php
└── routes/
    ├── api.php ✅ (newly created)
    └── web.php
```

---

## 🚦 Next Steps

1. **Install Sanctum**: Run the composer command above
2. **Test Authentication**: Use cURL or Postman
3. **Connect Frontend**: Update React Native app API endpoints
4. **Deploy**: Consider using Laravel Forge or similar

---

## 📝 Notes

- All API responses follow format: `{ "ok": true/false, ... }`
- Phone numbers are normalized to E.164 format (+91XXXXXXXXXX)
- Dates are ISO 8601 format
- Admin routes require `is_admin=true` in users table
- Session-based login exists for web, token-based for mobile

---

## 🆘 Support

For issues or questions:
- Check Laravel logs: `storage/logs/laravel.log`
- Enable debug mode: `APP_DEBUG=true` in `.env`
- Check database queries: `php artisan telescope` (if installed)
