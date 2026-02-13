# PRD: Number Checker

**Version:** 2.0  
**Status:** Draft  
**Date:** 2026-01-25

---

## 1. Overview

Fitur **Number Checker** memverifikasi apakah nomor telepon terdaftar di WhatsApp sebelum mengirim pesan. Fokus: **Cepat, akurat, hemat kuota**.

---

## 2. Goals

- ✅ Cek ribuan nomor dalam waktu singkat
- ✅ Filter nomor tidak aktif sebelum blast
- ✅ Export hasil ke Excel
- ✅ Integrasi dengan Contacts database

---

## 3. Quick Setup (3-Step)

### Step 1: Input Nomor

```
┌─────────────────────────────────────────────────┐
│ 🔍 Number Checker - Step 1/3                    │
├─────────────────────────────────────────────────┤
│ Sumber Nomor:                                   │
│                                                 │
│ ○ 📁 Upload File (Excel/CSV)                    │
│ ○ 📇 Dari Database Kontak                       │
│ ● 📝 Input Manual                               │
│                                                 │
│ ┌───────────────────────────────────────────┐   │
│ │ 6281234567890                             │   │
│ │ 6289876543210                             │   │
│ │ 6281111222233                             │   │
│ │ (satu nomor per baris)                    │   │
│ └───────────────────────────────────────────┘   │
│                                                 │
│ Total: 3 nomor                                  │
│                                                 │
│                              [Selanjutnya →]    │
└─────────────────────────────────────────────────┘
```

### Step 2: Pilih Akun Checker

```
┌─────────────────────────────────────────────────┐
│ 🔍 Number Checker - Step 2/3                    │
├─────────────────────────────────────────────────┤
│ Pilih akun untuk pengecekan:                    │
│                                                 │
│ ○ 🔵 Toko A                                     │
│ ● 🟢 Toko B (Recommended - paling sehat)        │
│ ○ 🟣 Personal                                   │
│                                                 │
│ ⚠️ Gunakan akun dengan trust level tinggi       │
│    untuk menghindari pembatasan.                │
│                                                 │
│ [← Kembali]                    [Selanjutnya →]  │
└─────────────────────────────────────────────────┘
```

### Step 3: Konfigurasi & Mulai

```
┌─────────────────────────────────────────────────┐
│ 🔍 Number Checker - Step 3/3                    │
├─────────────────────────────────────────────────┤
│ Pengaturan:                                     │
│                                                 │
│ Preset: [🚶 Normal ▼]                           │
│                                                 │
│ ⚙️ Detail:                                      │
│ ├── Delay antar cek: 2-5 detik                  │
│ └── Jeda setiap 50 nomor: 30 detik              │
│                                                 │
│ ─────────────────────────────────────────────── │
│ 📊 Ringkasan:                                   │
│ • Nomor: 500                                    │
│ • Akun: Toko B                                  │
│ • Estimasi waktu: ~20 menit                     │
│                                                 │
│ [← Kembali]               [🔍 Mulai Cek]        │
└─────────────────────────────────────────────────┘
```

---

## 4. Result States

| Status     | Icon | Description            |
| ---------- | ---- | ---------------------- |
| `active`   | ✅   | Terdaftar di WhatsApp  |
| `inactive` | ❌   | Tidak terdaftar        |
| `error`    | ⚠️   | Gagal mengecek         |
| `unknown`  | ❓   | Tidak dapat ditentukan |

---

## 5. Progress Dashboard

```
┌─────────────────────────────────────────────────┐
│ 🔍 Checker: Batch 24-Jan                        │
├─────────────────────────────────────────────────┤
│ Status: 🟢 RUNNING                              │
│                                                 │
│ [████████████░░░░░░░░] 60% (300/500)            │
│                                                 │
│ ✅ Aktif: 245                                   │
│ ❌ Tidak Aktif: 48                              │
│ ⚠️ Error: 7                                     │
│ ⏳ Sisa: 200                                    │
│                                                 │
│ [⏸️ Pause]  [⏹️ Stop]                           │
└─────────────────────────────────────────────────┘
```

---

## 6. Result Table

