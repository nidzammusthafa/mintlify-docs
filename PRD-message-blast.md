# PRD: Message Blast (WA Blast)

**Version:** 2.0  
**Status:** Draft  
**Date:** 2026-01-25

---

## 1. Overview

Fitur **Message Blast** memungkinkan pengguna mengirim pesan ke banyak nomor sekaligus dengan:

- Rotasi multi-akun otomatis
- Delay cerdas untuk menghindari spam detection
- Personalisasi pesan dengan variabel
- Progress tracking real-time

**Fokus Utama:** Kemudahan penggunaan dengan wizard step-by-step.

---

## 2. Goals

- ✅ Kirim ke ribuan nomor dalam satu klik
- ✅ Rotasi akun otomatis untuk mencegah ban
- ✅ Delay cerdas sesuai trust level akun
- ✅ Progress real-time dan pausable
- ✅ UI wizard yang mudah dipahami

---

## 3. User Flow: 4-Step Wizard

### Step 1: Pilih Akun Pengirim

```
┌─────────────────────────────────────────────────┐
│ 📤 Message Blast - Step 1/4                     │
│ ─────────────────────────────────────────────── │
│ Pilih Akun Pengirim:                            │
│                                                 │
│ [✓] 🔵 Toko A (Trust: VERIFIED, Limit: 120/150) │
│ [✓] 🟢 Toko B (Trust: TRUSTED, Limit: 50/300)   │
│ [ ] 🟣 Personal (Trust: NEWCOMER, Limit: 45/50) │
│                                                 │
│ Total Capacity: 175 pesan tersisa               │
│                                                 │
│                              [Selanjutnya →]    │
└─────────────────────────────────────────────────┘
```

### Step 2: Upload Penerima

```
┌─────────────────────────────────────────────────┐
│ 📤 Message Blast - Step 2/4                     │
│ ─────────────────────────────────────────────── │
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │     📁 Drag & drop file Excel/CSV         │  │
│  │         atau klik untuk upload            │  │
│  └───────────────────────────────────────────┘  │
│                                                 │
│ ✅ contacts.xlsx (250 kontak)                   │
│                                                 │
│ Kolom Nomor HP: [phone_number ▼]                │
│                                                 │
│ Preview:                                        │
│ ┌──────────────────────────────────────────┐    │
│ │ No │ phone_number  │ nama   │ kota       │    │
│ │ 1  │ 628123456789  │ Budi   │ Jakarta    │    │
│ │ 2  │ 628987654321  │ Ani    │ Surabaya   │    │
│ └──────────────────────────────────────────┘    │
│                                                 │
│ [← Kembali]                    [Selanjutnya →]  │
└─────────────────────────────────────────────────┘
```

### Step 3: Susun Pesan (Multi-Block)

User dapat menyusun **beberapa blok pesan** yang akan dikirim berurutan:

```
┌─────────────────────────────────────────────────┐
│ 📤 Message Blast - Step 3/4                     │
│ ─────────────────────────────────────────────── │
│                                                 │
│ 📝 Blok Pesan (kirim berurutan):                │
│                                                 │
│ ┌─ Blok 1: Salam ───────────────────────────┐   │
│ │ Halo {nama}! 👋                           │   │
│ │ Apa kabar? Semoga sehat selalu.           │   │
│ └───────────────────────────────────── [✕] ─┘   │
│                                                 │
│ ┌─ Blok 2: Media ───────────────────────────┐   │
│ │ 🖼️ promo-banner.jpg                       │   │
│ │ Caption: Lihat promo spesial ini!         │   │
│ └───────────────────────────────────── [✕] ─┘   │
│                                                 │
│ ┌─ Blok 3: Isi Iklan ───────────────────────┐   │
│ │ 🎉 PROMO AKHIR TAHUN!                     │   │
│ │ Diskon s/d 50% untuk {kota}.              │   │
│ │ Kunjungi toko kami sekarang!              │   │
│ └───────────────────────────────────── [✕] ─┘   │
│                                                 │
│ [+ Tambah Blok Teks]  [+ Tambah Media]          │
│ [📋 Gunakan Template]                           │
│                                                 │
│ Variabel: {nama} {kota} {phone_number}          │
│                                                 │
│ [← Kembali]                    [Selanjutnya →]  │
└─────────────────────────────────────────────────┘
```

