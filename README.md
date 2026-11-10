# Kompetisi Klasifikasi Kesesuaian Judul dan Isi Berita

## 1. Overview

Di tengah pesatnya arus informasi digital, judul berita menjadi bagian pertama yang digunakan pembaca untuk memahami suatu peristiwa. Oleh karena itu, judul seharusnya mampu merepresentasikan informasi utama dalam isi berita secara tepat. Ketidaksesuaian antara judul dan isi dapat menimbulkan kesalahpahaman, terlebih ketika keduanya membahas topik atau tokoh yang serupa tetapi merujuk pada peristiwa yang berbeda.

Mendeteksi ketidaksesuaian tersebut bukan sekadar mencari kesamaan kata. Dua teks dapat memuat banyak kata kunci yang sama, tetapi berbeda dalam penyebutan tokoh, lokasi, waktu, angka, tindakan, maupun konteks peristiwa. Sebaliknya, judul dan isi yang menggunakan susunan kata berbeda belum tentu tidak sesuai apabila tetap menyampaikan informasi yang sama.

Melalui kompetisi ini, peserta ditantang untuk mengembangkan model klasifikasi yang mampu memahami hubungan antara judul dan isi berita serta menentukan apakah keduanya **Sesuai** atau **Tidak Sesuai**.

## 2. Problem Statement

Peserta diminta membangun **model klasifikasi biner** berdasarkan pasangan judul dan isi berita. Untuk setiap pasangan pada data uji, model harus memprediksi salah satu dari dua label berikut:

- **Sesuai (1):** Informasi utama dalam judul didukung dan dibahas dalam isi berita.
- **Tidak Sesuai (0):** Informasi utama dalam judul tidak didukung oleh isi, membahas peristiwa yang berbeda, atau memiliki perbedaan konteks yang mengubah makna berita.

Model diharapkan tidak hanya mengandalkan kemiripan kata, tetapi juga mampu mengenali hubungan **kontekstual** antara kedua teks. Hal ini penting karena beberapa pasangan dapat memiliki topik, tokoh, lokasi, atau kata kunci yang serupa meskipun sebenarnya menyampaikan informasi yang berbeda.

Dataset kompetisi terdiri atas pasangan judul dan isi berita. Data training dilengkapi dengan label kesesuaian untuk melatih dan memvalidasi model, sedangkan data uji tidak memiliki label dan harus diprediksi oleh peserta.

## 3. Tujuan

Mendorong peserta untuk mengembangkan model klasifikasi teks yang akurat dan memiliki kemampuan generalisasi yang baik dalam menilai kesesuaian judul dengan isi berita. Peserta diharapkan mampu melakukan eksplorasi data, menerapkan teknik pemrosesan teks yang tepat, membangun representasi fitur yang informatif, serta memilih pendekatan pemodelan yang sesuai.

Model diharapkan mampu menangani beragam tingkat kesulitan, mulai dari pasangan dengan topik yang jelas berbeda hingga pasangan yang memiliki kemiripan tinggi.

## 4. Evaluation

Performa model dievaluasi menggunakan **Macro F1-Score** sebagai metrik utama.

Macro F1-Score menghitung F1-Score untuk setiap kelas, kemudian dirata-ratakan:

```
F1_macro = (F1_0 + F1_1) / 2
```

Nilai berada pada rentang 0 hingga 1. Nilai yang lebih tinggi menunjukkan performa klasifikasi yang lebih baik pada kedua kelas.

## 5. Submission File

Untuk setiap data pada test set, prediksi label kesesuaian judul-isi.

File submission harus berbentuk CSV dengan header dan dua kolom:

| Kolom   | Deskripsi                                                               |
| ------- | ----------------------------------------------------------------------- |
| `id`    | Identifier pada test set (contoh: `te00001` — format terbaru `teXXXXX`) |
| `label` | Hasil prediksi `1` (Sesuai) atau `0` (Tidak Sesuai)                     |

Contoh `submission.csv`:

```csv
id,label
te00000,1
te00001,0
te00002,1
```

Pastikan format submission sesuai dengan `sample_submission.csv` yang disediakan (`data/sample_submission.csv` — 3603 baris, `id` string `te00000`–`te03602`).

## 6. Dataset Description

Dataset terdiri dari pasangan judul dan isi berita yang diklasifikasikan ke dalam dua kategori:

- **Sesuai (1):** Informasi utama dalam judul didukung dan dibahas dalam isi berita.
- **Tidak Sesuai (0):** Informasi utama dalam judul tidak didukung oleh isi berita, merujuk pada peristiwa yang berbeda, atau memiliki perbedaan konteks yang mengubah makna berita.

Setiap pasangan dapat memiliki variasi dari segi topik, tokoh, lokasi, angka, waktu, dan konteks peristiwa. Beberapa judul dan isi juga dapat mengandung kata-kata yang serupa meskipun sebenarnya membahas informasi yang berbeda.

### Kolom Dataset