```
┌─────────────────────────────────────────────────────────────────┐
│ 📊 Hasil Pengecekan                  Filter: [All ▼]  [🔍]     │
├─────────────────────────────────────────────────────────────────┤
│ No │ Nomor           │ Status        │ Checked At    │ Aksi    │
├────┼─────────────────┼───────────────┼───────────────┼─────────┤
│ 1  │ +6281234567890  │ ✅ Aktif      │ 10:30:15      │ [💾]    │
│ 2  │ +6289876543210  │ ✅ Aktif      │ 10:30:18      │ [💾]    │
│ 3  │ +6281111222233  │ ❌ Tidak Aktif │ 10:30:21      │ [🗑️]    │
│ 4  │ +6282222333344  │ ✅ Aktif      │ 10:30:24      │ [💾]    │
└────┴─────────────────┴───────────────┴───────────────┴─────────┘
                                      [Export Aktif 📥]
```

---

## 7. Post-Check Actions

### 7.1 Export to Excel

| Format        | Columns                   |
| ------------- | ------------------------- |
| All           | Nomor, Status, Checked At |
| Active Only   | Hanya nomor aktif         |
| Inactive Only | Hanya nomor tidak aktif   |

### 7.2 Save to Contacts

```
[💾 Simpan 245 nomor aktif ke Kontak]
```

### 7.3 Start Blast

```
[📤 Blast ke 245 nomor aktif]
```

→ Langsung buka wizard Blast dengan nomor hasil cek.

---

## 8. Delay Presets

| Preset    | Min Delay | Max Delay | Jeda/N | Extra Delay |
| --------- | --------- | --------- | ------ | ----------- |
| 🐢 Aman   | 5s        | 10s       | 20     | 60s         |
| 🚶 Normal | 2s        | 5s        | 50     | 30s         |
| 🏃 Cepat  | 1s        | 3s        | 100    | 15s         |

---

## 9. API Endpoints

| Method | Endpoint                         | Description    |
| ------ | -------------------------------- | -------------- |
| GET    | `/number-checker`                | List semua job |
| POST   | `/number-checker`                | Buat job baru  |
| GET    | `/number-checker/:jobId`         | Detail job     |
| POST   | `/number-checker/:jobId/start`   | Mulai          |
| POST   | `/number-checker/:jobId/pause`   | Pause          |
| POST   | `/number-checker/:jobId/resume`  | Resume         |
| POST   | `/number-checker/:jobId/stop`    | Stop           |
| DELETE | `/number-checker/:jobId`         | Hapus job      |
| GET    | `/number-checker/:jobId/results` | Get hasil      |
| GET    | `/number-checker/:jobId/export`  | Export Excel   |

---

## 10. WebSocket Events

| Event               | Direction       | Payload                                       |
| ------------------- | --------------- | --------------------------------------------- |
| `checker:progress`  | Server → Client | `{ jobId, checked, total, active, inactive }` |
| `checker:result`    | Server → Client | `{ number, status }`                          |
| `checker:completed` | Server → Client | `{ jobId, summary }`                          |

---

## 11. Success Metrics

- [ ] Cek 1000 nomor dalam < 30 menit
- [ ] Akurasi 99%+
- [ ] Zero false positives
- [ ] Export dalam < 5 detik

---

## 12. Integrasi dengan Fitur Lain

### 12.1 Dependency

| Fitur             | Status                         |
| ----------------- | ------------------------------ |
| Client Management | ⬅️ Akun untuk pengecekan       |
| Contacts          | ⬅️ Opsi "Dari Database Kontak" |

### 12.2 Provides to Other Features

| Consumer     | Data/Service                 |
| ------------ | ---------------------------- |
| **Blast**    | Hasil cek → langsung blast   |
| **Contacts** | Simpan nomor aktif ke kontak |

### 12.3 Integration Checklist

Saat membangun fitur ini, **WAJIB** integrasi:

- [ ] Step 1: Option "📇 Dari Database Kontak" → `GET /contacts`
- [ ] Step 2: Account picker → `GET /whatsapp/sessions`
- [ ] Post-check: "Simpan ke Kontak" → `POST /contacts/batch`
- [ ] Post-check: "Blast ke Aktif" → redirect ke Blast wizard dengan nomor

### 12.4 Flow ke Blast

```
Checker selesai
    ↓
[📤 Blast ke 245 nomor aktif]
    ↓
Blast Wizard Step 1 (pre-filled)
```

---

**Status:** Menunggu Review
