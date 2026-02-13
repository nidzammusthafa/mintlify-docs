# WazapSuite User Documentation

Dokumentasi pengguna untuk WazapSuite - All-in-One WhatsApp Marketing & Automation Platform.

## Cara Deploy ke Vercel

### Opsi 1: Dari Dashboard Vercel

1. Buka https://vercel.com
2. Login dengan akun GitHub Anda
3. Klik "Add New..." > "Project"
4. Pilih repository `wazap-suite-user-docs`
5. Klik "Deploy"

### Opsi 2: Dari CLI

```bash
npm i -g vercel
vercel
```

## Cara Development Lokal

```bash
# Install dependencies
npm install

# Jalankan dev server
npm run dev
```

Buka http://localhost:3000 untuk melihat dokumentasi.

## Struktur Folder

```
docs/
├── introduction.mdx       # Halaman utama
├── quickstart.mdx         # Panduan cepat
├── getting-started/       # Panduan memulai
├── whatsapp/             # Dokumentasi WhatsApp
├── scraper/              # Dokumentasi Scraper
├── ai/                   # Dokumentasi AI
├── settings/             # Pengaturan
└── troubleshooting/      # Pemecahan masalah
```

## Menambahkan Halaman Baru

Tambahkan file `.mdx` di folder yang sesuai, lalu update `mint.json` untuk include halaman tersebut di navigation.

## Kredit

Dibuat dengan [Mintlify](https://mintlify.com)
