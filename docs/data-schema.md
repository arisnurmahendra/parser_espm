# Skema Data dan JSON

Dokumen ini merangkum struktur data utama yang digunakan Parser ESPM.

## Config

Sumber:

- `default_config` di `parser_espm_refactor.ahk`.
- `defaultConfig` di `index.js`.
- `config.json` di disk.
- `configStore` di IndexedDB.

Contoh:

```json
{
  "Debug": "0",
  "color": "E8E8E8",
  "colorText": "000000",
  "filenameFormat": ["no", "sta", "jalur", "lajur", "indikator"],
  "font": "Segoe UI",
  "fontSize": 34,
  "heightRectangle": 100,
  "maxDim": 1280,
  "paddingText": 10,
  "pathDownload": "img_download",
  "pathExport": "img_export",
  "pathMode": "LoadFilePath",
  "sleep": 200,
  "transcolor": "FF",
  "transcolorText": "FF"
}
```

Field penting:

- `pathMode`: `SetWorkingDir`, `LoadFilePath`, atau `BASE64`.
- `pathDownload`: folder gambar asli.
- `pathExport`: folder gambar hasil overlay.
- `filenameFormat`: urutan tag pembentuk nama file.
- `font`, `fontSize`, `paddingText`, `heightRectangle`: style overlay.
- `color`, `transcolor`, `colorText`, `transcolorText`: warna dan alpha overlay.
- `maxDim`: ukuran maksimum sisi terpanjang gambar.
- `sleep`: delay proses.

## Data eSPM / IndexedDB `globalData`

Sumber data dari HTML eSPM. Disimpan di `SatemuanDB.temuanStore` dengan key `id`.

Contoh:

```json
{
  "id": 7927,
  "temuan": "https://espm.bpjt.pu.go.id/storage/app/uploads/public/...",
  "repair0": "https://espm.bpjt.pu.go.id/storage/app/uploads/public/...",
  "repair50": "https://espm.bpjt.pu.go.id/storage/app/uploads/public/...",
  "repair100": "https://espm.bpjt.pu.go.id/storage/app/uploads/public/...",
  "des": "sticker deliniator rusak",
  "desper": "Ganti sticker rusak dengan sticker baru",
  "timestamp": "15 Jul 2022 13:50:09",
  "end": "15 Jul 2022 13:54:07",
  "tanggal": "2022-07-15",
  "dur": "3 menit",
  "ruas": "Kertosono - Mojokerto",
  "indikator": "Petunjuk Jalan [ Guide Post ]",
  "jalur": "Jalur A",
  "lajur": "Bahu Luar",
  "lokasi": "STA 689750.000",
  "latitude": "-7.49147830",
  "longitude": "112.26539330",
  "repair_latitude_0": "-7.49147830",
  "repair_longitude_0": "112.26539330",
  "repair_latitude_50": "-7.49147830",
  "repair_longitude_50": "112.26539330",
  "repair_latitude_100": "-7.49147830",
  "repair_longitude_100": "112.26539330",
  "selesai": "15 Jul 2022 13:54:07"
}
```

Field penting untuk matching:

- `tanggal` atau `end`,
- `indikator`,
- `lokasi` atau STA,
- `jalur`,
- `lajur`,
- `repair100`,
- `repair_latitude_100`,
- `repair_longitude_100`.

## Data PDF `allData`

Dihasilkan dari laporan PDF. Setelah matching, `repair100` dan `latlng` akan diisi.

```json
{
  "no": "93",
  "tanggal": "2026-08-24 19:10:00",
  "indikator": "Penanganan Kecelakaan [ Korban Kecelakaan ]",
  "deskripsi": "",
  "sta": "688.500",
  "jalur": "JALUR A",
  "lajur": "LAJUR 1",
  "metode": "5 korban luka ringan dibawa ke RS terdekat",
  "durasi": "0 hari, 00 jam, 00 menit, 00 detik",
  "repair100": "https://espm.bpjt.pu.go.id/storage/app/uploads/public/...",
  "latlng": "-7.49103360, 112.23335120",
  "content": "blok teks asli dari PDF"
}
```

