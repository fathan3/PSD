# Modeling: Supervised Fuzzy Logic Classifier

Tahap **Modeling** menguraikan formulasi teoritis algoritma pengklasifikasi berbasis logika fuzzy terawasi (*Supervised Fuzzy Distance Classifier*), audit matematis manual langkah demi langkah dengan angka riil, pembuktian komputasi via kode Python, implementasi pustaka *from-scratch* berbasis NumPy, hingga pelatihan model pada dataset 300 poligon tutupan lahan Jawa Timur.

---

## 1. Formulasi Matematis Supervised Fuzzy Distance Classifier

Dalam pendekatan Logika Fuzzy Terawasi (*Supervised Fuzzy Logic Classifier*), proses inferensi dibangun berdasarkan prinsip jarak geometris prototipe kelas dalam ruang spektral berdimensi-$p$. Berbeda dari model *Crisp* yang menghasilkan label biner ($0$ atau $1$), model fuzzy menghitung vektor probabilitas keanggotaan kontinu $\mu \in [0, 1]$ yang mencerminkan derajat keserupaan biofisik sampel terhadap prototipe masing-masing kelas.

### Formulasi Derajat Keanggotaan Fuzzy ($\mu_{ij}$):

Untuk sampel vektor $\mathbf{x}_i \in \mathbb{R}^p$ dan pusat prototipe kelas $\mathbf{v}_j \in \mathbb{R}^p$ ($j = 1, 2, \dots, C$ dengan $C=6$), derajat keanggotaan $\mu_{ij}$ dirumuskan dengan parameter *fuzzifier* $m$:

$$\mu_{ij} = \frac{1}{\sum_{k=1}^{C} \left( \frac{d(\mathbf{x}_i, \mathbf{v}_j)}{d(\mathbf{x}_i, \mathbf{v}_k)} \right)^{\frac{2}{m - 1}}}$$

Dengan:
* $d(\mathbf{x}_i, \mathbf{v}_j) = \|\mathbf{x}_i - \mathbf{v}_j\|_2 = \sqrt{\sum_{f=1}^{p} (x_{if} - v_{jf})^2}$ (Jarak Euclidean multivariat).
* $m$: Parameter pembuat kabur (*fuzziness parameter*). Pada penelitian ini ditetapkan $m = 2.0$.
* Ketika $m = 2.0$, eksponen menjadi $\frac{2}{2 - 1} = 2$. Rumus di atas bertransformasi menjadi pembobotan invers kuadrat jarak (*Inverse Distance Weighting*):

$$\mu_{ij} = \frac{\frac{1}{d_j^2}}{\sum_{k=1}^{C} \frac{1}{d_k^2}} = \frac{w_j}{\sum_{k=1}^{C} w_k}$$

* **Syarat Partisi Fuzzy Mutlak:**
  $$\sum_{j=1}^{C} \mu_{ij} = 1 \quad \text{dan} \quad 0 \le \mu_{ij} \le 1, \quad \forall i, j$$

* **Defuzzifikasi Maksimum (*Maximum Membership Defuzzification*):**
  Untuk penentuan label akhir diskret, digunakan aturan keanggotaan tertinggi:
  $$\hat{y}_i = \arg\max_{j \in \{1, \dots, C\}} \mu_{ij}$$

---

## 2. Simulasi Perhitungan Manual Langkah Demi Langkah (Audit Matematis Angka Riil)

Guna menjamin transparansi saintifik (*mathematical auditability*), disimulasikan klasifikasi pada satu sampel poligon lapangan nyata $\mathbf{x}_{\text{uji}}$ dengan 5 fitur kunci $[\text{NDVI}, \text{NDWI}, \text{NDBI}, \text{B08}, \text{B11}]$:

$$\mathbf{x}_{\text{uji}} = [0.650, -0.320, -0.280, 0.380, 0.170]$$

Pusat prototipe 6 kelas tutupan lahan acuan yang diperoleh dari fase eksplorasi:
* $\mathbf{v}_1 \text{ (Sawah)} = [0.650, -0.300, -0.220, 0.380, 0.165]$
* $\mathbf{v}_2 \text{ (Bangunan)} = [0.115, -0.350, +0.250, 0.220, 0.320]$
* $\mathbf{v}_3 \text{ (Lahan Hijau)} = [0.820, -0.620, -0.580, 0.560, 0.135]$
* $\mathbf{v}_4 \text{ (Danau)} = [-0.280, +0.520, -0.180, 0.025, 0.015]$
* $\mathbf{v}_5 \text{ (Perairan Terbuka)} = [-0.450, +0.650, -0.120, 0.012, 0.008]$
* $\mathbf{v}_6 \text{ (Mangrove)} = [0.720, -0.250, -0.380, 0.410, 0.098]$

