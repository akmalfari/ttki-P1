# Evaluasi Generalisasi Lintas-Domain dan Trade-off Kinerja–Efisiensi Deteksi URL Phishing melalui Seleksi Fitur ANOVA

**Evaluating Cross-Domain Generalization and Performance–Efficiency Trade-offs in Phishing URL Detection Using ANOVA Feature Selection**

## Identitas penelitian

- Kode proyek: P1
- Andika Zafka Ujfaluzi
- Akmal Zaida Fari
- Program Studi Teknik Informatika
- Politeknik Elektronika Negeri Surabaya (PENS)
- Proyek TTKI Semester 5, 2026-1

## Tujuan penelitian

Mengevaluasi pengaruh kelompok fitur, strategi pembagian
data berbasis registered domain, dan seleksi fitur ANOVA
terhadap performa serta waktu inferensi model deteksi
URL phishing pada dataset PhiUSIIL.

Model yang digunakan adalah Logistic Regression (LR)
dan Random Forest (RF).

### Research question

Bagaimana pemilihan fitur URL-only dan validasi berbasis
kelompok domain memengaruhi F1, ROC-AUC, dan waktu
inferensi model deteksi phishing?

### Hipotesis

| Hipotesis | Pernyataan | Perbandingan |
|---|---|---|
| H1 | Hasil pengujian domain-grouped lebih rendah daripada random split | B vs C |
| H2 | Subset fitur URL-only berpotensi mengurangi biaya inferensi dengan kompromi performa | A vs B |

Perbandingan C vs D merupakan analisis tambahan H2
untuk mengevaluasi pengurangan fitur menggunakan ANOVA.

## Dataset

| Komponen | Keterangan |
|---|---|
| Nama | PhiUSIIL Phishing URL Dataset |
| Tanggal unduh | 12 November 2026 |
| Jumlah observasi | 235795 |
| Jumlah kolom | 56 |
| Legitimate | 134850 |
| Phishing | 100945 |
| Label asli | 0 = phishing, 1 = legitimate |
| Target eksperimen | y = 1 - label |
| Lisensi sumber asli | CC BY 4.0 |

Sumber dataset:

- Kaggle: https://www.kaggle.com/datasets/kaggleprollc/phishing-url-websites-dataset-phiusiil
- Dataset asli: https://doi.org/10.17632/shwpxscxy2.2
- Paper dataset: https://doi.org/10.1016/j.cose.2023.103545

### Integritas dataset

File yang digunakan:

`data/PhiUSIIL_Phishing_URL_Dataset.csv`

SHA-256 snapshot LF aktual:

`8e7e0331987e0f4dee0ca29df3b04e1913c19f2d60802c9a929c3e02aed2ff9a`

SHA-256 snapshot CRLF ekuivalen:

`a236549cd369cd80bd478ff8e1779cbf44c58d5c3f79f7a51a1adbed7d06d1c6`

Ekuivalensi kedua snapshot telah diperiksa melalui
konversi akhir baris.

| Hasil audit | Jumlah |
|---|---:|
| Missing cells | 0 |
| Duplikasi seluruh kolom | 0 |
| URL unik | 235370 |
| Kelompok URL berulang | 425 |
| Baris dengan URL berulang | 850 |
| Konflik label URL | 0 |

Tidak dilakukan deduplikasi atau perubahan label.

## Rancangan eksperimen

| Skenario | Strategi split | Kelompok fitur | Jumlah |
|---|---|---|---:|
| A | Random stratified | URL + halaman | 46 |
| B | Random stratified | URL-only | 18 |
| C | Registered-domain-grouped | URL-only | 18 |
| D | Registered-domain-grouped | URL-only + ANOVA | 5 |

Skenario A dan B menggunakan indeks split yang sama.
Skenario C dan D juga menggunakan indeks split yang sama.

### Konfigurasi evaluasi

- Random seed: 42.
- Development set: 188636 observasi.
- Final test set: 47159 observasi.
- Cross-validation: 5 fold pada development set.
- Random protocol: StratifiedKFold.
- Grouped protocol: StratifiedGroupKFold.
- Threshold klasifikasi: 0.5.
- Imputasi: median.
- Scaling: StandardScaler untuk LR.
- Seleksi fitur D: SelectKBest dengan f_classif, k=5.

Imputasi, scaling, dan seleksi fitur dilakukan di dalam
pipeline dan di-fit hanya menggunakan data latih.

Konfigurasi model dan daftar fitur tersedia pada
`configs/experiment_config.json`.

### Audit registered domain

