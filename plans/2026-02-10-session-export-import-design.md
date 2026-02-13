# Design: Session Export/Import (Backup & Restore)

**Date**: 2026-02-10
**Status**: Ready for Implementation
**Estimated Effort**: ~4 jam (backend only)

---

## 1. Overview

Fitur untuk mengexport dan mengimport sesi client WhatsApp sebagai file ZIP. Tujuan utama: **Backup & Restore** -- jika sesi hilang atau corrupt, user bisa merestore dari backup tanpa scan QR ulang.

## 2. Keputusan Desain

| Keputusan | Pilihan | Alasan |
|-----------|---------|--------|
| Scope data | Session files + WhatsappAccount only | 80% value ada di session credential; konfigurasi bisa disetup ulang |
| Use case | Backup & Restore | Fokus utama user |
| Duplikat handling | Overwrite (update, bukan delete+create) | Mempertahankan relasi existing (messages, templates, AI) |
| Post-import | Manual connect | User memilih kapan mau konek |
| Keamanan ZIP | Tanpa enkripsi | Simpel, user bertanggung jawab atas file |
| phoneNumber saat import | Set null | Menghindari unique constraint conflict |

## 3. API Endpoints

### Export Session
```
GET /whatsapp/session/:name/export

Response: ZIP file download
Headers:
  Content-Type: application/zip
  Content-Disposition: attachment; filename="<name>-backup-<timestamp>.zip"
```

### Import Session
```
POST /whatsapp/session/import
Content-Type: multipart/form-data
Body: file=<ZIP>

Response 201:
{
  "success": true,
  "message": "Session imported successfully",
  "clientName": "akun-1",
  "phoneNumber": null,
  "overwritten": false,
  "note": "Use POST /whatsapp/session/akun-1/start to connect"
}
```

## 4. ZIP Structure

```
akun-1-backup-20260210T120000.zip
├── manifest.json          # Metadata (version, timestamp, platform)
├── account.json           # WhatsappAccount fields (tanpa relasi)
└── session-akun-1/        # Chromium profile data (auth credentials)
    ├── Default/
    │   ├── IndexedDB/
    │   ├── Local Storage/
    │   └── ...
    └── ...
```

## 5. Arsitektur

### File Baru
- `new-server/src/core/whatsapp/services/session-backup.service.ts`

### File Dimodifikasi
- `new-server/src/core/whatsapp/whatsapp.controller.ts` (2 endpoints)
- `new-server/src/core/whatsapp/whatsapp.service.ts` (2 facade methods)
- `new-server/src/core/whatsapp/whatsapp.module.ts` (register service)

### Dependencies Baru
- `archiver` + `@types/archiver` -- ZIP creation (streaming)
- `adm-zip` + `@types/adm-zip` -- ZIP extraction

### Diagram
```
Controller ──> WhatsappService (Facade) ──> SessionBackupService
                                              ├── PrismaService (DB read/write)
                                              ├── ClientManagerService (stop/start client)
                                              └── archiver/adm-zip (ZIP ops)
```

## 6. Flow Details

### Export Flow
1. Validasi akun ada di DB dan session folder ada
2. **Stop client** jika sedang aktif (menghindari file lock)
3. Baca data WhatsappAccount dari DB
4. Buat ZIP streaming: manifest.json + account.json + session folder
5. Pipe ZIP ke HTTP response
6. **Restart client** jika sebelumnya aktif

### Import Flow
1. Validasi file ZIP (format, ukuran max 500MB)
2. Extract ke temp directory (OS tmpdir)
3. Validasi isi: manifest.json, account.json, session folder
4. Jika clientName sudah ada:
   - Stop client aktif
   - Hapus session folder lama
   - **Update** record (bukan delete+create, agar relasi terjaga)
5. Jika clientName belum ada: create record baru
6. phoneNumber di-set **null** (terisi otomatis saat connect)
7. Copy session folder ke `.wwebjs_auth/`
8. Cleanup temp directory

## 7. Edge Cases & Mitigasi

| Edge Case | Mitigasi |
|-----------|----------|
| phoneNumber conflict (unique constraint) | Set null saat import |
| Overwrite cascade delete relasi | Gunakan `update`, bukan `delete+create` |
| Session folder besar (>500MB) | Streaming ZIP + disk-based multer |
| Windows file locking | Wajib stopClient() sebelum operasi |
| ZIP path traversal attack | Library adm-zip sudah sanitize + validasi manual |
| Sesi expired di WhatsApp server | Dokumentasikan bahwa backup = credential lokal saja |
| Disk penuh | Error handling + cleanup di finally block |

## 8. Acceptance Criteria

- [ ] Export menghasilkan ZIP valid dengan manifest, account data, dan session files
- [ ] Import berhasil restore sesi -- setelah start, akun langsung login
- [ ] Overwrite akun existing tanpa kehilangan messages/templates/AI config
- [ ] Error handling jelas untuk semua kasus gagal
- [ ] Temp files selalu dibersihkan (success maupun error)
- [ ] Client di-stop sebelum export/import, di-restart setelah export jika sebelumnya aktif

## 9. Future Enhancements (Out of Scope v1)

- Export relasi (auto-reply rules, templates, AI agents)
- Frontend UI (tombol export/import di dashboard)
- Multi-session export dalam satu ZIP
- WebSocket progress events untuk operasi besar
- Enkripsi/password protection
- Schedule automatic backup

---

*Dokumen SPARC lengkap tersedia di `.sparc/session-export-import/`*