**Tipe Blok:**
| Tipe | Icon | Description |
|------|------|-------------|
| Text | 📝 | Pesan teks biasa |
| Media | 🖼️ | Gambar/Video/Dokumen dengan caption |
| Randomized | 🎲 | Pilih acak dari beberapa variasi |

---

## 3.1 Template Pesan

User dapat menyimpan dan menggunakan template pesan yang sudah dibuat sebelumnya.

### Menggunakan Template

```
┌─────────────────────────────────────────────────┐
│ 📋 Pilih Template                        [✕]   │
├─────────────────────────────────────────────────┤
│ 🔍 [Cari template...]                           │
│                                                 │
│ ┌───────────────────────────────────────────┐   │
│ │ 📌 Promo Akhir Tahun                      │   │
│ │ 3 blok • Terakhir dipakai: 2 hari lalu    │   │
│ └───────────────────────────────────────────┘   │
│ ┌───────────────────────────────────────────┐   │
│ │ 📌 Welcome Message                        │   │
│ │ 1 blok • Terakhir dipakai: 1 minggu lalu  │   │
│ └───────────────────────────────────────────┘   │
│ ┌───────────────────────────────────────────┐   │
│ │ 📌 Follow Up                              │   │
│ │ 2 blok • Terakhir dipakai: 3 hari lalu    │   │
│ └───────────────────────────────────────────┘   │
│                                                 │
│ [Batal]                        [Gunakan]        │
└─────────────────────────────────────────────────┘
```

### Menyimpan Sebagai Template

Setelah menyusun pesan, user dapat klik **"Simpan sebagai Template"** untuk menyimpan konfigurasi blok pesan dan bisa digunakan kembali.

### Step 4: Konfigurasi & Kirim

```
┌─────────────────────────────────────────────────┐
│ 📤 Message Blast - Step 4/4                     │
│ ─────────────────────────────────────────────── │
│ Pengaturan Pengiriman:                          │
│                                                 │
│ Preset: [🚶 Normal (Recommended) ▼]             │
│                                                 │
│ ⚙️ Pengaturan Lanjutan (opsional):              │
│ ├── Delay antar pesan: 8-15 detik               │
│ ├── Jeda setiap 10 penerima: 30 detik           │
│ └── Skip kontak di Address Book: ✓              │
│                                                 │
│ ─────────────────────────────────────────────── │
│ 📊 Ringkasan:                                   │
│ • Akun: Toko A, Toko B (2 akun)                 │
│ • Penerima: 250 kontak                          │
│ • Estimasi waktu: ~45 menit                     │
│                                                 │
│ [← Kembali]        [🚀 Mulai Blast]             │
└─────────────────────────────────────────────────┘
```

---

## 4. Multi-Sender Rotation

### 4.1 Algoritma Round-Robin

```
Accounts: [A, B]
Recipients: [1, 2, 3, 4, 5, 6]

Result:
  A → 1
  B → 2
  A → 3
  B → 4
  A → 5
  B → 6
```

### 4.2 Skip Exhausted Accounts

Jika akun mencapai daily limit, otomatis skip ke akun berikutnya:

```
A: Limit 2/2 (exhausted)
B: Limit 1/5 (available)

Recipients: [1, 2, 3]
Result:
  B → 1
  B → 2
  B → 3
```

---

## 5. Delay Presets

| Preset    | Min Delay | Max Delay | Jeda Setiap N | Extra Delay | Use Case            |
| --------- | --------- | --------- | ------------- | ----------- | ------------------- |
| 🐢 Aman   | 15s       | 30s       | 5             | 60s         | Akun baru           |
| 🚶 Normal | 8s        | 15s       | 10            | 30s         | Default/Recommended |
| 🏃 Cepat  | 3s        | 8s        | 20            | 15s         | Akun trusted        |
| ⚙️ Custom | -         | -         | -             | -           | Power user          |

---

## 6. Variable Substitution

### 6.1 Placeholder Format

| Placeholder     | Description            |
| --------------- | ---------------------- |
| `{column_name}` | Nilai dari kolom Excel |
| `{nama}`        | Kolom "nama"           |
| `{kota}`        | Kolom "kota"           |

### 6.2 Preview Real-time

Saat user mengetik pesan, preview langsung ditampilkan dengan data dari baris pertama Excel.

---

