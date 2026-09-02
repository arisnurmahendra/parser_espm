v1.0.4
Fix parse value "Lajur 2"

v1.0.5
- Sinkronisasi parser PDF dengan `indikatorMap[*].keywords`.
- Perbaikan parsing blok PDF yang terpisah pada pola `LAJUR` + angka di baris berikutnya.
- Validasi lajur bernomor diperluas sampai `Lajur 5`.
- Tambah indikator visual koneksi JavaScript-AHK di navbar.
- Perbaikan callback WebView agar selalu mengembalikan JSON string.
- Ubah payload download menjadi `manifestText` tab-delimited agar stabil pada AutoHotkey v2.1 alpha.
- `manifestPath.json` ditulis ulang oleh AHK dari data download aktif.
- Tambah logging `ahk_debug.log` untuk koneksi, manifest, worker download, dan overlay gambar.
- Perbaikan kompatibilitas `Lib/Gdip_All.ahk` pada variabel lokal GDI+ yang perlu diinisialisasi eksplisit.
- Update dokumentasi root dan `docs/` sesuai alur kode terbaru.
