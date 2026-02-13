# PLAN: Dokumentasi Lengkap Fitur WhatsApp Tools

**Plan ID:** `feature-documentation`  
**Created:** 2026-02-05  
**Status:** Completed
**Estimated Duration:** 20-30 jam

---

## 📋 Overview

Membuat dokumentasi lengkap untuk setiap fitur di project WhatsApp Tools, mencakup:

- Deskripsi fungsi dan tujuan
- Setiap parameter dan konfigurasi (termasuk nilai minimum/maksimum)
- User flow dan UI mockups
- API endpoints dan WebSocket events
- Integrasi dengan fitur lain
- Contoh penggunaan

---

## 🎯 Scope Fitur yang Akan Didokumentasikan

### 1. Message Blast (📤)

**Lokasi:** `new-server/src/core/blast/`, `new-client/src/app/(dashboard)/blast/`  
**PRD Existing:** `docs/PRD-message-blast.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur                 | Deskripsi                   | Parameter Detail                                    |
| ------------------------- | --------------------------- | --------------------------------------------------- |
| **Delay Presets**         | Jeda antar pengiriman pesan | `minDelay`, `maxDelay`, `jedaSetiapN`, `extraDelay` |
| **Delay Aman (🐢)**       | Untuk akun baru             | 15-30s, jeda setiap 5, extra 60s                    |
| **Delay Normal (🚶)**     | Default/Recommended         | 8-15s, jeda setiap 10, extra 30s                    |
| **Delay Cepat (🏃)**      | Untuk akun trusted          | 3-8s, jeda setiap 20, extra 15s                     |
| **Delay Custom (⚙️)**     | Konfigurasi manual          | User-defined semua parameter                        |
| **Multi-Sender Rotation** | Round-robin akun pengirim   | Algoritma skip exhausted accounts                   |
| **Message Blocks**        | Blok pesan berurutan        | Text, Media, Randomized                             |
| **Variable Substitution** | Placeholder personalisasi   | Format `{column_name}`                              |
| **Message Variations**    | Anti-spam randomization     | Array variants per blok                             |
| **Job Progress**          | Real-time tracking          | sent, failed, pending, percentage                   |
| **Job Control**           | Pause/Resume/Stop           | Status lifecycle, recovery                          |
| **Skip Contacts**         | Skip Address Book           | Filter `hasReceivedMessage`                         |
| **Warmer Integration**    | Jeda dengan warmer          | alternating execution flow                          |
| **Scheduling**            | Time windows                | Multiple windows per day, days of week              |

---

### 2. Account Warmer (🔥)

**Lokasi:** `new-server/src/core/warmer/`, `new-client/src/app/(dashboard)/warmer/`  
**PRD Existing:** `docs/PRD-warmer.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur                 | Deskripsi                 | Parameter Detail                           |
| ------------------------- | ------------------------- | ------------------------------------------ |
| **Pair Generation**       | Auto-pair semua kombinasi | n akun → n\*(n-1) pasangan                 |
| **Delay Presets Warmer**  | Jeda antar pesan warmer   | `minDelay`, `maxDelay`, `msgPerPair`       |
| **Warmer Lambat (🐢)**    | Akun sangat baru          | 30-60s delay, 3-5 msg/pair                 |
| **Warmer Normal (🚶)**    | Default                   | 10-30s delay, 5-10 msg/pair                |
| **Warmer Cepat (🏃)**     | Akun verified             | 5-15s delay, 10-15 msg/pair                |
| **Warmer Custom**         | Preset user               | All params configurable                    |
| **Message Templates**     | Pesan percakapan natural  | ID/EN language support                     |
| **Round-Robin Execution** | Urutan pair               | Sequential per-pair completion             |
| **Blast Coordination**    | Alternating mode          | `blastPauseAfterNRecipients`               |
| **Progress Tracking**     | Real-time log             | currentPair, sent/total                    |
| **Interval Config**       | Jeda setelah N blast      | `intervalMs`, `minMessages`, `maxMessages` |

---

### 3. Scheduler System (⏱️)

