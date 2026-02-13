# Fitur: Message Blast (📤)

## 1. Overview

Fitur **Message Blast** adalah modul inti dari WhatsApp Tools yang memungkinkan pengguna mengirim pesan massal ke ribuan nomor sekaligus. Fitur ini dirancang dengan mekanisme keamanan tingkat lanjut untuk meminimalkan risiko pemblokiran akun (banned) melalui rotasi akun otomatis, delay cerdas, dan variasi pesan.

Fokus utama fitur ini adalah kemudahan penggunaan melalui wizard step-by-step yang memandu pengguna dari pemilihan akun hingga eksekusi pengiriman.

## 2. Komponen Detail

### 2.1 Delay Presets (Jeda Pengiriman)

**Deskripsi:**
Konfigurasi jeda antar pesan untuk meniru perilaku manusia. Tersedia preset yang disesuaikan dengan "trust level" akun.

**Parameter:**

| Parameter | Tipe | Default | Min | Max | Deskripsi |
|-----------|------|---------|-----|-----|-----------|
| `minDelay` | Integer (detik) | 8 | 1 | 300 | Batas bawah jeda acak antar pesan |
| `maxDelay` | Integer (detik) | 15 | 2 | 600 | Batas atas jeda acak antar pesan |
| `jedaSetiapN` | Integer | 10 | 1 | 1000 | Jumlah pesan sebelum jeda panjang (istirahat) |
| `extraDelay` | Integer (detik) | 30 | 0 | 3600 | Durasi istirahat setelah mengirim N pesan |

**Preset Values:**

| Preset | Deskripsi | Konfigurasi |
|--------|-----------|-------------|
| **Delay Aman (🐢)** | Untuk akun baru/beresiko tinggi | 15-30s delay, istirahat 60s setiap 5 pesan |
| **Delay Normal (🚶)** | Default/Recommended | 8-15s delay, istirahat 30s setiap 10 pesan |
| **Delay Cepat (🏃)** | Untuk akun verified/tua | 3-8s delay, istirahat 15s setiap 20 pesan |
| **Custom (⚙️)** | Konfigurasi manual | User menentukan semua parameter sendiri |

### 2.2 Multi-Sender Rotation

**Deskripsi:**
Mekanisme rotasi otomatis antar akun pengirim yang dipilih. Sistem menggunakan algoritma Round-Robin untuk membagi beban pengiriman secara merata.

**Logika:**
1. **Round-Robin:** Pesan 1 -> Akun A, Pesan 2 -> Akun B, Pesan 3 -> Akun A, dst.
2. **Skip Exhausted:** Jika akun mencapai `dailyLimit` atau terputus, sistem otomatis melewati akun tersebut dan menggunakan akun berikutnya yang tersedia.
3. **Failover:** Jika semua akun gagal/exhausted, job akan otomatis di-pause.

### 2.3 Message Blocks & Variations

**Deskripsi:**
Struktur pesan yang fleksibel mendukung multi-blok (teks, gambar, video) yang dikirim berurutan, serta variasi konten untuk menghindari deteksi spam (fingerprinting).

**Parameter Blok:**

| Parameter | Tipe | Opsi | Deskripsi |
|-----------|------|------|-----------|
| `type` | String | `text`, `media`, `randomized` | Jenis blok pesan |
| `content` | String | - | Isi pesan teks atau caption |
| `mediaUrl` | String | - | URL file untuk blok media |
| `variants` | Array<String> | - | Daftar variasi pesan (hanya untuk tipe `randomized`) |

**Contoh Struktur JSON:**
```json
[
  {
    "type": "text",
    "content": "Halo {nama}! 👋"
  },
  {
    "type": "randomized",
    "variants": [
      "Ada promo spesial buat kamu.",
      "Cek penawaran terbaru kami.",
      "Jangan lewatkan diskon ini."
    ]
  },
  {
    "type": "media",
    "mediaUrl": "https://example.com/image.jpg",
    "content": "Brosur promo"
  }
]
```

### 2.4 Variable Substitution

**Deskripsi:**
Personalisasi pesan menggunakan data dari kolom file Excel/CSV yang diupload.

**Format:** `{nama_kolom}`

**Contoh:**
Jika file Excel memiliki kolom: `name`, `city`, `phone`
Template: `"Halo {name} dari {city}!"`
Hasil: `"Halo Budi dari Jakarta!"`

### 2.5 Warmer Integration

**Deskripsi:**
Integrasi dengan Account Warmer untuk mode pengiriman yang lebih aman. Blast akan bergantian dengan aktivitas warming.

**Parameter:**
| Parameter | Tipe | Deskripsi |
|-----------|------|-----------|
| `mode` | String | `SMART_DELAY` (biasa) atau `WITH_WARMER` |
| `blastPauseAfter` | Integer | Jumlah penerima blast sebelum beralih ke warmer |
| `warmerMessages` | Integer | Jumlah pesan warmer yang dikirim saat blast pause |

## 3. User Flow

1. **Pilih Akun:** User memilih satu atau lebih akun pengirim.
2. **Upload Data:** User upload file Excel/CSV berisi daftar kontak.
3. **Susun Pesan:** User membuat pesan (bisa multi-block) dan melihat preview realtime.
4. **Konfigurasi:** User memilih Delay Preset dan opsi tambahan (skip contacts, schedule).
5. **Eksekusi:** User memantau progress bar dan log pengiriman realtime.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/blast` | Mendapatkan daftar semua job blast |
| `POST` | `/blast` | Membuat job blast baru |
| `GET` | `/blast/:jobId` | Mendapatkan detail job spesifik |
| `POST` | `/blast/:jobId/start` | Memulai/Melanjutkan job |
| `POST` | `/blast/:jobId/pause` | Menghentikan sementara job |
| `POST` | `/blast/:jobId/stop` | Menghentikan job secara permanen |
| `GET` | `/blast/:jobId/log` | Mendapatkan log pengiriman detail |

## 5. WebSocket Events

| Event | Direction | Payload | Deskripsi |
| ----- | --------- | ------- | --------- |
| `blast:progress` | Server → Client | `{ jobId, sent, failed, total, percent }` | Update progress bar |
| `blast:sent` | Server → Client | `{ jobId, recipient, status }` | Notifikasi per pesan terkirim |
| `blast:log` | Server → Client | `{ logEntry }` | Baris baru untuk tabel log |
| `blast:status` | Server → Client | `{ jobId, status }` | Perubahan status job (RUNNING/PAUSED) |

## 6. Integrasi

- **Contacts Management:**
  - Blast mengecek `hasReceivedMessage` untuk fitur "Skip Contacts".
  - Setelah sukses kirim, status kontak diupdate: `hasReceivedMessage = true`, `lastContactedAt = now`.
- **Account Warmer:**
  - Blast dapat memicu job Warmer secara otomatis jika mode `WITH_WARMER` aktif.
- **Analytics:**
  - Setiap pesan sukses/gagal dicatat ke sistem analitik harian per akun.
- **Scheduler:**
  - Job blast dapat dijadwalkan untuk berjalan otomatis pada `TimeWindow` tertentu.
