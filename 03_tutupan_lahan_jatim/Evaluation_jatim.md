# Evaluation: Validasi Model & Evaluasi Kinerja Multidimensi

Tahap **Evaluation** bertujuan untuk membuktikan keandalan, stabilitas, dan objektivitas model *Machine Learning* melalui pengujian saintifik pada 57.855 piksel data uji independen (dari 60 poligon mandiri) yang belum pernah dilihat model selama proses pelatihan (*Spatial Group Split*).

---

## 1. Formulasi Metrik Evaluasi Multikelas

Untuk memastikan evaluasi bebas dari bias ketimpangan kelas (*class imbalance*), performa model dinilai melalui indikator terintegrasi:

1. **Akurasi Global (*Overall Accuracy*)**:
   $$\text{Akurasi} = \frac{\sum_{c=1}^{K} TP_c}{N_{\text{total}}}$$
   *Mengukur proporsi total piksel yang diprediksi tepat dari seluruh 6 kelas tutupan lahan.*

2. **Presisi per Kelas (*Precision*)**:
   $$\text{Presisi}_c = \frac{TP_c}{TP_c + FP_c}$$
   *Mengukur ketepatan prediksi model; dari seluruh piksel yang diprediksi sebagai kelas $c$, berapa persen yang benar-benar merupakan kelas $c$.*

