# Desktop App Improvements — SPARC Plan

> **Phase**: Specification → Architecture  
> **Date**: 2026-03-01

## 1. Specification (Tujuan)

Meningkatkan fitur dan pengalaman pengguna desktop launcher (Electron) WazapSuite:

| #   | Fitur                             | Deskripsi                                                                                                       |
| --- | --------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| F1  | **Dark Mode Icon Fix**            | Icon toggle dark mode saat dark mode tidak terlihat — perlu diperbaiki kontras/visibilitas SVG icon             |
| F2  | **Restart & Stop Server Buttons** | Tombol Restart dan Stop server sudah ada di dashboard, verifikasi dan improve UX-nya                            |
| F3  | **Hide to System Tray**           | Tombol baru untuk menyembunyikan window ke system tray (kanan bawah taskbar), bisa dibuka kembali via tray icon |
| F4  | **Build Guide (EXE)**             | Dokumentasi build yang benar agar output .exe tidak error (Prisma, Puppeteer, externalized node_modules)        |

## 2. Pseudocode (Alur Logika)

### F1 — Dark Mode Icon Fix

**Root Cause Analysis:**

- Button `.theme-toggle` di `index.html` memiliki 2 SVG icon: `#theme-icon-sun` dan `#theme-icon-moon`
- `updateThemeIcon(isDark)` toggle `display: block/none` pada masing-masing
- **Tapi**: CSS class `.hidden` didefinisikan sebagai `display: none !important` — ini menimpa inline `style.display`
- Saat halaman load dalam dark mode, `#theme-icon-sun` memiliki class `hidden` di HTML → `!important` menang

**Fix:**

```
// Di updateThemeIcon():
// Ganti style.display = "block/none" → toggle class "hidden" sebagai gantinya
function updateThemeIcon(isDark) {
  sun.classList.toggle("hidden", !isDark);   // Sun visible saat dark mode
  moon.classList.toggle("hidden", isDark);   // Moon visible saat light mode
}
```

---

### F2 — Restart & Stop Server

**Status**: Sudah diimplementasikan ✅

Buttons sudah ada di `index.html` (line 260–271) dan event handlers di script (line 1130–1135):

- `btn-start-server` → `wazap.server.start()`
- `btn-stop-server` → `wazap.server.stop()`
- `btn-restart-server` → stop → delay 1s → start

IPC handlers sudah registered di `ipc/index.ts` (line 41–58).

**Optional Improvement**: Tambah konfirmasi dialog sebelum Stop/Restart saat server sedang running, dan tambah visual feedback saat proses restart (loading spinner on button).

---

### F3 — Hide to System Tray

```
// Di main/index.ts:
1. Import Tray, Menu, nativeImage dari electron
2. Buat tray icon saat app.ready
3. Pasang context menu: [Show, Restart Server, Stop Server, Quit]
4. Override window close:
   - if (!isQuitting): event.preventDefault() → mainWindow.hide()
   - Tray tetap ada di taskbar
5. Tray double-click: mainWindow.show() + mainWindow.focus()
6. Real quit: set isQuitting=true lalu app.quit()

// Di preload: tambah window:hide API
// Di renderer: tambah button "Hide to Tray" di window controls
```

---

### F4 — Build Guide (Referensi Teknis)

Build pipeline saat ini:

```
[new-server] npm run build:sea
  ↓ esbuild bundle → dist/main.js
  ↓ Node SEA compile → dist/server.exe
  ↓ npm install --omit=dev → dist/node_modules/
    (Prisma, Puppeteer, sharp, whatsapp-web.js — EXTERNALIZED)

[desktop] npm run dist
  ↓ scripts/prepare.js:
    ↓ Copy server.exe + .env + prisma/ + node_modules/ + cloudflared → temp/
    ↓ Zip → resources/server.zip
  ↓ electron-builder → release/WazapSuite.Setup.X.Y.Z.exe
```

**Critical Considerations:**