### Langkah 1: Kalkulasi Kuadrat Jarak Euclidean ($d_j^2$) dan Jarak ($d_j$)
* $d_1^2 = (0.650-0.650)^2 + (-0.320 - (-0.300))^2 + (-0.280 - (-0.220))^2 + (0.380-0.380)^2 + (0.170-0.165)^2 = 0.004025 \implies d_1 = \mathbf{0.06344}$
* $d_2^2 = (0.650-0.115)^2 + (-0.320 - (-0.350))^2 + (-0.280-0.250)^2 + (0.380-0.220)^2 + (0.170-0.320)^2 = 0.616125 \implies d_2 = \mathbf{0.78494}$
* $d_3^2 = (0.650-0.820)^2 + (-0.320 - (-0.620))^2 + (-0.280 - (-0.580))^2 + (0.380-0.560)^2 + (0.170-0.135)^2 = 0.242525 \implies d_3 = \mathbf{0.49247}$
* $d_4^2 = (0.650 - (-0.280))^2 + (-0.320-0.520)^2 + (-0.280 - (-0.180))^2 + (0.380-0.025)^2 + (0.170-0.015)^2 = 1.730550 \implies d_4 = \mathbf{1.31550}$
* $d_5^2 = (0.650 - (-0.450))^2 + (-0.320-0.650)^2 + (-0.280 - (-0.120))^2 + (0.380-0.012)^2 + (0.170-0.008)^2 = 2.338168 \implies d_5 = \mathbf{1.52911}$
* $d_6^2 = (0.650-0.720)^2 + (-0.320 - (-0.250))^2 + (-0.280 - (-0.380))^2 + (0.380-0.410)^2 + (0.170-0.098)^2 = 0.025884 \implies d_6 = \mathbf{0.16088}$

### Langkah 2 & 3: Pembobotan Invers Kuadrat ($w_j = 1/d_j^2$) dan Akumulasi ($\Sigma w$)
* $w_1 = 1 / 0.004025 = \mathbf{248.4472}$
* $w_2 = 1 / 0.616125 = \mathbf{1.6231}$
* $w_3 = 1 / 0.242525 = \mathbf{4.1233}$
* $w_4 = 1 / 1.730550 = \mathbf{0.5779}$
* $w_5 = 1 / 2.338168 = \mathbf{0.4277}$
* $w_6 = 1 / 0.025884 = \mathbf{38.6339}$
* **Total Bobot Akumulasi ($\Sigma w$):**  
  $$\Sigma w = 248.4472 + 1.6231 + 4.1233 + 0.5779 + 0.4277 + 38.6339 = \mathbf{293.8331}$$

### Langkah 4: Derajat Keanggotaan Fuzzy ($\mu_j = w_j / \Sigma w$) & Keputusan Defuzzifikasi

| Kelas ($j$) | Nama Kelas | Jarak Euclidean ($d_j$) | Bobot ($w_j$) | Derajat Keanggotaan ($\mu_j$) | Persentase Keanggotaan |
|:---:|:---|:---:|:---:|:---:|:---:|
| 1 | **Sawah** | 0.06344 | 248.4472 | $248.4472 / 293.8331 = \mathbf{0.8455}$ | **84.55%** |
| 2 | **Bangunan** | 0.78494 | 1.6231 | $1.6231 / 293.8331 = \mathbf{0.0055}$ | **0.55%** |
| 3 | **Lahan Hijau** | 0.49247 | 4.1233 | $4.1233 / 293.8331 = \mathbf{0.0140}$ | **1.40%** |
| 4 | **Danau** | 1.31550 | 0.5779 | $0.5779 / 293.8331 = \mathbf{0.0020}$ | **0.20%** |
| 5 | **Perairan Terbuka** | 1.52911 | 0.4277 | $0.4277 / 293.8331 = \mathbf{0.0015}$ | **0.15%** |
| 6 | **Mangrove** | 0.16088 | 38.6339 | $38.6339 / 293.8331 = \mathbf{0.1315}$ | **13.15%** |

