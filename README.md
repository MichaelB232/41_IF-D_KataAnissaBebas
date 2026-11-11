# Analisis Regresi dalam Memprediksi Konsumsi Energi Terbarukan di Amerika Serikat Berdasarkan Sektor Pengguna, Faktor Musiman dan Tren Tahunan

Berkas pendukung laporan praktikum Data Science, tahap **Data Understanding, Data Cleaning, Preprocessing, dan Feature Selection (Progress 2 - Kerangka Kerja CRISP-DM)**.

Program Studi Informatika, Fakultas Teknik Industri, UPN "Veteran" Yogyakarta, 2026.  
Dosen Pengampu: Dhimas Arief Dharmawan.

| Nama | NIM |
|---|---|
| Artsetya Evan Justine Wijaya | 123240169 |
| Michael Bintang F.S.J | 123240145 |
| Anisa Zahida | 123240049 |
| Hardanu Ilham Purnama | 123240166 |

---

## Matriks Pemenuhan Capaian Pembelajaran (Progress 2)

Seluruh 14 butir capaian pada lembar capaian Progress 2 telah dipenuhi dan terdokumentasi secara lengkap:

| No | Aspek Penilaian | Item Penilaian / Capaian | Status | Rujukan Berkas / Bagian |
|:---:|---|---|:---:|---|
| **1** | Data Understanding | Sumber data & lisensi/izin | ✅ Lengkap | Notebook Bab 1, README bagian Sumber Data |
| **2** | Data Understanding | Data dictionary (variabel, tipe, unit) | ✅ Lengkap | Notebook Bab 2 (Tabel Kamus Data 17+ variabel) |
| **3** | Data Understanding | Kualitas data (missing, outlier, duplikasi) | ✅ Lengkap | Notebook Bab 3, `outputs/tabel_audit_kualitas_data.csv` |
| **4** | Data Understanding | EDA berinsight (distribusi/korelasi) | ✅ Lengkap | Notebook Bab 5, `figures/gambarB1`, `figures/gambarB4` |
| **5** | Data Understanding | Representativitas & bias sampel | ✅ Lengkap | Notebook Bab 5.5 (Evaluasi 4 dimensi bias sampel) |
| **6** | EDA | Statistik deskriptif ditampilkan dan disimpulkan | ✅ Lengkap | Notebook Bab 5.1, `outputs/tabel_deskriptif_sektor.csv` |
| **7** | EDA | Visualisasi fitur dan target, beri penjelasan atas temuan | ✅ Lengkap | Notebook Bab 5.2 & 5.3 (Narasi di bawah grafik) |
| **8** | EDA | Analisis korelasi fitur vs target (3 insight cleaning/modeling) | ✅ Lengkap | Notebook Bab 5.4.1 (3 insight utama terstruktur) |
| **9** | Audit Kualitas Data | Menghitung missing value, outlier, duplikat tiap kolom | ✅ Lengkap | `outputs/tabel_audit_kualitas_data.csv` |
| **10** | Cleaning | Menyelesaikan masalah kualitas data dengan strategi tepat | ✅ Lengkap | Notebook Bab 4.1 (Drop Transportation, 2024, fusi hydro, perbaikan typo) |
| **11** | Cleaning | Menampilkan kondisi sebelum vs sesudah dibersihkan | ✅ Lengkap | Notebook Bab 4.2, `outputs/tabel_sebelum_vs_sesudah_cleaning.csv`, `figures/gambar5` |
| **12** | Preprocessing | Encoding, scaling, dan penanganan data imbalance | ✅ Lengkap | Notebook Bab 7.1 - 7.3, `outputs/X_train_preprocessed.csv` |
| **13** | Preprocessing | Splitting dataset train/test (kronologis time-series) | ✅ Lengkap | Notebook Bab 7.4, `outputs/train_dataset.csv`, `outputs/test_dataset.csv`, `figures/gambar7` |
| **14** | Feature Selection | Memilih fitur dengan alasan & bukti empiris | ✅ Lengkap | Notebook Bab 6, `outputs/tabel_matriks_feature_selection.csv`, `outputs/tabel_evaluasi_model.csv`, `figures/gambar6` |

---

## Sumber Data

- **Dataset:** *U.S. Renewable Energy Consumption* (Alistair King), Kaggle  
  https://www.kaggle.com/datasets/alistairking/renewable-energy-consumption-in-the-u-s
- **Sumber Asli:** U.S. Energy Information Administration (EIA), *Monthly Energy Review*  
  https://www.eia.gov/totalenergy/data/monthly/
- **Lisensi:** U.S. Government Works (Public Domain)
- **Satuan:** trillion British Thermal Units (trillion BTU)
- **Isi Mentah:** 3.065 baris (613 bulan x 5 sektor), Januari 1973 sampai Januari 2024, 17 kolom.
- **Data Bersih (Siap Modeling):** 2.448 baris (612 bulan utuh x 4 sektor aktif: Electric Power, Industrial, Residential, Commercial), Januari 1973 sampai Desember 2023.

---

## Struktur Folder Proyek