## 7. Message Variations (Anti-Spam)

Untuk menghindari deteksi spam, user bisa membuat variasi pesan:

```typescript
messageBlocks: [
  {
    type: "randomized",
    variants: [
      "Halo {nama}! Ada promo nih.",
      "Hi {nama}, cek promo terbaru!",
      "Hai {nama}! Jangan lewatkan diskon ini.",
    ],
  },
];
```

Sistem akan secara acak memilih salah satu variasi untuk setiap penerima.

---

## 8. Job Progress & Control

### 8.1 Progress Dashboard

```
┌─────────────────────────────────────────────────┐
│ 📤 Blast: Promo Akhir Tahun                     │
│ ─────────────────────────────────────────────── │
│ Status: 🟢 RUNNING                              │
│                                                 │
│ [███████████░░░░░░░░░] 55% (138/250)            │
│                                                 │
│ ✅ Terkirim: 130                                │
│ ❌ Gagal: 8                                     │
│ ⏳ Sisa: 112                                    │
│                                                 │
│ Estimasi selesai: 20 menit lagi                 │
│                                                 │
│ [⏸️ Pause]  [⏹️ Stop]  [📋 Lihat Log]           │
└─────────────────────────────────────────────────┘
```

### 8.2 Job Status Lifecycle

```
PENDING → SCHEDULED → IN_PROGRESS ⇄ PAUSED
                           ↓
              COMPLETED / CANCELLED / FAILED
```

### 8.3 Control Actions

| Action | Description                       |
| ------ | --------------------------------- |
| Pause  | Jeda pengiriman, bisa dilanjutkan |
| Resume | Lanjutkan dari posisi terakhir    |
| Stop   | Hentikan permanen                 |
| Delete | Hapus job dan log                 |

---

## 9. Progress Table & Message Log

### 9.1 Progress Table UI

Tabel real-time yang menampilkan status setiap pengiriman:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 📊 Progress Pengiriman                              Filter: [All ▼]  [🔍]  │
├─────────────────────────────────────────────────────────────────────────────┤
│ No │ Nomor Target    │ Pengirim      │ Pesan (Preview)    │ Status │ Error │
├────┼─────────────────┼───────────────┼────────────────────┼────────┼───────┤
│ 1  │ +6281234567890  │ 🔵 Toko A     │ "Halo Budi! 👋..." │ ✅ Sent │       │
│ 2  │ +6289876543210  │ 🟢 Toko B     │ "Halo Ani! 👋..."  │ ✅ Sent │       │
│ 3  │ +6281111222233  │ 🔵 Toko A     │ "Halo Doni! 👋..." │ ❌ Fail │ No WA │
│ 4  │ +6282222333344  │ 🟢 Toko B     │ "Halo Eka! 👋..."  │ ⏳ Pend │       │
│ 5  │ +6283333444455  │ 🔵 Toko A     │ "Halo Fira! 👋..." │ ⏳ Pend │       │
└────┴─────────────────┴───────────────┴────────────────────┴────────┴───────┘
                                                    [Export Excel 📥]
