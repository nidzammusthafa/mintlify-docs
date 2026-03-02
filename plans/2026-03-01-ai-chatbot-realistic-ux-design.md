# AI Chatbot — Realistic Message UX Design (v2)

**Tanggal**: 2026-03-01  
**Status**: Draft — Menunggu Persetujuan

## Latar Belakang

Saat AI chatbot membalas pesan WhatsApp, respons langsung terkirim tanpa simulasi perilaku manusia. Ini membuat pengirim merasa berinteraksi dengan bot.

## Tujuan

1. **Read Receipt Instan** — Centang biru langsung saat pesan diterima
2. **Delay Membaca** — Jeda proporsional setelah read receipt
3. **Typing Indicator** — "Sedang mengetik..." selama AI memproses
4. **Toggle Setting** — Bisa diaktifkan/nonaktifkan di AI Studio

## Formula Delay

```
readingDelay = clamp(800 + wordCount × 250, 800, 8000) ms
```

~250 kata/menit — sedikit lebih cepat dari manusia (~200 wpm).

## Alur

1. Pesan masuk → buffer (5s, sudah ada)
2. **Send seen** (centang biru) — instan
3. **Wait** `readingDelay` ms — simulasi membaca
4. **Send typing** — "sedang mengetik..."
5. AI generate response
6. **Clear state** → Kirim pesan balasan

## File yang Dimodifikasi

| File                        | Perubahan              |
| --------------------------- | ---------------------- |
| `schema.prisma`             | +field `simulateHuman` |
| `client-manager.service.ts` | +2 method baru         |
| `whatsapp.service.ts`       | +2 facade method       |
| `ai-agent.service.ts`       | +delay + typing logic  |
| `ai-response.handler.ts`    | +typing untuk flow     |
| `ai-agent-api.ts`           | +field interface       |
| `ai-settings.tsx`           | +toggle Switch UI      |
