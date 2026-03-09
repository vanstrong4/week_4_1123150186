# 1. Membuat Project Firebase

1. Buka https://firebase.google.com
2. Klik Go to Console.
3. Klik Create a new project.
4. Isi nama project → klik Continue → klik Create Project.
5. Tunggu sampai proses selesai → klik Continue.

---

# 2. Membuat Web App di Firebase

1. Di dalam project Firebase, klik Add App.
2. Pilih Web.
3. Isi App Nickname.
4. Klik Register App.
Setelah selesai, Firebase akan menampilkan konfigurasi aplikasi.

Yang perlu diambil adalah:
```
apiKey
```
Nilai ini akan digunakan di Postman Environment sebagai:
```
FIREBASE_API_KEY
```

---

# 3. Mengaktifkan Authentication

1. Masuk ke menu Authentication.
2. Klik Get Started.
3. Pilih Sign-in Method.
4. Aktifkan Email/Password.
5. Klik Save.
Setelah itu Firebase sudah bisa dipakai untuk register dan login user.

---

# 4. Setup Environment di Postman

1. Buka Postman.
2. Klik Environments di sidebar kiri.
3. Klik Create New Environment.
4. Tambahkan variabel berikut.

| Variable | Initial Value | Keterangan |
|---------|---------|---------|
| FIREBASE_API_KEY  | AIzaSyB_xxx...  | Web API Key dari Firebase Console  |
| FIREBASE_ID_TOKEN  |   | Diisi otomatis setelah login (via Test script)  |
| BACKEND_BASE_URL  | http://localhost:8080/v1 | Base URL backend kamu  |
| BACKEND_TOKEN  |   | Token JWT dari backend (diisi setelah verify)  |
| USER_EMAIL  | test@example.com | Email untuk testing  |
| USER_PASSWORD  | Test@12345 | Password untuk testing  |

Environment ini dipakai agar tidak perlu mengetik ulang data di setiap request.

---

# 5. Step 1 — Register User
Digunakan untuk membuat akun baru di Firebase.

### ENDPOINT
```bash
POST
https://identitytoolkit.googleapis.com/v1/accounts:signUp?key={{FIREBASE_API_KEY}}
```

### HEADERS

| Key | Value | Keterangan |
|---------|---------|---------|
| Content-Type  | application/json | Wajib untuk semua Firebase REST API  |

### Request Body (raw JSON)
```bash
{
  "email": "{{USER_EMAIL}}",
  "password": "{{USER_PASSWORD}}",
  "returnSecureToken": true
}
```

### Response
- Sukses
```bash
Response: 200 OK
{
  "kind": "identitytoolkit#SignupNewUserResponse",
  "localId": "aBcDeFgHiJkLmN",
  "email": "test@example.com",
  "displayName": "",
  "idToken": "eyJhbGciOiJSUzI1...",
  "registered": false,
  "refreshToken": "AMf-vBxK...",
  "expiresIn": "3600"
}
```

- Error
```bash
Response: 400 Bad Request
{
  "error": {
    "code": 400,
    "message": "EMAIL_EXISTS",
    "status": "INVALID_ARGUMENT"
  }
}
```


| Error Code | Artinya | Solusi |
|---------|---------|---------|
| EMAIL_EXISTS  | Email sudah terdaftar di Firebase | Gunakan email lain atau cek apakah sudah register  |
| INVALID_EMAIL  | Format email salah | Pastikan format email benar:user@domain.com  |
| WEAK_PASSWORD  | Password kurang dari 6 karakter | Gunakan password minimal 6 karakter  |
| OPERATION_NOT_ALLOWED  | Email/Password auth belum diaktifkan | Aktifkan di Firebase Console → Authentication → Sign-in method  |
| TOO_MANY_ATTEMPTS_TRY_LATER  | Terlalu banyak percobaan | Tunggu beberapa menit, atau clear IP block di Firebase Console  |

### Postman Test Script — Auto-save Token
- Copy paste ke tab "Tests" di Postman agar idToken tersimpan otomatis:
```bash
// Postman → Tests tab:
const json = pm.response.json();
if (pm.response.code === 200) {
  pm.environment.set("FIREBASE_ID_TOKEN", json.idToken);
  pm.environment.set("FIREBASE_LOCAL_ID", json.localId);
  pm.environment.set("FIREBASE_REFRESH_TOKEN", json.refreshToken);
  console.log("Register sukses. UID:", json.localId);
  console.log("PERHATIAN: Email belum diverifikasi. Lanjut ke Step 2.");
} else {
  console.log("Register gagal:", json.error.message);
}
```
---

# 6. Step 2 — Kirim Email Verifikasi
Digunakan untuk mengirim email verifikasi ke user.

### ENDPOINT
```bash
POST
https://identitytoolkit.googleapis.com/v1/accounts:sendOobCode?key={{FIREBASE_API_KEY}}
```

### HEADERS

| Key | Value | Keterangan |
|---------|---------|---------|
| Content-Type  | application/json | Wajib untuk semua Firebase REST API  |

### Request Body (raw JSON)
```bash
{
  "requestType": "VERIFY_EMAIL",
  "idToken": "{{FIREBASE_ID_TOKEN}}"
}
```