| Kolom     | Deskripsi                                                                  |
| --------- | -------------------------------------------------------------------------- |
| `id`      | Identitas unik (`tr00000`–`tr14396` di train, `te00000`–`te03602` di test) |
| `title`   | Judul berita yang akan dinilai kesesuaiannya                               |
| `content` | Isi / badan berita sebagai dasar penilaian                                 |
| `label`   | Target klasifikasi (`1` = Sesuai, `0` = Tidak Sesuai) — hanya di train     |

### Files (Dataset Update 2026)

- `train.csv` — 14.397 baris (12960 Sesuai, 1437 Tidak Sesuai) | `data/train.csv`
- `test.csv` — 3.603 baris tanpa label | `data/test.csv`
- `sample_submission.csv` — 3.603 baris `teXXXXX, label` | `data/sample_submission.csv`

| Info   | Detail                                                                                                                                                                                               |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Size   | ~38.98 MB                                                                                                                                                                                            |
| Type   | csv                                                                                                                                                                                                  |
| Update | Dataset lama `dataOld/` (14400 train / 3600 test, id integer) → dataset baru `data/` (14397/3603, id string). Perubahan utama: hard negative meningkat drastis (overlap >0.7 pada label 0: 49 → 954) |

> **Catatan perubahan:** Dibandingkan `dataOld/`, dataset baru menghapus 3 baris train label 0, menambah 3 baris test, mengubah format `id` integer → string (`tr`/`te`), serta meningkatkan kesulitan (mean overlap label 0 naik 0.339 → 0.749 — menuntut pemahaman kontekstual, bukan lexical overlap saja). Lihat `eda/`.

---

## 7. ⚠️ Aturan Tambahan — Update Resmi Panitia IFest 2026 (Rilis 21.00 WIB)

> **Pengumuman Panitia:**
> Halo @Data Analysis Competition! Sebelumnya, kami memohon maaf yang sebesar-besarnya atas keterlambatan pengiriman informasi ini. Dataset resmi untuk babak penyisihan Data Analysis Competition IFest 2026 akan dirilis malam ini pukul 21.00 WIB dan dapat diakses melalui channel ini.

### Larangan Penggunaan Pre-trained Model

**Peserta TIDAK diperbolehkan menggunakan pre-trained model dalam bentuk apa pun**, termasuk namun tidak terbatas pada:

- `BERT`, `GPT`, `LLM` (semua varian)
- `ResNet`, `VGG`, dan arsitektur vision/audio pre-trained lainnya
- Model machine learning / deep learning lain yang sudah dilatih sebelumnya (termasuk embeddings pre-trained seperti Word2Vec/GloVe/FastText pre-trained, IndoBERT, dll.)

Aturan ini diberlakukan untuk **menjaga keadilan (fairness)** serta menguji **pemahaman fundamental dan kreativitas peserta** dalam mengolah data secara mandiri.

### Konsekuensi Pelanggaran

> **Apabila terdapat ketidaksesuaian hasil dengan notebook maka PENILAIAN NOTEBOOK OTOMATIS 0.**

Panitia akan melakukan pengecekan kesesuaian antara `submission.csv` dengan notebook yang dikumpulkan. Hasil harus **reproducible** hanya dengan library standar / model yang dilatih from-scratch di dalam notebook.

### Implikasi untuk Pengerjaan

- Wajib membangun model **from scratch** (TF-IDF, BoW, Word2Vec yang dilatih sendiri di `train.csv`, Naive Bayes, Logistic Regression, SVM, Decision Tree, Random Forest, XGBoost/LightGBM from scratch, MLP sederhana tanpa bobot pre-trained, dll.)
- Diperbolehkan menggunakan library umum (`scikit-learn`, `pandas`, `numpy`, `nltk`, `Sastrawi`) **selama tidak memuat bobot pre-trained eksternal**.
- Dilarang `transformers.AutoModel.from_pretrained`, `sentence-transformers`, `gensim.downloader.load`, atau API LLM eksternal untuk ekstraksi fitur/inferensi.

## 8. Aturan Tambahan — Klarifikasi Panitia atas Metode Pemodelan dan Pelabelan

Halo <@&1532002202296778762> !

Menanggapi beberapa pertanyaan mengenai metode pemodelan dan pelabelan data, berikut ketentuan yang perlu diperhatikan:

**1. Apakah Hyperparameter Optimization (HPO) diperbolehkan?**

Ya. Peserta diperbolehkan melakukan **Hyperparameter Optimization (HPO)** menggunakan Optuna maupun fitur HPO yang tersedia dalam framework AutoML.

Namun, penggunaan AutoML **hanya diperbolehkan pada komponen optimasi hyperparameter**. Fitur AutoML lainnya tidak diperbolehkan.

**2. Apakah penggunaan data eksternal diperbolehkan?**

Ya. Peserta diperbolehkan menggunakan data eksternal sebagai tambahan data latih. Namun, seluruh model tetap harus dilatih dari awal. Penggunaan **pretrained model, pretrained weights, maupun proses pretraining** tidak diperbolehkan.

**3. Apakah pelabelan data diperbolehkan?**

Ya. Pelabelan atau perbaikan label hanya diperbolehkan pada data latih, yaitu data train resmi maupun data eksternal yang digunakan sebagai tambahan data latih.

