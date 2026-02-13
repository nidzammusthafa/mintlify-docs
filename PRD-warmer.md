# PRD: Account Warmer

**Version:** 2.0  
**Status:** Draft  
**Date:** 2026-01-25

---

## 1. Overview

Fitur **Account Warmer** meningkatkan trust score akun WhatsApp dengan simulasi percakapan natural antar akun internal. Fokus: **Mudah digunakan, quick setup, auto-pair generation**.

---

## 2. Goals

- ✅ Tingkatkan trust level akun baru
- ✅ Hindari spam detection
- ✅ Percakapan terlihat natural
- ✅ Setup dalam < 1 menit
- ✅ Progress real-time

---

## 3. Quick Setup (2-Step)

### Step 1: Pilih Akun

```
┌─────────────────────────────────────────────────┐
│ 🔥 Account Warmer - Step 1/2                    │
├─────────────────────────────────────────────────┤
│ Pilih akun untuk warming (min 2):               │
│                                                 │
│ [✓] 🔵 Toko A (Trust: NEWCOMER)                 │
│ [✓] 🟢 Toko B (Trust: NEWCOMER)                 │
│ [✓] 🟣 Personal (Trust: VERIFIED)               │
│ [ ] 🟠 Bisnis C (Trust: TRUSTED) ← tidak perlu  │
│                                                 │
│ 3 akun dipilih → 6 pasangan percakapan          │
│                                                 │
│                              [Selanjutnya →]    │
└─────────────────────────────────────────────────┘
```

### Step 2: Konfigurasi & Mulai

```
┌─────────────────────────────────────────────────┐
│ 🔥 Account Warmer - Step 2/2                    │
├─────────────────────────────────────────────────┤
│ Pengaturan:                                     │
│                                                 │
│ Preset: [🚶 Normal (Recommended) ▼]             │
│                                                 │
│ ⚙️ Detail (opsional):                           │
│ ├── Jumlah pesan per pasangan: 5-10             │
│ ├── Delay antar pesan: 10-30 detik              │
│ └── Bahasa pesan: [Bahasa Indonesia ▼]          │
│                                                 │
│ ─────────────────────────────────────────────── │
│ 📊 Ringkasan:                                   │
│ • Pasangan: A↔B, A↔C, B↔C (6 arah)              │
│ • Total pesan: ~45 pesan                        │
│ • Estimasi waktu: ~15 menit                     │
│                                                 │
│ [← Kembali]               [🔥 Mulai Warming]    │
└─────────────────────────────────────────────────┘
```

---

## 4. Pair Generation Algorithm

### 4.1 Auto-Pair

Sistem otomatis membuat semua kombinasi pasangan:

```
Input: [A, B, C]

Output Pairs:
  A → B
  A → C
  B → A
  B → C
  C → A
  C → B
```

### 4.2 Round-Robin Execution

```
Pair 1: A → B (5 pesan)
Pair 2: A → C (5 pesan)
Pair 3: B → A (5 pesan)
...dst
```

---

## 5. Delay Configuration

| Preset    | Min Delay | Max Delay | Msg/Pair | Use Case         |
| --------- | --------- | --------- | -------- | ---------------- |
| 🐢 Lambat | 30s       | 60s       | 3-5      | Akun sangat baru |
| 🚶 Normal | 10s       | 30s       | 5-10     | Default          |
| 🏃 Cepat  | 5s        | 15s       | 10-15    | Akun verified    |

---

## 6. Message Templates

### 6.1 Bahasa Indonesia

```
[
  "Halo, apa kabar?",
  "Baik, kamu gimana?",
  "Alhamdulillah baik juga",
  "Lagi sibuk apa?",
  "Ini lagi santai aja",
  "Oke deh, nanti ngobrol lagi ya",
  "Siap, sampai nanti!"
]
```

### 6.2 English

```
[
  "Hey, how are you?",
  "I'm good, and you?",
  "Great thanks!",
  "What are you up to?",
  "Just chilling",
  "Cool, talk later!",
  "Sure, bye!"
]
```

