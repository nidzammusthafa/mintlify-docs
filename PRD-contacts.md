# PRD: Contacts Management

**Version:** 2.0  
**Status:** Draft  
**Date:** 2026-01-25

---

## 1. Overview

Fitur **Contacts** memungkinkan pengguna mengelola database kontak untuk blast, filtering, dan tracking. Fokus: **CRUD mudah, import massal dari Excel, dan custom fields**.

---

## 2. Goals

- ✅ CRUD lengkap untuk kontak
- ✅ Import massal dari Excel/CSV
- ✅ Auto-map kolom Excel ke field database
- ✅ Custom fields (max 5 untuk keamanan)
- ✅ Integrasi dengan Blast (skip/mark sent)

---

## 3. Contact Schema

### 3.1 Fixed Fields (Built-in)

| Field                | Icon | Type    | Required | Description             |
| -------------------- | ---- | ------- | -------- | ----------------------- |
| `id`                 | -    | String  | Auto     | UUID                    |
| `name`               | 👤   | String  | ✅       | Nama kontak             |
| `phoneNumber`        | 📱   | String  | ✅       | Nomor WhatsApp (unique) |
| `address`            | 📍   | String  | -        | Alamat lengkap          |
| `city`               | 🏙️   | String  | -        | Kota                    |
| `state`              | 🗺️   | String  | -        | Provinsi                |
| `country`            | 🌍   | String  | -        | Negara                  |
| `postalCode`         | 📮   | String  | -        | Kode pos                |
| `email`              | ✉️   | String  | -        | Email                   |
| `website`            | 🌐   | String  | -        | Website                 |
| `businessName`       | 🏢   | String  | -        | Nama bisnis             |
| `businessCategory`   | 🏷️   | String  | -        | Kategori bisnis         |
| `rating`             | ⭐   | Float   | -        | Rating (1-5)            |
| `reviews`            | 📊   | Int     | -        | Jumlah review           |
| `isBusiness`         | 🏪   | Boolean | -        | Apakah akun bisnis      |
| `hasReceivedMessage` | ✅   | Boolean | -        | Sudah pernah dikirimi   |
| `status`             | 🔘   | Enum    | -        | ACTIVE / INACTIVE       |

### 3.2 Custom Fields (Max 5)

User dapat menambah field tambahan:

```typescript
customFields: {
  "custom1": "Nilai 1",
  "custom2": "Nilai 2",
  // Max 5 fields
}
```

**Keamanan:** Sistem membatasi maksimal 5 custom field untuk mencegah database bloat.

---

## 4. CRUD Operations

### 4.1 Create Contact

```
┌─────────────────────────────────────────────────┐
│ ➕ Tambah Kontak                          [✕]   │
├─────────────────────────────────────────────────┤
│ 👤 Nama*         : [                    ]       │
│ 📱 Nomor HP*     : [+628                ]       │
│ 🏢 Nama Bisnis   : [                    ]       │
│ 🏷️ Kategori      : [Pilih ▼            ]       │
│ 📍 Alamat        : [                    ]       │
│ 🏙️ Kota          : [                    ]       │
│ ✉️ Email         : [                    ]       │
│ 🌐 Website       : [                    ]       │
│                                                 │
│ ➕ Tambah Custom Field (3/5 tersisa)            │
│                                                 │
│ [Batal]                           [💾 Simpan]   │
└─────────────────────────────────────────────────┘
```

### 4.2 Contact List

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 📇 Kontak                    [🔍 Cari...]        [+ Tambah] [📥 Import]    │
├─────────────────────────────────────────────────────────────────────────────┤
│ Filter: [Semua ▼] [Kota ▼] [Kategori ▼]                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ █ │ Nama          │ No. HP        │ Bisnis         │ Kota    │ Sent │ Aksi │
├───┼───────────────┼───────────────┼────────────────┼─────────┼──────┼──────┤
│ ☐ │ Budi Santoso  │ +62812345...  │ Toko Elektr... │ Jakarta │ ✅   │ ⋮    │
│ ☐ │ Ani Widya     │ +62898765...  │ Salon Cantik   │ Bandung │ ❌   │ ⋮    │
│ ☐ │ Doni Rahman   │ +62811122...  │ -              │ Surabaya│ ✅   │ ⋮    │
└───┴───────────────┴───────────────┴────────────────┴─────────┴──────┴──────┘
     [◀ Prev] 1 2 3 ... 10 [Next ▶]         Showing 1-20 of 450 contacts
