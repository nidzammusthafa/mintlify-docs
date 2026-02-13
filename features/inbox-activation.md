# Inbox Activation Feature

## Overview
Fitur untuk mengaktifkan/menonaktifkan inbox per akun WhatsApp. Jika dinonaktifkan, pesan masuk **tidak akan disimpan ke database** dan **tidak akan dikirim ke frontend**.

## Use Cases

**Kapan Menonaktifkan Inbox?**

1. **Spam Prevention Account** - Akun khusus untuk blast yang tidak perlu menerima reply
2. **Testing/Development Account** - Akun yang digunakan untuk testing
3. **High Volume Account** - Akun yang menerima banyak pesan tidak penting
4. **Bot-Only Account** - Akun yang hanya digunakan untuk kirim pesan

## Implementation

### Database Schema

**Model:** `WhatsappAccount`

```prisma
model WhatsappAccount {
  // ... existing fields
  
  // Inbox Settings
  inboxEnabled     Boolean  @default(true)  // Enable/disable inbox for this account
  
  // ... rest of fields
}
```

**Migration:** `20260206022616_add_inbox_enabled_setting`

---

### Backend

#### **1. Event Handler Logic**

**File:** `new-server/src/core/whatsapp/services/event-handler.service.ts`

**Location:** Method `handleIncomingMessage()` - Line ~336

**Logic:**
```typescript
const account = await this.prisma.whatsappAccount.findUnique({
  where: { clientName: clientId },
});

// Check if inbox is enabled
if (!account.inboxEnabled) {
  this.logger.debug(
    `[Inbox Disabled] Skipping message save for ${clientId}`
  );

  // Still process auto-reply and AI
  if (!isAutoResponse && !isGroup) {
    const senderNumber = msg.from.replace('@c.us', '').replace('@g.us', '');
    
    const handledByAutoReply = await this.autoReplyService.checkAndRespond(...);
    
    if (!handledByAutoReply) {
      void this.aiAgentService.processMessage(...);
    }
  }

  return; // Skip inbox save and frontend emit
}

// Continue with normal inbox save...
```

#### **2. API Endpoint**

**File:** `new-server/src/core/whatsapp/whatsapp.controller.ts`

```typescript
@Put('session/:name')
async updateSession(
  @Param('name') name: string,
  @Body() body: {
    // ... existing fields
    inboxEnabled?: boolean;  // NEW
  },
) {
  return this.whatsappService.updateAccountSettings(name, body);
}
```

**File:** `new-server/src/core/whatsapp/whatsapp.service.ts`

```typescript
async updateAccountSettings(
  clientName: string,
  data: {
    // ... existing fields
    inboxEnabled?: boolean;  // NEW
  },
) {
  const updated = await this.prisma.whatsappAccount.update({
    where: { clientName },
    data,
  });
  
  return updated;
}
```

---

### Frontend

#### **1. Type Definition**

**File:** `new-client/src/types/account.ts`

```typescript
export interface WhatsappAccount {
  // ... existing fields
  
  // Settings
  autoStart?: boolean;
  syncLimit?: number;
  storageMode?: string;
  headlessMode?: boolean;
  inboxEnabled?: boolean;  // NEW
  isBusiness?: boolean;
}
```

#### **2. Quick Settings Dialog**

**File:** `new-client/src/components/accounts/quick-settings-dialog.tsx`

**Added:**
- State: `inboxEnabled`
- Toggle component untuk "Enable Inbox"
- Placed as first toggle (paling atas)
- Icon: `Inbox` dari lucide-react
- Badge: "Active" / "Disabled"

```typescript
<SettingToggle
  label="Enable Inbox"
  description="Save incoming messages to database and show in inbox"
  checked={inboxEnabled}
  onCheckedChange={setInboxEnabled}
  icon={<Inbox className="h-4 w-4" />}
  badge={inboxEnabled ? "Active" : "Disabled"}
/>
```

#### **3. Visual Indicator**

**File:** `new-client/src/components/accounts/account-card.tsx`

**Added:** Badge "Inbox Off" yang muncul jika `inboxEnabled = false`

```typescript
{account.inboxEnabled === false && (
  <Badge
    variant="outline"
    className="... bg-orange-50 text-orange-700 ..."
  >
    <InboxX className="mr-1 h-3 w-3" />
    Inbox Off
  </Badge>
)}
```

#### **4. API Service**

**File:** `new-client/src/services/whatsapp-api.ts`

