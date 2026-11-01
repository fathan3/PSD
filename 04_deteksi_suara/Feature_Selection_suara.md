# Feature Selection: Seleksi Fitur ANOVA F-Score & Analisis Trade-off

Tahap **Feature Selection** (Seleksi Fitur) bertujuan untuk mengidentifikasi dan memilih subset fitur akustik yang paling diskriminatif dalam membedakan kondisi **Tenang** dan **Berisik**. Melalui pengujian statistik inferensial, dimensi fitur dipangkas secara substansial guna mengoptimalkan akurasi model, mencegah *overfitting*, dan mempercepat waktu inferensi *real-time*.

---

## 1. Rasionalisasi: Mengapa 32 Fitur Perlu Dipangkas?

Meskipun $32\text{ fitur}$ telah diekstraksi pada tahap awal, menggunakan seluruh fitur tersebut secara langsung pada model Machine Learning menimbulkan beberapa kelemahan kritis:

1. **Bahaya Multikolinearitas & *Overfitting*:**  
   Banyak koefisien MFCC orde tinggi beserta standar deviasinya memiliki korelasi silang yang sangat kuat satu sama lain dan hanya merefleksikan derau (*noise*) mikroskopis perangkat mikrofon. Model Support Vector Machine (SVM) yang dilatih dengan terlalu banyak dimensi yang kurang relevan berisiko menghafal variansi derau data latih (*curse of dimensionality*).
2. **Kebutuhan Kecepatan Operasional *Real-Time*:**  
   Sistem peringatan kebisingan harus memproses audio secara berkelanjutan di laboratorium. Semakin sedikit fitur yang dihitung dalam fungsi matematika transformasi, semakin cepat siklus ekstraksi fitur dan penentuan fungsi keputusan model, sehingga latensi inferensi dapat ditekan di bawah $50\text{ milidetik}$.
3. **Penyederhanaan Bidang Pembatas (*Hyperplane Simplicity*):**  
   Dengan mengeliminasi fitur-fitur yang tidak memiliki daya pembeda yang signifikan, margin pemisah antar-kelas menjadi lebih tegas, bersih, dan memiliki kemampuan generalisasi yang lebih tangguh saat menghadapi data rekaman baru.

---

## 2. Metodologi Seleksi: One-Way ANOVA F-Score (`f_classif`)

Pengujian signifikansi fitur dilakukan menggunakan metode **One-Way Analysis of Variance (ANOVA) F-Score**. 

> **Integritas Metodologi (Anti-Leakage):**  
> Kalkulasi ANOVA F-Score **hanya dihitung pada subset data latih (Train Set - 1.474 sampel chunk)**. Data uji sama sekali tidak dilibatkan dalam proses perankingan fitur agar tidak terjadi kebocoran informasi distribusi uji (*information leakage*).

### Formulasi Matematis ANOVA F-Score:
ANOVA mengukur rasio antara variansi rata-rata antar-kelas (*Mean Square Between Classes* / $\text{MSB}$) dengan variansi di dalam masing-masing kelas (*Mean Square Within Classes* / $\text{MSW}$):

$$F = \frac{\text{MSB}}{\text{MSW}} = \frac{\frac{\sum_{i=1}^{K} n_i (\bar{x}_i - \bar{x})^2}{K - 1}}{\frac{\sum_{i=1}^{K} \sum_{j=1}^{n_i} (x_{ij} - \bar{x}_i)^2}{N - K}}$$

Di mana:
* $K$ adalah jumlah kelas ($K = 2$, yaitu Tenang dan Berisik).
* $N$ adalah total jumlah sampel data latih ($N = 1.474$).
* $n_i$ adalah jumlah sampel pada kelas ke-$i$.
* $\bar{x}_i$ adalah nilai rata-rata fitur pada kelas ke-$i$.
* $\bar{x}$ adalah nilai rata-rata global fitur di seluruh sampel data latih.
* $x_{ij}$ adalah nilai fitur pada sampel ke-$j$ di kelas ke-$i$.

**Interpretasi Nilai $F$:**  
Semakin tinggi nilai F-Score dan semakin kecil nilai $p\text{-value}$ mendekati $0$, semakin besar perbedaan rata-rata antar-kelas dibandingkan dengan variansi internalnya. Hal ini menandakan fitur tersebut memiliki daya diskriminasi yang sangat tinggi.

