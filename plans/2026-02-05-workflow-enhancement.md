# Workflow Feature Enhancement: HTTP Request & Function Nodes

## Goal

Membuat fitur workflow dapat berjalan maksimal untuk chatbot AI sales dengan kemampuan:

1. **HTTP Request Node** - Fetch ke API eksternal (cek coverage lokasi WiFi)
2. **Code/Function Node** - Menjalankan logika custom JavaScript
3. **Delay Node** - Jeda sebelum mengirim pesan berikutnya

## Use Case: Ahmad Marketing Chatbot

```mermaid
flowchart TD
  T[Trigger: keyword "wifi", "internet"] --> ASK["AI: Minta lokasi user"]
  ASK --> WAIT["Waiting for user response"]
  WAIT --> FETCH["HTTP Request: Cek Coverage API"]
  FETCH --> COND{"Condition: response.covered == true?"}
  COND -->|Yes| COVERED["AI: Kirim daftar paket + promo"]
  COND -->|No| NOT_COVERED["Static: Mohon maaf belum tercover..."]
  NOT_COVERED --> CROSS["AI: Cross-sell website/Wazap"]
```

## Current State Analysis

### ✅ Node Types Already Implemented (`ai-agent.service.ts`)

| Node Type           | Status | Notes                                   |
| ------------------- | ------ | --------------------------------------- |
| `trigger`           | ✅     | Keyword matching works                  |
| `static-message`    | ✅     | Sends fixed message                     |
| `ai-response`       | ✅     | Uses LLM to generate reply              |
| `condition`         | ✅     | `contains:keyword` or `exact:keyword`   |
| `switch`            | ✅     | Multi-branch based on message content   |
| `intent-classifier` | ✅     | Uses LLM for intent detection           |
| `variable`          | ⚠️     | Only regex extraction from user message |
| `confirmation`      | ✅     | Yes/No branching                        |

### ❌ Node Types NOT YET Implemented

| Node Type           | Priority  | Description                                    |
| ------------------- | --------- | ---------------------------------------------- |
| `http-request`      | 🔴 HIGH   | Fetch external API, store response in variable |
| `code` / `function` | 🟡 MEDIUM | Run JavaScript logic, transform data           |
| `delay`             | 🟢 LOW    | Wait N seconds before next node                |
| `entity-extractor`  | 🟢 LOW    | Extract named entities using LLM               |
| `sentiment`         | 🟢 LOW    | Detect sentiment using LLM                     |
| `loop`              | 🟢 LOW    | Iterate over array data                        |

---

## Proposed Changes

### 1. Backend: HTTP Request Node Handler

#### [MODIFY] [ai-agent.service.ts](file:///e:/Projects/whatsapp-tools/new-server/src/core/ai-agent/ai-agent.service.ts)

Add handler for `http-request` node type inside `executeFlow` method (after line 494):

```typescript
} else if (currentNode.type === 'http-request') {
  const url = currentNode.data.url as string;
  const method = (currentNode.data.method as string) || 'GET';
  const headers = (currentNode.data.headers as Record<string, string>) || {};
  const body = currentNode.data.body as string;
  const responseVariable = (currentNode.data.responseVariable as string) || 'httpResponse';

  // Substitute variables in URL
  let finalUrl = url;
  let vars: Record<string, string> = {};
  try {
    if (conversation.variables) vars = JSON.parse(conversation.variables);
    Object.entries(vars).forEach(([key, val]) => {
      finalUrl = finalUrl.replace(`{{${key}}}`, val);
    });
  } catch {}

  try {
    const response = await fetch(finalUrl, {
      method,
      headers: { 'Content-Type': 'application/json', ...headers },
      body: method !== 'GET' ? body : undefined,
    });
    const data = await response.json();

    // Store response in conversation variables
    vars[responseVariable] = JSON.stringify(data);
    await this.prisma.aiConversation.update({
      where: { id: conversation.id },
      data: { variables: JSON.stringify(vars) },
    });
  } catch (err) {
    this.logger.error(`HTTP Request failed: ${String(err)}`);
    vars[responseVariable] = JSON.stringify({ error: String(err) });
    await this.prisma.aiConversation.update({
      where: { id: conversation.id },
      data: { variables: JSON.stringify(vars) },
    });
  }
}
```

---

### 2. Backend: Enhanced Condition Node (Check API Response)

The current condition only checks user message. We need to also check variables:

#### [MODIFY] evaluateCondition method

```typescript
private evaluateCondition(
  condition: string,
  message: string,
  variables: Record<string, any> = {}
): boolean {
  const lowerMsg = message.toLowerCase();
  const [type, valStr] = condition.split(':');
  if (!valStr) return false;

  // New: Check variable values like "var:httpResponse.covered=true"
  if (type === 'var') {
    const [varPath, expected] = valStr.split('=');
    const [varName, ...path] = varPath.split('.');
    let value = variables[varName];
    if (typeof value === 'string') {
      try { value = JSON.parse(value); } catch {}
    }
    for (const key of path) {
      value = value?.[key];
    }
    return String(value) === expected;
  }

  const keywords = valStr.split(',').map(k => k.trim().toLowerCase()).filter(Boolean);
  if (type === 'contains') return keywords.some(k => lowerMsg.includes(k));
  if (type === 'exact') return keywords.some(k => lowerMsg === k);
  return false;
}
```

---

### 3. Backend: Code/Function Node Handler