**Lokasi:** `new-server/src/core/scheduler/`  
**PLAN Existing:** `docs/PLAN-blast-scheduler.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur             | Deskripsi                  | Parameter Detail                         |
| --------------------- | -------------------------- | ---------------------------------------- |
| **Time Windows**      | Rentang waktu aktif        | `start` (HH:mm), `end` (HH:mm)           |
| **Multiple Windows**  | Beberapa slot per hari     | Array of TimeWindow                      |
| **Days of Week**      | Hari aktif                 | 0-6 (Sun-Sat)                            |
| **Timezone**          | Zona waktu user            | Default: Asia/Jakarta                    |
| **Start Date**        | Tanggal mulai jadwal       | ISO date string                          |
| **End Date**          | Tanggal selesai (optional) | ISO date string                          |
| **Auto-Start**        | Mulai otomatis saat window | Cron check setiap menit                  |
| **Auto-Pause**        | Pause saat keluar window   | `onWindowExit` behavior                  |
| **Auto-Resume**       | Resume saat masuk window   | `TIME_PAUSED` → `RUNNING`                |
| **Countdown Display** | Waktu tersisa              | untilStart, untilEnd                     |
| **Schedule Types**    | Tipe eksekusi              | IMMEDIATE, SCHEDULED, TIME_PAUSED        |
| **Pause Reasons**     | Alasan pause               | USER, OUTSIDE_WINDOW, DAILY_LIMIT, ERROR |

---

### 4. Contacts Management (📇)

**Lokasi:** `new-server/src/core/contacts/`, `new-client/src/app/(dashboard)/contacts/`  
**PRD Existing:** `docs/PRD-contacts.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur              | Deskripsi               | Parameter Detail                              |
| ---------------------- | ----------------------- | --------------------------------------------- |
| **Contact Schema**     | Field standar           | name, phoneNumber, city, address, email, etc. |
| **Custom Fields**      | Field tambahan user     | Max 5 custom fields                           |
| **Import Excel/CSV**   | Upload bulk             | Column mapping, skip duplicates               |
| **Export**             | Download data           | Format Excel                                  |
| **hasReceivedMessage** | Status sudah dikontak   | Auto-update setelah blast                     |
| **lastContactedAt**    | Waktu terakhir dikontak | Timestamp auto-update                         |
| **businessName**       | Nama bisnis             | String                                        |
| **businessCategory**   | Kategori bisnis         | String                                        |
| **rating**             | Rating kontak           | 1-5 stars                                     |
| **Labels**             | Label/tag kontak        | Many-to-many relation                         |
| **Search & Filter**    | Pencarian               | By name, phone, city, etc.                    |
| **Pagination**         | Load bertahap           | Infinite scroll / page                        |

---

### 5. Number Checker (🔍)

**Lokasi:** `new-server/src/core/number-checker/`, `new-client/src/app/(dashboard)/checker/`  
**PRD Existing:** `docs/PRD-number-checker.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur             | Deskripsi            | Parameter Detail            |
| --------------------- | -------------------- | --------------------------- |
| **Bulk Check**        | Cek banyak nomor     | Upload file atau paste      |
| **Check Result**      | Hasil cek            | isRegistered, hasProfilePic |
| **Direct Blast**      | Langsung blast hasil | Integration with blast      |
| **Export Results**    | Download hasil       | Valid/Invalid separate      |
| **Delay Antar Check** | Jeda cek             | Configurable delay          |
| **Progress Tracking** | Real-time progress   | checked/total               |

---

### 6. Maps Scraper (🗺️)

**Lokasi:** `new-server/src/core/maps-scraper/`, `new-client/src/app/(dashboard)/scrape/`  
**PLAN Existing:** `docs/PLAN-maps-scraper.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur             | Deskripsi          | Parameter Detail                                              |
| --------------------- | ------------------ | ------------------------------------------------------------- |
| **Conventional Mode** | Scrape by keyword  | Query string, location                                        |
| **Coordinate Mode**   | Scrape by lat/lng  | Lat, lng, radius                                              |
| **Data Extracted**    | Hasil scrape       | name, phone, address, lat, lng, rating, reviewsCount, website |
| **Infinite Scroll**   | Load semua hasil   | Automatic scrolling                                           |
| **Job Control**       | Start/Stop/Pause   | Job management                                                |
| **Export Results**    | Download data      | Excel format                                                  |
| **Save to Contacts**  | Simpan ke database | Integration                                                   |

---

### 7. AI Flow Builder (🤖)

**Lokasi:** `new-server/src/core/ai-agent/`, `new-client/src/app/(dashboard)/ai/`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur                 | Deskripsi             | Parameter Detail          |
| ------------------------- | --------------------- | ------------------------- |
| **Flow Editor**           | Visual node editor    | Node types, connections   |
| **Trigger Nodes**         | Start points          | Message received, keyword |
| **AI Response Nodes**     | Gemini integration    | Model, prompt, context    |
| **Fetch Nodes**           | API calls             | URL, method, headers      |
| **Function Nodes**        | Custom logic          | JavaScript code           |
| **Condition Nodes**       | Branching             | If/else conditions        |
| **Action Nodes**          | Send message          | Text, media               |
| **Message Deduplication** | Anti-spam             | messageId tracking        |
| **Conversation Lock**     | Prevent duplicate     | Redis-based locking       |
| **Flow Resume**           | Continue conversation | Context preservation      |

