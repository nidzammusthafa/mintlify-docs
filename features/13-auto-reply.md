# Fitur: Auto-Reply (🤖)

## 1. Overview

**Auto-Reply** adalah fitur bot sederhana yang membalas pesan masuk secara otomatis berdasarkan kata kunci (keyword) tertentu. Fitur ini berguna untuk menjawab pertanyaan umum (FAQ) secara instan tanpa intervensi manusia, 24/7.

## 2. Komponen Detail

### 2.1 Keyword Triggers

**Deskripsi:**
Mekanisme pencocokan pesan masuk dengan aturan balasan.

**Jenis Match:**
- **Exact Match:** Pesan harus persis sama. (Contoh: "Menu")
- **Contains:** Pesan mengandung kata tersebut. (Contoh: "harga", akan merespon "berapa harganya?")
- **Start With:** Pesan diawali kata tersebut. (Contoh: "/info")

### 2.2 Response Actions

**Deskripsi:**
Apa yang dilakukan bot saat keyword cocok.
- **Reply Text:** Balas dengan teks biasa.
- **Reply Media:** Balas dengan gambar/brosur.
- **Reply Template:** Balas menggunakan Template Pesan yang sudah ada.
- **Trigger Flow:** Menjalankan alur percakapan AI Flow (lanjutan).

### 2.3 Active Hours & Account Binding

**Parameter:**
- `activeHours`: Rentang waktu bot aktif (misal: 17:00 - 08:00 untuk "Di Luar Jam Kerja").
- `accounts`: Bot ini aktif di akun mana saja? (Specific Account atau All Accounts).

## 3. User Flow

1.  User membuat Auto-Reply baru: "Info Rekening".
2.  Keyword: "rekening", "transfer", "bayar". Match Type: Contains.
3.  Response: "Silakan transfer ke BCA 123456 a.n Toko Saya".
4.  Simpan.
5.  Pelanggan chat: "Min, mau bayar transfer kemana?".
6.  Bot mendeteksi kata "bayar" & "transfer".
7.  Bot membalas otomatis info rekening dalam < 2 detik.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/auto-reply` | List aturan auto-reply |
| `POST` | `/auto-reply` | Buat aturan baru |
| `PUT` | `/auto-reply/:id` | Edit aturan |
| `DELETE` | `/auto-reply/:id` | Hapus aturan |
| `POST` | `/auto-reply/:id/toggle` | Aktifkan/Nonaktifkan aturan |

## 5. Integrasi

-   **Inbox:** Auto-reply memonitor pesan masuk di Inbox. Pesan yang dibalas bot akan ditandai icon "🤖" di chat list.
-   **Templates:** Respon bisa mengambil dari template agar konsisten.
-   **AI Flow Builder:** Auto-reply sederhana bisa di-upgrade menjadi trigger untuk Flow kompleks.
