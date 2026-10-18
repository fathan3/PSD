# Modeling: Multi-Algoritma Machine Learning & Ensemble Learning

Tahap **Modeling** menguraikan formulasi teoritis, arsitektur komputasi, dan implementasi kode Python dari tiga model Machine Learning yang dikembangkan untuk klasifikasi tutupan lahan Provinsi Jawa Timur: **Random Forest Classifier**, **HistGradientBoosting Classifier**, dan **Ensemble Soft-Voting Classifier (RF + HGB)**.

---

## 1. Arsitektur & Formulasi Matematis Tiga Model Machine Learning

Guna menghasilkan model dengan akurasi dan daya generalisasi tertinggi pada dataset citra satelit Sentinel-2A Level-2A, dilakukan perancangan dan pelatihan tiga model komparasi:

### 1.1 Model 1: Random Forest Classifier (Baseline Standar Penginderaan Jauh)
* **Konsep Dasar**: Algoritma *ensemble bagging* (*Bootstrap Aggregating*) yang mengagregasi 100 pohon keputusan (*decision trees*) independen secara paralel. Setiap pohon dilatih menggunakan sampel acak dengan pengembalian (*bootstrap sample*), dan pada setiap percabangan node (*split node*) hanya diuji subset acak fitur berukuran $\sqrt{N_{\text{fitur}}} = \sqrt{9} = 3$ fitur.
* **Penyeimbang Bobot Kelas Adaptif (`class_weight='balanced'`)**:  
  Dataset tutupan lahan memiliki ketimpangan frekuensi kelas yang sangat tajam (Laut memiliki $128.000$ piksel, sedangkan Bangunan hanya $1.800$ piksel). Tanpa penyeimbang, pohon keputusan akan terbias pada kelas mayoritas. Bobot kelas dihitung secara otomatis menggunakan formulasi:
  $$w_c = \frac{N_{\text{total}}}{K \times N_c}$$
  di mana $N_{\text{total}} = 227.636$ adalah total sampel piksel data latih, $K = 6$ adalah jumlah kelas LULC, dan $N_c$ adalah jumlah sampel piksel kelas $c$.
* **Kriteria Pemisahan Node (Gini Impurity)**:
  $$I_G(t) = 1 - \sum_{i=1}^{K} p(i \mid t)^2$$
  di mana $p(i \mid t)$ adalah proporsi sampel kelas $i$ yang berada pada node $t$.
* **Keunggulan**: Sangat stabil terhadap autokorelasi multivariat antar-indeks spektral, tidak membutuhkan normalisasi penskalaan fitur numerik, serta menghasilkan metrik *Gini Feature Importance* yang dapat diinterpretasikan secara biofisik.

---

### 1.2 Model 2: HistGradientBoosting Classifier (Gradient Boosting Setara LightGBM)
* **Konsep Dasar**: Algoritma *ensemble boosting* sekuensial yang membangun pohon keputusan secara bertahap (*stage-wise additive model*) untuk meminimalkan fungsi loss residual dari pohon-pohon sebelumnya.
* **Histogram Integer Binning (256 Bins)**:  
  Seluruh fitur numerik kontinu dikuantisasi ke dalam 256 bin bilangan bulat diskrit (*integer binning*). Teknik ini mereduksi kompleksitas komputasi penelusuran titik pisah (*best split point*) dari $\mathcal{O}(N_{\text{data}} \times N_{\text{fitur}})$ menjadi $\mathcal{O}(N_{\text{bin}} \times N_{\text{fitur}})$, menghasilkan akselerasi pelatihan hingga 10x lebih cepat pada ratusan ribu data piksel.
* **Optimasi Fungsi Loss Multikelas (Multinomial Cross-Entropy / Log-Loss)**:  
  Setiap pohon baru mencocokkan nilai residual negatif gradien (*pseudo-residuals*):
  $$r_{ik} = -\left[\frac{\partial L(y_i, f(x_i))}{\partial f_k(x_i)}\right] = y_{ik} - p_k(x_i)$$
  di mana $p_k(x_i) = \frac{e^{f_k(x_i)}}{\sum_{j=1}^K e^{f_j(x_i)}}$ adalah probabilitas softmax sampel $i$ terhadap kelas $k$.

---