Sebagai contoh, peserta diperbolehkan melakukan pseudo-labeling pada data eksternal yang belum memiliki label, kemudian menggunakannya sebagai tambahan data latih. Namun, pseudo-labeling pada data test resmi tidak diperbolehkan. Peserta tidak boleh menggunakan prediksi model sebagai label sementara pada data test, lalu memasukkan kembali data tersebut ke dalam proses training, validasi, pemilihan model, atau fine-tuning.

Prediksi pada data test hanya diperbolehkan untuk menghasilkan file submission dan tidak boleh digunakan kembali dalam pengembangan model.

Selamat melanjutkan pengerjaan dan tunjukkan hasil terbaik dari tim kalian!

## 9. 📢 **[PENEGASAN COMPETITION RULES DAC IFEST 2026]**

Halo, teman-teman <@&1532002202296778762> 👋🏻

Menindaklanjuti beberapa pertanyaan yang masuk, panitia telah memperjelas **Competition Rules** pada halaman Kaggle, khususnya pada poin 4, 7, dan 8:

4. Penggunaan model, checkpoint, bobot, atau embedding pretrained, seperti IndoBERT, BERT, XLM-R, dan FastText pretrained, tidak diperbolehkan. Arsitektur tersebut tetap boleh digunakan apabila seluruh bobot diinisialisasi secara acak dan dilatih sepenuhnya dari awal.

5. Peserta diperbolehkan menggunakan lebih dari satu model atau metode ensemble. Penggunaan AutoML tidak diperbolehkan, kecuali komponen yang hanya digunakan untuk Hyperparameter Optimization (HPO), seperti Optuna.

6. Pelabelan, perbaikan label, dan augmentasi hanya diperbolehkan pada data latih. Data test hanya boleh digunakan pada tahap inferensi untuk menghasilkan submission dan tidak boleh dilabeli secara manual, dijadikan pseudo-label, maupun digunakan untuk training, fitting, validasi, atau tuning. Upaya memperoleh label asli data test melalui kebocoran atau sumber yang tidak diperbolehkan termasuk rekonstruksi label dan dilarang.

Penyesuaian ini merupakan **penegasan terhadap aturan yang telah berlaku**. Mohon seluruh peserta membaca kembali bagian **Rules** pada halaman Kaggle dan memastikan metode yang digunakan telah sesuai.

Terima kasih atas perhatian dan kerja samanya. Semangat melanjutkan pengerjaan! 🚀

## 10. Pendekatan yang Diimplementasikan

### Babak Penyisihan — Kesesuaian Judul dan Isi Berita

Notebook utama ada di [`penyisihan/notebooks/psi-1_notebook.ipynb`](penyisihan/notebooks/psi-1_notebook.ipynb). Pipeline membersihkan pasangan judul–isi, membuat fitur kesamaan leksikal dan semantik (termasuk entitas, angka, negasi, penyelarasan, dan *retrieval*), serta memakai leksikon eksternal yang disebutkan dalam notebook. Sebuah *cross-encoder* GRU dilatih dari awal; keluaran model ini digabung dengan fitur lain oleh **LightGBM, XGBoost, dan CatBoost**, masing-masing dengan dua *seed*. Ambang klasifikasi dipilih dari prediksi *out-of-fold* untuk memaksimalkan Macro F1, lalu diterapkan pada prediksi data uji. Model pralatih tidak digunakan sesuai ketentuan lomba.

### Babak Final — Prediksi Risiko Kekurangan Air

Permasalahan final **berbeda dari penyisihan**: memprediksi apakah sub-DAS (HUC12) akan mengalami *water stress* pada bulan berikutnya. Data final resmi terdiri atas sekitar **378.780** baris latih (riwayat 12 bulan per baris), **11.928** baris uji, dan contoh submisi. Fitur mencakup aliran dan suplai air, penggunaan air, klimatologi, atribut DAS, serta jaringan sungai; target `label` bernilai 1 jika terjadi *water stress*. Metriknya **Average Precision (AP)**, dan keluaran yang diminta adalah skor risiko `0–1` per `row_id`.

[`final/notebooks/Final_DAC_2026_psi-1.ipynb`](final/notebooks/Final_DAC_2026_psi-1.ipynb) mem-parsing angka dengan format campuran, memperbaiki kesalahan satuan klimatologi, dan membangun fitur rasio yang tahan terhadap pergeseran skala data uji. Empat anggota ensemble—**LightGBM** serta tiga **RealMLP** dengan target ordinal berbeda—dilatih menggunakan beberapa *seed*, kemudian probabilitasnya dirata-ratakan. Rincian dataset, validasi berdasarkan waktu, aturan model, dan format submisi dijelaskan di [`final/README.md`](final/README.md).

**Ketersediaan data:** berkas `data/train.csv`, `data/test.csv`, `data/sample_submission.csv`, leksikon eksternal penyisihan, dan dataset final **tidak ada dalam salinan ini maupun folder sumber yang diperiksa**. Kedua notebook memerlukan data tersebut; notebook final juga mengacu pada lokasi input Kaggle. Uraian data penyisihan lebih lanjut tersedia di [`DATA.md`](DATA.md).
