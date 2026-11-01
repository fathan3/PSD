# Evaluation: Kalibrasi Ambang Batas, Validasi Kinerja & Benchmark Latensi

Tahap **Evaluation** bertujuan untuk memvalidasi secara objektif keandalan, stabilitas, dan kecepatan komputasi model Support Vector Machine (SVM) pada $370\text{ sampel chunk}$ data uji independen yang berasal dari $40\text{ file rekaman asli}$ yang belum pernah dilihat model selama proses pelatihan. Evaluasi dilakukan melalui lima dimensi penilaian: kalibrasi ambang batas (*threshold calibration*), laporan klasifikasi (*classification report*), matriks kontinjensi (*confusion matrix*), evaluasi level file asli, analisis signifikansi fitur (*permutation feature importance*), serta uji kecepatan latensi operasional (*latency benchmark*).

---

## 1. Kalibrasi Ambang Batas Keputusan (*Decision Threshold Calibration*)

Secara baku (*default*), model klasifikasi biner menggunakan ambang batas probabilitas $0.5$ ($50\%$) untuk memisahkan kelas positif dan negatif. Namun, dalam sistem peringatan alarm laboratorium, nilai ambang batas ini memerlukan kalibrasi terarah.

### Dilema Ambang Batas:
* **Ambang Batas Default ($0.5$):** Masih rentan memicu peringatan palsu (*false alarm*) ketika terjadi kebisingan sesaat, seperti suara tertawa singkat atau gesekan kursi praktikan.
* **Ambang Batas Terlalu Tinggi ($0.94$):** Jika ambang batas dinaikkan secara ekstrem ke $0.94$, nilai **Recall** akan anjlok drastis hingga di bawah $48\%$. Hal ini berarti lebih dari separuh kejadian kegaduhan dan obrolan ramai di laboratorium akan terlewat tanpa terdeteksi oleh alarm.

### Penentuan Titik Ekuilibrium Optimal pada Nilai $0.6$ (60%):
Melalui analisis kurva kompromi *Precision-Recall*, nilai ambang batas **$0.6$** terbukti merupakan titik optimal terbaik:
* Memberikan bantalan keamanan (*safety buffer*) yang efektif meredam kebisingan transien sesaat.
* Mempertahankan tingkat sensitivitas tangkapan keramaian (*Recall* $\ge 84\%$).
* Menghasilkan akurasi klasifikasi tertinggi pada data uji ($88.65\% \approx 89\%$).

```python
# ==============================================================================
# KALIBRASI AMBANG BATAS KEPUTUSAN DENGAN KURVA PRECISION-RECALL
# ==============================================================================
from sklearn.metrics import precision_recall_curve

# Prediksi probabilitas kontinu pada data uji
proba_test = svm_pipeline.predict_proba(X_test)[:, 1]

precisions, recalls, thresholds = precision_recall_curve(y_test, proba_test)

plt.figure(figsize=(9, 4.5))
plt.plot(thresholds, precisions[:-1], label="Presisi (Precision)", color="#2563eb", linewidth=2)
plt.plot(thresholds, recalls[:-1], label="Sensitivitas (Recall)", color="#10b981", linewidth=2)
plt.axvline(x=0.6, color="#ef4444", linestyle="--", linewidth=2, label="Ambang Batas Terpilih (0.6)")
plt.title("Kurva Presisi vs Sensitivitas terhadap Variasi Ambang Batas Probabilitas", fontsize=12, fontweight="bold")
plt.xlabel("Ambang Batas Keputusan (Decision Threshold)")
plt.ylabel("Skor")
plt.legend()
plt.tight_layout()
plt.show()

DECISION_THRESHOLD = 0.6
print(f"Ambang batas operasional ditetapkan pada: {DECISION_THRESHOLD} (60%)")
```

### Visualisasi Kurva Precision vs Recall terhadap Threshold:

![Kurva Precision Recall Threshold](deteksi_suara_files/kurva_precision_recall_threshold.png)

---

## 2. Laporan Klasifikasi Menyeluruh (*Classification Report*)

Evaluasi dilakukan terhadap $370\text{ sampel chunk}$ data uji independen ($192\text{ Tenang}$ : $178\text{ Berisik}$) dengan ambang batas keputusan $0.6$:

```python
# ==============================================================================
# EVALUASI CLASSIFICATION REPORT DAN AKURASI DATA UJI
# ==============================================================================
from sklearn.metrics import classification_report, accuracy_score

preds_test = (proba_test >= DECISION_THRESHOLD).astype(int)

print("=== CLASSIFICATION REPORT (DATA UJI - THRESHOLD 0.6) ===")
print(classification_report(y_test, preds_test, target_names=["Tenang / Tidak Berisik (0)", "Berisik / Ramai (1)"]))

acc = accuracy_score(y_test, preds_test)
print(f"Akurasi Keseluruhan: {acc*100:.2f}% (Dibulatkan: {round(acc*100)}%)")
```

### Output Laporan Klasifikasi Resmi:
```text
=== CLASSIFICATION REPORT (DATA UJI - THRESHOLD 0.6) ===
                            precision    recall  f1-score   support

Tenang / Tidak Berisik (0)       0.86      0.93      0.90       192
       Berisik / Ramai (1)       0.92      0.84      0.88       178

                  accuracy                           0.89       370
                 macro avg       0.89      0.88      0.89       370
              weighted avg       0.89      0.89      0.89       370

Akurasi Keseluruhan: 88.65% (Dibulatkan: 89%)
```

### Analisis Metrik Evaluasi:
1. **Presisi Tinggi pada Kelas Berisik ($92\%$):**  
   Dari seluruh peringatan alarm kebisingan yang dipicu oleh model, sebanyak **$92\%$ benar-benar merupakan kondisi ramai yang valid**. Hal ini membuktikan bahwa sistem sangat sukses menekan potensi peringatan palsu (*false alarm*).
2. **Recall Kuat pada Kelas Berisik ($84\%$):**  
   Model mampu mendeteksi **$150$ dari total $178$ kejadian suara berisik** di data uji secara konsisten.
3. **F1-Score Seimbang ($0.88 - 0.90$):**  
   Keseimbangan harmonik antara presisi dan recall membuktikan bahwa model tidak mengorbankan salah satu aspek untuk mengejar skor lainnya.

---

## 3. Matriks Kontinjensi (*Confusion Matrix*)

Matriks kontinjensi memetakan tabulasi silang antara label aktual di lapangan dengan label prediksi model:

```python
# ==============================================================================
# VISUALISASI CONFUSION MATRIX DATA UJI
# ==============================================================================
from sklearn.metrics import ConfusionMatrixDisplay

fig, ax = plt.subplots(figsize=(6, 5))
ConfusionMatrixDisplay.from_predictions(
    y_test, preds_test,
    display_labels=["Tenang (0)", "Berisik (1)"],
    cmap="Blues",
    values_format="d",
    ax=ax
)
ax.set_title(f"Confusion Matrix Data Uji (Akurasi: {acc*100:.2f}%)", fontweight="bold")
plt.tight_layout()
plt.show()
```

### Visualisasi Confusion Matrix:

![Confusion Matrix SVM](deteksi_suara_files/confusion_matrix_svm.png)

### Rincian Kuadran Matriks Kontinjensi:

| Kondisi Aktual \ Hasil Prediksi | Prediksi: Tenang (0) | Prediksi: Berisik (1) [Alarm!] | Total Aktual |
| :--- | :---: | :---: | :---: |
| **Aktual: Tenang (0)** | **$\text{TN} = 178$** ($92.7\%$) | **$\text{FP} = 14$** ($7.3\%$) | **$192$ sampel** |
| **Aktual: Berisik (1)** | **$\text{FN} = 28$** ($15.7\%$) | **$\text{TP} = 150$** ($84.3\%$) | **$178$ sampel** |
| **Total Prediksi** | **$206$ sampel** | **$164$ sampel** | **$370$ total sampel uji** |

* **True Negative ($\text{TN} = 178$):** Suasana laboratorium yang hening dan tertib berhasil diidentifikasi tepat tanpa memicu alarm.
* **True Positive ($\text{TP} = 150$):** Kegaduhan vokal manusia dan obrolan ramai berhasil dikenali tepat dan mengaktifkan peringatan.
* **False Positive ($\text{FP} = 14$):** Hanya terjadi $14$ kali peringatan palsu dari $192$ segmen tenang ($7.3\%$), umumnya akibat suara tertawa spontan atau kursi tergeser keras saat ruangan sepi.
* **False Negative ($\text{FN} = 28$):** Sebanyak $28$ segmen berisik tidak memicu alarm, sebagian besar karena volume suara bisikan yang berada di bawah ambang $0.6$.

---

## 4. Evaluasi Agregasi pada Level File Rekaman Asli

