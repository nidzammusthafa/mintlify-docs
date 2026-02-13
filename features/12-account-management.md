# Fitur: Account Management (👥)

## 1. Overview

Fitur **Account Management** memungkinkan pengguna mengelola siklus hidup (lifecycle) koneksi WhatsApp: mulai dari scan QR code, monitoring status koneksi, hingga pemutusan sambungan. Fitur ini mendukung multi-session, artinya aplikasi dapat menjalankan banyak nomor WhatsApp secara bersamaan.

## 2. Komponen Detail

### 2.1 Session Management

**Deskripsi:**
Kontrol penuh terhadap sesi WhatsApp Web (Puppeteer/Baileys).

**Status Sesi:**
- `STOPPED`: Sesi mati/belum jalan.
- `STARTING`: Sedang memuat browser.
- `SCAN_QR`: Menunggu scan QR Code.
- `READY`: Terhubung dan siap kirim pesan.
- `DISCONNECTED`: Terputus (internet mati/logout dari HP).

### 2.2 Connection Methods

**Metode Koneksi:**
1.  **QR Code:** Standard WhatsApp Web login.
2.  **Pairing Code:** Login menggunakan kode 8 digit (tanpa kamera).

### 2.3 Trust Level System

**Deskripsi:**
Sistem klasifikasi akun berdasarkan umur dan kesehatan akun untuk menentukan limit pengiriman yang aman.

**Level:**
1.  **NEWCOMER (T1):** Akun baru connect. Limit rendah, delay tinggi.
2.  **TRUSTED (T2):** Akun aktif > 7 hari. Limit sedang.
3.  **VERIFIED (T3):** Akun tua/bisnis. Limit tinggi, delay minim.

### 2.4 Auto-Recovery

**Deskripsi:**
Mekanisme penyembuhan diri (self-healing). Jika server restart atau crash, sistem akan otomatis mencoba menghubungkan kembali semua sesi yang sebelumnya aktif (READY) tanpa perlu scan ulang.

## 3. User Flow

### Skenario: Menambah Akun Baru
1.  Masuk menu **Accounts**.
2.  Klik **Add Account**.
3.  Pilih metode **Scan QR**.
4.  Muncul QR Code di layar.
5.  User scan menggunakan HP.
6.  Status berubah: `SCAN_QR` -> `READY`.
7.  Sistem menetapkan Trust Level awal: `NEWCOMER`.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/whatsapp/sessions` | List semua sesi & statusnya |
| `POST` | `/whatsapp/sessions` | Buat sesi baru (Start) |
| `DELETE` | `/whatsapp/sessions/:id` | Hapus sesi (Logout) |
| `GET` | `/whatsapp/sessions/:id/qr` | Ambil gambar QR Code terbaru |
| `POST` | `/whatsapp/sessions/:id/restart` | Restart sesi paksa |

## 5. Integrasi

-   **Semua Fitur:** Seluruh fitur (Blast, Inbox, Warmer) bergantung pada `sessionId` yang dikelola di sini.
-   **Settings:** Mengambil konfigurasi limit default berdasarkan Trust Level.
-   **Dashboard:** Menampilkan widget status koneksi (misal: "3 Online, 1 Disconnected").
