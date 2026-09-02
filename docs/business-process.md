# Proses Bisnis

Dokumen ini menjelaskan alur kerja bisnis Parser ESPM dari pengambilan data sampai gambar siap ekspor.

## Tujuan

Parser ESPM membantu tim memproses data Self Assessment eSPM menjadi gambar hasil perbaikan yang sudah diberi overlay informasi penting. Aplikasi mengurangi pekerjaan manual dalam mencari gambar `Repair 100%`, memberi nama file, memberi timestamp/lokasi, dan menyiapkan hasil untuk pelaporan.

## Aktor

- Operator aplikasi: pengguna yang membuka eSPM, mengambil data, memilih PDF, dan menjalankan proses gambar.
- Sistem eSPM BPJT: sumber HTML data Satemuan dan URL gambar.
- Laporan PDF pemenuhan SA: sumber daftar entri yang akan dicocokkan dengan database gambar.
- Aplikasi desktop AHK: backend lokal untuk download dan manipulasi gambar.
- WebView HTML/JS: frontend lokal untuk parsing, tabel, konfigurasi, dan kontrol proses.

## Alur Utama

1. Operator membuka aplikasi desktop.
2. Aplikasi memuat `index.html` melalui host lokal `http://ahk.localhost/index.html`.
3. Operator membuka jendela eSPM dari aplikasi atau menempel HTML eSPM secara manual.
4. HTML eSPM diparse menjadi record `globalData`.
5. `globalData` disimpan ke IndexedDB `SatemuanDB.temuanStore`.
6. Operator memilih laporan PDF pemenuhan SA.
7. PDF dibaca oleh PDF.js dan dipecah menjadi blok entri.
8. Setiap blok divalidasi:
   - header nomor dan tanggal,
   - STA,
   - indikator,
   - jalur,
   - lajur,
   - tanggal/metode/durasi penanganan.
9. Data PDF dicocokkan dengan IndexedDB.
10. Jika match ditemukan, field `repair100` dan `latlng` diisi dari database.
11. Operator menekan `Download & Proses`.
12. JavaScript membuat `manifestText` dari gambar valid dan mengirimnya ke AHK.
13. AHK mengubah `manifestText` menjadi array data, menulis ulang `manifestPath.json`, lalu mulai worker download.
14. AHK mendownload gambar, membaca orientasi, resize, membuat overlay, dan menyimpan output.
15. AHK mengirim progres/hasil ke WebView.
16. WebView menampilkan kartu hasil di tab `Process Gambar`.

## Alur Scraping eSPM

Ada dua jalur input data eSPM:

- Manual: operator menempel HTML ke `#sourceInput`, lalu klik `Proses Source`.
- WebView otomatis: operator membuka jendela eSPM, menjalankan auto-scroll, lalu AHK mengambil `#listmon.innerHTML` dan mengirimnya ke halaman utama.

Parser mencari struktur kartu Satemuan, hidden input, URL carousel gambar, indikator, jalur, lajur, STA, tanggal, durasi, dan koordinat.

## Alur Parsing PDF

Parser PDF menggabungkan teks per halaman menjadi `pageText`, lalu memecah blok dengan pola:

```text
<no> <yyyy-mm-dd hh:mm:ss> ...
```

Setelah itu parser menangani kasus khusus, misalnya angka lajur yang terbaca sebagai blok baru:

```text
... JALUR A LAJUR
1 2026-08-24 19:10:00 ...
```

Blok final seharusnya menjadi:

```text
... JALUR A LAJUR 1 2026-08-24 19:10:00 ...
```

## Alur Matching

Matching memakai kombinasi:

- tanggal,
- indikator,
- STA,
- jalur,
- lajur,
- kecocokan teks indikator/deskripsi.

Jika exact match ditemukan, skor dianggap tinggi. Jika tidak, parser mencari kandidat terbaik dan menyimpan alasan gagal ke `lastParseErrors`.

## Output Bisnis

Output utama adalah gambar `.jpg` hasil overlay di folder `pathExport`. Gambar diberi nama dari kombinasi field yang diatur di `filenameFormat`, misalnya:

```text
01 Lokasi 0+000 Non Lajur Ambulance.jpg
```

Overlay menampilkan:

- tanggal,
- latitude/longitude repair 100%,
- teks ruas tetap `Ruas Tol Jombang-Mojokerto`.

## Alur Download Teknis

Payload download dari JavaScript ke AHK memakai format `manifestText`, satu gambar per baris:

```text
fileName<TAB>tanggal<TAB>no<TAB>latlng<TAB>url
```

AHK sengaja tidak mem-parse ulang array JSON besar saat download. Pendekatan ini membuat callback WebView lebih stabil pada AutoHotkey v2.1 alpha dan tetap menghasilkan `manifestPath.json` untuk audit/debug.

## Validasi Manual

Operator tetap perlu memeriksa:

- jumlah blok PDF terbaca,
- jumlah pairing berhasil/gagal,
- kecocokan indikator,
- kecocokan STA, jalur, dan lajur,
- gambar hasil overlay,
- lokasi folder export.
