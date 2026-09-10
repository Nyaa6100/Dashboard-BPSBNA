# 📊 Dashboard Statistik Daerah — BPS Banjarnegara

Dashboard interaktif berbasis **Streamlit** bertema **aksen resmi BPS**
(biru tua `#0A3D6E` + aksen merah `#E23A34`) dengan logo resmi BPS tertanam.

## ✨ Fitur

| Fitur | Keterangan |
|---|---|
| 🛡️ Logo BPS | Tertanam di sidebar & banner (SVG, tanpa dependensi eksternal saat runtime) |
| 🔔 Kartu fokus dinamis | Mengikuti indikator yang dipilih pengguna + perbandingan otomatis vs tahun sebelumnya |
| 🗂️ Peta 21 indikator | Kartu ringkas seluruh indikator terpantau; yang belum terisi tampil redup *"belum tersedia"* |
| 📈 Grafik tren | Interaktif (Plotly), garis/batang |
| 📍 Wilayah terkunci | Data ditampilkan untuk **Kabupaten Banjarnegara** saja (opsi pemilihan wilayah dihapus) |
| 🎯 Gauge Kategori IPM | IPM L/P vs ambang kategori resmi BPS (60 Sedang · 70 Tinggi · 80 Sangat Tinggi); tampil saat indikator IPM dipilih |
| ⚖️ Dual-axis Kemiskinan | Batang % penduduk miskin vs garis garis kemiskinan Rp (1996–2025); tampil saat varian *Tingkat Kemiskinan* dipilih |
| ⚫ Scatter Ekonomi–Pengangguran | Sebaran tahunan Pertumbuhan Ekonomi vs TPT dengan garis rata-rata tiap sumbu |
| 🥧 Komposisi penduduk | Pie *penduduk miskin vs sejahtera* dari Persentase Penduduk Miskin tahun terbaru dalam rentang; legenda hanya menampilkan **Penduduk Miskin**. Seksi ini tampil **hanya saat indikator *Garis Kemiskinan* dipilih** di sidebar |
| 🧾 Tabel rinci | Format angka Indonesia + unduh CSV |
| ⚡ Sinkron otomatis | Setiap sesi aplikasi dimulai, data terbaru diambil dari berkas induk; bila jaringan gagal dipakai salinan lokal `data/` |

Antarmuka tidak menampilkan elemen konektor maupun keterangan sumber data.

## 🚀 Cara Menjalankan

```bash
pip install -r requirements.txt   # sekali saja
streamlit run app.py
```

Browser terbuka otomatis di `http://localhost:8501`.

## ⚡ Alur Pembaruan Data (untuk pegawai)

1. Tambah/ubah baris tahun baru pada **berkas spreadsheet induk**.
2. Buka (atau muat ulang) dashboard — data terbaru **otomatis** diambil dan
   disalin ke folder `data/`.
3. Tanpa internet? Dashboard memakai salinan lokal terakhir secara transparan.

Tidak ada tombol atau langkah sinkronisasi manual; tidak ada kode yang perlu
diubah untuk menambah periode baru.

## 🗂️ Struktur Proyek

```
Dashboard/
├─ app.py                  # aplikasi Streamlit (UI)
├─ data_loader.py          # parser XLSX + pemetaan 21 indikator kanonik
├─ requirements.txt
├─ .streamlit/config.toml  # tema aksen BPS
├─ assets/
│  └─ lambang_bps.svg      # logo resmi BPS (Wikimedia Commons)
├─ data/                   # hanya file kanonik yang dimuat aplikasi
│  ├─ Data_BPS.xlsx        #   salinan lokal berkas induk
│  ├─ bps_api_tambahan.xlsx#   hasil panen WebAPI BPS (indikator tambahan)
│  ├─ _live_snapshot.xlsx  #   snapshot sinkronisasi live (dibuat otomatis)
│  ├─ _duplikat_api/       #   hasil panen API yang duplikat (tidak dimuat)
│  ├─ _arsip_scrap/        #   arsip hasil scraping/API lama (tidak dimuat)
│  └─ _versi_lama/         #   versi data lama (tidak dimuat)
└─ tools/
   ├─ inspect_data.py      # inspeksi struktur workbook
   ├─ check_loader.py      # uji parser + assert nilai contoh + kanonik
   └─ fetch_logo.py        # unduh ulang logo bila diperlukan
```

### Anti-duplikasi & prioritas file

Aplikasi memuat `FILE_PRIORITY` = [`_live_snapshot.xlsx`, `Data_BPS.xlsx`,
`bps_api_tambahan.xlsx`]. Data live **selalu digabung** dengan file lokal —
bukan menggantikannya. Bila ada indikator+tahun yang sama:

1. Baris **identik lintas-file dibuang** — angka tidak lagi terhitung dobel.
2. **Konflik nilai** diselesaikan deterministik: baris yang bernilai menang,
   sisanya mengikuti urutan prioritas (snapshot live lebih baru → menang).

File hasil scraping/API lama tidak ikut dimuat karena parsing-nya menggabungkan
beberapa varian ke satu nama indikator, sehingga satu tahun punya banyak nilai
sekaligus (grafik zigzag, KPI salah ambil).

## 🔌 Panen Data WebAPI BPS (`tools/bps_api_harvest.py`)

Key API disimpan di `bps_api.json` (domain `3304` Banjarnegara). Temuan penting:

- Model dinamis `data` v1 mengembalikan konten kosong untuk domain ini, dan
  WebAPI v2 menolak key pengguna (`ERRAUTH002` — khusus sistem internal BPS).
- Jalur yang berfungsi: `list/view statictable` v1 → unduh Excel resmi →
  konversi wide→long. File `.xls` harus dikonversi `.xlsx` terlebih dahulu.

Indikator hasil panen (masuk `data/bps_api_tambahan.xlsx`):

| Indikator | Sumber | Cakupan |
|---|---|---|
| Tingkat Pengangguran Terbuka | static table 198 | 2007–2019 |
| Tingkat Partisipasi Angkatan Kerja | static table 200 | 2007–2019 |
| Pertumbuhan Ekonomi | static table 156 (Seri 2010, baris PDRB) | 2011–2019 |
| Inflasi Kota Purwokerto | static table 197 (laju Desember y-o-y) | 2014–2018 |
| Laju Pertumbuhan Penduduk | static table 211 (kabupaten + per kecamatan, prefiks `Kecamatan`) | 2019 |
| Rata-rata Pengeluaran per Kapita *(bonus)* | static table 8 | 2009–2019 |

Tidak tersedia via API untuk domain ini: Gini Ratio, Indeks Kemahalan
Konstruksi, Pendapatan Perkapita, Rasio Ekspor & Investasi terhadap PDRB
(variabel dinamisnya ada tetapi datanya kosong; tidak ada static table-nya).

Hasil panen yang ternyata duplikat terhadap data dimuat otomatis dipisahkan ke
`data/_duplikat_api/`. Untuk memuat ulang: `python tools/bps_api_harvest.py`.

## 📐 Format Data yang Didukung

Long format per baris (header opsional, nama kolom nilai bebas):

| Wilayah      | Indikator                          | Tahun | Nilai    |
|--------------|------------------------------------|-------|----------|
| Banjarnegara | Garis Kemiskinan (Rp/kapita/bln)   | 1996  | 32.917   |
| Banjarnegara | Persentase Penduduk Miskin (persen)| 1999  | 52,38    |

Aturan pembacaan otomatis:

- Satu sheet = satu topik indikator; nama sheet dipakai bila kolom indikator
  tidak ada.
- Dimensi tambahan (**Jenis Kelamin**) dilebur ke nama indikator →
  *Indeks Pembangunan Manusia (Laki-laki)*.
- Angka gaya Indonesia: `32.917` = 32917 · `52,38` = 52.38 · `437.8` = 437.8.
- Sel kosong / `-` menjadi *missing* (grafik putus, tidak error).
- Sheet kosong & baris sisa diabaikan.

### Peta 21 Indikator Kanonik

Urutan kartu mengikuti daftar indikator resmi (lihat `CANONICAL` di
`data_loader.py`). Parser memetakan hasil bacaan sheet ke nama kanonik lewat
`canon_key()` — termasuk varian dengan dua mode kartu: **IPM L/P** tampil
berdampingan dalam satu kartu (*Laki-laki* kiri, *Perempuan* kanan), sedangkan
kartu *Tingkat Kemiskinan* memakai **toggle ⇄ 2 varian** antara Garis
Kemiskinan dan % Penduduk Miskin. Kartu KPI *Periode Tersedia* menyesuaikan
rentang tahun indikator yang dipilih.

## 🎨 Palet Warna

| Peran | Warna |
|---|---|
| Biru utama BPS | `#0A3D6E` |
| Biru sekunder | `#1976C5` |
| Aksen merah logo | `#E23A34` |
| Hijau (turun = baik) | `#1F9D66` |
| Amber (garis rata-rata) | `#F2A93B` |

## 🔧 Pemecahan Masalah

- **"Belum ada data yang dapat dimuat"** → buka dengan koneksi internet
  tersedia agar salinan lokal pertama berhasil dibuat, atau letakkan berkas
  `.xlsx` di folder `data/`.
- **Angka aneh** → parser mengikuti konvensi Indonesia (titik ribuan,
  koma desimal); pastikan sel angka tidak bercampur teks lain.
- **Logo hilang** → jalankan `python tools/fetch_logo.py`.