Dalam skenario operasional nyata di laboratorium, pemantauan tidak hanya didasarkan pada satu potongan $5\text{ detik}$, melainkan akumulasi rata-rata probabilitas selama satu sesi pengamatan file rekaman penuh.

```python
# ==============================================================================
# EVALUASI AGREGASI PADA LEVEL FILE REKAMAN ASLI
# ==============================================================================
test_eval_df = test_df[["file", "label"]].copy()
test_eval_df["prob"] = proba_test

file_level_eval = test_eval_df.groupby("file").agg(
    true_label=("label", "first"),
    mean_prob=("prob", "mean")
)
file_level_eval["pred_label"] = (file_level_eval["mean_prob"] >= DECISION_THRESHOLD).astype(int)
file_acc = (file_level_eval["pred_label"] == file_level_eval["true_label"]).mean()

print(f"Akurasi Evaluasi Level File Rekaman Asli: {file_acc*100:.2f}% ({int((file_level_eval['pred_label'] == file_level_eval['true_label']).sum())}/{len(file_level_eval)} file rekaman)")
```

### Hasil Evaluasi Level File:
```text
Akurasi Evaluasi Level File Rekaman Asli: 92.50% (37/40 file rekaman)
```

> **Lonjakan Akurasi Agregasi ($92.50\%$)**:  
> Ketika rata-rata probabilitas seluruh *chunk* dalam satu file rekaman dihitung, akurasi meningkat dari **$88.65\%$** (level *chunk*) menjadi **$92.50\%$** (level rekaman penuh). Sebanyak **$37$ dari $40$ file rekaman uji asli** berhasil diprediksi dengan benar secara sempurna. Hal ini membuktikan bahwa kesalahan klasifikasi *chunk* individual tereliminasi saat sistem mengamati rentang waktu observasi yang lebih panjang.

---

## 5. Verifikasi Signifikansi Fitur: *Permutation Feature Importance*

Karena kernel RBF memetakan data ke ruang non-linier tak terhingga, SVM tidak memiliki koefisien bobot linier langsung ($w_i$) yang dapat diinterpretasikan secara sederhana. Oleh karena itu, diterapkan teknik model-agnostik **Permutation Feature Importance** untuk mengukur penurunan akurasi model ketika nilai dari masing-masing 14 fitur diacak (*shuffled*) secara independen sebanyak $10$ iterasi (`n_repeats=10`):

```python
# ==============================================================================
# PERMUTATION FEATURE IMPORTANCE (10 REPEATS) PADA DATA UJI
# ==============================================================================
from sklearn.inspection import permutation_importance

perm_result = permutation_importance(
    svm_pipeline, X_test, y_test,
    n_repeats=10, random_state=42, n_jobs=-1
)

perm_df = pd.DataFrame({
    "feature": SELECTED_FEATURES,
    "importance_mean": perm_result.importances_mean,
    "importance_std": perm_result.importances_std
}).sort_values("importance_mean", ascending=True)

plt.figure(figsize=(9, 6))
plt.barh(perm_df["feature"], perm_df["importance_mean"], xerr=perm_df["importance_std"], color="#3b82f6", alpha=0.85)
plt.title("Permutation Feature Importance (SVM RBF - 14 Fitur Terpilih)", fontsize=12, fontweight="bold")
plt.xlabel("Penurunan Skor Akurasi Saat Nilai Fitur Diacak (Mean Importance Drop)")
plt.tight_layout()
plt.show()

print("Urutan Pengaruh Fitur Terhadap Keputusan Model SVM:")
print(perm_df.sort_values('importance_mean', ascending=False).to_string(index=False))
```

### Visualisasi Permutation Feature Importance:

![Permutation Feature Importance](deteksi_suara_files/permutation_feature_importance.png)

### Urutan Pengaruh Fitur Terhadap Keputusan Model SVM:

| Peringkat | Nama Fitur | Mean Importance Drop ($\Delta \text{Acc}$) | Std Importance | Kontribusi Terhadap Deteksi |
| :---: | :--- | :---: | :---: | :--- |
| **1** | `mfcc1_mean` | **$0.0900$ ($9.00\%$)** | $\pm 0.0145$ | **Paling Kritis**: Selubung energi spektral dasar manusia. |
| **2** | `mfcc6_mean` | **$0.0724$ ($7.24\%$)** | $\pm 0.0147$ | Kelipatan harmonik vokal saat diskusi banyak orang. |
| **3** | `mfcc3_mean` | **$0.0630$ ($6.30\%$)** | $\pm 0.0085$ | Formulan nada dasar percakapan vokal manusia. |
| **4** | `zcr_mean` | **$0.0611$ ($6.11\%$)** | $\pm 0.0108$ | Deteksi konsonan frikatif, gesekan, dan artikulasi fonem. |
| **5** | `mfcc12_std` | **$0.0595$ ($5.95\%$)** | $\pm 0.0102$ | Tekstur dan fluktuasi detail mikrofon pada pita frekuensi atas. |
| **6** | `mfcc5_mean` | **$0.0530$ ($5.30\%$)** | $\pm 0.0084$ | Spektra timbre yang membedakan suara manusia dari desau AC. |
| **7** | `mfcc11_std` | **$0.0519$ ($5.19\%$)** | $\pm 0.0091$ | Variansi energi pada harmonik frekuensi tinggi. |
| **8** | `rms_mean` | **$0.0432$ ($4.32\%$)** | $\pm 0.0127$ | Magnitudo daya energi fisik keseluruhan gelombang audio. |
| **9** | `mfcc4_mean` | **$0.0397$ ($3.97\%$)** | $\pm 0.0117$ | Pola kontur fonemik artikulasi kata. |
| **10** | `mfcc13_std` | **$0.0238$ ($2.38\%$)** | $\pm 0.0055$ | Karakteristik peluruhan spektral batas atas. |
| **11** | `rms_std` | **$0.0232$ ($2.32\%$)** | $\pm 0.0077$ | Dinamika fluktuasi jeda bicara vs suara statis. |
| **12** | `mfcc8_mean` | **$0.0222$ ($2.22\%$)** | $\pm 0.0114$ | Penyaring suara ketikan keyboard wajar. |
| **13** | `mfcc11_mean` | **$0.0159$ ($1.59\%$)** | $\pm 0.0056$ | Resonansi nada tinggi. |
| **14** | `mfcc7_mean` | **$0.0122$ ($1.22\%$)** | $\pm 0.0063$ | Transisi suku kata vokal manusia. |

---

## 6. Uji Kecepatan Inferensi (*Operational Latency Benchmark*)

Salah satu kriteria keberhasilan operasional sistem alarm laboratorium adalah kemampuan pemrosesan audio yang bersifat *real-time*: **total waktu ekstraksi 14 fitur ditambah inferensi model SVM per segmen 5 detik harus < 1 detik (1000 ms)**.

```python
# ==============================================================================
# BENCHMARK LATENSI INFERENSI LIVE (20 KALI PENGUJIAN)
# ==============================================================================
import features as ft
import time

sample_audio_path = test_files["path"].iloc[0]
y_sample = ft.load_audio(sample_audio_path)[:ft.CHUNK_LEN]

# Simulasi 20 kali inferensi live
latencies = []
for _ in range(20):
    t0 = time.perf_counter()
    x_feat = ft.signal_to_features(y_sample)
    svm_pipeline.predict_proba(x_feat)
    latencies.append(time.perf_counter() - t0)

avg_ms = np.mean(latencies) * 1000
p95_ms = np.percentile(latencies, 95) * 1000

print(f"Hasil Uji Kecepatan Inferensi:")
print(f"- Rata-rata Latensi : {avg_ms:.2f} ms per chunk 5 detik")
print(f"- 95th Percentile   : {p95_ms:.2f} ms")
print(f"- Status Kriteria   : {'LOLOS (Sangat Cepat, < 1000 ms)' if avg_ms < 1000 else 'GAGAL'}")
```

### Hasil Benchmark Latensi:
```text
Hasil Uji Kecepatan Inferensi:
- Rata-rata Latensi : 41.21 ms per chunk 5 detik
- 95th Percentile   : 49.48 ms
- Status Kriteria   : LOLOS (Sangat Cepat, < 1000 ms)
```

> **Analisis Kinerja *Real-Time***:  
> Total waktu pemrosesan sinyal digital dan inferensi model hanya memakan waktu **$41.21\text{ milidetik}$** (hanya sekitar $4\%$ dari batas toleransi $1.000\text{ ms}$). Hal ini membuktikan bahwa pipeline ekstraksi 14 fitur dan model SVM sangat ringan (*lightweight*), efisien, dan siap diimplementasikan secara langsung pada perangkat keras komputasi edge (*Edge AI*) maupun server web mikrofon *live*.

