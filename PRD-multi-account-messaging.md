# PRD: Unified Inbox (Conversation)

**Version:** 2.0  
**Status:** Draft  
**Date:** 2026-01-25

---

## 1. Overview

Fitur **Unified Inbox** memungkinkan pengguna mengelola semua percakapan WhatsApp dari banyak akun dalam satu tampilan terpusat. Fokus utama: **kemudahan filter, kejelasan akun pengirim/penerima, dan pengiriman yang aman dari kesalahan.**

---

## 2. Goals

- ✅ Semua pesan dari semua akun dalam satu inbox
- ✅ Filter fleksibel: tampilkan semua akun, atau pilih akun tertentu (multi-select)
- ✅ Jelas mana pesan diterima akun A vs akun B dari nomor yang sama
- ✅ Mudah memilih akun pengirim saat mengirim/membalas pesan
- ✅ 100% tidak ada kesalahan akun saat mengirim

---

## 3. User Stories

### 3.1 Lihat Semua Pesan

> Saya punya 5 akun WhatsApp dan ingin melihat semua pesan dalam satu tampilan.

### 3.2 Filter Akun Tertentu

> Saya hanya ingin melihat pesan dari akun "Toko A" dan "Toko B" saja, tidak yang lain.

### 3.3 Identifikasi Akun Penerima

> Nomor +62812345 mengirim pesan ke akun A dan B. Saya ingin tahu pesan mana yang masuk ke akun A dan mana ke akun B.

### 3.4 Pilih Akun Pengirim

> Saat membalas atau mengirim pesan baru, saya ingin memilih dengan mudah akun mana yang digunakan.

---

## 4. Multi-Account Filter

### 4.1 Filter UI

```
┌─────────────────────────────────────────────────┐
│ 📥 Inbox                                        │
├─────────────────────────────────────────────────┤
│ Filter Akun: [✓ All] [✓ Toko A] [✓ Toko B] [ ] Personal │
│                                                 │
│ ───────────────────────────────────────────────│
│ [Chat List...]                                  │
└─────────────────────────────────────────────────┘
```

### 4.2 Filter Behavior

| Selection           | Result                          |
| ------------------- | ------------------------------- |
| "All" checked       | Tampilkan semua chat            |
| Only "Toko A"       | Tampilkan chat akun Toko A saja |
| "Toko A" + "Toko B" | Tampilkan chat kedua akun saja  |
| None selected       | Tampilkan semua (fallback)      |

### 4.3 Filter State

Filter disimpan di URL query param atau local storage:

```
/inbox?accounts=acc-001,acc-002
```

---

## 5. Account Identification

### 5.1 Masalah: Nomor Sama, Akun Berbeda

Jika kontak +62812345 mengirim pesan ke "Akun A" dan "Akun B", sistem HARUS menampilkan sebagai **2 thread terpisah**:

```
┌─────────────────────────────────────────────────┐
│ 📱 +62812345             [🔵 Akun A]            │
│ "Halo, barang ready?"    10:30 AM              │
├─────────────────────────────────────────────────┤
│ 📱 +62812345             [🟢 Akun B]            │
│ "Mau tanya harga..."     10:25 AM              │
└─────────────────────────────────────────────────┘
```

### 5.2 Thread ID Structure

```typescript
threadId = `${accountId}:${contactNumber}`;
```

**Contoh:**

- `acc-001:6281234567890` → Chat +62812345 via Akun A
- `acc-002:6281234567890` → Chat +62812345 via Akun B

### 5.3 Visual Badge Per Akun

| Akun     | Badge Color | Badge Text   |
| -------- | ----------- | ------------ |
| Toko A   | 🔵 Blue     | "[Toko A]"   |
| Toko B   | 🟢 Green    | "[Toko B]"   |
| Personal | 🟣 Purple   | "[Personal]" |

Warna di-generate berdasarkan hash dari `accountId` untuk konsistensi.

---

## 6. Smart Account Selection

### 6.1 Auto-Select Logic

Sistem otomatis menentukan akun berdasarkan histori chat:

