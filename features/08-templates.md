# Fitur: Templates (📝)

## 1. Overview

Fitur **Templates** memungkinkan pengguna menyimpan dan mengelola format pesan yang sering digunakan agar dapat diakses kembali dengan cepat saat melakukan Blast atau membalas pesan di Inbox. Fitur ini dilengkapi dengan sistem **Scoring/Poin** untuk mengukur efektivitas setiap template berdasarkan respon penerima.

## 2. Komponen Detail

### 2.1 Template Structure

**Deskripsi:**
Struktur data template yang fleksibel mendukung teks dan media.

**Parameter:**
| Parameter | Tipe | Deskripsi |
|---|---|---|
| `name` | String | Nama template (untuk identifikasi) |
| `content` | String | Isi pesan (bisa mengandung variabel `{nama}`) |
| `category` | String | Kategori (Promo, Greeting, Follow-up) |
| `mediaUrl` | String | URL gambar/video (opsional) |
| `variables` | Array | Daftar variabel yang terdeteksi otomatis |

### 2.2 Template Scoring System

**Deskripsi:**
Sistem poin untuk menilai performa template.
**Rumus:** `Poin = (Read x 2) + (Reply x 3)`

**Indikator:**
- 🔴 **Low:** < 50 poin
- 🟡 **Medium:** 50-150 poin
- 🟢 **Good:** 150-300 poin
- 🌟 **Excellent:** > 300 poin

### 2.3 Variable Detection

**Deskripsi:**
Sistem otomatis mendeteksi pola `{text}` dalam konten template sebagai variabel. Saat template digunakan di Blast, variabel ini akan dicocokkan dengan kolom Excel. Saat di Inbox, user diminta mengisi nilai variabel secara manual.

## 3. User Flow

1.  **Create:** User membuat template "Promo Gajian" dengan konten "Halo {nama}, diskon 50% hari ini!".
2.  **Usage (Blast):** User memilih template ini saat setup Blast. Sistem otomatis memetakan `{nama}` ke kolom nama di Excel.
3.  **Tracking:**
    -   100 pesan terkirim.
    -   80 dibaca (+160 poin).
    -   10 dibalas (+30 poin).
    -   Total Skor: 190 (Good 🟢).
4.  **Usage (Inbox):** Di menu chat, user mengetik "/" dan memilih template "Promo Gajian" untuk membalas chat pelanggan secara instan.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/templates` | List semua template |
| `POST` | `/templates` | Buat template baru |
| `GET` | `/templates/:id` | Detail template |
| `PUT` | `/templates/:id` | Update template |
| `DELETE` | `/templates/:id` | Hapus template |
| `GET` | `/templates/top` | Get template dengan performa terbaik |
| `POST` | `/templates/:id/usage` | Catat penggunaan (untuk analitik) |

## 5. Integrasi

-   **Message Blast:** Template picker muncul di step penyusunan pesan.
-   **Inbox:** Quick access template (`/` command) di kolom input chat.
-   **Analytics:** Laporan performa template (Top 5 Templates) di dashboard analitik.
