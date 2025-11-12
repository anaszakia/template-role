# Dokumentasi Fitur Lupa Password dengan OTP

## Overview
Fitur lupa password menggunakan sistem **OTP (One-Time Password) 5 digit** yang dikirim via email dengan validasi waktu **30 detik**. Sistem ini lebih aman dan modern dibanding link reset password tradisional.

## Files yang Dibuat/Dimodifikasi

### 1. Controllers
- **`app/Http/Controllers/Auth/ForgotPasswordController.php`**
  - Menangani form request lupa password
  - Mengirim link reset password via email

- **`app/Http/Controllers/Auth/ResetPasswordController.php`**
  - Menampilkan form reset password
  - Memproses reset password baru

### 2. Views
- **`resources/views/auth/forgot-password.blade.php`**
  - Halaman form lupa password
  - Design minimalis selaras dengan login/register

- **`resources/views/auth/reset-password.blade.php`**
  - Halaman form reset password
  - Password strength indicator
  - Password confirmation field

### 3. Notifications
- **`app/Notifications/ResetPasswordNotification.php`**
  - Custom email notification untuk reset password
  - Template email dalam Bahasa Indonesia

### 4. Models
- **`app/Models/User.php`**
  - Menambahkan method `sendPasswordResetNotification()`
  - Override default Laravel notification

### 5. Routes
- **`routes/web.php`**
  ```php
  // Forgot Password Routes
  Route::get('/forgot-password', [ForgotPasswordController::class, 'showLinkRequestForm'])
       ->name('password.request');
  
  Route::post('/forgot-password', [ForgotPasswordController::class, 'sendResetLinkEmail'])
       ->name('password.email');
  
  // Reset Password Routes
  Route::get('/reset-password/{token}', [ResetPasswordController::class, 'showResetForm'])
       ->name('password.reset');
  
  Route::post('/reset-password', [ResetPasswordController::class, 'reset'])
       ->name('password.update');
  ```

### 6. Migrations
- **`database/migrations/2025_11_12_034941_create_password_reset_tokens_table.php`**
  - Tabel untuk menyimpan OTP code dan expiry time
  - Struktur: email (primary), otp (5 digit), expires_at, created_at

## Cara Menggunakan

### Flow Lengkap Reset Password dengan OTP:

#### Step 1: Request OTP
1. User klik "Lupa password?" di halaman login
2. User masukkan email yang terdaftar
3. Sistem generate OTP 5 digit random
4. Sistem kirim OTP via email
5. Tampilan berubah ke form verifikasi OTP

#### Step 2: Verifikasi OTP (30 Detik)
1. User check email dan copy kode OTP 5 digit
2. User masukkan OTP di form verifikasi
3. Timer countdown 30 detik dimulai
4. Jika valid → lanjut ke reset password
5. Jika expired/salah → bisa request ulang

#### Step 3: Reset Password
1. User masukkan password baru + konfirmasi
2. System update password di database
3. OTP dihapus dari database
4. User redirect ke login dengan success message

## Konfigurasi Email

Pastikan konfigurasi email di `.env` sudah benar:

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your-email@gmail.com
MAIL_FROM_NAME="${APP_NAME}"
```

### Testing dengan Mailtrap (Development)
```env
MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your-mailtrap-username
MAIL_PASSWORD=your-mailtrap-password
MAIL_ENCRYPTION=null
```

## Security Features

1. **OTP Expiration**: OTP hanya valid **30 detik** (sangat singkat untuk keamanan maksimal)
2. **Email Verification**: OTP hanya dikirim ke email yang terdaftar di sistem
3. **One-Time Use**: OTP otomatis dihapus setelah berhasil digunakan
4. **Random Generation**: OTP 5 digit random (00000-99999)
5. **Password Hashing**: Password baru di-hash dengan bcrypt
6. **Remember Token Reset**: Remember token di-regenerate setelah reset
7. **Real-time Countdown**: User bisa lihat sisa waktu OTP di UI

## Fitur Tambahan

### OTP UI Features
- **5 Input Box Terpisah**: Setiap digit OTP punya input sendiri
- **Auto-focus**: Otomatis pindah ke input berikutnya
- **Countdown Timer**: Tampilan real-time 00:30 detik
- **Resend Option**: Tombol kirim ulang OTP setelah expired
- **Copy-Paste Support**: Bisa paste 5 digit sekaligus

### Password Strength Indicator
- Lemah: < 8 karakter atau hanya lowercase
- Sedang: >= 8 karakter + uppercase/lowercase
- Kuat: >= 8 karakter + uppercase + lowercase + angka + special char

### User Experience
- Design minimalis selaras dengan login/register
- 2-step form (email → OTP verification)
- Animasi smooth transition
- Error messages yang jelas dan informatif
- Email template profesional dengan styling
- Mobile responsive
- AJAX request (no page reload)

## Testing

### Test Flow Lengkap:

```bash
# 1. Setup Email (Mailtrap untuk testing)
- Buka https://mailtrap.io
- Copy SMTP credentials
- Update .env dengan credentials Mailtrap