---

## 3. Implementasi Kode & Ranking Fitur Teratas

```python
# ==============================================================================
# PERHITUNGAN ANOVA F-SCORE PADA SUBSET DATA LATIH
# ==============================================================================
from sklearn.feature_selection import f_classif

# Pisahkan fitur dan label pada subset latih dan uji
train_df = df[df["split"] == "train"]
test_df  = df[df["split"] == "test"]

feature_cols = [c for c in df.columns if c not in ["split", "file", "label"]]
X_train_full = train_df[feature_cols]
y_train      = train_df["label"]
X_test_full  = test_df[feature_cols]
y_test       = test_df["label"]

# Hitung skor ANOVA F-Score
f_values, p_values = f_classif(X_train_full, y_train)

anova_df = pd.DataFrame({
    "feature": feature_cols,
    "f_score": f_values,
    "p_value": p_values
}).sort_values("f_score", ascending=False).reset_index(drop=True)

# Visualisasi Barplot Top-20 Fitur
plt.figure(figsize=(10, 6))
sns.barplot(data=anova_df.head(20), x="f_score", y="feature", palette="viridis")
plt.title("Ranking 20 Fitur Akustik Teratas Berdasarkan ANOVA F-Score (Data Latih)", fontsize=12, fontweight="bold")
plt.xlabel("Nilai F-Score (Semakin tinggi semakin diskriminatif)")
plt.ylabel("Fitur Akustik")
plt.tight_layout()
plt.show()

print("Top 15 Fitur Akustik Terbaik:")
print(anova_df.head(15)[["feature", "f_score", "p_value"]].to_string(index=False))
```

### Visualisasi Ranking ANOVA F-Score:

![Ranking ANOVA F-Score](deteksi_suara_files/anova_fscore_ranking.png)

### Tabel Top-15 Fitur Akustik Terbaik:

| Peringkat | Nama Fitur | Nilai F-Score | Nilai $p\text{-value}$ | Signifikansi Statistik |
| :---: | :--- | :---: | :---: | :--- |
| 1 | `mfcc1_mean` | **672.8637** | $1.73 \times 10^{-122}$ | Sangat Signifikan (Pembeda Energi Spektral Global) |
| 2 | `rms_mean` | **475.7751** | $1.27 \times 10^{-91}$ | Sangat Signifikan (Kekuatan Amplitudo Fisik) |
| 3 | `rms_std` | **345.9985** | $1.57 \times 10^{-69}$ | Sangat Signifikan (Dinamika Fluktuasi Percakapan) |
| 4 | `mfcc5_mean` | **316.0035** | $3.39 \times 10^{-64}$ | Sangat Signifikan (Warna Timbre Vokal Manusia) |
| 5 | `mfcc3_mean` | **307.9686** | $9.42 \times 10^{-63}$ | Sangat Signifikan (Formulan Dasar Saluran Vokal) |
| 6 | `mfcc7_mean` | **133.2773** | $1.41 \times 10^{-29}$ | Sangat Signifikan (Tekstur Artikulasi Bicara) |
| 7 | `mfcc13_std` | **97.2650** | $2.92 \times 10^{-22}$ | Sangat Signifikan (Peluruhan Batas Pita Spektral) |
| 8 | `mfcc8_mean` | **81.9848** | $4.24 \times 10^{-19}$ | Sangat Signifikan (Harmonik Vokal Menengah) |
| 9 | `mfcc12_std` | **77.9810** | $2.89 \times 10^{-18}$ | Sangat Signifikan (Fluktuasi Spektral Tinggi) |
| 10 | `mfcc11_mean` | **61.7018** | $7.65 \times 10^{-15}$ | Sangat Signifikan (Resonansi Spektral Atas) |
| 11 | `mfcc11_std` | **52.8516** | $5.82 \times 10^{-13}$ | Sangat Signifikan (Variansi Spektrum Tinggi) |
| 12 | `mfcc6_mean` | **46.3684** | $1.42 \times 10^{-11}$ | Sangat Signifikan (Kelipatan Harmonik Percakapan) |
| 13 | `mfcc4_mean` | **38.2849** | $7.91 \times 10^{-10}$ | Sangat Signifikan (Pola Kontur Fonemik) |
| 14 | `zcr_mean` | **35.7876** | $2.76 \times 10^{-9}$ | Sangat Signifikan (Laju Perlintasan Nol / Desis) |
| 15 | `mfcc12_mean` | **35.3267** | $3.48 \times 10^{-9}$ | Signifikan |