| Skenario                                 | Behavior                          |
| ---------------------------------------- | --------------------------------- |
| Kontak X hanya pernah chat dengan Akun A | **Auto-select A** (tanpa picker)  |
| Kontak X pernah chat dengan A dan B      | **User pilih** (tampilkan picker) |
| Kontak baru (belum pernah chat)          | **User pilih**                    |

### 6.2 Single Thread → Langsung Kirim

Jika kontak hanya punya 1 thread → langsung balas tanpa pilih akun:

```
┌─────────────────────────────────────────────────┐
│ +62812345 [🔵 Toko A]                           │
├─────────────────────────────────────────────────┤
│ [Pesan-pesan...]                                │
├─────────────────────────────────────────────────┤
│ [Ketik pesan...]                      [Kirim] │
└─────────────────────────────────────────────────┘
```

**Tidak ada dropdown.** Akun otomatis: Toko A.

### 6.3 Multiple Threads → Tampilkan Picker

Jika kontak punya thread di beberapa akun → tampilkan dropdown:

```
┌─────────────────────────────────────────────────┐
│ +62812345                                       │
│ Tersedia di: 🔵 Toko A | 🟢 Toko B              │
├─────────────────────────────────────────────────┤
│ Kirim sebagai: [🔵 Toko A ▼]     ← DROPDOWN     │
│ [Ketik pesan...]                      [Kirim] │
└─────────────────────────────────────────────────┘
```

### 6.4 Kontak Baru → Wajib Pilih

Saat kirim ke nomor baru yang belum pernah chat:

```
┌─────────────────────────────────────────────────┐
│ 📝 Pesan Baru                                   │
├─────────────────────────────────────────────────┤
│ Kepada: [+628...]                               │
│ Kirim sebagai: [Pilih akun ▼]   ← REQUIRED     │
│ [Ketik pesan...]                      [Kirim] │
└─────────────────────────────────────────────────┘
```

### 6.5 Implementation Logic

```typescript
function getAccountsForContact(contactNumber: string): string[] {
  // Cari semua akun yang pernah menerima pesan dari nomor ini
  return db.inboxMessage
    .findMany({
      where: { senderNumber: contactNumber },
      select: { accountId: true },
      distinct: ["accountId"],
    })
    .map((t) => t.accountId);
}

// Saat user buka chat:
const accounts = getAccountsForContact("+62812345");

if (accounts.length === 0) {
  // Kontak baru → wajib pilih
  showAccountPicker = true;
} else if (accounts.length === 1) {
  // Single account → auto-select, hide picker
  selectedAccount = accounts[0];
  showAccountPicker = false;
} else {
  // Multiple accounts → show picker
  showAccountPicker = true;
  availableAccounts = accounts;
}
```

---

## 7. Data Model

### 7.1 InboxMessage

| Field          | Type     | Description                     |
| -------------- | -------- | ------------------------------- |
| `id`           | String   | Message ID                      |
| `accountId`    | String   | **Akun penerima**               |
| `threadId`     | String   | `${accountId}:${contactNumber}` |
| `senderNumber` | String   | Nomor pengirim                  |
| `senderName`   | String?  | Nama kontak                     |
| `body`         | String   | Isi pesan                       |
| `messageType`  | Enum     | text, image, video, document    |
| `mediaPath`    | String?  | Path file media                 |
| `isRead`       | Boolean  | Status dibaca                   |
| `receivedAt`   | DateTime | Waktu diterima                  |

### 7.2 OutboxMessage

| Field             | Type     | Description                            |
| ----------------- | -------- | -------------------------------------- |
| `id`              | String   | Message ID                             |
| `accountId`       | String   | **Akun pengirim**                      |
| `threadId`        | String   | `${accountId}:${contactNumber}`        |
| `recipientNumber` | String   | Nomor tujuan                           |
| `body`            | String   | Isi pesan                              |
| `status`          | Enum     | pending, sent, delivered, read, failed |
| `sentAt`          | DateTime | Waktu kirim                            |

---

## 8. API Endpoints

| Method | Endpoint                          | Description                      |
| ------ | --------------------------------- | -------------------------------- |
| GET    | `/inbox/chats?accounts=a,b`       | List chat threads (filtered)     |
| GET    | `/inbox/chats/:threadId/messages` | Get messages in thread           |
| POST   | `/inbox/chats/:threadId/reply`    | Reply (auto-use thread account)  |
| POST   | `/outbox/send`                    | Send new (manual select account) |
| PATCH  | `/inbox/messages/:id/read`        | Mark as read                     |