### 1.3 Model 3: Ensemble Soft-Voting Classifier (RF + HGB) 🏆 (MODEL TERBAIK)
* **Konsep Dasar**: Memadukan kekuatan dua paradigma *ensemble* yang komplementer—**reduksi variansi** dari paradigma *bagging* (Random Forest) dan **reduksi bias** dari paradigma *gradient boosting* (HistGradientBoosting)—melalui rata-rata probabilitas posterior terbobot (*Soft-Voting*):
  $$P_{\text{ensemble}}(y = c \mid X) = \frac{1}{2} \Big( P_{\text{RF}}(y = c \mid X) + P_{\text{HGB}}(y = c \mid X) \Big)$$
* **Aturan Keputusan Prediksi Akhir**:
  $$\hat{y} = \arg\max_{c \in \{1,\dots,6\}} P_{\text{ensemble}}(y = c \mid X)$$
* **Keunggulan Empiris Terbukti**:
  1. *Dekomposisi Bias-Variansi*: Menekan error prediksi yang bersumber dari fluktuasi pohon individual pada RF maupun sensitivitas boosting terhadap piksel noise/awan tipis.
  2. *Peningkatan Presisi Kelas Minoritas*: Presisi deteksi pada kelas Bangunan meningkat drastis menjadi **71.93%** dengan recall **80.00%**.
  3. *Akurasi Global Tertinggi*: Berhasil membukukan **Akurasi Global 89.06%** dan **Macro F1-Score 81.21%**, melampaui kedua model tunggalnya.

---

## 2. Implementasi Kode Pelatihan Tiga Model di Python

Berikut adalah implementasi kode Python lengkap untuk melatih ketiga model Machine Learning dan menyimpan seluruh objek model serialisasi ke dalam format `.joblib`:

```python
# ==============================================================================
# FASE 4.1 PELATIHAN 3 ARSITEKTUR MODEL MACHINE LEARNING
# ==============================================================================

# 1. Model 1: Random Forest Classifier
print(">> [1/3] Melatih Random Forest Classifier (100 Estimators, Class Balanced)...")
rf_model = RandomForestClassifier(
    n_estimators=100,
    max_depth=16,
    class_weight="balanced",
    random_state=42,
    n_jobs=-1
)
rf_model.fit(X_train, y_train)
y_pred_rf = rf_model.predict(X_test)
print("✅ Random Forest selesai dilatih!")

# 2. Model 2: HistGradientBoosting Classifier
print(">> [2/3] Melatih HistGradientBoosting Classifier (Gradient Boosting)...")
hgb_model = HistGradientBoostingClassifier(
    max_iter=100,
    learning_rate=0.1,
    random_state=42
)
hgb_model.fit(X_train, y_train)
y_pred_hgb = hgb_model.predict(X_test)
print("✅ HistGradientBoosting selesai dilatih!")

# 3. Model 3: Ensemble Soft-Voting Classifier (RF + HGB) -> MODEL TERBAIK
print(">> [3/3] Melatih Ensemble Soft-Voting (Random Forest + HistGradientBoosting)...")
ensemble_model = VotingClassifier(
    estimators=[("rf", rf_model), ("hgb", hgb_model)],
    voting="soft",
    n_jobs=-1
)
ensemble_model.fit(X_train, y_train)
y_pred_ens = ensemble_model.predict(X_test)
print("✅ Ensemble Soft-Voting selesai dilatih!")

# Simpan ketiga model ke format serialisasi joblib
joblib.dump(rf_model, OUT_DIR / "model_rf.joblib")
joblib.dump(hgb_model, OUT_DIR / "model_hgb.joblib")
joblib.dump(ensemble_model, OUT_DIR / "model_ensemble.joblib")
print("✅ Ketiga model tersimpan di direktori hasil_evaluasi/")
```

**Keluaran Terminal:**
```text
>> [1/3] Melatih Random Forest Classifier (100 Estimators, Class Balanced)...
✅ Random Forest selesai dilatih!
>> [2/3] Melatih HistGradientBoosting Classifier (Gradient Boosting)...
✅ HistGradientBoosting selesai dilatih!
>> [3/3] Melatih Ensemble Soft-Voting (Random Forest + HistGradientBoosting)...
✅ Ensemble Soft-Voting selesai dilatih!
✅ Ketiga model tersimpan di direktori hasil_evaluasi/
```

Seluruh artefak model yang tersimpan di direktori `hasil_evaluasi/` siap diintegrasikan ke dalam dashboard aplikasi web geospasial Streamlit Cloud.