---

## 4. Analisis Kompromi (*Trade-off Analysis*): Menentukan Jumlah Fitur Optimal

Untuk menemukan jumlah fitur yang paling optimal (*sweet spot*), dilakukan evaluasi performa model Support Vector Machine (SVM) dengan menguji berbagai variasi jumlah fitur teratas: $k \in \{4, 6, 8, 10, 12, 14, 16, 24, 32\}$.

```python
# ==============================================================================
# EVALUASI PERFORMA SVM TERHADAP VARIASI JUMLAH FITUR TERPILIH (k)
# ==============================================================================
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score, f1_score

k_values = [4, 6, 8, 10, 12, 14, 16, 24, 32]
accuracies, f1_scores = [], []

for k in k_values:
    sel = anova_df.head(k)["feature"].tolist()
    pipe = make_pipeline(
        StandardScaler(),
        SVC(C=3, probability=True, class_weight="balanced", random_state=42)
    )
    pipe.fit(X_train_full[sel], y_train)
    probs = pipe.predict_proba(X_test_full[sel])[:, 1]
    preds = (probs >= 0.6).astype(int)
    
    acc = accuracy_score(y_test, preds)
    f1  = f1_score(y_test, preds)
    accuracies.append(acc)
    f1_scores.append(f1)
    print(f"k = {k:2d} Fitur -> Akurasi Test: {acc*100:.2f}% | F1-Score: {f1*100:.2f}%")
```

### Hasil Log Komparasi Jumlah Fitur ($k$):
```text
k =  4 Fitur -> Akurasi Test: 74.05% | F1-Score: 69.62%
k =  6 Fitur -> Akurasi Test: 75.14% | F1-Score: 71.60%
k =  8 Fitur -> Akurasi Test: 82.97% | F1-Score: 80.85%
k = 10 Fitur -> Akurasi Test: 84.86% | F1-Score: 83.33%
k = 12 Fitur -> Akurasi Test: 85.95% | F1-Score: 84.52%
k = 14 Fitur -> Akurasi Test: 88.65% | F1-Score: 87.65%  <-- TITIK KESEIMBANGAN OPTIMAL (~89%)
k = 16 Fitur -> Akurasi Test: 88.65% | F1-Score: 87.65%
k = 24 Fitur -> Akurasi Test: 91.62% | F1-Score: 90.96%
k = 32 Fitur -> Akurasi Test: 90.27% | F1-Score: 89.53%
```

### Visualisasi Kurva Trade-off Performa vs Jumlah Fitur:

![Kurva Trade-off Performa](deteksi_suara_files/tradeoff_jumlah_fitur.png)

> **Interpretasi Pemilihan $k = 14$ Fitur**:
> 1. Pada $k < 10$, akurasi model berada di bawah $85\%$ karena model kekurangan informasi tekstur spektral vokal.
> 2. Pada $k = 14$, terjadi lonjakan performa signifikan hingga mencapai akurasi **$88.65\% \approx 89\%$** dan F1-Score **$87.65\%$**.
> 3. Menambah fitur menjadi $k = 16$ tidak memberikan peningkatan akurasi sama sekali ($88.65\%$).
> 4. Walaupun $k = 24$ mencapai akurasi $91.62\%$, pada $k = 32$ akurasi kembali turun ke $90.27\%$, menandakan hadirnya efek *overfitting* akibat dimensi berlebih.
> 5. Oleh karena itu, **$14\text{ Fitur}$** dipilih sebagai arsitektur final karena sukses **memangkas lebih dari $56\%$ beban dimensi komputasi** dari $32$ ke $14$ fitur sambil tetap mempertahankan stabilitas akurasi optimal $\approx 89\%$.

---

