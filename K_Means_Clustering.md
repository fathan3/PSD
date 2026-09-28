# K-Means Clustering Polutan

Dokumen ini menjelaskan proses pengelompokan (*clustering*) area berdasarkan tingkat dan karakteristik polutan (khususnya $\text{NO}_2$, $\text{SO}_2$, dan $\text{CO}$). Setelah melakukan ekstraksi fitur (menggunakan TSFEL sejumlah 68 fitur) pada data deret waktu polutan dari berbagai wilayah observasi dan menggabungkannya ke dalam satu matriks fitur, dilakukan reduksi dimensi serta pengelompokan pola.

Tujuan utama dari tahapan ini adalah untuk mengidentifikasi daerah-daerah mana saja yang memiliki karakteristik, tren fluktuasi, dan tingkat keparahan polusi udara yang serupa secara objektif.

---

## 1. Implementasi K-Means dengan Python (Scikit-Learn)

Untuk mengimplementasikan proses pengelompokan ini, kita menggunakan bahasa pemrograman Python memanfaatkan pustaka standar **Scikit-Learn** (`sklearn`). Pustaka ini menyediakan modul yang efisien dan stabil untuk pemrosesan *Machine Learning* seperti standarisasi data (`StandardScaler`), reduksi dimensi (`PCA`), serta algoritma pengelompokan (`KMeans`).

### 1.1 Evaluasi Jumlah Cluster (Elbow Method)

Guna menentukan jumlah kelompok (*cluster*) yang paling optimal, digunakan teknik **Elbow Method**. Model K-Means dilatih secara iteratif dengan memvariasikan jumlah kluster (misalnya dari $K=2$ hingga $K=10$), lalu menghitung nilai inersia (*Sum of Squared Distances* internal ke pusat kluster) pada setiap iterasi.

#### 1. NO2

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

# 1. Memuat data ekstraksi fitur NO2
df = pd.read_csv('ekstraksi_fitur_no2.csv')
fitur = df.drop(['id', 'nama', 'daerah'], axis=1)

# 2. Standarisasi dan Reduksi Dimensi (PCA 37 Komponen)
scaler = StandardScaler()
fitur_scaled = scaler.fit_transform(fitur)
fitur_pca = PCA(n_components=37).fit_transform(fitur_scaled)

# 3. Pengujian Inersia untuk K = 2 hingga 10
inertia = []
K_range = range(2, 11)
for k in K_range:
    kmeans_temp = KMeans(n_clusters=k, random_state=42, n_init=10)
    kmeans_temp.fit(fitur_pca)
    inertia.append(kmeans_temp.inertia_)

# 4. Visualisasi Grafik Elbow
plt.figure(figsize=(8, 5))
plt.plot(K_range, inertia, marker='o', linestyle='--')
plt.title('Evaluasi Jumlah Cluster dengan Elbow Method (NO2)')
plt.xlabel('Jumlah Cluster (K)')
plt.ylabel('Inersia (Jarak Kuadrat)')
plt.grid(True)
plt.show()
```

![Evaluasi Jumlah Cluster dengan Elbow Method (NO2)](clustering_polutan/elbow_no2.png)

#### 2. SO2

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

# 1. Memuat data ekstraksi fitur SO2
df = pd.read_csv('ekstraksi_fitur_so2.csv')
fitur = df.drop(['id', 'nama', 'daerah'], axis=1)

# 2. Standarisasi dan Reduksi Dimensi (PCA 37 Komponen)
scaler = StandardScaler()
fitur_scaled = scaler.fit_transform(fitur)
fitur_pca = PCA(n_components=37).fit_transform(fitur_scaled)

# 3. Pengujian Inersia untuk K = 2 hingga 10
inertia = []
K_range = range(2, 11)
for k in K_range:
    kmeans_temp = KMeans(n_clusters=k, random_state=42, n_init=10)
    kmeans_temp.fit(fitur_pca)
    inertia.append(kmeans_temp.inertia_)

# 4. Visualisasi Grafik Elbow
plt.figure(figsize=(8, 5))
plt.plot(K_range, inertia, marker='o', linestyle='--')
plt.title('Evaluasi Jumlah Cluster dengan Elbow Method (SO2)')
plt.xlabel('Jumlah Cluster (K)')
plt.ylabel('Inersia (Jarak Kuadrat)')
plt.grid(True)
plt.show()
```