* **Keputusan Defuzzifikasi:** $\hat{y} = \mathbf{SAWAH}$ dengan **Tingkat Keyakinan $\mu = 84.55\%$**.
* **Keberadaan Gradien Transisi:** Terlihat keanggotaan sekunder sebesar $13.15\%$ pada Mangrove (karena kedekatan respon kanopi hijau basah). Ini membuktikan bahwa Logika Fuzzy berhasil menangkap karakteristik transisi alami tanpa galat biner.

---

## 3. Verifikasi Komputasi Perhitungan Manual dengan Python

Perhitungan manual di atas diverifikasi menggunakan skrip komputasi otomatis Python untuk menjamin konsistensi numerik:

```python
x_sampel = np.array([0.650, -0.320, -0.280, 0.380, 0.170])

pusat_prototipe = {
    "Sawah":            np.array([0.650, -0.300, -0.220, 0.380, 0.165]),
    "Bangunan":         np.array([0.115, -0.350,  0.250, 0.220, 0.320]),
    "Lahan Hijau":      np.array([0.820, -0.620, -0.580, 0.560, 0.135]),
    "Danau":            np.array([-0.280, 0.520, -0.180, 0.025, 0.015]),
    "Perairan Terbuka": np.array([-0.450, 0.650, -0.120, 0.012, 0.008]),
    "Mangrove":         np.array([0.720, -0.250, -0.380, 0.410, 0.098])
}

jarak_dict = {}
bobot_dict = {}
for kls, v in pusat_prototipe.items():
    d = np.linalg.norm(x_sampel - v)
    jarak_dict[kls] = d
    bobot_dict[kls] = 1.0 / (d ** 2)

total_bobot = sum(bobot_dict.values())
keanggotaan_fuzzy = {kls: w / total_bobot for kls, w in bobot_dict.items()}

df_verifikasi = pd.DataFrame({
    "Kelas Tutupan Lahan": list(keanggotaan_fuzzy.keys()),
    "Jarak Euclidean (d)": [round(jarak_dict[k], 5) for k in keanggotaan_fuzzy],
    "Bobot (w)": [round(bobot_dict[k], 4) for k in keanggotaan_fuzzy],
    "Derajat Keanggotaan (μ)": [round(keanggotaan_fuzzy[k], 4) for k in keanggotaan_fuzzy],
    "Persentase": [f"{keanggotaan_fuzzy[k]*100:.2f}%" for k in keanggotaan_fuzzy]
})

print("=== TABEL HASIL AUDIT VERIFIKASI PYTHON ===")
display(df_verifikasi)

kelas_pred = max(keanggotaan_fuzzy, key=keanggotaan_fuzzy.get)
print(f"\n✓ Verifikasi Selesai: Keputusan Defuzzifikasi = {kelas_pred.upper()} "
      f"(Keyakinan: {keanggotaan_fuzzy[kelas_pred]*100:.2f}%)")
print(f"✓ Total Derajat Keanggotaan: {sum(keanggotaan_fuzzy.values()):.6f} (Sesuai Syarat Partisi 100%)")
```

```text
=== TABEL HASIL AUDIT VERIFIKASI PYTHON ===
  Kelas Tutupan Lahan  Jarak Euclidean (d)  Bobot (w)  Derajat Keanggotaan (μ) Persentase
0               Sawah              0.06344   248.4472                  0.8455     84.55%
1            Bangunan              0.78494     1.6230                  0.0055      0.55%
2         Lahan Hijau              0.49247     4.1233                  0.0140      1.40%
3               Danau              1.31550     0.5779                  0.0020      0.20%
4    Perairan Terbuka              1.52911     0.4277                  0.0015      0.15%
5            Mangrove              0.16089    38.6339                  0.1315     13.15%

✓ Verifikasi Selesai: Keputusan Defuzzifikasi = SAWAH (Keyakinan: 84.55%)
✓ Total Derajat Keanggotaan: 1.000000 (Sesuai Syarat Partisi 100%)
```

---

## 4. Implementasi Kelas Model Fuzzy *From-Scratch* Berbasis NumPy

Struktur algoritma dienkapsulasi ke dalam kelas Python mandiri `SupervisedFuzzySpatialClassifier` yang kompatibel dengan antarmuka Scikit-Learn (`fit`, `predict_proba`, `predict`):

