# PRD: Message Analytics

**Version:** 2.0  
**Status:** Draft  
**Date:** 2026-01-25

---

## 1. Overview

Fitur **Analytics** menyediakan visualisasi data performa pengiriman pesan dengan periode harian, mingguan, dan bulanan. Fokus: **Dashboard yang mudah dibaca, insight actionable, per-akun tracking**.

---

## 2. Goals

- ✅ Lihat performa semua akun dalam satu dashboard
- ✅ Drill-down ke analytics per-akun
- ✅ Periode fleksibel: harian, mingguan, bulanan
- ✅ Visualisasi chart yang mudah dipahami
- ✅ Export data untuk analisis lanjutan

---

## 3. Dashboard Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 📊 Analytics Dashboard                    Periode: [Mingguan ▼] [📅 Custom] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │ 📤 Terkirim │  │ ✅ Delivered│  │ ❌ Gagal    │  │ 📥 Diterima │        │
│  │   1,234     │  │   1,180     │  │     54      │  │    892      │        │
│  │    ↑ 12%    │  │    ↑ 15%    │  │    ↓ 8%     │  │    ↑ 23%    │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      📈 Trend Pengiriman                            │   │
│  │   ^                                                                 │   │
│  │ 200│     ╭──╮                                                       │   │
│  │    │    ╱    ╲    ╭──────╮                                          │   │
│  │ 100│   ╱      ╲──╯      ╲    ╭─                                     │   │
│  │    │  ╱                  ╲──╯                                       │   │
│  │   0└──────────────────────────────────>                             │   │
│  │      Sen  Sel  Rab  Kam  Jum  Sab  Min                              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────┐  ┌──────────────────────────────┐        │
│  │  🥧 Distribusi Status        │  │  📊 Per-Account Performance  │        │
│  │                              │  │                              │        │
│  │     ╭────╮                   │  │  Toko A  ████████░░ 80%      │        │
│  │    ╱ 95% ╲ Delivered         │  │  Toko B  ██████████ 95%      │        │
│  │   │      │                   │  │  Personal████░░░░░░ 45%      │        │
│  │    ╲ 5% ╱  Failed            │  │                              │        │
│  │     ╰────╯                   │  │                              │        │
│  └──────────────────────────────┘  └──────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Metrics Summary Cards

| Card            | Metric           | Description                  |
| --------------- | ---------------- | ---------------------------- |
| 📤 Terkirim     | `sentCount`      | Total pesan dikirim          |
| ✅ Delivered    | `deliveredCount` | Pesan sampai ke penerima     |
| ❌ Gagal        | `failedCount`    | Pesan gagal terkirim         |
| 📥 Diterima     | `receivedCount`  | Pesan masuk (inbox)          |
| 📈 Success Rate | `successRate`    | % Delivered / Sent           |
| ↑↓ Perubahan    | `changePercent`  | Dibanding periode sebelumnya |

---

## 5. Periode Analisis

### 5.1 Tabs

```
[📅 Harian] [📆 Mingguan] [🗓️ Bulanan] [⚙️ Custom]
```

### 5.2 Data Points per Periode

| Periode  | Range             | Data Points          |
| -------- | ----------------- | -------------------- |
| Harian   | 7 hari terakhir   | Per hari (7 titik)   |
| Mingguan | 4 minggu terakhir | Per minggu (4 titik) |
| Bulanan  | 12 bulan terakhir | Per bulan (12 titik) |
| Custom   | User pilih        | Flexible             |

---

## 6. Per-Account Analytics

### 6.1 Account Selector

```
┌─────────────────────────────────────────────────┐
│ 📊 Analytics Per Akun                           │
├─────────────────────────────────────────────────┤
│ Pilih Akun: [🔵 Toko A ▼]                       │
│                                                 │
│ Atau bandingkan multiple akun:                  │
│ [✓] 🔵 Toko A  [✓] 🟢 Toko B  [ ] 🟣 Personal   │
└─────────────────────────────────────────────────┘
```

### 6.2 Per-Account Metrics

| Metric      | Description                  |
| ----------- | ---------------------------- |
| Sent        | Pesan terkirim dari akun ini |
| Delivered   | Pesan delivered              |
| Failed      | Pesan gagal                  |
| Received    | Pesan diterima akun ini      |
| Read Rate   | % pesan dibaca penerima      |
| Reply Rate  | % pesan dibalas              |
| Trust Level | NEWCOMER → VETERAN           |
| Daily Usage | Usage vs daily limit         |

### 6.3 Account Comparison Chart