```

### 9.2 Kolom Tabel

| Kolom            | Description                              |
| ---------------- | ---------------------------------------- |
| **No**           | Nomor urut pengiriman                    |
| **Nomor Target** | Nomor WhatsApp tujuan                    |
| **Pengirim**     | Akun yang digunakan (dengan badge warna) |
| **Pesan**        | Preview pesan yang dikirim (truncated)   |
| **Status**       | ✅ Sent / ❌ Failed / ⏳ Pending         |
| **Error**        | Alasan gagal (jika ada)                  |

### 9.3 Filter & Search

| Filter  | Options                       |
| ------- | ----------------------------- |
| Status  | All / Sent / Failed / Pending |
| Account | All / Toko A / Toko B / ...   |
| Search  | Cari berdasarkan nomor        |

### 9.4 Export Log

User dapat export log ke Excel:

- Semua kolom termasuk
- Filter aktif diterapkan
- Timestamp di nama file

---

## 10. API Endpoints

| Method | Endpoint                   | Description         |
| ------ | -------------------------- | ------------------- |
| GET    | `/blast`                   | List semua job      |
| POST   | `/blast`                   | Buat job baru       |
| GET    | `/blast/:jobId`            | Detail job          |
| POST   | `/blast/:jobId/start`      | Mulai job           |
| POST   | `/blast/:jobId/pause`      | Pause job           |
| POST   | `/blast/:jobId/resume`     | Resume job          |
| POST   | `/blast/:jobId/stop`       | Stop job            |
| DELETE | `/blast/:jobId`            | Hapus job           |
| GET    | `/blast/:jobId/log`        | Get message log     |
| GET    | `/blast/:jobId/log/export` | Export log ke Excel |

---

## 11. WebSocket Events

| Event                | Direction       | Payload                                   |
| -------------------- | --------------- | ----------------------------------------- |
| `blast:progress`     | Server → Client | `{ jobId, sent, failed, total, percent }` |
| `blast:message-sent` | Server → Client | `{ jobId, recipientNumber, status }`      |
| `blast:completed`    | Server → Client | `{ jobId, summary }`                      |
| `blast:error`        | Server → Client | `{ jobId, error }`                        |

---

## 12. Quick Access

### 12.1 Dashboard Widget

```
┌─────────────────────────────────────────────────┐
│ 📤 Message Blast                                │
├─────────────────────────────────────────────────┤
│ [+ Blast Baru]                                  │
│                                                 │
│ Recent Jobs:                                    │
│ • Promo Akhir Tahun (55% - Running)             │
│ • Flash Sale (Completed - 500 sent)             │
│ • Welcome Message (Paused - 120/300)            │
└─────────────────────────────────────────────────┘
```

### 12.2 Sidebar Entry

- Icon: 📤 atau Megaphone
- Label: "Message Blast"
- Badge: Jumlah job running

---

## 13. Contacts Integration

### 13.1 Skip Kontak di Address Book

Saat blast, user dapat memilih untuk skip nomor yang sudah ada di database kontak:

```
[✓] Skip kontak yang ada di Address Book
    └── 45 nomor akan di-skip (sudah pernah dihubungi)
