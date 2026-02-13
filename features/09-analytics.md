# Fitur: Analytics (📊)

## 1. Overview

Fitur **Analytics** menyediakan dashboard visual untuk memantau performa pengiriman pesan, kesehatan akun, dan statistik interaksi. Data disajikan dalam berbagai periode waktu (Harian, Mingguan, Bulanan) untuk membantu pengguna mengambil keputusan berbasis data.

## 2. Komponen Detail

### 2.1 Metrics Utama

**Deskripsi:**
Indikator kinerja utama (KPI) yang dilacak sistem.

| Metric | Deskripsi | Sumber Data |
|---|---|---|
| **Sent** | Total pesan yang berhasil dikirim dari server | Blast Service, Inbox |
| **Delivered** | Pesan yang sampai ke HP penerima (Centang dua) | WebSocket Event (ACK) |
| **Read** | Pesan yang dibaca penerima (Centang biru) | WebSocket Event (ACK) |
| **Failed** | Pesan gagal kirim (koneksi putus/nomor salah) | WebSocket Event (ACK) |
| **Response Rate** | Persentase pesan yang dibalas oleh penerima | Inbox Service |

### 2.2 Per-Account Analytics

**Deskripsi:**
Statistik terpisah untuk setiap akun WhatsApp yang terhubung. Memungkinkan user membandingkan kinerja antar akun (misal: CS 1 vs CS 2).

**Data Point:**
- Trust Score (Newcomer/Verified)
- Pesan terkirim hari ini vs Limit Harian
- Grafik aktivitas per jam

### 2.3 Visualisasi Chart

**Jenis Grafik:**
- **Line Chart:** Tren pengiriman pesan selama 7/30 hari terakhir.
- **Pie Chart:** Distribusi status pesan (Sent vs Failed vs Read).
- **Bar Chart:** Perbandingan performa antar akun.

## 3. User Flow

1.  **Dashboard:** User membuka menu Analytics.
2.  **Filter:** User memilih rentang waktu "Last 30 Days" dan Akun "Semua".
3.  **Analisis:**
    -   User melihat grafik pengiriman menurun di akhir pekan.
    -   User melihat "Toko A" memiliki failure rate tinggi (perlu dicek koneksinya).
4.  **Export:** User mendownload laporan dalam format Excel untuk presentasi mingguan.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/analytics/summary` | Ringkasan metrik utama (cards) |
| `GET` | `/analytics/daily` | Data tren harian |
| `GET` | `/analytics/weekly` | Data tren mingguan |
| `GET` | `/analytics/monthly` | Data tren bulanan |
| `GET` | `/analytics/account/:id` | Statistik spesifik per akun |
| `GET` | `/analytics/export` | Download laporan Excel |

## 5. Integrasi

-   **Message Blast:** Sumber utama data volume pesan keluar.
-   **Inbox:** Sumber data pesan masuk dan balasan (response rate).
-   **Client Management:** Menyediakan daftar akun untuk filter.
-   **Templates:** Data efektivitas template ditampilkan di widget "Top Templates".