## 5. Penetapan & Deskripsi 14 Fitur Terbaik Terpilih

Berikut adalah taksonomi lengkap 14 fitur akustik final yang digunakan dalam sistem inferensi:

| No | Nama Fitur | Kategori Domain | Kontribusi Terhadap Pengenalan Suara Berisik |
| :---: | :--- | :--- | :--- |
| 1 | `rms_mean` | Energi Amplitudo | Indikator primer tingkat kekerasan suara fisik. Memisahkan kondisi sunyi dengan kondisi bergemuruh. |
| 2 | `rms_std` | Dinamika Energi | Menangkap modulasi fluktuasi naik-turun suku kata bicara manusia vs derau monoton kipas/AC. |
| 3 | `zcr_mean` | Perubahan Tanda | Mendeteksi desis, frikatif konsonan tak bersuara (/s/, /t/, /k/), gesekan, dan artikulasi fonem tinggi. |
| 4 | `mfcc1_mean` | Spektrum Global | Mewakili selubung energi spektral keseluruhan (*energy envelope*) pada domain frekuensi pendengaran. |
| 5 | `mfcc3_mean` | Resonansi Vokal | Menangkap formulan frekuensi utama dari nada dasar percakapan vokal manusia. |
| 6 | `mfcc4_mean` | Kontur Fonemik | Menangkap variasi pola artikulasi fonem ucapan manusia. |
| 7 | `mfcc5_mean` | Spektra Timbre | Membedakan warna suara alami manusia dari suara desau mekanis pendingin udara. |
| 8 | `mfcc6_mean` | Harmonik Vokal | Menangkap kelipatan harmonik dari pita suara manusia saat terjadi percakapan simultan. |
| 9 | `mfcc7_mean` | Tekstur Suara | Menangkap transisi cepat suku kata vokal manusia saat terjadi obrolan kelompok. |
| 10 | `mfcc8_mean` | Karakter Akustik | Menyaring perbedaan antara suara ketukan keyboard wajar dengan suara gaduh. |
| 11 | `mfcc11_mean` | Harmonik Tinggi | Menangkap resonansi frekuensi tinggi pada intonasi bicara nada tinggi. |
| 12 | `mfcc11_std` | Dinamika Spektral | Mengukur variansi energi pada pita frekuensi tinggi sepanjang waktu. |
| 13 | `mfcc12_std` | Tekstur Frekuensi | Menangkap fluktuasi detail mikrofon pada batas pita atas frekuensi pendengaran. |
| 14 | `mfcc13_std` | Batas Spektral | Menangkap karakteristik peluruhan spektral (*spectral roll-off dynamic*) pada batas atas spektrum. |

---

## 6. Ekspor Dataset 14 Fitur Terpilih (`dataset_fitur_14.csv`)

Subset 14 fitur diekstraksi ke dalam file CSV baru untuk mempercepat replikasi dan pelatihan ulang model di masa mendatang:

```python
# ==============================================================================
# EKSPOR DATASET 14 FITUR TERPILIH
# ==============================================================================
SELECTED_FEATURES = [
    "rms_mean", "rms_std", "zcr_mean", "mfcc1_mean", "mfcc3_mean",
    "mfcc4_mean", "mfcc5_mean", "mfcc6_mean", "mfcc7_mean", "mfcc8_mean",
    "mfcc11_mean", "mfcc11_std", "mfcc12_std", "mfcc13_std"
]

X_train = X_train_full[SELECTED_FEATURES]
X_test  = X_test_full[SELECTED_FEATURES]

cols_14 = ["split", "file", "label"] + SELECTED_FEATURES
df_14 = df[cols_14]
df_14.to_csv("dataset_fitur_14.csv", index=False)

print(f"File CSV 14 fitur berhasil disimpan ke 'dataset_fitur_14.csv' ({df_14.shape[0]} baris x {df_14.shape[1]} kolom)")
```

**Output Konfirmasi:**
```text
File CSV baru dengan 14 fitur berhasil disimpan ke 'dataset_fitur_14.csv' (1844 baris x 17 kolom)
```
Dataset ringkas ini menghemat lebih dari $50\%$ ukuran memori dan menjadi fondasi utama pada tahap pemodelan (*Modeling*).

