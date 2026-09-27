# Preprocessing dan Ekstraksi Fitur

## Preprocessing: Penanganan Outliers dan Interpolasi

Setelah melalui tahap pemahaman data awal, kita menemukan sejumlah kendala berupa hilangnya rekaman data pada tanggal tertentu (*missing values*) serta kemunculan nilai-nilai anomali (*outliers*). Sebagai upaya normalisasi agar dataset ini siap digunakan untuk ekstraksi fitur yang solid, kita melakukan prosedur pembersihan berlapis. Pendekatan yang dipakai adalah pemfilteran statistik menggunakan batasan *Interquartile Range* (IQR) yang diikuti dengan algoritma **capping iteratif** dan **interpolasi linier** untuk menambal data yang kosong serta memastikan dataset bersih sempurna dari pencilan ekstrem.

### 1. Deteksi dan Visualisasi Outlier (Metode IQR)

Teknik rentang antar-kuartil (IQR) diaplikasikan untuk menyaring nilai ekstrem pada masing-masing parameter polutan ($\text{NO}_2$, $\text{SO}_2$, dan $\text{CO}$). Nilai di bawah batas bawah (*lower bound*) atau melampaui batas atas (*upper bound*) ditandai sebagai anomali atau *outlier*.

$$ \text{IQR} = Q_3 - Q_1 $$
$$ \text{Lower Bound} = Q_1 - 1.5 \times \text{IQR} $$
$$ \text{Upper Bound} = Q_3 + 1.5 \times \text{IQR} $$

