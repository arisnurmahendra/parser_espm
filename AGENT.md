# Agent Guide

Panduan ini ditujukan untuk developer atau coding agent yang akan memelihara Parser ESPM.

## Prinsip Kerja

- Baca `parser_espm_refactor.ahk`, `index.html`, `index.js`, `style.css`, dan dokumen di `docs/` sebelum perubahan besar.
- Jaga kompatibilitas AutoHotkey v2.1 alpha dan WebViewToo.
- Pertahankan kontrak komunikasi JavaScript-AHK. Callback yang dipanggil dari JS harus selalu mengembalikan nilai.
- Jangan menghapus atau menimpa backup database, PDF laporan, arsip, atau output gambar tanpa instruksi eksplisit.
- Copy file ke folder GitHub eksternal dan `git push` hanya boleh dilakukan setelah ada perintah eksplisit dari pengguna.
- Hindari refactor besar saat memperbaiki parser. Format PDF dan HTML eSPM sensitif terhadap regex kecil.

## Area Kode Penting

- `index.js`
  - `extractAllSatemuanData`: parsing HTML eSPM menjadi `globalData`.
  - `openDB`, `saveToIndexedDB`, `getAllFromIndexedDB`: penyimpanan lokal.
  - `$('#pdfInput').on('change', ...)`: parsing laporan PDF.
  - `validIndikatorSet`, `validJalurSet`, `validLajurSet`: daftar validasi parsing PDF.
  - `matchAndMergeWithDB`, `findBestMatch`: pairing PDF dengan database.
  - `$('#downloadAllBtn').on('click', ...)`: membuat `manifestText` download untuk AHK.
  - `ahkWebMessage`: menerima pesan dari AHK.
  - `renderPdfTable`, `renderCardsWithDelay`: tampilan hasil.

- `parser_espm_refactor.ahk`
  - `WebJsonEvent`: menerima `ahk.SubmitJson(...)` dari JS.
  - `WebViewMessageHandler`: menerima `ahk.message(...)` dari WebView eSPM.
  - `ESPM_TextMining`: auto-scroll halaman eSPM dan mengambil HTML.
  - `ParseDownloadManifestText`: mengubah payload teks baris/tab menjadi array `Map`.
  - `DownloadImages`: memproses data download.
  - `DownloadFile`: download gambar via WinHttp.
  - `CompressAndOverlayImage`: resize, orientasi, overlay, dan export gambar.
  - `LoadConfigFromFile`, `SaveJsonToDisk`: konfigurasi runtime.

## Kontrak Komunikasi

JavaScript ke AHK memakai:

```js
const manifestText = validData
  .map(item => [item.fileName, item.tanggal, item.no, item.latlng, item.url].join("\t"))
  .join("\n");

await ahk.SubmitJson(JSON.stringify({ action: "download", manifestText }));
```

AHK harus mengembalikan JSON string seperti:

```json
{"ok":true,"action":"download"}
```

AHK ke JavaScript memakai `PostWebMessageAsJson(...)`, diterima oleh `window.chrome.webview.addEventListener("message", ahkWebMessage)`.

Catatan: `manifestPath.json` tetap ditulis ulang oleh AHK untuk arsip/debug, tetapi proses download tidak bergantung pada parsing ulang JSON array besar. Ini menghindari error `No value was returned` pada AutoHotkey v2.1 alpha tertentu.

## Aturan Validasi Domain

- Jalur valid: `Jalur A`, `Jalur B`, `Non Jalur`.
- Lajur valid: `Non Lajur`, `Bahu Luar`, `Bahu Dalam`, `Ramp`, `Akses`, `Lajur Motor`, dan lajur bernomor `Lajur 1` sampai `Lajur 5`.
- Indikator valid berasal dari `indikatorMap[*].keywords`.
- STA PDF umumnya dibaca sebagai angka dengan tiga desimal, contoh `688.500`.

## Checklist Perubahan

1. Jalankan pencarian cepat dengan `rg` untuk melihat semua pemakaian fungsi/field yang akan diubah.
2. Untuk perubahan JS, jalankan `node --check index.js`.
3. Untuk perubahan AHK, jalankan script secara manual jika AutoHotkey tidak tersedia di PATH.
4. Uji minimal workflow terkait:
   - HTML eSPM masuk ke IndexedDB.
   - PDF terbaca tanpa parsing error baru.
   - Matching menghasilkan `repair100`.
   - Payload `download` terkirim ke AHK.
   - Gambar tampil di tab `Process Gambar`.
5. Update dokumen jika mengubah skema, format JSON, atau proses bisnis.

## Risiko Umum

- Regex PDF terlalu longgar dapat memecah angka lajur menjadi nomor blok baru.
- Regex indikator terlalu pendek dapat menangkap keyword umum sebelum keyword spesifik.
- Callback AHK tanpa `return` bisa memunculkan error `No value has been return`.
- Payload JSON besar atau top-level array yang diparse ulang di AHK bisa memunculkan `No value was returned`; gunakan `manifestText` untuk jalur download.
- Beberapa fungsi lama di `Lib/Gdip_All.ahk` perlu inisialisasi variabel lokal eksplisit pada AHK v2.1 alpha baru.
- DataTable lama perlu di-clear dan diisi ulang saat PDF baru dibaca.
- Mode `BASE64` menghasilkan payload besar ke WebView; cocok untuk preview tetapi bisa berat untuk banyak gambar.
