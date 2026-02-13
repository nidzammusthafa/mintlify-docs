# Rencana Implementasi ROADMAP

> **Tanggal**: 8 Februari 2026  
> **Total Tasks**: 17 items  
> **Estimasi Total**: 8-12 hari kerja

---

## Ringkasan Eksekutif

Dokumen ini berisi perencanaan detail untuk menyelesaikan semua item di ROADMAP.md. Setiap task dikategorikan berdasarkan prioritas, estimasi waktu, dan skill yang relevan untuk eksekusi.

---

## Kategorisasi Tasks

### 📊 Overview by Category

| Kategori                 | Jumlah  | Estimasi |
| ------------------------ | ------- | -------- |
| 🔴 Frontend (new-client) | 8 tasks | 3-4 hari |
| 🟢 Backend (new-server)  | 5 tasks | 2-3 hari |
| 🟣 Electron/Desktop      | 4 tasks | 2-3 hari |
| 🔵 Full Stack            | 3 tasks | 1-2 hari |

---

## Phase 1: Critical Bugs & Core Features

**Estimasi: 3-4 hari | Prioritas: TINGGI**

### Task 1.1: Loading Sync Message WhatsApp

> **Lokasi**: `new-client/` & `new-server/`  
> **Tipe**: Full Stack  
> **Estimasi**: 4-6 jam

**Deskripsi:**
Tampilkan loading sync message saat akun WhatsApp pertama login di bagian card. Contoh: "Memuat 10 pesan... 150 pesan... Selesai sinkronisasi"

**Skills yang Digunakan:**

- `token-efficient-development` - Akses folder yang tepat
- `nestjs-real-time` - WebSocket untuk real-time sync status
- `frontend-tailwind-architect` - Styling komponen log

**Implementasi:**

```
Backend (new-server):
├── src/whatsapp/whatsapp.gateway.ts
│   └── Emit event 'sync:progress' dengan { count, status }
└── src/whatsapp/whatsapp.service.ts
    └── Hook ke event 'message_sync' dari whatsapp-web.js

Frontend (new-client):
├── src/components/accounts/account-card.tsx
│   └── Tambah SyncStatusLog component
└── src/hooks/use-sync-status.ts
    └── Listen WebSocket 'sync:progress'
```

**Acceptance Criteria:**

- [ ] Log muncul di bawah card saat sync berlangsung
- [ ] Log hilang setelah sync selesai (dengan animasi fade)
- [ ] Tidak merusak layout card existing

---

### Task 1.2: Warmer Jobs Data Tidak Muncul/Hilang Langsung

> **Lokasi**: `new-client/`  
> **Tipe**: Frontend  
> **Estimasi**: 2-3 jam

**Deskripsi:**
Data warmer jobs tidak muncul langsung saat dibuat dan tidak segera hilang saat dihapus.

**Skills yang Digunakan:**

- `token-efficient-development` - Fokus new-client only
- `frontend-tailwind-architect` - Optimistic UI update

**Root Cause Analysis:**
Kemungkinan issue di React Query cache invalidation atau optimistic update.

**Implementasi:**

```
Frontend (new-client):
├── src/hooks/use-warmer-jobs.ts
│   └── Tambah optimistic update pada mutation
├── src/components/warmer/warmer-table.tsx
│   └── Pastikan refetch setelah create/delete
└── src/lib/query-client.ts
    └── Konfigurasi staleTime & refetchOnMount
```

**Acceptance Criteria:**

- [ ] Job muncul instant setelah create (optimistic)
- [ ] Job hilang instant setelah delete (optimistic)
- [ ] Rollback jika API gagal

---

### Task 1.3: Perbaiki Tabel Warmer

> **Lokasi**: `new-client/`  
> **Tipe**: Frontend  
> **Estimasi**: 1-2 jam

**Deskripsi:**
Jangan tampilkan ID user, cukup nomor telepon pengirim/penerima dengan format `+62 812-3456-7890`, serta tampilkan pesan yang dikirim.

