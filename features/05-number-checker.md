# Fitur: Number Checker (🔍)

## 1. Overview

**Number Checker** adalah utilitas untuk memvalidasi apakah sebuah nomor telepon terdaftar di WhatsApp atau tidak. Fitur ini sangat krusial untuk menjaga kesehatan akun pengirim (sender) dengan memastikan pesan hanya dikirim ke nomor aktif, mengurangi rasio kegagalan pengiriman yang bisa memicu deteksi spam.

## 2. Komponen Detail

### 2.1 Check Methods (Metode Cek)

**Deskripsi:**
Sistem mendukung berbagai sumber input nomor untuk diperiksa.

**Sumber Input:**
1. **Input Manual:** Copy-paste daftar nomor (satu per baris).
2. **Upload File:** Upload Excel/CSV berisi kolom nomor telepon.
3. **Database Kontak:** Memilih kontak dari database yang belum divalidasi.

### 2.2 Result States

**Deskripsi:**
Setiap nomor yang dicek akan memiliki salah satu status berikut:

| Status | Icon | Deskripsi |
|---|---|---|
| `active` | ✅ | Nomor terdaftar di WhatsApp |
| `inactive` | ❌ | Nomor tidak terdaftar di WhatsApp |
| `error` | ⚠️ | Gagal mengecek (koneksi/timeout) |
| `processing` | ⏳ | Sedang dalam antrian cek |

### 2.3 Delay Presets

**Deskripsi:**
Pengecekan nomor juga membutuhkan jeda agar akun pengecek tidak terblokir karena aktivitas tidak wajar (checking too fast).

**Preset:**

| Preset | Delay Antar Cek | Jeda Istirahat | Deskripsi |
|---|---|---|---|
| **Aman (🐢)** | 5-10 detik | 60s setiap 20 nomor | Sangat aman, tapi lambat |
| **Normal (🚶)** | 2-5 detik | 30s setiap 50 nomor | Recommended |
| **Cepat (🏃)** | 1-3 detik | 15s setiap 100 nomor | Untuk akun "tumbal" atau sangat kuat |

## 3. User Flow

1. **Input:** User memasukkan 1.000 nomor dari file Excel.
2. **Pilih Akun:** User memilih akun WhatsApp yang akan digunakan untuk mengecek (disarankan akun sekunder/tumbal, bukan akun utama).
3. **Proses:** Sistem mengecek satu per satu sesuai delay preset.
4. **Hasil:** Progress bar berjalan. User melihat counter: 500 Valid, 200 Invalid, 300 Sisa.
5. **Aksi Lanjutan:** Setelah selesai, user memilih "Simpan Nomor Aktif ke Kontak" atau "Langsung Blast ke Nomor Aktif".

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `POST` | `/number-checker` | Buat job pengecekan baru |
| `GET` | `/number-checker/:jobId` | Lihat progress & status job |
| `POST` | `/number-checker/:jobId/start` | Mulai proses pengecekan |
| `POST` | `/number-checker/:jobId/stop` | Hentikan proses |
| `GET` | `/number-checker/:jobId/results` | Download hasil (JSON/Excel) |

## 5. WebSocket Events

| Event | Direction | Payload | Deskripsi |
| ----- | --------- | ------- | --------- |
| `checker:progress` | Server → Client | `{ jobId, checked, total, active, inactive }` | Update progress real-time |
| `checker:result` | Server → Client | `{ phoneNumber, status }` | Hasil per nomor (untuk live list) |

## 6. Integrasi

- **Contacts:**
  - Opsi "Simpan Valid ke Kontak" akan memfilter hasil `active` dan menyimpannya ke database kontak.
- **Message Blast:**
  - Opsi "Blast Nomor Valid" akan membuat job Blast baru dengan recipient list yang sudah dibersihkan (hanya `active`).