![Evaluasi Jumlah Cluster dengan Elbow Method (SO2)](clustering_polutan/elbow_so2.png)

#### 3. CO

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

# 1. Memuat data ekstraksi fitur CO
df = pd.read_csv('ekstraksi_fitur_co.csv')
fitur = df.drop(['id', 'nama', 'daerah'], axis=1)

# 2. Standarisasi dan Reduksi Dimensi (PCA 37 Komponen)
scaler = StandardScaler()
fitur_scaled = scaler.fit_transform(fitur)
fitur_pca = PCA(n_components=37).fit_transform(fitur_scaled)

# 3. Pengujian Inersia untuk K = 2 hingga 10
inertia = []
K_range = range(2, 11)
for k in K_range:
    kmeans_temp = KMeans(n_clusters=k, random_state=42, n_init=10)
    kmeans_temp.fit(fitur_pca)
    inertia.append(kmeans_temp.inertia_)

# 4. Visualisasi Grafik Elbow
plt.figure(figsize=(8, 5))
plt.plot(K_range, inertia, marker='o', linestyle='--')
plt.title('Evaluasi Jumlah Cluster dengan Elbow Method (CO)')
plt.xlabel('Jumlah Cluster (K)')
plt.ylabel('Inersia (Jarak Kuadrat)')
plt.grid(True)
plt.show()
```

![Evaluasi Jumlah Cluster dengan Elbow Method (CO)](clustering_polutan/elbow_co.png)

> **Catatan Analisis Elbow**: Titik kelengkukan ("siku") pada kurva inersia mengindikasikan batas efisiensi penambahan jumlah kluster. Ketika landaian grafik mulai melambat secara signifikan, jumlah $K$ di titik pembelokan tersebut dipilih sebagai nilai kluster paling ideal.


---

### 1.2 Visualisasi Scatter Plot PCA dan Profiling

Setelah nilai $K$ yang optimal ditentukan, algoritma K-Means dijalankan untuk mengelompokkan sampel data ke dalam kluster final. Hasil pengelompokan kemudian dipetakan ke dalam bentuk **Scatter Plot** dan diprofilkan untuk melihat persebaran jumlah wilayah pada setiap kluster.

#### 1. NO2

```python
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

CSV_PATH = "ekstraksi_fitur_no2.csv"
KOLOM_NAMA = "nama"
KOLOM_FITUR = """abs_energy auc autocorr average_power calc_centroid calc_max calc_mean
calc_median calc_min calc_std calc_var dfa distance ecdf ecdf_percentile ecdf_percentile_count
ecdf_slope entropy fundamental_frequency higuchi_fractal_dimension hist_mode human_range_energy
hurst_exponent interq_range kurtosis lempel_ziv lpcc max_frequency max_power_spectrum
maximum_fractal_length mean_abs_deviation mean_abs_diff mean_diff median_abs_deviation
median_abs_diff median_diff median_frequency mfcc mse negative_turning neighbourhood_peaks
petrosian_fractal_dimension pk_pk_distance positive_turning power_bandwidth rms skewness slope
spectral_centroid spectral_decrease spectral_distance spectral_entropy spectral_kurtosis
spectral_positive_turning spectral_roll_off spectral_roll_on spectral_skewness spectral_slope
spectral_spread spectral_variation spectrogram_mean_coeff sum_abs_diff wavelet_abs_mean
wavelet_energy wavelet_entropy wavelet_std wavelet_var zero_cross""".split()
K = 5
N_DIMENSI = 37
GUNAKAN_SCALING = True

if N_DIMENSI > len(KOLOM_FITUR):
    raise ValueError(
        f"N_DIMENSI ({N_DIMENSI}) tidak boleh lebih besar dari jumlah fitur ({len(KOLOM_FITUR)})."
    )