#### A. Deteksi Outlier NO2

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("NO2_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

# Hitung IQR
Q1 = df['NO2'].quantile(0.25)
Q3 = df['NO2'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter outlier
outliers_iqr = df[(df['NO2'] < lower_bound) | (df['NO2'] > upper_bound)]

print("Jumlah Outlier (IQR):", len(outliers_iqr))
print(outliers_iqr[['date', 'NO2']].head())
```

```text
Jumlah Outlier (IQR): 7
         date           NO2
25 2025-09-24 -6.785994e-06
64 2025-11-02 -2.512900e-07
65 2025-11-03 -7.859995e-07
68 2025-11-06 -2.390128e-06
69 2025-11-07 -2.924838e-06
```

```python
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['NO2'], label="NO2", linewidth=1)

plt.scatter(outliers_iqr['date'], outliers_iqr['NO2'],
            color='red', marker='o', label="Outliers")

plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue',   linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data NO2 (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar NO2")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

![png](ekstraksi-files/ekstraksi1.png)

#### B. Deteksi Outlier SO2

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("SO2_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

# Hitung IQR
Q1 = df['SO2'].quantile(0.25)
Q3 = df['SO2'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter outlier
outliers_iqr = df[(df['SO2'] < lower_bound) | (df['SO2'] > upper_bound)]

print("Jumlah Outlier (IQR):", len(outliers_iqr))
print(outliers_iqr[['date', 'SO2']].head())
```

```text
Jumlah Outlier (IQR): 6
          date       SO2
19  2025-09-15  0.000540
40  2025-10-06 -0.000510
165 2026-02-08  0.000546
259 2026-05-13 -0.000675
318 2026-07-11 -0.000506
```

```python
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['SO2'], label="SO2", linewidth=1)

plt.scatter(outliers_iqr['date'], outliers_iqr['SO2'],
            color='red', marker='o', label="Outliers")

plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue',   linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data SO2 (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar SO2")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

#### C. Deteksi Outlier CO

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("CO_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

# Hitung IQR
Q1 = df['CO'].quantile(0.25)
Q3 = df['CO'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter outlier
outliers_iqr = df[(df['CO'] < lower_bound) | (df['CO'] > upper_bound)]

print("Jumlah Outlier (IQR):", len(outliers_iqr))
print(outliers_iqr[['date', 'CO']].head())
```

```text
Jumlah Outlier (IQR): 5
          date        CO
43  2025-10-09  0.040473
320 2026-07-13  0.040105
347 2026-08-09  0.039287
348 2026-08-10  0.038860
357 2026-08-19  0.038800
```

```python
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['CO'], label="CO", linewidth=1)

plt.scatter(outliers_iqr['date'], outliers_iqr['CO'],
            color='red', marker='o', label="Outliers")

plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue',   linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data CO (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar CO")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

---

### 2. Penanganan Outlier dan Interpolasi Data

Setelah posisi *outlier* terdeteksi, nilai-nilai ekstrem diubah menjadi `NaN`. Untuk menjaga kesinambungan urutan waktu, kekosongan data ditambal menggunakan **interpolasi linier**, dilanjutkan dengan *backward fill* (`bfill`) dan *forward fill* (`ffill`). Selanjutnya, digunakan **Algoritma Capping Iteratif** dengan pengaman presisi desimal ($10^{-12}$) untuk memastikan kuantitas outlier akhir mutlak 0.

#### A. Penanganan Outlier & Interpolasi NO2

```python
import pandas as pd
import numpy as np

# 1. Ambil data asli
df = pd.read_csv("NO2_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

# 2. Hitung IQR awal
Q1_init = df['NO2'].quantile(0.25)
Q3_init = df['NO2'].quantile(0.75)
IQR_init = Q3_init - Q1_init
lower_init = Q1_init - 1.5 * IQR_init
upper_init = Q3_init + 1.5 * IQR_init

# 3. Ubah outlier awal menjadi NaN
df['NO2_cleaned'] = df['NO2'].mask((df['NO2'] < lower_init) | (df['NO2'] > upper_init))

# 4. Interpolasi linear
df['NO2_filled'] = df['NO2_cleaned'].interpolate(method='linear')
df['NO2_filled'] = df['NO2_filled'].bfill().ffill()

# 5. Algoritma Iteratif untuk Menghilangkan Outlier Baru dengan Pengaman Presisi Desimal
data_final = df['NO2_filled'].copy()
while True:
    Q1_curr = data_final.quantile(0.25)
    Q3_curr = data_final.quantile(0.75)
    IQR_curr = Q3_curr - Q1_curr
    lower_curr = Q1_curr - 1.5 * IQR_curr
    upper_curr = Q3_curr + 1.5 * IQR_curr

    # Deteksi outlier
    outliers = data_final[(data_final < lower_curr) | (data_final > upper_curr)]
    if len(outliers) == 0:
        break

    # Pangkas dengan margin pengaman kecil (1e-12)
    safe_lower = lower_curr + 1e-12
    safe_upper = upper_curr - 1e-12
    data_final = data_final.clip(lower=safe_lower, upper=safe_upper)

# 6. Simpan dataset final
df_no2_warudoyong = pd.DataFrame({
    "date": df['date'],
    "NO2": data_final
})
df_no2_warudoyong.to_csv("NO2_Warudoyong_filled.csv", index=False)
print("Pemrosesan selesai! Silakan jalankan sel pengecekan di bawahnya.")
```

**Verifikasi Akhir Outlier NO2:**

```python
import pandas as pd
import numpy as np

df = pd.read_csv("NO2_Warudoyong_filled.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

Q1 = df['NO2'].quantile(0.25)
Q3 = df['NO2'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

outliers_iqr = df[(df['NO2'] < lower_bound) | (df['NO2'] > upper_bound)]

print("Jumlah Outlier (IQR) Akhir setelah Capping Iteratif:", len(outliers_iqr))
print("Batas bawah:", lower_bound)
print("Batas atas:", upper_bound)
```

```text
Jumlah Outlier (IQR) Akhir setelah Capping Iteratif: 0
Batas bawah: 2.6228252636428772e-06
Batas atas: 6.936550308485798e-05
```

```python
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['NO2'], label="NO2", linewidth=1)

plt.scatter(outliers_iqr['date'], outliers_iqr['NO2'],
            color='red', marker='o', label="Outliers")

plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue',   linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data NO2 Setelah Bersih (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar NO2")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

![png](ekstraksi-files/ekstraksi2.png)

#### B. Penanganan Outlier & Interpolasi SO2

```python
import pandas as pd
import numpy as np

# 1. Ambil data asli
df = pd.read_csv("SO2_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

# 2. Hitung IQR awal
Q1_init = df['SO2'].quantile(0.25)
Q3_init = df['SO2'].quantile(0.75)
IQR_init = Q3_init - Q1_init
lower_init = Q1_init - 1.5 * IQR_init
upper_init = Q3_init + 1.5 * IQR_init

# 3. Ubah outlier awal menjadi NaN
df['SO2_cleaned'] = df['SO2'].mask((df['SO2'] < lower_init) | (df['SO2'] > upper_init))

# 4. Interpolasi linear
df['SO2_filled'] = df['SO2_cleaned'].interpolate(method='linear')
df['SO2_filled'] = df['SO2_filled'].bfill().ffill()

# 5. Algoritma Iteratif
data_final = df['SO2_filled'].copy()
while True:
    Q1_curr = data_final.quantile(0.25)
    Q3_curr = data_final.quantile(0.75)
    IQR_curr = Q3_curr - Q1_curr
    lower_curr = Q1_curr - 1.5 * IQR_curr
    upper_curr = Q3_curr + 1.5 * IQR_curr

    outliers = data_final[(data_final < lower_curr) | (data_final > upper_curr)]
    if len(outliers) == 0:
        break

    safe_lower = lower_curr + 1e-12
    safe_upper = upper_curr - 1e-12
    data_final = data_final.clip(lower=safe_lower, upper=safe_upper)

# 6. Simpan dataset final
df_SO2_warudoyong = pd.DataFrame({
    "date": df['date'],
    "SO2": data_final
})
df_SO2_warudoyong.to_csv("SO2_Warudoyong_filled.csv", index=False)
print("Pemrosesan selesai!")
```

**Verifikasi Akhir Outlier SO2:**

```python
import pandas as pd
import numpy as np

df = pd.read_csv("SO2_Warudoyong_filled.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

Q1 = df['SO2'].quantile(0.25)
Q3 = df['SO2'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

outliers_iqr = df[(df['SO2'] < lower_bound) | (df['SO2'] > upper_bound)]

print("Jumlah Outlier (IQR) Akhir setelah Capping Iteratif:", len(outliers_iqr))
print("Batas bawah:", lower_bound)
print("Batas atas:", upper_bound)
```

```text
Jumlah Outlier (IQR) Akhir setelah Capping Iteratif: 0
Batas bawah: -0.00046624713750967496
Batas atas: 0.0005028181208357249
```

#### C. Penanganan Outlier & Interpolasi CO

```python
import pandas as pd
import numpy as np

# 1. Ambil data asli
df = pd.read_csv("CO_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

# 2. Hitung IQR awal
Q1_init = df['CO'].quantile(0.25)
Q3_init = df['CO'].quantile(0.75)
IQR_init = Q3_init - Q1_init
lower_init = Q1_init - 1.5 * IQR_init
upper_init = Q3_init + 1.5 * IQR_init

# 3. Ubah outlier awal menjadi NaN
df['CO_cleaned'] = df['CO'].mask((df['CO'] < lower_init) | (df['CO'] > upper_init))

# 4. Interpolasi linear
df['CO_filled'] = df['CO_cleaned'].interpolate(method='linear')
df['CO_filled'] = df['CO_filled'].bfill().ffill()

# 5. Algoritma Iteratif
data_final = df['CO_filled'].copy()
while True:
    Q1_curr = data_final.quantile(0.25)
    Q3_curr = data_final.quantile(0.75)
    IQR_curr = Q3_curr - Q1_curr
    lower_curr = Q1_curr - 1.5 * IQR_curr
    upper_curr = Q3_curr + 1.5 * IQR_curr

    outliers = data_final[(data_final < lower_curr) | (data_final > upper_curr)]
    if len(outliers) == 0:
        break

    safe_lower = lower_curr + 1e-12
    safe_upper = upper_curr - 1e-12
    data_final = data_final.clip(lower=safe_lower, upper=safe_upper)

# 6. Simpan dataset final
df_CO_warudoyong = pd.DataFrame({
    "date": df['date'],
    "CO": data_final
})
df_CO_warudoyong.to_csv("CO_Warudoyong_filled.csv", index=False)
print("Pemrosesan selesai!")
```

**Verifikasi Akhir Outlier CO:**

```python
import pandas as pd
import numpy as np

df = pd.read_csv("CO_Warudoyong_filled.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

Q1 = df['CO'].quantile(0.25)
Q3 = df['CO'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

outliers_iqr = df[(df['CO'] < lower_bound) | (df['CO'] > upper_bound)]

print("Jumlah Outlier (IQR) Akhir setelah Capping Iteratif:", len(outliers_iqr))
print("Batas bawah:", lower_bound)
print("Batas atas:", upper_bound)
```

```text
Jumlah Outlier (IQR) Akhir setelah Capping Iteratif: 0
Batas bawah: 0.017772719236160533
Batas atas: 0.03875323956637375
```

---

## Ekstraksi Fitur Deret Waktu (Time Series)

Kini kita memiliki set data deret waktu polutan udara yang terbebas dari *missing values* dan *outliers*. Tahap krusial berikutnya adalah melakukan rekayasa ekstraksi fitur (*feature engineering*). Proses ini ditujukan untuk membedah karakteristik tersembunyi dari fluktuasi harian polutan menjadi ragam variabel prediktor baru yang nantinya siap ditelan oleh model algoritma *Machine Learning* / *Deep Learning*.

Untuk mempercepat kalkulasi ekstraksi massal ini, kita mengeksploitasi kapabilitas *library* Python bernama **`tsfel`** (*Time Series Feature Extraction Library*).

Di bawah ini adalah prosedur skrip untuk mengekstraksi 68 jenis fitur secara komprehensif pada variabel $\text{NO}_2$ (metode ini diduplikasi untuk gas $\text{CO}$, $\text{SO}_2$, maupun $\text{O}_3$):

```python
import pandas as pd
import numpy as np
import inspect
import tsfel.feature_extraction.features as tsfel_features

# ---------- 1. Muat data yang sudah dibersihkan ----------
df = pd.read_csv('NO2_Warudoyong_filled.csv')
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

target_pollutant = 'NO2'

# Pastikan data di-casting ke tipe numerik.
df[target_pollutant] = pd.to_numeric(df[target_pollutant], errors='coerce')

# Interpolasi terakhir untuk berjaga-jaga apabila terdapat sisa format nan
df_clean = df.set_index('date').interpolate(method='time').ffill().bfill()
fs = 1
signal_1d = df_clean[target_pollutant].astype(float).values

# ---------- 2. Inisiasi 68 Daftar Fitur TSFEL ----------
FEATURE_LIST = """abs_energy auc autocorr average_power calc_centroid calc_max calc_mean
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

print("Jumlah fitur yang diminta:", len(FEATURE_LIST))

# ---------- 3. Fungsi ekstraksi dan penyeragaman output  ----------

def to_scalar(result):
    if isinstance(result, dict) and "values" in result:
        result = result["values"]
    if isinstance(result, (list, tuple, np.ndarray)):
        arr = np.asarray(result, dtype=float)
        return float(np.nanmean(arr))
    return float(result)

def extract_one(fn_name, signal, fs):
    fn = getattr(tsfel_features, fn_name)
    params = inspect.signature(fn).parameters
    if "fs" in params:
        result = fn(signal, fs)
    else:
        result = fn(signal)
    return to_scalar(result)

row = {}
for fn_name in FEATURE_LIST:
    row[fn_name] = extract_one(fn_name, signal_1d, fs)

extracted_features_final = pd.DataFrame([row])

print(f"Berhasil! Jumlah fitur yang diekstrak pada {target_pollutant}: {extracted_features_final.shape[1]}")

# Export hasil ke file CSV
extracted_features_final.to_csv(f'{target_pollutant}_Warudoyong_TSFEL.csv', index=False)
```

```text
Jumlah fitur yang diminta: 68
Berhasil! Jumlah fitur yang diekstrak pada NO2: 68
```

Preview ringkas matriks 68 fitur TSFEL yang dihasilkan:

```python
import pandas as pd
pd.set_option('display.max_columns', None)
df = pd.read_csv("NO2_Warudoyong_TSFEL.csv")
df.head(10)
```

---

## Penjelasan dan Formula Matematis 68 Fitur TSFEL

Pustaka TSFEL memecah struktur 68 fitur deret waktu ke dalam tiga domain utama: **Statistical (Statistik)**, **Temporal (Waktu)**, dan **Spectral (Frekuensi & Wavelet)**. Di bawah ini adalah penjelasan komprehensif beserta contoh formula matematis untuk masing-masing dari 68 fitur tersebut:

---

### I. Domain Statistical (17 Fitur)

Domain statistik mengukur karakteristik distribusi nilai sinyal $x = [x_1, x_2, \dots, x_N]$ tanpa mempedulikan urutan waktu kronologis.

1. **`calc_mean` (Rata-rata)**
   Rata-rata aritmatika dari seluruh nilai observasi.
   $$ \mu = \frac{1}{N} \sum_{i=1}^{N} x_i $$

2. **`calc_std` (Standar Deviasi)**
   Mengukur tingkat sebaran atau dispersi data di sekitar nilai rata-rata.
   $$ \sigma = \sqrt{\frac{1}{N-1} \sum_{i=1}^{N} (x_i - \mu)^2} $$

3. **`calc_var` (Varians)**
   Kuadrat dari standar deviasi, mengukur volatilitas sinyal.
   $$ \sigma^2 = \frac{1}{N-1} \sum_{i=1}^{N} (x_i - \mu)^2 $$

4. **`calc_min` (Nilai Minimum)**
   Nilai observasi terendah dalam deret waktu.
   $$ x_{\min} = \min_{1 \le i \le N} x_i $$

5. **`calc_max` (Nilai Maksimum)**
   Nilai observasi tertinggi dalam deret waktu.
   $$ x_{\max} = \max_{1 \le i \le N} x_i $$

6. **`calc_median` (Median)**
   Nilai tengah dari data yang diurutkan.
   $$ \text{Median}(x) = x_{\left(\frac{N+1}{2}\right)} \quad (\text{untuk } N \text{ ganjil}) $$

7. **`skewness` (Kemiringan)**
   Mengukur derajat ketidaksimetrisan distribusi data terhadap rata-rata.
   $$ S = \frac{\frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^3}{\sigma^3} $$

8. **`kurtosis` (Kurtosis)**
   Mengukur keruncingan puncak kurva distribusi data dan ketebalan ekor (*outliers*).
   $$ K = \frac{\frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^4}{\sigma^4} - 3 $$

9. **`interq_range` (Interquartile Range / IQR)**
   Lebar jangkauan antara kuartil ketiga ($Q_3$) dan kuartil pertama ($Q_1$).
   $$ \text{IQR} = Q_3 - Q_1 $$

10. **`mean_abs_deviation` (Mean Absolute Deviation)**
    Rata-rata deviasi absolut tiap sampel terhadap rata-rata.
    $$ \text{MAD}_{\text{mean}} = \frac{1}{N} \sum_{i=1}^{N} |x_i - \mu| $$

11. **`median_abs_deviation` (Median Absolute Deviation)**
    Median dari simpangan absolut sampel terhadap median data.
    $$ \text{MAD}_{\text{median}} = \text{Median}\Big(|x_i - \text{Median}(x)|\Big) $$

12. **`rms` (Root Mean Square)**
    Akar dari rata-rata kuadrat nilai sinyal, mewakili energi rata-rata.
    $$ \text{RMS} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} x_i^2} $$

13. **`hist_mode` (Modus Histogram)**
    Nilai tengah interval histogram yang memiliki frekuensi kemunculan terbanyak.
    $$ \text{Mode} = \arg\max_{b_k} \text{Hist}(b_k) $$

14. **`ecdf` (Empirical Cumulative Distribution Function)**
    Fungsi distribusi kumulatif empiris dari sinyal pada titik $t$.
    $$ F_N(t) = \frac{1}{N} \sum_{i=1}^{N} \mathbb{I}(x_i \le t) $$

15. **`ecdf_percentile` (Persentil ECDF)**
    Nilai ambang batas sampel pada persentil probabilitas kumulatif $p$.
    $$ P_p = F_N^{-1}(p) \quad (p \in [0, 1]) $$

16. **`ecdf_percentile_count` (Jumlah Persentil ECDF)**
    Jumlah sampel yang berada di dalam rentang persentil tertentu.
    $$ N_p = \sum_{i=1}^{N} \mathbb{I}(q_{p1} \le x_i \le q_{p2}) $$

17. **`ecdf_slope` (Kemiringan ECDF)**
    Laju perubahan atau gradien kurva ECDF antar dua titik persentil.
    $$ \text{Slope}_{\text{ECDF}} = \frac{F_N(b) - F_N(a)}{b - a} $$

---

### II. Domain Temporal (25 Fitur)

Domain temporal menganalisis dinamika perubahan sinyal berdasarkan dimensi kronologi waktu.

18. **`abs_energy` (Energi Absolut Total)**
    Jumlah total daya kuadrat dari seluruh sampel sinyal.
    $$ E = \sum_{i=1}^{N} x_i^2 $$

19. **`auc` (Area Under the Curve)**
    Luas daerah di bawah kurva sinyal menggunakan aturan trapesium.
    $$ \text{AUC} = \sum_{i=1}^{N-1} \frac{x_i + x_{i+1}}{2} \Delta t $$

20. **`autocorr` (Autokorelasi)**
    Korelasi sinyal dengan dirinya sendiri pada waktu terpisah (*lag* $k$).
    $$ R(k) = \sum_{i=1}^{N-k} x_i x_{i+k} $$

21. **`average_power` (Daya Rata-Rata)**
    Besar energi rata-rata yang ditransmisikan per satuan waktu.
    $$ P_{\text{avg}} = \frac{1}{N} \sum_{i=1}^{N} x_i^2 $$

22. **`calc_centroid` (Pusat Gravitasi Waktu)**
    Titik pusat berat distribusi sinyal sepanjang garis waktu.
    $$ C_{\text{waktu}} = \frac{\sum_{i=1}^{N} i \cdot |x_i|}{\sum_{i=1}^{N} |x_i|} $$

23. **`dfa` (Detrended Fluctuation Analysis)**
    Mengukur sifat korelasi jangka panjang (derajat dependensi fraktal $\alpha$).
    $$ F(n) = \sqrt{\frac{1}{N} \sum_{k=1}^{N} \big(y(k) - y_n(k)\big)^2} \propto n^\alpha $$

24. **`distance` (Panjang Lintasan Sinyal)**
    Total panjang jarak geometris perjalanan sinyal.
    $$ D = \sum_{i=1}^{N-1} \sqrt{1 + (x_{i+1} - x_i)^2} $$

25. **`entropy` (Entropi Shannon)**
    Tingkat acak atau ketidakpastian sinyal.
    $$ H = -\sum_{k} p_k \log_2(p_k) $$

26. **`higuchi_fractal_dimension` (Dimensi Fraktal Higuchi)**
    Mengukur kompleksitas bentuk fraktal sinyal ($D_H$).
    $$ L(m, k) \propto k^{-D_H} $$

27. **`hurst_exponent` (Eksponen Hurst)**
    Menentukan tingkat persistensi tren deret waktu ($H$).
    $$ \frac{R(n)}{S(n)} \propto n^H $$

28. **`lempel_ziv` (Kompleksitas Lempel-Ziv)**
    Mengevaluasi laju keanekaragaman pola sekuensial sinyal biner.
    $$ C_{LZ} = \frac{C(N)}{\frac{N}{\log_2 N}} $$

29. **`maximum_fractal_length` (Panjang Fraktal Maksimum)**
    Panjang kurva fraktal terbesar pada interval skala tertentu.
    $$ L_{\max} = \max_{k} L(k) $$

30. **`mean_abs_diff` (Mean Absolute Difference)**
    Rata-rata dari selisih absolut antar sampel berurutan.
    $$ \text{MAD}_{\text{diff}} = \frac{1}{N-1} \sum_{i=1}^{N-1} |x_{i+1} - x_i| $$

31. **`mean_diff` (Mean Difference)**
    Rata-rata perubahan selisih langsung antar sampel.
    $$ \text{Mean}_{\text{diff}} = \frac{x_N - x_1}{N-1} $$

32. **`median_abs_diff` (Median Absolute Difference)**
    Nilai tengah dari selisih absolut antar sampel berurutan.
    $$ \text{Median}_{\text{abs\_diff}} = \text{Median}\big(|x_{i+1} - x_i|\big) $$

33. **`median_diff` (Median Difference)**
    Nilai tengah dari selisih perubahan sampel berurutan.
    $$ \text{Median}_{\text{diff}} = \text{Median}(x_{i+1} - x_i) $$

34. **`mse` (Mean Squared Error)**
    Rata-rata kuadrat kesalahan sinyal terhadap rata-rata.
    $$ \text{MSE} = \frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^2 $$

35. **`negative_turning` (Titik Balik Negatif)**
    Jumlah lembah lokal (titik perubahan arah dari turun ke naik).
    $$ N_{\text{neg}} = \sum_{i=2}^{N-1} \mathbb{I}(x_i < x_{i-1} \text{ dan } x_i < x_{i+1}) $$

36. **`positive_turning` (Titik Balik Positif)**
    Jumlah puncak lokal (titik perubahan arah dari naik ke turun).
    $$ N_{\text{pos}} = \sum_{i=2}^{N-1} \mathbb{I}(x_i > x_{i-1} \text{ dan } x_i > x_{i+1}) $$

37. **`neighbourhood_peaks` (Puncak Tetangga)**
    Jumlah puncak lokal yang dominan di dalam batas interval $r$.
    $$ N_{\text{peaks}} = \sum_{i=1+r}^{N-r} \mathbb{I}\left(x_i = \max_{i-r \le j \le i+r} x_j\right) $$

38. **`petrosian_fractal_dimension` (Dimensi Fraktal Petrosian)**
    Dimensi fraktal berbasis tanda selisih sinyal ($D_P$).
    $$ D_P = \frac{\log_{10} N}{\log_{10} N + \log_{10}\left(\frac{N}{N + 0.4 N_{\Delta}}\right)} $$

39. **`pk_pk_distance` (Jarak Peak-to-Peak)**
    Beda tinggi maksimum antara nilai puncak tertinggi dan dasar terendah.
    $$ V_{pp} = \max(x) - \min(x) $$

40. **`slope` (Kemiringan Tren Linier)**
    Gradien garis regresi linier sinyal terhadap waktu.
    $$ \beta = \frac{\sum_{i=1}^{N} (t_i - \bar{t})(x_i - \bar{x})}{\sum_{i=1}^{N} (t_i - \bar{t})^2} $$

41. **`sum_abs_diff` (Jumlah Selisih Absolut)**
    Total kumulatif fluktuasi perubahan antar titik.
    $$ \text{SAD} = \sum_{i=1}^{N-1} |x_{i+1} - x_i| $$

42. **`zero_cross` (Laju Pelintasan Nol)**
    Jumlah berapa kali sinyal berganti tanda positif-negatif.
    $$ \text{ZCR} = \sum_{i=1}^{N-1} \mathbb{I}\big(\text{sgn}(x_i) \neq \text{sgn}(x_{i+1})\big) $$

---

### III. Domain Spectral & Wavelet (26 Fitur)

Domain spektral mengubah sinyal domain waktu menjadi domain frekuensi $P(f_k)$ melalui Transformasi Fourier (FFT) atau Transformasi Wavelet.

43. **`fundamental_frequency` (Frekuensi Dasar)**
    Komponen frekuensi $f_0$ dengan spektrum daya tertinggi.
    $$ f_0 = \arg\max_{f > 0} P(f) $$

44. **`max_frequency` (Frekuensi Maksimum)**
    Frekuensi tertinggi yang memiliki komponen daya bermakna.
    $$ f_{\max} = \max \{ f_k \mid P(f_k) > \epsilon \} $$

45. **`median_frequency` (Frekuensi Median)**
    Frekuensi yang membagi dua total energi daya spektrum.
    $$ \sum_{k=1}^{k_{\text{med}}} P(f_k) = \frac{1}{2} \sum_{k=1}^{K} P(f_k) $$

46. **`human_range_energy` (Energi Rentang Manusia)**
    Konsentrasi energi pada pita frekuensi tertentu.
    $$ E_{\text{human}} = \sum_{f \in [f_1, f_2]} P(f) $$

47. **`lpcc` (Linear Prediction Cepstral Coefficients)**
    Koefisien kepstral berbasis model prediksi linier.
    $$ c_m = a_m + \sum_{k=1}^{m-1} \left(\frac{k}{m}\right) c_k a_{m-k} $$

48. **`mfcc` (Mel-Frequency Cepstral Coefficients)**
    Koefisien kepstral berbasis skala pita filter Mel.
    $$ \text{MFCC}_m = \sum_{k=1}^{K} \log(S_k) \cos\left[ m \left(k - \frac{1}{2}\right) \frac{\pi}{K} \right] $$

49. **`max_power_spectrum` (Spektrum Daya Maksimum)**
    Nilai puncak terbesar dari densitas spektrum daya.
    $$ P_{\max} = \max_{k} P(f_k) $$

50. **`power_bandwidth` (Power Bandwidth)**
    Lebar rentang pita frekuensi yang menampung $99\%$ total energi daya.
    $$ \text{BW}_{\text{power}} = f_{\text{high}} - f_{\text{low}} $$

51. **`spectral_centroid` (Spectral Centroid)**
    Titik pusat berat (gravitasi) spektrum frekuensi.
    $$ C_{\text{spectral}} = \frac{\sum_{k=1}^{K} f_k P(f_k)}{\sum_{k=1}^{K} P(f_k)} $$

52. **`spectral_decrease` (Spectral Decrease)**
    Laju penurunan energi dari frekuensi rendah ke tinggi.
    $$ \text{SD} = \frac{1}{\sum_{k=2}^{K} P(f_k)} \sum_{k=2}^{K} \frac{P(f_k) - P(f_1)}{k - 1} $$

53. **`spectral_distance` (Spectral Distance)**
    Jarak perbedaan bentuk antar spektrum frekuensi berurutan.
    $$ d_{\text{spectral}} = \sqrt{\sum_{k=1}^{K} \big(P_1(f_k) - P_2(f_k)\big)^2} $$

54. **`spectral_entropy` (Spectral Entropy)**
    Ukuran acak/keanekaragaman distribusi energi spektral.
    $$ H_{\text{spectral}} = -\sum_{k=1}^{K} p_k \log_2(p_k), \quad p_k = \frac{P(f_k)}{\sum P(f_j)} $$

55. **`spectral_kurtosis` (Spectral Kurtosis)**
    Tingkat keruncingan puncak distribusi energi spektral.
    $$ K_{\text{spectral}} = \frac{\sum_{k=1}^{K} (f_k - C_{\text{spectral}})^4 P(f_k)}{\text{Spread}^4 \sum P(f_k)} - 3 $$

56. **`spectral_positive_turning` (Spectral Positive Turning Points)**
    Jumlah puncak perbelokan positif pada kurva spektrum frekuensi.
    $$ N_{\text{spec\_pos}} = \sum_{k=2}^{K-1} \mathbb{I}\big(P(f_k) > P(f_{k-1}) \text{ dan } P(f_k) > P(f_{k+1})\big) $$

57. **`spectral_roll_off` (Spectral Roll-Off)**
    Frekuensi ambang bawah di mana $85\%$ total energi spektram terkumpul.
    $$ \sum_{k=1}^{k_{\text{roll}}} P(f_k) = 0.85 \sum_{k=1}^{K} P(f_k) $$

58. **`spectral_roll_on` (Spectral Roll-On)**
    Frekuensi ambang batas tempat akumulasi $5\%$ awal energi spektram.
    $$ \sum_{k=1}^{k_{\text{on}}} P(f_k) = 0.05 \sum_{k=1}^{K} P(f_k) $$

59. **`spectral_skewness` (Spectral Skewness)**
    Derajat asimetri distribusi energi spektral.
    $$ S_{\text{spectral}} = \frac{\sum_{k=1}^{K} (f_k - C_{\text{spectral}})^3 P(f_k)}{\text{Spread}^3 \sum P(f_k)} $$

60. **`spectral_slope` (Spectral Slope)**
    Kemiringan tren garis spektrum daya via regresi linier.
    $$ \text{Slope}_{\text{spectral}} = \frac{K \sum f_k P(f_k) - \sum f_k \sum P(f_k)}{K \sum f_k^2 - (\sum f_k)^2} $$

61. **`spectral_spread` (Spectral Spread)**
    Sebaran kuadratik energi spektral di sekitar nilai *centroid*.
    $$ \text{Spread}_{\text{spectral}} = \sqrt{\frac{\sum_{k=1}^{K} (f_k - C_{\text{spectral}})^2 P(f_k)}{\sum_{k=1}^{K} P(f_k)}} $$

62. **`spectral_variation` (Spectral Variation)**
    Variasi perubahan bentuk spektrum antar kerangka frekuensi.
    $$ \text{SV} = 1 - \frac{\sum_{k=1}^{K} P_1(f_k) P_2(f_k)}{\sqrt{\sum P_1(f_k)^2 \sum P_2(f_k)^2}} $$

63. **`spectrogram_mean_coeff` (Rata-rata Koefisien Spektrogram)**
    Nilai rata-rata dari matriks spektrogram waktu-frekuensi $S(t, f)$.
    $$ \text{Mean}_{\text{spectrogram}} = \frac{1}{T \cdot F} \sum_{t=1}^{T} \sum_{f=1}^{F} |S(t, f)| $$

64. **`wavelet_abs_mean` (Wavelet Absolute Mean)**
    Rata-rata mutlak dari koefisien dekomposisi Wavelet $W_a(b)$.
    $$ \text{Mean}_{\text{wavelet}} = \frac{1}{M} \sum_{m=1}^{M} |W_a(b_m)| $$

65. **`wavelet_energy` (Wavelet Energy)**
    Jumlah total daya energi koefisien Wavelet.
    $$ E_{\text{wavelet}} = \sum_{m=1}^{M} W_a(b_m)^2 $$

66. **`wavelet_entropy` (Wavelet Entropy)**
    Entropi Shannon dari probabilitas distribusi koefisien Wavelet.
    $$ H_{\text{wavelet}} = -\sum_{m=1}^{M} p_m \log_2(p_m), \quad p_m = \frac{W_a(b_m)^2}{E_{\text{wavelet}}} $$

67. **`wavelet_std` (Wavelet Standard Deviation)**
    Deviasi standar dari koefisien dekomposisi Wavelet.
    $$ \sigma_{\text{wavelet}} = \sqrt{\frac{1}{M-1} \sum_{m=1}^{M} (W_a(b_m) - \mu_{\text{wavelet}})^2} $$

68. **`wavelet_var` (Wavelet Variance)**
    Varians dari koefisien dekomposisi Wavelet.
    $$ \sigma^2_{\text{wavelet}} = \frac{1}{M-1} \sum_{m=1}^{M} (W_a(b_m) - \mu_{\text{wavelet}})^2 $$
