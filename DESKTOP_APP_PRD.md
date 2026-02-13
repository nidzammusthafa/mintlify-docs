# Product Requirement Document (PRD)

## WazapSuite Desktop Launcher

| Metadata         | Detail                                                                    |
| :--------------- | :------------------------------------------------------------------------ |
| **Project Name** | WazapSuite Desktop                                                        |
| **Version**      | 1.0.4                                                                     |
| **Status**       | Active Development                                                        |
| **Platform**     | Windows (Target), macOS/Linux (Supported)                                 |
| **Stack**        | Electron, NestJS (Bundled SEA), Next.js (Static Export/Localhost), SQLite |

---

## 1. Ringkasan Eksekutif

WazapSuite Desktop adalah aplikasi manajemen lifecycle untuk server WhatsApp automation. Aplikasi ini membungkus kompleksitas backend (NestJS), database (SQLite), dan frontend (Next.js) menjadi satu unit yang dapat diinstal (`.exe`). Fokus utamanya adalah memberikan pengalaman "Zero-Config" kepada pengguna akhir, di mana server, database, dan dependensi dikelola secara otomatis di latar belakang.

## 2. Tujuan Bisnis & Teknis

1.  **Perlindungan Kekayaan Intelektual (Anti-Reverse Engineering):** Melindungi source code asli (proprietary) dari akses, penyalinan, atau modifikasi oleh pihak yang tidak berwenang. Logika bisnis dibundle dan di-inject langsung ke dalam binary executable (`.exe`), membuatnya jauh lebih sulit untuk direkayasa ulang dibandingkan distribusi kode JavaScript mentah.
2.  **Kemudahan Distribusi:** Menghilangkan kebutuhan user untuk menginstall Node.js, Git, atau konfigurasi terminal.
3.  **Stabilitas Proses:** Mengelola proses server sebagai _child process_ yang robust (auto-restart, error logging).
4.  **Isolasi Environment:** Menggunakan teknik _Single Executable Application (SEA)_ untuk membundle server dan dependensinya agar tidak terpengaruh environment OS user.
5.  **Keamanan Data:** Data tersimpan lokal di mesin user (`AppData`), menjaga privasi data WhatsApp.

---

## 3. Spesifikasi Fitur (Functional Requirements)

### 3.1 Server Lifecycle Management

- **Server Orchestration:** Electron Main Process bertanggung jawab men-spawn file `main.exe` (NestJS SEA).
- **Resource Extraction:**
  - Mendeteksi perubahan versi aplikasi.
  - Mengekstrak `server.zip` (berisi executable & node_modules penting) ke folder `UserData`.
- **Startup Sequence:**
  1.  Cek integritas file server.
  2.  Migrasi skema Database SQLite (`better-sqlite3` native migration).
  3.  Inject Environment Variables (`PORT`, `DATABASE_URL`, `CHROME_PATH`).
  4.  Spawn process & monitoring stdout/stderr.

### 3.2 Connectivity & Real-time

- **Socket.IO Bridge:**
  - Menyediakan komunikasi real-time antara Frontend (Renderer) dan Backend (Server Process).
  - Handling port dinamis dan konfigurasi CORS yang ketat namun fleksibel untuk localhost.
  - **Fitur:** Scan QR Code, Status Koneksi WA, Progress Scraping.
- **Fallback Mechanism:** Support otomatis transport `websocket` dengan fallback ke `polling` jika jaringan lokal memblokir upgrade header.

### 3.3 Database Management

- **Automated Migrations:** Aplikasi secara otomatis menjalankan perintah DDL (Alter Table) saat mendeteksi database versi lama.
- **Data Persistence:** Database `data.db` disimpan di folder persisten pengguna, terpisah dari file executable yang bisa di-overwrite saat update.

### 3.4 User Interface (Electron Renderer)

- **Splash Screen:** Indikator visual saat server sedang booting atau melakukan migrasi data.
- **Server Status Dashboard:** Menampilkan log realtime dari server untuk diagnosa masalah.
- **Config UI:**
  - Port Selector (Default: 4000).
  - Toggle Headless Mode (untuk debugging browser puppeteer).
  - Chrome Path Selector (Custom path vs Bundled).

---

## 4. Arsitektur Teknis

### 4.1 Struktur Bundle (SEA Strategy)

Untuk mengatasi masalah path dan dependensi native, server dibuild dengan strategi berikut:

- **Core Logic:** Di-bundle menjadi satu file `server.bundle.js` menggunakan `esbuild`.
- **Runtime:** Di-inject ke dalam binary Node.js (SEA Blob Injection).
- **External Dependencies:** Library native (`prisma`, `better-sqlite3`, `socket.io`) diletakkan di folder `node_modules` eksternal yang berdampingan dengan `.exe` untuk stabilitas maksimal.

### 4.2 Alur Komunikasi

1.  **Electron Main** -> Spawn `main.exe`.
2.  **Server (NestJS)** -> Start HTTP & Socket Server di `localhost:PORT`.
3.  **Electron Renderer** -> Load UI Next.js -> Connect ke `http://localhost:PORT` via Fetch & Socket.IO Client.

---

## 5. Build & Deployment Workflow

### 5.1 Pre-Build (Server)

1.  Compile NestJS (`npm run build`).
2.  Bundle dengan `esbuild` (Externalize native modules).
3.  Generate SEA Blob (`sea-config.json`).
4.  Inject Blob ke `node.exe` -> Output: `main.exe`.

### 5.2 Packaging (Desktop)

1.  Copy `main.exe`, `.env`, `package.json`, dan `prisma/schema`.
2.  Copy & Filter `node_modules` (Hanya production deps).
3.  Zip semua resource server menjadi `server.zip`.
4.  Electron Builder memaketkan `server.zip` ke dalam installer aplikasi (`resources/server.zip`).

---

## 6. Masalah Umum & Solusi (Troubleshooting)

- **Socket.IO 404:** Disebabkan oleh client probing ke default namespace sebelum server siap. _Solusi: Implementasi Queueing Event di Gateway._
- **Native Module Error:** Disebabkan path `binding.node` tidak ditemukan. _Solusi: Externalize dependensi di `build-sea.js`._
- **Database Locked/Corrupt:** _Solusi: Graceful shutdown handler di `server-manager.ts`._