**Skills yang Digunakan:**

- `token-efficient-development` - Fokus new-client only
- `frontend-tailwind-architect` - Tabel styling

**Implementasi:**

```
Frontend (new-client):
├── src/components/warmer/warmer-columns.tsx
│   ├── Hapus kolom userId
│   ├── Format nomor dengan libphonenumber-js
│   └── Tambah kolom message (truncate 50 chars)
└── src/lib/format-phone.ts
    └── Utility function format nomor Indonesia
```

**Acceptance Criteria:**

- [ ] Kolom: Pengirim | Penerima | Pesan | Status | Waktu
- [ ] Format nomor: +62 812-3456-7890
- [ ] Pesan ditruncate dengan ellipsis

---

### Task 1.4: Schedule Warmer Tidak Berjalan

> **Lokasi**: `new-server/`  
> **Tipe**: Backend  
> **Estimasi**: 3-4 jam

**Deskripsi:**
Fitur schedule warmer belum berjalan sesuai jadwal yang ditentukan.

**Skills yang Digunakan:**

- `token-efficient-development` - Fokus new-server only
- `nestjs-real-time` - Cron jobs & scheduler

**Root Cause Analysis:**
Kemungkinan issue di:

1. Cron expression tidak valid
2. Scheduler service tidak di-inject
3. TimeZone mismatch

**Implementasi:**

```
Backend (new-server):
├── src/scheduler/scheduler.module.ts
│   └── Pastikan SchedulerModule registered
├── src/warmer/warmer.scheduler.ts
│   ├── @Cron decorator dengan timezone
│   └── Logic execute warmer jobs
└── src/warmer/warmer.service.ts
    └── Method executeScheduledJobs()
```

**Testing:**

```bash
# Set schedule 1 menit dari sekarang
# Pastikan job berjalan
# Cek logs di terminal
```

**Acceptance Criteria:**

- [ ] Schedule warmer berjalan sesuai waktu yang ditentukan
- [ ] Timezone Indonesia (Asia/Jakarta)
- [ ] Log execution di console

---

## Phase 2: AI Chatbot Enhancements

**Estimasi: 2-3 hari | Prioritas: TINGGI**

### Task 2.1: Fitur Blok Nomor dari Inbox

> **Lokasi**: `new-client/` & `new-server/`  
> **Tipe**: Full Stack  
> **Estimasi**: 3-4 jam

**Deskripsi:**
Tambahkan kemampuan untuk memblokir nomor langsung dari inbox.

**Skills yang Digunakan:**

- `token-efficient-development` - Akses kedua folder
- `frontend-tailwind-architect` - UI button & modal
- `error-handling-patterns` - Handle block errors

**Implementasi:**

```
Backend (new-server):
├── src/contacts/contacts.controller.ts
│   └── POST /contacts/:id/block
├── src/contacts/contacts.service.ts
│   └── blockNumber(phoneNumber)
└── src/whatsapp/whatsapp.service.ts
    └── client.blockContact(contactId)

Frontend (new-client):
├── src/components/inbox/chat-header.tsx
│   └── Tambah button "Block" dengan icon
├── src/components/inbox/block-dialog.tsx
│   └── Konfirmasi dialog
└── src/hooks/use-block-contact.ts
    └── Mutation untuk block API
```

**Acceptance Criteria:**

- [ ] Button block di header chat
- [ ] Konfirmasi dialog sebelum block
- [ ] Nomor yang diblock tidak bisa kirim pesan
- [ ] List blocked contacts di Settings

---

### Task 2.2: Matikan AI Chatbot untuk Nomor Tertentu

> **Lokasi**: `new-client/` & `new-server/`  
> **Tipe**: Full Stack  
> **Estimasi**: 4-5 jam

**Deskripsi:**
Simpan toggle matikan AI chatbot di bagian atas inbox untuk nomor tertentu.

**Skills yang Digunakan:**

- `token-efficient-development` - Akses kedua folder
- `frontend-tailwind-architect` - Toggle switch UI
- `nestjs-real-time` - Sync state real-time

