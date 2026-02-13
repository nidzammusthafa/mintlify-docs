# WazapSuite - System Requirements

## Minimum System Requirements

### Operating System
- **Windows**: Windows 10 (64-bit) versi 1903 atau lebih tinggi
- **Windows Server**: 2019 atau lebih tinggi (untuk deployment server)
- **Mac**: Tidak tersedia saat ini (Windows only)
- **Linux**: Tidak tersedia saat ini (Windows only)

### Hardware Requirements

#### Processor (CPU)
- **Minimum**: Intel Core i3 generasi ke-6 / AMD Ryzen 3 atau setara
- **Recommended**: Intel Core i5 generasi ke-8 / AMD Ryzen 5 atau lebih tinggi
- **Catatan**: WhatsApp automation membutuhkan CPU yang cukup kuat untuk menjalankan browser Chrome di background

#### Memory (RAM)
- **Minimum**: 4 GB RAM
- **Recommended**: 8 GB RAM atau lebih
- **Optimal**: 16 GB RAM (untuk penggunaan berat dengan multiple sessions)
- **Catatan**: Setiap session WhatsApp membutuhkan ~500MB-1GB RAM tambahan

#### Storage (Disk Space)
- **Minimum**: 2 GB ruang kosong
- **Recommended**: 5 GB ruang kosong
- **Breakdown**:
  - Aplikasi: ~250 MB
  - Chrome/Chromium: ~150 MB
  - Database & Logs: ~500 MB (dapat tumbuh sesuai penggunaan)
  - Temp files: ~100 MB
- **Tipe**: SSD direkomendasikan untuk performa yang lebih baik

#### Graphics (GPU)
- **Minimum**: Integrated graphics (Intel HD Graphics 520 atau setara)
- **Catatan**: Tidak memerlukan GPU khusus, namun Chrome membutuhkan akselerasi hardware yang memadai

### Software Requirements

#### Browser
- **Wajib**: Google Chrome atau Chromium
- **Versi**: Chrome 100+ (versi terbaru direkomendasikan)
- **Cara Install**: Aplikasi akan membantu mengarahkan ke halaman download jika Chrome belum terinstall
- **Catatan**: Browser harus dapat diakses oleh aplikasi untuk WhatsApp Web automation

#### Runtime Dependencies (Otomatis Included)
- Node.js runtime (bundled dalam aplikasi)
- SQLite database (bundled dalam aplikasi)
- Tidak perlu install dependency tambahan

#### Additional Software
- **Visual C++ Redistributable**: Versi 2015-2022 (biasanya sudah terinstall di Windows modern)
- **.NET Framework**: 4.7.2 atau lebih tinggi (untuk beberapa fitur sistem)

### Network Requirements

#### Internet Connection
- **Kecepatan**: Minimum 5 Mbps (upload & download)
- **Recommended**: 10+ Mbps untuk performa optimal
- **Stabilitas**: Koneksi yang stabil sangat penting untuk WhatsApp Web

#### Port Requirements
- **Default**: Port 4000 (dapat diubah dalam settings)
- **Firewall**: Harus mengizinkan aplikasi mengakses local network
- **Proxy**: Mendukung penggunaan proxy (konfigurasi manual diperlukan)

### Virtual Machine (VM) Support

#### Supported
- ✅ VirtualBox 6.1+
- ✅ VMware Workstation/Player 16+
- ✅ Windows Sandbox
- ✅ Hyper-V

#### Requirements untuk VM
- **Enable Virtualization**: VT-x/AMD-V harus di-enable di BIOS
- **RAM Allocation**: Minimal 4GB dedicated untuk VM
- **Shared Clipboard**: Direkomendasikan untuk kemudahan copy-paste

## Recommended System Requirements

### Untuk Personal Use (1-5 WhatsApp Numbers)
- Windows 10/11 64-bit terbaru
- Intel Core i5 / AMD Ryzen 5
- 8 GB RAM
- 5 GB SSD storage
- Koneksi internet 10 Mbps stabil

### Untuk Business Use (5-20 WhatsApp Numbers)
- Windows 10/11 64-bit Pro/Enterprise
- Intel Core i7 / AMD Ryzen 7
- 16 GB RAM
- 10 GB SSD storage
- Koneksi internet 25+ Mbps stabil

### Untuk Enterprise/Heavy Usage (20+ WhatsApp Numbers)
- Windows Server 2019/2022 atau Windows 11 Pro
- Intel Core i9 / AMD Ryzen 9 atau Xeon
- 32 GB RAM atau lebih
- 20 GB SSD storage (NVMe recommended)
- Koneksi dedicated 50+ Mbps

## Compatibility Notes

### Antivirus & Security Software
- ✅ Windows Defender (Compatible)
- ✅ Kaspersky, Norton, Bitdefender (terkadang perlu whitelist)
- ⚠️ Malwarebytes (mungkin mendeteksi Puppeteer, perlu whitelist)
- 📝 **Note**: Beberapa antivirus mungkin mendeteksi automation tools, silakan whitelist aplikasi

### Other Software Compatibility
- ✅ Microsoft Office Suite
- ✅ Google Chrome (sudah terinstall atau akan diarahkan install)
- ✅ Firefox, Edge (tidak konflik)
- ⚠️ Other WhatsApp automation tools (tidak disarankan berjalan bersamaan)

### Cloud/Desktop Environment
- ✅ Physical PC/Laptop
- ✅ Virtual Machine (dengan specs yang memadai)
- ❌ Windows Subsystem for Linux (WSL) - Tidak didukung
- ❌ Remote Desktop tanpa GPU (akan bermasalah dengan Chrome)

## Pre-Installation Checklist

Sebelum membeli/menginstall, pastikan:

- [ ] Windows 10/11 64-bit dengan update terbaru
- [ ] Minimal 4GB RAM tersedia (8GB+ direkomendasikan)
- [ ] Minimal 2GB disk space tersedia
- [ ] Koneksi internet stabil
- [ ] Akses administrator (untuk install Chrome jika belum ada)
- [ ] Antivirus tidak terlalu agresif (atau siap whitelist)

## Known Limitations

1. **Windows Only**: Aplikasi hanya berjalan di Windows (tidak support Mac/Linux)
2. **Chrome Dependency**: Membutuhkan Chrome/Chromium terinstall
3. **RAM Usage**: Penggunaan RAM akan bertambah seiring jumlah session WhatsApp
4. **Single Instance**: Tidak disarankan menjalankan multiple instance aplikasi
5. **VM Performance**: Performa di VM mungkin 20-30% lebih lambat dibanding native

## Troubleshooting Hardware Issues

### Aplikasi lambat/terasa berat
- Pastikan RAM tersedia minimal 4GB
- Tutup aplikasi lain yang tidak perlu
- Gunakan SSD jika masih menggunakan HDD

### Chrome tidak terdeteksi
- Install Google Chrome dari google.com/chrome
- Pastikan Chrome dapat dibuka secara manual
- Coba restart aplikasi setelah install Chrome

### Koneksi WhatsApp sering terputus
- Pastikan koneksi internet stabil
- Cek firewall tidak memblokir aplikasi
- Hindari menggunakan VPN saat menjalankan aplikasi

## Support

Jika hardware Anda memenuhi requirements minimal namun mengalami masalah, silakan hubungi support kami dengan menyertakan:
- Spesifikasi PC/laptop Anda
- Screenshot error (jika ada)
- Log file dari aplikasi

---

**Last Updated**: February 2026  
**Applies to Version**: v1.0.6+