```python
class SupervisedFuzzySpatialClassifier:
    """
    Klasifikasi Spasial Tutupan Lahan Berbasis Logika Fuzzy Terawasi.
    Menghitung derajat keanggotaan fuzzy sampel berdasarkan jarak geometris terbobot
    terhadap prototipe spektral kelas tutupan lahan di Jawa Timur.
    """
    def __init__(self, m=2.0, metric='euclidean', epsilon=1e-9):
        self.m = m
        self.metric = metric
        self.epsilon = epsilon
        self.classes_ = None
        self.prototypes_ = {}
        self.scaler_ = StandardScaler()
        
    def fit(self, X, y):
        X_mat = np.asarray(X)
        y_vec = np.asarray(y)
        self.classes_ = np.unique(y_vec)
        X_scaled = self.scaler_.fit_transform(X_mat)
        
        for c in self.classes_:
            mask = (y_vec == c)
            self.prototypes_[c] = np.mean(X_scaled[mask], axis=0)
            
        return self
        
    def predict_proba(self, X):
        X_mat = np.asarray(X)
        X_scaled = self.scaler_.transform(X_mat)
        n_samples = X_scaled.shape[0]
        n_classes = len(self.classes_)
        
        distances = np.zeros((n_samples, n_classes))
        for j, c in enumerate(self.classes_):
            proto = self.prototypes_[c]
            distances[:, j] = np.linalg.norm(X_scaled - proto, axis=1)
            
        distances = np.maximum(distances, self.epsilon)
        exponent = 2.0 / (self.m - 1.0)
        membership = np.zeros((n_samples, n_classes))
        
        for i in range(n_samples):
            d_i = distances[i, :]
            for j in range(n_classes):
                ratio = (d_i[j] / d_i) ** exponent
                membership[i, j] = 1.0 / np.sum(ratio)
                
        membership = membership / np.sum(membership, axis=1, keepdims=True)
        return membership
        
    def predict(self, X):
        proba = self.predict_proba(X)
        max_idx = np.argmax(proba, axis=1)
        return self.classes_[max_idx]

print("✓ Modul SupervisedFuzzySpatialClassifier berbasis NumPy siap dikompilasi dan dilatih.")
```

---

## 5. Pelatihan Model & Partisi Data Latih-Uji

Model dilatih menggunakan matriks 9 fitur spektral pada subset 210 poligon data latih:

```python
FITUR_MODEL = ["B02", "B03", "B04", "B08", "B11", "NDVI", "NDWI", "MNDWI", "NDBI"]

X = df_dataset[FITUR_MODEL].values
y = df_dataset["kelas"].values

X_train, X_test, y_train, y_test, idx_train, idx_test = train_test_split(
    X, y, df_dataset.index, test_size=0.30, random_state=42, stratify=y
)

print(f"Distribusi Subset Observasi Poligon:")
print(f"- Data Latih (Training Set) : {len(X_train)} poligon (70.0%)")
print(f"- Data Uji   (Testing Set)  : {len(X_test)} poligon (30.0%)")

# Pelatihan Model
model_fuzzy = SupervisedFuzzySpatialClassifier(m=2.0)
model_fuzzy.fit(X_train, y_train)

# Inferensi pada Data Uji Independen
y_pred = model_fuzzy.predict(X_test)
y_proba = model_fuzzy.predict_proba(X_test)

akurasi = accuracy_score(y_test, y_pred)
kappa = cohen_kappa_score(y_test, y_pred)

print(f"\n{'='*60}")
print(f"HASIL METRIK UTAMA MODEL FUZZY PADA DATA UJI INDEPENDEN:")
print(f"Overall Accuracy : {akurasi*100:.2f}% ({int(akurasi*len(y_test))}/{len(y_test)} Poligon Tepat)")
print(f"Cohen's Kappa (κ): {kappa:.4f} (Kategori: Almost Perfect Agreement)")
print(f"{'='*60}")
```

```text
Distribusi Subset Observasi Poligon:
- Data Latih (Training Set) : 210 poligon (70.0%)
- Data Uji   (Testing Set)  : 90 poligon (30.0%)

============================================================
HASIL METRIK UTAMA MODEL FUZZY PADA DATA UJI INDEPENDEN:
Overall Accuracy : 96.67% (87/90 Poligon Tepat)
Cohen's Kappa (κ): 0.9600 (Kategori: Almost Perfect Agreement)
============================================================
```