### Response
- Sukses
```bash
Response: 200 OK
{
  "kind": "identitytoolkit#GetOobConfirmationCodeResponse",
  "email": "test@example.com" // ← Email tujuan pengiriman
}
```

- Error
```bash
Response: 400 Bad Request
{
  "error": {
    "code": 400,
    "message": "INVALID_ID_TOKEN", // ← idToken sudah kadaluarsa
    "status": "INVALID_ARGUMENT"
  }
}
```


| Error Code | Artinya | Solusi |
|---------|---------|---------|
| INVALID_ID_TOKEN  | idToken kadaluarsa atau tidak valid | Login ulang di Step 4 untuk mendapat idToken baru, lalu kirim ulang  |
| USER_NOT_FOUND  | User tidak ditemukan di Firebase | Pastikan register berhasil di Step 1 |
| TOO_MANY_ATTEMPTS_TRY_LATER  | Firebase membatasi pengiriman email | Tunggu beberapa menit  |


### Postman Test Script
```bash
// Postman → Tests tab:
if (pm.response.code === 200) {
  const json = pm.response.json();
  console.log("Email verifikasi dikirim ke:", json.email);
  console.log("Sekarang buka inbox email dan klik link verifikasi.");
  console.log("Setelah klik, lanjut ke Step 3 untuk cek status.");
} else {
  console.log("Gagal kirim email:", pm.response.json().error.message);
}
```
---
# 7. Step 3 — Cek Status Verifikasi Email
Digunakan untuk mengecek apakah email sudah diverifikasi.

### ENDPOINT (A)

```bash
POST
https://identitytoolkit.googleapis.com/v1/accounts:lookup?key={{FIREBASE_API_KEY}}
```

### Request Body (raw JSON)
```bash
{
  "idToken": "{{FIREBASE_ID_TOKEN}}"
}
```

### Response
- Sukses
```bash
Response: 200 OK (email belum verify)
{
  "kind": "identitytoolkit#GetAccountInfoResponse",
  "users": [
      {
        "localId": "aBcDeFgHiJkLmN",
        "email": "test@example.com",
        "displayName": "Test User",
        "passwordHash": "UkVEQUNURUQ=",
        "emailVerified": false, // ← BELUM DIVERIFIKASI
        "passwordUpdatedAt": 1700000000000,
        "providerUserInfo": [ ... ],
        "validSince": "1700000000",
        "lastLoginAt": "1700000000000",
        "createdAt": "1700000000000"
      }
  ]
}
```
```bash
Response: 200 OK (email verified)
{
  "kind": "identitytoolkit#GetAccountInfoResponse",
  "users": [
      {
        "localId": "aBcDeFgHiJkLmN",
        "email": "test@example.com",
        "emailVerified": true, // ← SUDAH DIVERIFIKASI
        "lastLoginAt": "1700000000000",
        ...
      }
  ]
}
```
---

# 8. Step 4 — Login
Digunakan untuk login menggunakan email dan password.

### ENDPOINT
```bash
POST
https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key={{FIREBASE_API_KEY}}
```

### Request Body (raw JSON)
```bash
{
  "email": "{{USER_EMAIL}}",
  "password": "{{USER_PASSWORD}}",
  "returnSecureToken": true
}
```

### Response
- Sukses
```bash
Response: 200 OK
{
  "kind": "identitytoolkit#VerifyPasswordResponse",
  "localId": "aBcDeFgHiJkLmN",
  "email": "test@example.com",
  "displayName": "Test User",
  "idToken": "eyJhbGciOiJSUzI1...", // ← Firebase ID Token BARU
  "registered": true,
  "refreshToken": "AMf-vBxK...",
  "expiresIn": "3600"
}
```

- Error
```bash
Response: 400 Bad Request
{
  "error": {
    "code": 400,
    "message": "INVALID_PASSWORD",
    "errors": [{ "message": "INVALID_PASSWORD", "domain": "global" }]
  }
}
```


| Error Code | Artinya | Solusi |
|---------|---------|---------|
| INVALID_PASSWORD  | Password salah | Cek kembali password  |
| EMAIL_NOT_FOUND  | Email belum terdaftar | Lakukan register di Step 1 terlebih dahulu  |
| USER_DISABLED  | Akun di-disable oleh admin | Hubungi admin Firebase Console  |
| INVALID_EMAIL  | Format email salah | Pastikan format email benar  |
| TOO_MANY_ATTEMPTS_TRY_LATER  | Login di-block karena terlalu banyak gagal | Tunggu beberapa menit  |

### Postman Test Script — Auto-Update Token
```bash
// Postman → Tests tab:
const json = pm.response.json();
if (pm.response.code === 200) {
  // Update environment dengan idToken BARU hasil login
  pm.environment.set("FIREBASE_ID_TOKEN", json.idToken);
  pm.environment.set("FIREBASE_REFRESH_TOKEN", json.refreshToken);
  console.log("Login berhasil. Token diperbarui.");
  console.log("Lanjut ke Step 5: kirim token ke backend.");
} else {
  console.log("Login gagal:", json.error.message);
}
```












