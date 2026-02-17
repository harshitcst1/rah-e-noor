# 🚀 Quick Setup - RAH E NOOR Backend

## Step 1: Install Sanctum
```powershell
cd C:\Users\hm901\Desktop\rah-app\rah-backend
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

## Step 2: Create Admin User
```powershell
php artisan tinker
```

Then run:
```php
$user = \App\Models\User::create([
    'name' => 'Admin User',
    'email' => 'admin@rahenoor.com',
    'phone_e164' => '+919876543210',
    'phone_verified_at' => now(),
    'password' => bcrypt('admin123'),
    'is_admin' => true,
    'city' => 'Rajkot',
    'daily_goal' => 1000,
]);
exit
```

## Step 3: Start Server
```powershell
php artisan serve --host=0.0.0.0 --port=8000
```

## Step 4: Test API
Open new PowerShell terminal:

```powershell
# Test public endpoint
curl http://localhost:8000/api/darood-types

# Get auth token
curl -X POST http://localhost:8000/api/login/password `
  -H "Content-Type: application/json" `
  -d '{\"phone\":\"9876543210\",\"password\":\"admin123\"}'

# Copy the token from response and test protected endpoint
curl http://localhost:8000/api/profile `
  -H "Authorization: Bearer YOUR_TOKEN_HERE" `
  -H "Accept: application/json"
```

## ✅ Success!
If you see JSON responses, your backend is working!

## 📚 Next Steps
- Read [BACKEND_SETUP.md](BACKEND_SETUP.md) for full API documentation
- Read [REACT_NATIVE_INTEGRATION.md](REACT_NATIVE_INTEGRATION.md) to connect your mobile app
- Check [routes/api.php](routes/api.php) for all available endpoints

## 🔧 Troubleshooting

**Error: "Class 'Laravel\Sanctum\HasApiTokens' not found"**
```powershell
composer require laravel/sanctum
```

**Database connection error**
- Check MySQL is running
- Verify `.env` database credentials
- Run: `php artisan config:clear`

**CORS errors from mobile app**
- Backend already configured in `config/cors.php`
- Ensure Laravel is running with `--host=0.0.0.0`
