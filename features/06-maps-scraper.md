# Fitur: Maps Scraper (🗺️)

## 1. Overview

**Maps Scraper** adalah fitur pencari leads otomatis yang mengambil data bisnis (Nama, No HP, Alamat, Kategori) langsung dari Google Maps. Fitur ini sangat powerful untuk mendapatkan database prospek baru yang tertarget berdasarkan lokasi dan kategori usaha.

## 2. Komponen Detail

### 2.1 Scraping Modes

Terdapat dua mode operasi utama:

#### A. Conventional Mode (Search Query)
User memasukkan kata kunci pencarian seperti di aplikasi Maps biasa.
- **Input:** "Coffee shop di Jakarta Selatan", "Bengkel motor Bandung"
- **Output:** ~20-100 hasil per query (tergantung limit scroll Google Maps).
- **Use Case:** Pencarian cepat dan spesifik.

#### B. Coordinate Mode (Grid Search)
User menentukan titik koordinat dan radius, sistem akan mencari bisnis di sekitar titik tersebut secara presisi.
- **Input:** Latitude, Longitude, Keyword.
- **Output:** Maksimal 20 hasil per titik koordinat (limit API mobile).
- **Keunggulan:** Bisa mencakup area luas dengan memecahnya menjadi grid (jaring-jaring) koordinat, memastikan coverage area yang lebih merata dibanding search biasa.

### 2.2 OSM Generator (Grid Generator)

**Deskripsi:**
Alat bantu untuk menghasilkan titik-titik koordinat (Grid) berdasarkan nama wilayah administratif (Kecamatan/Kota).

**Fitur:**
- Menggunakan **OpenStreetMap (Nominatim)** untuk mendapatkan batas wilayah (polygon).
- **Grid Spacing:** Mengatur jarak antar titik (misal: setiap 500 meter).
- **Visualisasi:** Menampilkan titik-titik grid di peta sebelum proses scraping dimulai.

### 2.3 Data Extraction Fields

Data yang diambil dari setiap lokasi bisnis:
- `name`: Nama Bisnis
- `phoneRaw`: Nomor telepon (format asli di maps)
- `phoneFormatted`: Nomor telepon dinormalisasi (628xxx)
- `category`: Kategori bisnis
- `address`: Alamat lengkap
- `rating`: Bintang (0-5)
- `reviewsCount`: Jumlah ulasan
- `website`: Website URL
- `mapsLink`: Link Google Maps

### 2.4 Phone Formatting

**Deskripsi:**
Sistem otomatis memperbaiki format nomor telepon agar siap dipakai untuk WhatsApp Blast.
- `(021) 123456` -> `6221123456`
- `0812-3456-7890` -> `6281234567890`
- `+62 812...` -> `62812...`

## 3. User Flow

1. **Setup Job:** User memilih mode (Conventional/Coordinate).
2. **Input Query:**
   - Conventional: Masukkan list keywords.
   - Coordinate: Generate grid dari nama kecamatan -> "Kecamatan Coblong".
3. **Start Scraping:** Sistem menjalankan browser otomatis (Headless Chrome) di background.
4. **Monitoring:** User melihat data masuk secara real-time di tabel hasil.
5. **Filtering:** User memfilter hasil yang memiliki nomor telepon saja (`hasPhone = true`).
6. **Export/Save:** User menyimpan hasil scraping ke menu Contacts atau download Excel.

## 4. API Reference

| Method | Endpoint | Deskripsi |
| ------ | -------- | --------- |
| `POST` | `/maps-scraper/jobs` | Buat job scraping baru |
| `POST` | `/maps-scraper/generate-coords` | Generate grid koordinat dari OSM |
| `GET` | `/maps-scraper/jobs/:id/results` | Ambil hasil scraping (paginated) |
| `POST` | `/maps-scraper/jobs/:id/start` | Jalankan job |
| `POST` | `/maps-scraper/jobs/:id/stop` | Hentikan job |
| `POST` | `/maps-scraper/jobs/:id/save-to-contacts` | Simpan hasil ke fitur Contacts |

## 5. Integrasi

- **Puppeteer Service:** Menggunakan instance browser terpisah agar tidak mengganggu sesi WhatsApp.
- **Contacts:** Integrasi langsung untuk menyimpan leads hasil scraping menjadi kontak siap blast.
- **Maps Visualization:** Menggunakan LeafletJS untuk menampilkan persebaran lokasi bisnis yang berhasil discrape.
