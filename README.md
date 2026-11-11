# Analisis Regresi Konsumsi Energi Terbarukan di Amerika Serikat
### *U.S. Renewable Energy Consumption Forecasting based on Sectoral Utilization, Seasonal Cycles, and Long-Term Trends*

![CRISP-DM](https://img.shields.io/badge/Methodology-CRISP--DM-blue.svg)
![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12-brightgreen.svg)
![License](https://img.shields.io/badge/License-U.S.%20Government%20Works-lightgrey.svg)
![Data Source](https://img.shields.io/badge/Data-U.S.%20EIA%20%2F%20Kaggle-orange.svg)
![Model](https://img.shields.io/badge/Model-Multiple%20Linear%20Regression-purple.svg)

---

## 📌 Gambaran Proyek (*Project Overview*)

Transisi energi global menuntut pemahaman mendalam terhadap pola pemanfaatan energi bersih. Di Amerika Serikat, konsumsi energi terbarukan memiliki karakteristik yang sangat heterogen: laju adopsi bervariasi tajam antar-sektor pengguna dan menunjukkan fluktuasi periodik tahunan yang dipengaruhi oleh faktor cuaca dan musim.

Repositori ini memuat seluruh kode, visualisasi, dan dataset pendukung untuk proyek penelitian:  
**"Analisis Regresi dalam Memprediksi Konsumsi Energi Terbarukan di Amerika Serikat Berdasarkan Sektor Pengguna, Faktor Musiman dan Tren Tahunan"**.

Proyek ini bertujuan untuk membangun model peramalan konsumsi energi terbarukan berbasis **Multiple Linear Regression (OLS)** yang akurat, teruji secara ekonometrik, dan bebas dari *data leakage*, sekaligus memberikan rekomendasi strategis bagi utilitas listrik, pembuat kebijakan, dan pelaku industri energi.

---

## 🎯 Sasaran & Metrik Pemodelan (*SMART Objectives*)

1. **Akurasi Prediksi *Out-of-Time*:**
   - Membangun model regresi yang dievaluasi pada data uji kronologis 5 tahun terakhir (2019–2023, tanpa pengacakan).
   - Target keberhasilan: **$Adjusted\ R^2 \ge 0,80$**, **$\text{RMSE} \le 40\text{ trillion BTU}$**, dan **$\text{MAPE} < 10\%$**.
   - Model melampaui tolok ukur dasar (*mean predictor* $R^2 = -0,14$ dan model sektor saja $R^2 = 0,62$).
2. **Kelayakan Asumsi Klasik OLS:**
   - Menguji dan memenuhi 4 asumsi klasik ekonometrika:
     - **Multikolinearitas:** $\text{VIF} < 5$ untuk seluruh variabel independen.
     - **Autokorelasi:** Uji Durbin-Watson / Breusch-Godfrey ($p > 0,05$).
     - **Heteroskedastisitas:** Uji Breusch-Pagan ($p > 0,05$).
     - **Normalitas Residual:** Uji Jarque-Bera ($p > 0,05$).
3. **Dampak Bisnis & Kebijakan Publik:**
   - Mengestimasi periode dan besaran beban puncak musiman (*peak demand*) untuk efisiensi jadwal pemeliharaan jaringan (*grid maintenance*) dan kontrak pengadaan energi jangka panjang (*Power Purchase Agreements*).

---

## 🔬 Metodologi Penelitian (Kerangka CRISP-DM)

Proyek ini disusun secara sistematis mengacu pada standar industri **CRISP-DM (Cross-Industry Standard Process for Data Mining)**:

```
[1. Business Understanding] ──► [2. Data Understanding] ──► [3. Data Cleaning]
                                                                     │
[6. Evaluation & Policy]   ◄── [5. Modeling & Assumptions] ◄── [4. Data Preparation]
```

### 1. Data Understanding & Audit
- **Sumber Data:** U.S. Energy Information Administration (EIA) *Monthly Energy Review* (1973–2024), 3.065 baris awal $\times$ 17 kolom numerik/kategori.
- **Audit Kualitas:** Pemeriksaan nilai 0 sebagai representasi *not reported/missing*, konfirmasi ketiadaan duplikasi baris, serta identifikasi pencilan (*IQR rule*).
- **Analisis Deret Waktu:** Dekomposisi STL (*Seasonal-Trend decomposition using LOESS*), uji stasioneritas Augmented Dickey-Fuller (ADF), dan uji beda musiman Kruskal-Wallis ($p < 0,001$).

### 2. Data Cleaning
- **Koreksi Tipografi:** Standardisasi inkonsistensi ejaan `"Commerical"` $\rightarrow$ `"Commercial"`.
- **Fusi Fitur Hidro:** Penyatuan kolom komplementer `Hydroelectric Power` dan `Conventional Hydroelectric Power` menjadi satu fitur konsisten `hydro`.
- **Eliminasi Sektor Non-Informatif:** Menghapus sektor *Transportation* karena nilai target 100% *missing*.
- **Pembersihan Periode Parsial:** Mengeliminasi Januari 2024 untuk menjaga keutuhan siklus tahunan (menghasilkan 2.448 baris bersih periode 1973–2023).

### 3. Feature Selection & Engineering
- **Pencegahan *Target Leakage*:** Sub-komponen energi mentah (*hydro, solar, wind, biomass, geothermal*) dikeluarkan dari prediktor karena total konsumsi merupakan penjumlahan linier eksak dari komponen-komponen tersebut.
- **Fitur Baru yang Direkayasa:**
  - `t`: Indeks waktu kontinu (0 sampai 611 bulan) untuk memodelkan laju tren jangka panjang.
  - `Sector_t`: Interaksi sektor $\times$ tren untuk menangkap kemiringan tren yang berlainan antar-sektor.
  - `Month` (Dummy / Cyclical $\sin$ & $\cos$): Representasi periodik fluktuasi 12-bulanan kalender.

### 4. Data Preprocessing & Splitting
- **Skema Splitting Kronologis:** Pembagian berbasis waktu: **Data Latih (1973–2018: 2.208 observasi)** dan **Data Uji (2019–2023: 240 observasi)**. Pengacakan acak (*random shuffle*) dilarang guna menghindari *look-ahead bias* / *temporal leakage*.
- **Encoding & Scaling:** One-Hot Encoding pada sektor dan bulan; Min-Max dan Standard Scaling dihitung eksklusif pada data latih (*fit on train, transform on test*).

---

## 📊 Hasil dan Bukti Empiris (*Ablation Study*)

Pengujian bertahap pada himpunan data uji (*out-of-time test set*) membuktikan kontribusi signifikan dari seleksi dan rekayasa fitur:

| Tingkat Model | Fitur yang Digunakan | $R^2$ Test | RMSE (Trillion BTU) | MAPE (%) | Keterangan |
|---|---|:---:|:---:|:---:|---|
| **Baseline 1** | Tanpa fitur (Intersept Rerata) | $-0,22$ | $107,4$ | $127,8\%$ | Tolok ukur terendah tanpa informasi waktu/sektor |
| **Model 2** | `Sector` (One-Hot) | $0,47$ | $70,7$ | $36,3\%$ | Menangkap variasi skala dasar antar-sektor |
| **Model 3** | `Sector` $+$ `Sector_t` (Interaksi Tren) | $0,85$ | $37,3$ | $15,8\%$ | Menangkap divergensi arah pertumbuhan tiap sektor |
| **Model 4** | `Sector` $+$ `Sector_t` $+$ `Month` (Lengkap) | **$0,854$** | **$37,1$** | **$16,6\%$** | **Konfigurasi fitur optimal (menjelaskan >85% variansi)** |

---

## 📁 Struktur Berkas Repositori

```
├── Renewable_Energy_Data_Understanding.ipynb   # Notebook utama (Data Understanding s/d Preprocessing)
├── README.md                                    # Dokumentasi komprehensif proyek
├── requirements.txt                             # Daftar dependensi pustaka Python
├── data/
│   └── dataset.csv                             # Dataset mentah asli EIA/Kaggle (3.065 baris)
├── figures/                                    # Grafik visualisasi resolusi tinggi (PNG)
│   ├── gambar1_pola_nan.png                    # Heatmap missing data per sektor dan jenis energi
│   ├── gambar2_deret_waktu_sektor.png          # Deret waktu konsumsi bulanan vs MA 12-bulan
│   ├── gambar3_indeks_musiman.png              # Indeks musiman rasio per sektor
│   ├── gambar4_bauran_electric_power.png       # Evolusi transisi bauran energi listrik
│   ├── gambar5_sebelum_vs_sesudah_cleaning.png # Metrik perbandingan kondisi data sebelum vs sesudah cleaning
│   ├── gambar6_evaluasi_feature_selection.png  # Bukti kenaikan R² dan penurunan RMSE per model
│   ├── gambar7_train_test_split.png            # Visualisasi time-series split (Train vs Test)
│   ├── gambarB1_distribusi_total_sektor.png    # Histogram distribusi konsumsi tiap sektor
│   ├── gambarB2_boxplot_indeks_bulan.png       # Variasi sebaran rasio musiman per bulan
│   ├── gambarB3_acf_selisih_pertama.png        # Plot Autokorelasi (ACF) setelah differencing
│   └── gambarB4_korelasi_antar_sektor.png      # Matriks korelasi silang antar sektor
└── outputs/                                    # Dataset bersih, partisi train/test, dan tabel analisis (CSV)
    ├── dataset_bersih.csv                      # Data bersih konsumsi (2.448 baris x 17 kolom)
    ├── train_dataset.csv                       # Data latih kronologis 1973–2018 (2.208 baris)
    ├── test_dataset.csv                        # Data uji kronologis 2019–2023 (240 baris)
    ├── X_train_preprocessed.csv                # Matriks fitur latih siap regresi
    ├── X_test_preprocessed.csv                 # Matriks fitur uji siap regresi
    ├── y_train.csv                             # Target latih
    ├── y_test.csv                              # Target uji
    ├── tabel_audit_kualitas_data.csv           # Ringkasan audit missing, duplikat, outlier per kolom
    ├── tabel_sebelum_vs_sesudah_cleaning.csv   # Perbandingan kuantitatif pembersihan data
    ├── tabel_matriks_feature_selection.csv     # Matriks alasan seleksi fitur dipakai/dibuang
    ├── tabel_evaluasi_model.csv                # Rekapitulasi metrik performa model
    ├── tabel_deskriptif_sektor.csv             # Statistik deskriptif dan laju CAGR per sektor
    ├── tabel_tren_musiman_stasioneritas.csv    # Evaluasi STL, Kruskal-Wallis, ADF, dan ACF
    ├── tabel_bauran_electric_power.csv         # Pangsa bauran sumber energi listrik
    ├── tabel_outlier_iqr.csv                   # Deteksi outlier aturan IQR per sektor
    ├── tabel_validasi_rumus_total.csv          # Verifikasi konsistensi penjumlahan target
    ├── tabel_korelasi_waktu_bulan.csv          # Korelasi Spearman terhadap waktu dan musiman
    └── verifikasi_angka_business_understanding.csv # Verifikasi angka SMART
```

---

## 🚀 Panduan Menjalankan Proyek (*Quickstart*)

### 1. Kloning Repositori
```bash
git clone https://github.com/MichaelB232/41_IF-D_KataAnissaBebas.git
cd 41_IF-D_KataAnissaBebas
```

### 2. Siapkan Lingkungan Kerja & Dependensi
Disarankan menggunakan *virtual environment*:
```bash
python -m venv .venv
# Mengaktifkan venv:
# Windows (PowerShell):
.venv\Scripts\Activate.ps1
# Linux / macOS:
source .venv/bin/activate

# Pasang pustaka pendukung:
pip install -r requirements.txt
```

### 3. Eksekusi Analisis
Buka dan jalankan Jupyter Notebook:
```bash
jupyter notebook Renewable_Energy_Data_Understanding.ipynb
```
Pilih **Kernel $\rightarrow$ Restart & Run All**. Seluruh visualisasi pada folder `figures/` dan berkas pada `outputs/` akan ter-generate ulang secara deterministik.

---

## 👥 Tim Peneliti & Kontributor

Program Studi Informatika, Jurusan Informatika, Fakultas Teknik Industri  
**Universitas Pembangunan Nasional "Veteran" Yogyakarta, 2026**

- **Dosen Pengampu:** Dhimas Arief Dharmawan
- **Anggota Kelompok:**
  1. Artsetya Evan Justine Wijaya (NIM: `123240169`)
  2. Michael Bintang F.S.J (NIM: `123240145`)
  3. Anisa Zahida (NIM: `123240049`)
  4. Hardanu Ilham Purnama (NIM: `123240166`)
