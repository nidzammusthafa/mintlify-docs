# Fitur: Inbox (📥)

## 1. Overview

Fitur **Inbox** (Kotak Masuk) adalah pusat komunikasi yang menyatukan pesan dari semua akun WhatsApp yang terhubung dalam satu antarmuka terpadu (Unified Inbox). Fitur ini memudahkan tim CS/Admin untuk membalas pesan pelanggan tanpa perlu berpindah-pindah device atau tab browser.

## 2. Komponen Detail

### 2.1 Unified Inbox & Multi-Account View

**Deskripsi:**
Menampilkan daftar percakapan dari seluruh akun WA yang terhubung. User bisa memfilter tampilan untuk melihat "Semua Akun" atau spesifik satu akun saja.

**Fitur:**
- **Account Badge:** Ikon kecil di setiap chat list yang menunjukkan pesan tersebut masuk ke akun mana (misal: Toko A, CS 1).
- **Unread Counter:** Indikator jumlah pesan belum dibaca per akun.
- **Sorting:** Urutkan berdasarkan pesan terbaru (default) atau pesan belum dibaca.

### 2.2 Chat Interface

**Deskripsi:**
Area percakapan standar dengan kemampuan rich media.

**Fitur:**
- **Send Message:** Teks, Gambar, Dokumen.
- **Quick Reply:** Akses cepat ke Template Pesan menggunakan shortcut `/`.
- **Sender Info:** Panel samping yang menampilkan detail kontak (Nama, Lokasi, Label) yang diambil dari database Contacts.

### 2.3 Spam Filter

**Deskripsi:**
Sistem otomatis yang menyaring pesan masuk berdasarkan kata kunci yang ditentukan di Settings. Pesan spam tidak akan memicu notifikasi dan langsung masuk folder Spam/Archived.

### 2.4 Label Assignment

**Deskripsi:**
User dapat memberikan label pada percakapan untuk pengorganisasian (misal: "Pending Payment", "Komplain", "VIP"). Label ini sinkron dengan fitur Labels Management.

## 3. User Flow

1.  **Notifikasi:** Admin menerima notifikasi desktop "Pesan baru dari Budi ke Toko A".
2.  **Open Inbox:** Admin membuka menu Inbox.
3.  **Read:** Admin membaca pesan Budi.
4.  **Context:** Admin melihat panel samping: Budi adalah pelanggan VIP dari Jakarta.
5.  **Reply:** Admin mengetik `/promo` dan memilih template "Promo Diskon".
6.  **Label:** Admin menandai chat dengan label "Menunggu Transfer".

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/inbox/chats` | List percakapan (paginated) |
| `GET` | `/inbox/chats/:id/messages` | List pesan dalam satu chat |
| `POST` | `/inbox/chats/:id/send` | Kirim balasan |
| `POST` | `/inbox/chats/:id/read` | Tandai sudah dibaca |
| `DELETE` | `/inbox/chats/:id` | Hapus percakapan |
| `POST` | `/inbox/chats/:id/labels` | Tambah/Hapus label chat |

## 5. Integrasi

-   **Client Management:** Inbox bergantung penuh pada koneksi aktif akun WA.
-   **Contacts:** Menampilkan data profil pengirim.
-   **Templates:** Quick reply shortcut.
-   **Labels:** Manajemen tag percakapan.
-   **Analytics:** Mencatat statistik pesan masuk dan response time.