Ekstraksi domain menggunakan tldextract 5.4.0 dengan
suffix_list_urls=() dan include_psl_private_domains=True.

Public Suffix List menggunakan snapshot bawaan package.

Jumlah kelompok registered domain: 197709.

| Pemeriksaan development/test | Random | Grouped |
|---|---:|---:|
| Overlap indeks baris | 0 | 0 |
| Overlap URL identik | 155 | 0 |
| Overlap registered domain | 3462 | 0 |

Seluruh fold grouped juga memiliki overlap URL dan
registered domain sebesar nol.

Audit menggunakan indeks eksperimen yang tersimpan,
bukan pembagian data yang dibuat ulang secara terpisah.

## Hasil eksperimen

### Cross-validation dan final test

| Skenario | Model | F1 CV (mean ± SD) | F1 test | ROC-AUC test |
|---|---|---|---:|---:|
| A | LR | 0.9990 ± 0.0001 | 0.999306 | 0.999992 |
| A | RF | 0.9998 ± 0.0001 | 0.999926 | 1.000000 |
| B | LR | 0.9953 ± 0.0004 | 0.995799 | 0.998301 |
| B | RF | 0.9967 ± 0.0002 | 0.997171 | 0.998488 |
| C | LR | 0.9952 ± 0.0002 | 0.995398 | 0.998520 |
| C | RF | 0.9966 ± 0.0005 | 0.997220 | 0.998407 |
| D | LR | 0.9839 ± 0.0009 | 0.985052 | 0.997074 |
| D | RF | 0.9936 ± 0.0006 | 0.994204 | 0.997853 |

Kelas positif adalah phishing.

Metrik lengkap, termasuk precision, recall, confusion
matrix, dan hasil setiap fold, tersedia pada folder
results.

### Paired timing benchmark

Waktu diukur menggunakan batch 1000 observasi,
3 warm-up, dan 30 pengulangan.

Batch identik digunakan untuk skenario dalam protokol
split yang sama. Batch berasal dari development set.

Satuan tabel: milidetik per batch 1000 observasi.

| Model | Thread mode | A | B | C | D |
|---|---|---:|---:|---:|---:|
| LR | Default | 1.850 | 0.580 | 0.572 | 0.673 |
| LR | Single-thread | 1.614 | 0.582 | 0.568 | 0.672 |
| RF | Default | 35.444 | 24.170 | 24.101 | 25.633 |
| RF | Single-thread | 15.122 | 15.869 | 15.731 | 19.791 |

Nilai merupakan median waktu predict_proba beserta
transformasi pipeline pada fitur yang sudah tersedia.

Waktu tidak mencakup ekstraksi fitur, pengunduhan
halaman, atau komunikasi jaringan.

Timing awal dipertahankan sebagai catatan historis.
Interpretasi efisiensi menggunakan paired timing.

### Interpretasi hipotesis

H1 tidak didukung secara konsisten. Perubahan F1 dan
ROC-AUC dari B ke C memiliki arah yang berbeda
antar model dan metrik.

H2 didukung sebagian secara deskriptif. Penggunaan
URL-only menurunkan F1 dan ROC-AUC kedua model.
Median waktu inferensi berkurang pada tiga dari empat
kombinasi model dan mode thread.

Seleksi ANOVA dari 18 menjadi 5 fitur mengurangi
jumlah fitur sebesar 72.22%, tetapi menurunkan
performa dan tidak mempercepat median inferensi
pada seluruh kombinasi yang diuji.

Tidak dilakukan pengujian signifikansi statistik
atau pengujian ekuivalensi.

### Fitur terpilih ANOVA

Kedua model memilih lima fitur berikut pada
development set penuh:

- LetterRatioInURL
- DegitRatioInURL
- NoOfOtherSpecialCharsInURL
- SpacialCharRatioInURL
- IsHTTPS

Nama kolom mengikuti dataset asli.

Pipeline D masih menerima 18 fitur sebelum
transformasi seleksi menjadi 5 fitur.

## Keterbatasan penelitian

- Seluruh 134850 URL legitimate menggunakan HTTPS
  dan merupakan homepage berdasarkan definisi audit.
- Terdapat 187151 ketidaksesuaian URLLength terhadap
  panjang string URL dan 493 ketidaksesuaian IsHTTPS
  terhadap skema URL.
- Penyebab ketidaksesuaian tersebut belum dipastikan.
- Eksperimen menggunakan fitur bawaan dataset,
  bukan ekstraksi ulang seluruh fitur dari URL mentah.
- Perbedaan karakteristik antar kelas berpotensi
  menimbulkan dataset-specific shortcuts, tetapi
  bukan bukti langsung target leakage.
