# Product Requirement Document (PRD)

## WazapSuite Desktop Launcher

| Metadata         | Detail                                              |
| :--------------- | :-------------------------------------------------- |
| **Project Name** | WazapSuite Desktop                                  |
| **Version**      | 1.0.4 (Draft)                                       |
| **Status**       | In Development / Stabilization                      |
| **Platform**     | Windows (Primary), macOS/Linux (Secondary)          |
| **Frameworks**   | Electron, NestJS (Server), Next.js (Client), SQLite |

---

## 1. Ringkasan Eksekutif (Executive Summary)

WazapSuite Desktop Launcher bukan sekadar "pembungkus" web. Ini adalah **Application Lifecycle Manager** yang cerdas. Aplikasi ini bertugas membungkus kompleksitas arsitektur _Client-Server_ menjadi satu file executable (`.exe`) yang mudah diinstal oleh pengguna awam (non-teknis). Launcher ini mengelola proses backend, database, pembaruan, dan integrasi sistem operasi secara transparan.

## 2. Tujuan & Sasaran (Objectives)

1.  **Zero-Config Deployment:** Pengguna cukup instal dan jalankan. Tidak perlu install Node.js, setup database, atau command line.
2.  **Robust Process Management:** Memastikan server NestJS berjalan stabil di background, restart otomatis jika crash, dan cleanup saat aplikasi ditutup.
3.  **Environment Isolation:** Mengisolasi dependensi (Socket.IO, Prisma Engines, Puppeteer) agar tidak konflik dengan sistem pengguna.
4.  **Native Integration:** Memberikan pengalaman desktop native (Tray icon, Start on Boot, Notifications).

---

## 3. Fitur Utama (Functional Requirements)

### 3.1 Server Lifecycle Management (Core)

- **Startup Orchestration:**
  - Mengekstrak resource server (`server.zip`) jika versi berubah.
  - Memastikan integritas database (`initial.db` template vs `data.db` user).
  - Menjalankan migrasi database SQLite secara otomatis sebelum server start.
  - Menjalankan `main.exe` (NestJS SEA) sebagai child process dengan environment variable yang diinjeksi dinamis (`PORT`, `DATABASE_URL`).
- **Health Monitoring:**
  - Memantau stdout/stderr dari proses server.
  - Mendeteksi status "Ready" (Socket.IO & HTTP) sebelum memuat UI.
  - Auto-restart jika server crash tiba-tiba.
- **Graceful Shutdown:**
  - Mematikan proses server dan browser (Chrome/Puppeteer) saat aplikasi ditutup untuk mencegah _zombie processes_.

### 3.2 User Interface (Renderer)

- **Splash Screen:** Menampilkan status loading yang detail (misal: "Extracting resources...", "Migrating database...", "Starting WebSocket...").
- **Main Dashboard:** Memuat antarmuka Next.js (frontend) melalui `webview` atau `BrowserWindow`.
- **Settings UI (Native):**
  - Konfigurasi Port Server (Default: 4000).
  - Path Browser Kustom (opsi menggunakan Chrome sistem vs Portable).
  - Toggle "Run in Background".
- **Log Viewer:** Jendela khusus untuk melihat log realtime dari Server (NestJS) dan Client untuk keperluan debugging user.

### 3.3 Database & Storage

- **Persistent Storage:** Menyimpan data pengguna (`data.db`, session WhatsApp) di folder `AppData/Roaming/WazapSuite` (user data directory).
- **Schema Migration:** Mekanisme otomatis untuk menambah kolom/tabel baru pada database pengguna saat aplikasi diupdate (via `better-sqlite3` di main process).

### 3.4 Konektivitas & Real-time

- **Socket.IO Bridge:** Memastikan koneksi WebSocket stabil antara UI (Renderer) dan Backend (Child Process) untuk fitur realtime (QR Code, Status Pesan).
- **Tunneling Manager (Optional):** Integrasi built-in dengan Ngrok/Cloudflare Tunnel untuk akses publik (webhook).

### 3.5 Native Features

- **Tray Icon:** Aplikasi dapat diminimalisir ke system tray.
- **Deep Linking:** Menangani protokol kustom (misal `wazap://`) untuk membuka chat atau aksi tertentu.
- **Auto Updater:** Memeriksa dan mengunduh update aplikasi secara otomatis (via `electron-updater`).

---

## 4. Arsitektur Teknis (Technical Architecture)

### 4.1 Struktur File Production (Setelah Install)

```text
C:\Users\User\AppData\Local\Programs\WazapSuite\
├── WazapSuite.exe (Electron Main)
└── resources\
    ├── app.asar (Source code Electron & Next.js Static Export)
    └── server.zip (Bundled NestJS + Node Modules)

C:\Users\User\AppData\Roaming\WazapSuite\ (UserData)
├── config.json (User preferences)
├── data.db (SQLite Database)
├── server\ (Extracted Server)
│   ├── main.exe (NestJS Single Executable)
│   ├── .env
│   ├── prisma\
│   └── node_modules\ (Native deps: Socket.IO, Prisma Engines)
└── logs\
```

### 4.2 Alur Komunikasi (Data Flow)

1.  **Electron Main** start -> Baca Config -> Spawn `server/main.exe`.
2.  **Server (NestJS)** start -> Init SQLite -> Init Socket.IO Adapter -> Listen Port 4000.
3.  **Electron Main** deteksi Server Ready -> Load URL `http://localhost:4000` (atau static file) ke MainWindow.
4.  **Frontend (Next.js)** load -> Fetch API `GET /health` -> Connect Socket.IO.

---

## 5. Spesifikasi Non-Fungsional

- **Performance:**
  - Waktu startup (Splash screen) < 5 detik pada first run.
  - Penggunaan RAM Idle < 300MB (gabungan Electron + Server).
- **Reliability:**
  - Mekanisme _retry_ koneksi Socket.IO (Polling -> WebSocket).
  - Backup otomatis database sebelum migrasi skema.
- **Security:**
  - Server hanya listen pada `localhost` (127.0.0.1).
  - CORS dibatasi hanya untuk origin Electron.
  - Token sensitif disimpan menggunakan `keytar` (system keychain) jika memungkinkan.

---

## 6. Roadmap Pengembangan (Fase Perbaikan Bug Saat Ini)

### Fase 1: Stabilitas Core (Current Priority)

- [x] Fix WebSocket Race Condition (Server not ready).
- [x] Fix Bundling Issue (Missing dependencies di `.exe`).
- [ ] Stabilisasi Database Path (Mencegah error `malformed disk image`).
- [ ] Implementasi Log Viewer di UI untuk user.

### Fase 2: Peningkatan UX

- [ ] Bundle Chrome Portable (untuk mengeliminasi dependensi Chrome user).
- [ ] Installer yang lebih ringan (Optimasi ukuran `server.zip`).

### Fase 3: Fitur Lanjutan

- [ ] Multi-device management dashboard.
- [ ] Plugin system untuk custom automation script.