1. **Prisma**: Engine binaries MUST be in `node_modules/@prisma/engines/`. Schema migrations handled via raw SQL in `ServerManager.ts`, NOT `prisma migrate deploy`
2. **Puppeteer/Chrome**: NOT bundled in SEA. App uses external Chrome via `chrome-manager.ts` (auto-download atau user path)
3. **better-sqlite3**: Native module, needs `electron-builder install-app-deps` for correct Electron ABI
4. **ASAR**: Enabled, with `asarUnpack: ["**/*.node"]` for native modules
5. **Version naming**: Output MUST be renamed from hyphens to dots for auto-updater (`WazapSuite.Setup.X.Y.Z.exe`)

## 3. Architecture (Proposed Changes)

### Komponen: Desktop Main Process

#### [MODIFY] [index.ts](file:///e:/Projects/whatsapp-tools/desktop/src/main/index.ts)

- Import `Tray`, `Menu`, `nativeImage` dari Electron
- Buat `tray` instance dengan icon app saat `app.ready`
- Context menu: Show App, Separator, Restart Server, Stop Server, Separator, Quit
- Override close behavior:
  - `mainWindow.on('close')`: jika `!isQuitting` → `event.preventDefault()` + `mainWindow.hide()`
  - Tray double-click → `mainWindow.show()` + `mainWindow.focus()`
- Add tray cleanup on quit

#### [MODIFY] [ipc/index.ts](file:///e:/Projects/whatsapp-tools/desktop/src/main/ipc/index.ts)

- Tambah handler `window:hide` → hide window ke tray (tanpa close)

#### [MODIFY] [index.ts (preload)](file:///e:/Projects/whatsapp-tools/desktop/src/preload/index.ts)

- Expose `window.hide` via contextBridge

---

### Komponen: Desktop Renderer

#### [MODIFY] [index.html](file:///e:/Projects/whatsapp-tools/desktop/src/renderer/index.html)

1. **F1 — Dark Mode Icon Fix**:
   - Fix `updateThemeIcon()` function → gunakan `classList.toggle("hidden")` bukan `style.display`

2. **F3 — Hide to Tray Button**:
   - Tambah button baru di `.window-controls` (antara theme toggle dan minimize):
     ```html
     <button
       class="control-btn"
       id="btn-hide"
       title="Hide to tray"
       aria-label="Hide to tray"
     >
       <!-- Down/tray arrow icon SVG -->
     </button>
     ```
   - Bind ke `window.wazap.window.hide()`

---

### Komponen: CSS

#### [MODIFY] [main.css](file:///e:/Projects/whatsapp-tools/desktop/src/renderer/styles/main.css)

- Tidak perlu perubahan besar — `.dark .theme-toggle` sudah memiliki styling yang benar (`color: #ffffff !important`, `border-color: #666666 !important`). Root cause ada di JavaScript.

---

## 4. Ringkasan Perubahan

| File                  | Perubahan                                            |
| --------------------- | ---------------------------------------------------- |
| `main/index.ts`       | +Tray icon, override close → hide, tray context menu |
| `main/ipc/index.ts`   | +1 IPC handler `window:hide`                         |
| `preload/index.ts`    | +1 API `window.hide`                                 |
| `renderer/index.html` | Fix dark mode icon toggle, +Hide to Tray button      |

## 5. Verification Plan

### Manual Testing

1. **Dark Mode Icon**: Toggle theme → verifikasi icon terlihat jelas di kedua mode
2. **Hide to Tray**: Klik hide → window hilang → icon muncul di system tray → double-click → window kembali
3. **Tray Context Menu**: Klik kanan tray icon → Show/Restart/Stop/Quit semua berfungsi
4. **Window Close vs Hide**: Klik ✕ → window hide (bukan quit). Quit hanya dari tray context menu
5. **Server Restart/Stop**: Verifikasi tombol existing masih bekerja normal

### Build Testing

1. `npm run build` di desktop → verifikasi TypeScript compile sukses
2. `npm run dev` → test launch + semua fitur
3. (Optional) `npm run dist` → full build test `.exe`
