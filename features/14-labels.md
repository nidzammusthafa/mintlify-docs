# Fitur: Labels (🏷️)

## 1. Overview

Fitur **Labels** menyediakan sistem pengorganisasian visual untuk chat dan kontak. Mirip dengan fitur label di WhatsApp Business, fitur ini memungkinkan user menandai percakapan dengan warna dan nama tertentu untuk memudahkan filtering dan prioritas.

## 2. Komponen Detail

### 2.1 Label Management (CRUD)

**Deskripsi:**
User dapat membuat label custom sesuai kebutuhan alur kerja bisnis.

**Atribut Label:**
- `name`: Nama label (contoh: "Tagihan", "Prospek Panas").
- `color`: Kode warna hex (untuk visualisasi badge).
- `icon`: Ikon opsional (emoji/iconset).

### 2.2 Label Assignment

**Deskripsi:**
Proses menempelkan label ke entitas.
- **Chat Label:** Menandai satu thread percakapan.
- **Contact Label:** Menandai profil kontak (permanen).

Satu entitas bisa memiliki **banyak label** (Many-to-Many relationship).

### 2.3 Label Filtering

**Deskripsi:**
Kemampuan menyaring daftar Inbox atau Kontak berdasarkan label.
Contoh: "Tampilkan hanya chat dengan label 'Komplain'".

## 3. User Flow

1.  **Setup:** Admin membuat label: "VIP" (Warna Emas), "Pending" (Warna Merah), "Done" (Warna Hijau).
2.  **Assign:** Saat chatting dengan pelanggan yang baru transfer, Admin klik "Add Label" -> Pilih "VIP".
3.  **View:** Di daftar chat, muncul badge kecil berwarna emas bertuliskan "VIP" di samping nama pelanggan.
4.  **Filter:** Di akhir hari, Admin memfilter Inbox dengan label "Pending" untuk memproses pesanan yang belum selesai.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/labels` | List semua label |
| `POST` | `/labels` | Buat label baru |
| `PUT` | `/labels/:id` | Edit label (ganti warna/nama) |
| `DELETE` | `/labels/:id` | Hapus label |
| `POST` | `/labels/reorder` | Simpan urutan prioritas label |

## 5. Integrasi

-   **Inbox:** Tampilan utama label ada di daftar chat dan detail chat.
-   **Contacts:** Label juga muncul di menu manajemen kontak.
-   **Settings:** Manajemen label (CRUD) dilakukan di menu Settings.