```
┌─────────────────────────────────────────────────────────────────┐
│ 📊 Perbandingan Akun (Sent)                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 200 |  ████                                                     │
│     |  ████  ████                                               │
│ 150 |  ████  ████                                               │
│     |  ████  ████  ████                                         │
│ 100 |  ████  ████  ████                                         │
│     |  ████  ████  ████  ████                                   │
│  50 |  ████  ████  ████  ████                                   │
│     |  ████  ████  ████  ████                                   │
│   0 └──────────────────────────                                 │
│      Toko A Toko B Personal                                     │
│                                                                 │
│ Legend: ████ Sent  ░░░░ Delivered  ▓▓▓▓ Failed                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Charts

### 7.1 Chart Types

| Type        | Use Case                         |
| ----------- | -------------------------------- |
| Line Chart  | Trend over time                  |
| Bar Chart   | Comparison between accounts      |
| Pie Chart   | Distribution (Success vs Failed) |
| Stacked Bar | Multiple metrics per period      |

### 7.2 Interactive Features

- Hover untuk detail
- Click untuk drill-down
- Zoom pada time range
- Toggle series visibility

---

## 8. Data Model

### 8.1 Analytics Entry

```typescript
interface AnalyticsEntry {
  id: string;
  accountId: string;
  period: "DAILY" | "WEEKLY" | "MONTHLY";
  date: Date; // Start of period

  // Outbound metrics
  sentCount: number;
  deliveredCount: number;
  failedCount: number;
  readCount: number;
  repliedCount: number;

  // Inbound metrics
  receivedCount: number;

  // Calculated
  successRate: number; // deliveredCount / sentCount
  readRate: number; // readCount / deliveredCount
  replyRate: number; // repliedCount / deliveredCount
}
```

### 8.2 Aggregation

Data di-aggregate setiap:

- Jam (untuk harian)
- Hari (untuk mingguan)
- Minggu (untuk bulanan)

---

## 9. API Endpoints

| Method | Endpoint                             | Description               |
| ------ | ------------------------------------ | ------------------------- |
| GET    | `/analytics/summary`                 | Summary cards             |
| GET    | `/analytics/daily`                   | Stats harian (7 hari)     |
| GET    | `/analytics/weekly`                  | Stats mingguan (4 minggu) |
| GET    | `/analytics/monthly`                 | Stats bulanan (12 bulan)  |
| GET    | `/analytics/custom?from=&to=`        | Custom range              |
| GET    | `/analytics/account/:id`             | Per-account stats         |
| GET    | `/analytics/comparison?accounts=a,b` | Multi-account comparison  |
| POST   | `/analytics/reset`                   | Reset semua data          |
| POST   | `/analytics/reset/:accountId`        | Reset per akun            |
| GET    | `/analytics/export`                  | Export ke Excel           |

---

## 10. Export

### 10.1 Format

- **Excel (.xlsx)** - Default
- **CSV (.csv)** - Simple

### 10.2 Columns

| Column       | Description       |
| ------------ | ----------------- |
| Date         | Tanggal/periode   |
| Account      | Nama akun         |
| Sent         | Jumlah terkirim   |
| Delivered    | Jumlah delivered  |
| Failed       | Jumlah gagal      |
| Success Rate | Persentase sukses |

---

## 11. Quick Access

### 11.1 Dashboard Widget

```
┌───────────────────────────┐
│ 📊 Ringkasan Hari Ini     │
├───────────────────────────┤
│ 📤 Sent: 245              │
│ ✅ Rate: 95%              │
│ 📥 Inbox: 32 new          │
│                           │
│ [Lihat Detail →]          │
└───────────────────────────┘
```

### 11.2 Sidebar Entry

- Icon: 📊
- Label: "Analytics"
- Badge: (none)

---

## 12. Success Metrics

- [ ] Dashboard load < 2 detik
- [ ] Chart render < 500ms
- [ ] Data akurat 100%
- [ ] Export < 5 detik untuk 12 bulan data

---

## 13. Integrasi dengan Fitur Lain

### 13.1 Dependency

| Fitur             | Status                                    |
| ----------------- | ----------------------------------------- |
| Client Management | ⬅️ Per-account metrics                    |
| Inbox             | ⬅️ receivedCount, replyCount              |
| Blast             | ⬅️ sentCount, deliveredCount, failedCount |
| Templates         | ⬅️ Top performing templates               |

### 13.2 Provides to Other Features

| Consumer      | Data/Service           |
| ------------- | ---------------------- |
| **Dashboard** | Summary widget         |
| **Settings**  | Reset analytics option |

### 13.3 Integration Checklist

Saat membangun fitur lain, **WAJIB** update analytics:

- [ ] Blast: Setiap pesan terkirim → increment `sentCount`
- [ ] Blast: Setiap delivered → increment `deliveredCount`
- [ ] Blast: Setiap failed → increment `failedCount`
- [ ] Blast: Setiap read ack → increment `readCount`
- [ ] Inbox: Setiap pesan masuk → increment `receivedCount`
- [ ] Inbox: Setiap reply → increment `repliedCount`
- [ ] Templates: Fetch `GET /templates/top` untuk widget

### 13.4 Analytics Update Pattern

```typescript
// Setiap event, panggil service
analyticsService.increment(accountId, "sentCount");
analyticsService.increment(accountId, "deliveredCount");
// dst...
```

---

**Status:** Menunggu Review
