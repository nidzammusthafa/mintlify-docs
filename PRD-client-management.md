# PRD: WhatsApp Client Management

**Version:** 2.0  
**Status:** Draft  
**Date:** 2026-01-25

---

## 1. Overview

Fitur **Client Management** adalah inti dari aplikasi WhatsApp Tools yang memungkinkan pengguna mengelola banyak akun WhatsApp secara bersamaan. Dokumen ini mendefinisikan spesifikasi lengkap untuk implementasi Client Management pada arsitektur baru (NestJS + Next.js).

---

## 2. Goals & Non-Goals

### Goals

- CRUD lengkap untuk akun WhatsApp (Create, Read, Update, Delete)
- Status real-time yang sinkron dengan database
- Kontrol aktivasi manual (Start/Stop) per-akun
- Login via QR Code atau Pairing Code
- Multi-client management dengan batch operations
- Pengaturan per-akun (sync limit, auto-start, trust level)

### Non-Goals

- Integrasi dengan WhatsApp Business API (hanya whatsapp-web.js)
- Enkripsi end-to-end tambahan (sudah bawaan WA)

---

## 3. User Stories

### 3.1 Menambah Akun Baru

> Sebagai pengguna, saya ingin menambah akun WhatsApp baru dengan scan QR code atau pairing code agar saya bisa mengelola akun tersebut.

### 3.2 Lihat Status Akun

> Sebagai pengguna, saya ingin melihat status semua akun (connected, disconnected, loading, qr, error) secara real-time.

### 3.3 Start/Stop Akun

> Sebagai pengguna, saya ingin menyalakan atau mematikan akun secara manual tanpa harus logout.

### 3.4 Edit Akun

> Sebagai pengguna, saya ingin mengganti nama akun, mengatur preferensi sync, dan trust level.

### 3.5 Hapus Akun

> Sebagai pengguna, saya ingin menghapus akun secara permanen beserta semua data terkait.

### 3.6 Logout Akun

> Sebagai pengguna, saya ingin logout tanpa menghapus data session agar bisa login ulang nanti.

### 3.7 Batch Operations

> Sebagai pengguna, saya ingin menyalakan/mematikan/menghapus banyak akun sekaligus.

---

## 4. Functional Requirements

### 4.1 Account CRUD

| Operation        | Endpoint                  | Method | Description                   |
| ---------------- | ------------------------- | ------ | ----------------------------- |
| Create (QR)      | `/whatsapp/session`       | POST   | Buat sesi baru dengan QR code |
| Create (Pairing) | `/whatsapp/session`       | POST   | Buat sesi dengan pairing code |
| Read All         | `/whatsapp/sessions`      | GET    | Dapatkan semua sesi           |
| Read One         | `/whatsapp/session/:name` | GET    | Detail satu sesi              |
| Update           | `/whatsapp/session/:name` | PUT    | Update nama/settings          |
| Delete           | `/whatsapp/session/:name` | DELETE | Hapus permanen                |

### 4.2 Session Control

| Operation    | Endpoint                               | Method | Description                     |
| ------------ | -------------------------------------- | ------ | ------------------------------- |
| Start        | `/whatsapp/session/:name/start`        | POST   | Aktifkan sesi                   |
| Stop         | `/whatsapp/session/:name/stop`         | POST   | Nonaktifkan sesi (tidak logout) |
| Logout       | `/whatsapp/session/:name/logout`       | POST   | Logout dari WhatsApp            |
| Sync History | `/whatsapp/session/:name/sync-history` | POST   | Sinkronisasi pesan lama         |

### 4.3 Batch Operations

| Operation      | Endpoint                      | Method | Description             |
| -------------- | ----------------------------- | ------ | ----------------------- |
| Start Multiple | `/whatsapp/sessions/start`    | POST   | Aktifkan banyak sesi    |
| Stop Multiple  | `/whatsapp/sessions/stop`     | POST   | Nonaktifkan banyak sesi |
| Stop All       | `/whatsapp/sessions/stop-all` | POST   | Nonaktifkan semua sesi  |

### 4.4 Account Settings

| Field         | Type    | Default    | Description                               |
| ------------- | ------- | ---------- | ----------------------------------------- |
| `autoStart`   | Boolean | false      | Auto-start saat server boot               |
| `syncLimit`   | Int     | 50         | Jumlah pesan yang disinkronisasi per chat |
| `storageMode` | Enum    | filesystem | Lokasi penyimpanan media                  |
| `trustLevel`  | Int     | 1          | Trust level untuk rate limiting           |
| `dailyLimit`  | Int     | 50         | Batas pesan harian                        |

---

## 5. Real-time Status

### 5.1 WebSocket Events

**Server → Client:**

```typescript
interface WhatsAppStatusUpdate {
  accountId: string;
  status: AccountStatus; // loading | qr | pairing | authenticated | ready | disconnected | error
  message?: string;
  qrCode?: string; // Base64 QR code
  pairingCode?: string; // 8-digit pairing code
  phoneNumber?: string;
  battery?: number;
}
```

**Event Names:**

- `whatsapp-status` - Status update per-akun
- `whatsapp-all-statuses` - Broadcast semua status (on connect)

### 5.2 Status Flow

```
IDLE → loading → qr/pairing → authenticated → ready ⇄ disconnected
                    ↓                             ↓
                  error ←─────────────────────────┘
```