# 1. Baca data
df = pd.read_csv(CSV_PATH)
data = df.dropna(subset=KOLOM_FITUR).copy()

# 2. Standarisasi Fitur
if GUNAKAN_SCALING:
    X = StandardScaler().fit_transform(data[KOLOM_FITUR])
else:
    X = data[KOLOM_FITUR].values

# 3. PCA
pca = PCA(n_components=N_DIMENSI, random_state=42)
X_pca = pca.fit_transform(X)

print(f"PCA: {len(KOLOM_FITUR)} fitur -> {N_DIMENSI} dimensi")

# 4. K-Means Clustering (K = 5)
kmeans = KMeans(n_clusters=K, random_state=42, n_init=10)
data["cluster"] = kmeans.fit_predict(X_pca)
data["cluster_label"] = "cluster_" + data["cluster"].astype(str)

# 5. Output Ringkasan Hasil
print(f"\nInertia: {kmeans.inertia_:.4f}")
print("\nJumlah data per cluster:")
print(data["cluster_label"].value_counts().sort_index())

# 6. Scatter plot persebaran wilayah
label_cluster = [f"cluster_{i}" for i in range(K)]
y = data["cluster"]
x = range(len(data))

plt.figure(figsize=(14, 5))
plt.scatter(x, y, s=25, color="#6b8bd6")
plt.xticks(x, data[KOLOM_NAMA], rotation=90, fontsize=7)
plt.yticks(range(K), label_cluster)
plt.ylim(-0.5, K - 0.5)
plt.title("Scatter Plot Persebaran Kluster (NO2)", loc="left", fontweight="bold")
plt.xlabel("nama")
plt.ylabel("Cluster")
plt.grid(True, color="#eeeeee")
plt.tight_layout()
plt.show()
```

```text
PCA: 68 fitur -> 37 dimensi

Inertia: 655.7898

Jumlah data per cluster:
cluster_label
cluster_0     2
cluster_1    22
cluster_2     1
cluster_3     1
cluster_4    11
Name: count, dtype: int64
```

![Scatter Plot Persebaran Kluster (NO2)](clustering_polutan/scatter_no2.png)

#### 2. SO2

```python
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

CSV_PATH = "ekstraksi_fitur_so2.csv"
KOLOM_NAMA = "nama"
KOLOM_FITUR = """abs_energy auc autocorr average_power calc_centroid calc_max calc_mean
calc_median calc_min calc_std calc_var dfa distance ecdf ecdf_percentile ecdf_percentile_count
ecdf_slope entropy fundamental_frequency higuchi_fractal_dimension hist_mode human_range_energy
hurst_exponent interq_range kurtosis lempel_ziv lpcc max_frequency max_power_spectrum
maximum_fractal_length mean_abs_deviation mean_abs_diff mean_diff median_abs_deviation
median_abs_diff median_diff median_frequency mfcc mse negative_turning neighbourhood_peaks
petrosian_fractal_dimension pk_pk_distance positive_turning power_bandwidth rms skewness slope
spectral_centroid spectral_decrease spectral_distance spectral_entropy spectral_kurtosis
spectral_positive_turning spectral_roll_off spectral_roll_on spectral_skewness spectral_slope
spectral_spread spectral_variation spectrogram_mean_coeff sum_abs_diff wavelet_abs_mean
wavelet_energy wavelet_entropy wavelet_std wavelet_var zero_cross""".split()
K = 5
N_DIMENSI = 37
GUNAKAN_SCALING = True

if N_DIMENSI > len(KOLOM_FITUR):
    raise ValueError(
        f"N_DIMENSI ({N_DIMENSI}) tidak boleh lebih besar dari jumlah fitur ({len(KOLOM_FITUR)})."
    )

# 1. Baca data
df = pd.read_csv(CSV_PATH)
data = df.dropna(subset=KOLOM_FITUR).copy()