**Implementasi:**

```
Backend (new-server):
├── src/ai/ai-settings.entity.ts
│   └── { phoneNumber, aiEnabled: boolean }
├── src/ai/ai.controller.ts
│   └── PATCH /ai/settings/:phoneNumber
└── src/ai/ai.service.ts
    └── isAiEnabledForNumber(phone): boolean

Frontend (new-client):
├── src/components/inbox/ai-toggle.tsx
│   └── Switch component di header inbox
├── src/hooks/use-ai-settings.ts
│   └── Query & mutation AI settings
└── src/components/inbox/inbox-header.tsx
    └── Integrate AiToggle component
```

**Database Schema:**

```sql
CREATE TABLE ai_settings (
  id INTEGER PRIMARY KEY,
  phone_number TEXT UNIQUE,
  ai_enabled BOOLEAN DEFAULT true,
  disabled_at DATETIME,
  disabled_by TEXT
);
```

**Acceptance Criteria:**

- [ ] Toggle switch di atas inbox per chat
- [ ] State tersimpan di database
- [ ] AI tidak reply jika disabled
- [ ] Visual indicator (badge "AI Off")

---

### Task 2.3: Matikan AI Jika Ada Pesan Baru dari Admin

> **Lokasi**: `new-server/`  
> **Tipe**: Backend  
> **Estimasi**: 3-4 jam

**Deskripsi:**
AI chatbot otomatis nonaktif jika admin mengirim pesan manual (sebelum AI menjawab).

**Skills yang Digunakan:**

- `token-efficient-development` - Fokus new-server only
- `error-handling-patterns` - Handle race conditions

**Logic Flow:**

```
1. Pesan masuk → Queue ke AI processing
2. Cek: Ada pesan outgoing dari admin dalam window 30 detik?
   - Ya → Skip AI response, tandai "admin takeover"
   - Tidak → Lanjut AI processing
3. Setelah admin reply → Disable AI untuk chat ini (30 menit)
```

**Implementasi:**

```
Backend (new-server):
├── src/ai/ai.service.ts
│   ├── checkAdminTakeover(chatId): boolean
│   └── setAdminTakeover(chatId, duration)
├── src/whatsapp/whatsapp.gateway.ts
│   └── Listen 'message_create' untuk outgoing
└── src/ai/ai.guard.ts
    └── Guard untuk skip AI jika admin takeover
```

**Acceptance Criteria:**

- [ ] AI tidak reply jika admin sudah reply dalam 30 detik
- [ ] Admin takeover berlaku 30 menit
- [ ] Visual indicator di inbox "Admin Mode"
- [ ] Auto-resume AI setelah timeout

---

### Task 2.4: AI Delay Response (Batch Messages)

> **Lokasi**: `new-server/`  
> **Tipe**: Backend  
> **Estimasi**: 4-5 jam

**Deskripsi:**
AI tidak langsung menjawab, tapi menunggu beberapa detik untuk mengumpulkan pesan beruntun dari user yang suka kirim pesan panjang dalam beberapa kali pengiriman.

**Skills yang Digunakan:**

- `token-efficient-development` - Fokus new-server only
- `nestjs-real-time` - Debounce & queue system
- `error-handling-patterns` - Handle timeout edge cases

**Logic Flow:**

```
1. Pesan masuk → Simpan ke buffer dengan timestamp
2. Start timer 5 detik (configurable)
3. Jika pesan baru masuk sebelum timer habis:
   - Reset timer
   - Append ke buffer
4. Timer habis → Gabung semua pesan → Proses AI
5. Kirim 1 respons untuk semua pesan
```

**Implementasi:**

```
Backend (new-server):
├── src/ai/message-buffer.service.ts
│   ├── addToBuffer(chatId, message)
│   ├── getBufferedMessages(chatId): Message[]
│   └── clearBuffer(chatId)
├── src/ai/ai.processor.ts
│   └── Debounced processing dengan configurable delay
└── src/settings/ai-settings.entity.ts
    └── responseDelaySeconds: number (default 5)
```

