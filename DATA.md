# Kondisi Data — DAC IFEST 2026 Tim psi-1

Dokumen ini menjelaskan kondisi dataset lomba dan dataset eksternal yang dipakai pipeline HIDRA secara komprehensif. Angka di bawah diverifikasi langsung dari berkas di `data/` dan laporan di `eda/eda_report.md`.

---

## 1. Dataset Lomba

### 1.1 Berkas dan ukuran

| Berkas                       | Baris  | Kolom                       | Ukuran  |
| ---------------------------- | ------ | --------------------------- | ------- |
| `data/train.csv`             | 14.397 | `id, title, content, label` | ~31 MB  |
| `data/test.csv`              | 3.603  | `id, title, content`        | ~7,6 MB |
| `data/sample_submission.csv` | 3.603  | `id, label`                 | ~39 KB  |

Format `id` berupa string yaitu `tr00000`–`tr14396` untuk train dan `te00000`–`te03602` untuk test. Format CSV bersih yaitu tidak ada koma, kutip, atau newline di dalam field dan tanda baca pada teks sudah di-strip panitia.

### 1.2 Distribusi label

| Label          | Jumlah | Proporsi |
| -------------- | ------ | -------- |
| 1 Sesuai       | 12.960 | 90,0%    |
| 0 Tidak Sesuai | 1.437  | 10,0%    |

Rasio imbalance 9 berbanding 1. Model naive yang memprediksi semua Sesuai hanya mendapat Macro F1 sekitar 0,47 sehingga penanganan imbalance bersifat wajib. Pipeline menanganinya lewat class weight balanced, scale pos weight dan threshold tuning pada OOF.

### 1.3 Kualitas data

Tidak ada nilai hilang pada kolom kunci train maupun test. Tidak ada duplikat `id`. Terdapat 3 baris train dengan pasangan judul dan isi yang identik persis. Sampel ini tidak mengganggu pemodelan tetapi menjadi alasan grouping pada validasi agar pasangan identik tidak terbelah antar fold.

### 1.4 Panjang teks

| Teks  | Rata-rata kata | Median kata | Rentang |
| ----- | -------------- | ----------- | ------- |
| Judul | ~10            | 10          | 3–23    |
| Isi   | ~310           | 276         | 3–4238  |

Rata-rata per label hampir identik sehingga panjang saja bukan sinyal pembeda. Pencilan isi yang sangat panjang ada pada kedua kelas dan juga pada test. Dampak ke pipeline yaitu isi dipotong pada batas aman (1200 karakter untuk TF-IDF dan 200–400 token untuk cross encoder) dan fitur panjang hanya dipakai sebagai pendukung.

### 1.5 Overlap judul ke isi dan hard subset

Overlap dihitung sebagai Jaccard berbasis himpunan kata antara judul dan isi.

| Label          | Mean overlap | Median overlap |
| -------------- | ------------ | -------------- |
| 1 Sesuai       | 0,796        | 0,80           |
| 0 Tidak Sesuai | 0,749        | 0,75           |

Selisih mean yang tipis membuat classifier berbasis overlap murni gagal. Dua subset sulit menjadi tantangan utama.

- Hard negative yaitu label 0 dengan overlap di atas 0,7 berjumlah 954 baris atau sekitar 66% dari kelas Tidak Sesuai. Polanya adalah judul dan isi memakai banyak kata sama tetapi entitas atau angka ditukar sehingga makna berubah total.
- Hard positive yaitu label 1 dengan overlap di bawah 0,5 berjumlah 278 baris atau sekitar 2% dari kelas Sesuai. Polanya adalah parafrase dan sinonim.

Dibanding dataset lama, mean overlap label 0 naik dari 0,339 menjadi 0,749 sehingga tingkat kesulitan naik drastis dan menuntut pemahaman kontekstual.

### 1.6 Kata dominan

Judul kedua label didominasi kata corona, covid, positif, kasus dan vaksin. Domain Covid mencakup sekitar 30% judul. Skor TF-IDF rata-rata per kelas tidak menunjukkan pemisahan kuat. Dampak ke pipeline yaitu model dilarang menghafal kata kunci Covid dan dibekali fitur entitas, sinonim serta Word2Vec yang dilatih sendiri.