# 2. Standarisasi Fitur
if GUNAKAN_SCALING:
    X = StandardScaler().fit_transform(data[KOLOM_FITUR])
else:
    X = data[KOLOM_FITUR].values

# 3. PCA
pca = PCA(n_components=N_DIMENSI, random_state=42)
X_pca = pca.fit_transform(X)

print(f"PCA: {len(KOLOM_FITUR)} fitur -> {N_DIMENSI} dimensi")

# 4. K-Means Clustering (K = 5)
kmeans = KMeans(n_clusters=K, random_state=42, n_init=10)
data["cluster"] = kmeans.fit_predict(X_pca)
data["cluster_label"] = "cluster_" + data["cluster"].astype(str)

# 5. Output Ringkasan Hasil
print(f"\nInertia: {kmeans.inertia_:.4f}")
print("\nJumlah data per cluster:")
print(data["cluster_label"].value_counts().sort_index())

# 6. Scatter plot persebaran wilayah
label_cluster = [f"cluster_{i}" for i in range(K)]
y = data["cluster"]
x = range(len(data))

plt.figure(figsize=(14, 5))
plt.scatter(x, y, s=25, color="#6b8bd6")
plt.xticks(x, data[KOLOM_NAMA], rotation=90, fontsize=7)
plt.yticks(range(K), label_cluster)
plt.ylim(-0.5, K - 0.5)
plt.title("Scatter Plot Persebaran Kluster (SO2)", loc="left", fontweight="bold")
plt.xlabel("nama")
plt.ylabel("Cluster")
plt.grid(True, color="#eeeeee")
plt.tight_layout()
plt.show()
```

```text
PCA: 68 fitur -> 37 dimensi

Inertia: 413.6787

Jumlah data per cluster:
cluster_label
cluster_0     3
cluster_1    31
cluster_2     1
cluster_3     1
cluster_4     1
Name: count, dtype: int64
```

![Scatter Plot Persebaran Kluster (SO2)](clustering_polutan/scatter_so2.png)

#### 3. CO

```python
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

CSV_PATH = "ekstraksi_fitur_co.csv"
KOLOM_NAMA = "nama"
KOLOM_FITUR = """abs_energy auc autocorr average_power calc_centroid calc_max calc_mean
calc_median calc_min calc_std calc_var dfa distance ecdf ecdf_percentile ecdf_percentile_count
ecdf_slope entropy fundamental_frequency higuchi_fractal_dimension hist_mode human_range_energy
hurst_exponent interq_range kurtosis lempel_ziv lpcc max_frequency max_power_spectrum
maximum_fractal_length mean_abs_deviation mean_abs_diff mean_diff median_abs_deviation
median_abs_diff median_diff median_frequency mfcc mse negative_turning neighbourhood_peaks
petrosian_fractal_dimension pk_pk_distance positive_turning power_bandwidth rms skewness slope
spectral_centroid spectral_decrease spectral_distance spectral_entropy spectral_kurtosis
spectral_positive_turning spectral_roll_off spectral_roll_on spectral_skewness spectral_slope
spectral_spread spectral_variation spectrogram_mean_coeff sum_abs_diff wavelet_abs_mean
wavelet_energy wavelet_entropy wavelet_std wavelet_var zero_cross""".split()
K = 5
N_DIMENSI = 37
GUNAKAN_SCALING = True

if N_DIMENSI > len(KOLOM_FITUR):
    raise ValueError(
        f"N_DIMENSI ({N_DIMENSI}) tidak boleh lebih besar dari jumlah fitur ({len(KOLOM_FITUR)})."
    )

# 1. Baca data
df = pd.read_csv(CSV_PATH)
data = df.dropna(subset=KOLOM_FITUR).copy()

# 2. Standarisasi Fitur
if GUNAKAN_SCALING:
    X = StandardScaler().fit_transform(data[KOLOM_FITUR])
else:
    X = data[KOLOM_FITUR].values

# 3. PCA
pca = PCA(n_components=N_DIMENSI, random_state=42)
X_pca = pca.fit_transform(X)

