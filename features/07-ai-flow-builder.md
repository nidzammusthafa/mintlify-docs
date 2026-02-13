# Fitur: AI Flow Builder (🤖)

## 1. Overview

**AI Flow Builder** adalah editor visual (drag-and-drop) untuk membuat alur percakapan otomatis (chatbot) yang cerdas. Berbeda dengan auto-reply biasa yang kaku, AI Flow Builder memungkinkan pembuatan skenario percakapan kompleks, integrasi dengan AI (LLM), pemanggilan API eksternal, dan logika percabangan.

## 2. Komponen Detail

### 2.1 Flow Editor

**Deskripsi:**
Canvas visual berbasis node dimana user dapat merangkai alur percakapan.

**Jenis Node:**
1.  **Trigger Node:** Titik awal percakapan (contoh: Pesan masuk berisi kata "Harga").
2.  **Message Node:** Mengirim pesan balasan (Teks, Gambar, Tombol).
3.  **Condition Node:** Percabangan logika (If/Else). Contoh: Jika jam kerja -> Node A, jika di luar jam kerja -> Node B.
4.  **AI Response Node:** Menghasilkan balasan dinamis menggunakan LLM (Gemini/GPT) berdasarkan prompt dan konteks percakapan.
5.  **API Request Node:** Mengambil data dari server eksternal (contoh: Cek stok barang).
6.  **Function Node:** Eksekusi kode JavaScript custom untuk logika kompleks.
7.  **Delay Node:** Menunggu beberapa detik/menit sebelum lanjut ke node berikutnya.

### 2.2 AI Integration (Gemini)

**Deskripsi:**
Integrasi mendalam dengan Google Gemini untuk pemahaman bahasa alami (NLP).

**Parameter:**
- `model`: Versi model (gemini-pro, gemini-flash).
- `systemInstruction`: Instruksi persona bot (misal: "Kamu adalah CS ramah toko sepatu").
- `temperature`: Tingkat kreativitas balasan (0.0 - 1.0).
- `contextWindow`: Jumlah chat sebelumnya yang diingat bot.

### 2.3 Conversation Lock & Deduplication

**Deskripsi:**
Mekanisme keamanan untuk mencegah bot membalas berulang kali pada pesan yang sama atau saat sedang memproses pesan.

**Logika:**
- **Deduplication:** Setiap pesan masuk dicatat `messageId`-nya. Jika ID sama masuk lagi, diabaikan.
- **Locking:** Saat flow berjalan untuk user A, sistem mengunci sesi user A di Redis agar tidak ada proses paralel yang menimpa state.

## 3. User Flow

### Skenario: Customer Service Otomatis
1.  **Trigger:** User mengirim pesan "Mau tanya produk".
2.  **AI Node:** Bot menganalisis maksud user dan menjawab sapaan.
3.  **Menu Node:** Bot mengirim pesan List Menu (Katalog, Cek Resi, Bicara dengan Admin).
4.  **Condition:**
    - Jika pilih "Katalog" -> Kirim gambar produk.
    - Jika pilih "Cek Resi" -> Minta nomor resi -> API Node (Cek ke ekspedisi) -> Balas status.
    - Jika pilih "Admin" -> Stop Flow -> Notifikasi ke Inbox Admin.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `GET` | `/flows` | List semua flow |
| `POST` | `/flows` | Buat flow baru |
| `GET` | `/flows/:id` | Detail konfigurasi flow |
| `PUT` | `/flows/:id` | Update flow (simpan canvas) |
| `POST` | `/flows/:id/activate` | Aktifkan flow |
| `POST` | `/flows/:id/deactivate` | Nonaktifkan flow |
| `POST` | `/flows/test` | Simulasi flow (dry run) |

## 5. Integrasi

-   **Inbox:** Flow yang aktif akan menangani pesan masuk sebelum masuk ke Inbox (kecuali diset "Handover to Agent").
-   **Contacts:** Flow dapat menyimpan data yang dikumpulkan dari user (misal: nama, email) ke custom fields kontak.
-   **External API:** Kemampuan webhook untuk integrasi dengan CRM atau database stok pihak ketiga.