```
renewable_energy_us_project/
├── Renewable_Energy_Data_Understanding.ipynb   notebook utama (Data Understanding s/d Preprocessing & Feature Selection)
├── README.md                                    dokumentasi proyek dan panduan capaian Progress 2
├── requirements.txt                             daftar dependensi lingkungan kerja
├── data/
│   └── dataset.csv                             dataset mentah asli dari EIA/Kaggle
├── figures/                                    visualisasi grafik hasil analisis (PNG)
│   ├── gambar1_pola_nan.png                    heatmap ketersediaan dan missing data per sektor
│   ├── gambar2_deret_waktu_sektor.png          tren deret waktu konsumsi bulanan dan moving average 12 bulan
│   ├── gambar3_indeks_musiman.png              indeks musiman konsumsi energi per sektor
│   ├── gambar4_bauran_electric_power.png       transformasi bauran energi sektor Electric Power
│   ├── gambar5_sebelum_vs_sesudah_cleaning.png perbandingan kuantitatif kondisi sebelum vs sesudah pembersihan
│   ├── gambar6_evaluasi_feature_selection.png  bukti empiris lonjakan R² dan penurunan RMSE hasil seleksi fitur
│   ├── gambar7_train_test_split.png            visualisasi pembagian data latih (1973-2018) dan data uji (2019-2023)
│   ├── gambarB1_distribusi_total_sektor.png    histogram sebaran konsumsi bulanan per sektor
│   ├── gambarB2_boxplot_indeks_bulan.png       boxplot sebaran rasio musiman per bulan
│   ├── gambarB3_acf_selisih_pertama.png        autokorelasi (ACF) setelah pembedaan pertama (differencing)
│   └── gambarB4_korelasi_antar_sektor.png      matriks korelasi antar-sektor (level vs selisih 12 bulan)
└── outputs/                                    tabel ringkasan, dataset bersih, dan partisi train/test (CSV)
    ├── dataset_bersih.csv                      dataset bersih siap modeling (2.448 baris x 17 kolom)
    ├── train_dataset.csv                       himpunan data latih kronologis (1973-2018: 2.208 baris)
    ├── test_dataset.csv                        himpunan data uji kronologis (2019-2023: 240 baris)
    ├── X_train_preprocessed.csv                matriks fitur data latih (One-Hot, Scaled, Interactions)
    ├── X_test_preprocessed.csv                 matriks fitur data uji (One-Hot, Scaled, Interactions)
    ├── y_train.csv                             vektor target data latih
    ├── y_test.csv                              vektor target data uji
    ├── tabel_audit_kualitas_data.csv           ringkasan audit missing, duplikat, dan outlier tiap kolom
    ├── tabel_sebelum_vs_sesudah_cleaning.csv   tabel komparasi sebelum vs sesudah cleaning
    ├── tabel_matriks_feature_selection.csv     matriks alasan & bukti fitur dipakai/dibuang/direkayasa
    ├── tabel_evaluasi_model.csv                performa model bertahap (R², RMSE, MAPE)
    ├── tabel_deskriptif_sektor.csv             statistik deskriptif per sektor & CAGR
    ├── tabel_tren_musiman_stasioneritas.csv    kekuatan tren, musiman, uji Kruskal-Wallis, ADF test
    ├── tabel_bauran_electric_power.csv         persentase bauran sumber energi listrik
    ├── tabel_outlier_iqr.csv                   deteksi outlier aturan IQR per sektor
    ├── tabel_validasi_rumus_total.csv          verifikasi konsistensi penjumlahan target
    ├── tabel_korelasi_waktu_bulan.csv          korelasi Spearman target terhadap waktu dan siklus bulan
    ├── tabel_korelasi_antar_sektor_level.csv   matriks korelasi silang antar sektor
    ├── tabel_korelasi_antar_sektor_selisih12.csv matriks korelasi diferensiasi tahunan
    └── verifikasi_angka_business_understanding.csv evaluasi tujuan SMART
```

---

## Ringkasan Metodologi Data Preparation (Progress 2)

1. **Audit & Data Cleaning:**
   - Standarisasi typo `"Commerical"` $\rightarrow$ `"Commercial"`.
   - Konversi nilai representasi `0` menjadi `NaN`.
   - Fusi kolom hidro `Hydroelectric Power` dan `Conventional Hydroelectric Power` menjadi `hydro`.
   - Eliminasi sektor *Transportation* (100% missing pada target `Total Renewable Energy`).
   - Eliminasi tahun parsial 2024 (hanya 1 bulan data) guna menjaga keutuhan siklus tahunan 1973–2023.

2. **Feature Selection & Engineering:**
   - **Fitur Dipakai:** `Sector` (One-Hot), `t` (indeks waktu kontinu), `Sector_t` (interaksi tren per sektor), dan dummy `Month` (komponen musiman).
   - **Fitur Dibuang:** Sub-komponen energi mentah (`hydro`, `solar`, `wind`, dll.) dibuang dari input untuk mencegah **target leakage** dan multikolinearitas sempurna karena target merupakan penjumlahan linier langsung dari komponen-komponen tersebut.
   - **Bukti Empiris:** Penambahan fitur seleksi terbukti meningkatkan $R^2$ dari $-0,22$ (baseline mean) $\rightarrow$ $0,47$ (sektor) $\rightarrow$ $0,85$ (sektor x tren) $\rightarrow$ **$0,854$ (lengkap + musiman)** dengan RMSE turun dari $107,4$ ke $37,1$.

3. **Preprocessing & Splitting:**
   - **Encoding:** One-Hot Encoding untuk variabel nominal `Sector` dan `Month` (dengan kategori referensi untuk menghindari *dummy variable trap*).
   - **Scaling:** Standard Scaling dan Min-Max Scaling pada fitur numerik kontinu $t$ yang di-*fit* hanya pada data latih dan di-*transform* pada data uji.
   - **Data Splitting:** Pembagian kronologis (*time-series split*) periode 1973–2018 sebagai **Data Latih (2.208 baris)** dan 2019–2023 sebagai **Data Uji (240 baris)** untuk menghindari *data leakage* / *look-ahead bias*.