```typescript
updateAccountSettings: async (
  sessionName: string,
  settings: {
    // ... existing fields
    inboxEnabled?: boolean;  // NEW
  },
) => {
  return apiFetch(`/whatsapp/session/${sessionName}`, {
    method: "PUT",
    body: JSON.stringify(settings),
  });
},
```

---

## Behavior

### Ketika `inboxEnabled = false`:

**✅ TETAP Berjalan:**
- Event listener aktif
- Spam filter berjalan
- Warmer detection berjalan
- **Auto-reply tetap berfungsi** ✅
- **AI Agent tetap berfungsi** ✅
- Message acknowledgment tracking

**❌ TIDAK Berjalan:**
- ❌ Pesan TIDAK disimpan ke `InboxMessage` table
- ❌ Pesan TIDAK dikirim ke frontend via socket (`inbox:new-message`)
- ❌ Pesan TIDAK muncul di halaman `/inbox`
- ❌ Analytics incoming TIDAK di-track
- ❌ Media TIDAK di-download (karena return sebelum media download)

---

## File Changes

### Backend (3 files)
- ✏️ `new-server/prisma/schema.prisma` - Added `inboxEnabled` field
- ✏️ `new-server/src/core/whatsapp/services/event-handler.service.ts` - Added conditional check
- ✏️ `new-server/src/core/whatsapp/whatsapp.controller.ts` - Added parameter
- ✏️ `new-server/src/core/whatsapp/whatsapp.service.ts` - Added parameter

### Frontend (4 files)
- ✏️ `new-client/src/types/account.ts` - Added field
- ✏️ `new-client/src/components/accounts/quick-settings-dialog.tsx` - Added toggle
- ✏️ `new-client/src/components/accounts/account-card.tsx` - Added badge indicator
- ✏️ `new-client/src/services/whatsapp-api.ts` - Added parameter

### Migration (1 file)
- 🆕 `new-server/prisma/migrations/20260206022616_add_inbox_enabled_setting/migration.sql`

**Total: 8 files changed, 1 migration created**

---

## Testing Guide

### 1. Test Default Behavior (inboxEnabled = true)
- [ ] Create new account
- [ ] Verify `inboxEnabled` defaults to `true` in database
- [ ] Send message to account
- [ ] Verify message appears in inbox
- [ ] Verify message saved in database

### 2. Test Inbox Disabled
- [ ] Open Quick Settings for an account
- [ ] Toggle "Enable Inbox" to OFF
- [ ] Save settings
- [ ] Verify badge "Inbox Off" appears on account card
- [ ] Send message to account
- [ ] Verify message DOES NOT appear in inbox
- [ ] Verify message NOT saved in `InboxMessage` table
- [ ] Verify auto-reply still works (if configured)
- [ ] Verify AI agent still works (if configured)

### 3. Test Toggle ON/OFF
- [ ] Disable inbox
- [ ] Send message → verify not saved
- [ ] Enable inbox again
- [ ] Send message → verify saved
- [ ] Repeat multiple times

### 4. Test Multiple Accounts
- [ ] Set Account A: inbox enabled
- [ ] Set Account B: inbox disabled
- [ ] Send to Account A → should save
- [ ] Send to Account B → should NOT save

### 5. Test Visual Indicators
- [ ] Verify "Inbox Off" badge only shows when disabled
- [ ] Verify toggle state in Quick Settings reflects database value
- [ ] Test dark mode appearance

---

## Migration Command

```bash
cd new-server
npx prisma migrate dev --name add_inbox_enabled_setting
```

---

## API Examples

### Update Inbox Setting

```bash
# Disable inbox
curl -X PUT http://localhost:4000/whatsapp/session/test-account \
  -H "Content-Type: application/json" \
  -d '{"inboxEnabled": false}'

# Enable inbox
curl -X PUT http://localhost:4000/whatsapp/session/test-account \
  -H "Content-Type: application/json" \
  -d '{"inboxEnabled": true}'
```

---

## Important Notes

1. **Default Value**: `true` - Semua akun existing tetap normal
2. **Auto-Reply & AI**: Tetap berfungsi meskipun inbox disabled
3. **Media Download**: Skip jika inbox disabled (menghemat storage)
4. **Warmer**: Tetap skip inbox save untuk warmer messages
5. **Backward Compatible**: No breaking changes untuk existing functionality

---

## Future Enhancements

- [ ] Add inbox disabled icon di header halaman inbox
- [ ] Add filter di inbox page untuk hide disabled accounts
- [ ] Add analytics for messages processed but not saved
- [ ] Add confirmation dialog before disabling inbox
- [ ] Add bulk enable/disable for multiple accounts