Jika parsing gagal:

```json
{
  "no": "93",
  "tanggal": "",
  "indikator": "[JALUR] Jalur / Lajur tidak ditemukan",
  "deskripsi": "",
  "sta": "",
  "jalur": "",
  "lajur": "",
  "metode": "",
  "durasi": "",
  "repair100": "",
  "error": "Gagal parse block",
  "message": "[JALUR] Jalur / Lajur tidak ditemukan",
  "content": "blok teks asli dari PDF"
}
```

## Manifest Download `manifestPath.json`

Dibuat saat tombol `Download & Proses` ditekan. Hanya baris dengan `repair100` valid yang diproses. File ini ditulis ulang oleh AHK dari payload `manifestText`.

```json
[
  {
    "fileName": "01 Lokasi 0+000 Non Lajur Ambulance",
    "tanggal": "2026-08-01 07:15:00",
    "no": "01",
    "latlng": "-7.49103360, 112.23335120",
    "url": "https://espm.bpjt.pu.go.id/storage/app/uploads/public/..."
  }
]
```

Field:

- `fileName`: nama file tanpa ekstensi.
- `tanggal`: tanggal untuk overlay.
- `no`: nomor urut PDF.
- `latlng`: koordinat overlay.
- `url`: URL gambar `repair100`.

## Payload Download `manifestText`

Payload aktif dari JavaScript ke AHK untuk proses download adalah teks tab-delimited, bukan array JSON besar. Setiap baris merepresentasikan satu gambar.

```text
fileName<TAB>tanggal<TAB>no<TAB>latlng<TAB>url
```

Contoh:

```text
01 Lokasi 0+000 Non Lajur Ambulance	2026-08-01 07:15:00	01	-7.49103360, 112.23335120	https://espm.bpjt.pu.go.id/storage/app/uploads/public/...
```

AHK membaca payload ini dengan `ParseDownloadManifestText()` menjadi array `Map`:

```ahk
Map(
  "fileName", parts[1],
  "tanggal", parts[2],
  "no", parts[3],
  "latlng", parts[4],
  "url", parts[5]
)
```

## Pesan JavaScript ke AHK

Request konfigurasi:

```json
{
  "action": "request_config"
}
```

Simpan konfigurasi:

```json
{
  "action": "config",
  "data": {}
}
```

Download:

```json
{
  "action": "download",
  "manifestText": "fileName\\ttanggal\\tno\\tlatlng\\turl\\n..."
}
```

Response AHK:

```json
{
  "ok": true,
  "action": "download"
}
```

## Pesan AHK ke JavaScript

Kirim konfigurasi:

```json
{
  "type": "config",
  "data": {}
}
```

Update kartu gambar dengan base64:

```json
{
  "action": "updateCard",
  "index": 1,
  "total": 10,
  "no": "01",
  "fileName": "01 Lokasi 0+000 Non Lajur Ambulance",
  "base64": "data:image/jpeg;base64,..."
}
```

Kirim hasil HTML eSPM:

```json
{
  "cmd": "espm_html",
  "count": 127,
  "html": "<div>...</div>"
}
```

## Master Validasi

Indikator:

- Bersumber dari `indikatorMap[*].keywords`.
- Dipakai untuk deteksi awal indikator PDF.
- Keyword sebaiknya diurutkan dari paling panjang ke paling pendek agar keyword umum tidak menangkap lebih dulu.

Jalur:

- `Jalur A`
- `Jalur B`
- `Non Jalur`

Lajur:

- `Non Lajur`
- `Bahu Luar`
- `Lajur 1`
- `Lajur 2`
- `Lajur 3`
- `Lajur 4`
- `Lajur 5`
- `Bahu Dalam`
- `Ramp`
- `Akses`
- `Lajur Motor`

Tag nama file:

- `id`
- `no`
- `indikator`
- `sta`
- `jalur`
- `lajur`
- `metode`
- `deskripsi`
- `durasi`
- `lokasi`
