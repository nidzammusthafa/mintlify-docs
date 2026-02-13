# Fitur: Account Warmer (🔥)

## 1. Overview

Fitur **Account Warmer** bertujuan untuk meningkatkan "Trust Score" akun WhatsApp, terutama akun baru, dengan melakukan simulasi percakapan natural antar akun internal. Fitur ini berjalan otomatis saling membalas pesan antar akun milik pengguna untuk menciptakan riwayat chat yang aktif dan organik.

## 2. Komponen Detail

### 2.1 Pair Generation (Pembuatan Pasangan)

**Deskripsi:**
Sistem otomatis membuat kombinasi pasangan percakapan dari daftar akun yang dipilih pengguna.

**Algoritma:**
Semua kombinasi unik arah (`n * (n-1)`).
Jika User memilih Akun A, B, C:
- A → B
- A → C
- B → A
- B → C
- C → A
- C → B

### 2.2 Delay Presets Warmer

**Deskripsi:**
Pengaturan kecepatan percakapan warmer. Berbeda dengan blast, delay ini mengatur jeda "mengetik" dan membalas.

**Parameter:**

| Parameter | Tipe | Default | Min | Max | Deskripsi |
|-----------|------|---------|-----|-----|-----------|
| `minDelay` | Integer (detik) | 10 | 2 | 300 | Min waktu sebelum membalas |
| `maxDelay` | Integer (detik) | 30 | 5 | 600 | Max waktu sebelum membalas |
| `msgPerPair` | Integer | 5 | 1 | 50 | Jumlah pesan tukar-menukar per sesi pasangan |

**Preset Values:**

| Preset | Deskripsi | Konfigurasi |
|--------|-----------|-------------|
| **Warmer Lambat (🐢)** | Akun sangat baru | Delay 30-60s, 3-5 pesan/pasangan |
| **Warmer Normal (🚶)** | Default | Delay 10-30s, 5-10 pesan/pasangan |
| **Warmer Cepat (🏃)** | Akun verified | Delay 5-15s, 10-15 pesan/pasangan |

### 2.3 Message Templates (Percakapan Natural)

**Deskripsi:**
Kumpulan skenario percakapan yang terlihat alami. Sistem memilih percakapan secara acak agar tidak terlihat robotik.

**Opsi Bahasa:**
- `ID` (Bahasa Indonesia): "Halo", "Apa kabar?", "Lagi dimana?"
- `EN` (English): "Hi", "How are you?", "Any plans?"

### 2.4 Interval Configuration

**Deskripsi:**
Jeda waktu antar sesi warming untuk menghindari aktivitas terus-menerus 24 jam yang mencurigakan.

**Parameter:**
| Parameter | Tipe | Default | Deskripsi |
|-----------|------|---------|-----------|
| `intervalMs` | Integer | 60000 | Jeda istirahat setelah satu putaran round-robin selesai |

## 3. User Flow

1. **Pilih Akun:** User memilih minimal 2 akun yang akan saling warming (disarankan 3+).
2. **Konfigurasi:** User memilih preset (Normal recommended) dan bahasa percakapan.
3. **Review:** User melihat ringkasan jumlah pasangan dan estimasi total pesan.
4. **Start:** User memulai proses warming.
5. **Monitoring:** Dashboard menampilkan percakapan real-time yang sedang terjadi ("A sedang mengetik ke B...").

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/warmer` | List job warmer |
| `POST` | `/warmer` | Buat job warmer baru |
| `GET` | `/warmer/:jobId` | Detail progress warmer |
| `POST` | `/warmer/:jobId/start` | Start warming |
| `POST` | `/warmer/:jobId/pause` | Pause warming |
| `POST` | `/warmer/:jobId/stop` | Stop warming |

## 5. WebSocket Events

| Event | Direction | Payload | Deskripsi |
| ----- | --------- | ------- | --------- |
| `warmer:progress` | Server → Client | `{ jobId, currentPair, sent, total }` | Progress bar update |
| `warmer:activity` | Server → Client | `{ from, to, message, status }` | Menampilkan chat bubble real-time |
| `warmer:completed` | Server → Client | `{ jobId, summary }` | Notifikasi selesai |

## 6. Integrasi

- **Blast Integration:**
  - Warmer bisa dipanggil oleh Job Blast sebagai selingan (Intermission).
  - Contoh: Blast 50 pesan → Pause Blast → Run Warmer 5 menit → Resume Blast.
- **Client Management:**
  - Status koneksi akun dicek real-time. Jika satu akun disconnect, pasangan yang melibatkan akun tersebut di-skip.
- **Scheduler:**
  - Warmer bisa dijadwalkan berjalan otomatis setiap hari pada jam tertentu (misal: pagi sebelum jam kerja).