```typescript
} else if (currentNode.type === 'code' || currentNode.type === 'function') {
  const code = currentNode.data.code as string;
  const outputVariable = (currentNode.data.outputVariable as string) || 'codeResult';

  let vars: Record<string, any> = {};
  try {
    if (conversation.variables) vars = JSON.parse(conversation.variables);
  } catch {}

  try {
    // Create a sandboxed function
    const fn = new Function('vars', 'userMessage', `
      ${code}
    `);
    const result = fn(vars, userMessage);
    vars[outputVariable] = typeof result === 'object' ? JSON.stringify(result) : String(result);
    await this.prisma.aiConversation.update({
      where: { id: conversation.id },
      data: { variables: JSON.stringify(vars) },
    });
  } catch (err) {
    this.logger.error(`Code execution failed: ${String(err)}`);
  }
}
```

> [!CAUTION]
> Executing arbitrary code is a security risk. Consider using a sandboxed VM like `vm2` or limiting to expression evaluation only.

---

### 4. Backend: Delay Node Handler

```typescript
} else if (currentNode.type === 'delay') {
  const seconds = (currentNode.data.seconds as number) || 1;
  await new Promise(resolve => setTimeout(resolve, seconds * 1000));
}
```

---

## Sample Flow JSON for Ahmad Chatbot

Akan saya buatkan file `prompts/flows/ahmad-wifi-coverage-check.json`:

```json
{
  "name": "WiFi Coverage Check",
  "description": "Cek coverage WiFi IdPlay berdasarkan lokasi user",
  "trigger": { "type": "keyword", "values": ["wifi", "internet", "idplay"] },
  "nodes": [
    {
      "id": "trigger",
      "type": "trigger",
      "data": { "keyword": "wifi, internet, idplay" }
    },
    {
      "id": "ask-location",
      "type": "ai-response",
      "data": {
        "prompt": "Tanyakan lokasi user dengan ramah (kota/kecamatan di Jawa Barat)"
      }
    },
    {
      "id": "extract-location",
      "type": "variable",
      "data": {
        "variableName": "location",
        "pattern": "(?:di|daerah|lokasi|kota|wilayah)\\s+([\\w\\s]+)"
      }
    },
    {
      "id": "check-coverage",
      "type": "http-request",
      "data": {
        "url": "https://api.idplay.co.id/coverage/check?location={{location}}",
        "method": "GET",
        "responseVariable": "coverageResult"
      }
    },
    {
      "id": "branch-coverage",
      "type": "condition",
      "data": {
        "condition": "var:coverageResult.covered=true",
        "labels": { "true": "Tercover", "false": "Tidak Tercover" }
      }
    },
    {
      "id": "send-packages",
      "type": "ai-response",
      "data": {
        "prompt": "Sampaikan dengan antusias bahwa lokasi tercover! Berikan daftar paket WiFi IdPlay dengan promo GRATIS INSTALASI.",
        "temperature": 0.7
      }
    },
    {
      "id": "not-covered",
      "type": "static-message",
      "data": {
        "message": "_Waduh mohon maaf ya Kak_ 🙏\n\nUntuk wilayah {{location}} belum tercover jaringan IdPlay.\n\nTapi kakak bisa cek di *idplay.co.id* siapa tau ada mitra lain di sekitar sana."
      }
    },
    {
      "id": "cross-sell",
      "type": "ai-response",
      "data": {
        "prompt": "Lakukan cross-selling dengan halus. Tawarkan jasa website atau Wazap Suite jika user punya usaha.",
        "temperature": 0.7
      }
    }
  ],
  "edges": [
    { "source": "trigger", "target": "ask-location" },
    { "source": "ask-location", "target": "extract-location" },
    { "source": "extract-location", "target": "check-coverage" },
    { "source": "check-coverage", "target": "branch-coverage" },
    {
      "source": "branch-coverage",
      "target": "send-packages",
      "sourceHandle": "true"
    },
    {
      "source": "branch-coverage",
      "target": "not-covered",
      "sourceHandle": "false"
    },
    { "source": "not-covered", "target": "cross-sell" }
  ]
}
```

---

## Verification Plan

### 1. Unit Test (Existing Test Discovery)

Perlu cek apakah ada test file untuk `ai-agent.service.ts`:

```bash
find new-server -name "*.spec.ts" | xargs grep -l "AiAgentService"
```

### 2. Manual Testing

**Prerequisites:**

1. Server running (`npm run start:dev` in `new-server`)
2. Client running (`npm run dev` in `new-client`)
3. WhatsApp account connected

**Test Steps:**

1. Buka Flow Builder di UI
2. Buat flow baru dengan trigger "wifi"
3. Tambahkan node HTTP Request dengan URL mock
4. Tambahkan Condition node dengan `var:response.covered=true`
5. Simpan dan aktifkan flow
6. Kirim pesan "mau tanya wifi" ke WhatsApp bot
7. Verifikasi bot meminta lokasi
8. Balas dengan lokasi "Bandung"
9. Verifikasi bot melakukan fetch dan memberikan respons sesuai hasil

---

## Questions for User

1. **Coverage API**: Apakah sudah ada API endpoint untuk cek coverage WiFi IdPlay? Atau perlu saya buatkan mock endpoint?

2. **Security Code Node**: Apakah fitur Code/Function node dibutuhkan sekarang? Ini memiliki risiko keamanan jika tidak di-sandbox dengan benar.

3. **Variable Substitution**: Untuk template message dengan variabel `{{location}}`, apakah cukup dengan simple string replace atau perlu templating engine?
