# Fitur: Settings (⚙️)

## 1. Overview

Fitur **Settings** adalah pusat konfigurasi aplikasi yang mengontrol perilaku global sistem, preferensi pengguna, dan parameter keamanan. Pengaturan dikelompokkan berdasarkan kategori untuk memudahkan navigasi.

## 2. Komponen Detail

### 2.1 Kategori Pengaturan

#### A. Notifikasi
Mengatur alert desktop dan suara.
- `desktop_notification`: On/Off
- `sound_alert`: On/Off
- `email_report`: Daily summary ke email owner

#### B. Blast Defaults
Konfigurasi bawaan untuk fitur Blast agar user tidak perlu setting ulang setiap kali.
- `default_delay_preset`: (Safe/Normal/Fast)
- `auto_skip_existing`: On/Off (Skip kontak yang sudah ada di address book)
- `auto_save_contact`: On/Off (Otomatis simpan nomor sukses ke kontak)

#### C. Warmer Defaults
Konfigurasi bawaan untuk fitur Warmer.
- `default_warmer_preset`: Preset yang dipilih otomatis
- `default_language`: Bahasa percakapan warmer (ID/EN)

#### D. Spam Filter & Whitelist
- `blocked_keywords`: Daftar kata yang memicu auto-archive/delete pesan masuk.
- `whitelist_numbers`: Daftar nomor prioritas yang tidak boleh kena filter/limit.

#### E. Tampilan (Display)
- `theme`: Light/Dark Mode
- `date_format`: DD/MM/YYYY atau MM/DD/YYYY
- `sidebar_mode`: Expanded/Collapsed

#### F. Data Management
- `backup`: Download database JSON/SQL dump.
- `reset`: Factory reset, hapus semua log, hapus semua kontak.

## 3. User Flow

### Skenario: Mengubah Default Delay
1.  User masuk ke menu **Settings**.
2.  Pilih tab **Blast Defaults**.
3.  Ubah **Default Delay Preset** dari "Normal" ke "Safe" (karena akun baru sering kena ban).
4.  Sistem auto-save.
5.  Saat user membuat Job Blast baru, preset "Safe" otomatis terpilih.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/settings` | Ambil semua konfigurasi |
| `PUT` | `/settings` | Update konfigurasi (partial update) |
| `POST` | `/settings/reset` | Reset ke default factory |
| `GET` | `/settings/export` | Backup konfigurasi ke file |
| `POST` | `/settings/import` | Restore konfigurasi dari file |

## 5. Integrasi

-   **Semua Fitur:** Hampir semua fitur membaca konfigurasi dari Settings.
    -   **Blast:** Membaca `blast_defaults`.
    -   **Warmer:** Membaca `warmer_defaults`.
    -   **Inbox:** Membaca `spam_filter` dan `display`.
    -   **System:** Membaca `notification` dan `theme`.