```

### 13.2 Auto-Save Recipient ke Kontak

Setelah blast selesai, user dapat menyimpan recipient baru ke database kontak:

```
[Simpan 205 nomor baru ke Kontak]
```

### 13.3 Contacts Schema Reference

| Field                | Icon | Description      |
| -------------------- | ---- | ---------------- |
| `name`               | 👤   | Nama kontak      |
| `phoneNumber`        | 📱   | Nomor WhatsApp   |
| `city`               | 🏙️   | Kota             |
| `address`            | 📍   | Alamat lengkap   |
| `email`              | ✉️   | Email            |
| `website`            | 🌐   | Website          |
| `businessName`       | 🏢   | Nama bisnis      |
| `businessCategory`   | 🏷️   | Kategori bisnis  |
| `rating`             | ⭐   | Rating (1-5)     |
| `hasReceivedMessage` | ✅   | Pernah dihubungi |

### 13.4 Auto-Update Contact Status

Saat pesan terkirim ke nomor X yang ada di database:

```typescript
// Setelah pesan berhasil terkirim
await db.contact.update({
  where: { phoneNumber: recipientNumber },
  data: {
    hasReceivedMessage: true,
    lastContactedAt: new Date(),
  },
});
```

---

## 14. Warmer Integration

### 14.1 Jeda Modes

User dapat memilih mode jeda:

| Mode                   | Description                                  |
| ---------------------- | -------------------------------------------- |
| **Jeda Cerdas**        | Jeda berdasarkan delay preset (waktu)        |
| **Jeda dengan Warmer** | Blast berhenti saat warmer jalan, bergantian |

```
┌─────────────────────────────────────────────────┐
│ ⏱️ Mode Jeda                                    │
├─────────────────────────────────────────────────┤
│ ○ Jeda Cerdas (delay preset)                    │
│ ● Jeda dengan Warmer                            │
│   └── Blast pause saat warming, bergantian      │
└─────────────────────────────────────────────────┘
```

### 14.2 Alternating Execution Flow

```
┌──────────────────────────────────────────────────────────────┐
│ Timeline Blast + Warmer (Alternating Mode)                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ BLAST  ████████░░░░░░░░████████░░░░░░░░████████              │
│ WARMER ░░░░░░░░████████░░░░░░░░████████░░░░░░░░              │
│                                                              │
│ └─ 10 pesan ─┘└ warming ┘└─ 10 pesan ─┘└ warming ┘           │
└──────────────────────────────────────────────────────────────┘
```

**Mekanisme:**

1. Blast mengirim N pesan (configurable)
2. Blast **PAUSE**
3. Warmer jalan dengan M pesan
4. Warmer selesai
5. Blast **RESUME**
6. Ulangi sampai selesai

### 14.3 Warmer Preset (Reusable Config)

User dapat menyimpan konfigurasi warmer dengan nama untuk digunakan kembali:

```
┌─────────────────────────────────────────────────┐
│ 🔥 Pilih Warmer Preset                          │
├─────────────────────────────────────────────────┤
│                                                 │
│ ○ 🐢 Slow Warmer                                │
│   └── 2-3 pesan, delay 30-60s, interval 60s    │
│                                                 │
│ ● 🚶 Normal Warmer (Recommended)                │
│   └── 3-5 pesan, delay 10-30s, interval 30s    │
│                                                 │
│ ○ 🏃 Fast Warmer                                │
│   └── 5-8 pesan, delay 5-15s, interval 20s     │
│                                                 │
│ ○ ⚙️ Custom: "Warmer Toko" ──────── [✏️ Edit]   │
│   └── 4-6 pesan, delay 15-25s, interval 45s    │
│                                                 │
│ [+ Buat Preset Baru]                            │
└─────────────────────────────────────────────────┘
```

### 14.4 Warmer Preset Schema

```typescript
interface WarmerPreset {
  id: string;
  name: string; // Nama preset (untuk identifikasi)
  minMessages: number; // Min pesan per interval
  maxMessages: number; // Max pesan per interval
  intervalMs: number; // Jeda setelah N blast messages
  minDelayMs: number; // Min delay antar pesan warmer
  maxDelayMs: number; // Max delay antar pesan warmer
  language: "id" | "en";
  blastPauseAfterNRecipients: number; // Pause blast setelah N penerima
  isDefault: boolean; // Preset bawaan sistem
}
```

### 14.5 Blast + Warmer Config UI

```
┌─────────────────────────────────────────────────┐
│ ⚙️ Pengaturan Warmer                            │
├─────────────────────────────────────────────────┤
│ [✓] Aktifkan Jeda dengan Warmer                 │
│                                                 │
│ Warmer Preset: [🚶 Normal Warmer ▼]             │
│                                                 │
│ Pause blast setiap: [10] penerima               │
│ Kemudian jalankan: [3-5] pesan warmer           │
│                                                 │
│ 📊 Estimasi:                                    │
│ • 250 penerima = 25 sesi warmer                 │
│ • Total waktu: ~90 menit                        │
└─────────────────────────────────────────────────┘
```

---

## 15. Job Persistence & Recovery

### 15.1 Overview

Blast dan Warmer job disimpan di database untuk **recovery otomatis** saat server restart/crash.

### 15.2 Job State Storage

```typescript
interface BlastJobState {
  jobId: string;
  status: "PENDING" | "RUNNING" | "PAUSED" | "COMPLETED" | "FAILED";

  // Progress tracking
  currentRecipientIndex: number;
  sentCount: number;
  failedCount: number;

  // Full config (untuk resume)
  senderAccountIds: string[];
  recipients: Recipient[];
  messageBlocks: MessageBlock[];
  delayConfig: DelayConfig;
  warmerPresetId?: string;

  // Timestamps
  createdAt: DateTime;
  startedAt?: DateTime;
  pausedAt?: DateTime;
  completedAt?: DateTime;

  // Crash recovery
  lastProcessedAt: DateTime;
  isRecoverable: boolean;
}
```

### 15.3 Auto-Resume on Server Start

```typescript
// Saat server start
async function recoverJobs() {
  // Cari job yang masih running saat crash
  const interruptedJobs = await db.blastJob.findMany({
    where: {
      status: "RUNNING",
      isRecoverable: true,
    },
  });

  for (const job of interruptedJobs) {
    // Update status ke PAUSED (agar user aware)
    await db.blastJob.update({
      where: { id: job.id },
      data: { status: "PAUSED" },
    });

    // Notify user via WebSocket
    socket.emit("blast:recovered", {
      jobId: job.id,
      message: "Job ditemukan setelah restart. Klik Resume untuk melanjutkan.",
    });
  }
}
```

### 15.4 Recovery UI

```
┌─────────────────────────────────────────────────┐
│ ⚠️ Job Ditemukan Setelah Restart                │
├─────────────────────────────────────────────────┤
│ Blast "Promo Akhir Tahun" terinterupsi.         │
│                                                 │
│ Progress: 120/250 (48%)                         │
│ Terakhir aktif: 09:45:30                        │
│                                                 │
│ [▶️ Resume]  [🗑️ Hapus]  [📋 Lihat Detail]      │
└─────────────────────────────────────────────────┘
```

### 15.5 Warmer Job Persistence

Warmer juga disimpan untuk recovery:

```typescript
interface WarmerJobState {
  jobId: string;
  status: "IDLE" | "RUNNING" | "PAUSED" | "COMPLETED";

