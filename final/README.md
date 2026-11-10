# Final Round IFEST Data Analysis Competition 2026

> Prediksi risiko kekurangan air (_water stress_) bulan berikutnya per sub-DAS.

## 1. Ringkasan Tugas

Ketersediaan air ditentukan oleh pasokan dan penggunaan yang berubah dari waktu ke waktu.
Peserta membangun model _machine learning_ untuk peringatan dini: **apakah suatu sub-DAS (HUC12) akan mengalami _water stress_ bulan depan**, dari riwayat hidrologi + penggunaan air.

- Input: 1 baris = 1 sub-DAS pada 1 bulan _origin_, berisi jendela 12 bulan ke belakang (`lag0..lag11`).
- Output: skor risiko kontinu `0–1` (makin tinggi = makin berisiko).
- Model harus mengenali: pola musiman, kondisi hidrologis, kebutuhan air, dan hubungan antarwilayah dalam jaringan sungai, serta menggeneralisasi ke sub-DAS / periode baru.

## 2. Evaluasi

Metrik: **Average Precision (AP) / PR-AUC**.

```text
AP = sum_n (R_n - R_{n-1}) * P_n
```

- `P_n` = Precision pada threshold ke-n, `R_n` = Recall pada threshold ke-n.
- Dipilih karena biner tidak seimbang (positif ~20,4% di train). Kirim **probabilitas**, bukan label biner.
- AP hanya peduli **urutan** skor, bukan skala absolutnya.

## 3. Data

Lihat `data/data_dictionary.csv` untuk definisi lengkap.

| File                              | Isi                                                                                                                        |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `train.csv` (~378.780 baris)      | Fitur + `label` (`1` jika SUI target >= threshold). = 2.196 `id` x 168 `origin` berurutan.                                 |
| `test_public.csv` (~11.928 baris) | Fitur tanpa `label` + `row_id`. = 2.982 `id` x 4 `origin` (bulan origin 2, 4, 10, 12). ~73,6% `id` pernah muncul di train. |
| `sample_submission.csv`           | Contoh format submisi.                                                                                                     |

Fitur utama:

- Identitas / jaringan: `id`, `to_id` (`OUTLET` jika hilir), `origin_id` (acak, bukan urutan waktu), `month` (campuran angka + nama).
- Statik: `inc_aream2`, `cum_aream2`, `pop_dec_*`, `pop_acs_2020`.
- Dinamis 12 lag: `q_bf_inc` (baseflow), `q_qf_inc` (quickflow), `q_ws_cum` (supply kumulatif), `q_ir/th/ps_wd_inc` (withdrawal irigasi / termoelektrik / publik).
- Klimatologi: `clim_median_now` (bulan origin), `clim_median_next` (bulan target). Konstan per (`id`, bulan kalender).
- `aux_*`: kolom umpan / duplikat (lihat §4).

## 4. Jebakan Data (temuan notebook `notebooks/Final_DAC_2026_psi-1.ipynb`)

1. **Format angka campuran:** gaya AS `71,086,157.853` bercampur gaya Eropa `244668757,724`. Token ambigu seperti `244,668` diselesaikan via modus per `id` (statik) atau per (`id`, `month`) (klimatologi).
2. **Format bulan campuran:** `'1'..'12'` + nama bulan 3 varian penulisan -> dikanonkan ke `month_num`.
3. **`aux_*` adalah umpan:** `aux_constant_*` konstan, `aux_twin_unit_*` = kolom lag / 1000, `aux_fake_mm_*` = `inc_aream2` / 10000 + noise, `aux_junk_id_*` = `ref-XXXXXXXX`, `aux_noise_*` = noise terstandarisasi. Dibuang.
4. **Pergeseran skala test:** semua `q_*` + `clim_*` di test dikalikan satu faktor per `id` (kuantil 5–95%: 0,27–4,06, konstan dalam satu `id`). Nilai mentah dan lookup per-`id` dari train rusak; **hanya rasio dalam-baris yang selamat**. Solusi: 103 fitur invarian-skala (rasio klimatologi + bentuk jendela + fisika).
5. **`origin_id` teracak:** kronologi train direkonstruksi dari overlap lag (lag0..10 baris lama = lag1..11 baris baru) -> 168 bulan berurutan, `t=0` bulan 9 sampai `t=167` bulan 8. Test berada di periode yang tidak ada di train -> validasi harus **blok waktu** (`t >= 134`, purge 11 bulan), bukan split acak / per-`id`.
6. **Cold-start:** ~26% `id` test baru -> dilarang fitur memorizing `id`.
7. **Ketidakseimbangan kelas:** baseline 0,20. Fitur `1 - ws_lag0 / clim_now` saja sudah AP ~0,46.