**Configuration:**

```typescript
// Default settings
const AI_RESPONSE_DELAY = 5000; // 5 detik
const AI_MAX_BUFFER_TIME = 30000; // Max 30 detik tunggu
const AI_MAX_BUFFER_MESSAGES = 10; // Max 10 pesan
```

**Acceptance Criteria:**

- [ ] AI menunggu 5 detik setelah pesan terakhir
- [ ] Pesan beruntun digabung jadi 1 context
- [ ] Configurable delay di settings
- [ ] Max wait time 30 detik (force process)

---

## Phase 3: Desktop App (Electron)

**Estimasi: 2-3 hari | Prioritas: TINGGI**

### Task 3.1: Auto Update Aplikasi Desktop

> **Lokasi**: Electron main process  
> **Tipe**: Electron  
> **Estimasi**: 4-6 jam

**Deskripsi:**
Implementasi auto-update menggunakan electron-updater dengan repository releases di `https://github.com/nidzammusthafa/wazap-suite-releases`

**Skills yang Digunakan:**

- `electron-secure-build` - Secure update mechanism
- `error-handling-patterns` - Handle update failures

**Implementasi:**

```
Electron:
├── src/main/updater.ts
│   ├── checkForUpdates()
│   ├── downloadUpdate()
│   └── installUpdate()
├── src/main/main.ts
│   └── Initialize updater on app ready
└── electron-builder.yml
    └── publish config untuk GitHub releases
```

**electron-builder.yml:**

```yaml
publish:
  provider: github
  owner: nidzammusthafa
  repo: wazap-suite-releases
  releaseType: release

# Auto update settings
autoUpdate:
  checkAtStartup: true
  checkInterval: 3600000 # 1 jam
```

**Update Flow:**

```
1. App start → Check update (background)
2. Update available → Show notification
3. User confirm → Download in background
4. Download complete → Prompt restart
5. Restart → Install update → Launch new version
```

**Acceptance Criteria:**

- [ ] Check update saat app start
- [ ] Notification jika ada update
- [ ] Download progress indicator
- [ ] Seamless install & restart
- [ ] Rollback jika update gagal

---

### Task 3.2: Preserve Data Saat Update

> **Lokasi**: Electron main process  
> **Tipe**: Electron  
> **Estimasi**: 3-4 jam

**Deskripsi:**
Memastikan data SQLite dan sesi WhatsApp tidak terhapus saat update aplikasi.

**Skills yang Digunakan:**

- `electron-secure-build` - Secure data storage
- `error-handling-patterns` - Backup & restore

**Current Data Locations:**

```
Windows:
├── %APPDATA%/wazap-suite/
│   ├── database.sqlite     # SQLite database
│   ├── sessions/           # WhatsApp sessions
│   └── config.json         # App settings
```

**Implementasi:**

```
Electron:
├── src/main/data-manager.ts
│   ├── getDataPath(): string
│   ├── backupData(): Promise<void>
│   └── restoreData(): Promise<void>
├── src/main/updater.ts
│   └── Before update → Backup data
└── electron-builder.yml
    └── nsis.perMachine: false (user data)
```

**NSIS Config (Windows Installer):**

```yaml
nsis:
  oneClick: false
  perMachine: false # Install di user folder
  allowToChangeInstallationDirectory: false
  deleteAppDataOnUninstall: false # PENTING!
```

**Acceptance Criteria:**

- [ ] Data SQLite preserved setelah update
- [ ] Sesi WhatsApp tetap aktif
- [ ] Backup otomatis sebelum update
- [ ] Restore otomatis jika update gagal

---

### Task 3.3: Copy/Export Logs & Error Reporting

> **Lokasi**: Electron main process & `new-client/`  
> **Tipe**: Full Stack  
> **Estimasi**: 3-4 jam

**Deskripsi:**
Tambahkan fitur copy/export logs dan perencanaan pengiriman logs error ke developer.

**Skills yang Digunakan:**

