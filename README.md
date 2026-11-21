# P1 — Split Strategy, Feature Group, and Feature Selection in Phishing URL Detection: A Controlled Comparison on PhiUSIIL

**Dika dan Akmal** · Sarjana Terapan Teknik Informatika PENS, Semester 5  
Proyek TTKI 2026-1

---

## Tujuan penelitian

Penelitian ini mengevaluasi secara terkendali pengaruh tiga faktor terhadap F1-score, ROC-AUC, dan waktu prediksi model deteksi phishing berbasis fitur URL pada dataset PhiUSIIL:

| Hipotesis | Perbandingan | Pertanyaan |
|---|---|---|
| H1 | Skenario B → C | Apakah split per registered domain menurunkan performa dibanding split acak? |
| H2 | Skenario C → D | Apakah pengurangan ke 5 fitur URL menghemat waktu prediksi dengan performa yang masih memadai? |
| H3 | Skenario A → B | Apakah fitur halaman (F-Page) berkontribusi terhadap performa dibanding URL saja? |

Model yang dibandingkan: **Logistic Regression** dan **Random Forest**.  
Evaluasi dibatasi pada analisis offline; URL phishing tidak dikunjungi.

---

## Dataset

| Komponen | Isian |
|---|---|
| Nama | PhiUSIIL Phishing URL Dataset |
| Sumber | https://www.kaggle.com/datasets/kaggleprollc/phishing-url-websites-dataset-phiusiil |
| Versi / tanggal unduh | 12 November 2026 |
| Ukuran | 235.795 baris, 56 kolom (54 fitur + FILENAME + label) |
| Distribusi kelas | 134.850 legitimate, 100.945 phishing |
| Lisensi | CC BY 4.0 |
| SHA-256 | `a236549cd369cd80bd478ff8e1779cbf44c58d5c3f79f7a51a1adbed7d06d1c6` |

> Dataset **tidak diunggah** ke repositori ini karena ukuran dan pertimbangan lisensi.  
> Unduh dari Kaggle, letakkan di `data/PhiUSIIL_Phishing_URL_Dataset.csv`.

---

## Struktur repositori

```
.
├── data/                          # Dataset (tidak di-commit)
│   └── PhiUSIIL_Phishing_URL_Dataset.csv
├── configs/
│   └── experiment_config.json     # Seed, fitur, parameter model, kebijakan PSL
├── notebooks/
│   ├── 01_data_audit.ipynb        # Audit dataset: duplikat, missing, distribusi domain
│   ├── 02_baseline.ipynb          # Skenario A (F-URL + F-Page, split acak)
│   ├── 03_experiments.ipynb       # Skenario B, C, D
│   └── 04_final_test_and_timing.ipynb  # Test final + pengukuran waktu prediksi + plot
├── results/
│   ├── split_indices_random.npz   # Indeks split acak (seed 42)
│   ├── split_indices_grouped.npz  # Indeks split per registered domain (seed 42)
│   ├── metrics_per_fold.csv       # Metrik per fold semua skenario
│   ├── all_scenarios_cv_summary.csv    # Rata-rata ± SD CV per skenario × model
│   ├── selected_features.csv      # 5 fitur terpilih skenario D (per fold)
│   ├── test_final_results.csv     # Hasil evaluasi test holdout final
│   ├── test_final_selected_features_D.csv  # Fitur D pada dev set penuh
│   ├── timing_results.csv         # Waktu predict_proba (median, IQR)
│   ├── timing_environment.json    # Spesifikasi perangkat pengukuran waktu
│   ├── final_table_cv_vs_test.csv # Tabel gabungan CV + test final
│   └── plots_h1h2h3.png           # Visualisasi 4 panel H1/H2/H3
├── README.md
├── requirements.txt               # Versi paket minimum
└── environment-freeze.txt         # Output pip freeze lingkungan aktual
```

---

## Cara menjalankan

### 1. Siapkan environment

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Unduh dataset

Unduh dari Kaggle lalu letakkan di:
```
data/PhiUSIIL_Phishing_URL_Dataset.csv
```

Verifikasi SHA-256:
```bash
# Linux / macOS
sha256sum data/PhiUSIIL_Phishing_URL_Dataset.csv

# Windows (PowerShell)
Get-FileHash data\PhiUSIIL_Phishing_URL_Dataset.csv -Algorithm SHA256
```
Harus menghasilkan: `a236549cd369cd80bd478ff8e1779cbf44c58d5c3f79f7a51a1adbed7d06d1c6`

### 3. Jalankan notebook secara berurutan

```
01_data_audit.ipynb        → audit awal, tidak mengubah data
02_baseline.ipynb          → skenario A + simpan split_indices_random.npz
03_experiments.ipynb       → skenario B, C, D + simpan split_indices_grouped.npz
04_final_test_and_timing.ipynb → test final + waktu + plot
```

> **Penting:** Pada notebook 04, ubah `RUN_FINAL_TEST = False` menjadi `RUN_FINAL_TEST = True` **sekali saja**, setelah seluruh rancangan dibekukan. Notebook menolak menimpa hasil yang sudah ada kecuali `OVERWRITE_FINAL = True`.

### 4. Seed

Seed global: **42**. Ditetapkan di `configs/experiment_config.json` dan dipakai di semua notebook.

---

## Skenario eksperimen

| Skenario | Split | Fitur | Jumlah fitur | Perbandingan |
|---|---|---|---|---|
| A | Random stratified | F-URL + F-Page | 46 | Acuan |
| B | Random stratified (sama dengan A) | F-URL | 18 | A→B: H3 |
| C | Registered-domain-disjoint | F-URL | 18 | B→C: H1 |
| D | Registered-domain-disjoint (sama dengan C) | 5 fitur F-URL hasil SelectKBest | 5 | C→D: H2 |

**Kebijakan PSL:** `include_psl_private_domains=True` (tldextract) — berlaku untuk skenario C dan D.  
Imputasi, scaling (LR), dan seleksi fitur (D) di-fit hanya pada data latih setiap fold.

---

## Ringkasan hasil

| Hipotesis | ΔF1 LR (test) | ΔF1 RF (test) | Kesimpulan |
|---|---|---|---|
| H1: B→C (protokol split) | −0,0004 | 0,0000 | Tidak terdukung — tidak ada perbedaan terukur |
| H2: C→D (kurangi fitur) — performa | −0,0103 | −0,0030 | Terdukung sebagian |
| H2: C→D — waktu prediksi | LR 2,43× lebih lambat | RF 1,35× lebih lambat | Tidak terdukung |
| H3: A→B (kelompok fitur) | −0,0035 | −0,0027 | Terdukung |

Waktu diukur pada: Python 3.12.3, sklearn 1.9.1, 4 CPU logis, batch 1.000 baris, 30 ulangan.

---

## Penggunaan AI dan kode pihak ketiga

Kode dibantu dengan Claude (Anthropic) sebagai alat bantu penulisan dan debugging.  
Seluruh angka hasil berasal dari eksperimen kelompok sendiri.  
Repositori referensi: https://github.com/pradhyumnaag30/PhiUSIIL-Phishing-URL-Detection (titik awal implementasi, bukan sumber angka).

---

## Lisensi kode

Kode dalam repositori ini dirilis di bawah lisensi **MIT**.  
Dataset mengikuti lisensi CC BY 4.0 dari sumber aslinya.