### 5.3 Database Sync

Setiap perubahan status WAJIB:

1. Di-emit via WebSocket ke semua connected clients
2. Disimpan ke database (`connectionStatus` field)

---

## 6. UI Requirements

### 6.1 Account Card

- Avatar (dari WA atau initial)
- Nama akun & nomor telepon
- Status badge dengan warna
- Tombol Start/Stop (contextual)
- Dropdown menu: Open Chat, Disconnect, Settings, Delete
- Battery & last active (jika connected)
- Trust level indicator

### 6.2 Add Account Dialog

- Mode toggle: QR Code vs Pairing Code
- QR code display dengan countdown timer
- Pairing code display dengan copy button
- Input nama akun
- Settings awal (syncLimit)

### 6.3 Account Settings Modal

- Nama akun (editable)
- Auto-start toggle
- Sync limit slider (0-500)
- Storage mode selector
- Trust level display
- Daily limit display
- Reset message count button

### 6.4 Batch Actions Bar

- Muncul saat ada akun yang diselect
- Tombol: Start Selected, Stop Selected, Delete Selected
- Counter selected items

---

## 7. Phone Number Pairing (Login Tanpa QR)

Fitur alternatif untuk login WhatsApp tanpa scan QR Code.

### 7.1 Flow Penggunaan

1. User memilih opsi "Link with Phone Number" di Add Account dialog
2. User memasukkan nomor HP (format `628...`)
3. Server request pairing code ke WhatsApp via `whatsapp-web.js`
4. Server emit kode pairing (e.g. `ABC-123-XYZ`) ke UI
5. User masukkan kode di HP (Menu "Link Device" > "Link with phone number instead")

### 7.2 WebSocket Event

| Event                   | Direction       | Payload               |
| ----------------------- | --------------- | --------------------- |
| `whatsapp-pairing-code` | Server → Client | `{ accountId, code }` |

### 7.3 Use Cases

- Kamera HP rusak
- Server remote (tidak bisa scan QR di layar)
- Menggunakan emulator Android

---

## 8. Technical Specifications

### 8.1 Database Schema

```prisma
model WhatsappAccount {
  id               String   @id @default(uuid())
  clientName       String   @unique
  phoneNumber      String?
  isLoggedIn       Boolean  @default(false)
  connectionStatus String   @default("DISCONNECTED")

  // Configuration
  autoStart        Boolean  @default(false)
  syncLimit        Int      @default(50)
  storageMode      String   @default("filesystem")

  // Trust Level
  trustLevel       Int      @default(1)
  dailyLimit       Int      @default(50)
  messagesSentToday Int     @default(0)

  // Timestamps
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt
  lastActiveAt     DateTime?
}
```

### 8.2 Backend Services

- `WhatsappService` - Facade service
- `ClientManagerService` - Client lifecycle management
- `EventHandlerService` - WhatsApp event handlers
- `TrustLevelService` - Rate limiting logic

### 8.3 Frontend Stores

- `whatsapp-store.ts` - Account state management
- Real-time sync via `WhatsappListener` component

---

## 9. Migration from Old System

| Old Feature                 | New Implementation                |
| --------------------------- | --------------------------------- |
| `initializeClient`          | `createClient` + start on demand  |
| `initializeMultipleClients` | Batch start endpoint              |
| `disconnectClient`          | `stopClient` (preserve session)   |
| `logoutClient`              | `deleteClient` (logout + cleanup) |
| `renameClient`              | Not in v2.0 (update via PUT)      |
| `setClientAsMain`           | Removed (not used)                |
| `setWhatsAppAccountRating`  | `updateTrustLevel`                |
| `resetMessageCount`         | Built into daily cron job         |

---

## 10. Success Metrics

- [ ] Semua status terupdate dalam < 500ms
- [ ] UI reflect database state 100% akurat
- [ ] Start/Stop latency < 2 detik
- [ ] Support minimal 10 concurrent clients
- [ ] Memory footprint < 500MB per client

---

## 11. Open Questions

1. Apakah perlu fitur "Set as Main Client" untuk prioritas tertentu?
2. Haruskah ada limit maksimal client per user?
3. Perlu fitur export/import session?

---

## 12. Integrasi dengan Fitur Lain

### 12.1 Dependency

| Fitur    | Status                                |
| -------- | ------------------------------------- |
| Settings | ⬅️ Reads default config dari Settings |

### 12.2 Provides to Other Features

| Consumer           | Data/Service                                |
| ------------------ | ------------------------------------------- |
| **Semua Fitur**    | Account list, status, phoneNumber           |
| **Blast**          | Akun pengirim, daily limit, trust level     |
| **Warmer**         | Akun pairs untuk warming                    |
| **Inbox**          | Multi-account filter, sender identification |
| **Number Checker** | Akun untuk pengecekan                       |
| **Analytics**      | Per-account metrics                         |

### 12.3 Integration Checklist

Saat membangun fitur lain, **WAJIB** integrasi:

- [ ] Get account list: `GET /whatsapp/sessions`
- [ ] Check account status sebelum kirim pesan
- [ ] Subscribe `whatsapp-status` event untuk real-time updates
- [ ] Validate `dailyLimit` sebelum blast
- [ ] Update `messagesSentToday` setelah kirim pesan

---

**Status:** Menunggu Review
