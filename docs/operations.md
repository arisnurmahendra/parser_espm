# Operasi dan Troubleshooting

## Menjalankan Aplikasi

1. Pastikan AutoHotkey v2.1 alpha dan WebView2 Runtime tersedia.
2. Jalankan `parser_espm_refactor.ahk`.
3. Jika `Debug` bernilai `1`, Developer Tools akan dibuka otomatis.
4. UI utama dimuat dari `http://ahk.localhost/index.html`.

## Workflow Harian

1. Ambil/scrape data eSPM.
2. Cek jumlah data di tab `Database Images`.
3. Backup IndexedDB jika data penting.
4. Pilih laporan PDF di tab `PDF Extract Data`.
5. Cek jumlah blok terbaca dan data gagal pairing.
6. Jalankan `Download & Proses` hanya jika ada baris dengan `repair100`.
7. Cek output di `img_export`.

## Backup dan Restore

Backup:

- Gunakan tombol `Backup DB`.
- File backup berisi array record `globalData`.

Restore:

- Gunakan input `Restore DB`.
- Pastikan file JSON backup adalah array object dengan field `id`.

## Masalah Umum

### `Deprecated API usage: No "GlobalWorkerOptions.workerSrc" specified`

Penyebab:

- PDF.js belum diberi path worker.

Solusi:

- Pastikan `index.js` mengatur:

```js
pdfjsLib.GlobalWorkerOptions.workerSrc = "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.9.179/pdf.worker.min.js";
```

### `Total blok terdeteksi: 127`, tetapi `Kirim ke AHK: 0/127 data valid`

Penyebab:

- PDF berhasil diparse, tetapi tidak ada baris yang punya URL `repair100`.
- Matching PDF ke IndexedDB gagal.
- Database eSPM belum diisi atau bukan periode yang sesuai.

Solusi:

- Cek tab `Database Images`.
- Pastikan data eSPM sudah discrape untuk periode yang sama dengan PDF.
- Cek modal gagal pairing.
- Cocokkan indikator, STA, jalur, lajur, dan tanggal.

### `No value has been return`

Penyebab:

- Callback AHK yang dipanggil dari JavaScript tidak mengembalikan nilai.
- Payload yang dikirim ke `SubmitJson` terlalu kompleks atau berisi array besar sehingga `JSON.parse(Message, false, true)` gagal sebelum `action` terbaca.
- Parser JSON AHK atau library lama tidak cocok dengan versi AutoHotkey v2.1 alpha yang sedang dipakai.

Solusi:

- Pastikan `WebJsonEvent` dan `WebViewMessageHandler` selalu `return` JSON string.
- Cek `ahk_debug.log`. Jika hanya muncul `WebJsonEvent error` tanpa `WebJsonEvent action=download`, masalah terjadi saat parse payload utama.
- Untuk workflow download, gunakan `manifestText`, bukan `data: validData` atau parsing ulang `manifestJson` array besar di AHK.

Contoh:

```ahk
return JSON.stringify(Map("ok", true, "action", "download"))
```

Payload download yang benar:

```js
await ahk.SubmitJson(JSON.stringify({ action: "download", manifestText }));
```

### Error `[JALUR] Jalur / Lajur tidak ditemukan`

Penyebab:

- Format PDF tidak sesuai pola parser.
- Angka lajur terbaca sebagai awal blok baru.
- Teks PDF memakai variasi case/spasi yang belum ditangani.

Solusi:

- Cek blok error di console.
- Pastikan pola `JALUR A LAJUR 1` atau variasinya tergabung dalam satu blok.
- Perbarui regex lajur/jalur jika format PDF berubah.

### Error `[STA] STA tidak ditemukan`

Penyebab:

- Parser memecah blok di tengah data.
- Baris lanjutan dikira record baru.
- PDF tidak menyertakan STA dengan pola tiga desimal.

Solusi:

- Cek apakah blok sebelumnya berakhir dengan `LAJUR`.
- Gabungkan blok lanjutan sebelum parsing field.

### Gambar gagal download

Penyebab:

- URL expired atau tidak dapat diakses.
- Koneksi internet bermasalah.
- Folder `pathDownload` tidak dapat dibuat/ditulis.

Solusi:

- Buka URL gambar di browser.
- Cek akses folder.
- Cek `manifestPath.json`.

### Gambar output tidak tampil

Penyebab:

- `pathMode` tidak cocok dengan mode runtime.
- Path mengandung karakter yang belum di-encode.
- Base64 terlalu besar.
- File belum terbentuk di `img_export` karena proses overlay/save gagal.

Solusi:

- Coba `pathMode: BASE64`.
- Untuk compiled app, gunakan virtual host `ahk.localhost`.
- Pastikan file ada di `img_export`.

### File ada di `img_download`, tetapi `img_export` kosong

Penyebab:

- Download gambar berhasil, tetapi `CompressAndOverlayImage()` gagal.
- GDI+ gagal load gambar, menulis teks overlay, atau menyimpan file.
- Pada AutoHotkey v2.1 alpha baru, library `Gdip_All.ahk` lama bisa memunculkan `This local variable has not been assigned a value` jika variabel output `DllCall` belum diinisialisasi eksplisit.

Solusi:

- Cek `ahk_debug.log` untuk baris `overlay: start`, `overlay: bitmap loaded value`, `overlay: orientation`, dan `overlay: save to`.
- Pastikan `Lib/Gdip_All.ahk` menginisialisasi variabel seperti `pBitmap`, `pBitmapOld`, `pCodec`, dan hasil regex di `Gdip_TextToGraphics()`.
- Jika muncul `Gagal simpan gambar export. Kode GDI+`, cek kode return dan path yang ditampilkan.
- Pastikan folder `pathExport` bisa dibuat/ditulis.

## Validasi Setelah Perubahan

Untuk JavaScript:

```bash
node --check index.js
```

Untuk AHK:

- Jalankan script secara manual.
- Buka DevTools dengan `F9`.
- Uji minimal `request_config`, parse HTML, parse PDF, dan download satu gambar.

## Catatan Maintenance

- Hindari mengubah regex PDF tanpa contoh blok asli.
- Jika menambah field baru, update `defaultConfig.validKeys`, dokumentasi skema, dan proses export.
- Jika mengubah payload AHK, update skema `manifestText`, `manifestPath.json`, dan `WebJsonEvent`.
- Jika eSPM mengubah HTML, prioritas cek `extractAllSatemuanData`.
