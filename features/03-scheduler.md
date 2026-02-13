# Fitur: Scheduler System (⏱️)

## 1. Overview

**Scheduler System** adalah mesin otomatisasi yang mengontrol kapan job Blast atau Warmer berjalan. Fitur ini memungkinkan pengguna mengatur "Jam Kerja" untuk bot, memastikan pesan hanya dikirim pada waktu yang tepat dan menghindari pengiriman di luar jam wajar yang bisa memicu laporan spam.

## 2. Komponen Detail

### 2.1 Time Windows (Jendela Waktu)

**Deskripsi:**
Rentang waktu spesifik di mana job diizinkan berjalan. Di luar rentang ini, job akan otomatis berstatus `TIME_PAUSED`.

**Parameter:**

| Parameter | Tipe | Format | Contoh | Deskripsi |
|-----------|------|--------|--------|-----------|
| `start` | String | HH:mm | "09:00" | Waktu mulai aktif |
| `end` | String | HH:mm | "17:00" | Waktu berhenti aktif |

### 2.2 Schedule Configuration

**Deskripsi:**
Objek konfigurasi lengkap yang melekat pada setiap Blast/Warmer Job.

**Parameter:**

| Parameter | Tipe | Default | Deskripsi |
|-----------|------|---------|-----------|
| `type` | Enum | `IMMEDIATE` | `IMMEDIATE` (langsung), `SCHEDULED` (dijadwalkan) |
| `startDate` | DateString | Now | Tanggal mulai job |
| `endDate` | DateString | null | Tanggal otomatis stop job (opsional) |
| `daysOfWeek` | Array[Int] | [0..6] | Hari aktif (0=Minggu, 1=Senin, dst) |
| `timeWindows` | Array[Window] | - | Daftar slot waktu aktif per hari |
| `timezone` | String | "Asia/Jakarta" | Zona waktu referensi |

### 2.3 Auto-Pause & Auto-Resume

**Logika:**
- **Check Interval:** Scheduler mengecek setiap menit (`cron: * * * * *`).
- **On Enter Window:** Jika job status `TIME_PAUSED` dan waktu sekarang masuk window -> ubah ke `RUNNING`.
- **On Exit Window:** Jika job status `RUNNING` dan waktu sekarang keluar window -> ubah ke `TIME_PAUSED`.

### 2.4 Countdown Display

**Deskripsi:**
Informasi visual kepada pengguna mengenai status jadwal.
- Jika `WAITING`: "Mulai dalam 2 jam 30 menit"
- Jika `RUNNING`: "Pause otomatis dalam 45 menit"
- Jika `PAUSED`: "Lanjut lagi besok jam 09:00"

## 3. User Flow

1. **Saat Buat Job:** Di halaman konfigurasi Blast/Warmer, user mengaktifkan toggle "Schedule".
2. **Setup Waktu:** User memilih tanggal mulai dan jam operasional (bisa multiple, misal: 09:00-12:00 & 13:00-17:00).
3. **Simpan Job:** Job tersimpan dengan status `SCHEDULED` (jika belum masuk waktu) atau `RUNNING` (jika sudah masuk).
4. **Otomatisasi:** Server menangani start/stop tanpa intervensi user.

## 4. API Reference

Scheduler tidak memiliki endpoint khusus karena tertanam dalam object Job Blast/Warmer. Namun, output object Job memiliki field tambahan:

```json
{
  "id": "job-123",
  "scheduleConfig": {
    "type": "scheduled",
    "timeWindows": [{"start": "09:00", "end": "17:00"}],
    "daysOfWeek": [1, 2, 3, 4, 5]
  },
  "nextWindowStart": "2026-02-06T09:00:00Z",
  "nextWindowEnd": "2026-02-06T17:00:00Z",
  "status": "TIME_PAUSED"
}
```

## 5. WebSocket Events

| Event | Direction | Payload | Deskripsi |
| ----- | --------- | ------- | --------- |
| `schedule:update` | Server → Client | `{ jobId, status, nextWindowStart, countdown }` | Update timer countdown di UI |

## 6. Integrasi

- **Message Blast:** Scheduler mengontrol state `BlastJob`.
- **Account Warmer:** Scheduler mengontrol state `WarmerJob`.
- **Settings:** Default timezone diambil dari pengaturan global aplikasi.