```

### 4.3 Bulk Actions

| Action            | Description                         |
| ----------------- | ----------------------------------- |
| Delete Selected   | Hapus kontak terpilih               |
| Export Selected   | Export ke Excel                     |
| Reset Sent Status | Reset `hasReceivedMessage` ke false |

---

## 5. Mass Import

### 5.1 Import Wizard

**Step 1: Upload File**

```
┌─────────────────────────────────────────────────┐
│ 📥 Import Kontak - Step 1/3                     │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │     📁 Drag & drop file Excel/CSV         │  │
│  │             atau klik untuk upload        │  │
│  └───────────────────────────────────────────┘  │
│                                                 │
│ Format didukung: .xlsx, .xls, .csv              │
│ Maksimal: 10,000 baris per file                 │
│                                                 │
│ [📥 Download Template]                          │
│                                                 │
│                              [Selanjutnya →]    │
└─────────────────────────────────────────────────┘
```

**Step 2: Map Kolom**

```
┌─────────────────────────────────────────────────┐
│ 📥 Import Kontak - Step 2/3                     │
├─────────────────────────────────────────────────┤
│ Mapping Kolom Excel → Database:                 │
│                                                 │
│ Kolom Excel          →  Field Database          │
│ ─────────────────────────────────────────────── │
│ [nama_pelanggan ▼]   →  👤 Nama                 │
│ [no_hp ▼]            →  📱 Nomor HP             │
│ [nama_toko ▼]        →  🏢 Nama Bisnis          │
│ [alamat_lengkap ▼]   →  📍 Alamat               │
│ [kota ▼]             →  🏙️ Kota                 │
│ [email ▼]            →  ✉️ Email                │
│ [- Skip - ▼]         →  (Tidak diimport)        │
│                                                 │
│ ➕ Map ke Custom Field:                         │
│ [jenis_usaha ▼]      →  📝 custom_1             │
│ [jumlah_karyawan ▼]  →  📝 custom_2             │
│                                                 │
│ [← Kembali]                    [Selanjutnya →]  │
└─────────────────────────────────────────────────┘
```

**Step 3: Preview & Confirm**

```
┌─────────────────────────────────────────────────┐
│ 📥 Import Kontak - Step 3/3                     │
├─────────────────────────────────────────────────┤
│ Preview (5 data pertama):                       │
│ ┌───────────────────────────────────────────┐   │
│ │ Nama       │ No. HP      │ Bisnis        │   │
│ │ Budi       │ 62812345... │ Toko Elektr...│   │
│ │ Ani        │ 62898765... │ Salon Cantik  │   │
│ └───────────────────────────────────────────┘   │
│                                                 │
│ ✅ 450 kontak valid                             │
│ ⚠️ 12 kontak duplikat (akan di-skip)            │
│ ❌ 3 kontak invalid (nomor tidak valid)         │
│                                                 │
│ [← Kembali]            [🚀 Import 450 Kontak]   │
└─────────────────────────────────────────────────┘
```

### 5.2 Column Auto-Detection

Sistem otomatis mendeteksi kolom berdasarkan nama:

| Nama Kolom Excel                | Auto-Map ke    |
| ------------------------------- | -------------- |
| `name`, `nama`, `nama_kontak`   | 👤 Nama        |
| `phone`, `hp`, `nomor`, `no_hp` | 📱 Nomor HP    |
| `city`, `kota`                  | 🏙️ Kota        |
| `address`, `alamat`             | 📍 Alamat      |
| `email`                         | ✉️ Email       |
| `business`, `usaha`, `toko`     | 🏢 Nama Bisnis |

---

## 6. API Endpoints

| Method | Endpoint                        | Description            |
| ------ | ------------------------------- | ---------------------- |
| GET    | `/contacts`                     | List semua (paginated) |
| GET    | `/contacts/:id`                 | Detail kontak          |
| POST   | `/contacts`                     | Tambah kontak          |
| PUT    | `/contacts/:id`                 | Update kontak          |
| DELETE | `/contacts/:id`                 | Hapus kontak           |
| POST   | `/contacts/import`              | Import dari file       |
| POST   | `/contacts/export`              | Export ke Excel        |
| DELETE | `/contacts/batch`               | Hapus banyak           |
| GET    | `/contacts/lookup/:phoneNumber` | Cari by nomor          |

---

## 7. Success Metrics

- [ ] Import 10,000 kontak dalam < 30 detik
- [ ] CRUD response < 200ms
- [ ] Zero data loss saat import
- [ ] Custom field max 5 enforced

---

## 8. Integrasi dengan Fitur Lain

### 8.1 Dependency

| Fitur    | Status                                 |
| -------- | -------------------------------------- |
| Settings | ⬅️ `customFields` config dari Settings |

### 8.2 Provides to Other Features

| Consumer           | Data/Service                                        |
| ------------------ | --------------------------------------------------- |
| **Blast**          | Recipients list, skip based on `hasReceivedMessage` |
| **Inbox**          | Sender profile panel, lookup by phoneNumber         |
| **Number Checker** | Save results ke contacts                            |
| **Analytics**      | Contact-based reporting                             |

### 8.3 Integration Checklist

Saat membangun fitur lain, **WAJIB** integrasi:

- [ ] Blast: Lookup contact by phoneNumber untuk variabel
- [ ] Blast: Update `hasReceivedMessage = true` setelah kirim
- [ ] Inbox: `GET /contacts/lookup/:phoneNumber` untuk sender panel
- [ ] Inbox: Button "Tambah ke Kontak" jika tidak ditemukan
- [ ] Number Checker: "Simpan ke Kontak" setelah cek selesai

---

**Status:** Menunggu Review