### 1.7 Train vs test

Distribusi panjang judul, panjang isi dan overlap pada test mirip train. Mean overlap test 0,787 berbanding train 0,788 sehingga tidak ada pergeseran besar. Rentang `id` terpisah dan test tidak berlabel. Asumsi i.i.d. dianggap wajar tetapi pipeline tetap memakai StratifiedGroupKFold agar validasi tidak bocor lewat judul atau isi yang identik.

### 1.8 Contoh pola

- Sesuai biasanya berupa judul yang hampir verbatim dengan kalimat awal isi.
- Tidak Sesuai yang mudah berupa topik yang jelas melompat.
- Hard negative berupa kata sama dengan entitas atau angka yang ditukar.

---

## 2. Dataset Eksternal

Seluruh dataset eksternal bersifat publik, gratis, open-source dan bukan ground truth yaitu tidak mengandung label test. Rincian lisensi dan cara unduh ulang ada di `data/external_data/README.md` dan dideklarasikan di markdown teratas notebook sesuai ketentuan guidebook.

| File               | Isi                          | Sumber                                               | Lisensi               | Kegunaan di pipeline                                                                                              |
| ------------------ | ---------------------------- | ---------------------------------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `stopwords-id.txt` | 772 stopword Indonesia       | stopwords-iso di GitHub                              | MIT / CC              | Overlap tanpa stopword agar kata konten lebih menonjol                                                            |
| `provinces.csv`    | 34 provinsi                  | Wilayah-Administratif-Indonesia, sumber BPS MFD 2018 | MIT skrip / ODbL data | Gazetteer lokasi untuk fitur loc overlap dan loc mismatch                                                         |
| `regencies.csv`    | 514 kabupaten/kota           | Sama dengan di atas                                  | Sama dengan di atas   | Perluas gazetteer ke level kabupaten/kota                                                                         |
| `districts.csv`    | 7215 kecamatan               | Sama dengan di atas                                  | Sama dengan di atas   | Perluas gazetteer ke kecamatan, cakupan judul naik 47% ke 69%. Kata arah wajib difilter agar tidak false mismatch |
| `id-names.txt`     | 1991 nama, 1742 token unik   | perlancar/references di GitHub                       | Publik                | Leksikon nama orang untuk fitur person mismatch misalnya tukar tokoh                                              |
| `kbba.txt`         | Kamus slang ke formal ~23 KB | insomniagung/kamus_kbba                              | Publik                | Normalisasi tak ke tidak, jkt ke jakarta dan jubir ke juru bicara sebelum overlap                                 |
| `tesaurus.json`    | 20.139 entri ~4,9 MB         | Tesaurus Bahasa Indonesia Pusat Bahasa 2008          | Publik                | Fitur synonym hit untuk membedakan parafrase label 1 misalnya pecah rekor vs rekor tertinggi                      |

Catatan kepatuhan.

- Semua leksikon dimuat hanya dari kosakata train sehingga tidak ada kebocoran dari test.
- Tidak ada bobot pre-trained dalam bentuk apapun. Semua murni data mentah dan leksikon teks yang dilatih from scratch di `train.csv`.
- `clickid_raw.csv` bersifat opsional dan manual yaitu 46.119 artikel mentah se-domain untuk fit IDF saja bukan baris latih. Tanpa berkas ini pipeline tetap jalan normal.
- `indonli_train/val.jsonl` sempat di-fetch lalu dihapus karena augmentasi tidak membantu akibat domain shift yaitu premis 1–2 kalimat versus isi 300 kata. Pelajarannya adalah data bantu harus se-domain dan se-bentuk.

---

## 3. Peta berkas terkait

- `eda/eda_report.md` dan `eda/figures/` yaitu laporan dan visual EDA.
- `scripts/eda.py` yaitu kode sumber EDA yang di-port ke notebook.
- `notebooks/psi-1_notebook.ipynb` yaitu pipeline final end-to-end.
- `outputs/` dan `artifact/` yaitu hasil generate ulang dan tidak di-track git.