---

### 8. Templates (📝)

**Lokasi:** `new-server/src/core/templates/`, `new-client/src/app/(dashboard)/templates/`  
**PRD Existing:** `docs/PRD-templates.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur              | Deskripsi             | Parameter Detail               |
| ---------------------- | --------------------- | ------------------------------ |
| **Template Creation**  | Buat template         | name, messageBlocks, variables |
| **Template Picker**    | Pilih template        | Search, recent                 |
| **Usage Tracking**     | Statistik penggunaan  | usageCount, lastUsedAt         |
| **Variable Detection** | Auto-detect variables | Parse `{varName}`              |
| **Template Scoring**   | Efektivitas           | Points system (+2 on read)     |
| **Categories**         | Kategori template     | Custom categories              |

---

### 9. Analytics (📊)

**Lokasi:** `new-server/src/core/analytics/`, `new-client/src/app/(dashboard)/analytics/`  
**PRD Existing:** `docs/PRD-analytics.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur               | Deskripsi          | Parameter Detail     |
| ----------------------- | ------------------ | -------------------- |
| **Sent Count**          | Pesan terkirim     | Per account, per day |
| **Delivered Count**     | Pesan tersampaikan | Double checkmark     |
| **Read Count**          | Pesan dibaca       | Blue checkmark       |
| **Failed Count**        | Pesan gagal        | Error tracking       |
| **Response Rate**       | Tingkat respons    | % replied            |
| **Chart Visualization** | Grafik             | Line, bar, pie       |
| **Date Range Filter**   | Filter periode     | Start/end date       |
| **Account Filter**      | Filter per akun    | Multi-select         |

---

### 10. Settings (⚙️)

**Lokasi:** `new-server/src/core/settings/`, `new-client/src/app/(dashboard)/settings/`  
**PRD Existing:** `docs/PRD-settings.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur                 | Deskripsi             | Parameter Detail               |
| ------------------------- | --------------------- | ------------------------------ |
| **Notification Settings** | Notifikasi            | Desktop, sound, email          |
| **Blast Defaults**        | Default blast config  | Delay preset, behavior         |
| **Warmer Defaults**       | Default warmer config | Preset, language               |
| **Warmer Presets CRUD**   | Kelola preset         | Create, edit, delete           |
| **Spam Filter**           | Filter keywords       | Blocked keywords list          |
| **Labels Management**     | Kelola labels         | Create, edit, reorder, delete  |
| **Contact Settings**      | Pengaturan kontak     | Custom fields, import settings |
| **Display Settings**      | Tampilan              | Theme, date/time format        |
| **Security Settings**     | Keamanan              | API key, whitelist             |
| **Backup & Reset**        | Data management       | Export, import, reset          |

---

### 11. Inbox (📥)

**Lokasi:** `new-server/src/core/inbox/`, `new-client/src/app/(dashboard)/inbox/`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur              | Deskripsi         | Parameter Detail    |
| ---------------------- | ----------------- | ------------------- |
| **Message List**       | Daftar pesan      | Per account, sorted |
| **Spam Filter**        | Filter otomatis   | Based on keywords   |
| **Label Assignment**   | Assign label      | Per conversation    |
| **Multi-Account View** | Gabung semua akun | Unified inbox       |
| **Reply**              | Balas pesan       | Text, media         |
| **Unread Badge**       | Counter unread    | Per account         |

---

### 12. Client/Account Management (👥)

**Lokasi:** `new-server/src/core/whatsapp/`, `new-client/src/app/(dashboard)/accounts/`  
**PRD Existing:** `docs/PRD-client-management.md`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur              | Deskripsi         | Parameter Detail            |
| ---------------------- | ----------------- | --------------------------- |
| **Session Management** | Kelola sesi WA    | Connect, disconnect, QR     |
| **Trust Level**        | Level kepercayaan | NEWCOMER, TRUSTED, VERIFIED |
| **Daily Limit**        | Batas harian      | sent/limit tracking         |
| **Session Recovery**   | Auto-reconnect    | On server restart           |
| **Multi-Session**      | Banyak akun       | Concurrent sessions         |
| **Status Monitoring**  | Real-time status  | Connected, disconnected     |

---

### 13. Auto-Reply (🤖)

**Lokasi:** `new-server/src/core/auto-reply/`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur              | Deskripsi          | Parameter Detail       |
| ---------------------- | ------------------ | ---------------------- |
| **Keyword Triggers**   | Trigger by keyword | Exact/partial match    |
| **Response Templates** | Template balasan   | Static or AI           |
| **Active Hours**       | Jam aktif          | Time range             |
| **Account Binding**    | Per-account config | Which accounts enabled |

---

### 14. Labels (🏷️)

**Lokasi:** `new-server/src/core/labels/`

#### Sub-fitur yang perlu didokumentasikan detail:

| Sub-fitur            | Deskripsi              | Parameter Detail  |
| -------------------- | ---------------------- | ----------------- |
| **Label CRUD**       | Kelola label           | Name, color, icon |
| **Label Assignment** | Assign ke chat/contact | Many-to-many      |
| **Label Filtering**  | Filter by label        | Inbox, contacts   |
| **Label Ordering**   | Urutan label           | Drag-drop reorder |

---

## 📝 Format Dokumentasi

Setiap fitur akan didokumentasikan dengan struktur:

````markdown
# Fitur: [Nama Fitur]

## 1. Overview

[Deskripsi singkat fitur]

## 2. Komponen Detail

### 2.1 [Sub-fitur 1]

**Deskripsi:** [Penjelasan]
**Parameter:**
| Parameter | Tipe | Default | Min | Max | Deskripsi |
|-----------|------|---------|-----|-----|-----------|
| param1 | ... | ... | ... | ... | ... |

**Contoh Penggunaan:**

```json
{...}
```
````

### 2.2 [Sub-fitur 2]

...

## 3. User Flow

[Diagram atau langkah-langkah]

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| ...    | ...      | ...       |

## 5. WebSocket Events

| Event | Direction | Payload |
| ----- | --------- | ------- |
| ...   | ...       | ...     |

## 6. Integrasi

[Hubungan dengan fitur lain]

```