## 5. Format Submisi

```csv
row_id,label
0,0.0142
1,0.7653
2,0.0031
```

- `row_id`: jangan diubah, tepat satu kali, urutan sama seperti `sample_submission.csv`.
- `label`: `0–1`, tanpa NaN.

## 6. Pendekatan Singkat (EXP-102)

`jendela 12 bulan + klimatologi now/next -> 103 fitur rasio -> LightGBM (600 pohon) + RealMLP (4x256, 8 epoch) -> rata-rata peringkat -> P(stress)`

- Validasi blok-waktu: LightGBM AP 0,7663, RealMLP 0,7573, Ensemble 0,7685.
- LightGBM kuat di origin Oktober, RealMLP kuat di Februari/Desember; korelasi peringkat ~0,92.
- Sengaja _underfit_ (lr kecil, epoch pendek, dropout 0,15, wd 0,05): model yang terlalu pas ke periode train anjlok di periode baru.

## 7. Cara Menjalankan

1. Ubah `Settings.DATA_DIR` / `OUTPUT_DIR` jika di Colab/Kaggle.
2. Jalankan `notebooks/Final_DAC_2026_psi-1.ipynb` dari awal sampai akhir (butuh GPU untuk RealMLP, CPU untuk LightGBM).
3. Output: `submission.csv` (`row_id,label`).

## 8. Aturan Lomba (Final DAC IFEST 2026)

### 8.1 Deliverables

- Source code notebook: `Final_DAC2026_NamaTim.ipynb`, wajib menyertakan seed untuk randomness.
- File PPT hasil analisis dataset baru yang disediakan panitia: `Presentasi_DAC2026_NamaTim`.
- Pengumpulan source code + PPT hanya oleh perwakilan tim.
- Notebook harus bisa di-run ulang dari awal sampai akhir tanpa error dengan output konsisten.

### 8.2 Presentasi final (Zoom)

- Presentasi hasil analisis di depan juri via Zoom Meeting dengan PPT yang sudah dikumpulkan.
- Durasi: 10 menit presentasi + 5 menit tanya jawab dewan juri.
- Keterlambatan tidak mendapat tambahan waktu.
- Tim yang tidak konfirmasi kehadiran sampai batas waktu dinyatakan didiskualifikasi.

### 8.3 Batasan pemodelan

- Dilarang dataset eksternal dalam bentuk apa pun selama analisis, latih, validasi, maupun pengembangan solusi.
- Dilarang pretrained model / checkpoint / weights / embeddings. Arsitektur standar/kompleks boleh dipakai asal seluruh bobot diinisialisasi acak dan dilatih sepenuhnya dari awal (*from scratch*).
- Dilarang pipeline AutoML penuh. Komponen Hyperparameter Optimization (HPO) seperti Optuna / Ray Tune / GridSearchCV diperbolehkan.
- Wajib kunci `random seed` di awal notebook untuk semua pustaka relevan (`random`, NumPy, scikit-learn, XGBoost, LightGBM, PyTorch, TensorFlow).

### 8.4 Sanksi & ketentuan umum

- Pelanggaran aturan dapat berujung diskualifikasi berdasarkan pemeriksaan dan keputusan panitia.
- Pengumuman juara dan pemberian hadiah pada tanggal yang ditentukan via email dan media publikasi IFEST 2026.
- Ketentuan teknis dapat berubah sewaktu-waktu dan diinformasikan via kanal resmi IFEST 2026.
