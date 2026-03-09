# 1. Membuat Project Firebase

1. Buka https://firebase.google.com
2. Klik Go to Console.
3. Klik Create a new project.
4. Isi nama project → klik Continue → klik Create Project.
5. Tunggu sampai proses selesai → klik Continue.

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

# 3. Mengaktifkan Authentication

1. Masuk ke menu Authentication.
2. Klik Get Started.
3. Pilih Sign-in Method.
4. Aktifkan Email/Password.
5. Klik Save.
Setelah itu Firebase sudah bisa dipakai untuk register dan login user.

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
