### 8.1 Send Request Body

```typescript
interface SendRequest {
  accountId: string; // WAJIB: akun pengirim
  recipientNumber: string;
  body: string;
  mediaPath?: string;
}
```

---

## 9. Label Management

### 9.1 Overview

Label digunakan untuk **menandai dan mengorganisir chat** (BUKAN untuk filter pesan masuk). User dapat menambah, mengedit, dan menghapus label.

### 9.2 Label UI

```
┌─────────────────────────────────────────────────┐
│ 🏷️ Label Manager                         [+ Baru] │
├─────────────────────────────────────────────────┤
│ 🔴 Urgent          │ 5 chat  │ [✏️] [🗑️]       │
│ 🟢 Follow Up       │ 12 chat │ [✏️] [🗑️]       │
│ 🔵 VIP Customer    │ 8 chat  │ [✏️] [🗑️]       │
│ 🟡 Pending Payment │ 3 chat  │ [✏️] [🗑️]       │
│ 🟣 Resolved        │ 25 chat │ [✏️] [🗑️]       │
└─────────────────────────────────────────────────┘
```

### 9.3 Assign Label ke Chat

Pada chat header atau context menu:

```
[🏷️ Add Label ▼]
├── 🔴 Urgent
├── 🟢 Follow Up
├── 🔵 VIP Customer
└── [+ Buat Label Baru]
```

### 9.4 Filter by Label

```
Filter: [All Labels ▼] [🔴 Urgent] [🟢 Follow Up]
```

### 9.5 Label Schema

| Field   | Type   | Description     |
| ------- | ------ | --------------- |
| `id`    | String | Label ID        |
| `name`  | String | Nama label      |
| `color` | String | Warna hex       |
| `order` | Int    | Urutan tampilan |

---

## 10. Spam Filter (Auto-Block Messages)

### 10.1 Overview

Sistem otomatis memblokir pesan yang mengandung kata-kata tertentu agar **tidak masuk ke inbox**. Cocok untuk memfilter auto-reply spam.

### 10.2 Default Blocked Keywords

Sistem sudah memiliki keywords default:

```
[
  "terima kasih telah menghubungi",
  "pesan anda telah diterima",
  "balasan otomatis",
  "auto reply",
  "di luar jam kerja",
  "akan segera menghubungi"
]
```

### 10.3 Keyword Manager UI

```
┌─────────────────────────────────────────────────┐
│ 🚫 Spam Filter                                  │
├─────────────────────────────────────────────────┤
│ ✅ Filter aktif (15 pesan diblokir hari ini)    │
│                                                 │
│ Keywords yang diblokir:                         │
│ ┌───────────────────────────────────────────┐   │
│ │ [✓] terima kasih telah menghubungi   [🗑️] │   │
│ │ [✓] pesan anda telah diterima        [🗑️] │   │
│ │ [✓] balasan otomatis                 [🗑️] │   │
│ │ [✓] auto reply                       [🗑️] │   │
│ │ [✓] di luar jam kerja                [🗑️] │   │
│ │ [ ] keyword custom user              [🗑️] │   │
│ └───────────────────────────────────────────┘   │
│                                                 │
│ [+ Tambah Keyword]                              │
│                                                 │
│ ⚙️ Pengaturan:                                  │
│ [✓] Case insensitive                            │
│ [✓] Partial match (mengandung kata)             │
└─────────────────────────────────────────────────┘
```

### 10.4 Blocked Messages Log

User dapat melihat pesan yang terblokir:

```
┌─────────────────────────────────────────────────┐
│ 📋 Pesan Terblokir (15)               [Kosongkan] │
├─────────────────────────────────────────────────┤
│ +62812345... │ "Terima kasih telah..." │ 10:30 │
│ +62898765... │ "Pesan anda telah..."   │ 10:25 │
│ +62811122... │ "Balasan otomatis..."   │ 10:20 │
└─────────────────────────────────────────────────┘
           [↩️ Pulihkan ke Inbox]
```