- `electron-secure-build` - Secure log handling
- `error-handling-patterns` - Error collection
- `frontend-tailwind-architect` - Logs viewer UI

**Implementasi:**

```
Electron:
├── src/main/logger.ts
│   ├── getLogPath(): string
│   ├── exportLogs(): Promise<string>
│   └── sanitizeLogs(): string (hapus secrets)
├── src/main/ipc-handlers.ts
│   └── 'logs:export', 'logs:open-folder'
└── src/preload/preload.ts
    └── Expose logs API

Frontend (new-client):
├── src/components/settings/logs-section.tsx
│   ├── Button "Open Logs Folder"
│   ├── Button "Copy Logs"
│   └── Button "Report Bug" (planned)
└── src/hooks/use-logs.ts
    └── IPC calls untuk logs
```

**Log Sanitization:**

```typescript
// Hapus data sensitif sebelum export
const SENSITIVE_PATTERNS = [
  /apiKey['":\s]+['"][^'"]+['"]/gi,
  /password['":\s]+['"][^'"]+['"]/gi,
  /token['":\s]+['"][^'"]+['"]/gi,
  /session['":\s]+['"][^'"]+['"]/gi,
];
```

**Acceptance Criteria:**

- [ ] Button "Open Logs" di Settings
- [ ] Copy logs ke clipboard (sanitized)
- [ ] Logs folder langsung terbuka
- [ ] Placeholder untuk "Report Bug" feature

---

### Task 3.4: Audit Console Logs (Security)

> **Lokasi**: `new-server/`  
> **Tipe**: Backend  
> **Estimasi**: 2-3 jam

**Deskripsi:**
Periksa semua console.log/logger dan pastikan tidak mengekspos data rahasia.

**Skills yang Digunakan:**

- `electron-secure-build` - Security audit
- `token-efficient-development` - Fokus new-server

**Audit Checklist:**

```
[ ] API Keys tidak di-log
[ ] Session tokens tidak di-log
[ ] Password/credentials tidak di-log
[ ] Phone numbers di-mask (628xxx...xxx)
[ ] Message content di-truncate (max 50 chars)
```

**Implementasi:**

```
Backend (new-server):
├── src/common/logger.service.ts
│   ├── sanitize(data): any
│   └── maskPhoneNumber(phone): string
├── src/common/logger.interceptor.ts
│   └── Intercept all logs & sanitize
└── Audit semua file:
    └── grep -r "console.log\|logger\." src/
```

**Masking Rules:**

```typescript
// Phone: 6281234567890 → 628xxx...890
// Token: abc123def456 → abc***456
// Message: "Halo ini pesan panjang..." → "Halo ini pe..."
```

**Acceptance Criteria:**

- [ ] Audit report semua console.log
- [ ] Sensitive data di-mask
- [ ] Production logger level (warn, error only)
- [ ] No raw credentials in logs

---

## Phase 4: UI/UX Improvements

**Estimasi: 1-2 hari | Prioritas: MEDIUM**

### Task 4.1: Tech Stack Info di Desktop App

> **Lokasi**: `new-client/`  
> **Tipe**: Frontend  
> **Estimasi**: 1-2 jam

**Deskripsi:**
Tambahkan informasi tech stack server di aplikasi desktop (About section).

**Skills yang Digunakan:**

- `frontend-tailwind-architect` - About page styling

**Implementasi:**

```
Frontend (new-client):
├── src/components/settings/about-section.tsx
│   └── Tambah Server Info card
└── src/app/settings/page.tsx
    └── Import AboutSection
```

**Content:**

```
Server Tech Stack:
├── Runtime: Node.js v20.x
├── Framework: NestJS 10.x
├── Database: SQLite 3.x
├── WhatsApp: whatsapp-web.js 1.x
├── Real-time: Socket.IO 4.x
└── AI: Google Gemini / OpenAI
```

**Acceptance Criteria:**

- [ ] About section di Settings
- [ ] Server version info
- [ ] Build date/version

---