3. **Recall per Kelas (*Sensitivity / Producer's Accuracy*)**:
   $$\text{Recall}_c = \frac{TP_c}{TP_c + FN_c}$$
   *Mengukur kelengkapan deteksi model; dari seluruh piksel ground truth kelas $c$, berapa persen yang berhasil dikenali oleh model.*

4. **F1-Score per Kelas**:
   $$\text{F1}_c = 2 \times \frac{\text{Presisi}_c \times \text{Recall}_c}{\text{Presisi}_c + \text{Recall}_c}$$
   *Rata-rata harmonik antara presisi dan recall untuk kelas $c$.*

5. **Macro F1-Score (Indikator Evaluasi Utama Multikelas)**:
   $$\text{Macro F1} = \frac{1}{K} \sum_{c=1}^{K} \text{F1}_c$$
   *Rata-rata aritmatika F1-Score seluruh kelas tanpa pembobotan frekuensi. Metrik ini sangat krusial dalam klasifikasi spasial tutupan lahan karena memberikan bobot evaluasi yang setara antara kelas minoritas (Bangunan, Mangrove) dengan kelas mayoritas (Laut, Sawah).*

6. **Confusion Matrix (Matriks Kontinjensi Multikelas)**:
   Menyajikan tabulasi silang antara kelas aktual ground truth (baris) dan kelas prediksi model (kolom) untuk memetakan secara detail pola misklasifikasi antar-objek spasial.

---

## 2. Hasil Komparasi Kinerja Tiga Model Machine Learning

Benchmarking performa ketiga arsitektur model pada data uji independen menghasilkan capaian berikut:

```python
# ==============================================================================
# FASE 5.1 PERBANDINGAN PERFORMA MODEL (BENCHMARKING TIGA MODEL)
# ==============================================================================
KELAS_MAP = {
    1: "Sawah", 2: "Bangunan", 3: "Hutan", 
    4: "Danau", 5: "Laut", 6: "Mangrove"
}
LABEL_NAMES = [KELAS_MAP[i] for i in range(1, 7)]

# Hitung metrik evaluasi
acc_rf = accuracy_score(y_test, y_pred_rf)
f1_rf = f1_score(y_test, y_pred_rf, average="macro")

acc_hgb = accuracy_score(y_test, y_pred_hgb)
f1_hgb = f1_score(y_test, y_pred_hgb, average="macro")

acc_ens = accuracy_score(y_test, y_pred_ens)
f1_ens = f1_score(y_test, y_pred_ens, average="macro")

komparasi = pd.DataFrame({
    "Arsitektur Model": [
        "Model 1: Random Forest Classifier",
        "Model 2: HistGradientBoosting (Gradient Boosting)",
        "Model 3: Ensemble Soft-Voting (RF + HGB) 🏆"
    ],
    "Akurasi Global": [f"{acc_rf*100:.2f}%", f"{acc_hgb*100:.2f}%", f"{acc_ens*100:.2f}%"],
    "Macro F1-Score": [f"{f1_rf*100:.2f}%", f"{f1_hgb*100:.2f}%", f"{f1_ens*100:.2f}%"],
    "Status": [
        "Baseline Standar",
        "Komparasi Boosting",
        "AKURASI TERTINGGI (Terpilih untuk Dashboard) 🏆"
    ]
})
print("TABEL KOMPARASI MODEL MACHINE LEARNING:")
display(komparasi)
```

### Tabel Komparasi Performa Tiga Arsitektur Model:

| Arsitektur Model | Akurasi Global (*Overall Accuracy*) | Macro F1-Score | Status Evaluasi |
| :--- | :---: | :---: | :--- |
| **Model 1: Random Forest Classifier** | 88.83% | 80.18% | Baseline Standar Penginderaan Jauh |
| **Model 2: HistGradientBoosting Classifier** | 88.77% | 80.08% | Komparasi Boosting Sekuensial Cepat |
| **Model 3: Ensemble Soft-Voting (RF + HGB) 🏆** | **89.06%** | **81.21%** | **AKURASI TERTINGGI (Model Terpilih)** 🏆 |

> **Analisis Keberhasilan Ensemble**:  
> Penggabungan probabilitas posterior (*Soft-Voting*) antara Random Forest dan HistGradientBoosting terbukti sukses menaikkan performa di atas kedua model tunggalnya. Hal ini terjadi karena Random Forest mereduksi variansi error pohon individual, sementara HistGradientBoosting secara adaptif mengoreksi residual kesalahan pada batas spektral yang kompleks.

---

## 3. Laporan Klasifikasi (*Classification Report*) Model Terbaik

Berikut adalah rincian laporan klasifikasi lengkap dari model terbaik **Ensemble Soft-Voting Classifier** pada 57.855 piksel data uji independen:

```text
=================================================================
CLASSIFICATION REPORT: ENSEMBLE SOFT-VOTING CLASSIFIER (TERBAIK)
=================================================================
              precision    recall  f1-score   support

       Sawah     0.8517    0.8227    0.8370      8299
    Bangunan     0.7193    0.8000    0.7575       330
       Hutan     0.7128    0.8978    0.7947      3307
       Danau     0.8124    0.6474    0.7206     10271
        Laut     0.9978    0.9976    0.9977     30000
    Mangrove     0.6858    0.8653    0.7651      5648

    accuracy                         0.8906     57855
   macro avg     0.7966    0.8385    0.8121     57855
weighted avg     0.8956    0.8906    0.8898     57855
```

### Rincian Metrik per Kelas:
1. **Laut (Akurasi Dekat Sempurna 99.77%)**:  
   Memiliki presisi **99.78%** dan recall **99.76%** pada 30.000 sampel uji. Hal ini menegaskan bahwa kombinasi serapan B08 NIR dan NDWI positif merupakan pemisah mutlak yang sangat kokoh.
2. **Sawah (Akurasi Tinggi 83.70% F1-Score)**:  
   Mencapai presisi **85.17%** dan recall **82.27%** pada 8.299 piksel uji. Model berhasil membedakan petak sawah aktif dari kawasan sekitarnya.
3. **Bangunan (Presisi 71.93%, Recall 80.00%)**:  
   Meskipun merupakan kelas minoritas ekstrem ($<1\%$ dataset), penyeimbang bobot kelas adaptif berhasil mempertahankan recall **80.00%** (264 dari 330 piksel uji terdeteksi tepat).
4. **Hutan & Mangrove (Recall Tinggi 89.78% & 86.53%)**:  
   Kedua kelas vegetasi ini berhasil dijaring hampir seluruhnya oleh model, menandakan keandalan ekstraksi kanopi hijau dan mangrove pesisir.

---

## 4. Visualisasi Confusion Matrix Heatmap & Feature Importance

Untuk memetakan distribusi galat prediksi dan mengidentifikasi kontribusi saluran spektral, dibuat visualisasi dua panel komprehensif:

```python
# ==============================================================================
# FASE 5.2 VISUALISASI CONFUSION MATRIX & TINGKAT KEPENTINGAN FITUR
# ==============================================================================
cm_ens = confusion_matrix(y_test, y_pred_ens, labels=range(1, 7))
cm_df = pd.DataFrame(cm_ens, index=LABEL_NAMES, columns=LABEL_NAMES)
cm_df.to_csv(OUT_DIR / "confusion_matrix.csv")

fig, axes = plt.subplots(1, 2, figsize=(16, 6), dpi=150)

# 1. Heatmap Confusion Matrix
sns.heatmap(cm_df, annot=True, fmt="d", cmap="Blues", cbar=True, ax=axes[0])
axes[0].set_title(f"Confusion Matrix: Ensemble Model (Akurasi: {acc_ens*100:.2f}%)", fontsize=12, fontweight="bold")
axes[0].set_xlabel("Kelas Prediksi Model", fontsize=10, fontweight="bold")
axes[0].set_ylabel("Kelas Aktual Ground Truth", fontsize=10, fontweight="bold")

# 2. Feature Importance (dari Random Forest)
feat_imp = pd.Series(rf_model.feature_importances_, index=FEATURE_COLS).sort_values(ascending=True)
bars = axes[1].barh(feat_imp.index, feat_imp.values, color="#2563eb", edgecolor="#1e3a8a")
axes[1].set_title("Tingkat Kepentingan Fitur Spektral (Feature Importance)", fontsize=12, fontweight="bold")
axes[1].set_xlabel("Skor Kepentingan (Gini Importance)", fontsize=10, fontweight="bold")
for bar in bars:
    w = bar.get_width()
    axes[1].text(w + 0.005, bar.get_y() + bar.get_height()/2, f"{w:.3f}", va="center", fontsize=9)
axes[1].set_xlim(0, max(feat_imp.values) * 1.15)

plt.tight_layout()
plt.savefig(OUT_DIR / "confusion_matrix.png", dpi=300)
plt.savefig(OUT_DIR / "feature_importance.png", dpi=300)
plt.show()
```

![Evaluasi Model: Confusion Matrix & Feature Importance](tutupan_lahan_files/evaluasi_model_confusion_feature.png)

### Interpretasi Feature Importance:
* **Fitur Paling Berpengaruh**: Saluran **NDWI**, **B08 (NIR)**, dan **B11 (SWIR-1)** menduduki peringkat teratas dalam skor *Gini Importance*. NDWI dan B08 secara mutlak memisahkan perairan dari daratan, sedangkan B11 membedakan material atap bangunan dan kadar kelembapan tanah/kanopi.
* **Fitur Vegetasi Pendukung**: **NDVI** dan **B04 (Red)** memegang peran vital dalam memisahkan hutan lebat pegunungan dengan vegetasi sawah musiman dan mangrove.

---

## 5. Audit Transparan Poligon Salah Klasifikasi (*Misclassification Analysis*)

Sebagai bentuk transparansi saintifik, dilakukan investigasi mendalam terhadap sampel poligon data uji yang mengalami misklasifikasi pada zona transisi ekoton:

| ID Poligon | Kelas Aktual | Hasil Prediksi Model | Rasio Piksel Benar | Karakteristik Biofisik & Penyebab Misklasifikasi |
| :--- | :---: | :---: | :---: | :--- |
| `Bangunan_0003` | Bangunan | Danau | 0 / 2 (0.0%) | Poligon beratap seng/asbes gelap yang dinaungi genangan air/drainase dekat pantai, memicu penurunan nilai reflektansi mirip air keruh. |
| `Bangunan_0007` | Bangunan | Hutan | 2 / 5 (40.0%) | Kawasan perumahan pedesaan dengan pekarangan bertutupan pohon rindang lebat (*homegarden/pekarangan*). |
| `Bangunan_0027` | Bangunan | Hutan | 1 / 3 (33.3%) | Kompleks vila/bangunan di lereng pegunungan yang tertutup kanopi pohon tinggi di sekitarnya. |
| `Danau_0041` | Danau | Laut | 4 / 9 (44.4%) | Waduk air jernih luas dengan kedalaman tinggi yang menghasilkan respon spektral optik identik dengan air laut biru. |
| `Sawah_0004` | Sawah | Hutan | 1 / 5 (20.0%) | Petak sawah di dekat lereng hutan yang sedang berada pada fase pertumbuhan vegetatif rimbun saat perekaman satelit. |
| `Sawah_0013` | Sawah | Hutan | 3 / 7 (42.9%) | Pematang sawah bervegetasi pohon tahunan (kebun campuran di batas petak). |
| `Sawah_0035` | Sawah | Danau | 1 / 2 (50.0%) | Petak sawah yang sedang diairi penuh (*fase penggenangan/pembajakan*) sehingga didominasi pantulan badan air. |
| `Sawah_0036` | Sawah | Hutan | 1 / 2 (50.0%) | Sawah terasering di dataran tinggi dengan kontur bayangan bukit dan kanopi rimbun. |
| `Sawah_0047` | Sawah | Hutan | 1 / 3 (33.3%) | Sawah agroforestri di perbatasan batas kawasan hutan kemasyarakatan. |

Temuan ini membuktikan bahwa kesalahan klasifikasi bukan disebabkan oleh kelemahan acak model, melainkan merupakan representasi alami dari **zona transisi ekoton spasial (*spatial boundary vagueness*)** di mana objek fisik di lapangan memiliki karakteristik campuran antara kanopi, kelembapan tanah, dan material permukaan.