---

## 7. Progress Dashboard

```
┌─────────────────────────────────────────────────┐
│ 🔥 Warmer: Session 24-Jan                       │
├─────────────────────────────────────────────────┤
│ Status: 🟢 RUNNING                              │
│                                                 │
│ [████████░░░░░░░░░░░░] 40% (18/45 pesan)        │
│                                                 │
│ Pasangan aktif: A → C                           │
│ Pesan terakhir: "Lagi sibuk apa?"               │
│                                                 │
│ Log:                                            │
│ 10:30 A → B: "Halo, apa kabar?"                 │
│ 10:31 B → A: "Baik, kamu gimana?"               │
│ 10:32 A → B: "Alhamdulillah baik juga"          │
│ ...                                             │
│                                                 │
│ [⏸️ Pause]  [⏹️ Stop]                           │
└─────────────────────────────────────────────────┘
```

---

## 8. Job Status

| Status    | Description     |
| --------- | --------------- |
| IDLE      | Belum mulai     |
| RUNNING   | Sedang berjalan |
| PAUSED    | Dijeda          |
| COMPLETED | Selesai         |
| ERROR     | Gagal           |

---

## 9. Blast Integration

Warmer dapat dijalankan paralel dengan Blast:

```typescript
blastConfig: {
  enableWhatsappWarmer: true,
  whatsappWarmerMinMessages: 3,
  whatsappWarmerMaxMessages: 5,
  whatsappWarmerDelayMs: 30000,  // Setiap 30 detik
  whatsappWarmerLanguage: "id"
}
```

---

## 10. API Endpoints

| Method | Endpoint                | Description    |
| ------ | ----------------------- | -------------- |
| GET    | `/warmer`               | List semua job |
| POST   | `/warmer`               | Buat job baru  |
| GET    | `/warmer/:jobId`        | Detail job     |
| POST   | `/warmer/:jobId/start`  | Mulai          |
| POST   | `/warmer/:jobId/pause`  | Pause          |
| POST   | `/warmer/:jobId/resume` | Resume         |
| POST   | `/warmer/:jobId/stop`   | Stop           |
| DELETE | `/warmer/:jobId`        | Hapus job      |

---

## 11. WebSocket Events

| Event                 | Direction       | Payload                               |
| --------------------- | --------------- | ------------------------------------- |
| `warmer:progress`     | Server → Client | `{ jobId, sent, total, currentPair }` |
| `warmer:message-sent` | Server → Client | `{ from, to, message }`               |
| `warmer:completed`    | Server → Client | `{ jobId, summary }`                  |

---

## 12. Success Metrics

- [ ] Setup dalam < 1 menit
- [ ] 0 error akun tertukar
- [ ] Trust level naik setelah warming
- [ ] Tidak terdeteksi sebagai spam

---

## 13. Integrasi dengan Fitur Lain

### 13.1 Dependency

| Fitur             | Status                             |
| ----------------- | ---------------------------------- |
| Client Management | ⬅️ Akun list untuk pair generation |
| Settings          | ⬅️ Warmer presets config           |

### 13.2 Provides to Other Features

| Consumer     | Data/Service                                |
| ------------ | ------------------------------------------- |
| **Blast**    | Warmer preset picker, alternating execution |
| **Settings** | Warmer preset management                    |

### 13.3 Integration Checklist

Saat membangun Blast, **WAJIB** integrasi:

- [ ] Blast: Pilih warmer preset dari `GET /settings` → `warmerPresets`
- [ ] Blast: Jeda mode "dengan warmer" → trigger warmer job
- [ ] Blast: `linkedBlastJobId` untuk koordinasi pause/resume
- [ ] Settings: CRUD warmer presets

### 13.4 Koordinasi dengan Blast

```typescript
// Blast koordinasi dengan Warmer
if (blastConfig.pauseMode === "WARMER") {
  // Setiap N penerima
  blastService.pause();
  await warmerService.runSession(presetId, linkedBlastJobId);
  blastService.resume();
}
```

---

**Status:** Menunggu Review