---

## 11. Auto-Response Detection

### 11.1 Overview

Sistem mendeteksi apakah pesan adalah **auto-reply atau pesan personal asli**.

### 11.2 Detection Logic

```typescript
function isAutoResponse(message: string): boolean {
  const autoResponsePatterns = [
    /terima kasih.*menghubungi/i,
    /pesan.*diterima/i,
    /balasan otomatis/i,
    /auto.?reply/i,
    /akan.*menghubungi.*kembali/i,
  ];
  return autoResponsePatterns.some((p) => p.test(message));
}
```

### 11.3 Personal Message Notification

Jika pengirim yang sebelumnya mengirim auto-reply, sekarang mengirim pesan personal:

```
┌─────────────────────────────────────────────────┐
│ 🔔 Pesan Personal dari +62812345...             │
│                                                 │
│ Pengirim ini sebelumnya mengirim auto-reply.    │
│ Sekarang mengirim pesan asli:                   │
│                                                 │
│ "Halo, saya tertarik dengan produk Anda..."     │
│                                                 │
│ [Lihat Chat]                    [Dismiss]       │
└─────────────────────────────────────────────────┘
```

### 11.4 UI Indicator

Pada chat list, tampilkan badge:

- 🤖 = Terakhir pesan adalah auto-reply
- 👤 = Terakhir pesan adalah personal

---

## 12. WebSocket Events

| Event                | Direction       | Payload                            |
| -------------------- | --------------- | ---------------------------------- |
| `inbox:new-message`  | Server → Client | `{ threadId, accountId, message }` |
| `inbox:message-read` | Server → Client | `{ messageId }`                    |
| `outbox:sent`        | Server → Client | `{ messageId, status }`            |
| `outbox:failed`      | Server → Client | `{ messageId, error }`             |

---

## 10. UI Components

### 10.1 Account Filter Bar

- Checkbox multi-select untuk filter akun
- "Select All" / "Clear" buttons
- Persist selection ke URL/localStorage

### 10.2 Chat List Item

- Contact name/number
- **Account badge (warna + nama)**
- Last message preview
- Unread count
- Timestamp

### 10.3 Chat Window Header

- Contact info
- **Account indicator besar dan jelas**
- Account dropdown (untuk switch)

### 10.4 Message Composer

- Account selector (dropdown atau locked)
- Text input
- Media attachment button
- Send button

---

## 11. Sender Profile Panel (via Dialog)

Saat user menerima pesan, dapat melihat detail pengirim melalui Dialog interaktif yang mudah diakses.

### 11.1 Dialog Trigger
- Klik pada Avatar atau Nama Kontak di header chat window.
- Tombol "Info" (ikon (i)) di header.

### 11.2 Dialog UI (Profile Card)

```
┌───────────────────────────────────────────────────┐
│ [X] Contact Info                                  │
├───────────────────────────────────────────────────┤
│        [Foto Profil WA]                           │
│           Budi Santoso                            │
│           📱 +6281234567890                       │
│                                                   │
│ [Button: WhatsApp] [Button: CRM]                  │
│                                                   │
│ ───────────────── WhatsApp ────────────────────── │
│  📝 Status: "Hidup itu indah"                     │
│  📅 Last Seen: Hari ini, 10:30                    │
│                                                   │
│ ─────────────── Data Kontak ───────────────────── │
│  ✅ Tersimpan di Kontak                           │
│                                                   │
│  👤 Nama       : Budi Santoso                     │
│  🏢 Bisnis     : Toko Elektronik Jaya             │
│  🏷️ Kategori   : Retail Electronics              │
│  📍 Alamat     : Jl. Sudirman No. 45              │
│  ✉️ Email      : budi@tokoelektronik.com          │
│  ⭐ Rating     : ★★★★☆ (4.2)                      │
│                                                   │
│ ─────────────── Riwayat ───────────────────────── │
│  📅 Terakhir: 15 Jan 2026                         │
│                                                   │
│  [✏️ Edit Kontak]  [🗑️ Hapus]                     │
└───────────────────────────────────────────────────┘
```

### 11.2 Kontak Tidak Ditemukan

Jika nomor tidak ada di database kontak:

```
┌───────────────────────────────────────────────────┐
│        [Foto Profil WA]                           │
│           📱 +6281234567890                       │
│           (Nama dari WA)                          │
│                                                   │
│ ─────────────── Data Kontak ───────────────────── │
│  ⚠️ Tidak tersimpan di Kontak                     │
│                                                   │
│  [+ Tambah ke Kontak]                             │
└───────────────────────────────────────────────────┘
```

### 11.3 Contact Data Schema

| Field                | Icon | Type    | Description        |
| -------------------- | ---- | ------- | ------------------ |
| `name`               | 👤   | String  | Nama kontak        |
| `phoneNumber`        | 📱   | String  | Nomor WhatsApp     |
| `address`            | 📍   | String  | Alamat lengkap     |
| `city`               | 🏙️   | String  | Kota               |
| `state`              | 🗺️   | String  | Provinsi           |
| `country`            | 🌍   | String  | Negara             |
| `postalCode`         | 📮   | String  | Kode pos           |
| `email`              | ✉️   | String  | Email              |
| `website`            | 🌐   | String  | Website            |
| `businessName`       | 🏢   | String  | Nama bisnis        |
| `businessCategory`   | 🏷️   | String  | Kategori bisnis    |
| `rating`             | ⭐   | Float   | Rating (1-5)       |
| `reviews`            | 📊   | Int     | Jumlah review      |
| `isBusiness`         | 🏪   | Boolean | Apakah akun bisnis |
| `hasReceivedMessage` | ✅   | Boolean | Pernah dihubungi   |
| `latitude`           | 📍   | Float   | Koordinat lat      |
| `longitude`          | 📍   | Float   | Koordinat lng      |

---

## 12. API Tambahan

### 12.1 Contact Lookup

| Method | Endpoint                        | Description          |
| ------ | ------------------------------- | -------------------- |
| GET    | `/contacts/lookup/:phoneNumber` | Cari kontak by nomor |
| POST   | `/contacts`                     | Tambah kontak baru   |
| PUT    | `/contacts/:id`                 | Update kontak        |

### 12.2 WhatsApp Profile

| Method | Endpoint                                    | Description   |
| ------ | ------------------------------------------- | ------------- |
| GET    | `/whatsapp/:accountId/profile/:phoneNumber` | Get profil WA |

---

## 13. Success Metrics

- [ ] 100% akurasi akun (tidak ada salah kirim)
- [ ] Filter response time < 100ms
- [ ] Clear account indicator di setiap chat
- [ ] Pesan real-time update < 500ms
- [ ] Profile panel load < 500ms
- [ ] Contact lookup accurate

---

## 14. Integrasi dengan Fitur Lain

### 14.1 Dependency

| Fitur             | Status                                            |
| ----------------- | ------------------------------------------------- |
| Client Management | ⬅️ Akun list untuk filter, sender identity        |
| Contacts          | ⬅️ Sender profile lookup                          |
| Templates         | ⬅️ Quick reply template picker                    |
| Settings          | ⬅️ Spam filter keywords, labels, display settings |

### 14.2 Provides to Other Features

| Consumer      | Data/Service                        |
| ------------- | ----------------------------------- |
| **Analytics** | receivedCount, replyCount           |
| **Templates** | Trigger +3 poin saat reply detected |

### 14.3 Integration Checklist

Saat membangun fitur ini, **WAJIB** integrasi:

- [ ] Account filter: `GET /whatsapp/sessions`
- [ ] Sender lookup: `GET /contacts/lookup/:phoneNumber`
- [ ] "Tambah ke Kontak": `POST /contacts`
- [ ] Spam filter: `GET /settings` → `spamFilter.keywords`
- [ ] Labels: `GET /settings` → `labels`
- [ ] Template quick access: `GET /templates/top`
- [ ] Point tracking: Saat reply → lookup `templateId` → `POST /templates/:id/reply`
- [ ] Analytics: Update `receivedCount` di analytics entry

### 14.4 Spam Filter Flow

```
Pesan masuk
    ↓
Check against spamFilter.keywords
    ↓
├── Match → Ke blocked log (tidak masuk inbox)
└── No match → Masuk inbox
```

---

**Status:** Menunggu Review