### Task 4.2: Textarea Max Height (Maps Scraper)

> **Lokasi**: `new-client/`  
> **Tipe**: Frontend  
> **Estimasi**: 30 menit

**Deskripsi:**
Tambahkan tinggi maksimal pada textarea di Maps Scraper baru.

**Skills yang Digunakan:**

- `frontend-tailwind-architect` - Styling constraint

**Implementasi:**

```tsx
// src/components/scraper/query-textarea.tsx
<Textarea
  className="min-h-[100px] max-h-[300px] overflow-y-auto resize-y"
  placeholder="Masukkan query..."
/>
```

**Acceptance Criteria:**

- [ ] Max height 300px
- [ ] Scrollable jika content overflow
- [ ] Resizable (drag corner)

---

### Task 4.3: Auto-send Hasil Scrape

> **Lokasi**: `new-client/` & `new-server/`  
> **Tipe**: Full Stack  
> **Estimasi**: 2-3 jam

**Deskripsi:**
Fitur pengiriman hasil scrape otomatis setelah satu query berhasil didapatkan.

**Skills yang Digunakan:**

- `token-efficient-development` - Akses kedua folder
- `frontend-tailwind-architect` - Toggle UI

**Implementasi:**

```
Backend (new-server):
├── src/scraper/scraper.service.ts
│   └── autoSaveToContacts option
└── src/scraper/scraper.controller.ts
    └── POST /scraper/jobs { autoSave: boolean }

Frontend (new-client):
├── src/components/scraper/scraper-form.tsx
│   └── Toggle "Auto-save to Contacts"
└── src/hooks/use-scraper.ts
    └── Include autoSave param
```

**Acceptance Criteria:**

- [ ] Toggle "Auto-save results"
- [ ] Hasil langsung masuk Contacts
- [ ] Notification setelah save

---

### Task 4.4: Hapus Komponen Live Feed

> **Lokasi**: `new-client/`  
> **Tipe**: Frontend  
> **Estimasi**: 30 menit

**Deskripsi:**
Hapus komponen live feed yang tidak digunakan.

**Skills yang Digunakan:**

- `token-efficient-development` - Cari dan hapus

**Implementasi:**

```bash
# Cari file live-feed
grep -r "live-feed\|LiveFeed" new-client/src/

# Hapus file & import
rm src/components/live-feed.tsx
# Update imports di parent components
```

**Acceptance Criteria:**

- [ ] Komponen LiveFeed dihapus
- [ ] Tidak ada import error
- [ ] Build sukses

---

### Task 4.5: Sticky Header & Scrollbar Tabel Scraper

> **Lokasi**: `new-client/`  
> **Tipe**: Frontend  
> **Estimasi**: 1-2 jam

**Deskripsi:**
Tingkatkan tabel scraper agar header sticky dan scrollbar horizontal selalu visible.

**Skills yang Digunakan:**

- `frontend-tailwind-architect` - Advanced table styling

**Implementasi:**

```tsx
// src/components/scraper/results-table.tsx
<div className="relative overflow-auto max-h-[600px]">
  <Table>
    <TableHeader className="sticky top-0 z-10 bg-background shadow-sm">
      {/* Headers */}
    </TableHeader>
    <TableBody>
      {/* Rows */}
    </TableBody>
  </Table>
</div>

// Sticky horizontal scrollbar
<div className="sticky bottom-0 overflow-x-auto bg-background">
  <div style={{ width: tableWidth }} className="h-4" />
</div>
```

**CSS Additions:**

```css
/* Sticky scrollbar trick */
.table-container {
  overflow: auto;
  max-height: 600px;
}

.table-container::-webkit-scrollbar {
  height: 12px;
  position: sticky;
  bottom: 0;
}
```

**Acceptance Criteria:**

- [ ] Header sticky saat scroll vertikal
- [ ] Scrollbar horizontal selalu visible
- [ ] Smooth scroll experience

---

## Timeline Eksekusi

### Minggu 1 (Hari 1-5)

