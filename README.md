# Analisis Regresi Konsumsi Energi Terbarukan di Amerika Serikat
### *U.S. Renewable Energy Consumption Forecasting based on Sectoral Utilization, Seasonal Cycles, and Long-Term Trends*

![CRISP-DM](https://img.shields.io/badge/Methodology-CRISP--DM-blue.svg)
![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12-brightgreen.svg)
![License](https://img.shields.io/badge/License-U.S.%20Government%20Works-lightgrey.svg)
![Data Source](https://img.shields.io/badge/Data-U.S.%20EIA%20%2F%20Kaggle-orange.svg)
![Model](https://img.shields.io/badge/Models-OLS%20%7C%20Holt--Winters%20ETS%20%7C%20Lag%20OLS-purple.svg)

---

## 📌 Gambaran Proyek (*Project Overview*)

Transisi energi global menuntut pemahaman mendalam terhadap pola pemanfaatan energi bersih. Di Amerika Serikat, konsumsi energi terbarukan memiliki karakteristik yang sangat heterogen: laju adopsi bervariasi tajam antar-sektor pengguna dan menunjukkan fluktuasi periodik tahunan yang dipengaruhi oleh faktor cuaca dan musim.

Repositori ini memuat seluruh kode, visualisasi, dan dataset pendukung untuk proyek penelitian:  
**"Analisis Regresi dalam Memprediksi Konsumsi Energi Terbarukan di Amerika Serikat Berdasarkan Sektor Pengguna, Faktor Musiman dan Tren Tahunan"**.

Proyek ini mengimplementasikan alur **CRISP-DM** end-to-end — mulai dari audit kesehatan data, pembersihan bebas kebocoran (*leakage-free preprocessing*), *ablation study* berbasis **Multiple Linear Regression (OLS)** beserta uji 4 asumsi klasik & koreksi **Newey-West HAC**, hingga pembandingan model *challenger* non-linear serta **Auto-AIC Holt-Winters ETS** dan **Lag-1/Lag-12 OLS** untuk rekomendasi kebijakan energi dan operasional *grid*.

---

## 🎯 Sasaran & Metrik Pemodelan (*SMART Objectives*)

1. **Akurasi Prediksi *Out-of-Time*:**
   - Membangun model peramalan yang dievaluasi pada data uji kronologis 5 tahun terakhir (2019–2023, tanpa pengacakan).
   - Target keberhasilan: **$Adjusted\ R^2 \ge 0,80$**, **$\text{RMSE} \le 40\text{ trillion BTU}$**, dan **$\text{MAPE} < 10\%$**.
   - Model wajib melampaui tolok ukur dasar (*Baseline Trend* dan *Seasonal Naive*).
2. **Kelayakan Asumsi Klasik OLS & Mitigasi:**
   - Menguji 4 asumsi klasik ekonometrika pada OLS:
     - **Multikolinearitas:** $\text{VIF} < 5$ (membedakan multikolinearitas struktural pada term interaksi).
     - **Autokorelasi:** Uji Durbin-Watson (global & *within-sector*) serta Breusch-Godfrey ($p \ge 0,05$).
     - **Heteroskedastisitas:** Uji Breusch-Pagan ($p \ge 0,05$).
     - **Normalitas Residual:** Uji Jarque-Bera ($p \ge 0,05$) & Normal Q-Q Plot.
   - Menerapkan *Newey-West HAC robust standard errors* untuk inferensi valid saat terjadi heteroskedastisitas & autokorelasi, serta mengevaluasi model deret waktu (*Holt-Winters ETS* & *Lag OLS*) berdasarkan hasil diagnostik.
3. **Dampak Bisnis & Kebijakan Publik:**
   - Mengestimasi periode dan besaran beban puncak musiman (*peak demand*) per sektor untuk efisiensi jadwal pemeliharaan jaringan (*grid maintenance*) dan kontrak pengadaan energi jangka panjang (*Power Purchase Agreements*).

---

## 🔬 Metodologi Penelitian (Kerangka CRISP-DM)

Proyek ini disusun secara sistematis mengacu pada standar industri **CRISP-DM (Cross-Industry Standard Process for Data Mining)**:

```
[1. Business Understanding] ──► [2. Data Understanding] ──► [3. Data Cleaning]
                                                                     │
[6. Evaluation & Policy]   ◄── [5. Modeling & Challengers] ◄── [4. Feature Engineering]
```

### 1. Data Understanding & Audit
- **Sumber Data:** U.S. Energy Information Administration (EIA) *Monthly Energy Review* (Jan 1973–Jan 2024), 3.065 baris awal $\times$ 17 kolom numerik/kategori.
- **Audit Kualitas:** Pemeriksaan *zero-inflation* (`0` sebagai *not reported/not meaningful*), konfirmasi ketiadaan duplikasi pada kunci komposit `(Year, Month, Sector)`, serta investigasi forensik nilai anomali negatif pada kolom hidro (Agustus 2007).
- **Visual EDA & Uji Statistik:** Analisis distribusi skala per sektor, pola tren 51 tahun (*12-month moving average*), rasio musiman bulanan, serta arsip analisis STL/ADF/Kruskal-Wallis di `notebooks/archive/`.

### 2. Data Cleaning
- **Koreksi Tipografi:** Standardisasi inkonsistensi ejaan `"Commerical"` $\rightarrow$ `"Commercial"`.
- **Eliminasi Sektor Non-Informatif:** Menghapus sektor *Transportation* (613 baris) karena nilai target 100% bernilai nol.
- **Pembersihan Periode Parsial:** Mengeliminasi Januari 2024 (4 baris) untuk menjaga keutuhan 51 tahun penuh (menghasilkan **2.448 baris bersih** periode **1973–2023**, 4 sektor $\times$ 612 bulan).
- **Fusi Fitur Hidro yang Aman:** Melakukan *clipping* nilai negatif (`< 0` $\rightarrow$ `0`) terlebih dahulu sebelum menyatukan kolom komplementer `Hydroelectric Power` dan `Conventional Hydroelectric Power` menggunakan `.max(axis=1)` menjadi `hydro`.

### 3. Feature Engineering & Preprocessing
- **Pencegahan *Target Leakage*:** Sub-komponen energi mentah (*hydro, solar, wind, biomass, geothermal, biofuels*) dikeluarkan dari prediktor karena merupakan komponen penyusun *ex-post* dari `Total Renewable Energy`.
- **Fitur yang Direkayasa:**
  - `t`: Indeks waktu kontinu terindeks-nol (`0` sampai `611`) mulai Januari 1973.
  - `Sector` & `Month` One-Hot Encoding (`drop_first=True`, `dtype=int`) untuk mencegah *Dummy Variable Trap* (baseline: `Commercial` dan `Januari`).
  - `Sector × t`: Term interaksi agar setiap sektor memiliki kemiringan laju pertumbuhan masing-masing.
- **Skema Splitting Kronologis & Scaling:** **Data Latih (1973–2018: 2.208 observasi)** dan **Data Uji (2019–2023: 240 observasi)** tanpa pengacakan acak. `StandardScaler` di-*fit* eksklusif pada fitur kontinu data latih (`t` dan `Sector × t`), sedangkan dummy biner `0/1` dipertahankan utuh.

---

## 📊 Hasil Pemodelan & Tolok Ukur Empiris (*Out-of-Time* 2019–2023)

### 1. *Ablation Study* Baseline OLS (`Model 1` → `Model 4`)
| Tingkat Model | Fitur yang Digunakan | Test Adj. $R^2$ | Test RMSE (Trillion BTU) | Keterangan |
|---|---|:---:|:---:|---|
| **Model 1** | `t` (Tren Global) | $-0,033$ | $98,5$ | Satu garis tren gagal membedakan skala sektor |
| **Model 2** | `t` + `Sector` | $0,660$ | $56,2$ | Menangkap perbedaan skala dasar antar-sektor |
| **Model 3** | `t` + `Sector` + `Sector × t` | $0,848$ | $37,3$ | Menangkap divergensi kemiringan pertumbuhan per sektor |
| **Model 4** | + `Month` (Full Baseline OLS) | $0,843$ | $37,1$ | Lulus $Adj.\ R^2 \ge 0,80$ & $RMSE \le 40$, namun masih kalah dari *Seasonal Naive* |

### 2. *Master Leaderboard*: Baseline vs *Challenger Models*
Diagnostik visual dan uji asumsi klasik OLS (Breusch-Pagan, Durbin-Watson, Jarque-Bera) mengungkap bahwa *Baseline OLS* terdistorsi oleh musiman gabungan lintas sektor dan tren melengkung non-linear. Pengujian model penantang menghasilkan peningkatan signifikan:

| Model | Horizon | Test $R^2$ | Test RMSE (BTU) | Test MAPE (%) | Status Target (<10% MAPE) |
|---|---|:---:|:---:|:---:|:---:|
| **Seasonal Naive (2018 Repeat)** | 5-Yr Blind | $0,9595$ | $19,56$ | $10,58\%$ | ❌ |
| **Model 4: Baseline OLS** | 5-Yr Blind | $0,8544$ | $37,08$ | $16,60\%$ | ❌ |
| **Model 5: Improved OLS (`Sec×t²` + `Sec×Mo`)** | 5-Yr Blind | $0,9419$ | $23,42$ | $15,54\%$ | ❌ |
| **Model 6: Holt-Winters (All Damped)** | 5-Yr Blind | $0,9491$ | $21,93$ | $10,59\%$ | ❌ |
| **Model 6b: Auto-AIC Holt-Winters [5-Yr Winner]** | **5-Yr Blind** | **$0,9621$** | **$18,92$** | **$10,10\%$** | ⚠️ *(Selisih 0,10%)* |
| **Model 7: Lag-1 / Lag-12 OLS [1-Mo Winner]** | **1-Mo Ahead** | **$0,9925$** | **$8,41$** | **$5,97\%$** | ✅ **PASS** |

### 3. Perencanaan Musiman & Kebijakan per Sektor (Aktual 2019–2023)
| Sektor | Pangsa (%) | Rata-rata Bulanan (BTU) | Pertumbuhan 2019→2023 | Bulan Puncak (*Peak*) | Bulan Terendah (*Off-Peak*) | Ayunan Musiman (BTU) |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| **Electric Power** | $48,5\%$ | $251,91$ | $+17,5\%$ | Mei (`281,68`) | September (`223,72`) | $57,97$ |
| **Industrial** | $37,4\%$ | $194,22$ | $-7,6\%$ | Januari (`204,33`) | Februari (`183,39`) | $20,94$ |
| **Residential** | $10,3\%$ | $53,27$ | $+0,4\%$ | Juli (`57,97`) | Februari (`46,21`) | $11,76$ |
| **Commercial** | $3,9\%$ | $20,00$ | $+30,0\%$ | Juli (`21,69`) | Februari (`17,83`) | $3,86$ |

---

## 📁 Struktur Berkas Repositori

Repositori ini menggunakan struktur modular standar industri (*Cookiecutter Data Science style*):

```
├── data/
│   ├── raw/
│   │   └── dataset.csv                         # Dataset mentah asli EIA/Kaggle (3.065 baris)
│   └── processed/                              # Data bersih (renewable_clean.csv) & matriks train/test
├── notebooks/
│   ├── 01_data_understanding.ipynb             # Audit kualitas data, forensik anomali, & Visual EDA
│   ├── 02_data_cleaning.ipynb                  # Pembersihan data, clipping & fusi hidro, assertion gates
│   ├── 03_feature_engineering.ipynb            # Anti-leakage, time index t, split kronologis, encoding & scaling
│   ├── 04_modeling_and_evaluation.ipynb        # Ablation OLS, uji asumsi klasik + HAC, challengers & evaluasi
│   └── archive/
│       └── Renewable_Energy_Data_Understanding.ipynb # Arsip notebook eksplorasi awal (UTS)
├── outputs/
│   ├── model_benchmark_2019_2023.csv           # Tabel perbandingan metrik seluruh model
│   └── sector_seasonal_planning.csv            # Ringkasan beban puncak & perencanaan musiman per sektor
├── reports/
│   └── figures/                                # Grafik evaluasi & visualisasi resolusi tinggi (PNG)
├── .gitignore
├── LICENSE
├── README.md                                   # Dokumentasi komprehensif proyek
└── requirements.txt                            # Daftar dependensi pustaka Python
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

### 3. Eksekusi Analisis Secara Bertahap
Jalankan notebook di dalam folder `notebooks/` secara berurutan dari awal hingga akhir:
1. `notebooks/01_data_understanding.ipynb` — Audit kesehatan data, forensik anomali, dan Visual EDA.
2. `notebooks/02_data_cleaning.ipynb` — Eksekusi rencana pembersihan data dan ekspor `data/processed/renewable_clean.csv`.
3. `notebooks/03_feature_engineering.ipynb` — Rekayasa fitur bebas kebocoran, split kronologis, encoding, scaling, dan ekspor matriks latih/uji.
4. `notebooks/04_modeling_and_evaluation.ipynb` — *Ablation Study*, uji 4 asumsi klasik + *Newey-West HAC*, *Challenger Modeling* (`Holt-Winters ETS` & `Lag OLS`), serta ekspor grafik dan laporan `outputs/`.

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