print(f"PCA: {len(KOLOM_FITUR)} fitur -> {N_DIMENSI} dimensi")

# 4. K-Means Clustering (K = 5)
kmeans = KMeans(n_clusters=K, random_state=42, n_init=10)
data["cluster"] = kmeans.fit_predict(X_pca)
data["cluster_label"] = "cluster_" + data["cluster"].astype(str)

# 5. Output Ringkasan Hasil
print(f"\nInertia: {kmeans.inertia_:.4f}")
print("\nJumlah data per cluster:")
print(data["cluster_label"].value_counts().sort_index())

# 6. Scatter plot persebaran wilayah
label_cluster = [f"cluster_{i}" for i in range(K)]
y = data["cluster"]
x = range(len(data))

plt.figure(figsize=(14, 5))
plt.scatter(x, y, s=25, color="#6b8bd6")
plt.xticks(x, data[KOLOM_NAMA], rotation=90, fontsize=7)
plt.yticks(range(K), label_cluster)
plt.ylim(-0.5, K - 0.5)
plt.title("Scatter Plot Persebaran Kluster (CO)", loc="left", fontweight="bold")
plt.xlabel("nama")
plt.ylabel("Cluster")
plt.grid(True, color="#eeeeee")
plt.tight_layout()
plt.show()
```

```text
PCA: 68 fitur -> 37 dimensi

Inertia: 413.8895