  // Progress
  currentPairIndex: number;
  sentCount: number;
  allPairs: AccountPair[];

  // Config
  presetId: string;
  linkedBlastJobId?: string; // Jika run bareng blast

  // Recovery
  lastProcessedAt: DateTime;
  isRecoverable: boolean;
}
```

---

## 16. Template Management

### 14.1 Template Schema

| Field           | Type     | Description             |
| --------------- | -------- | ----------------------- |
| `id`            | String   | Template ID             |
| `name`          | String   | Nama template           |
| `messageBlocks` | JSON     | Konfigurasi blok pesan  |
| `variables`     | String[] | Variabel yang digunakan |
| `lastUsedAt`    | DateTime | Terakhir digunakan      |
| `usageCount`    | Int      | Jumlah penggunaan       |

### 14.2 API Endpoints

| Method | Endpoint         | Description         |
| ------ | ---------------- | ------------------- |
| GET    | `/templates`     | List semua template |
| POST   | `/templates`     | Buat template baru  |
| GET    | `/templates/:id` | Detail template     |
| PUT    | `/templates/:id` | Update template     |
| DELETE | `/templates/:id` | Hapus template      |

---

## 15. Success Metrics

- [ ] Wizard selesai dalam < 2 menit
- [ ] 0 error "salah akun"
- [ ] Pause/Resume tidak kehilangan progress
- [ ] Support 10,000+ recipients per job
- [ ] Template dapat digunakan dalam < 3 klik

---

## 18. Integrasi dengan Fitur Lain

### 18.1 Dependency

| Fitur             | Status                                        |
| ----------------- | --------------------------------------------- |
| Client Management | ⬅️ Akun pengirim, daily limit, trust level    |
| Contacts          | ⬅️ Recipients list, skip `hasReceivedMessage` |
| Templates         | ⬅️ Template picker, simpan `templateId`       |
| Warmer            | ⬅️ Preset picker, alternating execution       |
| Settings          | ⬅️ Default delay, warmer config               |
| Number Checker    | ⬅️ Hasil cek → langsung blast                 |

### 18.2 Provides to Other Features

| Consumer      | Data/Service                                      |
| ------------- | ------------------------------------------------- |
| **Analytics** | sentCount, deliveredCount, failedCount, readCount |
| **Templates** | +2 poin saat read, track templateId               |
| **Contacts**  | Update `hasReceivedMessage = true`                |

### 18.3 Integration Checklist

Saat membangun fitur ini, **WAJIB** integrasi:

- [ ] Account picker: `GET /whatsapp/sessions` (filter connected only)
- [ ] Recipients: `GET /contacts` atau direct input
- [ ] Skip sent: Filter `hasReceivedMessage = false`
- [ ] Templates: `GET /templates/top` untuk picker
- [ ] Simpan `templateId` di outbox message
- [ ] Warmer preset: `GET /settings` → `warmerPresets`
- [ ] Jeda mode: Koordinasi dengan Warmer service
- [ ] Setelah kirim: `PATCH /contacts/:id` → `hasReceivedMessage = true`
- [ ] Analytics: Increment counters setiap event
- [ ] Pada read ack: `POST /templates/:id/read` (+2 poin)

### 18.4 Event → Analytics Flow

```typescript
// Blast service
onMessageSent() → analyticsService.increment("sentCount")
onDelivered()   → analyticsService.increment("deliveredCount")
onFailed()      → analyticsService.increment("failedCount")
onRead()        → analyticsService.increment("readCount")
                → templateService.addPoints(templateId, 2)
```

---

**Status:** Menunggu Review