# 2. Test Request OTP
- Buka http://localhost/forgot-password
- Masukkan email user yang terdaftar
- Check inbox Mailtrap → lihat email OTP

# 3. Test Verifikasi OTP
- Copy 5 digit OTP dari email
- Masukkan di form verifikasi
- Perhatikan countdown timer (30 detik)
- Jika berhasil → redirect ke reset password

# 4. Test Reset Password
- Masukkan password baru
- Konfirmasi password
- Submit → redirect ke login
- Login dengan password baru

# 5. Test Edge Cases
- Test dengan email tidak terdaftar → error
- Test OTP salah → error message
- Tunggu > 30 detik → OTP expired
- Test resend OTP → dapat OTP baru
```

## Troubleshooting

### Email OTP tidak terkirim
- **Cek konfigurasi SMTP di `.env`**
  ```env
  MAIL_MAILER=smtp
  MAIL_HOST=sandbox.smtp.mailtrap.io
  MAIL_PORT=2525
  MAIL_USERNAME=your-username
  MAIL_PASSWORD=your-password
  ```
- **Cek log error**: `storage/logs/laravel.log`
- **Test koneksi SMTP**: Pastikan credentials benar
- **Firewall**: Pastikan port SMTP tidak diblokir

### OTP expired/invalid
- OTP hanya valid **30 detik**
- Klik "Kirim ulang kode" untuk request OTP baru
- Pastikan waktu server sudah sinkron

### OTP tidak cocok
- Pastikan copy exact 5 digit dari email
- Perhatikan huruf capital (meski OTP angka semua)
- Coba request OTP baru jika masih error

### Countdown tidak jalan
- Cek JavaScript console untuk error
- Pastikan browser support JavaScript
- Clear cache browser

### Email tidak terdaftar
- Pastikan user sudah register
- Cek tabel `users` di database
- Email harus exact match (case-sensitive)

## Database Structure

```sql
CREATE TABLE password_reset_tokens (
    email VARCHAR(255) PRIMARY KEY,
    otp VARCHAR(5) NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP NULL
);
```

## API Endpoints

```php
POST /forgot-password/send-otp
{
    "email": "user@example.com"
}
Response: {
    "success": true,
    "message": "Kode OTP telah dikirim",
    "email": "user@example.com"
}

POST /forgot-password/verify-otp
{
    "email": "user@example.com",
    "otp": "12345"
}
Response: {
    "success": true,
    "message": "Kode OTP valid",
    "email": "user@example.com"
}

POST /reset-password
{
    "email": "user@example.com",
    "password": "newpassword",
    "password_confirmation": "newpassword"
}
```

## Notes
- ⏱️ **OTP berlaku hanya 30 detik** (sangat singkat untuk keamanan)
- 🔢 **OTP 5 digit angka** (00000 - 99999)
- 📧 **Email template HTML** dengan design profesional
- 🔄 **Auto-delete OTP** setelah digunakan atau expired
- 🎨 **UI/UX modern** dengan countdown timer real-time
- 📱 **Mobile responsive** dan user-friendly
- ✅ Semua text dalam **Bahasa Indonesia**
- 🔒 Terintegrasi dengan sistem **authentication** yang ada