| Hari | Tasks                                           | Estimasi |
| ---- | ----------------------------------------------- | -------- |
| 1    | Task 1.1 (Sync Loading) + Task 1.2 (Warmer Bug) | 6-8 jam  |
| 2    | Task 1.3 (Tabel Warmer) + Task 1.4 (Schedule)   | 5-6 jam  |
| 3    | Task 2.1 (Block) + Task 2.2 (AI Toggle)         | 7-9 jam  |
| 4    | Task 2.3 (Admin Takeover) + Task 2.4 (AI Delay) | 7-9 jam  |
| 5    | Task 3.1 (Auto Update)                          | 4-6 jam  |

### Minggu 2 (Hari 6-8)

| Hari | Tasks                                      | Estimasi |
| ---- | ------------------------------------------ | -------- |
| 6    | Task 3.2 (Preserve Data) + Task 3.3 (Logs) | 6-8 jam  |
| 7    | Task 3.4 (Audit) + Task 4.1-4.2            | 4-5 jam  |
| 8    | Task 4.3-4.5 + Testing + Buffer            | 4-5 jam  |

---

## Risk Assessment

### High Risk

| Task          | Risk             | Mitigation                            |
| ------------- | ---------------- | ------------------------------------- |
| Auto Update   | Breaking changes | Test di staging, rollback mechanism   |
| Preserve Data | Data loss        | Backup sebelum update, restore script |
| AI Delay      | Race conditions  | Mutex lock, proper queue              |

### Medium Risk

| Task            | Risk            | Mitigation                             |
| --------------- | --------------- | -------------------------------------- |
| Schedule Warmer | Timezone issues | Use moment-timezone, test Asia/Jakarta |
| Admin Takeover  | Edge cases      | Clear timeout logic, state machine     |

### Low Risk

| Task               | Risk           | Mitigation                 |
| ------------------ | -------------- | -------------------------- |
| UI Tasks           | Minor styling  | Component testing          |
| Tabel improvements | Browser compat | Test Chrome, Firefox, Edge |

---

## Testing Strategy

### Unit Tests

```
[ ] AI buffer service
[ ] Phone number formatting
[ ] Log sanitization
[ ] Update checker
```

### Integration Tests

```
[ ] WebSocket sync progress
[ ] AI response flow
[ ] Auto-update download
```

### E2E Tests

```
[ ] Full warmer flow
[ ] Block contact flow
[ ] Scraper to contacts flow
```

---

## Post-Implementation

### Documentation

- [ ] Update API docs untuk endpoint baru
- [ ] Update user guide untuk fitur baru
- [ ] Release notes untuk version baru

### Monitoring

- [ ] Track AI response times
- [ ] Monitor update success rate
- [ ] Log error frequency

---

## Appendix: Skill Mapping

| Task            | Primary Skill                 | Secondary Skills              |
| --------------- | ----------------------------- | ----------------------------- |
| Sync Loading    | `nestjs-real-time`            | `frontend-tailwind-architect` |
| Warmer Bugs     | `token-efficient-development` | -                             |
| Schedule Warmer | `nestjs-real-time`            | `error-handling-patterns`     |
| Block Nomor     | `token-efficient-development` | `frontend-tailwind-architect` |
| AI Toggle       | `frontend-tailwind-architect` | `nestjs-real-time`            |
| Admin Takeover  | `error-handling-patterns`     | `nestjs-real-time`            |
| AI Delay        | `nestjs-real-time`            | `error-handling-patterns`     |
| Auto Update     | `electron-secure-build`       | `error-handling-patterns`     |
| Preserve Data   | `electron-secure-build`       | -                             |
| Logs Export     | `electron-secure-build`       | `frontend-tailwind-architect` |
| Audit Logs      | `electron-secure-build`       | `token-efficient-development` |
| UI Improvements | `frontend-tailwind-architect` | -                             |

---

> **Catatan**: Perencanaan ini bersifat estimasi. Waktu aktual dapat bervariasi berdasarkan kompleksitas yang ditemukan selama implementasi.