- Domain-grouped validation tidak menghilangkan
  seluruh kemungkinan bias dataset.
- Evaluasi eksternal dan temporal belum dilakukan.
- Hasil benchmark tidak membuktikan performa
  deteksi phishing pada kondisi operasional nyata.

Penelitian dilakukan secara offline tanpa mengunjungi
URL phishing.

## Struktur repositori

```text
.
├── configs/
│   └── experiment_config.json
├── data/
│   └── PhiUSIIL_Phishing_URL_Dataset.csv
├── notebooks/
│   ├── 01_data_audit.ipynb
│   ├── 02_baseline.ipynb
│   ├── 03_experiments.ipynb
│   └── 04_final_test_and_timing.ipynb
├── references/
│   └── literature_matrix.csv
├── results/
│   ├── data_audit.csv
│   ├── split_audit.csv
│   ├── split_indices_random.npz
│   ├── split_indices_grouped.npz
│   ├── metrics_per_fold.csv
│   ├── all_scenarios_cv_summary.csv
│   ├── selected_features.csv
│   ├── test_final_results.csv
│   ├── test_final_selected_features_D.csv
│   ├── final_table_cv_vs_test.csv
│   ├── timing_results.csv
│   ├── timing_environment.json
│   ├── timing_results_paired.csv
│   ├── timing_environment_paired.json
│   ├── timing_batch_indices_paired.npz
│   └── plots_h1h2h3_paired.png
├── environment-freeze.txt
├── environment-freeze-paired.txt
├── requirements.txt
└── README.md
```

Nama file grafik dipertahankan untuk kompatibilitas.
Isi grafik menggunakan H1, H2, dan analisis ANOVA.

## Environment dan reproducibility

Environment paired timing:

| Komponen | Versi |
|---|---|
| Python | 3.12.3 |
| NumPy | 2.5.5 |
| pandas | 3.1.1 |
| scikit-learn | 1.9.1 |
| tldextract | 5.4.0 |
| Matplotlib | 3.11.2 |
| threadpoolctl | 3.8.0 |
| joblib | 1.6.0 |
| Logical CPU | 4 |
| Platform | Linux x86_64 |

Dependency utama tersedia pada requirements.txt.

Snapshot lengkap environment paired timing tersedia
pada environment-freeze-paired.txt.

Environment eksperimen awal tidak memiliki snapshot
dependency lengkap. Versi package utama pada metadata
timing awal konsisten dengan paired timing.

Reproduksi independen dari environment baru belum
diverifikasi.

### Instalasi

```bash
python3.12 -m venv .venv
source .venv/bin/activate

python -m pip install -r environment-freeze-paired.txt
python -m pip check

python -m ipykernel install --user \
  --name ttki-p1 \
  --display-name "Python (TTKI P1)"
```

### Verifikasi dataset

```bash
sha256sum data/PhiUSIIL_Phishing_URL_Dataset.csv
```

Bandingkan checksum dengan bagian integritas dataset.

### Urutan notebook

1. 01_data_audit.ipynb
2. 02_baseline.ipynb
3. 03_experiments.ipynb
4. 04_final_test_and_timing.ipynb

Notebook 01 dapat dijalankan kembali setelah notebook
03 untuk memeriksa indeks split aktual yang tersimpan.

Untuk membaca hasil yang sudah tersedia, gunakan
CSV dan NPZ dalam folder results.

Untuk reproduksi penuh, gunakan salinan repositori
terpisah agar artefak hasil penelitian tidak tertimpa.

Aktifkan RUN_FINAL_TEST hanya pada salinan reproduksi
yang belum memiliki hasil final test.

Jangan menggunakan hasil test untuk memilih model,
fitur, parameter, threshold, atau hasil terbaik.

Pengukuran waktu dapat berbeda antar perangkat
dan kondisi beban sistem.

## Referensi dan penggunaan AI

Matriks literatur tersedia pada:

`references/literature_matrix.csv`

Status verifikasi sumber mengikuti matriks tersebut.

Repositori referensi implementasi:

https://github.com/pradhyumnaag30/PhiUSIIL-Phishing-URL-Detection

Claude dan ChatGPT digunakan sebagai alat bantu
penulisan kode, debugging, audit, dan penyuntingan.

Angka hasil berasal dari eksperimen kelompok.
Kelompok bertanggung jawab atas verifikasi kode,
interpretasi hasil, dan naskah penelitian.

## Lisensi

Dataset asli menggunakan CC BY 4.0.

Lisensi kode repositori belum ditetapkan secara formal.
