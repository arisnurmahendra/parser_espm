# Arsitektur Teknis

Parser ESPM memakai pola desktop shell dengan AutoHotkey sebagai host native dan halaman web lokal sebagai UI.

## Diagram Alur

```text
eSPM WebView
  -> AHK WebViewMessageHandler
  -> MyWindow.PostWebMessageAsJson
  -> index.js ahkWebMessage
  -> IndexedDB SatemuanDB
  -> PDF.js parser
  -> matching
  -> ahk.SubmitJson(action=download, manifestText)
  -> AHK WebJsonEvent
  -> ParseDownloadManifestText
  -> DownloadImages
  -> img_download + img_export
  -> WebView progress cards
```

## Frontend

`index.html` menyediakan tab utama:

- `Database Images`: input HTML, tabel IndexedDB, filter, summary, export.
- `PDF Extract Data`: input PDF, tabel hasil parsing/matching, konfigurasi path dan nama file.
- `Process Gambar`: progress download dan kartu hasil gambar.
- `How To`: dokumentasi penggunaan di aplikasi.
- `Tentang Aplikasi`: informasi aplikasi.

`index.js` menangani:

- state global (`globalData`, `allData`, `config`, `filenameOrder`),
- konfigurasi PDF.js worker,
- parsing HTML eSPM,
- CRUD IndexedDB,
- parsing PDF,
- matching PDF dengan database,
- render DataTables,
- komunikasi dengan AHK,
- indikator status koneksi JS-AHK,
- render progress gambar,
- backup/restore/export.

`style.css` hanya memuat styling ringan untuk textarea, drop zone, helper label, dan class hide.

## Backend AHK

`parser_espm_refactor.ahk` membuat dua WebView:

- `MyWindow`: UI utama lokal dari `index.html`.
- `ESPMWindow`: halaman eSPM untuk auto-scroll/scraping.

AHK bertanggung jawab untuk:

- membuat host virtual `ahk.localhost`,
- menerima callback dari JavaScript,
- membuka halaman eSPM,
- menjalankan script auto-scroll pada eSPM,
- mendownload gambar dengan WinHttp,
- memproses gambar menggunakan GDI+,
- mengirim update progress ke UI.

## Komunikasi

JavaScript ke AHK:

```js
ahk.SubmitJson(JSON.stringify({ action: "request_config" }));
ahk.SubmitJson(JSON.stringify({ action: "config", data: config }));
ahk.SubmitJson(JSON.stringify({ action: "download", manifestText }));
```

Untuk download gambar, JavaScript tidak mengirim array object besar sebagai `data`. `index.js` mengubah daftar gambar valid menjadi `manifestText` dengan format satu gambar per baris dan kolom dipisah tab:

```text
fileName<TAB>tanggal<TAB>no<TAB>latlng<TAB>url
```

AHK membaca format ini dengan `ParseDownloadManifestText()`, lalu menulis ulang `manifestPath.json` memakai `JSON.stringify(...)`. Pola ini dipakai karena lebih stabil pada AutoHotkey v2.1 alpha dibanding parsing ulang JSON array besar di callback WebView.

AHK ke JavaScript:

```ahk
MyWindow.PostWebMessageAsJson(JSON.stringify(packet))
```

Jendela eSPM ke AHK:

```js
ahk.message(JSON.stringify({ cmd: "espm_body", html: document.querySelector("#listmon")?.innerHTML }));
```

Semua callback AHK yang dipanggil dari JS sebaiknya mengembalikan JSON string dengan minimal field `ok`.

## Penyimpanan

IndexedDB:

- Database: `SatemuanDB`.
- Object store: `temuanStore`.
- Key path: `id`.
- Store tambahan: `configStore`, key `id: "config"`.

File lokal:

- `config.json`: konfigurasi persist di disk.
- `manifestPath.json`: manifest terakhir yang ditulis ulang AHK dari `manifestText`.
- `img_download/`: gambar asli dari eSPM.
- `img_export/`: gambar hasil overlay.

## Dependency Web

`index.html` memuat dependency via CDN:

- Bootstrap 5,
- jQuery,
- SweetAlert2,
- Font Awesome,
- Select2,
- DataTables,
- PDF.js,
- pdfmake,
- xlsx.

## Dependency AHK

- `Lib/WebViewToo.ahk`
- `Lib/WebView2.ahk`
- `Lib/Gdip_All.ahk`
- `Lib/JSON.ahk`
- `Lib/Promise.ahk`
- `Lib/ComVar.ahk`
- `Lib/32bit/WebView2Loader.dll`
- `Lib/64bit/WebView2Loader.dll`

## Compile

Resource compile disiapkan di bagian akhir `parser_espm_refactor.ahk`:

```ahk
;@Ahk2Exe-AddResource index.html, index.html
;@Ahk2Exe-AddResource style.css, style.css
;@Ahk2Exe-AddResource index.js, index.js
;@Ahk2Exe-AddResource config.json, config.json
```

Saat compiled, script mengekstrak ulang resource `index.html`, `index.js`, dan `style.css`, lalu mengatur atribut file.

## Catatan Kompatibilitas

- Semua callback WebView (`SubmitJson` dan `message`) harus mengembalikan string.
- Library `Lib/JSON.ahk` memakai method `JSON.parse(...)` lowercase.
- Beberapa fungsi `Lib/Gdip_All.ahk` menggunakan variabel output `DllCall`; pada AHK v2.1 alpha baru, variabel lokal penting seperti `pBitmap`, `pCodec`, dan hasil regex perlu diinisialisasi eksplisit.
