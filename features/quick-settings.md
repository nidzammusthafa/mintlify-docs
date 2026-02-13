# Quick Settings Feature

## Overview
Fitur Quick Settings memungkinkan user untuk mengubah pengaturan akun WhatsApp secara cepat langsung dari halaman `/accounts` tanpa perlu berpindah halaman.

## Fitur yang Tersedia

### Toggle Settings
1. **Headless Mode** - Menjalankan browser dalam mode tersembunyi
2. **Auto Start** - Memulai session otomatis saat server restart
3. **AI Chatbot** - Mengaktifkan/menonaktifkan AI auto-reply

### Number Settings
1. **Sync History Limit** - Jumlah pesan yang di-sync saat koneksi (0-500 messages)
2. **Daily Blast Limit** - Batas maksimal pesan blast per hari (10-1000 messages)

## Cara Menggunakan

1. Di halaman `/accounts`, klik icon **⋮** (three dots) pada account card
2. Pilih **"Quick Settings"** dari dropdown menu
3. Ubah pengaturan sesuai kebutuhan
4. Klik **"Save Changes"**

## File Structure

### Backend
```
new-server/src/core/
├── whatsapp/
│   ├── whatsapp.controller.ts    # Endpoint: PUT /whatsapp/session/:name
│   └── whatsapp.service.ts       # updateAccountSettings()
└── ai-agent/
    ├── ai-agent.controller.ts    # Endpoint: PATCH /ai-agent/:accountId/toggle
    └── ai-agent.service.ts
```

### Frontend
```
new-client/src/
├── components/accounts/
│   ├── quick-settings-dialog.tsx      # Dialog utama
│   ├── setting-toggle.tsx             # Toggle switch component
│   ├── setting-number-input.tsx       # Number input component
│   └── account-card.tsx               # Integrasi Quick Settings button
└── services/
    ├── whatsapp-api.ts                # updateAccountSettings()
    └── ai-agent-api.ts                # toggleAiAgent()
```

## API Endpoints

### Update WhatsApp Settings
```typescript
PUT /whatsapp/session/:name
Body: {
  autoStart?: boolean;
  headlessMode?: boolean;
  syncLimit?: number;
  dailyLimit?: number;
}
```

### Toggle AI Agent
```typescript
PATCH /ai-agent/:accountId/toggle
Body: {
  isActive: boolean
}
```

## Component Props

### QuickSettingsDialog
```typescript
interface QuickSettingsDialogProps {
  isOpen: boolean;
  onOpenChange: (open: boolean) => void;
  account: WhatsappAccount;
  onUpdate: () => void;  // Callback setelah save berhasil
}
```

### SettingToggle
```typescript
interface SettingToggleProps {
  label: string;
  description: string;
  checked: boolean;
  onCheckedChange: (checked: boolean) => void;
  icon?: React.ReactNode;
  badge?: string;
  disabled?: boolean;
}
```

### SettingNumberInput
```typescript
interface SettingNumberInputProps {
  label: string;
  description: string;
  value: number;
  onChange: (value: number) => void;
  min: number;
  max: number;
  unit: string;
  icon?: React.ReactNode;
  disabled?: boolean;
}
```

## Features

- ✅ Auto-fetch AI agent status saat dialog dibuka
- ✅ Loading state untuk AI toggle
- ✅ Input validation (min/max values)
- ✅ Toast notifications untuk success/error
- ✅ Link ke Advanced Settings
- ✅ Responsive design
- ✅ Disabled state untuk settings yang tidak bisa diubah
- ✅ Icons untuk setiap setting

## Error Handling

1. **AI Agent Not Found**: Jika AI agent belum dibuat, toggle tetap bisa digunakan tapi akan menampilkan warning
2. **API Errors**: Error akan ditampilkan via toast notification
3. **Invalid Input**: Input number akan di-validate sesuai min/max

## Future Enhancements

- [ ] Add preset buttons untuk daily limit (Safe: 50, Moderate: 100, Aggressive: 200)
- [ ] Add confirmation dialog untuk perubahan yang sensitive
- [ ] Add keyboard shortcuts (ESC to close, Ctrl+S to save)
- [ ] Optimistic updates untuk better UX
- [ ] Batch update untuk multiple accounts

## Testing Checklist

- [ ] Test semua toggle switches
- [ ] Test input validation (min/max)
- [ ] Test AI enable/disable
- [ ] Test dengan AI agent yang belum dibuat
- [ ] Test error handling
- [ ] Test responsive design
- [ ] Test link ke Advanced Settings
- [ ] Test save & refresh data

## Notes

- Quick Settings hanya mengubah pengaturan yang sering diakses
- Untuk pengaturan advanced (storage mode, etc.), gunakan halaman `/accounts/[id]/settings`
- AI Agent harus sudah dikonfigurasi terlebih dahulu untuk bisa di-toggle
