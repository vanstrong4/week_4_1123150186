# 1. Membuat Project Firebase

1. Buka https://firebase.google.com
2. Klik Go to Console.
3. Klik Create a new project.
4. Isi nama project → klik Continue → klik Create Project.
5. Tunggu sampai proses selesai → klik Continue.

### Example
<p align="center">
  <img width="300" src="https://github.com/user-attachments/assets/16391950-3616-4321-9f0d-3c37a5ad4730" />
  <img width="300" src="https://github.com/user-attachments/assets/6102f282-6b8c-49e7-b032-f3d060fbbed4" />
  <img width="300" src="https://github.com/user-attachments/assets/abe77512-a976-4857-a317-1acd7dd2f591" />
  <img width="300" src="https://github.com/user-attachments/assets/6f40a245-af24-4c35-81bd-a2ba4a6df2d2" />
  <img width="300" src="https://github.com/user-attachments/assets/6b6ab384-23ac-468c-9108-bad9351293fc" />
  <img width="300" src="https://github.com/user-attachments/assets/5fa70994-8f6f-433c-9173-44b513e7c9e8" />
</p>

---

# 2. Membuat Web App di Firebase

1. Di dalam project Firebase, klik Add App.
2. Pilih Web.
3. Isi App Nickname.
4. Klik Register App.
Setelah selesai, Firebase akan menampilkan konfigurasi aplikasi.

### Example
<p align="center">
  <img width="300" src="https://github.com/user-attachments/assets/a78167d2-eda9-4af3-bca7-8027e4c392aa" />
  <img width="300" src="https://github.com/user-attachments/assets/e064d897-1799-4d62-b1fe-5d4311b317df" />
  <img width="300" src="https://github.com/user-attachments/assets/d2fee123-f79e-47b1-b7f8-f7f524192d83" />
  <img width="300" src="https://github.com/user-attachments/assets/ae57180f-5538-4c32-8d8c-c6dcd925edc9" />
</p>

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

### Example
<p align="center">
  <img width="300" src="https://github.com/user-attachments/assets/0c890847-062e-4c71-a3fc-1cd7f183212f" />
  <img width="300" src="https://github.com/user-attachments/assets/d8c3b908-b916-4628-815f-17a4502e7d1f" />
  <img width="300" src="https://github.com/user-attachments/assets/d2aaed6e-27cd-4891-afc1-1c80243304e0" />
  <img width="300" src="https://github.com/user-attachments/assets/86a9086b-4a77-4c64-868e-cbaa9f0a1061" />

</p>

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

### Example
<p align="center">
  <img width="" src="https://github.com/user-attachments/assets/3e355348-d6af-41b3-b07f-47e78418fef2" />
</p>

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

### Tutorial 
<p align="center">
  <img width="300" src="https://github.com/user-attachments/assets/d748df23-8729-4826-9683-5eba019bcf0d" />
  <img width="300" src="https://github.com/user-attachments/assets/8a9dfdf6-d914-4d19-a214-a5a013086396" />
  <img width="300" src="https://github.com/user-attachments/assets/81c52b2f-3ac7-4e32-9f69-363b89cbafc9" />
  <img width="300" src="https://github.com/user-attachments/assets/520207a6-583f-4297-857a-f41c1ca00cd9" />
  <img width="300" src="https://github.com/user-attachments/assets/09747f3e-4a1a-4c04-abb5-aeb3030d4018" />
</p>

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

### Tutorial
<p align="center">
  <img width="300" src="https://github.com/user-attachments/assets/33bbeab4-6c36-4b60-ac9f-d2aaf0d8a0d9" />
  <img width="300" src="https://github.com/user-attachments/assets/d47c3bbb-44db-487f-8721-57ed3b1f7180" />
  <img width="300" src="https://github.com/user-attachments/assets/57377ef2-8307-4f31-bfc6-c09d5b6a6461" />
  <img width="300" src="https://github.com/user-attachments/assets/7f855480-fbc9-4c6c-a3be-d212380ec84b" />
  <img width="300" src="https://github.com/user-attachments/assets/a2ecb43a-5b6c-4f17-b979-13c0e98dd819" />
  <img width="300" src="https://github.com/user-attachments/assets/1202cb35-c61d-4b2a-b134-7f0a2b043d60" />
  <img width="300" src="https://github.com/user-attachments/assets/ce6efaa5-cece-441f-9974-03b0319aaa73" />

</p>


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

### Tutorial
<p align="center">
  <img width="300" src="https://github.com/user-attachments/assets/439f340c-1b1f-459b-8228-1e165c6d83ee" />
  <img width="300" src="https://github.com/user-attachments/assets/06951905-d70d-468b-91b6-ba3444209342" />
</p>

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
### Tutorial
<p align="center">
  <img width="300" src="https://github.com/user-attachments/assets/6eeef1be-35d0-4fea-94f7-2a8bbad5d449" />
  <img width="300" src="https://github.com/user-attachments/assets/ab3fcd92-b64a-4d21-b7f3-4570f47e00a9" />
  <img width="300" src="https://github.com/user-attachments/assets/f21faeeb-55a8-40a0-b06e-4502ce61550a" />
  <img width="300" src="https://github.com/user-attachments/assets/1d1dc5a0-32e9-4a05-a330-3b28fec46de8" />
</p>

---

# 9. Step 5 — Backend Verifikasi Token

Setelah login berhasil:
- Client mengirim Firebase ID Token ke backend.

Contoh:
```bash
POST 
/auth/verify-token
```

Backend akan:

1. Memverifikasi token ke Firebase.
2. Mengecek apakah email sudah verified.
3. Membuat user baru di database (jika belum ada).
4. Mengembalikan JWT milik backend.

Contoh response:
```bash
{
  "access_token": "eyJhbGciOiJIUzI1NiJ9...",
  "token_type": "Bearer",
  "expires_in": 86400
}
```

JWT ini berlaku 24 jam.

---

# 10. Step 6 — Request ke Backend
Setiap request ke backend harus menyertakan token ini.

Contoh:

```
GET 
/products
```

Header:
```
Authorization: Bearer BACKEND_TOKEN
```
Jika token tidak ada:
```
401 Unauthorized
```
Jika token expired:
```
TOKEN_EXPIRED
```










