# Parser ESPM

Parser ESPM adalah aplikasi desktop berbasis AutoHotkey v2 dan WebView2 untuk mengambil data Self Assessment dari eSPM BPJT, menyimpan data gambar ke IndexedDB, membaca laporan PDF pemenuhan SA, mencocokkan baris PDF dengan data gambar, lalu mengunduh dan memberi overlay informasi pada gambar hasil perbaikan.

## Fungsi Utama

- Parse HTML dari halaman eSPM Satemuan menjadi data terstruktur.
- Simpan, tampilkan, filter, backup, restore, dan ekspor data gambar melalui IndexedDB.
- Ekstrak laporan PDF pemenuhan Self Assessment menggunakan PDF.js.
- Matching data PDF dengan database gambar berdasarkan tanggal, indikator, STA, jalur, lajur, dan kemiripan teks.
- Kirim daftar gambar valid ke AutoHotkey untuk download, kompresi, overlay timestamp/lokasi, dan ekspor.
- Kelola konfigurasi overlay, folder output, mode pemuatan gambar, dan format nama file.

## Komponen

- `parser_espm_refactor.ahk`: aplikasi desktop, WebView2 host, integrasi eSPM, download gambar, kompresi, overlay, dan komunikasi AHK-JavaScript.
- `index.html`: antarmuka utama berbasis Bootstrap, DataTables, modal konfigurasi, tab workflow, dan dokumentasi in-app.
- `index.js`: parser HTML/PDF, IndexedDB, matching, render tabel, komunikasi WebView2, dan manajemen konfigurasi frontend.
- `style.css`: styling tambahan untuk textarea, drop zone, label bantuan, dan indikator koneksi AHK.
- `config.json`: konfigurasi runtime aplikasi.
- `manifestPath.json`: manifest terakhir daftar gambar valid yang diproses AHK. File ini ditulis ulang saat `Download & Proses`.
- `img_download/`: folder gambar asli hasil download.
- `img_export/`: folder gambar hasil kompresi dan overlay.
- `Lib/`: dependency AutoHotkey, termasuk `WebViewToo.ahk`, `Gdip_All.ahk`, dan `JSON.ahk`.

## Kebutuhan

- Windows.
- AutoHotkey v2.1 alpha sesuai deklarasi `#Requires AutoHotkey >= v2.1-alpha.31`.
- Microsoft Edge WebView2 Runtime.
- Koneksi internet untuk membuka eSPM, memuat CDN frontend, dan mengunduh gambar.
- Akses/login ke `https://espm.bpjt.pu.go.id/v1-satemuan`.

## Cara Pakai Ringkas

1. Jalankan `parser_espm_refactor.ahk` atau executable hasil compile.
2. Gunakan tab `Database Images` untuk mengambil data dari eSPM. Data bisa berasal dari hasil scraping WebView atau HTML yang ditempel manual.
3. Pastikan data tersimpan dan tampil di tabel database.
4. Masuk ke tab `PDF Extract Data`, pilih file laporan PDF pemenuhan SA.
5. Aplikasi membaca PDF, memecah blok data, memvalidasi indikator/jalur/lajur, lalu mencocokkannya dengan IndexedDB.
6. Periksa tabel PDF. Baris dengan ikon sukses memiliki URL `repair100` dan siap diproses.
7. Atur folder download/export dan format nama file.
8. Klik `Download & Proses`. JavaScript mengirim `manifestText` ke AHK, lalu AHK menulis ulang `manifestPath.json` dan menjalankan download/overlay.
9. Lihat progres dan hasil gambar di tab `Process Gambar`.

## Shortcut

- `F1`: buka tab bantuan.
- `F5`: reload halaman utama.
- `Ctrl + F5`: reload halaman utama.
- `F9`: buka Developer Tools WebView.
- `Ctrl + Esc`: keluar aplikasi.

## Dokumentasi Lanjutan

- [Proses bisnis](docs/business-process.md)
- [Arsitektur teknis](docs/architecture.md)
- [Skema data dan JSON](docs/data-schema.md)
- [Operasi dan troubleshooting](docs/operations.md)

## Catatan Penting

Aplikasi ini bergantung pada struktur HTML eSPM dan format teks laporan PDF. Jika eSPM atau PDF berubah format, parser dan aturan matching perlu disesuaikan. Untuk AutoHotkey v2.1 alpha baru, hindari payload array JSON besar pada callback WebView; workflow download memakai `manifestText` agar parsing AHK lebih stabil. Validasi manual tetap diperlukan sebelum hasil digunakan sebagai dokumen kerja resmi.
