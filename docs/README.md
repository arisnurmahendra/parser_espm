# Dokumentasi Parser ESPM

Folder ini berisi dokumentasi teknis dan operasional Parser ESPM.

## Daftar Dokumen

- [Proses bisnis](business-process.md): alur kerja dari scraping eSPM, parsing PDF, matching, sampai output gambar.
- [Arsitektur teknis](architecture.md): pembagian tanggung jawab `index.js`, WebView2, AutoHotkey, IndexedDB, dan GDI+.
- [Skema data dan JSON](data-schema.md): struktur `config`, IndexedDB, data PDF, `manifestText`, dan `manifestPath.json`.
- [Operasi dan troubleshooting](operations.md): cara menjalankan aplikasi, workflow harian, backup/restore, dan solusi error umum.

## Catatan Saat Ini

- Payload download aktif memakai `manifestText` tab-delimited agar stabil pada AutoHotkey v2.1 alpha.
- `manifestPath.json` tetap ditulis ulang oleh AHK sebagai manifest audit/debug.
- Validasi lajur bernomor mencakup `Lajur 1` sampai `Lajur 5`.
- Indikator valid berasal dari `indikatorMap[*].keywords` di `index.js`.