---

## 📂 Structure Dokumentasi

```

docs/features/
├── README.md # Index semua fitur
├── 01-message-blast.md # Detail Message Blast
├── 02-account-warmer.md # Detail Account Warmer
├── 03-scheduler.md # Detail Scheduler System
├── 04-contacts.md # Detail Contacts Management
├── 05-number-checker.md # Detail Number Checker
├── 06-maps-scraper.md # Detail Maps Scraper
├── 07-ai-flow-builder.md # Detail AI Flow Builder
├── 08-templates.md # Detail Templates
├── 09-analytics.md # Detail Analytics
├── 10-settings.md # Detail Settings
├── 11-inbox.md # Detail Inbox
├── 12-account-management.md # Detail Account Management
├── 13-auto-reply.md # Detail Auto-Reply
└── 14-labels.md # Detail Labels

```

---

## ✅ Task Checklist

### Phase 1: Preparation
- [x] Create `docs/features/` directory
- [x] Create `docs/features/README.md` index

### Phase 2: Core Features (Priority)
- [x] Document Message Blast (01)
- [x] Document Account Warmer (02)
- [x] Document Scheduler System (03)

### Phase 3: Data Features
- [x] Document Contacts Management (04)
- [x] Document Number Checker (05)
- [x] Document Maps Scraper (06)

### Phase 4: AI & Content
- [x] Document AI Flow Builder (07)
- [x] Document Templates (08)

### Phase 5: Monitoring & Settings
- [x] Document Analytics (09)
- [x] Document Settings (10)

### Phase 6: Communication
- [x] Document Inbox (11)
- [x] Document Account Management (12)
- [x] Document Auto-Reply (13)
- [x] Document Labels (14)

### Phase 7: Review & Polish
- [x] Review semua dokumentasi
- [x] Tambahkan cross-references
- [x] Validasi parameter values dengan source code

---

## 🔗 Reference Files

| Fitur | PRD Existing | Source Code |
|-------|--------------|-------------|
| Blast | `docs/PRD-message-blast.md` | `new-server/src/core/blast/` |
| Warmer | `docs/PRD-warmer.md` | `new-server/src/core/warmer/` |
| Scheduler | `docs/PLAN-blast-scheduler.md` | `new-server/src/core/scheduler/` |
| Contacts | `docs/PRD-contacts.md` | `new-server/src/core/contacts/` |
| Number Checker | `docs/PRD-number-checker.md` | `new-server/src/core/number-checker/` |
| Maps Scraper | `docs/PLAN-maps-scraper.md` | `new-server/src/core/maps-scraper/` |
| AI Agent | - | `new-server/src/core/ai-agent/` |
| Templates | `docs/PRD-templates.md` | `new-server/src/core/templates/` |
| Analytics | `docs/PRD-analytics.md` | `new-server/src/core/analytics/` |
| Settings | `docs/PRD-settings.md` | `new-server/src/core/settings/` |
| Inbox | - | `new-server/src/core/inbox/` |
| Accounts | `docs/PRD-client-management.md` | `new-server/src/core/whatsapp/` |
| Auto-Reply | - | `new-server/src/core/auto-reply/` |
| Labels | - | `new-server/src/core/labels/` |

---

**Status:** Selesai
```