Jumlah data per cluster:
cluster_label
cluster_0    12
cluster_1     1
cluster_2     2
cluster_3     1
cluster_4    21
Name: count, dtype: int64
```

![Scatter Plot Persebaran Kluster (CO)](clustering_polutan/scatter_co.png)


---

## 2. Alur Kerja (Workflow) Clustering KNIME

Selain eksekusi berbasis skrip Python, pengolahan *clustering* juga dirancang secara visual menggunakan perangkat lunak **KNIME Analytics Platform** untuk memastikan transparansi alur kerja pemrosesan data.

![Visualisasi Alur Kerja (Workflow) K-Means pada KNIME](clustering_polutan/alur.png)

Berdasarkan struktur diagram *workflow* KNIME di atas, pemrosesan data dilakukan melalui 3 jalur paralel (masing-masing untuk polutan $\text{NO}_2$, $\text{CO}$, dan $\text{SO}_2$) dari satu titik sumber basis data utama:

1. **MySQL Connector $\rightarrow$ DB Table Selector $\rightarrow$ DB Reader**:
   Konektor utama menggunakan node **MySQL Connector** untuk menghubungkan KNIME ke basis data MySQL server. Alur kemudian terbagi menjadi 3 cabang paralel:
   - Cabang 1: **NO2 DB Table Selector** $\rightarrow$ **DB Reader** untuk membaca data fitur $\text{NO}_2$.
   - Cabang 2: **CO DB Table Selector** $\rightarrow$ **DB Reader** untuk membaca data fitur $\text{CO}$.
   - Cabang 3: **SO2 DB Table Selector** $\rightarrow$ **DB Reader** untuk membaca data fitur $\text{SO}_2$.

2. **PCA (Principal Component Analysis)**:
   Masing-masing cabang mengalirkan data ke node **PCA** untuk mereduksi dimensi matriks fitur deret waktu TSFEL yang kompleks menjadi komponen utama (*Principal Components*) tanpa mengabaikan informasi varians penting.

3. **k-Means**:
   Node **k-Means** pada setiap jalur mengeksekusi algoritma *Unsupervised Learning* untuk mengelompokkan wilayah-wilayah berdasarkan kedekatan jarak matematis fitur PCA yang dihasilkan.

4. **Scatter Plot**:
   Node **Scatter Plot** di ujung setiap rantai berfungsi memvisualisasikan titik persebaran sampel data terhadap kelompok (*cluster*) yang terbentuk secara interaktif.

---

## 3. Interpretasi Hasil Clustering (Scatter Plot KNIME)

Berikut adalah visualisasi *Scatter Plot* hasil *Clustering K-Means* dari platform KNIME untuk masing-masing polutan ($\text{NO}_2$, $\text{CO}$, dan $\text{SO}_2$) beserta analisis deskriptif yang disesuaikan secara presisi dengan tampilan grafik fisik.

### 3.1 Polutan NO2 (K = 3)

![Visualisasi Scatter Plot Hasil Clustering K-Means (NO2)](clustering_polutan/plot_no2.png)

Berdasarkan visualisasi grafik *Scatter Plot* KNIME untuk polutan **$\text{NO}_2$**, algoritma K-Means membagi titik-titik data ke dalam **3 kelompok utama (`cluster_0`, `cluster_1`, dan `cluster_2`)**:

- **Dominasi Kelompok Utama (`cluster_0`)**: Sebagian besar sampel wilayah berkumpul secara rapat pada `cluster_0`. Hal ini menunjukkan adanya pola konsentrasi emisi $\text{NO}_2$ baseline/standar yang dialami oleh mayoritas kawasan observasi.
- **Kelompok Fluktuasi Menengah (`cluster_1`)**: Terdapat sekelompok sampel wilayah yang terpisah ke dalam `cluster_1`, mencerminkan karakteristik dinamika $\text{NO}_2$ dengan tingkat fluktuasi menengah.
- **Anomali / Outlier Terisolasi (`cluster_2`)**: `cluster_2` hanya diisi oleh 1 titik sampel spesifik yang berada terisolasi di bagian bawah grafik. Hal ini menandakan adanya perbedaan nilai fitur polutan $\text{NO}_2$ yang sangat ekstrem dan signifikan dibandingkan sampel wilayah lainnya.

---

### 3.2 Polutan CO (K = 3)

![Visualisasi Scatter Plot Hasil Clustering K-Means (CO)](clustering_polutan/plot_co.png)

Untuk polutan **$\text{CO}$**, pembagian kluster pada hasil pemetaan KNIME juga terbagi ke dalam **3 kelompok (`cluster_0`, `cluster_1`, dan `cluster_2`)**:

- **Dominasi Kelompok Atas (`cluster_2`)**: `cluster_2` menjadi kelompok yang paling mendominasi, menampung hampir seluruh titik data sampel wilayah di bagian atas grafik. Ini mengindikasikan homogenitas tren emisi Karbon Monoksida pada mayoritas titik pengamatan.
- **Outlier Terisolasi Pertama (`cluster_0`)**: Terlihat 1 titik data di bagian tengah sebelah kanan yang terpisah masuk ke dalam `cluster_0`.
- **Outlier Terisolasi Kedua (`cluster_1`)**: Terlihat 1 titik data di bagian kanan bawah yang menempati `cluster_1` secara tersendiri, menunjukkan sifat anomali emisi Karbon Monoksida yang khas pada titik pengamatan tersebut.

---

### 3.3 Polutan SO2 (K = 3)

![Visualisasi Scatter Plot Hasil Clustering K-Means (SO2)](clustering_polutan/plot_so2.png)

Hasil visualisasi grafik *Scatter Plot* untuk polutan **$\text{SO}_2$** memperlihatkan struktur pengelompokan ke dalam **3 kelompok (`cluster_0`, `cluster_1`, dan `cluster_2`)**:

- **Dominasi Kelompok Mayoritas (`cluster_2`)**: Sama seperti pada pola emisi $\text{CO}$, `cluster_2` berada pada bagian atas grafik dan menampung hampir seluruh sampel wilayah observasi, mencerminkan kondisi rata-rata paparan $\text{SO}_2$ yang umum dialami kawasan.
- **Anomali Terpisah Pertama (`cluster_0`)**: Satu sampel data terlepas dari gerombolan utama dan masuk ke `cluster_0` di bagian tengah grafik.
- **Anomali Terpisah Kedua (`cluster_1`)**: Satu sampel data terpisah di bagian bawah grafik dan menempati `cluster_1`, mengindikasikan adanya pencilan konsentrasi Sulfur Dioksida yang tajam pada lokasi observasi spesifik tersebut.


