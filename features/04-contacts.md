# Fitur: Contacts Management (📇)

## 1. Overview

Fitur **Contacts Management** adalah pusat data pelanggan di WhatsApp Tools. Fitur ini tidak hanya menyimpan nomor telepon, tetapi juga profil lengkap pelanggan termasuk alamat, email, bisnis, dan data custom lainnya. Fitur ini dirancang untuk menangani ribuan kontak dengan performa tinggi, mendukung import massal, dan integrasi erat dengan fitur Blast.

## 2. Komponen Detail

### 2.1 Contact Schema

**Deskripsi:**
Struktur data kontak mencakup field standar dan field kustom.

**Standard Fields:**
| Field | Tipe | Deskripsi |
|---|---|---|
| `name` | String | Nama lengkap kontak (Wajib) |
| `phoneNumber` | String | Nomor WhatsApp format internasional (628...) (Unik, Wajib) |
| `businessName` | String | Nama toko/bisnis |
| `businessCategory` | String | Kategori bisnis (misal: "Otomotif", "Kuliner") |
| `city` | String | Kota domisili |
| `address` | String | Alamat lengkap |
| `email` | String | Alamat email |
| `website` | String | URL website |
| `rating` | Float | Rating prioritas (1-5) |
| `hasReceivedMessage` | Boolean | Penanda apakah sudah pernah diblast (untuk fitur skip) |
| `lastContactedAt` | DateTime | Waktu terakhir pesan dikirim ke kontak ini |

**Custom Fields:**
User dapat menambahkan hingga 5 field tambahan sesuai kebutuhan bisnis.
Contoh: `nomor_pesanan`, `tanggal_lahir`, `jenis_member`.

### 2.2 Import Wizard

**Deskripsi:**
Proses import kontak massal dari file Excel/CSV dengan fitur mapping kolom cerdas.

**Flow:**
1. **Upload:** User upload file `.xlsx` atau `.csv`.
2. **Mapping:** Sistem mencoba mengenali kolom secara otomatis (misal: kolom "No HP" -> `phoneNumber`). User memverifikasi atau memperbaiki mapping.
3. **Preview & Validate:** Sistem menampilkan preview data dan menandai duplikat atau format nomor salah.
4. **Execution:** Data valid disimpan ke database.

**Auto-Detection Rules:**
- `phone`, `hp`, `nomor`, `wa` -> `phoneNumber`
- `name`, `nama`, `kontak` -> `name`
- `city`, `kota`, `lokasi` -> `city`

### 2.3 Search & Filter

**Deskripsi:**
Kemampuan mencari dan menyaring kontak untuk manajemen atau seleksi blast.

**Parameter Filter:**
- `city`: Filter berdasarkan kota.
- `businessCategory`: Filter berdasarkan kategori.
- `hasReceivedMessage`: Filter yang sudah/belum dikontak.
- `tags`: Filter berdasarkan label/tag.

## 3. User Flow

### Skenario: Import Kontak Baru
1. Masuk ke halaman **Contacts**.
2. Klik tombol **Import**.
3. Upload file Excel berisi 500 data pelanggan.
4. Pastikan kolom "Nama" dan "No WA" terpetakan dengan benar.
5. Klik **Import**.
6. 495 kontak berhasil masuk, 5 gagal karena nomor tidak valid.

### Skenario: Update Status Blast
(Otomatis)
1. User menjalankan Blast ke 100 kontak.
2. Blast sukses terkirim ke "Budi".
3. Sistem otomatis update kontak "Budi": `hasReceivedMessage = true`, `lastContactedAt = Now`.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/contacts` | List kontak dengan pagination & filter |
| `POST` | `/contacts` | Tambah satu kontak manual |
| `POST` | `/contacts/import` | Upload & import file kontak |
| `POST` | `/contacts/export` | Download kontak ke Excel |
| `GET` | `/contacts/:id` | Detail kontak |
| `PUT` | `/contacts/:id` | Update data kontak |
| `DELETE` | `/contacts/:id` | Hapus kontak |
| `DELETE` | `/contacts/batch` | Hapus banyak kontak sekaligus |

## 5. Integrasi

- **Message Blast:**
  - Kontak menjadi target penerima blast.
  - Variabel blast (`{name}`, `{city}`) diambil dari field kontak.
- **Number Checker:**
  - Hasil validasi number checker bisa langsung disimpan sebagai kontak baru.
- **Maps Scraper:**
  - Data hasil scraping (Google Maps) bisa diexport ke Contacts.
- **Inbox:**
  - Saat pesan masuk dari nomor tak dikenal, sistem mengecek database Contacts untuk menampilkan nama (jika ada).
